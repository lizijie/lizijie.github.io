---
layout: post
title: luarocks里一个纯lua热更的方案 xlua实时热更示例
key: 202602211504
tags: lua 热更
---

luarocks里搜索到的热更方案`https://luarocks.org/modules/jinq0123/hotfix`，该方案也是笔者在好多年前，在工作中看到别人用到，自己也在后面的项目尝试使用。

- 以文件为热更单元
- 满足大部分热更情况，不能热更内容将在后面选节提及，只稍加注意就能避免
- 对工程无入侵式编码

简单说一下，lua脚本return table的热更过程
- 若value不是function
    - 新table的新增值，复制到旧table
    - 新、旧table的同层级metatable,递归执行以上操作
- 若value是function
    - 旧function的upvalue复制到新function
        - 如果upvalue是table，递归执行以上[若value不是function]
- 全表扫描被引用的旧function，替换为新function
    - 扫描新table内部(上面复制)的相互引用的旧function
    - 扫描_G和debug.getregistry，旧function

缺点就是因全表扫描导致的性能消耗，在我的skynet项目中，做过性能收集。
- 16个逻辑核64G内存
- 一个skynet service只服务一个游戏角色逻辑
- 一次大概5个文件左右的bug热更修复
- 平均每个agent耗时（或者认为单个角色），大约在0.00x-0.0x（秒）之间

当然不能一概而论，不同机器、不同业务量，所得的性能数据可能会有差异，所以此处仅作参考。

但问题，如果好几百甚至上千同时在线，岂不是要卡好几秒？我实验过，如果是同时所有lua vm进行热更，确实会很卡。

不过，从出现问题到修复问题消耗的时间本质上是难以预期的，到最后执行技术热更这一步的耗时与前面的耗时占比几乎可以忽略不计。

也就是说，在执行技术热更时，快点热更完成和晚点完成在运营角度没有太大区分。当然时间不能太长，比如控制在几分钏内，所有节点服务都完成热更，基本都能接受。

笔者想说的目的是，用时间换性能，完全可用更精细化的技术流程，拉平这个热更耗时问题

- 使用队列，控制同一帧下，需要热更lua vm的数量
- 避免与与当前角色的高耗行为同时进行，比如战斗。延时后到战斗离开场景时再热更。

按我的线上热更经验，如果做到以上，游戏前端几乎无感。

# 目前发现不能热更的情况
- 未结束的协程函数
- 删除原有的引用。影响无害，因为最新为nil，也表明没有有其它引用它的地方
```lua
local function write_test_lua(s)
    local f = assert(io.open("test.lua", "w"))
    f:write(s)
    assert(f:close())
end  -- write_test_lua()

write_test_lua([[
        local M = {}
        function M:foo()
            return 1
        end
        return M
]])

local test = require("test")
print(test, test.foo, test.bar)

write_test_lua([[
        local M = {}

        -- 删除foo
        -- 新增bar
        function M:bar()
            return
        end

        return M
]])
hotfix.hotfix_module("test")
print(test, test.foo, test.bar)
```
输出如下：
```bash
table: 0x871014800      function: 0x871002580      nil
table: 0x871014800      nil     function: 0x8710025e0
```

- 外部缓存table的引用

```lua
write_test_lua([[
        local M = {}
        function M:foo()
            return "old"
        end
        return M
]])

local test = require("test")
print(test, test.foo)
local my_foo = test.foo
print(my_foo())

write_test_lua([[
        local M = {}

        function M:foo()
            return "new"
        end
        return M
]])
hotfix.hotfix_module("test")
print(test, test.foo)
print(my_foo())

```
输出如下：
```bash
table: 0x871014800      function: 0x871002580
old
table: 0x871014800      function: 0x8710025e0
old
```

# xlua开发环境实时热更示例
笔者非常有必要再次说明一下，`https://luarocks.org/modules/jinq0123/hotfix`是一个纯纯纯纯lua热更方案，它不绑定其它技术栈，只需要进行简单的接入工作即可工作。笔者早期是将其使用在后端开发中的，后来，为了免于每次修改完lua代码后，忍受几十秒的加载时间，重启运行`unity3d xlua`才能生效。我尝试在`unity3d xlua`加入这个方案。不过，发现它并不支持前端常见自定义加载模块的做法，见我提交的`https://github.com/jinq0123/hotfix/pull/6`。

如果想手机直连实现修改lua代码调试，理论上该方案也是可行的，但笔者没有尝试过，另需验证。

我写了几个简单的xlua接入示例，`https://github.com/lizijie/pure_lua_hotfix_xlua_example/blob/main/Assets/TestPureLuaHotfix/TestSimple/TestSimple.cs`

```c#
public void TestRef()
    {
        ClearRequire(TEST_LUA);

            // 写入初始Lua脚本
        WriteLuaCode(@"
local M = {}
function M:bar()
    return 'old'
end
return M");
 
        object[] objs = luaEnv.DoString($@"
        local lib = require '{TEST_LUA}'
        return function()
            return lib:bar()
        end
        ");

        // 热更前的输出
        LuaFunction fn = objs[0] as LuaFunction;
        object[] result1 = fn.Call();
        string s1 = Convert.ToString(result1[0]);

        // 修改文件内容(热更)
       WriteLuaCode(@"
local M = {}
function M:bar()
    return 'new'
end
return M");
        // 同一个c#引用，xlua在c#做了伪索引
        // 热更后的输出
        Hotfix(TEST_LUA);
        object[] result2 = fn.Call();
        string s2 = Convert.ToString(result2[0]);

        Debug.Assert(s1 == "old" && s2 == "new", $"ref by lua test failed, s1:{s1} s2:{s2}");
    }

    public void TestUpvalue()
    {
        ClearRequire(TEST_LUA);

        WriteLuaCode(@"
local n = 1
return function()
n = n + 1
return 'old', n
end");

        object[] objs = luaEnv.DoString($@"
        return require '{TEST_LUA}'
    ");
        LuaFunction fn = objs[0] as LuaFunction;

        // 热更前
        object[] result1 = fn.Call();
        string s1 = Convert.ToString(result1[0]);
        int n1 = Convert.ToInt32(result1[1]);

        WriteLuaCode(@"
local n = 1
return function ()
    n = n + 1
    return 'new', n
end");

        Hotfix(TEST_LUA);

        // 热更后
        object[] result2 = fn.Call();
        string s2 = Convert.ToString(result2[0]);
        int n2 = Convert.ToInt32(result2[1]);
        
        Debug.Assert(s1 == "old" && s2 == "new" && n1 == 2 && n2 == 3, $"upvalue test failed, s1:{s1} s2:{s2} n1:{n1} n2:{n2}");
    }

    public void TestButtonClickEvent()
    {
        ClearRequire(TEST_LUA);

         // 写入初始Lua脚本
        WriteLuaCode(@"
local M = {}
local n = 1
function M:onBtnClick()
    print('onBtnClick old', n)
    n = n + 1
end
return M");

        // 将Lua方法注册到Button.onClick
        object[] rets = luaEnv.DoString($@"
        local lib = require '{TEST_LUA}'
        return function(btn)
            btn.onClick:AddListener(lib.onBtnClick)
        end
        ");
        LuaFunction fn = rets[0] as LuaFunction;
        fn.Action(btn);
        btn.onClick.Invoke();

        // 修改Lua脚本，模拟热更新
        WriteLuaCode(@"
local M = {}
local n = 1
function M:onBtnClick()
    print('onBtnClick new', n)
    n = n + 1
end
return M");
        // 执行热更新
        Hotfix(TEST_LUA);
        btn.onClick.Invoke();
    }
```


<b>原文:<br>
<https://lizijie.github.io/2025/12/21/%E6%90%AD%E5%BB%BAloki%E6%97%A5%E5%BF%97%E8%81%9A%E5%90%88.html>
<br>

作者github:<br>
<https://github.com/lizijie>
</b>