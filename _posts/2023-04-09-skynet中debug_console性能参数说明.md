---
layout: post
title: skynet中debug_console性能参数说明
key: 202304091004
tags: lua luacheck
---

[TOC]

[增强skynet中debug_console.lua部分指令](https://lizijie.github.io/2022/11/20/%E5%A2%9E%E5%BC%BAskynet%E4%B8%ADdebug_console.lua%E9%83%A8%E5%88%86%E6%8C%87%E4%BB%A4.html)

# stat
task 有多个个非BREAK状态的协程
mqlen 当前service上下文消息队列长度
cpu 累计工作毫秒
message 累计处理消息数量

# netstat
type 连接类型[LISTEN|TCP|UDP|BIND|CLOSING]
type = 如果type=LISTEN, 会额外有rtime和wtime字段的性能指标
accept 已监听的连接数
socket ip地址
rtime 距离上次读操作（type=LISTEN，即上次accept，TCP/UPD上次读操作）到执行netstat命令时间隔时间秒，该指标不能完全作为性能的参考标准。
wtime 距离上次写操作（type=LISTEN，即上次accept，TCP/UPD上次写操作）到执行netstat命令时间隔时间秒，该指标不能完全作为性能的参考标
read 累计已读字节数量
write 累计已写字节数量
reading 是否可读
writing 是否可写

待续

  
<b>原文:<br>
<https://lizijie.github.io/2023/04/09/skynet%E4%B8%ADdebug_console%E6%80%A7%E8%83%BD%E5%8F%82%E6%95%B0%E8%AF%B4%E6%98%8E.html>
<br>
作者github:<br>
<https://github.com/lizijie>
</b>
