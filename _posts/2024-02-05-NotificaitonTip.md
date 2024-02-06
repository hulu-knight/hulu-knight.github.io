---
layout: post
title: android notification 消息提醒踩的坑
tags: kotlin&notification
author: hulu-knight
date: 2024-02-05 21:49 +0800
---


最近项目本来以为只是一次简单的通知更新，原来公司的项目一直用的就是系统的通知ui，这次产品要求使用自定义的消息通知样式，我之前虽然有过了解，知道打开用remoteview就可以实现了，只不过可能需要注意的东西有点多，但是真正做起来我才发现不是一般的多，主要还是android系统对自定义通知的限制极大，而且不同的android系统对消息通知的限制还各不相同，那么就让我们一起看看我都踩了哪些坑吧。

那么先来看看我觉得如果你想要进行自定义的消息ui需要注意的点都有哪些：
- remoteview内支持的view都有哪些
- notification根据android不同版本有着不同的高度限制
- 多条消息的折叠逻辑


## 1. remoteview内支持的view

- 支持的布局

1. FrameLayout
2. LinearLayout
3. RelativeLayout
4. GridLayout

- 支持的View

1. 一般View：Button、ImageButton、TextView、ImageView、ProgressBar
2. 集合：ListView、GridView、StackView、AdapterViewFlipper
3. 其余View：AnalogClock、Chronometer、ViewFlipper

## 2. notification严格的高度限制

### 1-1 三种通知状态

Headsup: 悬浮窗

contentremotview：下拉菜单折叠的通知

bigcontentremotview：下拉菜单展开后的通知

### 1-2 不同android系统的通知高度限制

Android6.0的悬浮窗展示、下拉菜单展开展示、下拉列表折叠的展示高度均是最小，如果你能适配到android6.0的自定义样式，那么基本以上版本的都可以正常展示（但是以android6.0的高度限制来做的ui，甚至还不如android12系统自带的ui限制）

~~由于严格的高度限制，所有同一个系统的手机的通知高度限制都是一样的，不按照手机分辨率来进行高度区分，所以大家在用remoteview设计布局的时候一定要注意，不能使用自己公司或者一些其他根据手机分辨率适配出来的dp值，只用系统的dp值来设计就可以了，不然可能会出现一些适配的展示问题。~~

## 3.android12系统的通知icon和app名称以及时间的展示区分

**android12系统以下** 的消息通知在使用remoteview的时候，会出现自定义的ui全部占满通知的情况，及你是看不到通知icon和app名称以及累计时间的如图：

悬浮窗：![](https://raw.githubusercontent.com/hulu-knight/Clouding-Pic/master/picture%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202024-02-06%20231444.png)

下拉菜单栏：![](https://raw.githubusercontent.com/hulu-knight/Clouding-Pic/master/picture.png)

**android12以上** 系统则会展示出来icon，appname，time并且包含折叠按钮。

悬浮窗：

- 折叠形态：![](https://raw.githubusercontent.com/hulu-knight/Clouding-Pic/master/picture%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202024-02-06%20235526.png)
- 展开形态：![](https://raw.githubusercontent.com/hulu-knight/Clouding-Pic/master/picture%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202024-02-06%20235521.png)

下拉菜单一样

**所以**12系统以下无论是否折叠，自定义的layout都没有沾满整个通知界面。

那么我们至少需要两套ui来分别区分android12系统以上和以下的通知样式了。当然如果你的自定义ui比较简单，你依然可以使用同一套ui，但是如果这样，还不如直接使用系统的通知，还不用自己适配。



但是需要注意展示的区域，理论上来说系统越高，可展示的自定义高度越高，但是指的是展示的区域，实际上如图所示，展示区域不同，也就是说明了上面不同系统的为什么不展示icon,名称和时间了

我们无法通过代码展示上述（icon和app名称以及时间）的信息，所以一定要事先确定android12以下需不需要展示累计时间，再来确定要不要使用remoteview

同样更新remoteview的ui，只能通过发一个新的通知消息来更新，那么如果你想单独设计一个类似系统的倒计时如“1分钟前”，”1小时前“，”1天前“，那么你就需要单独来设计这个逻辑来进行通知的更新，通常可以用service，或者job service来实现，这里暂时不细说。但是如果你想展示一个例如倒计时/计时/系统时间展示，你就需要在在考虑以上实现的基础上，考虑性能，内存的问题，所以请慎重看待「**是否需要android12以下系统展示消息通知的累计时间**」

毕竟相比于设置一个通知来说，这个可算是一个大的工程

## 4. Android 不同版本的的通知折叠逻辑区分

当你一个 App 发送了多条消息，那么通知将会按照不同版本来进行通知的折叠，并显示在下拉菜单栏里。大概可以分为以下三类：（以下均是在下拉菜单的展示逻辑）

- Android 8 以下 ：你发送的每一条消息都会优先折叠展示，可以通过下拉通知来展示不折叠的样式

- Android 9-11 ：发送的第一条消息展开状态展示，当第二条同等优先级的消息发送，消息1折叠，消息2展开状态展示，消息3发送后，消息1，2折叠，消息3完全展示，以此类推

- Android 12：发送第一条消息完全展示，当消息2发送，消息1，2一起被手机成一个消息列表，隐藏起来，你可以通过点击展开来展示出来消息1，2的折叠状态，同时依然可以分别对消息进行下来展开操作。

- Android 13：折叠逻辑同12，但是这里有一点不同的是，Android 13的第一条消息是完全展示，第二条折叠后成一个消息列表之后，你点开会发现它多出了一个icon，这个icon是你自己设置的bigicon，你可以选择不设置，这样展示的效果就和android12一样了，当然你也可以重新设计一版ui用来单独适配13系统。

  **值得注意的是，Android12并没有办法通过设置bigicon来展示像13 系统一样的图标。**
## 5. 关于如何设置remoteview部分控件的属性



## 6. Android 5，7不同的顶部菜单栏图标样式





## 总结

好了以上就是我踩到的一些坑了，大家如果了解了之后应该就能更容易地完成自定义消息通知央视样式了