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

**Extension**

* name
* email
* avatar
* username
* hex_color
* color_name
* domain
* url
* date: string of date, format: "YYYY-mm-dd"
* datetime: string of datetime, format: "YYYY-mm-dd HH:MM:DD"
* markdown: text of markdown


## Custom Types


|field|required|value|desc|
|-----|--------|-----|----|
|regexp|true if items is empty|string of regexp|Randomly build string that match this regexp|
|items|true if regexp is empty|non-array or array|randomly pick one if it is array, allow counter variable `i`, example `"num: %{i}"`|
|prefix|false|non-array or array|convert items to string, and add a prefix that is randomly pick one from this array|
|suffix|false|non-array or array|convert items to string, and add a suffix that is randomly pick one from this array|

```
types:
  my_type:
    items: 
      - 'string%{i}'
      - 'string%{i}'
    prefix: [a, b, c]
    suffix: [d, e, f]

  my_regexp_type:
    regexp: "^(a|b|c)string\d{1,5}(d|e|f)$"
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


|field|required|value|desc|
|-----|--------|-----|----|
|desc|false| string||
|http_code|false|default is 200||
|headers|false|hash|http header of response|
|headers['content_type']|false|string|how to format the data|
|data|true|any, string, hash, etc|api will return these data|


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
    headers: 
      content_type: 'application/json'
    data: 
      id: integer
```

## APIs

All api in here, if the host, path and request method match a request, it will render the response.

|field|required|value|desc|
|-----|--------|-----|----|
|title|true|||
|method|true|get, post, put...|request method|
|path|true|regexp, /api/v1/users|request path|
|labels|false|array|what plugins will apply to this api|
|desc|false|string|description of the api|
|query|false|hash, like {page: ".t/integer"}|defined request query data|
|body|false|like query|defined request body data|
|params|false|like query|defined request params data, caller need submit the field in query or body|
|response|true|hash, see response|some response this api able to return|


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
      success: .r/list_users # success is required
      invalid_page: .r/invalid_page # and you can use other keys
```

**Field arguments**

```
apis:
  - title: Get all users
    desc: more desc of this API
    method: GET
    path: /api/v1/users
    labels: ['auth']
    query:
      page: 1
      .page.desc: xxx # argument of page field
      .page.optional: true

      page_size: .t/integer
      .page_size: # arguments of page_size field
        desc: Max 100
        optional: true

    response:
      success: .r/list_users
      invalid_page: .r/invalid_page
```

**Valid field arguments:**

* desc:
* optional: true, false. Default value is false

**TODO:**

* optional: true, false, { 'if' => field_name }, { 'unless' => field_name }
* required: true, false, { 'if' => field_name }, { 'unless' => field_name }
* rejected: true, false, { 'if' => field_name }, { 'unless' => field_name }


## Project

If a request match the url, application will proxy or hack the request.


|field|required|value|desc|
|-----|--------|-----|----|
|host|true|regexp string||
|grpc|false|hash||
|grpc['dir']|true|./protobuf_dir|path of the protobufs, relative current project folder|
|grpc['protoc_args']|false|string, example "-I./path/to/include"|extend arguments when your use grpc|
|grpc['proto_files']|false|protobuf file matcher, default value is '**/*.proto'|only compile matched protobufs|

```
project:
  host: .+\.xjz\.pw # regexp
```


## Plugins

|field|required|value|desc|
|-----|--------|-----|----|
|labels|true|array|this plugin will apply to whice apis|
|template|false|hash|api template, it's default value of a api|

Example:

```
apis:
  - title: list users
    labels: [auth, paging]
    method: get
    path: /api/v1/users
    response:
      success:
        data:
          items: .p/user * 10

plugins:
  - labels: [auth]
    template:
      params:
        token: .t/string
      response:
        invalid_token:
          http_code: 401
          data:
            error: Invalid token


  - labels: [paging]
    template:
      params:
        page: .t/string
        page_size: .t/string
```

`GET /api/v1/users` will based on auth and paging template. So it has `token`, `page` and `page_size` params. Sure, it also get the `invalid_token` response.

