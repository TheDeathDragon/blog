---
title: 使用AndroidStudio开发系统应用代码技巧
date: 2024-08-14 15:22:51
category: 学习笔记
tags:
  - Android
  - Framework
---

如何通过使用 Android Studio 代替 Notepad++ 或 VSCode 进行复杂功能的开发，以获得更好的代码提示、跳转和格式化体验。

## 前言

我们工作的日常中，仅使用 Notepad++ 或者 VSCode 对代码进行编辑只能进行简单的修改，

如果要进行比较复杂的功能客制化之类的，建议还是使用 Android Studio 进行编辑开发，这样也有代码提示和补全。

查找方法定义、调用、实现也方便，还可以把代码格式化一下。

但是如果直接使用 AS 打开代码文件夹，代码基本上是一片红的，

因为系统应用使用了很多 `@hidden` 的方法，默认的Android SDK是无法索引的，这样很多代码就无法使用代码补全，

现在就教大家如何导入一些库，消掉这些代码爆红，提高我们开发效率。

另外，不要直接在服务器中直接打开文件夹，会导致服务器卡顿，

在服务器上 `zip -r App名字 App目录` 先把代码打成压缩包，复制回自己电脑上，

修改了代码之后，使用 `BeyoundCompare` 进行代码的对比，把修改对比过去服务器上再进行编译，

虽然说多了一点操作，但是这样不会影响其他人使用服务器。

## 步骤

假设我们要修改系统设置 `MtkSettings` 我们先去服务器上，打包一份代码

```bash
cd ./mssi_A14/vendor/mediatek/proprietary/packages/apps/
zip -r MtkSettings.zip MtkSettings/
```

然后把 `MtkSettings.zip` 复制到自己电脑上的一个地方，注意不要有中文路径，

接着在文件夹里面搜索带有 `gradle` 的文件，全部删掉，这一步很重要！！！

然后直接使用 AS 打开这个目录，等待项目索引完毕

### 替换 android.jar

这一步也适用于正常开发系统应用，这样就可以直接使用系统API了，

不过 AS 中可以提示补全和运行的时候有没有权限调用是两码事

首先点击进入项目结构菜单

![image-20240606140507784](/IMAGES/使用AndroidStudio开发系统应用代码技巧/image-20240606140507784.webp)

找到 SDK 的目录，查看 SDK 中 `android.jar` 是在哪里，后面使用去掉 `@hidden` 的 `android.jar` 替换掉，

这个文件可以去 [github.com/Reginer/aosp-android-jar](https://github.com/Reginer/aosp-android-jar) 的 release 下载

比如说我自己电脑上的路径是 `C:\Users\用户名\AppData\Local\Android\Sdk\platforms\android-34\`

![image-20240606141455452](/IMAGES/使用AndroidStudio开发系统应用代码技巧/image-20240606141455452.webp)

这个只要替换一次，后面其他项目就不用替换了

### 添加依赖库

打开 `Libraries` 标签页

![image-20240606141930696](/IMAGES/使用AndroidStudio开发系统应用代码技巧/image-20240606141930696.webp)

点击 **+** 号 ，添加一个 `java` 库，为了保险起见，我们这里再次添加我们刚才替换用的 `android.jar`

![image-20240606142026266](/IMAGES/使用AndroidStudio开发系统应用代码技巧/image-20240606142026266.webp)

这个时候可能会弹出来，让我们选择添加这个库到哪个模块，

我们按快捷键，`CTRL+A` 全选就好了，然后点 OK

![image-20240606142506124](/IMAGES/使用AndroidStudio开发系统应用代码技巧/image-20240606142506124.webp)

也可以批量添加库文件

![image-20240606142651358](/IMAGES/使用AndroidStudio开发系统应用代码技巧/image-20240606142651358.webp)

我后面会把常用的库都整理出来，

想知道自己的代码里面用到哪些库了，这个可以通过查看 `Android.mk` 或者 `Android.bp` 知道，

例如我们的 `MtkSettings` 的目录中，是用的 `Android.bp` 来编译的，我们可以看到，使用了很多 `androidx` 的依赖

![image-20240606142917020](/IMAGES/使用AndroidStudio开发系统应用代码技巧/image-20240606142917020.webp)

这个些库文件应该去哪里找呢？

我们打开服务器上的代码路径，在 `prebuilts\sdk\current` 当中，

有 `androidx` 以及其他库的文件夹，我们进去到 `androidx` 目录里面，

`prebuilts\sdk\current\androidx\m2repository\androidx`

比如说我们要找 `androidx.appcompat_appcompat` 这个库，顺着目录打开进去，

`prebuilts\sdk\current\androidx\m2repository\androidx\appcompat\appcompat\1.7.0-alpha03\`

可以看到一个后缀为 `.aar` 的文件，我们使用 `7zip` 打开这个 `.aar` 文件，

把里面的 `class.jar` 复制出来，并且给它改个名字，这个就是我们需要的库文件，

最好复制到自己电脑上的固定的一个地方，比如我就放在了 `D:\AospLibs`

![image-20240606143724594](/IMAGES/使用AndroidStudio开发系统应用代码技巧/image-20240606143724594.webp)

![image-20240606143824306](/IMAGES/使用AndroidStudio开发系统应用代码技巧/image-20240606143824306.webp)

准备好库文件之后，我们只要返回 AS 进行库的导入操作即可

### 添加 R 文件依赖

只是导入 `android.jar` 的话，应用本身的资源文件还是会爆红的

我们需要自己去代码编译出来的 `out_sys` 目录中找我们自己要编辑的 APP 的 `jar` 包，

这个里面会包含资源文件，也是就是 R 文件

这里说一下怎么去找，假如我们是 `MtkSettings` 我们去 Android.bp 里面看下资源文件是在哪个模块

![image-20240606145213029](/IMAGES/使用AndroidStudio开发系统应用代码技巧/image-20240606145213029.webp)

然后我们去 `out_sys_debug` 里面搜索一下 `MtkSettings-core`

```bash
cd out_sys
find -name "*.jar" | grep turbine-combined | grep MtkSettings-core
# 打印
./soong/.intermediates/vendor/mediatek/proprietary/packages/apps/MtkSettings/MtkSettings-core/android_common/turbine-combined/MtkSettings-core.jar
```

我们把这个文件也复制出来，添加到我们的 AS 当中，这样 R 的资源文件也有补全提示了，

这样除了第三方库之外，基本都有补全了，第三方库文件我们如果有对应的 `jar` 文件也可以直接添加进去

## 参考

基本没有报红了，偶尔有也是第三方库，或者新增了资源文件没有更新 R 导致的，

在 AS 里面写代码还是非常舒服的，就是一开始准备稍微麻烦一点

![image-20240606145808177](/IMAGES/使用AndroidStudio开发系统应用代码技巧/image-20240606145808177.webp)

## 提取 AAR 脚本

放在 SYS 的代码中运行，就可以自动提取全部 `androidx` 的库文件了，放在 `./libs` 文件夹里面

```bash
#!/bin/bash
CUR_DIR=$(pwd)
echo "CUR_DIR: $CUR_DIR"
TOP_DIR_NAME=$(dirname $CUR_DIR)
echo "TOP_DIR_NAME: $TOP_DIR_NAME"
ANDROID_X_DIR=$CUR_DIR/prebuilts/sdk/current/androidx/m2repository/androidx
echo "ANDROID_X_DIR: $ANDROID_X_DIR"

cd $ANDROID_X_DIR
echo "CUR_DIR: $(pwd)"

if [ ! -d $ANDROID_X_DIR ]; then
  echo "prebuilts not exist, please check is run in mssi_A13 or mssi_A14"
  exit 1
fi

mkdir -p $CUR_DIR/libs

find . -name "*.aar" -type f | while read aar_file; do
  aar_filename=$(basename "$aar_file")
  aar_basename="${aar_filename%.aar}"
  
  temp_dir=$(mktemp -d)
  
  unzip -q "$aar_file" -d "$temp_dir"
  
  if [ -f "$temp_dir/classes.jar" ]; then
    mv -v "$temp_dir/classes.jar" "$CUR_DIR/libs/${aar_basename}.jar"
  else
    echo "classes.jar not found in $aar_file"
  fi
  
  rm -rf "$temp_dir"
done

echo "All AAR files processed and classes.jar files moved to ./libs"

cd $CUR_DIR
find ./libs -type f
echo
echo "Please check the jar files in ./libs"
echo "Done"
echo
```

只需要运行一次导入即可。
