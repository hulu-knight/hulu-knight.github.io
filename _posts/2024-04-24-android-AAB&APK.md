---
layout: post
title: android AAB和APK包的区别
tags: android
author: hulu-knight
date: 2024-04-24 13:44 +0800

---



# AAB和APK包的区别

Google Play 应用商店正在不断发展，以满足安卓用户和开发者不断增长的需求和要求。其中最具颠覆性的变化之一将在 8 月到来，届时谷歌应用商店将改用 App Bundles 而不是 APK 作为其标准包格式，这一变化不仅会影响到开发者，也会影响到安卓用户，希望能有更好的效果。

AAB全称Android App Bundles

APK全称Android Package

早在2018年谷歌就启用了AAB新格式（AAB全称为Android App Bundles”），谷歌声称这种新格式将使应用程序文件更小，意味着aab分布式应用程序比通用apk平均少占用15% 的空间。更重要的是，它拓展了应用程序捆缚包的定义，只包含运行App时的必要代码。也就是说，下载了一部分之后，App就可以直接运行，无需等待下载完成再安装。

其实对使用者没有任何影响，甚至终端使用者根本不会看到aab包。严格来讲并非aab替换apk，只是开发者在Google Play发布应用时候由直接发布apk包改为发布aab包，之后由Google Play重新打包成apk。

只不过传统的apk包包含对所有设备的支持，也就是不管三星还是小米手机，一个apk包包含全部的支持。改用aab包发布之后，Google play会新针对不同设备的支持重新打包成apk文件，比如对应小米手机的apk包，就只包含小米手机的支持内容，而不再带有三星手机支持。

aab格式的包不能直接通过aab包的形式安装到手机。
