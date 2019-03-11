XJZ Documents
=============

## Quick Start


```yaml

partials:
  user:
    id: integer
    nickname: string len 10

responses:
  invalid_page:
    http_code: 400
    data:
      code: 1
      msg: Invalid page
    
  invalid_token:
    http_code: 400
    data:
      code: 1
      msg: Invalid token
    


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
        - items: $p/user * 2
          total: 2
      error:
        - $r/invalid_page

project:
  scheme: https
  host: xjz.pw

plugins:
  auth:
    query:
      token: string
    labels: ['auth']
    error: $r/invalid_token

```

## Template

* types: your data type. We'll use them in `partials` and `response`. Use prefix `t/`
* partials: your partial data. We'll use them in `response`. Use prefix `p/`
* responses: create a response, api can use it to generate data. Use prefix `r/`
* apis: all interface defintion.
* project: some config for this project
* plugins: plugins for this project. Apply to api by lables


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
    misc: $t/my_type # use data in value

responses:
  user_show:
    $ref: p/user # use data by $ref, value don't need $ prefix
    avatar: url
    phone_number: $t/phone_number # use data in value, must add $ prefix

  users:
    items: $p/user * 2 # use list of data. $data arg1 arg2 ...
    total: 2

  posts:
    id: integer
    misc: $t/my_type

  attachments: $f/images.png # use `f/` to read data from file
```

## System Types

## Responses

## APIs

## Project

## Plugins
