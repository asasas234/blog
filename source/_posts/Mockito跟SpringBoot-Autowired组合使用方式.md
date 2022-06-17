---
title: Mockito跟SpringBoot@Autowired组合使用方式
date: 2022-06-04 14:10:53
tags:
  - mockito
  - SpringBoot
---

有的时候，我们想绝大多数Bean在单测的时候，还是希望走正常的Bean注入,只有少部分Bean需要自己人工Mock结果，这个时候如何优雅的Mock Bean实例并自动和其他Bean实例一样进行注入呢？

可以通过Mockito的@MockBean注解解决，代码如下

```java
@SpringBootTest
class CubeDataCacheServiceTest {

    @Autowired
    private CubeDataCacheService cubeDataCacheService;

    @MockBean
    private DimensionLocalService dimensionLocalService;
}
```

以上代码把一些不相关的代码都去掉了，接着我们就可以正常的调mockito的API，进行正常的设置mock结果就好了。
