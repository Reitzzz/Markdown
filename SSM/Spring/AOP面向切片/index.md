# Spring AOP 面向切面编程学习目录

## 传统配置方式 (开发中已淘汰，了解AOP概念即可)
* [XML配置AOP](XML配置AOP.md)
* [接口实现AOP](接口实现AOP.md)

## 现代注解开发 (核心重点，实际项目开发绝对主流)
* [使用注解实现AOP](注解实现AOP.md)
    * [1.1 开启AOP注解支持与Bean注册](注解实现AOP.md#11-开启aop注解支持与bean注册)
    * [1.2 前置通知 (@Before)](注解实现AOP.md#12-前置通知-before)
    * [1.3 获取切入点信息 (JoinPoint)](注解实现AOP.md#13-获取切入点信息-joinpoint)
    * [1.4 命名绑定模式获取参数](注解实现AOP.md#14-命名绑定模式获取参数)
    * [1.5 其他通知注解 (@AfterReturning 等)](注解实现AOP.md#15-其他通知注解-afterreturning-等)
    * [1.6 环绕通知 (@Around)](注解实现AOP.md#16-环绕通知-around)
    * [1.7 总结](注解实现AOP.md#17-总结)