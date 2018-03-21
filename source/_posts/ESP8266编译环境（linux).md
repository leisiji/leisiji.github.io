---
title: ESP8266编译环境（linux)
date: 2018-03-21 11:11:14
tags:
---

首先说明，如果对于 linux 不是太熟悉，不建议在 linux 上进行开发，用 windows 虚拟机就可以进行了。
<!--more-->
当然也可以选择在 WSL 上搭建开发的环境。
linux 的好处是编译的速度大大加快。

# 方法一
如果第一个出现问题，可以进行逐个安装
```bash
sudo apt-get install git autoconf build-essential gperf bison flex texinfo libtool libncurses5-dev wget gawk libc6-dev-amd64 python-serial libexpat-dev
mkdir /opt/Espressif
sudo chown $username /opt/Espressif/
```
注意如果在 WSL 上，libc6-dev-amd64 要换成 libc6-dev。
接着就是安装 xtensa：
```bash
cd /opt/Espressif
git clone -b lx106 git://github.com/jcmvbkbc/crosstool-NG.git 
cd crosstool-NG
./bootstrap && ./configure --prefix=`pwd` && make && make install
./ct-ng xtensa-lx106-elf
./ct-ng build
PATH=$PWD/builds/xtensa-lx106-elf/bin:$PATH
```
由于 `./ct-ng build` 这一步需要从网上下载，所以需要比较长的时间。
Setting up the Espressif SDK：
```bash
cd /opt/Espressif
wget -O esp_iot_sdk_v0.9.3_14_11_21.zip https://github.com/esp8266/esp8266-wiki/raw/master/sdk/esp_iot_sdk_v0.9.3_14_11_21.zip
wget -O esp_iot_sdk_v0.9.3_14_11_21_patch1.zip https://github.com/esp8266/esp8266-wiki/raw/master/sdk/esp_iot_sdk_v0.9.3_14_11_21_patch1.zip
unzip esp_iot_sdk_v0.9.3_14_11_21.zip
unzip esp_iot_sdk_v0.9.3_14_11_21_patch1.zip
mv esp_iot_sdk_v0.9.3 ESP8266_SDK
mv License ESP8266_SDK/
```
patching：
```bash
cd /opt/Espressif/ESP8266_SDK
sed -i -e 's/xt-ar/xtensa-lx106-elf-ar/' -e 's/xt-xcc/xtensa-lx106-elf-gcc/' -e 's/xt-objcopy/xtensa-lx106-elf-objcopy/' Makefile
mv examples/IoT_Demo .
```
Installing the ESP image tool：
```bash
cd /opt/Espressif
wget -O esptool_0.0.2-1_i386.deb https://github.com/esp8266/esp8266-wiki/raw/master/deb/esptool_0.0.2-1_i386.deb
dpkg -i esptool_0.0.2-1_i386.deb
```
Installing the ESP upload tool：
```bash
cd /opt/Espressif
git clone https://github.com/themadinventor/esptool esptool-py
ln -s $PWD/esptool-py/esptool.py crosstool-NG/builds/xtensa-lx106-elf/bin/
```

> 如果不放心长时间下载那一步，可以用 nethogs 工具查看流量。上面都有一些小错误，按照错误提示上网找，或者按照缺少哪个依赖，就去安装吧。


# 方法二
推荐使用这一种。
这是一种比较简单的方法：
```bash
git clone --recursive https://github.com/pfalcon/esp-open-sdk
cd esp-open-sdk
make STANDALONE=y |& tee make0.log
vi .bashrc
// 在最后一行添加
export PATH="$HOME/esp-open-sdk/xtensa-lx106-elf/bin/:$PATH"    // 注意上一步会有提示，复制提示的命令
```

> 里面可能会有一个 `LD_LIBRARY_PATH` 的 bug，输入 $ unset LD_LIBRARY_PATH` 就可以重新输入安装的命令。

最后重新打开一个命令行验证：
```bash
xtensa-lx106-elf-gcc -v
// 如果不想输入太长的名字，也可以
alias xgcc="xtensa-lx106-elf-gcc"
```
make 那一步需要下载，也是需要比较长的时间

# 编译
从网上下载一个例子，接着进入 project 的目录，也就是包含 user 的那个文件夹，修改 `gen_misc.sh`：
```bash
export SDK_PATH=~/rtos      // 主要是修改这里 SDK 目录，一般可以自己下载好，以后就一直用这个路径
export BIN_PATH=./bin       // 生成二进制文件的目录，一般不需要修改
boot = new
app=1
spi_speed = 40
spi_mode = DIO
spi_size_map=5              // 这几个参数是可以通过命令行设置，也可以在这里直接修改，这样就不再需要命令行
cd libesphttpd              // 这个是 project 下的第三方库目录，如果不设置，下面的 cd .. 就要删除
make clean
cd ..
make clean
make BOOT=$boot APP=$app SPI_SPEED=$spi_speed SPI_MODE=$spi_mode SPI_SIZE_MAP=$spi_size_map
```

## 多版本 python

由于编译环境只能在 Python2.7 进行，因此使用 Python3 编译会报错（大多数是 print 的报错）。
这时可以使用 anaconda，具体的安装自己搜索。
这里只说明步骤：
```
conda create --name py27 python=2.7 // 创建新环境 py27
source activate py27                // 切换环境，命令行的前缀变成 py27
```
这时编译就不会出错了。


# 烧录
首先查看端口的占用：
```bash
dmesg | grep ttyS*
```
一般 ttyS0 对应 com1，ttyS1 对应 com2，当然也不一定是必然的。一般在命令行使用 tty。
使用 esptool：
```bash
$ pip install esptool
```
烧录命令：
```bash
esptool.py --port /dev/ttyUSB0 write_flash 0x1000 user1.2048.new.5.bin
```
多个 Flash 地址和文件名可以在同一个命令行中给出：
```bash
esptool.py --port /dev/ttyUSB0 write_flash 0x00000 my_app.elf-0x00000.bin 0x40000 my_app.elf-0x40000.bin
```
串口 Permission denied 情况：
```bash
cat /dev/ttyS*                      // 这时会提示权限不足
sudo chmod 777 /dev/ttyUSB0         // 修改权限为可读可写可执行，重启后失效
// 以下是永久的方法
ls -l /dev/ttyS*
sudo usermod -aG dialout leisiji   // 换成想用USB的用户名即可，把此用户名加入dialout用户组
// 最后注意一定要 log out 才可以使用
```

串口工具可以使用 minicom 或者 cutecom。推荐 cutecom。