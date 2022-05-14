---
title: Maven多模块统一管理版本
date: 2022-05-14 13:27:00
Tags:maven
---

之前写多模块项目的时候，一般都需要保证各模块的版本号是一致的，所以导致每次代码有变动需要升级版本的时候，都需要全局挨个替换，但是这种方式太麻烦，也不够优雅，由于最近工作是维护公司的apollo，所以我看了下apollo的pom文件里的方式，觉得还不错。

首先是要在多模块项目里的父pom上，做如下配置

```xml
<groupId>com.example</groupId>
<artifactId>demo-mulitmodule-release-plugin</artifactId>
<version>${reversion}</version>
<packaging>pom</packaging>
<properties>
	<reversion>0.0.1-SNAPSHOT</reversion>
</properties>
```

为了精简，以上我只贴了相关的配置，`packaging` 是写死的，必须是`pom`,version要通过properties设置属性引用的方式，这样方便在后续子pom里通过同样的方式去引用，这样后续版本变化只改这里就可以了。`groupId`和`artifactId` 写自己的就行，这里只是为了合后面子pom的配置保持上下文一致。

然后子pom的相关配置如下:

```xml
<parent>
	<groupId>com.example</groupId>
	<artifactId>demo-mulitmodule-release-plugin</artifactId>
	<version>${reversion}</version>
	<relativePath>../pom.xml</relativePath>
</parent>
```

上面配置就是明确子pom继承了前面的父pom，并通过`${reversion}`实现父pom版本号的引用和`relativePath`的设置来定位父pom文件的位置。
