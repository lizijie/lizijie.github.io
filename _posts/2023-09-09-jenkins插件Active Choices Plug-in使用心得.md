---
layout: post
title: jenkins插件Active Choices Plug-in使用心得
key: 202309092204
tags: jenkins
---

***软件环境***

* Active Choices Plug-in 2.7.2
* Jenkins 2.414.1 (docker: jenkins/jenkins:lts-jdk11)

# Groovy中调用shell

```groovy
// Choice下拉列出当前工作目录的所在子目录
def proc = "ls".execute()
proc.waitFor()
def output = proc.in.text
return output.split("\n") as ArrayList
```
![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2023-09-09-jenkins%E6%8F%92%E4%BB%B6Active%20Choices%20Plug-in%E4%BD%BF%E7%94%A8%E5%BF%83/7091c606-3056-43da-8637-3016a50e8a4f.png)

![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2023-09-09-jenkins%E6%8F%92%E4%BB%B6Active%20Choices%20Plug-in%E4%BD%BF%E7%94%A8%E5%BF%83/8b79cfb4-c791-4eb2-913e-afc790aa9148.png)

# Input text box 不可编辑

Input text box是Active Choices Reactive Reference Parameter里的一个选项，但它的使用非常奇怪。[官方wiki](https://plugins.jenkins.io/uno-choice/#plugin-content-active-choices-reactive-reference-parameter-configuration-options)，提到它只需要返回一个字符串，但是。。。但是。。。我困惑于这个UI控件的文本居然不可编辑，刚开始我一度认为是不是漏了设置什么选项导致的。但最后通过浏览器F12查看对应的html结构,该控件确实是不可编辑。。。(那为什么叫Input text box???)

![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2023-09-09-jenkins%E6%8F%92%E4%BB%B6Active%20Choices%20Plug-in%E4%BD%BF%E7%94%A8%E5%BF%83/f732cd24-335e-4edd-9653-ce9a5a6d9efa.png)

![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2023-09-09-jenkins%E6%8F%92%E4%BB%B6Active%20Choices%20Plug-in%E4%BD%BF%E7%94%A8%E5%BF%83/cb031320-bd50-4f30-a967-d78513e02bc5.png)

![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2023-09-09-jenkins%E6%8F%92%E4%BB%B6Active%20Choices%20Plug-in%E4%BD%BF%E7%94%A8%E5%BF%83/b2902160-6281-4a59-b8d9-3471c23ed12a.png)

# 可编辑输入框

你可以通使用Jenkins内置的控件String Parameter来创建输入框，但本文的主题是第三方插件Active Choices Plug-in，它能通过groovy script可以更灵活更复杂地控制输入框控件。这里用到Active Choices Reactive Reference Parameter的Formatted HTML选项，需要groovy返回符合html格式的字符串，而且更重要的是。。。更重要的是 。。。必须有一个HTML标签的名称是value，即 name="value"，因为这是插件约定了只有这个标签的值会被传递到构建过程[参考](https://wiki.jenkins-ci.org/display/JENKINS/Reactive+Reference+Dynamic+Parameter+Controls)

以下分别两个样例，最终均是在shell输出输入框的内容。其中的区别只是一个HTML标签name='foo'，另一个符合插件要求的name='value'。执行结果是前者$a输出是空的，后者$a输出'输入你的内容'

1. name='foo'

```groovy
"<input name='foo' class='setting-input' type='text' value='输入你的内容'>"
```

![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2023-09-09-jenkins%E6%8F%92%E4%BB%B6Active%20Choices%20Plug-in%E4%BD%BF%E7%94%A8%E5%BF%83/4304773b-58c7-4b25-9fe4-152349c82661.png)

![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2023-09-09-jenkins%E6%8F%92%E4%BB%B6Active%20Choices%20Plug-in%E4%BD%BF%E7%94%A8%E5%BF%83/b16e0805-38a3-4ff0-9573-7da92b777579.png)

![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2023-09-09-jenkins%E6%8F%92%E4%BB%B6Active%20Choices%20Plug-in%E4%BD%BF%E7%94%A8%E5%BF%83/462f7d8e-0803-4ff8-b36c-03f3b916a374.png)

![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2023-09-09-jenkins%E6%8F%92%E4%BB%B6Active%20Choices%20Plug-in%E4%BD%BF%E7%94%A8%E5%BF%83/cb0c6f3c-2780-4ce3-b1f5-81123b31c6df.png)

2. name='value'，其它参数同上
   
```groovy
"<input name='value' class='setting-input' type='text' value='输入你的内容'>"
```

![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2023-09-09-jenkins%E6%8F%92%E4%BB%B6Active%20Choices%20Plug-in%E4%BD%BF%E7%94%A8%E5%BF%83/a6e7719a-1aa8-4f47-921b-25c5539bc034.png)

![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2023-09-09-jenkins%E6%8F%92%E4%BB%B6Active%20Choices%20Plug-in%E4%BD%BF%E7%94%A8%E5%BF%83/6e2a4846-39e3-4f52-8f93-ef126d0d9450.png)

如果你仔细看，上图$a尾部多了输出一个逗号符号，这不是我手抖多打了个符号。插件wiki对此有解释，大意是插件了担心你没有提供name='value'的标签导致不符合预期的结果，于是创建了一个不可见的name='value'的input标签。但当你主动填写name='value'的标签时，这个内部创建的不可见并不会取消，最终就有两个name='value'的标签。如果不需要这个隐藏的标签，需要勾上Omit value field选项
Advanced Option: Omit Value Field

> By default, 'Reactive References' pass to the build a hidden <input name="value" value="">. It means that your 'Reactive Reference' parameter will always be empty, but you can use a 'Formatted HTML' parameter and instruct the plug-in to not include this hidden value parameter.
> You can click the 'Advanced' button and there you will find an option to omit the value field. This will you let you define a value for the hidden parameter.


![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2023-09-09-jenkins%E6%8F%92%E4%BB%B6Active%20Choices%20Plug-in%E4%BD%BF%E7%94%A8%E5%BF%83/1d284a64-2b22-44fa-b931-207073eefb07.png)

![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2023-09-09-jenkins%E6%8F%92%E4%BB%B6Active%20Choices%20Plug-in%E4%BD%BF%E7%94%A8%E5%BF%83/eeffde43-9793-47e8-a8f7-2d196cc915b0.png)

多个name='value'标签的值，返回给$a会是数组吗？毕竟逗号分隔确实也让我也有这方面的猜想，实验结果是$a是以逗号拼接多个值的字符串。
我另外开了一个pipleline工程，多增加一个input标签，并且在pipeline里打印$a的值和$a的getClass

![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2023-09-09-jenkins%E6%8F%92%E4%BB%B6Active%20Choices%20Plug-in%E4%BD%BF%E7%94%A8%E5%BF%83/0c9c1a37-29db-46ac-bc94-ad04f95035ba.png)

![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2023-09-09-jenkins%E6%8F%92%E4%BB%B6Active%20Choices%20Plug-in%E4%BD%BF%E7%94%A8%E5%BF%83/9496ebcd-4167-4ca4-bcd1-a8e77db7ba78.png)

![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2023-09-09-jenkins%E6%8F%92%E4%BB%B6Active%20Choices%20Plug-in%E4%BD%BF%E7%94%A8%E5%BF%83/a435606a-2b1c-47d9-ba76-a0d3fc911800.png)


<b>原文:<br>
<https://lizijie.github.io/2023/09/09/jenkins%E6%8F%92%E4%BB%B6Active-Choices-Plug-in%E4%BD%BF%E7%94%A8%E5%BF%83%E5%BE%97.html>
<br>
作者github:<br>
<https://github.com/lizijie>
</b>



