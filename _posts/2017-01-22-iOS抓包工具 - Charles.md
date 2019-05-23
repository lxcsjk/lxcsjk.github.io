---
layout: post
title:  "iOS抓包工具 - Charles"
date:   2017-01-22 20:00:03 +0800
categories: jekyll update
tags: Charles
---

过年也在家也没啥事情，闲着无聊就整理整理自己学习iOS的经历以及解决的问题and知识点（虽然我很菜😂😂）。温故而知新吧😀😀

---

**Charles** 一个网络抓包工具，可以清晰看到网络请求参数值以及返回的response结果。对于日常开发还是非常有帮助的。

### 安装 Charles

[有钱请支持正版](https://www.charlesproxy.com/)

[没钱戳这里(请支持正版)](http://lxc.xiaocblog.com/charles-proxy-4.0.1.zip)

拖入  **应用程序**  即可

---

### Charles的简单使用

1. 将Charles设置成系统的代理服务器。

如图。

![](http://lxc.xiaocblog.com/QQ20170122-193921@2x.png)

然后就可以看到很多网络请求出现在Charles的界面里

![](http://lxc.xiaocblog.com/QQ20170122-193551@2x.png)

2. 显示视图

Charles提供了两种显示视图的方式，分别为 **Structure** 和 **Sequence**。

区别如下：

- **Structure** 视图将网络请求按访问的域名分类。
- **Sequence** 视图将网络请求按访问的时间排序。

前面的图为**Structure** 

下面的图为**Sequence**
![](http://lxc.xiaocblog.com/QQ20170122-194802@2x.png)

单击其中一个网络请求，根据面板上的tag即可在下面空白看到相应的**request**请求参数和**response**返回结果了

3. 过滤网络请求

- 在 **Sequence** 视图模式有一个 **Filter** 的输入框

![](http://lxc.xiaocblog.com/QQ20170122-195706@2x.png)

- 另一种方式，选中一个在右键出的下拉菜单中点击 **Focus**，然后**Filter** 的输入框后面勾选 **Focused** 

![](http://lxc.xiaocblog.com/QQ20170122-200349@2x.png)

- 在 **Structure** 视图模式 在菜单栏选择 ```Proxy->Recording Settings```，然后选择 Include 栏，选择添加一个项目，然后填入需要监控的协议，主机地址，端口号。

![](http://lxc.xiaocblog.com/QQ20170122-200139@2x.png)

---

### 抓取移动设备上的请求封包

1. 设置Charles的代理功能 如图

![](http://lxc.xiaocblog.com/QQ20170122-201137@2x.png)

2. 手机上的设置

PC和手机最好用同一个网关要方便点，毕竟都是在办公室工作嘛。

- 获取到电脑的ip地址 

![](http://lxc.xiaocblog.com/QQ20170122-201522@2x.png)

- 设置手机的HTTP代理

进入到 "设置" - "无线局域网" - "【你的WiFi名字】旁边的详情"
向下拉到最底，填入你PC的ip和Charles的代理端口号 

![](http://lxc.xiaocblog.com/6646E7369EAC5D316F3E2C9AFD4C5513.png)

选择 **Allow** 

![](http://lxc.xiaocblog.com/QQ20170122-202158@2x.png)

在面版将会看到手机发出接受的网络请求了。

---

### 抓包HTTPS

1. 安装 **Charles** 的CA证书

"菜单栏" - "SSL Proxying" - "Install Charles Root Certificate"

![](http://lxc.xiaocblog.com/QQ20170122-202745@2x.png)

在钥匙串中可以搜索到
![](http://lxc.xiaocblog.com/QQ20170122-202909@2x.png)

然后选中一条 HTTPS 请求，右键选择 **Enable SSL proxy** 然后就能看到不再乱码的请求信息了

![](http://lxc.xiaocblog.com/QQ20170122-203656@2x.png)

2. 抓包移动设备的SSL请求

这里需要在手机上安装一个证书，如图

![](http://lxc.xiaocblog.com/C140ACC2-C585-4CCA-9CE2-394FCA2B9359.png)

![](http://lxc.xiaocblog.com/QQ20170122-203821@2x.png)

弹出一个提示告诉你 需要设置代理然后用手机浏览器打开一个地址安装一个证书  [地址](http://chls.pro/ssl)

![](http://lxc.xiaocblog.com/1D32AC22C2B0BF8458259E30A7DF427F.png)

然后和上面一样 **Enable SSL proxy** 

example ~

抓包keep的小视频

![](http://lxc.xiaocblog.com/QQ20170122-205909@2x.png)


---

### 模拟网络环境差

Charles提供这样的支持，只需要更改添加配置与host即可。如图

![](http://lxc.xiaocblog.com/QQ20170122-205122@2x.png)

---

目前常常用到就是这么多~~~ 如果有不足可以看看 [唐巧大神的博客](http://blog.devtang.com/2015/11/14/charles-introduction/)
