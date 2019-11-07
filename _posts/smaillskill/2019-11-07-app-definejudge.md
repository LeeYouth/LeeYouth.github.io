---
layout: post
title: iOS中使用宏定义判断
categories: Objective-C
description: iOS中使用宏定义判断
keywords: Objective-C, 宏定义,iOS
---


#### iOS中使用宏定义判断

------------

使用格式
```
#ifdef DEBUG

static NSString *const kFPBaseUrl =@"http://（1）";

#else

static NSString *const kFPBaseUrl =@"http://（2）";

#endif

```

1. 宏定义判断系统版本，避免高版本API到低版本使用报错（如果是使用高版本sdk编译的（如iOS10），将最终的应用安装至低版本的设备上（iOS9的系统），此时如果不加入运行时判断就会出现问题（可能是crash））
``` 
#if __IPHONE_OS_VERSION_MAX_ALLOWED >= 70000  
if ([[[UIDevice currentDevice] systemVersion] floatValue] >= 7.0) {    
        self.window.tintColor = [UIColor redColor];    
    }    
#endif
 ```
