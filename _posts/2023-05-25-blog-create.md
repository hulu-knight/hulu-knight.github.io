---
layout: post
title: 关于使用 Jekyll + Github 搭建个人博客
tags: kotlin
author: hulu-knight
date: 2023-05-23 20:52 +0800
---



​	有一天晚上我在敲代码的时候，遇到了一个十分常见的 bug ，但是我却怎么也想不起来应该如何处理，我清楚的记得在几个月之前我曾经为了这个问题而大动干戈，最后改掉 bug 的那份快乐让我记忆犹新。但是现在面对同样的一个问题（至少我印象当中是同一个问题）的时候，我却还要付出好多的经历去网上查找解决办法（当然是因为我还是太笨了，脑子记不住）。所以我认为我有必要对一些东西做一些记录，这个时候我想到了搭建一个博客。在努力搜寻之下，我最终选择了使用** Jekyll + Github** 来进行个人博客的搭建，本文章也是基于此帮助新人从 0 搭建博，同时也记录一下自己的踩坑过程。

​	当然你也可以通过其他静态网站：Hugo、Hexo等来进行搭建，直接从网上 down 一个模版下来就可以使用。或者你有前端基础，可以直接手写前端代码，比如用原生css + html + javascript等来进行搭建。各位可以根据自己喜欢的方式选择搭建并自行去网上查询，本文章只进行 Jekyll 搭建博客的讲解。

# 战前准备  

​	作为一名程序员，我们都应该知道，如果在写一个程序之前不做任何准备，直接拿着刺刀就往前冲，那么你的结局往往不会好的。哪有新兵不训练就上战场的，那么我们的新兵训练就是了解一些搭建知识和环境需求，我们要使用 Jekyll 和 Github 搭建，那么自然我们需要知道的有：

- 什么是 Jekyll 
- 什么是 Github Page
- 什么是 git

​	当然了，如果各位是久经沙场的老兵，那么这些内容完全可以跳过，直接开始我们的博客搭建。

> 以上只是一些常见概念，不看完全不影响学习本章内容，可以直接跳到 【战斗指南】 去学习

## 什么是 Jekyll  

## 什么是 Github Page  

## 什么是 git 

# 战斗指南

​	了解了一些基本的知识之后，我们这些新兵就应该可以很容易的依靠这些知识武装自己了，但是当你信心满满的扛枪战斗的时候，你就发现你根本打不过那些经验丰富的老兵的。当然了打仗也讲究一个方式方法，那么接下来我就教你如何一步一步的取下对方的首级（听起来是不是有些残忍），那么采取一些手段是必不可少的，比如

- 创建一个Github Page

- 安装好你的 git

- 寻找一个你喜欢的 Jekyll 主题

- 配置好你的 rvm 环境

- 配置好 ruby 环境

- 将博客，真正变成你自己的！

- 试运行你的博客

- 将博客推送到 Github 中

  

## 创建一个github账号

​	首先我们需要拥有Github 账号，当然如果你已经有了一个 账号了，直接用就完事了。

**创建 Github 账号**

​	登录网址，选择注册，填写基本的注册信息

**创建一个只用于记录博客的 Github 仓库，即 Github Page**

​	在主页当中点击右上角的加号新建仓库，如下图所示

![image-20230525232858267](/Users/hulu_knight/Library/Application Support/typora-user-images/image-20230525232858267.png)

​	点击之后出现如下图所示![image-20230525233629537](/Users/hulu_knight/Library/Application Support/typora-user-images/image-20230525233629537.png)

- Owner（拥有者）选择你账号的昵称，你可以单击右上角点击头像来查看你的昵称

- Repository name （仓库名称）：填写username.github.io （注意将 username 替换成你自己的昵称，当以这个格式命名仓库的时候，Github 自动认为该仓库是 Github Page）

- 权限选择 Public （公开）

- 其他不要勾选，点击 Create respository 仓库创建成功

​	创建成功如图所示![截屏2023-05-25 23.40.21](/Users/hulu_knight/Desktop/截屏2023-05-25 23.40.21.png)

​	请保留此界面，我们在 Github 上的操作就已经暂时结束了，你可以放心的把界面最小化，然后我们应该**部署本地仓库**了！

​	当然，在部署本地仓库之前，你需要先明确你的电脑是否安装了 git

## 安装好你的 git

​	关于 git 的安装真的很简单，简单三步走1. 登录git官网 2.下载 git 3. 运行 git 安装程序

## 寻找一个你喜欢的 Jekyll 主题

​	来到一个叫 Jekyll themes 的网站，上面有许许多多大佬们已经制作好的 Jekyll 静态界面，你可以从中寻找一个你喜欢的页面来 down 到自己的本地仓库，作为你创建个人博客的第一步”选取一个博客模版“，界面如图![image-20230525235234131](/Users/hulu_knight/Library/Application Support/typora-user-images/image-20230525235234131.png)

​	随机选取一个你喜欢的，本篇文章选取的是 Not Pure Poole 主题，接下里均以此为例，各位可以根据自己的喜好来选去，使用方法大同小异。点击之后如图所示![截屏2023-05-25 23.55.01](/Users/hulu_knight/Desktop/截屏2023-05-25 23.55.01.png)

​	你可以点击 Demo 来直接从网页查看到该博客是什么样子的，主题的作者们通常都会制作一个简单的 demo 来让大家更好的感受博客界面，你可以点开试试，我这里就不展示了。

​	当你选中了自己喜欢的主题之后，点击 Homepage 跳转到主题作者的 Github ，你可以直接查看到该 Jekyll 的源代码，还有作者写的一些使用方法（通常在 README.md 中）。点击 Homepage 后如图![截屏2023-05-26 00.01.32](/Users/hulu_knight/Desktop/截屏2023-05-26 00.01.32.png)

​	你可以简单浏览一些主题的文件和 README.md 文档内容，然后点击绿色的 Code 按钮，将仓库的 HTTPS 地址复制下来用于我们之后的本地仓库创建。

<img src="/Users/hulu_knight/Library/Application Support/typora-user-images/image-20230526000529127.png" alt="image-20230526000529127" style="zoom:50%;" />

​	接下来打开你的**终端**，在里面输入`git clone HTTPS`, 请将 HTTPS 替换为你刚刚复制的仓库地址，然后等待出现以下图片情况表示 clone 完毕。

<img src="/Users/hulu_knight/Library/Application Support/typora-user-images/image-20230526001025071.png" alt="image-20230526001025071" style="zoom:50%;" />

​	这个时候你打开 访达 - 用户名 ，就会出现一个你刚才选择主题的文件夹，这就是本地仓库，暂时记住本地仓库的名称。如图<img src="/Users/hulu_knight/Desktop/截屏2023-05-26 00.17.39.png" alt="截屏2023-05-26 00.17.39" style="zoom:50%;" />

>  本地仓库你已经创建完毕了，暂时拥有了大佬的模版，我们想要把它变成自己的，那就需要先进行一些配置，安装好 rvm 和 ruby 来继续完成博客的搭建

## 配置好你的 rvm 和 ruby 环境

​	我们在开始之前需要配置 ruby 环境，我们这里使用 rvm 来进行 ruby 环境来安装。

**安装 rvm**

​	想要安装 rvm，我们需要先导入 GPG 密钥。你可以尝试在 ‘终端’ 输入以下命令行

```
gpg2 --keyserver keyserver.ubuntu.com --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB   // 在安装 rvm 之前，我们需要先导入 GPG 密钥
```

​	安装 rvm

```
\curl -sSL https://get.rvm.io | bash -s stable   // 安装 rvm
```

> 该方法是 [RVM 官网](rvm.io) 提供的安装方式，但是博主并没有安装成功，所以如果你也同样失败的话，建议去官网寻找相关问题的解决办法。或者直接使用博主整理的方法来进行安装，请点击 [关于如何在 Mac 上安装 RVM]() 



**查看可安装的 ruby 版本**

​	进入到本地仓库后，我们需要通过 rvm 来查询可以安装的 ruby 版本。在 ‘终端’ 输入以下命令行

```
rvm list known  // rvm 可以安装很多插件，我们只需要选去对应需要的 ruby 版本即可
```

​	运行结果如下

```
hulu_knight@192 ~ % rvm list known
# MRI Rubies
[ruby-]1.8.6[-p420]
[ruby-]1.8.7[-head] # security released on head
[ruby-]1.9.1[-p431]
[ruby-]1.9.2[-p330]
[ruby-]1.9.3[-p551]
[ruby-]2.0.0[-p648]
[ruby-]2.1[.10]
[ruby-]2.2[.10]
[ruby-]2.3[.8]
[ruby-]2.4[.10]
[ruby-]2.5[.8]
[ruby-]2.6[.6]
[ruby-]2.7[.2]
[ruby-]3[.0.0]
ruby-head

# for forks use: rvm install ruby-head-<name> --url https://github.com/github/ruby.git --branch 2.2

# JRuby
jruby-1.6[.8]
jruby-1.7[.27]
jruby-9.1[.17.0]
jruby[-9.2.14.0]
jruby-head

# Rubinius
rbx-1[.4.3]
rbx-2.3[.0]
rbx-2.4[.1]
rbx-2[.5.8]
rbx-3[.107]
rbx-4[.20]
rbx-5[.0]
rbx-head

# TruffleRuby
truffleruby[-20.3.0]

# Opal
opal

# Minimalistic ruby implementation - ISO 30170:2012
mruby-1.0.0
mruby-1.1.0
mruby-1.2.0
mruby-1.3.0
mruby-1[.4.1]
mruby-2.0.1
mruby-2[.1.1]
mruby[-head]

# Ruby Enterprise Edition
ree-1.8.6
ree[-1.8.7][-2012.02]

# Topaz
topaz

# MagLev
maglev-1.0.0
maglev-1.1[RC1]
maglev[-1.2Alpha4]
maglev-head

# Mac OS X Snow Leopard Or Newer
macruby-0.10
macruby-0.11
macruby[-0.12]
macruby-nightly
macruby-head

# IronRuby
ironruby[-1.1.3]
ironruby-head
hulu_knight@192 ~ % 

```



**安装并激活 ruby**

​	我们能从上图发现对应的 ruby 版本有好多 1.8.6 到 3 版本，博主这里建议安装2.7版本的ruby，因为它是目前最稳定的一个版本。安装对应的 ruby 2.7，在 ’终端‘ 输入以下命令

```
rvm install 2.7   // 安装对应的 ruby 2.7
```

​	激活下载的 ruby 2.7，在 ’终端‘ 输入以下命令

```
rvm use 2.7   // 激活下载的 ruby 2.7
```

运行结果如下，则 ruby 激活成功

```
hulu_knight@192 ~ % rvm install 2.7
Already installed ruby-2.7.2.
To reinstall use:

    rvm reinstall ruby-2.7.2

hulu_knight@192 ~ % rvm use 2.7
Using /Users/hulu_knight/.rvm/gems/ruby-2.7.2
hulu_knight@192 ~ % 

```



## 将博客，真正变成你自己的！  

​	以上的所有知识都算是前期的准备工作，到目前为止你什么都还没有真正的见到一个博客到底应该是什么样子的，接下来跟着博主一起来把别人的博客变成自己的吧。

**下载仓库所需插件**

​	首先我们先找到之前在自己克隆好的本地仓库，我以刚下载的 not-pure-poole 主题作为例子，进入到本地仓库。在 ‘终端’ 输入以下命令行

```
cd not-pure-poole  // 注意把注意需要把 not-pure-poole 替换成你的本地仓库名
```

​	进入到本地仓库之后，我们需要下载该仓库所需要的一些插件，在 ’终端‘ 输入以下命令行

```
bundle install   // 自动下载本地仓库所需要的插件
```



**启动 Jekyll 服务器**

​	经过以上步骤，你已经完全把别的大佬的博客模版 down 到了自己的本地，并且在本地配置好的插件，接下来我们就可以以本地为服务器来启动这个博客了。在 ’终端‘ 输入以下命令行

```
jekyll s
```

​	运行结果如下，则启动成功，本地服务器会提供给你一个网址，你可以通过该网址来访问本地服务器部署的博客。

```
hulu_knight@192 not-pure-poole % jekyll s
Configuration file: /Users/hulu_knight/not-pure-poole/_config.yml
            Source: /Users/hulu_knight/not-pure-poole
       Destination: /Users/hulu_knight/not-pure-poole/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
       Jekyll Feed: Generating feed for posts
                    done in 0.232 seconds.
 Auto-regeneration: enabled for '/Users/hulu_knight/not-pure-poole'
    Server address: http://127.0.0.1:4000  *** 就是这个网址 ***
  Server running... press ctrl-c to stop.


```

​	此时保证终端的服务器运行，然后在浏览器中运行输入该网址，你就能成功打开博客了！



## 试运行你的博客

​	当你打开以上网址之后你会发现信息都是别人的，那么接下来我们就来修改博客中的个人信息，将其变为自己的。在修改博客之前，我建议使用一些文本编辑器来修改，比如 sublime ，你可以直接在本地文件中一个一个更改，我这里使用 sublime 软件来举例，直接去网上搜索下载 sublime 最新版本就好，我们会涉及到在 ‘终端’ 中通过命令行启动 sublime ，如果不知道如何启动请看[关于如何在终端中启动 sublime 的方法](http://127.0.0.1:4000)）

## 将博客推送到 Github 仓库当中

# 战后护理

​	好吧好吧，我知道，各位经历了一场恶战，终于取得了战斗的胜利（可能还没有胜利），上战场嘛，受点轻伤什么的不足挂齿，咱们调整好自己的心态，看我妙手回春，保证治好你的各种伤病。不过本人医术有限，能医治的病也就一下那么几种，等以后艺术提高了在加进来！

- Mac 无法安装 brew，提示连接错误
- brew 安装报错，无法找到 git 文件路径
- MacOS 下三种修改 Hosts 文件的方式
- 关于如何稳定访问 Github 的方法
- MacOS Catalina 安装 rvm 是报连接错误
- MacOS 如何安装 RVM
- MacOS 安装 Homebrew 时报连接错误（和第一个问题合并观看）
- 查询 Ruby 版本是出现命令行名称警告
- 配置 git 的两种方法
- MacOS 终端的一些常用命令
- MacOS 创建 bash_profile 文件的方法
- 关于 Mac 终端密码无法显示的问题
- 关于编辑 Jekyll 文件是如何使用 sublime
- Github 仓库生成新的 SSH 密钥
- 将 Github 仓库的默认分支设置为 master
- 关于 Github Page 部署失败 Gem版本不匹配的问题

​	呼，打一场战斗可真不轻松，明明上战场的时候大家很快就打完了，怎么到了治病的时候就出现了这么多奇奇怪怪的毛病呢，不过没有办法，谁让我得保证大家能快快好起来了，如果你们还遇到了什么疑难杂症，尽管发给我，当然我能不能解决就是另外一说了，在问诊的时候可别忘记把自己的病症说清楚了噢。