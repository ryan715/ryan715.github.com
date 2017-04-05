---
layout: post
title: "capistrano puma deploy, rails console"
date: 2017-04-05 21:15:05 +0800
comments: true
categories: [rails, capistrano]
---

### 问题一

最近，有个项目需要用到自动部署，就把capistrano祭出来。噼里啪啦把相关的gem安装上，配置文件拉起来，一切都是按步就班，就不一一贴出来。最后运行时候提示

` Task :"puma:restart" not found.`

查看了logs也没有发现有什么猫腻，到底问题出在哪里？怀疑是puma的gem上面。rails5默认自带puma gem，是不是和capistrano冲突了？

### 解决

这个提示不好搜索，也没有任何的直接的关联答案。换个思路，找几篇最近的capistrano部署的tutorials，提取里面的deploy配置文件进行对比，写法基本一致。后来排查到Capfile:

`install_plugin Capistrano::Puma`

缺少了这个，加上后就好了。

### 总结

开发过程中，或多或少会用到以前的一些gem，gem升级以后使用的用法会有所改变，把相关的思路再重新捋一遍，如果没有太大问题估计就是一些配置有所改变或者遗漏，一一对应即可解决问题。

### 问题二
接前面，以前用capistrano部署完了就ok，也没有想到要在项目下面运行

`rails c`

Rails 5 环境下是运行不起来的，那么如何才能跑起来呢？
### 解决

在 deploy 配置文件里面，把 bin 从 set :linked_dirs 移除，再添加 

`set :bundle_binstubs, nil `

就可以解决。


