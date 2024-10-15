
# Windows NDK环境

在 Android Studio 中下载 NDK。
将 ndk 的 bin 目录添加到环境变量中。

`%ANDROID_NDK%\toolchains\llvm\prebuilt\windows-x86_64\bin`

新建 `hello.c` 文件，输出 `Hello, world!` 。

使用 `aarch64-linux-android21-clang.cmd hello.c` 编译。

使用 `adb push` 将编译后的 `a.out` 文件推送到手机 `/data/local/tmp` 目录。

`adb push a.out /data/local/tmp`

使用 `adb shell` 进入手机的命令行，添加执行权限 `chmod +x a.out` ，运行 `./a.out` 。

`adb shell`
`cd /data/local/tmp`
`chmod +x a.out`
`./a.out`

# Ubuntu 环境

## 安装 Ubuntu 虚拟机

https://cn.ubuntu.com/download/alternative-downloads

### 常用命令

系统管理：
* 重启：`sudo reboot`
* 关机：`sudo poweroff`
* 使用 root 运行命令：`sudo <命令>`
* 获取命令路径：`which <命令>`

使用 apt-get 命令安装软件：
* 更新软件源：`sudo apt-get update`
* 更新所有已安装软件：`sudo apt-get upgrade`
* 安装软件：`sudo apt-get install <软件名>`
* 卸载软件：`sudo apt-get remove <软件名>`
* 搜索软件：`apt-cache search <软件名>`
* 查看已安装软件：`apt list --installed`
* 查看软件版本：`apt-cache show <软件名>`

使用 dpkg 命令安装软件：
* 安装软件：`sudo dpkg -i <软件名>.deb`
* 卸载软件：`sudo dpkg -r <软件名>`

文件操作：
* 列出目录下所有文件：`ls`
* 进入目录：`cd <目录名>`
* 创建目录：`mkdir <目录名>`
* 删除目录：`rmdir <目录名>`
* 复制文件：`cp <源文件> <目标文件>`
* 移动文件：`mv <源文件> <目标文件>`
* 删除文件：`rm <文件名>`
* 查看文件内容：`cat <文件名>`
* 查看当前目录：`pwd`
* 修改文件权限：`chmod <权限> <文件名>`
* 文件内容搜索：`grep <字符串> <文件名>`
* 文件名查找：`find <目录名> -name <文件名>`

进程管理：
* 列出所有进程：`ps -ef`
* 杀死进程：`kill <进程号>`
* 查看进程信息：`top` 或 `htop`

bash 语法：
* 定义变量：`name=value`
* 导出为环境变量：`export name=value`
* 命令别名：`alias name='command'`
* 打印变量值：`echo $name`
* 使用变量：`$name` 或 `${name}`
* 管道命令：`command1 | command2`

常用软件：
* htop：查看系统资源占用：`sudo apt-get install htop`
* vim：编辑器：`sudo apt-get install vim`
  * 创建或打开文件：`vim <文件名>`
  * 进入编辑模式：按 `i`
  * 退出编辑模式：按 `Esc` 再按 `:` 再按 `wq` 保存并退出
* vmtools：虚拟机管理工具：
  * `sudo apt-get install open-vm-tools`
  * `sudo apt-get install open-vm-tools-desktop`
* Android Studio：Android 开发 IDE： https://developer.android.com/studio/install
* vscode：代码编辑器：`sudo snap install code --classic`

## 安装 NDK

在 Android Studio 中下载 NDK，或者手动下载：https://developer.android.com/ndk/downloads

### 配置环境变量

1. 打开 `~/.bashrc` 文件：`sudo vim ~/.bashrc`
2. 在文件末尾添加以下内容：
```
export ANDROID_SDK=/home/xichang/Android/Sdk/
export ANDROID_NDK=/home/xichang/Android/Sdk/ndk/28.0.12433566/
export PATH=$PATH:$ANDROID_SDK/platform-tools/:$ANDROID_NDK/toolchains/llvm/prebuilt/linux-x86_64/bin/
```
3. 保存并退出，执行 `source ~/.bashrc` 命令使环境变量生效。

### 配置 usb 规则

查看设备：`lsusb`
```
Bus 002 Device 002: ID 18d1:4ee7 Google Inc. Nexus/Pixel Device (charging + debug)
```

1. 创建 usb 规则文件：`sudo vim /etc/udev/rules.d/70-android.rules`
2. 在文件末尾添加以下内容：
  * `ATTRS{idVendor}=="18d1"` 表示匹配厂商 ID 为 18d1 的设备
  * `ATTRS{idProduct}=="4ee7"` 表示匹配产品 ID 为 4ee7 的设备
  * `MODE="0666"` 表示允许读写权限
  * `GROUP="plugdev"` 表示添加到 plugdev 组
  * `SYMLINK+="android_device"` 表示创建符号链接，方便后续使用
```
SUBSYSTEM=="usb", ATTRS{idVendor}=="18d1", ATTRS{idProduct}=="4ee7", MODE="0666", GROUP="plugdev", SYMLINK+="android_device", SYMLINK+="android_adb"
SUBSYSTEM=="usb", ATTRS{idVendor}=="18d1", ATTRS{idProduct}=="4ee0", MODE="0666", GROUP="plugdev", SYMLINK+="android_device", SYMLINK+="android_fastboot"
```
3. 保存并退出，执行 `sudo udevadm control --reload-rules` 命令使规则生效。
4. 重启 usb :
  * `sudo service udev restart`
  * `sudo udevadm trigger`
5. 重启 adb ：
  * `adb kill-server`
  * `adb start-server`

## 其他
`proxychains` 给单个程序代理
* `sodo apt install proxychains4`
* 编辑配置文件 `/etc/.proxychains4.conf`
  * 修改 `socks4 127.0.0.1 1080` 为 `socks4 127.0.0.1 7890`
* 使用 `proxychains4 curl www.google.com`

electerm 使用 ssh 连接 Ubuntu 虚拟机 https://github.com/electerm/electerm

LocalSend 局域网传输工具 https://localsend.org/zh-CN/download