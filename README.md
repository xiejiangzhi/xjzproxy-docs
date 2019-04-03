XJZ Documents
=============

## Examples

* example1.yml: a project that is single file
* example2.yml: a project that has many files
* example3.yml: a grpc project


## Quick Start


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
      success:
        - .r/list_users
      error:
        - .r/invalid_page

```

## Modules

* project: some config of this project
* types: define your data type. Reference a type `.t/a_type_name`
* partials: your partial data. We can use it in partials/responses/apis/plugins. Reference a partial `.p/a_partial_name`
* responses: define your responses, api can use it to generate data. example `.r/my_res_name`
* apis: all interface defintions.
* plugins: plugins for this project.


## Data references

```
types: 
  phone_nubmer:
    type: string # system type
    # arguments of the type
    length: 8
    set: 0123456789
    prefix: 138

  my_type:
    type: object # complex type
    data:
      a: integer
      b: 
        c: string

partials:
  user: 
    id: integer
    misc: .t/my_type # use data in value

responses:
  user_show:
    .*: .p/user # expand hash inside a hash by key `.*`
    avatar: url
    phone_number: .t/phone_number # use data in value

  users:
    items: .p/user * 2 # use list of data.
    total: 2

  posts:
    id: integer
    misc: .t/my_type

  attachments: .f/images.png # use `.f/` to read data from file
```

## System Types

* integer
* string
* array
* hash

## Custom Types

```
types:
  my_type:
    items: 
      - 'string%{i}'
      - 'string%{i}'
    prefix: [a, b, c]
    suffix: [d, e, f]
    script: |
      'asdf' + i.to_s
```

## Partials

```
partials:
  user:
```

## Responses

```
responses:
  show_user:
    http_code: 200
    data:
      id: 1
      name: 2
    desc:

  other:
    # defualt http_code is 200
    data: 
      id: integer
      


```

## APIs

## Project

## Plugins
