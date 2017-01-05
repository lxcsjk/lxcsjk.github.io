---
layout: post
title:  "Invalid Bundle. iPad Multitasking support requires these orientations"
date:   2016-12-22 20:00:03 +0800
categories: jekyll update
tags: iOS
---

前段时间把app适配一下iPad，再提交的时候出了一些问题 

>ERROR ITMS-90474: "Invalid Bundle. iPad Multitasking support requires these orientations: 'UIInterfaceOrientationPortrait,UIInterfaceOrientationPortraitUpsideDown,UIInterfaceOrientationLandscapeLeft,UIInterfaceOrientationLandscapeRight'. Found 'UIInterfaceOrientationPortrait,UIInterfaceOrientationLandscapeLeft,UIInterfaceOrientationLandscapeRight' in bundle "bundle_Name"

查了一下资料原来是 **ios9 iPad 的分屏适配**

---

So~
![](http://oh6uhie7j.bkt.clouddn.com/QQ20161224-132228@2x.png)

再次提交一下 OK啦

---