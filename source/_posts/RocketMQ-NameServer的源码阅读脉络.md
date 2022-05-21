---
title: RocketMQ-NameServer的源码阅读脉络
date: 2022-05-15 12:00:02
tags: RocketMQ
---

## 启动大概流程

1. `NamesrvStartup` 是启动类，会先调`createNamesrvController` 初始化配置，然后创建`NamesrvController`, 然后调Controller的静态start方法
2. `NamesrvController`的静态Start方法会先调`initialize`方法，然后注册程序destory的钩子，最后start方法启动
   1. 在`initialize` 方法里会执行`registerProcessor`方法，会在这里注册一个`NettyRequestProcessor` 这个是1个接口，表示的是RocketMQ请求处理，在NameServer里，有2个实现，`ClusterTestRequestProcessor` 和 `DefaultRequestProcessor`其中我们主要关键的是`DefaultRequestProcessor` 。 

## 处理请求路由

`DefaultRequestProcessor`是NameServer里我觉得最核心的类，因为它是NameServer里负责接收并处理请求的类，通过它的`processRequest`方法，可以很直接的看到NameServer都具体负责处理哪些请求，以及这些请求具体的实现，也都可以找的到。

## 核心类介绍

1. `KVConfigManager` 负责管理不同Namespace下KV的配置，并存储。
2. `RouteInfoManager` 负责管理路由相关信息
3. `NamesrvController` 是1个集中的初始化整个系统的类，并且持有其他核心类的依赖，其他类核心类都持有`NamesrvController`的引用，要跟另外个类调用，都是通过`NamesrvController`来获取这个类的引用，相当于1个解耦的作用。
