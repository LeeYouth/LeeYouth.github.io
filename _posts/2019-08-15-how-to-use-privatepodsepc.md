---
layout: post
title: 私人pod库的使用
categories: CocoaPods
description: 使用私人pod库需要注意的几个点。
keywords: CocoaPods, podsepc
---

###私人pod库的使用

@(Flutter学习)

使用私人pod库的需要在Podflie中添加这句话，指明你的版本库地址。
```
source ‘https://git.oschina.net/baiyingqiu/MyRepo.git’
```

注意是版本库的地址，而不是代码库的地址，很多教程都把我搞晕了~
若有还使用了公有的pod库，需要把公有库地址也带上
```
source 'https://github.com/CocoaPods/Specs.git'
```


最后的Podflie文件变成这个样子
```
source ‘https://github.com/CocoaPods/Specs.git’
source ‘https://git.oschina.net/baiyingqiu/MyRepo.git’
```

```

platform :ios, '8.0'

target ‘MyPodTest’ do
use_frameworks!

pod “BYPhoneNumTF” #公有库
pod ‘MyAdditions’ #我们的私有库
pod ‘BYAdditions’ #这是我又添加到版本库中的另一个代码库

end
```

测试：
```
pod install
```
