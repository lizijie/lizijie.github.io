---
layout: post
title: 使用swig导出c# wraper并调用native dll
key: 202001031911
tags: C#
---

[TOC]

[Building a C# module](http://www.swig.org/tutorial.html)是swig导出C# wraper并调用native dll的官方样例。本文没有使用样子里的gcc和mono-csc，而是用了window自家的编译指令cl和csc

我将所有执行命令放到[build.bat](https://github.com/lizijie/csharp_swig_tutorial/blob/master/build.bat)脚本中。因为使用了cl和csc，你需要在Visual Studio Command Prompt下运行build.bat，并且传入swig安装路径，如：
`build.bat d:\swigwin-3.0.12\swig.exe`


<br>	
<br>	
<b>原文:<br>
<https://lizijie.github.io/2020/01/03/%E4%BD%BF%E7%94%A8swig%E5%AF%BC%E5%87%BAc-wraper%E5%B9%B6%E8%B0%83%E7%94%A8native-dll.html>
<br>
作者github:<br>	
<https://github.com/lizijie>	
</b>
