---
layout: post
title: Jenkins流水线 ssh-agent执行远端代码
key: 202207172204
tags: jenkins
---

[TOC]

个人很反对在Jenkins窗口里写构建代码，原因是这样做不能对代码做版本管理。但将构建代码做到版本管理后，也产生了另外的问题，在提交代码与Jenkins之前来回切换调试构建流程，这过程可谓非常繁琐！！幸亏流水线的出现的出现，解决了这两个痛点。

本文主要介绍ssh-agent在流水线中的运用。鉴于[ssh-agent插件](https://plugins.jenkins.io/ssh-agent/)主页的操作说明不够详细，本人在配置时遇到一些问题，如**不知从哪里配置Jenkins凭据ID**、**在远端执行指令无效**。如果没了解过Jenkins流水线脚本化代码，建议先简单阅读一下[官网说明](https://www.jenkins.io/zh/doc/book/pipeline/syntax/)

# 配置ssh免密登录

ssh-agent插件实际是套用ssh登录，所以让ssh-agent正常工作必须先要配置好ssh环境。配置环境并不复杂且网上有大量可查资料，此处不再另述，或者参考这两篇文章[如何在Ubuntu上开启SSH服务](https://blog.csdn.net/weixin_41642203/article/details/122169451)和[设置 SSH 通过密钥登录](https://www.runoob.com/w3cnote/set-ssh-login-key.html)。


# 配置Jenkins凭据ID
以下是[ssh-agent官网插件](https://plugins.jenkins.io/ssh-agent/)主页的流水线操作样例
```jenkins
steps {
    sshagent(credentials: ['ssh-credentials-id']) {
        sh '''
            ssh user@example.com ...
        '''
    }
}
```
参数`ssh-credentials-id`是指什么？它是由Jenkins统一管理账号安全模块**Manage Credentials
**，为每个credentials分配的credentials-id，ssh-agent有提到配置步骤，不过....

>> First you need to add some SSH Credentials to your instance:
Jenkins | Manage Jenkins | Manage Credentials
![](https://raw.githubusercontent.com/jenkinsci/ssh-agent-plugin/master/docs/images/Screen_Shot_2012-10-26_at_12.25.04.png)
Note that only Private Key based credentials can be used.

我当时看完，还是愣没明白要什么配，按以上说明就没找到配置入口~_~!!此外，其中提到很重置的一点`Note that only Private Key based credentials can be used.`，是说ssh-agent只能用ssh密钥登录的方式，不能账号密码登录。
因此本文是基于Jenkins 2.354 重新梳理凭据ID的配置操作

![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2022-07-17-Jenkins%E6%B5%81%E6%B0%B4%E7%BA%BF%20ssh-agent%E6%89%A7%E8%A1%8C%E8%BF%9C%E7%AB%AF%E4%BB%A3%E7%A0%81/Snipaste_2022-07-19_23-31-01.png)
![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2022-07-17-Jenkins%E6%B5%81%E6%B0%B4%E7%BA%BF%20ssh-agent%E6%89%A7%E8%A1%8C%E8%BF%9C%E7%AB%AF%E4%BB%A3%E7%A0%81/Snipaste_2022-07-19_23-31-29.png)
![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2022-07-17-Jenkins%E6%B5%81%E6%B0%B4%E7%BA%BF%20ssh-agent%E6%89%A7%E8%A1%8C%E8%BF%9C%E7%AB%AF%E4%BB%A3%E7%A0%81/Snipaste_2022-07-19_23-31-49.png)
![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2022-07-17-Jenkins%E6%B5%81%E6%B0%B4%E7%BA%BF%20ssh-agent%E6%89%A7%E8%A1%8C%E8%BF%9C%E7%AB%AF%E4%BB%A3%E7%A0%81/Snipaste_2022-07-19_23-32-06.png)

这里有些信息，与上节[配置ssh免密登录](#配置ssh免密登录)有关
* **ID**选填，不填时自动生成
* **描述**选填
* **Username**是远端用户目录~/.ssh/authorized_keys配置了本端公钥的用户名
* **Private key**是本端私钥
* **passphrase**是生成私钥时密码的密钥，如果当时Enter回车跳过了密码输入，则此处不需要填。
![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2022-07-17-Jenkins%E6%B5%81%E6%B0%B4%E7%BA%BF%20ssh-agent%E6%89%A7%E8%A1%8C%E8%BF%9C%E7%AB%AF%E4%BB%A3%E7%A0%81/Snipaste_2022-07-19_23-34-35.png)

# 在远端执行指令
Jenkins声明式流程线中，需要在`sh`作用域内执行shell指令。先看看一个简单例子，以ssh登录后`pwd`打印用户目录路径为例
```jenkins
pipeline {
    agent any
    stages {
        stage ("test") {
            steps {
                sshagent(credentials: ['ssh-credentials-id']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no -tt YOUR_ACCOUT_NAME@REMOTE_ADDR
                        pwd
                    '''
                }
            }
        }
    }
}
```
然而`pwd`输出的结果是居然是本地路径，并非远端路径。因为`sh`的输入源依然是本地console。为了执行指令传到远端，需要用到`<<EOF`管道方法
```jenkins
pipeline {
    agent any
    stages {
        stage ("test") {
            steps {
                sshagent(credentials: ['ssh-credentials-id']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no -tt YOUR_ACCOUT_NAME@REMOTE_ADDR << EOF
                            pwd
                        exit
                    '''
                }
            }
        }
    }
}
```

# scp双向传输文件
将本地test文件拷贝到远端用户目录，接着在远端修改test，回传本地
```jenkins
pipeline {
    agent any
    stages {
        stage ("test") {
            steps {
                sshagent(credentials: ['ssh-credentials-id']) {
                    sh '''
                        touch test && echo "1" > test
                        scp -rC ./test YOUR_ACCOUT_NAME@REMOTE_ADDR:/YOUR_ACCOUT_HOME/
                        ssh -o StrictHostKeyChecking=no -tt YOUR_ACCOUT_NAME@REMOTE_ADDR << EOF
                            echo "2" >> test
                        exit
                    '''
                    sh '''
                        scp -rC YOUR_ACCOUT_NAME@REMOTE_ADDR:/YOUR_ACCOUT_HOME/test ./
                        cat test
                    '''
                }
            }
        }
    }
}
```

<br> 
<br> 
<b>原文:<br>
<https://lizijie.github.io/2022/07/17/Jenkins%E6%B5%81%E6%B0%B4%E7%BA%BF-ssh-agent%E6%89%A7%E8%A1%8C%E8%BF%9C%E7%AB%AF%E4%BB%A3%E7%A0%81.html>
<br>
作者github:<br>
<https://github.com/lizijie>
</b>
