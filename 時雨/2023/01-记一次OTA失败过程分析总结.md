---
title: 记一次OTA失败过程分析总结
date: 2023-01-11 21:00:00
category: 折腾笔记
tags:
  - Android
---

自己到公司的第一个要出货的项目遇到的OTA升级失败的问题，马上过年了，真是折磨，好在后面还是找到了蛛丝马迹，解决了问题。

## 报错

```bash
01-10 15:19:53.124851   966   966 I update_engine: [INFO:dynamic_partition_control_android.cc(331)] Loaded metadata from slot A in /dev/block/by-name/super
01-10 15:19:53.134328   966   966 E update_engine: [ERROR:dynamic_partition_control_android.cc(1153)] Device file /dev/block/by-name/preloader_a does not exist.
01-10 15:19:53.143754   966   966 E update_engine: [ERROR:install_plan.cc(158)] boot_control->GetPartitionDevice( partition.name, source_slot, &partition.source_path) failed.
01-10 15:19:53.153201   966   966 E update_engine: [ERROR:delta_performer.cc(767)] Unable to determine all the partition devices.
01-10 15:19:53.162377   966   966 E update_engine: [ERROR:download_action.cc(222)] Error ErrorCode::kInstallDeviceOpenError (7) in DeltaPerformer's Write method when processing the received payload -- Terminating processing
01-10 15:19:53.211258   966   966 I update_engine: [INFO:libcurl_http_fetcher.cc(623)] Requesting libcurl to terminate transfer.
```

这一条 `01-10 15:19:53.134328   966   966 E update_engine: [ERROR:dynamic_partition_control_android.cc(1153)] Device file /dev/block/by-name/preloader_a does not exist.`

## 分析

```c++
// dynamic_partition_control_android.cc:1151
// Try static partitions.
auto static_path = GetStaticDevicePath(device_dir, partition_name_suffix);
if (!DeviceExists(static_path)) {
    // 此时 static_path = /dev/block/by-name/preloader_a
    LOG(ERROR) << "Device file " << static_path << " does not exist.";
    return {};
}

// dynamic_partition_control_android.cc:1106
// 向上找到调用
std::string device_dir_str;
if (!GetDeviceDir(&device_dir_str)) {
    LOG(ERROR) << "Failed to GetDeviceDir()";
    return {};
}

// 来源其实是 GetDeviceDir()
const base::FilePath device_dir(device_dir_str);

// 查看头文件定义
// dynamic_partition_control_interface.h:148
// device_dir = /dev/block/by-name/ 
// partition_name_suffix = preloader_a

// Finds a possible location that list all block devices by name; and puts
// the result in |path|. Returns true on success.
// Sample result: /dev/block/by-name/
virtual bool GetDeviceDir(std::string* path) = 0;

// 查看方法实现
// dynamic_partition_control_android.cc:392
bool DynamicPartitionControlAndroid::GetDeviceDir(std::string* out) {
  // We can't use fs_mgr to look up |partition_name| because fstab
  // doesn't list every slot partition (it uses the slotselect option
  // to mask the suffix).
  //
  // We can however assume that there's an entry for the /misc mount
  // point and use that to get the device file for the misc
  // partition. This helps us locate the disk that |partition_name|
  // resides on. From there we'll assume that a by-name scheme is used
  // so we can just replace the trailing "misc" by the given
  // |partition_name| and suffix corresponding to |slot|, e.g.
  //
  //   /dev/block/platform/soc.0/7824900.sdhci/by-name/misc ->
  //   /dev/block/platform/soc.0/7824900.sdhci/by-name/boot_a
  //
  // If needed, it's possible to relax the by-name assumption in the
  // future by trawling /sys/block looking for the appropriate sibling
  // of misc and then finding an entry in /dev matching the sysfs
  // entry.
    
  // 调用的 get_bootloader_message_blk_device
  std::string err, misc_device = get_bootloader_message_blk_device(&err);
  if (misc_device.empty()) {
    LOG(ERROR) << "Unable to get misc block device: " << err;
    return false;
  }

  if (!utils::IsSymlink(misc_device.c_str())) {
    LOG(ERROR) << "Device file " << misc_device << " for /misc "
               << "is not a symlink.";
    return false;
  }
  *out = base::FilePath(misc_device).DirName().value();
  return true;
}

// bootable/recovery/bootloader_message/bootloader_message.cpp
// 这里又调用了get_misc_blk_device
std::string get_bootloader_message_blk_device(std::string* err) {
  std::string misc_blk_device = get_misc_blk_device(err);
  if (misc_blk_device.empty()) return "";
  if (!wait_for_device(misc_blk_device, err)) return "";
  return misc_blk_device;
}

// bootable/recovery/bootloader_message/bootloader_message.cpp
std::string get_misc_blk_device(std::string* err) {
  if (g_misc_device_for_test.has_value() && !g_misc_device_for_test->empty()) {
    return *g_misc_device_for_test;
  }
  Fstab fstab;// 这里就是查找fstab表了
  if (!ReadDefaultFstab(&fstab)) {
    *err = "failed to read default fstab";
    return "";
  }
  for (const auto& entry : fstab) {
    if (entry.mount_point == "/misc") {
      return entry.blk_device;
    }
  }

  *err = "failed to find /misc partition";
  return "";
}
```

流程已经很清楚了，这个时候进系统把文件导出看看

```shell
cat out\target\product\k62v1_64_bsp\vendor\etc\fstab.mt6765
# 基本都是都是 /dev/block/by-name/*
/dev/block/by-name/para /misc emmc defaults defaults
```

所以 `static_path` 是这里拿的路径，前缀是 `/dev/block/by-name/`

所以当 OTA 升级走到 `preloader` 的时候，就找不到这个路径

现在对比看一下软连接：

```shell
# 修改前
ls -al /dev/block/by-name/
lrwxrwxrwx 1 root root   20 2010-01-01 08:00 para -> /dev/block/mmcblk0p2
lrwxrwxrwx 1 root root   21 2010-01-01 08:00 persist -> /dev/block/mmcblk0p12
lrwxrwxrwx 1 root root   22 2010-01-01 08:00 preloader_raw_a -> /dev/block/mapper/pl_a
lrwxrwxrwx 1 root root   22 2010-01-01 08:00 preloader_raw_b -> /dev/block/mapper/pl_b
lrwxrwxrwx 1 root root   21 2010-01-01 08:00 proinfo -> /dev/block/mmcblk0p14
lrwxrwxrwx 1 root root   20 2010-01-01 08:00 protect1 -> /dev/block/mmcblk0p9
lrwxrwxrwx 1 root root   21 2010-01-01 08:00 protect2 -> /dev/block/mmcblk0p10
# 修改后
lrwxrwxrwx 1 root root   20 2010-01-01 08:00 para -> /dev/block/mmcblk0p2
lrwxrwxrwx 1 root root   21 2010-01-01 08:00 persist -> /dev/block/mmcblk0p12
# 在init.mt6765.rc以及init.recovery.mt6765.rc加上的 {
lrwxrwxrwx 1 root root   23 2010-01-01 08:00 preloader_a -> /dev/block/mmcblk0boot0
lrwxrwxrwx 1 root root   23 2010-01-01 08:00 preloader_b -> /dev/block/mmcblk0boot1
# 在init.mt6765.rc以及init.recovery.mt6765.rc加上的 }
lrwxrwxrwx 1 root root   22 2010-01-01 08:00 preloader_raw_a -> /dev/block/mapper/pl_a
lrwxrwxrwx 1 root root   22 2010-01-01 08:00 preloader_raw_b -> /dev/block/mapper/pl_b
lrwxrwxrwx 1 root root   21 2010-01-01 08:00 proinfo -> /dev/block/mmcblk0p14
lrwxrwxrwx 1 root root   20 2010-01-01 08:00 protect1 -> /dev/block/mmcblk0p9
lrwxrwxrwx 1 root root   21 2010-01-01 08:00 protect2 -> /dev/block/mmcblk0p10
```

同时查看搜索查看其他平台代码

```shell
:~/MTK_S0MP1/device/mediatek$ grep -nr "symlink /dev/block/mmcblk0boot0 /dev/block/by-name/preloader_a"
mt6877/init.mt6877.rc:172:    symlink /dev/block/mmcblk0boot0 /dev/block/by-name/preloader_a
mt6761/init.recovery.mt6761.rc:17:    symlink /dev/block/mmcblk0boot0 /dev/block/by-name/preloader_a
mt6761/init.mt6761.rc:197:    symlink /dev/block/mmcblk0boot0 /dev/block/by-name/preloader_a
mt6768/init.mt6768.rc:170:    symlink /dev/block/mmcblk0boot0 /dev/block/by-name/preloader_a
mt6768/init.recovery.mt6768.rc:18:    symlink /dev/block/mmcblk0boot0 /dev/block/by-name/preloader_a
mt6853/init.mt6853.rc:165:    symlink /dev/block/mmcblk0boot0 /dev/block/by-name/preloader_a
mt6833/init.recovery.mt6833.rc:17:    symlink /dev/block/mmcblk0boot0 /dev/block/by-name/preloader_a
mt6833/init.mt6833.rc:164:    symlink /dev/block/mmcblk0boot0 /dev/block/by-name/preloader_a
mt6765/init.recovery.mt6765.rc:19:    symlink /dev/block/mmcblk0boot0 /dev/block/by-name/preloader_a
mt6765/init.mt6765.rc:166:    symlink /dev/block/mmcblk0boot0 /dev/block/by-name/preloader_a
```

感觉就像是 MTK 忘了加上去，或者是我们老大合 Patch 的时候遗漏了

## 解决

```diff
on post-fs
    # Support A/B feature for emmc boot region
    symlink /dev/block/sda /dev/block/mmcblk0boot0
    symlink /dev/block/sdb /dev/block/mmcblk0boot1
    symlink /dev/block/mmcblk0boot0 /dev/block/platform/bootdevice/by-name/preloader_a
    symlink /dev/block/mmcblk0boot1 /dev/block/platform/bootdevice/by-name/preloader_b
+    symlink /dev/block/mmcblk0boot0 /dev/block/by-name/preloader_a
+    symlink /dev/block/mmcblk0boot1 /dev/block/by-name/preloader_b

    exec u:r:update_engine:s0 root root -- /system/bin/mtk_plpath_utils
```
