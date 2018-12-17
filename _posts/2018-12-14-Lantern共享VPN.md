---
layout: post
title: Lantern共享VPN
key: 201812141438
tags: vpn
---

[TOC]

### 修改Lantern配置文件
1. 去到lantern的安装路径C:\Users\xxxx\AppData\Roaming\Lantern
2. 打开settings.yaml
3. 将addr项修改为本机地址,端口号自定。如addr: 192.168.0.238:54634
4. 重启Lantern

PS：adr设置为127.0.0.1:54634共享不成功

![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2018-12-14-Lantern%E5%85%B1%E4%BA%ABVPN/settings_example.png)


### 设置设备网络代理
以下分别是手机和window10设置代理的界面
![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2018-12-14-Lantern%E5%85%B1%E4%BA%ABVPN/settings_example.png)

![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2018-12-14-Lantern%E5%85%B1%E4%BA%ABVPN/win10_net_settings.png)



<br>	
<br>	
<b>原文:<br>	
https://lizijie.github.io/2018/12/14/Lantern%E5%85%B1%E4%BA%ABVPN.html
<br>	
作者github:<br>	
<https://github.com/lizijie>	
</b>
