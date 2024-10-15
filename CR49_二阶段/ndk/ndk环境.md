
# Windows NDK����

�� Android Studio ������ NDK��
�� ndk �� bin Ŀ¼��ӵ����������С�

`%ANDROID_NDK%\toolchains\llvm\prebuilt\windows-x86_64\bin`

�½� `hello.c` �ļ������ `Hello, world!` ��

ʹ�� `aarch64-linux-android21-clang.cmd hello.c` ���롣

ʹ�� `adb push` �������� `a.out` �ļ����͵��ֻ� `/data/local/tmp` Ŀ¼��

`adb push a.out /data/local/tmp`

ʹ�� `adb shell` �����ֻ��������У����ִ��Ȩ�� `chmod +x a.out` ������ `./a.out` ��

`adb shell`
`cd /data/local/tmp`
`chmod +x a.out`
`./a.out`

# Ubuntu ����

## ��װ Ubuntu �����

https://cn.ubuntu.com/download/alternative-downloads

### ��������

ϵͳ����
* ������`sudo reboot`
* �ػ���`sudo poweroff`
* ʹ�� root �������`sudo <����>`
* ��ȡ����·����`which <����>`

ʹ�� apt-get ���װ�����
* �������Դ��`sudo apt-get update`
* ���������Ѱ�װ�����`sudo apt-get upgrade`
* ��װ�����`sudo apt-get install <�����>`
* ж�������`sudo apt-get remove <�����>`
* ���������`apt-cache search <�����>`
* �鿴�Ѱ�װ�����`apt list --installed`
* �鿴����汾��`apt-cache show <�����>`

ʹ�� dpkg ���װ�����
* ��װ�����`sudo dpkg -i <�����>.deb`
* ж�������`sudo dpkg -r <�����>`

�ļ�������
* �г�Ŀ¼�������ļ���`ls`
* ����Ŀ¼��`cd <Ŀ¼��>`
* ����Ŀ¼��`mkdir <Ŀ¼��>`
* ɾ��Ŀ¼��`rmdir <Ŀ¼��>`
* �����ļ���`cp <Դ�ļ�> <Ŀ���ļ�>`
* �ƶ��ļ���`mv <Դ�ļ�> <Ŀ���ļ�>`
* ɾ���ļ���`rm <�ļ���>`
* �鿴�ļ����ݣ�`cat <�ļ���>`
* �鿴��ǰĿ¼��`pwd`
* �޸��ļ�Ȩ�ޣ�`chmod <Ȩ��> <�ļ���>`
* �ļ�����������`grep <�ַ���> <�ļ���>`
* �ļ������ң�`find <Ŀ¼��> -name <�ļ���>`

���̹���
* �г����н��̣�`ps -ef`
* ɱ�����̣�`kill <���̺�>`
* �鿴������Ϣ��`top` �� `htop`

bash �﷨��
* ���������`name=value`
* ����Ϊ����������`export name=value`
* ���������`alias name='command'`
* ��ӡ����ֵ��`echo $name`
* ʹ�ñ�����`$name` �� `${name}`
* �ܵ����`command1 | command2`

���������
* htop���鿴ϵͳ��Դռ�ã�`sudo apt-get install htop`
* vim���༭����`sudo apt-get install vim`
  * ��������ļ���`vim <�ļ���>`
  * ����༭ģʽ���� `i`
  * �˳��༭ģʽ���� `Esc` �ٰ� `:` �ٰ� `wq` ���沢�˳�
* vmtools������������ߣ�
  * `sudo apt-get install open-vm-tools`
  * `sudo apt-get install open-vm-tools-desktop`
* Android Studio��Android ���� IDE�� https://developer.android.com/studio/install
* vscode������༭����`sudo snap install code --classic`

## ��װ NDK

�� Android Studio ������ NDK�������ֶ����أ�https://developer.android.com/ndk/downloads

### ���û�������

1. �� `~/.bashrc` �ļ���`sudo vim ~/.bashrc`
2. ���ļ�ĩβ����������ݣ�
```
export ANDROID_SDK=/home/xichang/Android/Sdk/
export ANDROID_NDK=/home/xichang/Android/Sdk/ndk/28.0.12433566/
export PATH=$PATH:$ANDROID_SDK/platform-tools/:$ANDROID_NDK/toolchains/llvm/prebuilt/linux-x86_64/bin/
```
3. ���沢�˳���ִ�� `source ~/.bashrc` ����ʹ����������Ч��

### ���� usb ����

�鿴�豸��`lsusb`
```
Bus 002 Device 002: ID 18d1:4ee7 Google Inc. Nexus/Pixel Device (charging + debug)
```

1. ���� usb �����ļ���`sudo vim /etc/udev/rules.d/70-android.rules`
2. ���ļ�ĩβ����������ݣ�
  * `ATTRS{idVendor}=="18d1"` ��ʾƥ�䳧�� ID Ϊ 18d1 ���豸
  * `ATTRS{idProduct}=="4ee7"` ��ʾƥ���Ʒ ID Ϊ 4ee7 ���豸
  * `MODE="0666"` ��ʾ�����дȨ��
  * `GROUP="plugdev"` ��ʾ��ӵ� plugdev ��
  * `SYMLINK+="android_device"` ��ʾ�����������ӣ��������ʹ��
```
SUBSYSTEM=="usb", ATTRS{idVendor}=="18d1", ATTRS{idProduct}=="4ee7", MODE="0666", GROUP="plugdev", SYMLINK+="android_device", SYMLINK+="android_adb"
SUBSYSTEM=="usb", ATTRS{idVendor}=="18d1", ATTRS{idProduct}=="4ee0", MODE="0666", GROUP="plugdev", SYMLINK+="android_device", SYMLINK+="android_fastboot"
```
3. ���沢�˳���ִ�� `sudo udevadm control --reload-rules` ����ʹ������Ч��
4. ���� usb :
  * `sudo service udev restart`
  * `sudo udevadm trigger`
5. ���� adb ��
  * `adb kill-server`
  * `adb start-server`

## ����
`proxychains` �������������
* `sodo apt install proxychains4`
* �༭�����ļ� `/etc/.proxychains4.conf`
  * �޸� `socks4 127.0.0.1 1080` Ϊ `socks4 127.0.0.1 7890`
* ʹ�� `proxychains4 curl www.google.com`

electerm ʹ�� ssh ���� Ubuntu ����� https://github.com/electerm/electerm

LocalSend ���������乤�� https://localsend.org/zh-CN/download