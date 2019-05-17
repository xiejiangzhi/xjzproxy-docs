---
description: We really develop the interface by document and check the interface by document.
---

XJZProxy
=========

## Features

* HTTP/HTTPS/HTTP2/GRPC Proxy
* Group request history based on host or connection
* Support filtering request history using simple search syntax
* Automatically generate interface document preview page
* Automatically generate response data
* Automatically compare requests, response data, and show differences
* Generate corresponding error statistics by filtering results


## Description

We really develop the interface by document and check the interface by document.

Maybe your workflow is like this

1. Create interface document.
2. Interface developer code by document.
3. Interface users develop by document and simulate data themselves.
4. Deveploy test server, then someone found inconsistencies in the code and the documentation.
5. Update document or code, and continue to the fourth step.


In this case, a document is just a convention. Developers and users can only wait until the last docking to determine that all interfaces are as correct as the documentation.


Based on the above, we have developed a tool to make the document a real and effective contract. The developer guarantees that its interface is the same parameter and return data format as the document. The user directly requests the interface according to the document and ensures that the calling parameters of the user are in accordance with the document convention.


## Preview

![app-1](./imgs/app-1.png)

![app-2](./imgs/app-2.png)


## Document Specification

[中文](https://github.com/xiejiangzhi/xjzproxy-docs/blob/master/SPEC-zh-cn.md) |
[English](https://github.com/xiejiangzhi/xjzproxy-docs/blob/master/SPEC.md)


## Quick Start

### Open XJZProxy

After opened, you'll see `Proxy Runnning` on top-right. If it's `Proxy Stopped`，Please check and change the port in `Proxy` tab, and turn on the proxy.

### Create a document for your project

Create a folder `my_project` under the `~/home/XJZProxy`（it's default directory of all projects, your can change it）

Then, add a configuration file with suffix `.yml`, such as `~/home/XJZProxy/my_project/app.yml`


```yaml

project:
  url: https://xjz.pw # 默认代理只处理项目中 url 匹配的请求

partials:
  user: # 定义一个用户模板，后面可以通过 `.p/user` 引用。可以定义任何形式的数据。
    id: .t/integer
    nickname: string len 10

responses:
  invalid_page: # 定义一个请求失败响应，可以通过 `.r/invalid_page` 来引用。必须使用规定的数据格式。
    http_code: 400
    data:
      code: 1
      msg: Invalid page
    
  list_users:
    http_code: 400
    data:
      # `.p/user * 2` 生成一个数组，里面两个 user 模板定义的元素
      # [ {id: x1, nickname: y1}, {id: x2, nickname: y2} ]
      items: .p/user * 2
      total: 2
    
apis:
  - title: Get all users # 必须的
    desc: more desc of this API # 可选的
    method: GET
    path: /api/v1/users # 当请求匹配 method 和 path 时，就会返回 response['success'] 中的数据
    labels: ['auth']
    query:
      page: .t/integer # 定义请求时必须传 page 参数在 query 中，类型为 integer
    response:
      success: .r/list_users # 必须的默认响应数据
      error: .r/invalid_page # 自定的其它响应数据，暂时只会在文档中看到

```

A few seconds after the document is saved, you will be prompted to add a new project. Now we can see the document in `Project` tab. If your configuration is incorrect, You will see an error message in the documentation.


Make sure all is right. we can access the API by proxy, it will return your defined data.

```
curl http://xjz.pw/api/v1/users --proxy localhost:9898
```

If you want to use HTTPS, Copy the Root CA URL in `Proxy` tab, then open your browser to download and install it.
