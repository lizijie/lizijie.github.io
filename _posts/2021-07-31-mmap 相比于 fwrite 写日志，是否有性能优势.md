---
layout: post
title: mmap相比于fwrite写日志，是否有性能优势?
key: 202107301500
tags: 操作系统
---

最近想写一个轻巧的日志库。在google搜索有关高性能日志的实现原理。

大多认为用mmap技术可以避免由内核到用户态的页交换。而且当进程崩溃时,内核保证mmap的内存最终都能写入磁盘（但具体啥时候程序已经不可控）。

有人认为日志库的一大特性就是顺序写，读/写缺页依然要中断，mmap与fwrite在这方面性能相差无几。

以下是我在v2ex提的帖子，反对使用mmap的理由较多
[mmap相比于fwrite写日志，是否有性能优势](https://www.v2ex.com/t/791638#reply27)

<br>	
<br>	
<b>原文:<br>
<https://lizijie.github.io/2021/07/31/mmap-%E7%9B%B8%E6%AF%94%E4%BA%8E-fwrite-%E5%86%99%E6%97%A5%E5%BF%97-%E6%98%AF%E5%90%A6%E6%9C%89%E6%80%A7%E8%83%BD%E4%BC%98%E5%8A%BF.html>
<br>
作者github:<br>	
<https://github.com/lizijie>
</b>
