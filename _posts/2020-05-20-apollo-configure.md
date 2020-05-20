---
title: 'Apollo 完全集成指南'
layout: post
subtitle: 'v1.6.1'
tags:
  - apollo
category: 
---

## 服务端配置

### apollo-configservice 

主服务（meta, eureka等)，每个环境至少一台，现网环境部署多台防止单点故障，通过 nginx 中转来均衡负载。

* 修改 config/application-github.properties 配置数据库，指定 ApolloConfigDB 所在地址

### apollo-portal

管理平台前台，多环境如果都能访问只需要一台

* 修改 `config/apollo-env.properties` 配置文件，为不同环境指定不同的 eureka (就是 configservice) 服务发现
* 修改 `config/application-github.properties` 配置数据库，指定为 ApolloPortalDB 所在地址
* 修改数据库 ApolloPortalDB.ServerConfig 表（或者通过管理平台 ➝ 管理员工具 ➝ 系统参数），配置可支持环境列表：apollo.portal.envs 

### apollo-adminservice

管理平台后台，每个环境都需要至少一台

* 修改 `config/application-github.properties` 配置数据库，指定为 `ApolloConfigDB 所在地址`

* 修改数据库 `ApolloConfigDB.ServerConfig` 表，eureka.service.url 配置连接到哪个 eureka （就是 configservice)

### 三个服务都需要的配置

* 如果同一个机器部署多台同名服务，修改 config/app.properties 配置指定不同 appId

* 修改 config/application-github.properties 配置数据库及连接配置，注意设置连接池大小，否则会占满数据库链接

      spring.datasource.initialSize=5
      spring.datasource.minIdle=5
      spring.datasource.maxActive=20
      spring.datasource.maxWait=30000
  
* 修改 `scripts/startup.sh` 文件，修改端口以及 log 文件所在目录

* 多网卡情况下，在 config 下创建 bootstrap.properties 文件（注意，不能是 application.properties，载入顺序不同），排除虚拟网卡等

      spring.cloud.inetutils.ignored-interfaces=lo.*

  
##  客户端配置

1. 引入 Maven（客户端代码无变动，所以没 1.6.1 版本）

   ```
   <dependency>
   	<groupId>com.ctrip.framework.apollo</groupId>
   	<artifactId>apollo-client</artifactId>
   	<version>1.6.0</version>
   </dependency>
   ```

2. 修改配置文件 application.yml

   1. 配置属性 app.id ，和你在管理平台创建的项目id一致
   2. 配置属性 apollo.bootstrap.enabled: true ，bootstrap阶段注入默认application namespace的配置，这样 spring 的配置也能写入 apollo 了

3. 为不同环境配置 meta (configserver) 服务器 `apollo.meta: http://xxxx:8080` (推荐写在 application-{profile}.yml 里)

4. **可选**：配置环境 (只要客户端指定了 meta, 不配置貌似也没事，只是日志里有找不到环境的提示)

   apollo 可以通过服务器环境变量 ENV，虚拟机变量 env 和配置文件里的 env 来设定环境（我用的 服务器环境变量 ENV，但注意启动脚本里别把它给覆盖了）

