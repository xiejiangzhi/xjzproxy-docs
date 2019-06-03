---
title: XJZProxy
description: 文档即接口，使用文档开发、测试接口。
---

## 功能

* 本地 API Mock 基于 YAML 文档，支持动态数据生成
* 自动生成、导出 API 文档
* 自动基于文档对比 API 请求响应数据格式，提前发现有问题的接口
* 根据请求历史生成统计数据
* HTTP/HTTPS/HTTP2/GRPC 代理


## 工作流

![workflow](/imgs/workflow.png)


## 介绍

让文档告诉你接口请求的参数，服务器返回的数据是否与文档中的定义一致。拿到文档不用等接口开发好就有可以调用。

在创建接口文档后，使用者可以直接通过代理访问所定义的接口。
如果你使用 `GRPC` 的话，甚至都不需要去一个个定义接口文档。
只需要指定 protobuf 在位置，然后就直接可以调用 `GPRC` 接口了。

完成简单的定义后，我们可以帮你生成看着不还错的文档。
团队中可以通过 git 来管理文档。

文档不再是只能看，开发者和接口使用者也不用等到所有接口都完成时才能知道接口调用是没问题的。

本工具将让文档参与到你的开发过程中。
开发者保证自己的接口是和文档上一样的参数及返回数据格式。
用户根据文档直接请求去调用接口，并确保自己的调用参数是符合文档约定的。


## 上手教程

在[这里](/zh-cn/quick-start)可以快速了解怎么写接口文档。

一个接口可以简单到只需要几行定义

```yaml
project:
  host: xjz.pw

apis:
  - title: Get a user information
    method: GET
    path: /api/v1/users/\d+
    response:
      success:
        http_code: 200
        data:
          id: 1
          name: User 1
```


## 界面预览

![app-1](/imgs/app-1.png)

![app-2](/imgs/app-2.png)


## 文档规范及示例

[SPEC](https://github.com/xiejiangzhi/xjzproxy-docs/blob/master/zh-cn/SPEC.md)

[示例项目](https://github.com/xiejiangzhi/xjzproxy-docs)

