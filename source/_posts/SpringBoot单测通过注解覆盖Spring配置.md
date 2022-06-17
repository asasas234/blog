---
title: SpringBoot单测通过注解覆盖Spring配置
date: 2022-06-05 14:58:09
tags:
  - Spring
  - SpringBoot
---

在我们写SpringBoot的单测的时候,有时候可能需要指定特殊的配置,用来覆盖原有的配置，以前的做法通常是2种

1. 在IDEA的RUN/DEBUG Configurations 弹窗上设置-D启动参数
2. 在单测上人工写1个`static` 代码块，然后里面写System.setProperty

以上第一种方式，如果我点的是IDEA左侧的启动按钮，会发现自动创建1个新的RUN/DEBUG 配置,以前设置的配置就不起左右了， 还需要人工修改太麻烦。

第二种方式又感觉不够优雅。

昨天我在写单测的时候，又网上搜了下，发现原来SpringBoot本身为我们提供了1个叫`@TestPropertySource` 的注解,具体使用实例如下

```java
@SpringBootTest
@TestPropertySource(properties = {
        "cube.cache.cacheFileDir=/tmp/memory-financial-model-server",
        "seepln.datasource.select=default",
        "datasource.mysql.password.encrypt=false",
        "spring.datasource.hikari.jdbc-url=jdbc:mysql://localhost:3306/default_fxfwtc?autoReconnect=true&useUnicode=true&allowMultiQueries=true&characterEncoding=utf8",
        "spring.datasource.hikari.username=root",
        "spring.datasource.hikari.password=xxx"
})
@ActiveProfiles("test")
class CubeDataCacheServiceTest {
}
```

只要像上面那样声明在测试类的上面,并按照这个格式去书写配置的KV项就好了。
