---
title: XJZProxy
description: 文档即接口，使用文档开发、测试接口。
---


## 快速上手

### 启动软件

打后软件后右上可以看到 `Proxy Runnning`，如果是 `Proxy Stopped` 状态，请在 `Proxy` 中检查更换默认端口再将开关切换到启动


### HTTPS ( 可选 )

如果要使用 HTTPS ，在 `Proxy` 中复制 Root CA URL，并打开浏览器下载安装。


### 创建接口项目

在 `~/home/XJZProxy`（默认项目目录，可修改）下创建一个项目目录 `my_project`

然后添加一个配置文件，以 `.yml` 为后缀，比如 `~/home/XJZProxy/my_project/app.yml`


```yaml

project:
  host: xjz.pw

apis:
  - title: Get all users
    method: GET
    path: /api/v1/users
    response:
      success:
        http_code: 200
        data:
          items:
            - id: 1
              name: User 1
            - id: 2
              name: User 2
          total: 2
```

文档保存后几秒，将会出现提示加了新的项目，现在可以在 `Project` 中查看项目对应的文档了。如果定义格式有问题，也会在文档中显示错误信息。


确认正确后，通过代码访问对应的接口将返回定义好的数据内容

```
curl http://xjz.pw/api/v1/users --proxy localhost:9898
```

### 多种响应

一个接口通常会有多种响应格式，比如返回错误信息、不同的数据类型。在 response 中，你可以随意定制。

```yaml

project:
  host: xjz.pw

apis:
  - title: Get all users
    method: GET
    path: /api/v1/users
    response:
      success:
        http_code: 200
        data:
          items:
            - id: 1
              name: User 1
            - id: 2
              name: User 2
          total: 2
      empty:
        http_code: 200
        data:
          items: []
          total: 0
      invalid_user:
        http_code: 401
        data:
          error: invalid user
```

保存后，在应用的 `Project` 文档中，找到对应接口的 response 可以选择切换。


![Choose Response](./imgs/app-3.png)


### 请求参数

大多数接口都会有请求参数，url 中的 `query` 和 http 请求中的 `body` 分别使用 `query` 和 `body` 来定义。当然我们也可以忽略参数位置，使用 `params` 告诉接口只要有这个参数就行了，不管是在 query 还是在 body 中

```yaml

project:
  host: xjz.pw

apis:
  - title: Get all users
    method: GET
    path: /api/v1/users
    query:
      page: 1
      page_size: 10
    response:
      success:
        http_code: 200
        data:
          items:
            - id: 1
              name: User 1
            - id: 2
              name: User 2
          total: 2

  - title: Top users
    method: GET
    path: /api/v1/top_users
    params:
      n: 2
    response:
      success:
        http_code: 200
        data:
          items: []
          total: 0
```


### 共享响应体

当多个接口有相同的响应体时，过多的重复会增加文档的复杂度。我们可以在 `responses` 中来定义共享的响应体。然后通过 `.r/{name}` 来引用

```yaml

project:
  host: xjz.pw

responses:
  list_users:
    http_code: 200
    data:
      items:
        - id: 1
          name: User 1
        - id: 2
          name: User 2
      total: 2

  empty_items:
    http_code: 200
    data:
      items: []
      total: 0

  invalid_user:
    http_code: 401
    data:
      error: invalid user

apis:
  - title: Get all users
    method: GET
    path: /api/v1/users
    response:
      success: .r/list_users
      empty: .r/empty_items
      invalid_user: .r/invalid_user

  - title: Top users
    method: GET
    path: /api/v1/top_users
    params:
      n: 2
    response:
      success: .r/list_users
      empty: .r/empty_items
      invalid_user: .r/invalid_user
```

### 使用模板定义重复内容

`user` 这个结构是通用的，通过 `partials` 来定义共享的数据结构。然后使用 `.p/{name}` 来引用，或者使用 `.p/{name} * {n}` 来引用并多个实例的数组。当然，你也可以使用 partials 来定义其它内容并引用。

```yaml

project:
  host: xjz.pw

partials:
  user:
    id: 1
    name: User 1

responses:
  list_users:
    http_code: 200
    data:
      items: .p/user * 2
      total: 2

  empty_items:
    http_code: 200
    data:
      items: []
      total: 0

  invalid_user:
    http_code: 401
    data:
      error: invalid user

apis:
  - title: Get all users
    method: GET
    path: /api/v1/users
    response:
      success: .r/list_users
      empty: .r/empty_items
      invalid_user: .r/invalid_user

  - title: Top users
    method: GET
    path: /api/v1/top_users
    params:
      n: 2
    response:
      success: .r/list_users
      empty: .r/empty_items
      invalid_user: .r/invalid_user
```


### 使用类型

相同结构的内容生成的数据都是固定的，我们可以只限制数据类型，然后将自动生成相应格式的数据。通过使用 `.t/{name}` 来引用系统类型或自定的类型。更多类型介绍可以参考[文档规范](./SPEC-zh-cn.md)

```yaml

project:
  host: xjz.pw

partials:
  user:
    id: .t/integer
    name: .t/name

responses:
  list_users:
    http_code: 200
    data:
      items: .p/user * 2
      total: 2

  empty_items:
    http_code: 200
    data:
      items: []
      total: 0

  invalid_user:
    http_code: 401
    data:
      error: invalid user

apis:
  - title: Get all users
    method: GET
    path: /api/v1/users
    response:
      success: .r/list_users
      empty: .r/empty_items
      invalid_user: .r/invalid_user

  - title: Top users
    method: GET
    path: /api/v1/top_users
    params:
      n: .t/integer
    response:
      success: .r/list_users
      empty: .r/empty_items
      invalid_user: .r/invalid_user
```


### 定制类型 

需要定制的类型？可以在 `types` 中定义。

```yaml

project:
  host: xjz.pw

types:
  git_repo:
    items:
      - a/a
      - b/a
      - xxx/yyy
    prefix:
      - "git@github.com:"
      - "git@gitlab.com:"
    suffix: ".git"

partials:
  user:
    id: .t/integer
    name: .t/name
  full_user:
    .*: .p/simple_user
    blog_repo: .t/git_repo

responses:
  list_users:
    http_code: 200
    data:
      items: .p/user * 2
      total: 2

  list_full_users:
    http_code: 200
    data:
      items: .p/full_user * 2
      total: 2

  empty_items:
    http_code: 200
    data:
      items: []
      total: 0

  invalid_user:
    http_code: 401
    data:
      error: invalid user

apis:
  - title: Get all users
    method: GET
    path: /api/v1/users
    response:
      success: .r/list_users
      empty: .r/empty_items
      invalid_user: .r/invalid_user

  - title: Top users
    method: GET
    path: /api/v1/top_users
    params:
      n: .t/integer
    response:
      success: .r/list_users
      full_users: .r/list_full_users
      empty: .r/empty_items
      invalid_user: .r/invalid_user
```

### API 模板

多个 API 有相同参数与响应配置时，使用 partial 过于繁琐？通过 `templates` 定义 api 模板，使用 labels 可以组合多个模板

```yaml

project:
  host: xjz.pw

types:
  git_repo:
    items:
      - a/a
      - b/a
      - xxx/yyy
    prefix:
      - "git@github.com:"
      - "git@gitlab.com:"
    suffix: ".git"

partials:
  user:
    id: .t/integer
    name: .t/name
  full_user:
    .*: .p/simple_user
    blog_repo: .t/git_repo

responses:
  list_users:
    http_code: 200
    data:
      items: .p/user * 2
      total: 2

  list_full_users:
    http_code: 200
    data:
      items: .p/full_user * 2
      total: 2

  empty_items:
    http_code: 200
    data:
      items: []
      total: 0

  invalid_user:
    http_code: 401
    data:
      error: invalid user

apis:
  - title: Get all users
    labels: [auth, paging]
    method: GET
    path: /api/v1/users
    response:
      success: .r/list_users

  - title: Top users
    labels: [auth]
    method: GET
    path: /api/v1/top_users
    params:
      n: .t/integer
    response:
      success: .r/list_users
      full_users: .r/list_full_users

plugins:
  - title: Auth user
    labels: [auth]
    template:
      params:
        token: .t/string
      response:
        invalid_user: .r/invalid_user

 - title: Paging
    labels: [paging]
    template:
      params:
        page: .t/string
        page_size: .t/string
      response:
        empty: .r/empty_items
```


### 拆分项目文件

太多数据定义在一个文件中，不方便查看修改，想要分门别类放到不同文件或是目录中？简单，直接拆就好了。每个 `type`, `partial`, `response`, `api`, `plugin` 都可以放到不同的文件中，只需要保持相同的数据结构就没问题。


app.yml

```yaml
project:
  host: xjz.pw
```

types.yml

```yaml
types:
  git_repo:
    items:
      - a/a
      - b/a
      - xxx/yyy
    prefix:
      - "git@github.com:"
      - "git@gitlab.com:"
    suffix: ".git"
```

partials/user.yaml

```yaml
partials:
  user:
    id: .t/integer
    name: .t/name
  full_user:
    .*: .p/simple_user
    blog_repo: .t/git_repo
```

responses/user.yml

```yaml
responses:
  list_users:
    http_code: 200
    data:
      items: .p/user * 2
      total: 2

  list_full_users:
    http_code: 200
    data:
      items: .p/full_user * 2
      total: 2
```

responses/general.yml

```yaml
responses:
  empty_items:
    http_code: 200
    data:
      items: []
      total: 0
```

responses/errors.yml

```yaml
responses:
  invalid_user:
    http_code: 401
    data:
      error: invalid user
```

apis/users.yml

```yaml
apis:
  - title: Get all users
    method: GET
    path: /api/v1/users
    response:
      success: .r/list_users
      empty: .r/empty_items
      invalid_user: .r/invalid_user

  - title: Top users
    method: GET
    path: /api/v1/top_users
    params:
      n: .t/integer
    response:
      success: .r/list_users
      full_users: .r/list_full_users
      empty: .r/empty_items
      invalid_user: .r/invalid_user
```

plugins.yml

```yaml
plugins:
  - title: Auth user
    labels: [auth]
    template:
      params:
        token: .t/string
      response:
        invalid_user: .r/invalid_user

 - title: Paging
    labels: [paging]
    template:
      params:
        page: .t/string
        page_size: .t/string
      response:
        empty: .r/empty_items
```


### 文档描述

接口比较复杂，需要一些描述？使用 `desc` 字段描述接口，使用 `.{field}` 前缀来定义请求参数或响应体的配置

apis/users.yml

```yaml
apis:
  - title: Get all users
    desc: one line description, `markdown` syntax
    method: GET
    path: /api/v1/users
    response:
      success:
        http_code: 200
        data:
          .items.desc: use one line to describe items
          items:
            - id: 1
              .id.desc: use one line to describe id
              name: b

  - title: Top users
    desc: |
      multiple line description
      `makrdown` syntax
      line 2
      line 3
    method: GET
    path: /api/v1/top_users
    params:
      n: .t/integer
      .n:
        desc: n is optional, default value is 5.
        optional: true
    response:
      success: .r/list_users
```

plugins.yml

```yaml
plugins:
  - title: Auth user
    labels: [auth]
    template:
      params:
        token: .t/string
        .token.desc: it's required. 
      response:
        invalid_user: .r/invalid_user

 - title: Paging
    labels: [paging]
    template:
      params:
        page: .t/string
        .page.desc: must > 0
        page_size: .t/string
        .page_page:
          desc: must > 0
          optional: true
      response:
        empty: .r/empty_items
```


## 文档规范及示例

[SPEC](https://github.com/xiejiangzhi/xjzproxy-docs/blob/master/SPEC-zh-cn.md)

[示例项目](https://github.com/xiejiangzhi/xjzproxy-docs/blob/master)

