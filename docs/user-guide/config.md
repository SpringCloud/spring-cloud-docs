# Spring Cloud Config

## Quick Start

#### Client Side Usage

## Spring Cloud Config Server

#### Environment Repository

#### Health Indicator

#### Security

#### Encryption and Decryption

#### Key Management

#### Creating a Key Store for Testing

#### Using Multiple Keys and Key Rotation

#### Serving Encrypted Properties

## Serving Alternative Formats

## Serving Plain Text

## Embedding the Config Server 

## Push Notifications and Spring Cloud Bus

许多源代码库提供者（如GitHub，gitlab或bitbucket）会通过webhook通知您在代码库的变化。你可以通过供应商的用户界面配置你的webhook，一个URL和一系列你感兴趣的事件。比如Github会通过POST请求webhook，body中包含提交的一些信息，请求头“X-Github-Event”等于“push”。如果你添加了依赖`spring-cloud-config-monitor`并在你的Config Server中激活了Spring Cloud Bus，一个"/monitor" endpoint会被启用。

当webhook被触发，Config Server认为发生了改变，并将针对应用程序发送一个`RefreshRemoteApplicationEvent`。变化检测可以考虑，但默认情况下它只是在寻找application name匹配的文件的变化（例如“foo.properties”是针对“foo”应用，“application.properties”是针对所有应用）。该策略，如果你想重写的行为是`PropertyPathNotificationExtractor`，这个类接受请求头和请求体作为参数，返回一个路径改变的列表。

默认配置基于Github，Gitlab或Bitbucket。除了来自Github，Gitlab或Bitbucket的JSON通知，您可以触发一个更改通知，向/monitor发起POST请求，参数是`path={name}`，这将广播匹配“{name}”的应用（可以包含通配符）

NOTE
> 当`spring-cloud-bus`在Config Server的服务端和客户端被激活时，`RefreshRemoteApplicationEvent`才会被传播

NOTE
> 默认的配置也检测到本地的Git仓库的文件系统的变化（这种情况下不适用webhook，只要你编辑一个配置文件的更新就会被发送）

## Spring Cloud Config Client

#### Config First Bootstrap

#### Discovery First Bootstrap

#### Config Client Fail Fast

#### Config Client Retry

#### Locating Remote Configuration Resources

#### Security






