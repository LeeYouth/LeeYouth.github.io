---
layout: post
title: iOS 集成FFmpeg
categories: FFmpeg
description: iOS 集成FFmpeg
keywords: FFmpeg, 播放器
---


iOS 集成FFmpeg

## iOS编码与解码介绍

> iOS 平台分两种：软编解码、硬编解码

| 类型      |    工具 |     硬件支持  |    后台 |    思路  |   
| :-------- | --------:| :--: |
| 硬编解码  | VideoToolBox |  非CPU或者专用处理器   | 支持编解码（H.264）   | VideoToolBox   |
|  ----  | AVAssetWriter |  非CPU或者专用处理器   | 支持编码     | 需要将视频写入本地文件，然后通过实时监听文件内容的改变，读取文件并处理封包       |

| 软编解码  | FFmpeg |  CPU   | 支持（rtmp,flv,ts）   | ----   |

## Mac上安装FFmpeg环境

- **方式一**：直接在官网上下载[FFmpeg download](https://ffmpeg.org/download.html#build-mac)
- **方式二**：手动编译: 下载源码, 可以在更改一些flag或源码后再编译脚本,较为灵活.

### 手动安装步骤
**通过Homebrew 安装 FFmpeg**
- 终端执行：执行 brew install ffmpeg --with-libvpx --with-libvorbis --with-ffplay
- 在终端中执行一下命令，等待安装完成即可：

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

```

- 安装好Homebrew，然后终端执行 "brew install ffmpeg"，等待完成即可。
执行结束，在终端中输入ffmpeg，验证是否安装成功。

**编译 iOS 下 FFmpeg**
- **安装 [gas-preprocessor](https://github.com/bigsen/gas-preprocessor.git)**
安装步骤(依次执行下面命令)

```
sudo git clone https://github.com/bigsen/gas-preprocessor.git  /usr/local/bin/gas
sudo cp /usr/local/bin/gas/gas-preprocessor.pl /usr/local/bin/gas-preprocessor.pl
sudo chmod 777 /usr/local/bin/gas-preprocessor.pl
sudo rm -rf /usr/local/bin/gas/

```

- **安装 yams**
> yasm是汇编编译器，因为ffmpeg中为了提高效率用到了汇编指令，所以编译时需要安装
安装步骤(依次执行下面命令)

```
curl http://www.tortall.net/projects/yasm/releases/yasm-1.2.0.tar.gz -o yasm-1.2.0.gz
tar -zxvf yasm-1.2.0.gz
cd yasm-1.2.0
./configure && make -j 4 && sudo make install

```

- **下载 [FFmpeg-iOS-build-script](https://github.com/kewlbear/FFmpeg-iOS-build-script.git)自动编译的脚本**

```
cd ~/downloads
git clone https://github.com/kewlbear/FFmpeg-iOS-build-script.git

```

1. 打开build-ffmpeg.sh并配置相关信息（我只修改了部分）

```
#!/bin/sh

# 目标FFmpeg版本
FF_VERSION="4.0.3"
# 使用最新版本
#FF_VERSION="snapshot-git"
if [[ $FFMPEG_VERSION != "" ]]; then
FF_VERSION=$FFMPEG_VERSION
fi
# 待编译源码，命名方式 ffmpeg-版本号
SOURCE="ffmpeg-$FF_VERSION"
# 输出FAT文件的路径
# 按成各个平台分别编译成库，最终合成一个FAT库
FAT="FFmpeg-iOS"

# 各平台的精简代码
# 存储编译过程中的一些文件，如在configure过程出现的问题将被记录于此
# configure日志路径: scratch/$ARCH/ffbuild/config.log
SCRATCH="scratch"

# 各平台编译输出，必须是绝对路径
THIN=`pwd`/"thin"

# 如需单独链接外部库赖，引入方式
#XXXLIB=/absolute/path/to/the/fat-path

# FFmpeg Configure 参数
CONFIGURE_FLAGS="--enable-cross-compile --disable-debug --disable-programs \
--disable-doc --enable-pic"

# 需要支持的目标平台（未设置参数时的默认开启范围）
ARCHS="arm64 armv7 x86_64 i386"

```

2. 使用命令运行脚本

```
./build-ffmpeg.sh
```

3. 漫长等待之后生成三个文件夹,FFmpeg-iOS文件夹内的include与lib文件正是我们所需要
![](/images/posts/ffmpeg/ffmpegbuildscript_output.png) 


## iOS项目上集成FFmpeg

1. 项目中添加依赖库（Link Binary With Libraries中点击加号添加）
- libz.tbd
- libbz2.tbd
- libiconv.tbd
- CoreMedia.framework
- VideoToolbox.framework
- AVFoundation.framework

2. 拖拽进项目中刚才生成的FFmpeg-iOS文件到项目中。
3. 设置Header Search Paths 路径，指向 项目中include目录 。

```
$(SRCROOT)/FFmpeg_iOS/FFmpeg/include

```

4. 在项目中导入头文件,编译通过

```
#ifdef __cplusplus
extern "C" {
#endif

#include "libavutil/opt.h"
#include "libavcodec/avcodec.h"
#include "libavformat/avformat.h"
#include "libswscale/swscale.h"

#ifdef __cplusplus
};
#endif

```

5. 接下来你就可以使用FFmpeg进行开发了~


----

**推荐文章**

[视频的基本参数及H264编解码相关概念](https://maxwellqi.github.io/ios-h264-summ/)

[VideoToolBox之视频编码](https://www.jianshu.com/p/06162a4731fb)

[视频软解码和硬解码的区别](https://blog.csdn.net/qq_15807167/article/details/52262559)

[iOS手动编译并搭建FFmpeg](https://juejin.im/post/5ceff73df265da1bb13f16f4)
