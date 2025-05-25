- [安装 `Android Studio`](#安装-android-studio)
- [输入出厂镜像](#输入出厂镜像)
- [刷入 `twrp`](#刷入-twrp)
- [安装面具获取 root 权限：](#安装面具获取-root-权限)
- [其他工具：](#其他工具)
  - [终端模拟器 termux](#终端模拟器-termux)
  - [开源软件商店 F-Droid](#开源软件商店-f-droid)
- [WiFi 图标不正确](#wifi-图标不正确)
- [安卓结构](#安卓结构)
  - [结构预览](#结构预览)

# 安装 `Android Studio`
下载网址： https://developer.android.google.cn/studio?hl=zh-cn

# 刷入出厂镜像
1. 下载镜像： https://developers.google.com/android/images?hl=zh-cn
2. 解压镜像： `sailfish-opm1.171019.011-factory-56d15350.zip`
3. 连接手机运行 `adb reboot bootloader` 命令 -> 运行解压后的 `flash-all.bat` 脚本
4. 运行 `fastboot reboot` 命令重启手机 -> 刷入成功

# 下载 `twrp`
下载地址： https://twrp.me/Devices/
下载 `twrp` ： https://dl.twrp.me/sailfish/twrp-3.3.0-0-sailfish.images

# 刷入面具获取 root 权限：
1. 下载 `Magisk` 安装包 ： https://magiskcn.com/magisk-download
2. 运行 `adb push magisk-v27.0.zip /sdcard/magisk.zip` 命令将 `Magisk` 安装包推送到手机
3. 运行 `adb reboot bootloader` 命令 -> 运行 `fastboot boot recovery twrp-3.3.0-0-sailfish.img` 命令使用 `twrp` 镜像
4. 在 `twrp` 界面中选择 `Install`  -> 选择 `magisk.zip` 安装包 -> 点击 `Reboot System` 重启手机
5. 运行 `adb install -r magisk.apk` 命令安装 `Magisk` 应用
6. 安装完成后重启手机

# 其他工具：
## 终端模拟器 termux
https://termux.dev/en/
## 开源软件商店 F-Droid
https://f-droid.org/zh_Hans/packages/

# WiFi 图标不正确
1. 下载 `CaptiveMgr`
2. 运行 `adb install CaptiveMgr.apk` 命令安装 `CaptiveMgr` 应用
3. 无法上网时，需要修改系统时间

# 安卓结构
## 结构预览
安卓程序，一般后缀名为 `.apk`，由三部分组成：
* `AndroidManifest.xml`： 应用的配置文件里面包含：
  * 应用的组件
  * 应用为了访问系统或其他应用的受保护部分而需要的具体权限
  * 应用所需的硬件和软件功能
* `classes.dex`： 应用的代码文件
* `resources.arsc`： 应用的资源文件

