---
title: Qt和FFmpeg实现播放器---环境搭建
date: 2018-05-31 19:26:00
tags: [Qt,FFmpeg]
---

---
简介：我是在Win10 64位系统下，使用的是Qt 5.6.1版本 和FFmpeg 3.4.2版本

之前根据网上教程搭建环境，在链接库这里一直失败，后来将FFmpeg从64位版本换成32位版本，编译链接就通过了，具体原因还不清楚

搭建流程如下：
> 1、先在https://ffmpeg.zeranoe.com/builds/下载win32位的dev和shared
> 2、分别解压，复制dev的lib和include地址加入qt的.pro文件中，类似：
>
> INCLUDEPATH += $$PWD\ffmpeg_dev\include
> LIBS += G:\document\zhd\qt_project\FFmpeg\ffmpeg_dev\lib\lib*.dll.a
>
> 3、在系统环境变量中加入shared文件夹中bin路径
> 4、创建一个纯C++项目，测试搭建是否成功，因为FFmpeg是C实现的，所以需要加上extern ”C“来调用头文件

测试代码如下：
```c++
#include <iostream>
using namespace std;
extern "C"
{
#include "libavcodec/avcodec.h"
#include "libavformat/avformat.h"
#include "libswscale/swscale.h"
#include "libavdevice/avdevice.h"
}

int main()
{
    cout<<"Hello FFmpeg"<<endl;
    av_register_all();
    unsigned version = avcodec_version();
    cout<<"version is:"<<version;
    return 0;
}
```

---