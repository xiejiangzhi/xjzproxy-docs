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


## 我们遇到的问题

通常在需求确定后，我们会先写一份接口文档给前端，然后前端按文档定义的接口去开发。

在这期间我们可能会遇见很多问题:

1. 结构松散或是没有结构的文档格式，很难去管理、协作，写起来也麻烦
2. 只有文档，使用者自己 mock 各种数据很麻烦，有时会等真实接口好了才会开发
3. 请求参数格式没按文档来，在个例下正常工作，到复杂的线上就出问题了
4. 后端接口没有文档格式来，在特定情况下前端没有处理而出问题
5. 接口改了，但文档没改，久而久之，文档形同虚设


## 我们的解决方案

1. 使用 YAML 书写结构化的文档，数据结构可以复用。一个接口甚至可以简单到只需要几行
2. 文档完成后，你就相当于有了一个本地的后端服务器，文档中所有接口可以直接调用
3. 在请求文档接口时，我们会按文档检查请求参数是否与文档定义一致。遇到不匹配的将会有相应提示
4. 在连接上真实服务器时，我们会帮助你检查数据的返回格式是否与文档定义一致。遇到不匹配的将会有相应提示
5. 以上方式让文档主动参与到开发中，将督促使用者去更新文档



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

