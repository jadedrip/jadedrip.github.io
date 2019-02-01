---
title: '解决 Hibernate 连接 Oracle 导致的 ORA-02289: sequence does not exist'
layout: post
subtitle: ''
tags:
  - spring boot
  - hibernate
  - oracle
category: java
---
用 Spring boot 连接 Oracle 的时候，尝试插入数据的时候提示了 ORA-02289: sequence does not exist 。

这是由于 Spring 使用了 Hibernate 来连接数据库，而 Oracle 的自增字段和表是独立的，因此你必须在明确的给表指明 sequence，或者创建一个名为 hibernate_sequence 的默认 sequence。