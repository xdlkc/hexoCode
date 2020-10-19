---
title: idea常用配置
date: 2019-11-13 21:46:29
tags:
categories: idea
---

idea屏蔽不常用的文件：

```
*.hprof;*.pyc;*.pyo;*.rbc;*.yarb;*~;CVS;__pycache__;_svn;vssver.scc;vssver2.scc;*.git;*.idea;.classpath;.project;.factorypath;.settings;.DS_Store;
```

<!--more-->

Idea java文件头：

```java
/**
 * 描述用途
 * <p>
 * </p>
 * DATE ${DATE}.
 * @author lkc.
 */
```



## 常用插件

- Alibaba Java Coding Guidelines：阿里巴巴出品的java代码风格检测插件
- Bash Support：bash相关支持插件
- Git Commit Template：git commit信息填写模板，用于规范化git提交内容
- GitLab Projects：GitLab项目插件，主要用来提MR等
- JProfiler：与JProfile本身结合的插件
- Lombok
- Maven Helper：能够检测pom文件中包含的重复依赖问题
- Sonar Lint：Sonar检测工具，能够与公司内的sonar平台结合使用，在本地就可以执行sonar检测
- YAML/Ansible Support