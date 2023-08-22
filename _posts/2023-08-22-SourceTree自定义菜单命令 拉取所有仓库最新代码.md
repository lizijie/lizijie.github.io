---
layout: post
title: SourceTree自定义菜单命令 拉取所有仓库最新代码
key: 202308212204
tags: lua skynet
---

技术和测试同事会在自己的电脑里，同时搭建前后端环境。但前端技术不会去关注后端的内容，只在需要的时候拉取最新后端内容。但是当工程包含多个外部代码仓库时，同事不熟悉工程结构难免偶尔会漏掉更新其中的某些仓库，导致运行异常浪费排查时间。对于这种只管拉取各个仓库最新代码，且不会去做任何修改和切换分支的情况，提供一键拉取所有仓库最新版本的脚本会更为适合。以下是一个仅支持git和svn仓库更新的模版脚本，

```shell
# file: YOUR_PROJ_ROOT/xxx/update_all_repo.sh
function git_repo() {
    dir=$1
    cd $dir
    echo "branch: "`git branch --show-current`
    git pull --all
    if [ $? -eq 0 ]; then
        echo $dir "更新成功"
    else
        echo $dir "更新失败"
    fi
}

functio svn_repo() {
    dir=$1
    cd $dir && svn up
    if [ $? -eq 0 ]; then
        echo $dir "更新成功"
    else
        echo $dir "更新失败"
    fi
}

# 样例：工程仓库结构
# + YOUR_PROJ_ROOT 工程根目录 git仓库
#   + config  配置表目录 svn仓库
#   + common  公共代码目录 svn仓库
#   + ....
server_root=YOUR_PROJ_ROOT
config_root=$server_root/config
common_root=$server_root/common

git_repo $server_root
svn_repo $common_root
svn_repo $confi_root

```
## 从SourceTree菜单命令执行

顶部菜单栏依次点开【工具】->【选项】->【自定义操作】，出现配置界面如下

![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2023-08-22-SourceTree%E8%87%AA%E5%AE%9A%E4%B9%89%E8%8F%9C%E5%8D%95%E5%91%BD%E4%BB%A4%20%E6%8B%89%E5%8F%96%E6%89%80%E6%9C%89%E4%BB%93%E5%BA%93%E6%9C%80%E6%96%B0%E4%BB%A3%E7%A0%81/Snipaste_2023-08-22_18-58-38.png)

![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2023-08-22-SourceTree%E8%87%AA%E5%AE%9A%E4%B9%89%E8%8F%9C%E5%8D%95%E5%91%BD%E4%BB%A4%20%E6%8B%89%E5%8F%96%E6%89%80%E6%9C%89%E4%BB%93%E5%BA%93%E6%9C%80%E6%96%B0%E4%BB%A3%E7%A0%81/Snipaste_2023-08-22_19-00-00.png)


## 从window git-bash执行

```bat
cmd /d ""YOUR_GIT_ROOT\bin\sh.exe" --login -i -- YOUR_PROJ_ROOT/xxx/update_all_repo.sh"
```


<b>原文:<br>
<https://lizijie.github.io/2023/08/22/SourceTree%E8%87%AA%E5%AE%9A%E4%B9%89%E8%8F%9C%E5%8D%95%E5%91%BD%E4%BB%A4-%E6%8B%89%E5%8F%96%E6%89%80%E6%9C%89%E4%BB%93%E5%BA%93%E6%9C%80%E6%96%B0%E4%BB%A3%E7%A0%81.html>
<br>
作者github:<br>
<https://github.com/lizijie>
</b>

