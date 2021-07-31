---
layout: post
title: mmap相比于fwrite写日志，是否有性能优势?
key: 202107301500
tags: 算法
---

最近想写一个轻巧的日志库。在google搜索有关高性能日志的实现原理。

大多认为用mmap技术可以避免由内核到用户态的页交换。而且当进程崩溃时,内核保证mmap的内存最终都能写入磁盘（但具体啥时候程序已经不可控）。

有人认为日志库的一大特性就是顺序写，读/写缺页依然要中断，mmap与fwrite在这方面性能相差无几。

以下是我在v2ex提的帖子。看大神们的回答着实受益匪浅
[mmap相比于fwrite写日志，是否有性能优势](https://www.v2ex.com/t/791638#reply27)

<br>	
<br>	
<b>原文:<br>
<https://lizijie.github.io/2021/07/17/%E7%94%9F%E6%88%90%E4%B8%8D%E9%87%8D%E5%A4%8D%E9%A2%98%E7%9B%AE-%E9%9A%8F%E6%9C%BA.html>
<br>
作者github:<br>	
<https://github.com/lizijie>
</b>
