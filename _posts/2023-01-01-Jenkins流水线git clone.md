---
layout: post
title: Jenkins流水线git clone
key: 202301012204
tags: jenkins pipeline
---

[TOC]

假设你对以下内容有基础的了解，本文不在另外赘述
1. jenkins pipeline基础语法
2. [Snippet Generator](https://www.jenkins.io/doc/book/pipeline/getting-started/#snippet-generator)导出代码片段
3. [Git plugin 4.11.5](https://plugins.jenkins.io/git/)插件 这是由于导出的git clone选项参数与其界面参数一一对应
4. [jeknins权限管理](https://www.jenkins.io/doc/book/using/using-credentials/#using-credentials) 密码由受jenkins管理，在pipeline中需要特殊方法获取。注：以下谈及到的账号credentialsId:"xxx"的权限类型为**Username with password**

注：以下测试环境使用的jenkins版本Jenkins (2.375.1)


# 方法一：Snippet Generator导出代码

测试环境的插件，如下
* [Pipeline: SCM Step 400.v6b_89a_1317c9a_	](https://plugins.jenkins.io/workflow-scm-step/)
* [Git plugin 4.11.5](https://plugins.jenkins.io/git/)

下面checkout那行代码是通过[Snippet Generator](https://www.jenkins.io/doc/book/pipeline/getting-started/#snippet-generator)导出的。这里只是样例，git clone选项参数不能照抄，具体视你的项目情况而定，特别是credentialsId:"xxx"的权限配置

```java
pipeline {
    agent any

    stages {
        stage("test git clone") {
            steps {
               checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'amao', url: 'https://github.com/lizijie/luacheck_blame_report.git']]]) 
            }
        }
    }
}
```

# 方法二：shell 命令行
测试环境的软件，如下
* git version 1.8.3.1

由于账号权限受jenkins管理，但命令行git clone时需要指定账号&密码，在pipeline中可通过`credentials("xxx")`获得账号密码，以下样例仅仅是权限类型为**Username with password**的处理。对于其它权限类型的获取方法，参考[handling-credentials](https://www.jenkins.io/doc/book/pipeline/jenkinsfile/#handling-credentials)
```java
pipeline {
    agent any

    environment {
        // jenkins此时自动展开两个变量，分别是存放账号名的PROJ_GIT_CREDS_USR，和存放密码的PROJ_GIT_CREDS_PSW
        PROJ_GIT_CREDS = credentials("xxx")
    }

    stages {
        stage("test git clone") {
            steps {
                script {
                    def usr = evn.PROJ_GIT_CREDS_USR
                    def pwd = evn.PROJ_GIT_CREDS_PSW
                    def cmd = "git clone https://$usr:$pwd@github.com/lizijie/luacheck_blame_report.git"
                    sh(script: cmd)
                }
            }
        }
    }
}
```

[](https://www.jenkins.io/doc/book/pipeline/jenkinsfile/#usernames-and-passwords)

  
  
<b>原文:<br>
<https://lizijie.github.io/2023/01/01/Jenkins%E6%B5%81%E6%B0%B4%E7%BA%BFgit-clone.html>
<br>
作者github:<br>
<https://github.com/lizijie>
</b>
