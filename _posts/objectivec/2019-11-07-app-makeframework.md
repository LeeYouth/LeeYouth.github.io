---
layout: post
title: iOS 制作.a静态库
categories: Objective-C
description: iOS 制作.a静态库
keywords: Objective-C, OC, iOS
---


#### iOS 制作.a静态库

**自己在制作.a静态库时的记录**
- 运行环境：Xcode 10.2.1
- 系统环境：macOS Catalina

------------
1. 新建项目选择 Cocoa Touch Static Library

2. 将自己需要制作的静态库的.h和.m文件拖入项目中。

3. 选择真机或者Generic iOS Device编辑不报错。

4. 在项目的Products找到生成的.a库即可使用

**注意：**
- Build Setting 中可以设置arm64，armv7，armv7s等。选择越多.a静态库文件越大
- 关闭Build Setting 中的Enable Bitcode 设置为NO
- PROJECT ---> Info ---- Deployment Target ---> 可以设置最低支持版本
