---
layout: post
title: 修复toluapp在lua5.3的整数错误
key: 201803131144
tags: lua
---

长期以来，lua number即用来表示整数类型，也用来表示浮点类型。
最近将旧工程lua从5.1.4升级到5.3.4后。发现以下问题，lua_pushnumber压入整数，该值在lua里打印却是带有“点号”浮动数
```c
// c/c++ File
int val = 1;
lua_pushnumber(L, val);
lua_setglobal(L, "value");
```
```lua
-- Lua File
print(value)
--5.1.4打印"1"
--5.3.4打印"1.0"
```
出现这个问题，原因是[Lua 5.3](http://www.lua.org/manual/5.3/readme.html#changes)之后，官方已明确区分integer和number类型
>
Changes since Lua 5.2
Here are the main changes introduced in Lua 5.3. The reference manual lists the incompatibilities that had to be introduced.
>
Main changes
integers (64-bit by default)
official support for 32-bit numbers
bitwise operators
basic utf-8 support
functions for packing and unpacking values

以下比较Lua的3个版本 5.1.5、5.2.4和5.3.0。
5.1.5与5.2.4在实现pushnumber、pushinteger没太大差别，但可以明显看到5.3.0使用setfltvalue、setivalue来分别处理浮点数、整数
```c
// Lua 5.1.5
// File: lapi.c
LUA_API void lua_pushnumber (lua_State *L, lua_Number n) {
  lua_lock(L);
  setnvalue(L->top, n);
  api_incr_top(L);
  lua_unlock(L);
}

LUA_API void lua_pushinteger (lua_State *L, lua_Integer n) {
  lua_lock(L);
  setnvalue(L->top, cast_num(n));
  api_incr_top(L);
  lua_unlock(L);
}

#define setnvalue(obj,x) \
  { TValue *i_o=(obj); i_o->value.n=(x); i_o->tt=LUA_TNUMBER; }
```
```c
// lua 5.2.4
LUA_API void lua_pushnumber (lua_State *L, lua_Number n) {
  lua_lock(L);
  setnvalue(L->top, n);
  luai_checknum(L, L->top,
    luaG_runerror(L, "C API - attempt to push a signaling NaN"));
  api_incr_top(L);
  lua_unlock(L);
}


LUA_API void lua_pushinteger (lua_State *L, lua_Integer n) {
  lua_lock(L);
  setnvalue(L->top, cast_num(n));
  api_incr_top(L);
  lua_unlock(L);
}

#define setnvalue(obj,x) \
	{ TValue *io_=(obj); num_(io_)=(x); lua_assert(  (io_)); }
```
```c
// Lua 5.3.0
// File: lapi.c
LUA_API void lua_pushnumber (lua_State *L, lua_Number n) {
  lua_lock(L);
  setfltvalue(L->top, n);
  api_incr_top(L);
  lua_unlock(L);
}


LUA_API void lua_pushinteger (lua_State *L, lua_Integer n) {
  lua_lock(L);
  setivalue(L->top, n);
  api_incr_top(L);
  lua_unlock(L);
}

#define setfltvalue(obj,x) \
  { TValue *io=(obj); val_(io).n=(x); settt_(io, LUA_TNUMFLT); }


#define setivalue(obj,x) \
  { TValue *io=(obj); val_(io).i=(x); settt_(io, LUA_TNUMINT); }
```

lua 5.3.x相比以前版本有较大差别，具体详细变化这里贴云风的文章 [从 Lua 5.2 迁移到 5.3](https://blog.codingnow.com/2015/01/lua_52_53.html)。
另外，本人修改[toluapp](https://github.com/lizijie/toluapp)，使其从pkg生成cpp时，使用lua_pushinteger压入整数


<br>
<br>
<b>原文:<br>
<https://lizijie.github.io/2018/03/13/%E4%BF%AE%E5%A4%8Dtoluapp%E5%9C%A8lua5.3%E7%9A%84%E6%95%B4%E6%95%B0%E9%94%99%E8%AF%AF.html>
<br>
作者github:<br>
<https://github.com/lizijie>
</b>
