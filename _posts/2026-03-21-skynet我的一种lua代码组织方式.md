---
layout: post
title: 2026-03-21-skynet我的一种lua代码组织方式
key: 202603211504
tags: lua skynet
---

请注意本文讨论的是Lua代码的目录组织而非C代码

之前参与的几个项目都是直接以origin/skynet作为工程根目录。但当在创建新工程时，我开始思考工程目录结构的问题。

此前，为了更加方便，我给[debug_console的性能参数增加排序功能](https://github.com/lizijie/skynet_debug_console_enhance)

但我不想直接修改原有代码，也不希望在内网维护修改后的skynet分支。

# 基于搜索优先级，加载lua代码

假设此时我有另外一份`debug_console`代码放在`./app/service/debug_console.lua`。根据[启动config](https://github.com/cloudwu/skynet/wiki/Config)和https://github.com/cloudwu/skynet/blob/master/lualib/loader.lua#L11，只需将`./app/service`在`luaservice`选项中靠前定义，即可优先加载`./app/service/debug_console.lua`而不是`skynet/service/debug_console.lua`

更进一步

- 为了保持skynet原有的目录风格:
	- `lualib`存放纯Lua代码
	- `service`存放skynet service
	- `luaclib`存放动态库

- skynet作为新项目的submodule
```bash
├── skynet
│   ├── service
│   ├── cservice
│   └── lualib
│   └── luaclib
│   └── ...
│
├── my_app
│   ├── service
│   ├── cservice
│   └── lualib
│   └── luaclib
│   └── ...
│
├── service
├── cservice
└── lualib
└── luaclib
└── ...
```

- skynet启动配置的解析实现使用了`luaL_newstate`(参见https://github.com/cloudwu/skynet/blob/master/skynet-src/skynet_main.c#L144-L157)。即启动配置可以包含Lua代码。不过`assert(load(code,[[@]]..filename,[[t]],result))()\n\`, result参数用于收集配置选项，占用env参数，导致`_G`缺失，无法使用辅助函数。如果你计划让启动配置设计得更为复杂，迫切需要用到辅助函数，给`result`再做一层元表索引到`_G`即可。

```lua
	setmetatable(result, { __index = { include = include } })\n\
```
修改后
```lua
	local meta = { include = include }\n\
	setmetatable(meta, {__index = _G})\n\

	setmetatable(result, { __index = meta})\n\
```

# 优先级加载示例

```lua
local function append_search_paths(root)
    luaservice = root.."/?.lua;" ..
                 root.."/service/?.lua;" ..
                 (luaservice or "")

    lua_path = root .. "/?.lua;" ..
               root.."/lualib/?.lua;"..
               (lua_path or "")

    lua_cpath = root .. "/luaclib/?.so;" ..
                (lua_cpath or "")

    cpath = root.."/cservice/?.so;" ..
            (cpath or "")

end

local root_list = {"./", "./my_app", "./skynet"}
for i = 1, #root_list, 1 do
    local root = root_list[i]
    append_search_paths(root)
end

thread = 8
logger = nil
logpath = "."
harbor = 0
start = "main"	-- main script
bootstrap = "snlua bootstrap"	-- The service for bootstrap
```



<b>原文:<br>
<https://lizijie.github.io/2026/03/02/skynet-sharetable.loadtable%E7%9B%B8%E5%90%8C%E5%BC%95%E7%94%A8table%E4%BC%9A%E6%B7%B1%E5%A4%8D%E5%88%B6.html>
<br>

作者github:<br>
<https://github.com/lizijie>
</b>