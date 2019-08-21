---
layout: post
title: _posts文件中.md文件头部信息
categories: GitHub
description: _posts文件中.md文件头部信息
keywords: GitHub, GitHub Blog
---


GitHub创建自己的个人博客。在_posts中添加文章，我目前添加的为markdown文件(.md结尾文件)，在.md结尾的文件中要包含头部信息：

```

---
layout: post
title: Flutter首次安装环境变量配置
categories: Flutter
description: Flutter首次安装环境变量配置
keywords: Flutter, Flutter Install
---

```

- _posts 文件夹中是我已发布的博客文章。
- _drafts 文件夹中是我尚未发布的博客文章。
- _wiki 文件夹中是我已发布的 wiki 页面。
- images 文件夹中是我的文章和页面里使用的图片。


**使用gittalk出现以下问题**

1. [创建OAuth application](https://github.com/settings/applications/new)中Homepage URL要与Authorization callback URL，除非你解析了新的域名地址吗，否则写一致。
2. 创建一个公共仓库。名字与第一步创建的Application name一致。
3. 在_config.yml文件修改你的gittalk注册的相关信息
-  enable: true    #用来做启用判断可以不用
-  clientID: 'clientID'    #OAuth Application
-  clientSecret: 'clientID'   #OAuth Application
-  repo: blog_comment仓库   #刚创建blog_comment用于gitalk Issue的仓库名称
-  owner: username    #github用户名

