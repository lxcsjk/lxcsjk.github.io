---
layout: post
title:  "iOS - 打ipa包和提交AppStore等事宜"
date:   2017-02-09 10:00:03 +0800
categories: jekyll update
tags: AppStore
---

整理一下关于打ipa包和提交AppStore一些琐碎简单又可能会有点烦人的流程吧...就是这样 = = 

---

### 准备工作

首先你需要有一个苹果的开发者帐号，一个Mac系统。

如果没有帐号可以在打开[http://developer.apple.com/](http://developer.apple.com/)申请加入苹果的开发者计划。支付99美元每年，怎么申请网上有详细的介绍，在此不多做介绍。

如果你已经有了一个开发者账号，打开 [http://developer.apple.com/](http://developer.apple.com/) 并登录到苹果Account，见下：

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-104236@2x.png)

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-104815@2x.png)

**点击右边的那个。**

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-105231@2x.png)

---

### 申请AppId

点击➕号如图

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-110648@2x.png)

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-110715@2x.png)

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-110853@2x.png)

然后点击 **continue** 即可

---

### 申请发布证书

打开 **钥匙串访问** 

在下图所示的界面，你的电子邮件地址：填你开发者账号的电子邮件地址，常用名称，默认就好，CA空，选择保存到磁盘，点击"继续"：

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-112002@2x.png)

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-112328@2x.png)

查看一下桌面，会发现多了一个 **CertificateSigningRequest.certSigningRequest** 的证书请求文件。

---

### 安装WWDR证书

点击左边 Certificates 中的 Production ，再点击右边的➕号

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-113013@2x.png)

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-113155@2x.png)

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-113201@2x.png)

点击 Choose File 选择我们前面生成在桌面的证书请求文件

然后点击 **Generate**

然后 点击 **Download** 下载后双击 即可添加到钥匙串

---

### 生成Provisioning文件

左边选择 Provisioning Profiles 选项下的 Distribution，来生成一个发布的准备文件：

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-120215@2x.png)

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-120301@2x.png)

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-120331@2x.png)

点击下面的 **Continue** 就可以了

等待几秒钟，Provisioning就可以下载了.

下载后双击添加到本地开发者账号中。

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-121009@2x.png)

---

### 打ipa包

打开 Xcode 选择对应的 **Provisioning** 如下图

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-122612@2x.png)

**确定这个地方不是模拟器和真机**

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-122650@2x.png)

**确定是发布版本不是debug不然包体积会很大**

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-122736@2x.png)

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-122756@2x.png)

最后 点击 **Product -> Archive** 

编译成功后，会弹出这个界面。如果失败请查看之前的哪一步出现了错误。

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-123606@2x.png)

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-123615@2x.png)

**选择一个开发者账号**

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-123634@2x.png)

**点击导出**

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-123701@2x.png)

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-123713@2x.png)

保存到硬盘。

---

### 上传到 iTunes Connect

打开 **Application Loader** (如果没安装去iTunes Connect寻找链接)

#### 注意 上传之前在iTunes Connect必须已经创建好你的app 标识就是Bundle Identifier

**选取ipa包**

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-124618@2x.png)

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-124653@2x.png)

**这里需要等待一些时间**

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-124710@2x.png)

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170209-124808@2x.png)

**更改一些build的版本号重新打包就可以了**

**ipa包检测通过后就能在iTunes Connect中相应的app清单中选择。**

**这仅仅是发布到App Store的流程肯定是不够，还有 关于推送证书的一些小坑 and 打测试ipa包到蒲公英或者PRE这一类分发平台的流程稍后继续。**


---