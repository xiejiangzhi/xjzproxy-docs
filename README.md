XJZ Documents
=============

## Document specification

[English](./SPEC.md) |
[中文](./SPEC-zh-cn.md)

## Examples

* example1.yml: a project that is single file
* example2.yml: a project that has many files
* example3.yml: a grpc project


## Quick Start

create a project folder `my_project`, and create a config file `my_project/app.yml`


```yaml

project:
  url: https://xjz.pw

partials:
  user:
    id: .t/integer
    nickname: string len 10

responses:
  invalid_page:
    http_code: 400
    data:
      code: 1
      msg: Invalid page
    
  list_users:
    http_code: 400
    data:
      items: .p/user * 2
      total: 2
    
apis:
  - title: Get all users
    desc: more desc of this API
    method: GET
    path: /api/v1/users
    labels: ['auth']
    query:
      page: 1
    response:
      success: .r/list_users
      error: .r/invalid_page

```

Add `my_project` folder and set your client use our HTTP proxy. Then you can access http://xjz.pw/api/v1/users ,
it will return data that defined by `my_project`



