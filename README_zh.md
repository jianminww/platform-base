# Platform Base Service 介绍

- [简介](#简介)
- [系统架构](#系统架构)
    - [傲空间平台](#傲空间平台)
    - [Base Service](#base-service)
- [环境变量](#环境变量)
- [构建和运行应用程序](#构建和运行应用程序)
- [使用 OpenAPI 和 Swagger UI](#使用-openapi-和-swagger-ui)
- [演进计划](#演进计划)
- [贡献指南](#贡献指南)
- [English Documents](/README.md)

## 简介

AO.space（傲空间）是一个以保护个人数据安全和隐私为核心的解决方案。通过端对端加密、基于设备认证等技术，确保用户完全掌控个人账号和数据。同时，采用平台透明转发、点对点加速、局域网直连等技术，让用户随时随地的极速访问个人数据。傲空间利用 PWA（Progressive Web App）和云原生技术，设计并打造前后端一体的应用生态。

AO.space（傲空间）由服务端、客户端、平台三个部分组成。服务端和客户端只运行在个人设备上，使用公钥认证建立加密通信通道。服务端是傲空间管理保护用户数据的核心部分，目前支持 x86_64 和 aarch64 两个架构，可运行在个人服务器、个人计算机等设备上。客户端让用户在不同平台上快速安全的访问个人数据，目前支持多个平台，包括 Android、iOS 和 Web ，方便用户随时随地使用。

AO.space Platform（即傲空间平台），则为个人设备提供透明通信通道服务和互联网访问的安全防护，并且可以进行私有部署。与其他解决方案中的平台不同，傲空间下个人账号的认证和鉴权只由运行在个人设备的服务端管理，傲空间平台无法管理和解析个人的任何数据，实现用户的个人数据完全由用户掌控在个人设备上。

## 系统架构

![傲空间平台&Platform Base架构.png](docs/zh/asserts/傲空间平台&Platform%20Base架构.png)

### 傲空间平台

傲空间平台的职责是为个人设备建立透明的通信信道。包含基础服务（Platform Base Service）、转发代理服务（Plarform Proxy Service）、中继转发服务器（Network Transit Server）。

- 基础服务（Platform Base Service）：为傲空间设备提供注册服务，以及协调和管理平台网络资源（域名，Network Server通信信道等）。
- 转发代理服务（Plarform Proxy）：为傲空间用户域名流量提供高可用转发和横向扩容的支持。
- 中继转发服务器（Network Transit Server）提供通过中继转发的方式穿透 NAT 访问设备的网络支持服务。将来自 Clients 的流量转发至傲空间设备。

> **_注意：_** 完整的傲空间平台部署指南，请参阅 [AOPlatform社区部署指南](https://ao.space/open/documentation/104002) 。

### Base Service

Base Service 是傲空间平台管理面的实现，主要提供以下功能：

1. 认证傲空间设备身份
2. 提供傲空间设备、用户、客户端注册功能
3. 协调和管理平台网络资源（域名，Network Transit Server通信信道等）
4. 傲空间平台切换

> **_注意：_** 项目使用了 Quarkus，它是一个 Red Hat 公司开源的云原生 Java 框架。如果您想了解有关Quarkus的更多信息，请访问其网站：[QUARKUS](https://quarkus.io/) 。

## 环境变量

所有应用程序配置都可以通过配置文件 “application.yml“ 进行设置，有关如何配置它们的详细信息，请参阅 [配置参考指南](https://quarkus.io/guides/config-reference )。以下是在容器启动期间可以更改的重要环境变量。

### 数据源
- QUARKUS_DATASOURCE_DB_KIND：用于设置数据库类型。默认设置：`mysql`
- QUARKUS_DATASOURCE_USERNAME：用于设置数据库的用户名。
- QUARKUS_DATASOURCE_PASSWORD：用于设置数据库的密码。
- QUARKUS_DATASOURCE_JDBC_URL：用于设置数据库的 jdbc url。默认设置：`jdbc:mysql://127.0.0.1:3306/community`

### 缓存
- QUARKUS_REDIS_HOSTS：用于设置 redis 的连接url。默认设置：`redis://localhost:6379`
- QUARKUS_REDIS_PASSWORD：用于设置 redis 的密码。

### 应用程序
- APP_REGISTRY_SUBDOMAIN：用于设置傲空间设备的“根域名”，也是傲空间用户域名的一部分。您需要在 DNS 和 Nignx 上配置根域名，请参阅 [AOPlatform社区部署指南](https://ao.space/open/documentation/104002)

有关配置名称和环境变量名称之间的命名转换规则，请参阅 [转换规则](https://github.com/eclipse/microprofile-config/blob/master/spec/src/main/asciidoc/configsources.asciidoc#default-configsources )。以下是来自 “application.yml” 的上述变量的所有默认值：

```默认配置
quarkus:
  datasource:
    db-kind: mysql
    username: root
    password: 123456
    jdbc:
      url: jdbc:mysql://127.0.0.1:3306/community?allowPublicKeyRetrieval=true&useSSL=false&serverTimezone=GMT%2B8
  redis:
    hosts: redis://localhost:6379
    password: 123456
app:
  registry:
    subdomain: XXXX # 傲空设备的根域名
```

## 构建和运行应用程序

### 手动构建和运行 jvm docker 镜像

1. `./mvnw package`
2. `cd /eulixplatform-registry`
3. `docker build --pull -f src/main/docker/Dockerfile.jvm -t platform-base-jvm-community:latest .`
4. `docker run -itd --name platform-base -p 8080:8080 -u root -e APP_REGISTRY_SUBDOMAIN="傲空间设备的根域名" platform-base-jvm-community:latest`

### 在开发模式下运行应用程序

您可以在开发模式下运行应用程序，该模式支持实时编码，使用：

```mvnw命令
./mvnw compile quarkus:dev
```

> **_注意：_** Quarkus现在附带一个开发UI，该UI仅在开发模式下可用：`http://localhost:8080/q/dev/`

### 打包并运行应用程序

应用程序可以使用以下方式打包：

```mvnw命令
./mvnw package
```

它会在 “target/quarkus-app/” 目录中生成 “quarkus-run.jar” 文件。 请注意，它不是一个 _über-jar_ ，因为依赖项被复制到“target/quarkus-app/lib/”目录中。 如果要构建 _über-jar_ ，请执行以下命令：

```mvnw命令
./mvnw package -Dquarkus.package.type=uber-jar
```

该应用程序现在可以使用如下命令运行：

```java命令
java -jar target/quarkus-app/quarkus-run.jar
```

## 使用 OpenAPI 和 Swagger UI

OpenAPI 描述符和 Swagger UI 前端来测试 REST 端点，访问地址：`http://localhost:8080/platform/q/swagger-ui/`

有关OpenAPI扩展的更多详细信息，请参阅 [使用OpenAPI和Swagger UI](https://quarkus.io/guides/openapi-swaggerui)

## 演进计划

- 提供局域网 IP 直连域名解析服务
- 转发代理服务（Platform Proxy Service）
- 平台侧基础服务的 Java 语言 SDK
- 平台侧基础服务的 golang 语言 SDK
- 基于 Mysql、Redis 等常用中间件的分布式锁

## 贡献指南

我们非常欢迎对本项目进行贡献。以下是一些指导原则和建议，希望能够帮助您参与到项目中来。

### 贡献代码

如果您想要为项目做出贡献，最好的方式就是提交代码。在提交代码之前，请确保您已经下载并熟悉了项目代码库，并且您的代码遵循了以下指导原则：

- 代码应当尽量简洁明了，并且易于维护和扩展。
- 代码应遵循项目约定的命名规范，以确保代码的一致性。
- 代码应当遵循项目的代码风格指南，可以参考项目代码库中已有的代码。

如果您想向项目提交代码，可以按照以下步骤进行：

- 在 GitHub 上 fork 该项目。
- 克隆您 fork 的项目到本地。
- 在本地进行您的修改和改进。
- 执行测试确保任何更改都无影响。
- 提交您的更改并新建一个 pull request。

### 代码质量

我们非常注重代码的质量，因此您提交的代码应当符合以下要求：

- 代码应当经过充分的测试，确保其正确性和稳定性。
- 代码应当遵循良好的设计原则和最佳实践。
- 代码应当尽可能地符合您所提交代码贡献的相关要求。

### 提交信息

在提交代码之前，请确保您提供了有意义而且详细的提交信息。这有助于我们更好地理解您的代码贡献并且更快速地合并它。

提交信息应当包含以下内容：

- 描述本次代码贡献的目的或者原因。
- 描述本次代码贡献的内容或者变化。
- （可选）描述本次代码贡献的测试方法或者结果。

提交信息应当清晰明了，并且符合项目代码库的提交信息约定。

### 问题报告

如果您在项目中遇到了问题，或者发现了错误，欢迎向我们提交问题报告。在提交问题报告之前，请确保您已经对问题进行了充分的调查和试验，并且尽量提供以下信息：

- 描述问题的现象和表现。
- 描述问题出现的场景和条件。
- 描述上下文信息或任何相关的背景信息。
- 描述您期望的行为信息。
- （可选）提供相关的截图或者报错信息。

问题报告应当清晰明了，并且符合项目代码库的问题报告约定。

### 功能请求

如果您想要向项目中添加新的功能或者特性，欢迎向我们提交功能请求。在提交功能请求之前，请确保您已经了解了项目的历史和现状，并且尽量提供以下信息：

- 描述您想要添加的功能或者特性。
- 描述这个功能或者特性的用途和目的。
- （可选）提供相关的实现思路或者建议。

功能请求应当清晰明了，并且符合项目代码库的功能请求约定。

### 感谢您的贡献

最后，感谢您对本项目的贡献。我们欢迎各种形式的贡献，包括但不限于代码贡献、问题报告、功能请求、文档编写等。我们相信在您的帮助下，本项目会变得更加完善和强大。
