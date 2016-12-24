---
layout: post
title:  "Missing or invalid signature. The bundle xxxx at bundle path 'Payload"
date:   2016-06-25 20:00:03 +0800
categories: jekyll update
tags: iOS AppStore
---

第一次提交AppStore，遇到很多坑 = =。

一直很迷 = = 

等结束整理一下。

---

提交AppStore时候报了这个

>Missing or invalid signature. The bundle '***' at bundle path 'Payload***

查了查 发现是 [AppleWWDRCA.cer](https://developer.apple.com/certificationauthority/AppleWWDRCA.cer) 证书过期，需要重新下载

---

- 先从链接下载证书，然后在 钥匙串 **Apple Worldwide Developer Relations Certification Authority** 过期证书删了

- 再把你的其他相关开发的证书删除 推送什么的。

- 重新导入下载的 **AppleWWDRCA.cer 和 其他相关的证书** 

- 重新打包提交就好了。


---