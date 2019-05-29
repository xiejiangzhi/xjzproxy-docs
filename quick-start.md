---
title: XJZProxy
description: The document is the interface, using document development and testing interfaces.
lang: en-US
---

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


If everything goes well, we can now access the API through the proxy, and it will return the data
you have previously defined.

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

When saved, the desired response for the corresponding API can be switched in the app's Project document.

![Choose Response](/imgs/app-3.png)


### API Parameters

Most of time, the API requires parameters to be provided when called. If the parameters are
passed through query string, they can be defined as query , otherwise if the parameters are
passed through HTTP body, they can be defined as body . You can also use params to make
the API compatible with parameters passed in both ways.

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

The data generated by the same structure of the content is fixed. But we can only constrains the data type, and then automatically generate the corresponding format of the data. Reference system type by using `.t/{name}`. For more types, please see [Document Specification](https://github.com/xiejiangzhi/xjzproxy-docs/blob/master/SPEC.md)

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

Need a custom type? It can be defined in `types`.

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


## Document Specification and Examples

[SPEC](https://github.com/xiejiangzhi/xjzproxy-docs/blob/master/SPEC.md)

[Examples](https://github.com/xiejiangzhi/xjzproxy-docs/blob/master)


