---
layout: post
title: Flutter首次安装环境变量配置
categories: Flutter
description: Flutter首次安装环境变量配置
keywords: Flutter, Flutter Install
---

Flutter首次安装环境变量配置。将Flutter代码中bin文件目录添加到系统环境变量中，这里和java环境配置大同小异，mac端具体操作如下：

```
第一步：
cd ~           // 在终端进入用户目录，一般打开终端默认就是

第二步：
open .bash_profile    // 打开配置文件
// 另 如果 .bash_profile文件不存在需先创建再打开，具体如下：
// touch .bash_profile
// open .bash_profile

第三步：
export PATH=${PATH}:/Users/xxx/flutter/bin:$PATH
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
// 复制三行到bash_profile文件中 保存关闭即可
// “/Users/xxx/flutter/bin” 为Flutter下载到本地的目录地址，其他两句为方便在天朝正常使用flutter而添加，无需改动

第四步：
source .bash_profile      // 执行文件 使命令生效
```
