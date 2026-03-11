---
layout: post
title: skynet sharetable.loadtable相同引用table会深复制
key: 202602211504
tags: lua skynet
---

部分策划配置表，会在后端代码运行期间进行二次转换。比如，抽卡以池子类型`drawType`字段和消耗货币`costCurrencyType`区分业务逻辑，而它们定义在同一张表`draw.lua`
``` lua
local datas = {
	{id = 1, drawType = 1, costCurrencyType = 1, ...}
	{id = 2, drawType = 1, costCurrencyType = 2, ...}
	{id = 3, drawType = 2, costCurrencyType = 1, ...}
	{id = 4, drawType = 3, costCurrencyType = 1, ...}
	{id = 5, drawType = 3, costCurrencyType = 2, ...}
	...
}
```

为了方便后端业务代码快速索引，习惯将`drawType`和`costCurrencyType`构建一个新的结构。

```lua
local drawTypeMap = {}

for k, v in pairs(datas) do
	local drawType = v.drawType
	local costCurrencyType = v.costCurrencyType
	if not drawTypeMap[drawType] then
		drawTypeMap[drawType] = {}
	end
	drawTypeMap[drawType][costCurrencyType] = v
end
```
由于`draw.id`与其它模块有外联关系，`drawTypeMap`并不能完全替代datas`的业务需要。最终传入sharetable的结构如下：
```lua
sharetable.loadtable("draw", {
	datas = datas,
	drawTypeMap = drawTypeMap,
}
```
或者是
```lua
sharetable.loadtable("draw", datas)
sharetable.loadtable("drawTypeMap", drawTypeMap)
```
从纯lua角度来看，`drawTypeMap`引用了相同的行数据，行数据应该只有一份。但是`sharetable.loadtable`在将数据传入c层前，`skynet.pack`递归遍历所有字段(c层是`lua-seri.c:luaseri_pack`)是深复制(`mallco`+`memcpy`)，而`skynet.unpack`也不可能还原回原来的引用关系。从实现过程看，`sharetable.loadtable`有大量的内存消耗`pack lua转c`和`unpack c转lua`。
```lua
-- file: skynet/lualib/skynet/sharetable.lua

local function loadtable(filename, ptr, len)
	close_matrix(files[filename])
	local m = core.matrix([[
		local unpack, ptr, len = ...
		return unpack(ptr, len)
	]], skynet.unpack, ptr, len)
	files[filename] = m
end

function sharetable.loadtable(filename, tbl)
	assert(type(tbl) == "table")
	skynet.call(sharetable.address, "lua", "loadtable", filename, skynet.pack(tbl))
end
```

以下是，针对`sharetable.loadtable`的测试代码

```lua
local skynet = require "skynet"
local sharetable = require "skynet.sharetable"

local function print_sharetable_info()
	local info = skynet.call(sharetable.address, "debug", "INFO")
	for k,v in pairs(info) do
		print(k, v.size / 1024 / 1024 .. " MB")
	end
end

skynet.start(function()

	local n = 1024*1024
	local sub = {}
	for i=1,n do
		sub[i] = i
	end

	local parent = {}

	parent.sub1 = sub
	sharetable.loadtable("parent", parent)
	print_sharetable_info()

	parent.sub2 = sub
	sharetable.loadtable("parent", parent)
	print_sharetable_info()
end)
```
输出
```bash
[:00000002] LAUNCH snlua bootstrap
[:00000003] LAUNCH snlua launcher
[:00000004] LAUNCH snlua cdummy
[:00000005] LAUNCH harbor 0 4
[:00000006] LAUNCH snlua datacenterd
[:00000007] LAUNCH snlua service_mgr
[:00000008] LAUNCH snlua testsharetable
[:00000009] LAUNCH snlua service_provider
[:0000000a] LAUNCH snlua service_cell sharetable
parent  9.0196285247802734 MB
parent  18.019736289978027 MB
[:00000002] KILL self
```

不过`skynet.pack`作为协议格式，不应该处理这个问题。解决办法是，使用索引关系而不是直接引用行数据。
```lua
local drawTypeMap = {}

for k, v in pairs(datas) do
	local drawType = v.drawType
	local costCurrencyType = v.costCurrencyType
	local id = v.id -- id

	if not drawTypeMap[drawType] then
		drawTypeMap[drawType] = {}
	end
	drawTypeMap[drawType][costCurrencyType] = id -- 修改！！！ 从直接引用v，改为id
end
```
更彻底的做法，原生配置表在引入在工程前，已处理好二次转换。也就是说，将二次转换为工作转移到导表工具里完成，然后使用`sharetable.loadfile`导入。


<b>原文:<br>
<https://lizijie.github.io/2026/03/02/skynet-sharetable.loadtable%E7%9B%B8%E5%90%8C%E5%BC%95%E7%94%A8table%E4%BC%9A%E6%B7%B1%E5%A4%8D%E5%88%B6.html>
<br>

作者github:<br>
<https://github.com/lizijie>
</b>