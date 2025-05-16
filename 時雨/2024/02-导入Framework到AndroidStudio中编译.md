---
title: 导入Framework到AndroidStudio中编译
date: 2024-01-16 15:52:00
category: 学习笔记
tags:
  - Android
  - Framework
---

平时有时候需要用到 Framework 中 `@hide` 的方法来编写系统APP，所以需要导入 Framework 到 Android Studio 中参与编译，有的时候也需要用 IDE 来修改如 Framework 或者 Settings 等，故记录一下把 Framework 导入到 Android Studio 中。

## Framework 作为依赖导入编译

`build.gradle` 里面，

加进去了虽然 Android Studio 爆红但是可以编译，

进 `out/target/common/obj/JAVA_LIBRARIES/framework-minus-apex_intermediates/classes.jar` 拷贝出来，

添加完直接 `gradle sync` 或者对着 `jar` 右击 `add as lib` 也可以。

```groovy
allprojects {
    gradle.projectsEvaluated {
        tasks.withType(JavaCompile) {
            Set<File> fileSet = options.bootstrapClasspath.getFiles()
            List<File> newFileList =  new ArrayList<>();
            newFileList.add(new File("libs/framework.jar"))
            newFileList.addAll(fileSet)
            options.bootstrapClasspath = files(
                    newFileList.toArray()
            )
        }
    }
}
dependencies {
    compileOnly files('libs/framework.jar')
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'androidx.preference:preference:1.2.0'
}
```

build.gradle.kts

```kotlin
// 加在 android 大括号里面
applicationVariants.all {  
 outputs.all {  
  if (this is ApkVariantOutputImpl) {  
   outputFileName = "SalesTracker.apk"  
  }  
 }  
}

dependencies {  
 implementation("androidx.core:core-ktx:1.10.1")  
 implementation(files("/libs/nvram.jar"))  
 compileOnly(files("/libs/framework.jar"))  
}
```

## 编译系统 APP 准备工作

如果要 `share uid` 要在 `AndroidManifest.xml` 加：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"  
xmlns:tools="http://schemas.android.com/tools"  
coreApp="true"  
android:sharedUserId="android.uid.system">
```

先根据代码生成签名密钥：

```bash
# 看有没有开 MTK_SIGNATURE_CUSTOMIZATION=yes
# 开了宏的话，走不同路径的，当然一般把这两个地方的密钥都替换掉
device\mediatek\security
build\target\product\security

openssl pkcs8 -inform DER -nocrypt -in platform.pk8 -out platform.pem

openssl pkcs12 -export -in platform.x509.pem -inkey platform.pem -out platform.p8 -password pass:android -name android

keytool -importkeystore -deststorepass android -destkeystore .keystore -srckeystore platform.p8 -srcstoretype PKCS12 -srcstorepass android

keytool -list -v -keystore .keystore

mv .keystore platform.keystore
```

复制 `platform.keystore` 到我们自己的APP目录 `project/app/`，

然后解锁设备：

```bash
adb root
adb shell remount
adb shell mkdir /system/priv-app/NvRamTest/
adb push .\NvRamTest.apk /system/priv-app/NvRamTest/
adb reboot
```

## Android Studio 导入 framework 源代码

先进行完整编译，

编译源码idegen模块及生成AS配置文件 `*.ipr`，

```bash
mmm development/tools/idegen/
./development/tools/idegen/idegen.sh

aidegen packages/apps/Settings frameworks/base

aidegen Settings framework -s
```

导入 Android Studio 前的一些操作，修改 `android.iml` 文件（将自己不用的代码去掉），

如下就是部分列表，可以自行删减：

```xml
<sourceFolder url="file://$MODULE_DIR$/./sdk/testapps/userLibTest/src" isTestSource="true"/>
<sourceFolder url="file://$MODULE_DIR$/./tools/external/fat32lib/src/main/java" isTestSource="false"/>
<excludeFolder url="file://$MODULE_DIR$/out/eclipse"/>
<excludeFolder url="file://$MODULE_DIR$/.repo"/>
<excludeFolder url="file://$MODULE_DIR$/external/bluetooth"/>
<excludeFolder url="file://$MODULE_DIR$/external/chromium"/>
<excludeFolder url="file://$MODULE_DIR$/external/icu4c"/>
<excludeFolder url="file://$MODULE_DIR$/external/webkit"/>
<excludeFolder url="file://$MODULE_DIR$/frameworks/base/docs"/>
<excludeFolder url="file://$MODULE_DIR$/out/host"/>
<excludeFolder url="file://$MODULE_DIR$/out/target/common/docs"/>
<excludeFolder url="file://$MODULE_DIR$/out/target/common/obj/JAVA_LIBRARIES/android_stubs_current_intermediates"/>
<excludeFolder url="file://$MODULE_DIR$/out/target/product"/>
<excludeFolder url="file://$MODULE_DIR$/prebuilt"/>
```
