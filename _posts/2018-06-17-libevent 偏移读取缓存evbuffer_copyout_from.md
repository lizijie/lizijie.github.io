---
layout: post
title: libevent 偏移读取缓存evbuffer_copyout_from
key: 201806170925
tags: libevent 缓存读取
---

本文约定的协议包格式
![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2018-06-17-%E5%81%8F%E7%A7%BB%E8%AF%BB%E5%8F%96%E7%BC%93%E5%AD%98evbuffer_copyout_from/libevent_packet_format.png)

**红色区域：**表示5个字节长度的协议包首部(PKG_HEAD)。前1字节存放标识码(TAG)，后2字节存放数据内容长度(BODY_SIZE)、再后2字节存放整个网络包长度(PKG_SIZE)<br>
**蓝色区域：**表示n个字节长度的数据内容(BODY)，

### 读取一个完整网络包
Libevent 2.0.5以前，必须先drain包头信息，才能取出其中的整包长度PKG_SIZE。再将之后收到的内容数据BODY，与包头合并成一个完整的网络包。
Libevent 2.0.5以后，新增了两个接口evbuffer_copyout_from，[见libevent-book](http://www.wangafu.net/~nickm/libevent-book/TOC.html) <br>
evbuffer_copyout_from允许偏移pos个字节位后，从libevent输入缓存区拷贝dataLen个字节长度的数据。该拷贝操作不会drain输入缓存区。
意味着evbuffer_copyout_from偏移PKG_SIZE_OFFSET取出整包长度PKG_SIZE，将其设置为输入缓存区的最低水位，那么等到下次EV_READ事件，就能一次取出整个协议包。<br>
与Libevent 2.0.5以前相比，减少了缓存包头以及合并数据包的操作。<br>

>
> /**
> Read data from the middle of an evbuffer, and leave the buffer unchanged.<br>
> <br>
> If more bytes are requested than are available in the evbuffer, we only extract as many bytes as were available.
> <br>
> @param buf the evbuffer to be read from<br>
> @param pos the position to start reading from<br>
> @param data_out the destination buffer to store the result<br>
> @param datlen the maximum size of the destination buffer<br>
> @return the number of bytes read, or -1 if we can't drain the buffer.<br>
> */<br>
> EVENT2_EXPORT_SYMBOL<br>
> ev_ssize_t evbuffer_copyout_from(struct evbuffer *buf, const struct evbuffer_ptr *pos, void *data_out, size_t datlen);<br>
>

**偏移读取缓存evbuffer_copyout_from参考例子：** <br>
```c
// 协议包头大小
usigned short PKG_SIZE_OFFSET = 3;
unsigned short HEAD_SIZE = 5;

void BodyHander(char* data, unsigned short size)
{
}

void conn_readcb(struct bufferevent *bev, void *ctx)
{
    struct evbuffer *input = bufferevent_get_input(bev);
    size_t recvLen = evbuffer_get_length(input);
    if (recvLen < HEAD_SIZE)
    {
        // 调整最低水位为包头长度，减少无效的输入回调
        bufferevent_setwatermark(bev, EV_READ, HEAD_SIZE, 0);
        return;
    }
    
    unsigned short pkgSize = 0;
    struct evbuffer_ptr pos;
    //  读取整包大小
    evbuffer_ptr_set(input, &pos, PKG_SIZE_OFFSET, EVBUFFER_PTR_SET);
    evbuffer_copyout_from(input, &ptr, &pkgSize, sizeof(unsigned short));

    pkgSize = ntohs(pkgSize);
    if (recvLen < pkgSize)
    {
        // 已经知道整包长度，调整最低水位
        bufferevent_setwatermark(bev, EV_READ, pkgSize, 0);
        return ;
    }

    // 取出整个网络包buff
    char* data= (char*) malloc(pkgSize);
    int buffSize = evbuffer_remove(input, data, pkgSize);

    unsigned char tag = *((unsigned c    har*)data);
    unsigned short bodySize = ntohs(*((unsigned short*)(data + 1)));

     BodyHander(data + HEAD_SIZE, bodySize);

    // 处理下个网络包
    recvLen = evbuffer_get_length(input);
    if (recvLen > 0 )
    {
        conn_readcb(bev, ctx);
    }

    free(data);
}

bufferevent_setcb(bev, conn_readcb, NULL, conn_eventcb, pNet);
bufferevent_enable(bev, EV_READ);
bufferevent_setwatermark(bev, EV_READ, HEAD_SIZE, 0);
```


<br>
<br>
<b>原文:<br>
<https://lizijie.github.io/2018/06/17/libevent-%E5%81%8F%E7%A7%BB%E8%AF%BB%E5%8F%96%E7%BC%93%E5%AD%98evbuffer_copyout_from.html>
<br>
作者github:<br>
<https://github.com/lizijie>
</b>
