---
title: 交叉编译Qt移植到OrangePI
date: 2018-06-02 21:25:44
tags: Qt
---

---

简介：交叉编译Qt5.5移植到OrangePI Prime上

（1）启动和配置Orange PI
1.先用SD Card Formatter将Tf卡格式化
2.使用Win32DiskImager将Debian_Desktop_Prime镜像烧进Tf卡中
3.将Tf卡插进OrangePI卡槽，用HDMI线接显示器，再接通电源，等待系统启动
4.进入登录界面，输入账户密码，默认都是orangepi
5.设置好网络，可连接wifi或者插网线
6.打开终端，输入sudo nano /etc/apt/sources.list,将原来的内容加#注释，并加入以下内容（中科大镜像源）：
deb http://mirrors.ustc.edu.cn/debian stable main contrib non-free
deb http://mirrors.ustc.edu.cn/debian stable-updates main contrib non-free
7.在终端输入 sudo apt-get install debian-keyring debian-archive-keyring
  再输入sudo apt-key update
8.输入sudo apt-get install apt-transport-https（可解决有些源更新不了的问题）
9.输入sudo apt-get update
10.输入./getlib.sh，等待库的安装完成（安装期间会出现询问选项，需要输入y继续）
11.输入./gettslib.sh
至此，Orange PI的基本配置已经完成

（2）PC端的配置及交叉编译qt流程
1.先安装debian系统，可以用单独一台主机或者VMware来实现安装，镜像放在img文件夹中
2.登入debian系统，将之前配置下载好库的Orange PI的Tf卡拔出，插进读卡器后连通PC端
3.查看Tf卡是否挂载成功，查询Tf的挂载路径，我的路径是/media/haxan/rootfs，然后打开pkg_config.sh进行路径修改，
注意：如果地址有误，会影响卡上系统库的链接。
4.运行pkg_config.sh（如果后续的配置出现库链接失败，或者头文件找不到的问题，可尝试直接在终端输入pkg_config.sh的内容）
5.解压qt-everywhere-opensource-src-5.5.0.tar.gz,复制arm-linux文件夹到/qt-everywhere-opensource-src-5.5.0/qtbase/mkspace/
6.复制Tf卡上的/lib/aarch64-linux-gnu/到pc上的/lib上
7.运行comqmake.sh脚本
8.cd 进 /qt-everywhere-opensource-src-5.5.0/qtbase,运行make -j8(八线程编译，根据具体电脑配置来选择)
9.编译成功后，输入sudo make install
安装成功后，会有/usr/local/qt5pi1以及/media/haxan/rootfs/usr/local/qt5pi1, 至此，qt5.5安装完成
9.修改pc机的/usr/lib/x86_64-linux-gun/qt-default/qtchooser/default.conf,修改第一行为/usr/local/qt5pi1/bin
注意事项：如果之前编译失败，之后重新修改配置，最好是把源码文件夹删掉，再重新解压源码包

（3）板子的程序运行
1.将Tf卡放回Orange PI上，执行export LD_LIBRARY_PATH=/lib/tslib/lib:$LD_LIBRARY_PATH
2.之后要在pc端上交叉编译时，需要把Tf卡插回PC，不然找不到qt库

（4）tslib的问题
export QT_QPA_EGLFS_DISABLE_INPUT=1						# 屏蔽eglfs内置输入
export TSLIB_ROOT=/lib/tslib							# 指定tslib主目录位置
export TSLIB_TSDEVICE=/dev/input/touchscreen0			# 指定触摸屏设备
export TSLIB_CONFILE=$TSLIB_ROOT/etc/ts.conf			# 指定TSLIB配置文件的位置
export TSLIB_PLUGINDIR=$TSLIB_ROOT/lib/ts				# 指定触摸屏插件所在路径
export TSLIB_FBDEVICE=/dev/fb0							# 指定帧缓冲设备
export TSLIB_CONSOLEDEVICE=none							# 设定控制台设备为none,否则默认为/dev/tty，,这样会出现”open consol device:No such file or directory KD…..”的错误
export LD_LIBRARY_PATH=/lib/tslib/lib:$LD_LIBRARY_PATH  # 指定TSLIB的库文件路径
export QWS_MOUSE_PROTO=tslib:/dev/input/touchscreen0	# 指定触摸屏设备




---