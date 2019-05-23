---
layout: post
title:  "Doesn't include the currently selected device "
date:   2016-11-30 20:00:03 +0800
categories: jekyll update
tags: iPad
---

昨晚把一直想买的iPad mini4给拿下了，晚上11.34下的单 第二天的3.30就拿到了，京东真是快。

本来一天都在弄文件下载，断点下载进度条多线程之类的事情，老大说把我们app RUN 到iPad上看看。R了一下bulid失败了。错误信息

> Provisioning profile "match Development com.XXX.YYY" doesn't include the currently selected device "XXX".
> Code signing is required for product type 'Application' in SDK 'iOS 10.0'

Xcode已经是最新的了，iPad是10.0.1绝对是没问题的应该。于是Google了一下。找到了这个解答

>[Xcode 8 cannot run on device, provisioning profile problems mentioning Apple Watch](http://stackoverflow.com/questions/39426287/xcode-8-cannot-run-on-device-provisioning-profile-problems-mentioning-apple-wat)

---

**Answer**


>I had the same issue today - XCode Version 8.0 (8A218a) - and fixed it with two simple steps (instead of the more complicated approach above:
>
1. add the Apple Watch to member center (did not find a copy&paste option either)
2. edit the development provisioning profile and add the watch to devices, save
3. go to XCode prefs, move the old provisioning profile to trash (right click on the name) and download the new version
4. set the new provisioning profile in project editor
No restart, clean or anything else needed. Worked like a charm

需要在描述文件添加设备的UDID

UDID可以通过 ```iTunes``` 拿到。
![](http://lxc.xiaocblog.com/QQ20161130-0@2x.png)
**如果看不到UDID多点几下我抹红的地方就行了**

拿到这个UDID 登录到 [苹果开发者中心](https://developer.apple.com)

登录账号后 找到

![](http://lxc.xiaocblog.com/QQ20161130-2@2x.png)

然后

![](http://lxc.xiaocblog.com/QQ20161130-3@2x.png)

![](http://lxc.xiaocblog.com/QQ20161130-4@2x.png)

最好在描述文件添加这个设备

点 **edit**

![](http://lxc.xiaocblog.com/QQ20161130-1@2x.png)

**Select All**

![](http://lxc.xiaocblog.com/QQ20161130-5@2x.png)

**Generate**

再去xcode 重新下载一下描述文件 clean 一下就OK了




