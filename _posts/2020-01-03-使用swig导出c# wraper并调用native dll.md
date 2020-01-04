---
layout: post
title: 使用swig导出c# wraper并调用native dll
key: 202001031911
tags: C#
---

[TOC]

Building a C# module是swig导出C# wraper并调用native dll的官方样例。与样例使用gcc和mono-csc不同，本文主要使用window自家的编译指令cl和csc，而其它部分没有区别。

我将所有执行命令放到build.bat脚本中。因为使用了cl和csc，你需要在Visual Studio Command Prompt下运行build.bat，并且传入swig安装路径，如：
`build.bat d:\swigwin-3.0.12\swig.exe`


<br>	
<br>	
<b>原文:<br>
<>
<br>
作者github:<br>	
<https://github.com/lizijie>	
</b>
