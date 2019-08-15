---
layout: post
title: pod私有库的提交以及使用
categories: CocoaPods
description: 使用私人pod库需要注意的几个点。
keywords: CocoaPods, podsepc
---

pod私有库库的提交使用一下代码，记得每次提交tag版本

``` objectivec
git add .
git commit -m"first commit"
git remote add origin https://git.coding.net/one_fools/OFPublicTool.git
git push origin master
git tag -m"tag 0.1.0" 0.1.0
git push --tags
```

使用私人pod库的需要在Podflie中添加这句话，指明你的版本库地址。
``` objectivec
source ‘https://git.oschina.net/baiyingqiu/MyRepo.git’
```

注意是版本库的地址，而不是代码库的地址，很多教程都把我搞晕了~
若有还使用了公有的pod库，需要把公有库地址也带上
``` objectivec
source 'https://github.com/CocoaPods/Specs.git'
```


最后的Podflie文件变成这个样子
``` objectivec
source ‘https://github.com/CocoaPods/Specs.git’
source ‘https://git.oschina.net/baiyingqiu/MyRepo.git’
```

``` objectivec

platform :ios, '8.0'

target ‘MyPodTest’ do
use_frameworks!

pod “BYPhoneNumTF” #公有库
pod ‘MyAdditions’ #我们的私有库
pod ‘BYAdditions’ #这是我又添加到版本库中的另一个代码库

end
```

测试：
``` objectivec
pod install
```



