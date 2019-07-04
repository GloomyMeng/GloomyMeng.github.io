---
title: Router 机制的一些理解(二)
date: 2019-07-04 13:53:19
tags: 
    - Swift
    - Objc
categories: iOS
keywords: 
    - Protocol Router 
    - Swift
description: 基于协议的移动端Router方案
---

上一篇文章主要就自己的想法和思路进行简单的梳理。这一篇文章将会给出一定的代码来实现。

首先给出一个现在公司使用的Router的代码例

通过统一的注册入口，并且这里可以支持服务端下发来实现动态跳转
```
+ (NSDictionary *)defaultRegisteModules {
             @"A"        : @"AController",
             @"B"        : @"BController",
             @"C"        : @"CController",
             @"D"        : @"DController",
             /// ....
    }
}
```

调用模块则使用：

```
[ModuleManager.manager  handleOpenModelWithString:@"app://page?jsonData={\"type\":\"A\",\"data\":{}}"];
```

现阶段或者未来会遇到的痛点个人整理如下：

- 复杂参数的传递困难，需要 model(调用方) -> JSON(URL) -> model(被调用方)，且难以维护，可以想像被调用方需要写胶水代码去判断是否可以完成调用。
- 更多是基于Controller 的页面路由，通用模块、小组件的部分还是没有实现解耦合。
- URL的硬编码以及维护。A模块依赖B模块，硬编码或者服务端下发URL的方式使得掩盖了模块的依赖关系。
- 路由流程的管理缺失，从 发起Router跳转 -> 找到被调用方 -> 调用前准备 -> 开始调用 -> 完成调用，整个链路的逻辑缺失，现阶段更多只是在由被调用方决定YES or NO。
- 调用方和被调用方基于 URL未实现解耦合，参数变化后两方均需要修改代码。

---

所以综上所述可以简单得出基于协议的需要解答的痛点，下面是画出的实现架构图（因为自己的blog 没有配置好CND，所以原谅我只能用代码的方式来实现）

```
/**
                    Required Module
                            |
                            |   (required interface)
                            |
    ----------------------------------------------------
    |                                                   |
    |      get router                                   |
    |      create distance by router                    |
    |      prepare distance by router setting           |   (Router Container)
    |      do perform distance action                   |
    |      ......                                       |
    |      prepare distance remove if need              |
    |      do remove distance action                    |
    |                                                   |
    ----------------------------------------------------
                            |
                            |   (provider interface)
                            |
                    Provider Module
 */
```

---

依赖于 Router的中间层来实现调用方和被调用方的解耦。

Provider Module 对外提供暴露 provider interface。 而调用方则是通过 required interface 进行调用。
这样来解决上面列出的痛点。

- 针对复杂参数，interface 本身是 protocol，所以很方便的支持各种类型传参，同时 还提供了编译器的类型检查。
- 支持 controller 和 service 的路由，即被调用方是什么并无关系。
- Router 中间层可以针对整个链路进行管理控制。
- 分类出 required interface 和 provider interface 是为了方便复用以及业务升级， 来实现多个required 对应同一个 privoder 或者多个 provider 对应同一个 required。以实现一方发生变化另一方无感。
- 针对已有的代码只需要增加对外的协议层即可，不需要改动原有代码。
- 规范化 interface 必然带来着需要更规范业务数据model 等一些额外内容，以便更好的复用。

PS： 下一篇文章我将详细描述代码设计逻辑 


