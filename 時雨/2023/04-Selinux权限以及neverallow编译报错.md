---
title: Selinux权限以及neverallow编译报错
date: 2023-08-23 16:23:00
category: 折腾笔记
tags:
  - 编程
  - SELinux
---

公司有个项目需要在上层对底层的节点进行数据写入，所以需要修补 `SeLinux` 权限，查阅资料之后实践修改成功，故记录一下。

## 报错

系统内缺少的权限报错如下：

```bash
[ 108.891334] (2)[463:logd.auditd]type=1400 audit(1702696933.400:1585): avc: denied { write } for comm="hiro.rearscreen" name="st_spilcd_dev" dev="tmpfs" ino=19498 scontext=u:r:system_app:s0 tcontext=u:object_r:device:s0 tclass=chr_file permissive=1 [ 108.891458] (2)[463:logd.auditd]type=1400 audit(1702696933.400:1586): avc: denied { open } for comm="hiro.rearscreen" path="/dev/st_spilcd_dev" dev="tmpfs" ino=19498 scontext=u:r:system_app:s0 tcontext=u:object_r:device:s0 tclass=chr_file permissive=1
```

尝试直接补上权限

`allow system_app device:chr_file write;`

但是编译却报了 `neverallow` 报错

## 分析

是什么原因导致的这个编译报错呢？

其实是当前这个 `process` 进程申请的权限过高了。

如果直接加 `system_app` 对 `chr_file` 那是很危险的权限，假如被恶意软件利用系统漏洞提权了，那么就可以轻而易举的对各种 `chr_file` 进行写入，谷歌不允许这种情况出现，所以把它加入了 `neverallow` 里面。
## 解决

那有没有办法绕过这个报错呢？

当然是有的，可以参考如下步骤：

1、先查看一下是哪些相关的代码申请的这个权限，主要是判断一下是申请的属性还是 `device` 节点访问权限等；

2、新增加一个 SELinux type；

根据申请的类型可以进修改一下访问的 `Object` 的 `Label` ，

比如我这里是 `device` 节点，那我就在 `device.te` 中：

`type spilcd_device, dev_type;`

3、绑定文件到这个 SELinux type；

因为我这里知道我访问的是 `/dev/mmcblk1rpmb` 节点，我在第一步中确认的，

所以在 `file_contexts` 中添加如下代码

`/dev/st_spilcd_dev       u:object_r:spilcd_device:s0`

4、修改 `te` 文件，添加 `process/domain` 的访问权限；

这里就比较简单了，直接添加对应的规则就可以了。

`allow system_app spilcd_device:chr_file rw_file_perms;`

```diff
diff --git a/device/mediatek/mt6833/init.mt6833.rc b/device/mediatek/mt6833/init.mt6833.rc
index 39154af8d3d..c6632853351 100644
--- a/device/mediatek/mt6833/init.mt6833.rc
+++ b/device/mediatek/mt6833/init.mt6833.rc
@@ -700,6 +700,8 @@ on post-fs-data
     chown system system /proc/secmem0
 
     chmod 0666 /dev/exm0
+    chmod 0666 /dev/st_spilcd_dev
+    chown system system /dev/st_spilcd_dev
 
 
        #Thermal
diff --git a/device/mediatek/sepolicy/basic/non_plat/device.te b/device/mediatek/sepolicy/basic/non_plat/device.te
index 61601cae974..664419818ba 100644
--- a/device/mediatek/sepolicy/basic/non_plat/device.te
+++ b/device/mediatek/sepolicy/basic/non_plat/device.te
@@ -379,3 +379,5 @@ type ccci_mdmonitor_device, dev_type;
 # Operator: S migration
 # Purpose: Add permission for vilte
 type ccci_vts_device, dev_type;
+
+type spilcd_device, dev_type;
diff --git a/device/mediatek/sepolicy/bsp/non_plat/file_contexts b/device/mediatek/sepolicy/bsp/non_plat/file_contexts
index 409daae398e..a1a88046e84 100644
--- a/device/mediatek/sepolicy/bsp/non_plat/file_contexts
+++ b/device/mediatek/sepolicy/bsp/non_plat/file_contexts
@@ -274,3 +274,5 @@
 #
 /mnt/vendor/persist/t6(/.*)?    u:object_r:tkcore_protect_data_file:s0
 /mnt/vendor/protect_f/tee(/.*)? u:object_r:tkcore_protect_data_file:s0
+
+/dev/st_spilcd_dev       u:object_r:spilcd_device:s0
diff --git a/device/mediatek/sepolicy/bsp/non_plat/system_app.te b/device/mediatek/sepolicy/bsp/non_plat/system_app.te
index 50ece4665cc..d0e1330a9ba 100644
--- a/device/mediatek/sepolicy/bsp/non_plat/system_app.te
+++ b/device/mediatek/sepolicy/bsp/non_plat/system_app.te
@@ -163,3 +163,5 @@ allow system_app mtk_audiohal_data_file:file create_file_perms;
 
 
 hal_client_domain(system_app, hal_fingerprint)
+
+allow system_app spilcd_device:chr_file rw_file_perms;
```