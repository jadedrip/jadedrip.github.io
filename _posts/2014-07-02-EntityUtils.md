---
layout:     post
title:      踩坑记：httpComponents 的 EntityUtils
subtitle:   
date:       2014-07-02
author:     Jadedrip
header-img: 
catalog: true
tags:
    - Java
---
今天写的一个服务程序，有人报告获得的数据中文乱码，而我是用 apache 通过 httpComponents 去取得数据的，于是开启日志的 debug 级别。

在日志里果然发现中文不见了，有乱码出现：

    2014-07-02 16:35:01.348 DEBUG [Wire.java:86] http-outgoing-8 << "<?xml version="1.0" encoding="UTF-8"?>... subject="[0xe6][0x88][0x91][0xe6][0x98][0xaf][0xe4][0xb8][0xad][0xe6][0x96][0x87][0xe4][0xb8][0xbb][0xe9][0xa2][0x98]" ...

我发出的报文怎么会乱码？明明我设置了 utf-8 编码的啊！

其实，这是第一个坑：httpComponents 打日志的时候，把中文转成了这种格式。其实是对的！
可怜的我在这个坑里转了好久才发现啊！

最后找了半天，通过抓包才终于发现，发送、接收到的中文报文都没问题，但是我解出来的中文乱码了。折腾半天后才发现，远程服务器返回时，没返回编码，而我获取包体的代码是用的 EntityUtils :

    CloseableHttpResponse httpResponse = httpClient.execute(get);
    HttpEntity httpResponseEntity = httpResponse.getEntity();
    String s = EntityUtils.toString(httpResponseEntity);

似乎没问题啊！但是，这就是个大坑了，**httpComponents 的默认代码并非 utf-8**！
于是这个 s 就乱了……
正确的写法其实是 

    EntityUtils.toString(httpResponseEntity, "utf-8");
   
顺便说一下，以前我、还有我同事都踩过的一个坑。

    EntityUtils.toString(httpResponseEntity, "utf-8");

这行代码在 http 请求时必须调用！或者说，返回的包体流必须被读完。即使返回的不是 200 OK！
以前由于对返回的包体内容不关心，所以没调。然后第一个请求可以成功，而第二个请求就卡住……
更坑的是 200 OK的时候读包体流，而错误的时候直接抛异常或者返回了。然后程序工作看起来正常，但时不时的卡啊卡……

