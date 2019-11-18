---
title: Java日志工具笔记
date: 2019-11-18 15:43:51
tags:
categories: Java
---

## Slf4j

需求：打印error时，既想打印带参数的字符串，也想打印异常堆栈信息

<!--more-->

Answer：只需调用log.error格式化字符串，并且把堆栈信息参数e放到最后一个即可

```java
@Slf4j
public class LogTest {
    @Test
    public void testLog() {
        String s = "Hello world";
        try {
            Integer i = Integer.valueOf(s);
        } catch (NumberFormatException e) {
            // 异常堆栈必须是最后一个参数
            log.error("Failed to format {}", s, e);
            // log.error("Failed to format {}", e, s);
        }
    }
}
```

Why?

跟随debug的脚步，我们会来到下面这个方法内部：

![image-20191118155221581](https://tva1.sinaimg.cn/large/006y8mN6ly1g928uf5pzoj319w0iwn6i.jpg)

显而易见，当判断传入的格式化参数数组的最后一个参数继承自Throwable类时，就会将其识别出堆栈信息供打印日志