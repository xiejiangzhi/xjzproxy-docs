XJZ Documents
=============

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
      success:
        - .r/list_users
      error:
        - .r/invalid_page

```

Add `my_project` folder and set your client use our HTTP proxy. Then you can access http://xjz.pw/api/v1/users ,
it will return data that defined by `my_project`


## Modules

* project: Some global config of this project
* types: Define your data type. Then `.t/a_type_name` will link to this type.
* partials: Your partial data. We can use them in partials/responses/apis/plugins. Link a partial `.p/a_partial_name`
* responses: Your responses data, api can use it to generate data. example `.r/my_res_name`
* apis: All interface defintions.
* plugins: Plugins for this project.


## Data link examples

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
    # use list of data.
    # it will create a array of two users
    items: .p/user * 2
    # equal items: [.p/user, .p/user]
    total: 2

  posts:
    id: integer
    misc: .t/my_type

  attachments: .f/images.png  # use `.f/` to read data from file, relative this project path
```

## System Types

**Base Type**

* integer
* float
* string: short text
* text: long text
* boolean
* date
* datetime

**Extension**

* name
* email
* avatar
* username
* hex_color
* color_name
* domain
* url
* markdown: text of markdown


## Custom Types

* items: randomly pick one, allow counter variable `i`, example `"num: %{i}"`
* prefix: convert items to string, and add a prefix that is randomly pick one from this array
* suffix: convert items to string, and add a suffix that is randomly pick one from this array
* script: Generate data by ruby code



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

Them is data template.

```
partials:
  user: # if you want to use value of user, use .p/user to link it.
    a: 1
```

## Responses

Like partials, but it has a fixed data schema

```
responses:
  show_user:
    desc: xyz
    # defualt http_code is 200
    http_code: 200
    data:
      id: 1
      name: 2

  response_name:
    desc: xxx
    http_code: 400
    http_headers: 
      a: 1
    data: 
      id: integer
```

## APIs

All api in here, if the host, path and request method match a request, it will render the response

```
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


## Project

If a request match the url, application will proxy or hack the request.

```
project:
  url: https?://xjz.pw # regexp
```
