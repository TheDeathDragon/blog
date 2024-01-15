---
title: Android签名Key的生成与使用
date: 2023-10-17 21:53:00
category: 折腾笔记
tags:
  - 编程
  - 脚本
  - Android
---

平时一直有客户索要平台 key 来进行客户 APP 的开发，同时项目在进行 GMS 认证的过程中，也是需要生成并替换掉默认 Key 的，否则无法通过 CTS 的测试，自己也写了一个脚本来快速对 APP 进行签名，故记录一下。

## 手动生成密钥

```bash
# 如果没有密钥可以先创建一个，有就直接用现有的
openssl genpkey -algorithm RSA -out key_rsa2048.pem -aes256
```
## 手动根据私钥生成 X509 和 PK8 密钥

```bash
openssl pkcs8 -topk8 -inform PEM -outform DER -in key_rsa2048.pem -out platform.pk8 -nocrypt

# 直接回车就好
openssl req -key key_rsa2048.pem -new -x509 -out platform.x509.pem
```
## 通过平台脚本生成 PEM

```bash
# Generate .rnd file
cd ~
openssl rand -writerand .rnd
cd -
# Generate the release key
development/tools/make_key releasekey '/C=CN/ST=Shenzhen/L=BaoAn/O=Shiro/OU=Shiro/CN=Shiro/emailAddress=me@Shiro.La'
```

## 通过 PK8 密钥生成 KeyStore

```bash
# 服务器上进入对于目录执行
# AOSP build\target\product\security
# MTK device\mediatek\security

openssl pkcs8 -inform DER -nocrypt -in platform.pk8 -out platform.pem

openssl pkcs12 -export -in platform.x509.pem -inkey platform.pem -out platform.p8 -password pass:android -name android

keytool -importkeystore -deststorepass android -destkeystore .keystore -srckeystore platform.p8 -srcstoretype PKCS12 -srcstorepass android

keytool -list -v -keystore .keystore

mv .keystore mtk.keystore

openssl pkcs8 -inform DER -nocrypt -in testkey.pk8 -out testkey.pem

openssl pkcs12 -export -in testkey.x509.pem -inkey testkey.pem -out testkey.p8 -password pass:android -name android

keytool -importkeystore -deststorepass android -destkeystore .keystore -srckeystore testkey.p8 -srcstoretype PKCS12 -srcstorepass android

keytool -list -v -keystore .keystore

mv .keystore testkey.keystore
```
## 把 Keystore 转换成 Jks

```bash
# 得配置好JDK环境变量先
keytool -importkeystore -srckeystore platform.keystore -destkeystore platform.p12 -deststoretype PKCS12
keytool -importkeystore -srckeystore platform.p12 -destkeystore platform.jks -deststoretype pkcs12

keytool -importkeystore -srckeystore mtk.keystore -destkeystore mtk.p12 -deststoretype PKCS12
keytool -importkeystore -srckeystore mtk.p12 -destkeystore mtk.jks -deststoretype pkcs12
```
## 生成 AVB_PUBKEY

如果要进行GMS认证的话，最好还是把 avb的key一并替换了，

替换完毕之后，还要手动测试一下：

`run cts -m CtsAppSecurityHostTestCases -t android.appsecurity.cts.ApexSignatureVerificationTest#testApexPubKeyIsNotWellKnownKey`

```bash
bionic/apex/com.android.runtime.avbpubkey
bionic/apex/com.android.runtime.pem
bionic/apex/com.android.runtime.pk8
bionic/apex/com.android.runtime.x509.pem
build/make/target/product/security/bluetooth.pk8
build/make/target/product/security/bluetooth.x509.pem
build/make/target/product/security/cts_uicc_2021.pk8
build/make/target/product/security/cts_uicc_2021.x509.pem
build/make/target/product/security/media.pk8
build/make/target/product/security/media.x509.pem
build/make/target/product/security/networkstack.pk8
build/make/target/product/security/networkstack.x509.pem
build/make/target/product/security/platform.pk8
build/make/target/product/security/platform.x509.pem
build/make/target/product/security/sdk_sandbox.pk8
build/make/target/product/security/sdk_sandbox.x509.pem
build/make/target/product/security/shared.pk8
build/make/target/product/security/shared.x509.pem
build/make/target/product/security/testkey.pk8
build/make/target/product/security/testkey.x509.pem
packages/modules/RuntimeI18n/apex/com.android.i18n.avbpubkey
packages/modules/RuntimeI18n/apex/com.android.i18n.pem
packages/modules/RuntimeI18n/apex/com.android.i18n.pk8
packages/modules/RuntimeI18n/apex/com.android.i18n.x509.pem
```

```bash
development/tools/make_key com.android.runtime '/C=CN/ST=Shenzhen/L=BaoAn/O=Shiro/OU=Shiro/CN=Shiro/emailAddress=me@Shiro.La'

openssl genrsa -out com.android.runtime.pem 4096
avbtool extract_public_key --key com.android.runtime.pem --output com.android.runtime.avbpubkey
```
## 签名 Bat 脚本

需要把 `zipalign.exe` 和 `apksigner.jar` 放到同一目录下，

这两个文件可以从 Android SDK 的 `build-tools` 中找到，

必须要有 Java 环境并且配置好环境变量，

通过前文的步骤生成 `.jks` 格式的密钥，

然后把脚本保存为 `sign.bat` 和前者放在一起，把待签名的 APK 也放在同一目录，

然后把 APK 拖到脚本的图标上面即可。

```powershell
@echo off
title APK Signer by RinShiro
SETLOCAL ENABLEDELAYEDEXPANSION
Color 3f
rem change KeyStorePath ALIAS_NAME STORE_PASS KEY_PASS
Set KeyStorePath="mtk.jks"
Set ALIAS_NAME="android"
Set STORE_PASS="android"
Set KEY_PASS="android"
Set ZIP_ALIGN="zipalign.exe"
Set APKSIGNER_PATH="apksigner.jar"

rem apk file name check
SET CLASSPATH=%1
Set FILE_PATH=%~dp1%~n1
Set FILE_NAME=%~nx1
Set FILE_DIR=%~dp1
Set ZIPALIGNED_PATH="%FILE_PATH%.zipaligned.apk"
Set SIGNED_FILE="%FILE_PATH%.signed.apk"


If %~x1==.apk (
	echo  apk file name check
	echo   - %CLASSPATH%
	echo  apk zipalign
	zipalign -f -v 4 "%CLASSPATH%" "%ZIPALIGNED_PATH%"
	echo.
	echo.
	Echo  apk zipalign signer
	echo   - %ZIPALIGNED_PATH%
	echo.
	echo.
	echo  apk signing
	echo.
	echo.
	java -jar %APKSIGNER_PATH% sign -v --out "%SIGNED_FILE%" --ks %KeyStorePath% --ks-pass pass:%STORE_PASS% --key-pass pass:%KEY_PASS% --ks-key-alias %ALIAS_NAME% "%ZIPALIGNED_PATH%" 
	Color 0C
	echo.
	echo.
	echo.
	echo  please check the console output
	Echo  if the console output has no error, the signature is successful
	Pause>nul
	Color 3f
	echo.
	echo.
	Echo  verify signature
	echo   - %SIGNED_FILE%
	echo.
	java -jar %APKSIGNER_PATH% verify -v --print-certs "%SIGNED_FILE%" | more
	echo.
	echo.
	echo  apk signing completed
	Pause>nul
	exit
)

echo  not a valid apk file
echo   - %CLASSPATH%
echo.
echo.
echo  exit
Pause>nul
endlocal
exit
```
