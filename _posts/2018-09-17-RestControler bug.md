---
layout:     post
title:      解决RestControler 无法返回 Iterable 类型的列表的问题
subtitle:   
date:       2018-09-17
author:     Jadedrip
header-img: 
catalog: true
tags:
    - Java
    - Spring
---
当使用 Spring 的 MVC 尝试返回 JSON 的过程中，碰到了一个奇怪的问题，我在 Spring boot 的程序里，@RestControler 的类，直接返回 Iterable 类型的对象，是 Ok，但在另一个存 Spring 的程序里就只返回了空值。

尝试更新版本等方法无效以后，通过调试终于发现了原因。Spring 注册 MessageConverter 的时候，会判断是否存在 jackson-databind, 如果不存在，会尝试使用 gson 来进行 json 的序列化，而 gson 貌似无法序列化 Iterable 类型的对象。

所以，解决方式就是在 Gradle 里，加入对 jackson 的引用：

    compile group: 'com.fasterxml.jackson.core', name: 'jackson-databind', version: '2.9.6'

PS：我准备去 gson 提个 issues , 看看能否搞定这个 bug.