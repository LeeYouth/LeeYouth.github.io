---
layout: post
title: FFmpeg基础介绍
categories: FFmpeg
description: FFmpeg基础介绍
keywords: FFmpeg, 播放器
---

FFmpeg基础介绍

## FFmpeg的定义

FFmpeg既是一款音视频编解码工具，同时也是一组音视频编解码开发套件，作为编解码开发套件，它为开发者提供了丰富的音视频的调用接口。

FFMpeg提供了多种媒体格式的封装和解封装，包括多种音视频编码、多种协议的流媒体、多种色彩格式转换、多种采样率转换、多种码率转换等；FFmpeg框架提供了多种丰富的插件模块，包含封装与解封装的插件、编码与解码的插件等。

FFmpeg中的“FF”指的是“Fast Forward”，FFmpeg中的“mpeg”则是“Moving Picture Experts Group（动态图像专家组）”。

## FFmpeg的基本组成

FFmpeg框架的基本组成包括AVFormat、AVFilter、AVDevice、AVUtil等模块库。
下面针对这些模块做一个大概的介绍。

- **FFmpeg的封装模块AVFormat**

AVFormat中实现了目前多媒体领域中的绝大多数媒体封装格式，包括封装和解封装，如MP4、FLV、KV、TS等文件封装格式，RTMP、RTSP、MMS、HLS等网络协议封装格式。
FFmpeg是否支持某种媒体封装格式，取决于编译时是否包含了该格式的封装库。根据实际需求，可进行媒体封装格式的拓展，增加自己定制的封装格式，即在AVFormat中增加自己的封装处理模块。

- **FFmpeg的编解码板块AVCodec**

AVCodec中实现了目前多媒体领域绝大多数常用的编解码格式，既支持编码，也支持解码。AVCodec除了支持MPEG4、AAC、MJPEG等自带的媒体编解码格式之外，还支持第三方的编解码器，如H.264（AVC）编码，需要使用x264编码器；H.265（HEVC）编码，需要使用X265编码器；MP3（mp3lame）编码，需要使用libmp3lame编码器。如果希望增加自己的编码格式，或者硬件编解码，则需要在AVCodec中增加相应的编解码模块。

- **FFmpeg的滤镜模块AVFilter**

AVFilter库提供了一个通用的音频、视频、字幕等滤镜处理框架。在AVFilter中，滤镜框架可以有多个输入或多个输出。

- **FFmpeg的视频图像转换计算模块swscale**

swscale模块提供了高级别的图像转换API，例如它允许进行图像缩放和像素格式转换，常见于将图像从1080p转换成720p或者480p等的缩放，或者将图像数据从YUV420P转换成YUYV，或者YUV转RGB等图像格式转换。

- **FFmpeg的音频转换计算模块swresample**

swresample模块提供了高级别的音频重采样API。例如它允许操作音频采样、音频通道布局转换与布局调整。

## FFmpeg的编解码工具ffmpeg

ffmpeg是FFmpeg源代码编译后生成一个可执行程序，其可以作为命令行工具使用。
首先列举一个简单的例子：

```
./ffmpeg -i input.mp4 output.avi

```
这是一条简单的ffmpeg命令，可以看到，ffmpeg通过-i参数将input.mp4作为输入源输入，然后进行转码与转封装操作，输出到output.avi中，这条命令主要做了如下工作：

1. 获得输入源input.mp4
2. 转码
3. 输出文件output.avi
看似简单的两步主要的工作，其实远远不止是从后缀名为MP4的文件输出成后缀名为AVI的文件，因为在ffmpeg中，MP4与AVI是两种文件封装格式，并不是后缀名就可以决定的，例如上面的命令行同样可以写成这样：

```
./ffmpeg -i input.mp4 -f avi output.dat

```

这条ffmpeg命令相当于前面的那条命令做了一些改变，加了一个“-f”进行约束，“-f"参数的工作非常重要，它制定了输出文件的容器格式，所以可以看到输出的文件为output.dat，文件后缀名为.dat，但是其主要工作依然与之前的指令相同。

ffmpeg的主要工作流程相对比较简单，具体如下：

1. 解封装
2. 解码
3. 编码
4. 封装

其中需要经过6个步骤，具体如下：
1. 读取输入源
2. 进行音视频的解封装
3. 解码每一帧音视频数据
4. 编码每一帧音视频数据
5. 进行音视频的重新封装
6. 输出到目标

具体的使用方法可以参考：
详细的使用说明（英文）：http://ffmpeg.org/ffmpeg.html

## FFmpeg的播放器ffplay

FFmpeg不但可以提供转码、转封装等功能，同时还提供了播放器相关功能，使用FFmpeg的avformat与avcodec，可以播放各种媒体文件或者流。

ffplay是FFmpeg源代码编译后生成的另一个可执行程序，与ffmpeg在FFmpeg项目中充当的角色基本相同，可以作为测试工具进行使用，ffplay提供了音视频显示和播放相关的图像信息、音频的波形信息等。

详细的使用说明（英文）：http://ffmpeg.org/ffplay.html

## FFmpeg的多媒体分析器ffprobe

ffprode也是FFmpeg源码编译后生成的一个可执行程序。ffprode是一个非常强大的多媒体分析工具，可以从媒体文件或者媒体流中获得你想要了解的媒体信息，比如音频的参数、视频的参数、媒体容器的参数信息等。

例如它可以帮助分析某个媒体容器中的音频是什么编码格式、视频是什么编码格式，同时还可以得到媒体文件中媒体的总时长、复合码率等信息。使用ffprode可以分析媒体文件中每个包的长度、包的类型、帧的信息等。

详细的使用说明（英文）：http://ffmpeg.org/ffprobe.html

















