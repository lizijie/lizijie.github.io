---
layout: post
title: zlog运行时增加format和rule规则
key: 201803222210
tags: 日志 zlog
---

[zlog](http://hardysimpson.github.io/zlog/UsersGuide-CN.html)是一个日志库，我对它的配置化操作很感兴趣

我希望zlog能在运行时增加format和rule规则，这样做有个好处（见[issues](https://github.com/HardySimpson/zlog/issues/118)
）： 相同的日志输出格式可以在运行时，统一定义。


将zlog fork下来，[尝试修改了一下](https://github.com/lizijie/zlog)。

如下，zlog.h添加增删[format]和[rule]规则的接口（删规则还未实现）

```c
int zlog_add_format(const char *ctx);
void zlog_del_format(const char *name);

int zlog_add_rule(const char *ctx);
void zlog_del_rule(const char *name);
```

test目录里增加了相应的测试用例[test_add_rule.c](https://github.com/lizijie/zlog/blob/master/test/test_add_rule.c)
```c
// File:test_add_rule.c
int main(int argc, char** argv)
{
    int rc;

    rc = zlog_init("test_add_rule.conf");
    if (rc) {
        printf("init failed\n");
        return -1;
    }

    zlog_add_format("simple = \"this is my _cat %m%n\"");
    zlog_add_rule("my_cat.*    \"test_add_rule.log\", 100MB * 32 ; simple");

    dzlog_set_category("my_cat");

    dzlog_info("hello, zlog");

    zlog_fini();
    
    return 0;
}
```

<br>
<br>
<b>原文:<br>
<https://lizijie.github.io/2018/03/22/zlog%E8%BF%90%E8%A1%8C%E6%97%B6%E5%A2%9E%E5%8A%A0format%E5%92%8Crule%E8%A7%84%E5%88%99.html>
<br>
作者github:<br>
<https://github.com/lizijie>
</b>
