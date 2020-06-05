---
layout: post
title: pod私有库的创建、提交及使用
categories: CocoaPods
description: pod私有库的创建、提交及使用。
keywords: CocoaPods, podsepc
---

**创建私有库podspec及索引库specs**
- **创建索引库**
- **创建私有仓库**
- **验证私有仓库**
- **提交私有仓库**
- **推送私有库到索引库**
- **项目使用私有库**

#### 创建索引库mySpecs
```
pod repo add mySpecs https://github.com/leeyouth/mySpecs.git 
```
#### 创建自己的私有仓库（任意位置）
1. 执行命令
```
pod lib create LYNetworkKit
```
2. 选择条件
```
What platform do you want to use?? [ iOS / macOS ]
 > ios
What language do you want to use?? [ Swift / ObjC ]
 > objc
Would you like to include a demo application with your library? [ Yes / No ]
 > yes
Which testing frameworks will you use? [ Specta / Kiwi / None ]
 > none
Would you like to do view based testing? [ Yes / No ]
 > no
What is your class prefix?
 > ly
```
3. 在Classes目录下添加你的库文件，删除replaceme.m。
4. cd到Example目录下，如果依赖的有私有库，添加相应的私有库或索引库git，执行pod install验证是否有错。 (执行 pod update可选参数：--verbose 可以查看详细的过程 | --no-repo-update 不升级本地的repo会快一些)
5. 编辑podspec文件
```
Pod::Spec.new do |s|
  s.name             = 'LYCommonKit'
  s.version          = '0.1.9'
  s.summary          = '通用组件，包含Toast，DBManager等.'

  s.description      = <<-DESC
TODO: Add long description of the pod here.
                       DESC

  s.homepage         = 'https://github.com/LeeYouth/LYCommonKit'
  # s.screenshots     = 'www.example.com/screenshots_1', 'www.example.com/screenshots_2'
  s.license          = { :type => 'MIT', :file => 'LICENSE' }
  s.author           = { 'LeeYouth' => 'yuanliyong@jd.com' }
  s.source           = { :git => 'https://github.com/LeeYouth/LYCommonKit.git', :tag => s.version.to_s }
  # s.social_media_url = 'https://twitter.com/<TWITTER_USERNAME>'

  s.platform     = :ios, "11.0"
  s.ios.deployment_target = '8.0'
  # 文件及子文件
  s.source_files = 'LYCommonKit/Classes/**/*'

  # 暴露指定的头文件
  # s.public_header_files = 'Pod/Classes/**/*.h'
  # 依赖的系统库
  s.frameworks =  'Foundation', 'UIKit', 'CoreData'
  # 依赖的三方库
  s.dependency 'AFNetworking'
  s.dependency 'MBProgressHUD'
  s.dependency 'JGProgressHUD'
  s.dependency 'YYKit'
  s.dependency 'FMDB'
end
```

#### 验证私有库是否有误:

- 私有库不依赖私有库

```
pod lib lint --allow-warnings
```
- 私有库依赖的有私有库
```
pod lib lint LYCommonUIKit.podspec --sources='https://github.com/LeeYouth/LY_MDSpecs.git,https://github.com/CocoaPods/Specs.git'  --allow-warnings
```
#### 验证通过之后，将文件push到git仓库中：
```
git init
git add .
git commit -m 'firstCommit'
// 首次关联本地仓库和远程仓库。
git remote add origin https://github.com/LeeYouth/LYCommonKit.git
// 第一次push如果报错的话可以加上-f
// git push -f origin master
git push origin master
//打标签（要与podsepc文件内的版本号相同）
git tag '0.1.0'
git push --tags
```
#### 提交完成之后将podsepc添加到私有索引库中：
```
# --verbose 可选
pod repo push LY_MDSpecs LYCommonUIKit.podspec --allow-warnings
```

#### 项目中使用私有库，podfile中添加如下：
```
source ‘https://github.com/CocoaPods/Specs.git’
source ‘https://github.com/leeyouth/mySpecs.git ’
```

---------
附：git一些常用命令
```
git init//初始化
git status//查看状态
git add .//添加文件到缓冲区
git commit -m "描述"//从缓冲区提交代码到仓库
git tag -a '0.0.1'  -m '描述'//添加tag
git tag //查看tag
git tag -d '0.0.1'//删除tag
git remote add origin https://github.com/xxx.git//关联本地仓库和远程仓库。
git push -f origin master//将本地库的代码推到远程库
git push --tags//将本地创建的tag推到远程库
git push origin :0.0.1//删除tag
```



