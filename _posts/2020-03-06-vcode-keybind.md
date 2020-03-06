---
title: 'Visual Code 合并重构和快速修复快捷键'
layout: post
subtitle: ''
tags:
  - vcode
category: 我
---

我原先使用 Intellij IDEA 来进行 Java 开发，而最近为了支持正版，改用 Visual Code 了（不使用盗版当然就是支持正版了，对吧！）。

安装了微软专门给 Java 配的组合包，用起来还算顺手，不过有一个小问题一直困扰我。原来 Intellij 里已经习惯了通过 Alt-Enter 组合键来修复错误，或者给添加本地变量，可 vcode 里是两个分开的命令——一个是快速修复，一个是重构，如果两个命令都绑定 Alt-Enter，那么只有重构会跳出来。

仔细研究了按键绑定里的 when 字段，发现了一个变量 hasWordHighlights ，如果有错误的地方，会有下划线，这个变量就是 true，给两个命令都绑定了 Alt-Enter，然后when一个配置 true，一个配置 false，试验了一下，完美！

这样就能用同一个快捷键，智能判断到底需要显示修复菜单还是重构菜单了，又能用原来的习惯来工作了，哈，我聪明。

具体来说：

**重构**的 When 配置

`editorHasCodeActionsProvider && editorTextFocus && !editorReadonly && !hasWordHighlights`

**快速修复**的 When 配置

`editorHasCodeActionsProvider && editorTextFocus && !editorReadonly && hasWordHighlights`

