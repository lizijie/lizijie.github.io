---
layout: post
title: git hook进行luacheck
key: 202603011504
tags: lua git luacheck
---

# git hook
参考官方[git hook文档](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90)，使用`pre-commit`钩子可以在提交前可对代码进行检查。以下让ai总结各个钩子的说明和例子

## 客户端钩子 (本地开发使用)

| 钩子名称 | 触发时机 | 作用描述 | 具体使用例子 |
| :--- | :--- | :--- | :--- |
| **`pre-commit`** | 执行 `git commit` 后，编写提交消息前 | 检查即将提交的代码快照 | 运行 **ESLint** 检查、执行单元测试、检查代码中是否遗留了 `console.log` 或临时密钥。 |
| **`prepare-commit-msg`** | 默认消息创建后，编辑器启动前 | 自动生成或修改提交消息模板 | 根据当前分支名自动在提交消息前缀加上 **Issue 编号**（如 `[FEAT-123] ...`）。 |
| **`commit-msg`** | 编写完消息并保存退出后 | 验证提交消息的格式是否合规 | 强制要求消息必须符合 **Angular 规范**（如必须以 `feat:`, `fix:` 等开头）。 |
| **`post-commit`** | 提交过程全部完成后 | 通知或后续处理（不影响提交结果） | 提交后发送桌面通知，或向本地日志文件记录一次提交活动。 |
| **`post-checkout`** | `git checkout` 或 `git switch` 成功后 | 切换分支后的环境清理或设置 | 切换分支后自动运行 `npm install` 同步依赖，或删除编译产生的临时文件。 |
| **`post-merge`** | `git merge` 成功后 | 合并代码后的同步操作 | 当远程代码合并进本地后，如果 `package.json` 有变动，自动提示或运行 **依赖更新**。 |
| **`pre-push`** | 执行 `git push` 后，远程引用更新前 | 防止不合格的代码被推送到云端 | 运行耗时较长的**全量集成测试**，如果测试失败则阻止推送，保护公共分支。 |
| **`pre-rebase`** | 执行 `git rebase` 前 | 运行变基前的安全性检查 | 检查是否正在对已经推送到服务器的分支进行变基（防止污染他人分支历史）。 |

## 服务端钩子 (代码仓库服务器使用)

| 钩子名称 | 触发时机 | 作用描述 | 具体使用例子 |
| :--- | :--- | :--- | :--- |
| **`pre-receive`** | 服务器收到推送请求，开始更新前 | 核心权限和合规性校验（全局） | 拒绝没有关联 Jira 任务 ID 的推送；**拒绝包含大文件**或敏感信息（如密码）的推送。 |
| **`update`** | 与 pre-receive 类似，但针对每个分支运行一次 | 细粒度的分支访问控制 | 禁止非管理员用户推送代码到 `master` 或 `production` 分支，但允许推送 `feature/*`。 |
| **`post-receive`** | 整个推送过程完成且更新成功后 | 触发自动化流程（CI/CD） | 自动触发 Jenkins/GitLab CI 流水线；**自动部署**代码到测试服务器；发送邮件或 Slack 钉钉通知。 |

# git commit前 时进行代码检查
在日常lua开发，笔者习惯在提交代码前，检查以下几个事项：
- 去除所有`print`调用
- `luacheck`静态检查代码
- ai code review

笔者已知的是`vscode github copilot`和`kilo`有该功能`ai review`，但笔者更期望是接入到pre-commit hook中，后续偿试接入ai检查。在未出现ai以前，笔者使用脚本来前两个目标

# pre-commit安装
以下是官方安装的原文，其实就是将`pre-commit`放到工程根目录`.git/hooks`下即可

> 钩子都被存储在 Git 目录下的 hooks 子目录中。 也即绝大部分项目中的 .git/hooks 。 当你用 git init 初始化一个新版本库时，Git 默认会在这个目录中放置一些示例脚本。 这些脚本除了本身可以被调用外，它们还透露了被触发时所传入的参数。 所有的示例都是 shell 脚本，其中一些还混杂了 Perl 代码，不过，任何正确命名的可执行脚本都可以正常使用 —— 你可以用 Ruby 或 Python，或任何你熟悉的语言编写它们。 这些示例的名字都是以 .sample 结尾，如果你想启用它们，得先移除这个后缀。

>把一个正确命名（不带扩展名）且可执行的文件放入 .git 目录下的 hooks 子目录中，即可激活该钩子脚本。 这样一来，它就能被 Git 调用。接下来，我们会讲解常用的钩子脚本类型。

# pre-commit shell脚本

https://github.com/lizijie/git_hook_example/tree/main/lua/pre-commit

# 关于luacheck
`luacheck`本身是必须搭配其配置才能发挥效果的，而且大部分配置不通用，特别是有些工程逻辑的全局标识luacheck识别不到，需要加入配置里。见[luacheck官方文档](https://luacheck.readthedocs.io/en/stable/)。我写了一个[luacheck配置示例](https://github.com/lizijie/luacheck_cfg_example/blob/main/.luacheckrc)可供参考。

<b>原文:<br>
<https://lizijie.github.io/2026/03/01/git-hook%E8%BF%9B%E8%A1%8Cluacheck.html>
<br>

作者github:<br>
<https://github.com/lizijie>
</b>