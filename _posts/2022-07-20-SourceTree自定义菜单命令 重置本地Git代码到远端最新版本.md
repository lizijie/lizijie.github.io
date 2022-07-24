---
layout: post
title: SourceTree自定义菜单命令 重置本地Git代码到远端最新版本
key: 202207202204
tags: git
---

为什么要做重置git当前分支代码这样一个菜单命令？源于非技术同事对git操作不了解，在SourceTree上各种胡乱操作一通后，引起了本地内容与远端仓库冲突，进而导致`git pull`失败。
对于非技术同事来说，他们基本不会去修改本地代码的，即对仓库只读，所以将其本地分支`git reset`到远端版本就可以就能解决。

SourceTree这么优秀的软件，当然有界面去操作重置本地分支
1. 先点【获取】按钮，弹出的窗点确认
2. 选中提交日志中，选中origin的最新版本
3. 右键后，选择【重置当前分支到此提交】，弹出的窗点确认
![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2022-07-20-SourceTree%E8%87%AA%E5%AE%9A%E4%B9%89%E8%8F%9C%E5%8D%95%E5%91%BD%E4%BB%A4%20%E9%87%8D%E7%BD%AE%E6%9C%AC%E5%9C%B0Git%E4%BB%A3%E7%A0%81%E5%88%B0%E8%BF%9C%E7%AB%AF%E6%9C%80%E6%96%B0%E7%89%88%E6%9C%AC/Snipaste_2022-07-24_23-13-32.png)

一顿操作猛如虎！！！尽管只有3个步骤，这对于非技能来讲，已经有点懵了....

所以借助SourceTree的**自定义操作**菜单，新建一条重置命令，只需点一下菜单，即可完成重置工作。SourceTree配置自定义操作如下，顶部菜单栏依次点开【工具】->【选项】->【自定义操作】，出现配置界面如下

![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2022-07-20-SourceTree%E8%87%AA%E5%AE%9A%E4%B9%89%E8%8F%9C%E5%8D%95%E5%91%BD%E4%BB%A4%20%E9%87%8D%E7%BD%AE%E6%9C%AC%E5%9C%B0Git%E4%BB%A3%E7%A0%81%E5%88%B0%E8%BF%9C%E7%AB%AF%E6%9C%80%E6%96%B0%E7%89%88%E6%9C%AC/Snipaste_2022-07-24_23-18-25.png)

利用`git-bash`运行我们的脚本，我这里的`test.sh`执行如下代码。关键代码是`git reset --hard origin/YOUR_BRANCH_NAME`

```shell
read -p "警告！重置当前分支，将会删除本地修改。按任务键继续执行。如取消则关闭窗口" -n 1

BRANCH_NAME=`git symbolic-ref --short -q HEAD`
ORIGIN_BRANCH_NAME=origin/${BRANCH_NAME}
echo reset target branch: ${ORIGIN_BRANCH_NAME}
git reset --hard ${ORIGIN_BRANCH_NAME}
if [ $? == 0 ]; then
    echo -e "\033[32m 成功 \033\0m"
else
    echo -e "\033[31m 失败 \033\0m"
fi

read -p "按任意键关闭" -n 1
```

![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2022-07-20-SourceTree%E8%87%AA%E5%AE%9A%E4%B9%89%E8%8F%9C%E5%8D%95%E5%91%BD%E4%BB%A4%20%E9%87%8D%E7%BD%AE%E6%9C%AC%E5%9C%B0Git%E4%BB%A3%E7%A0%81%E5%88%B0%E8%BF%9C%E7%AB%AF%E6%9C%80%E6%96%B0%E7%89%88%E6%9C%AC/Snipaste_2022-07-24_23-24-58.png)

<br> 
<br> 
<b>原文:<br>
<https://lizijie.github.io/2022/07/20/SourceTree%E8%87%AA%E5%AE%9A%E4%B9%89%E8%8F%9C%E5%8D%95%E5%91%BD%E4%BB%A4-%E9%87%8D%E7%BD%AE%E6%9C%AC%E5%9C%B0Git%E4%BB%A3%E7%A0%81%E5%88%B0%E8%BF%9C%E7%AB%AF%E6%9C%80%E6%96%B0%E7%89%88%E6%9C%AC.html>
<br>
作者github:<br>
<https://github.com/lizijie>
</b>
