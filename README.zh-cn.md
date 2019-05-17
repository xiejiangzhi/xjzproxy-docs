XJZProxy 文档
=============

## 项目文档规范

[中文](./SPEC-zh-cn.md) |
[English](./SPEC.md)


## 文档示例

* example1: 一个简单的单一文件接口定义
* example2: 项目按不同数据拆分成多个文件
* example3: 一个 GRPC 项目


## 快速上手

### 启动软件

打后软件后右上可以看到 `Proxy Runnning`，如果是 `Proxy Stopped` 状态，请在 `Proxy` 中检查更换默认端口再将开关切换到启动

### 创建接口项目

在 `~/home/XJZProxy`（默认项目目录，可修改）下创建一个项目目录 `my_project`

然后添加一个配置文件，以 `.yml` 为后缀，比如 `~/home/XJZProxy/my_project/app.yml`


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

文档保存后几秒，将会出现提示加了新的项目，现在可以在 `Project` 中查看项目对应的文档了。如果定义格式有问题，也会在文档中显示错误信息。


确认正确后，通过代码访问对应的接口将返回定义好的数据内容

```
curl http://xjz.pw/api/v1/users --proxy localhost:9898
```

如果要使用 HTTPS ，在 `Proxy` 中复制 Root CA URL，并打开浏览器下载安装。
