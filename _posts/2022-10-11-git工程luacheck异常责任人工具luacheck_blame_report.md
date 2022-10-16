---
layout: post
title: git工程luacheck异常责任人工具luacheck_blame_report
key: 202210112204
tags: git luacheck
---



luacheck是一个很不错的lua静态代码分析工具，它输出异常信息都值得去关注。[luacheck_blame_report](https://github.com/lizijie/luacheck_blame_report)创作的本意，是希望在持续化测试过程，能得到更多luacheck输出的异常有关联的信息，比如异常对应行代码的作者，这样可以更好在持续化测试中自动化及更快地反馈给代码作者

[luacheck_blame_report](https://github.com/lizijie/luacheck_blame_report)目前基于git仓库工程，整合luacheck与git blame筛选出与作者关联的异常数量以及具体的异常条目

# 用法
`sh check [需要检查的lua代码文件/目录] [luacheck配置文件(可选)]`

# 运行依赖
git、awk、wc、find

# 样例
[luacheck_blame_report](https://github.com/lizijie/luacheck_blame_report)子模块luacheck的异常报告
```
$ cd xxxx/luacheck_blame_report/luacheck
$ sh check.sh ./ ../..luacheckrc
--config ../.luacheckrc
Total: 123 warnings / 6 errors in 130 files
report file: /mnt/f/luacheck_blame_report/2022-10-16-09-51-55-lua_check_report
$ cat /mnt/f/luacheck_blame_report/2022-10-16-09-51-55-lua_check_report
author   number of warnings/errors
----------------------------
mpeterv:    69
NiteHawk:   8
Peter Melnichenko:  52
Total: 123 warnings / 6 errors in 130 files


mpeterv
    spec/configs/project/nested/ab.lua:1:7: accessing undefined variable a
    spec/configs/project/nested/ab.lua:1:10: accessing undefined variable b
    spec/configs/project/nested/nested/abc.lua:1:7: accessing undefined variable a
    spec/configs/project/nested/nested/abc.lua:1:10: accessing undefined variable b
    spec/configs/project/nested/nested/abc.lua:1:13: accessing undefined variable c
    spec/formatters/custom_formatter.lua:1:17: unused argument report
    spec/samples/argparse-0.2.0.lua:34:27: unused loop variable setter
    spec/samples/argparse-0.2.0.lua:117:27: unused argument self
    spec/samples/argparse-0.2.0.lua:125:27: unused argument self
    spec/samples/argparse-0.2.0.lua:523:1: line contains only whitespace
    spec/samples/argparse-0.2.0.lua:641:13: right side of assignment has less values than left side expects
    spec/samples/argparse-0.2.0.lua:660:121: line is too long (125 > 120)
    spec/samples/argparse-0.2.0.lua:933:121: line is too long (123 > 120)
    spec/samples/argparse-0.2.0.lua:942:7: accessing undefined variable _TEST
    spec/samples/argparse-0.2.0.lua:957:41: unused argument parser
    spec/samples/bad_code.lua:3:16: unused function helper
    spec/samples/bad_code.lua:3:23: unused variable length argument
    spec/samples/bad_code.lua:7:10: setting non-standard global variable embrace
    spec/samples/bad_code.lua:8:10: variable opt was previously defined as an argument on line 7
    spec/samples/bad_code.lua:9:11: accessing undefined variable hepler
    spec/samples/bad_flow.lua:1:28: empty if branch
    spec/samples/bad_flow.lua:6:4: empty do..end block
    spec/samples/bad_flow.lua:12:10: right side of assignment has less values than left side expects
    spec/samples/bad_flow.lua:16:10: right side of assignment has more values than left side expects
    spec/samples/bad_flow.lua:21:7: unreachable code
    spec/samples/bad_flow.lua:25:1: loop is executed at most once
    spec/samples/custom_std_inline_options.lua:5:1: invalid value of option 'std': unknown std 'other_std'
    spec/samples/custom_std_inline_options.lua:6:25: accessing undefined variable it
    spec/samples/defined.lua:4:4: accessing undefined variable baz
    spec/samples/global_inline_options.lua:6:10: unused global variable f
    spec/samples/global_inline_options.lua:7:4: setting non-standard global variable baz
    spec/samples/global_inline_options.lua:18:4: setting non-module global variable external
    spec/samples/inline_options.lua:6:16: unused function f
    spec/samples/inline_options.lua:15:1: accessing undefined variable baz
    spec/samples/inline_options.lua:22:10: unused variable g
    spec/samples/inline_options.lua:24:7: unused variable f
    spec/samples/inline_options.lua:24:10: unused variable g
    spec/samples/inline_options.lua:26:1: unpaired push directive
    spec/samples/inline_options.lua:28:4: unpaired pop directive
    spec/samples/inline_options.lua:34:1: empty do..end block
    spec/samples/inline_options.lua:35:10: empty if branch
    spec/samples/python_code.lua:1:6: expected '=' near '__future__'
    spec/samples/read_globals.lua:1:1: setting read-only global variable string
    spec/samples/read_globals.lua:2:1: setting undefined field append of global table
    spec/samples/read_globals.lua:6:1: mutating non-standard global variable baz
    spec/samples/read_globals.lua:6:21: accessing undefined variable baz
    spec/samples/read_globals_inline_options.lua:2:10: accessing undefined variable baz
    spec/samples/read_globals_inline_options.lua:3:11: setting non-standard global variable baz
    spec/samples/read_globals_inline_options.lua:3:16: mutating non-standard global variable baz
    spec/samples/redefined.lua:3:11: unused argument self
    spec/samples/redefined.lua:4:10: shadowing upvalue a on line 1
    spec/samples/redefined.lua:4:13: variable self is never set
    spec/samples/redefined.lua:4:13: variable self was previously defined as an argument on line 3
    spec/samples/redefined.lua:7:13: shadowing definition of variable a on line 4
    spec/samples/redefined.lua:8:7: accessing undefined variable each
    spec/samples/redefined.lua:8:32: shadowing upvalue self on line 4
    spec/samples/unused_code.lua:3:18: unused argument baz
    spec/samples/unused_code.lua:4:8: unused loop variable i
    spec/samples/unused_code.lua:5:13: unused variable q
    spec/samples/unused_code.lua:7:11: unused loop variable a
    spec/samples/unused_code.lua:7:14: unused loop variable b
    spec/samples/unused_code.lua:7:17: unused loop variable c
    spec/samples/unused_code.lua:13:7: value assigned to variable x is overwritten on line 14 before use
    spec/samples/unused_code.lua:14:1: value assigned to variable x is overwritten on line 15 before use
    spec/samples/unused_code.lua:21:7: variable z is never accessed
    spec/samples/unused_secondaries.lua:3:7: unused variable a
    spec/samples/unused_secondaries.lua:6:7: unused variable x
    spec/samples/unused_secondaries.lua:6:13: unused variable z
    spec/samples/unused_secondaries.lua:12:1: value assigned to variable o is unused


NiteHawk
    spec/samples/bad_whitespace.lua:4:26: line contains trailing whitespace
    spec/samples/bad_whitespace.lua:8:25: trailing whitespace in a comment
    spec/samples/bad_whitespace.lua:22:40: trailing whitespace in a comment
    spec/samples/bad_whitespace.lua:26:1: line contains only whitespace
    spec/samples/bad_whitespace.lua:27:1: line contains only whitespace
    spec/samples/bad_whitespace.lua:28:1: line contains only whitespace
    spec/samples/bad_whitespace.lua:29:1: line contains only whitespace
    spec/samples/bad_whitespace.lua:34:1: inconsistent indentation (SPACE followed by TAB)


Peter Melnichenko
    spec/projects/default_stds/nested/spec/sample_spec.lua:1:55: accessing undefined variable version
    spec/projects/default_stds/nested/spec/sample_spec.lua:1:64: accessing undefined variable read_globals
    spec/projects/default_stds/nested/spec/sample_spec.lua:2:1: accessing undefined variable ignored
    spec/projects/default_stds/normal_file.lua:1:1: accessing undefined variable it
    spec/projects/default_stds/normal_file.lua:1:45: accessing undefined variable version
    spec/projects/default_stds/normal_file.lua:1:54: accessing undefined variable read_globals
    spec/projects/default_stds/normal_file.lua:2:1: accessing undefined variable ignored
    spec/projects/default_stds/sample_spec.lua:1:44: accessing undefined variable version
    spec/projects/default_stds/sample_spec.lua:1:53: accessing undefined variable read_globals
    spec/projects/default_stds/test/nested_normal_file.lua:1:1: accessing undefined variable it
    spec/projects/default_stds/test/nested_normal_file.lua:1:63: accessing undefined variable version
    spec/projects/default_stds/test/nested_normal_file.lua:1:72: accessing undefined variable read_globals
    spec/projects/default_stds/test/sample_spec.lua:1:53: accessing undefined variable version
    spec/projects/default_stds/test/sample_spec.lua:1:62: accessing undefined variable read_globals
    spec/projects/default_stds/tests/nested/sample_spec.lua:1:60: accessing undefined variable version
    spec/projects/default_stds/tests/nested/sample_spec.lua:1:69: accessing undefined variable read_globals
    spec/projects/default_stds/tests/sample_spec.lua:1:33: accessing undefined variable version
    spec/projects/default_stds/tests/sample_spec.lua:1:42: accessing undefined variable read_globals
    spec/samples/bad_whitespace.lua:14:20: trailing whitespace in a string
    spec/samples/bad_whitespace.lua:17:30: trailing whitespace in a comment
    spec/samples/global_fields.lua:2:16: indirectly accessing undefined field upsert of global table
    spec/samples/global_fields.lua:8:1: indirectly setting undefined field insert.foo of global table
    spec/samples/global_fields.lua:23:7: accessing undefined field gfind of global string
    spec/samples/global_fields.lua:27:7: accessing undefined field find of global string
    spec/samples/global_fields.lua:32:7: accessing undefined variable server
    spec/samples/global_fields.lua:33:7: accessing undefined variable server
    spec/samples/global_fields.lua:34:1: mutating non-standard global variable server
    spec/samples/global_fields.lua:35:1: mutating non-standard global variable server
    spec/samples/global_fields.lua:36:1: mutating non-standard global variable server
    spec/samples/global_fields.lua:37:7: accessing undefined variable server
    spec/samples/global_fields.lua:38:1: mutating non-standard global variable server
    spec/samples/global_fields.lua:40:1: invalid value of option 'std': unknown std 'my_server'
    spec/samples/global_fields.lua:41:1: mutating non-standard global variable server
    spec/samples/global_fields.lua:42:1: mutating non-standard global variable server
    spec/samples/indirect_globals.lua:2:11: accessing undefined variable global
    spec/samples/indirect_globals.lua:5:1: indirectly setting undefined field concat.foo.bar of global table
    spec/samples/indirect_globals.lua:5:32: accessing undefined variable global
    spec/samples/line_length.lua:2:121: line is too long (123 > 120)
    spec/samples/line_length.lua:3:121: line is too long (164 > 120)
    spec/samples/line_length.lua:8:121: line is too long (134 > 120)
    spec/samples/line_length.lua:13:41: line is too long (47 > 40)
    spec/samples/line_length.lua:18:121: line is too long (132 > 120)
    spec/samples/line_length.lua:22:81: line is too long (85 > 80)
    spec/samples/line_length.lua:26:101: line is too long (104 > 100)
    spec/samples/line_length.lua:29:121: line is too long (125 > 120)
    spec/samples/reversed_fornum.lua:1:1: numeric for loop goes from #(expr) down to -1.5 but loop step is not negative
    spec/samples/utf8.lua:2:1: setting undefined field 분야 명 of global math
    spec/samples/utf8.lua:2:16: accessing undefined field 値 of global math
    spec/samples/utf8.lua:3:25: unused variable t
    spec/samples/utf8.lua:4:5: value assigned to field päällekkäinen nimi a\u{200B}b is overwritten on line 5 before use
    spec/samples/utf8_error.lua:2:11: expected statement near 'о'
    src/luacheck/unicode_printability_boundaries.lua:2:121: line is too long (7635 > 120)
```

<br> 
<br> 
<b>原文:<br>
<https://lizijie.github.io/2022/10/11/git%E5%B7%A5%E7%A8%8Bluacheck%E5%BC%82%E5%B8%B8%E8%B4%A3%E4%BB%BB%E4%BA%BA%E5%B7%A5%E5%85%B7luacheck_blame_report.html>
<br>
作者github:<br>
<https://github.com/lizijie>
</b>
