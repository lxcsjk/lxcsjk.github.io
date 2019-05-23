---
layout: post
title:  "CTAssetsPickerController 选中图片不显示对号的问题解决"
date:   2016-11-18 19:02:03 +0800
categories: jekyll update
tags: iOS
---

早上AppStore审核通过，下载来看看。突然发现一个选择图片时候选中的标识没有了。

如图 
对号木有了！！！木有了！！！
![对号木有了！！！木有了！！！](http://lxc.xiaocblog.com/20161118163705576)


本来应该是这样的

![这里写图片描述](http://lxc.xiaocblog.com/20161118164044906)


那么久开始看github 

<https://github.com/chiunam/CTAssetsPickerController>

这是提问

<https://github.com/chiunam/CTAssetsPickerController/issues>

已经有人遇到这个问题了

<https://github.com/chiunam/CTAssetsPickerController/issues/231>

原来是CocoaPods 1.0+  的问题。昨天才把 0.39 升级到 1.0+ 而且现在降回去也不方便  又慢 而且 repo已经不支持了 稳定版CocoaPods 都1.1.1了 

直接看最后的回答前面的 bula bula 说的试了没啥用 

这位大哥说的很对。
![这里写图片描述](http://lxc.xiaocblog.com/20161118165004449)

为啥呢。 因为图片没找到。

![这里写图片描述](http://lxc.xiaocblog.com/20161118165222983)

我看了一下源码还真是。上线前手贱升级了CocoaPods 还 

 ` pod update --verbose --no-repo-update`
 
 更新了库没测就直接打包提交了

然后找到CTAssetsPickerController的demo 再找到资源

![这里写图片描述](http://lxc.xiaocblog.com/QQ20161129-0@2x.png)

选中所有png 往提前new出来的Bundle里拖

![这里写图片描述](http://lxc.xiaocblog.com/20161118165839054)


看看项目的target有木有

![这里写图片描述](http://lxc.xiaocblog.com/20161118165947946)

然后Clean一下项目 重新R一下就可以了。