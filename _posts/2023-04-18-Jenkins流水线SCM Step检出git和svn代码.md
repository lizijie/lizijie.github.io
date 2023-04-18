---
layout: post
title: Jenkins流水线SCM Step检出git和svn代码
key: 202304182204
tags: jenkins git svn
---

[TOC]

本文主要介绍使用[Pipeline: SCM Step](https://plugins.jenkins.io/workflow-scm-step/)插件，展示如何对git或svn仓库，进行分支/标签切换。需要注意的是**Pipeline: SCM Step**只是给具体的scm做接口转发，视你使用的具体scm而定，还要安装对应的依赖插件。

* git仓库，安装[git](https://plugins.jenkins.io/git/)
* svn仓库，安装[Subversion](https://plugins.jenkins.io/subversion/)

关于**Pipeline: SCM Step**能支持的哪些SCM见[compatibility list](https://github.com/jenkinsci/pipeline-plugin/blob/master/COMPATIBILITY.md)

# git分支/标签
首先，要了解git内部管理分支/标签的数据结构，可以参考这里两篇文章[Git 内部原理 - Git 引用](https://git-scm.com/book/zh/v2/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-Git-%E5%BC%95%E7%94%A8)和[Git高级操作：refs和reflog](https://zhuanlan.zhihu.com/p/521722781)，有以下结论
* 分支的完整语法：`refs/remotes/<remoteRepoName>/<branchName>`
    * remoteRepoName：`git clone -o`或者`git remote add`时，定义的本地仓库别名，常定义为`origin`。
    * branchName：分支名称
* 标签的完整语法：`refs/tags/<tagName>`
    * branchName：标签名称

然后，从[git](https://www.jenkins.io/doc/pipeline/steps/params/scmgit/#scmgit)插件的文档了解到，下载分支/标签需要通过填写`branches.name`来指定

**注：根据实际项目情况，会用到多个git检出参数，比如子模块递归、子目录检出、账号凭据等，但限于篇幅且不想在本节暴露过多的参数，以免造成读者理解上的混乱。常见的参数都会逐渐在此后的章节中介绍。下例均以`https://github.com/lizijie/jenkins_git_test_proj.git`作为测试仓库。**

**检出分支**
```groovy
node {
    def branch_name = "refs/remotes/origin/main"

    stage("git branch/tag") {
        def result = checkout([
            $class: 'GitSCM', 
            branches: [[name: branch_name]], 
            userRemoteConfigs: [
                [url: 'https://github.com/lizijie/jenkins_git_test_proj.git']
            ]
        ])
        println("$result")
    }
}
```
**检出标签**
```groovy
node {
    def tag_name = "refs/tags/v0.1.0-alpha"

    stage("git branch/tag") {
        def result = checkout([
            $class: 'GitSCM', 
            branches: [[name: tag_name]], 
            userRemoteConfigs: [
                [url: 'https://github.com/lizijie/jenkins_git_test_proj.git']
            ]
        ])
        println("$result")
    }
}
```

svn分支/标签
待续

<br> 
<br> 
<b>原文:<br>
<https://lizijie.github.io/2023/04/18/Jenkins%E6%B5%81%E6%B0%B4%E7%BA%BFSCM-Step%E6%A3%80%E5%87%BAgit%E5%92%8Csvn%E4%BB%A3%E7%A0%81.html>
<br>
作者github:<br>
<https://github.com/lizijie>
</b>
