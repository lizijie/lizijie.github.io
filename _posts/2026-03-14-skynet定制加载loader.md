---
layout: post
title: skynet定制加载loader
key: 202603141504
tags: lua skynet
---

经历了好几个skynet项目，有些项目细节很适合在`lualib/loader.lua`里实现

- 区分service是skynet内置的还是游戏项目新建的
- 针对游戏项目新建的service在newservic成功后，有额外的初始化工作
	- 记录这些地址，后续广播消息用
	- skynet.dispatch额外需要的消息协议

当然，针对前两项是可以在项目早期对skynet.newservice包装一层新接口实现，并规范项目成员使用新接口`myapp.newservice`

```lua
function myapp.newservice(name, ...)
    local addr = assert(skynet.newservice(name, ...))
	skynet.call(addr, "lua", "init", ...)
	-- do something here
    return addr
end
```

为了保持接口的使用习惯一致，我不想直接修改原有代码或者在内网维护一份的skynet分支
实际上，skynet的启动配置有选项`lualoader`可以用来定义自己的`loader`。见[wiki config](https://github.com/cloudwu/skynet/wiki/Config)

```lua
--file: config
-- 此处省略其实配置
start = "main"	-- main script
bootstrap = "snlua bootstrap"	-- The service for b

-- 定义自己的loader
lualoader = "./my_lualib/loader.lua"

```

```lua
-- file:loader.lua

--[[
	此处省略部分代码
]]

_G.require = (require "skynet.require").require

-- 从这里开始修改
local ret = main(select(2, table.unpack(args)))
if ret.my_init then
	asset(pcall(ret.my_init))
	-- do something here
end
```

<b>原文:<br>
<https://lizijie.github.io/2026/03/14/skynet%E5%AE%9A%E5%88%B6%E5%8A%A0%E8%BD%BDloader.html>
<br>

作者github:<br>
<https://github.com/lizijie>
</b>