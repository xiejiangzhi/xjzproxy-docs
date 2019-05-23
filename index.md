---
title: XJZProxy
description: The document is the interface, using document development and testing interfaces.
---

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


### HTTPS

If you want to use HTTPS, Copy the Root CA URL in `Proxy` tab, then open your browser to download and install it.


### Create a document for your project

Create a folder `my_project` under the `~/home/XJZProxy`（it's default directory of all projects, your can change it）

Then, add a configuration file with suffix `.yml`, such as `~/home/XJZProxy/my_project/app.yml`


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

A few seconds after the document is saved, you will be prompted to add a new project. Now we can see the document in `Project` tab. If your configuration is incorrect, You will see an error message in the documentation.


Make sure all is right. we can access the API by proxy, it will return your defined data.

```
curl http://xjz.pw/api/v1/users --proxy localhost:9898
```

### Multiple Response

An interface usually has different response data, such as returning error messages or different data types. In apis `response`, you can customize it at will.

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

After saving, in the application's `Project` document, find the response of the corresponding interface and you can choose to switch.


### API Parameters

Most interfaces will have request parameters, `query` in the url and http `body` are defined using `query` and `body` respectively. Of course, we can also ignore the parameter position, use `params` to tell the interface as long as it has this parameter, whether it is in query or in body.

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


### Shared Responses

When multiple interfaces have the same response body, excessive duplication will increase the complexity of the document. We can define shared response bodies in `responses`. Then reference by `.r/{name}`

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

### Use Partials to define duplicate data

The `user` structure is generic, we can define it in `partials`. Then use `.p/{name}` to reference, or use `.p/{name} * {n}` to reference an array of multiple instances. Of course, you can also use `partials` to define other data and reference it.

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


### Use Type

The data generated by the same structure of the content is fixed. But we can only constrains the data type, and then automatically generate the corresponding format of the data. Reference system type by using `.t/{name}`. For more types, please see [Document Specification] (./SPEC.md)

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


### Customize Type

Need a custom type? Can be defined in `types`.

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

### API Template

多个 API 有相同参数与响应配置时，使用 partial 过于繁琐？通过 `templates` 定义 api 模板，使用 labels 可以组合多个模板
Is it too cumbersome to use partial when multiple APIs have the same parameters and response configuration? Define api templates in `templates` and combine templates with labels

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


### Split Project File

Too much data is defined in one file, it is not convenient to view the changes, want to classify them into different files or directories? Simple, just split it. Each `type`, `partial`, `response`, `api`, `plugin` can be placed in a different file, and only the same data structure needs to be maintained.


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


### API Description

The interface is more complicated and needs some description? Use the `desc` field to describe the interface and the `.{field}` prefix to define the configuration of the request parameter or response body.

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


## Document specification

[SPEC](https://github.com/xiejiangzhi/xjzproxy-docs/blob/master/SPEC.md)


