---
layout:     post
title:      少写了 @ResponseBody 引发的古怪问题
subtitle:   
date:       2014-06-11
author:     Jadedrip
header-img: 
catalog: true
tags:
    - Java
    - spring
---
最近在 Tomcat 里写一个异步的 HTTP 服务端，出于方便的考虑使用了 Spring MVC 注解来搞定，然后就发生了诡异的结果。
代码如下：

```java
    @RequestMapping("/test")
    public DeferredResult<String> test(
            final @RequestParam(required = true) String uid,
            final @RequestBody String s
    ) 
```

HTTP 请求可以进入我定义的方法，并且异步也工作正常，在 DeferredResult<String> 对象被 setResult 时返回，没有任何异常出现，但是！
客户端收到 404，并且内容里包含了我返回的字符串。

折腾一个下午以后，未果，第二天才偶尔发现，原来我忘记写 @ResponseBody 了，正确的定义应该是这样的：

```java
    @RequestMapping("/test")
    public @ResponseBody DeferredResult<String> test(
            final @RequestParam(required = true) String uid,
            final @RequestBody String s
    ) 
```

以这篇记录，纪念我浪费的又一天。


