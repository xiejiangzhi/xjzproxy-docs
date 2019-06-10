## 文档模块

* project: 项目的全局配置
* types: 创建自定义的类型，通过 `.t/` 前缀来使用，比如 `.t/a_type_name` 引用 `a_type_name` 类型
* partials: 局部模板，可以用在 partials, response 和 apis 中引用. 通过 `.p/` 前缀来使用，比如 `.p/a_partial_name` 引用 `a_partial_name` 模板
* responses: 接口的响应数据定义, api 中可以通过 `.r/my_res_name` 来引用
* apis: 所有的 api 接口定义
* plugins: 所以的插件定义


## 数据操作符

* `.t/{name}`: 引用类型，包括系统类型和自定类型
* `.p/{name}`: 引用局部模板
* `.r/{name}`: 引用响应体
* `.f/{path}`: 引用文件内容，路径相对于当前项目
* `.*`: 展开内容
* `.(t|p)/{name} * number`: 创建数据集合，长度为 number. `.t/name * 2` 相当于 `['.t/name', '.t/name']`


## 数据引用示例及特殊变量

```
types: 
  phone_nubmer:
    items: 
      - 12345678
      - 22345678
      - 32345678
      - 42345678
    prefix: [138, 131, 181]

  my_type:
    items: [1, 2, 3]

partials:
  user: 
    id: .t/integer # 系统类型
    misc: .t/my_type # 引用自定的数据类型 my_type

responses:
  user_show:
    # 展开 user 内容
    # 结果为 { id: '.t/integer', misc: '.t/my_type', avatar: ..., phone_number: ... }
    .*: .p/user
    avatar: url
    phone_number: .t/phone_number # use data in value

  users:
    # 生成数据集合
    # 相当于 items: [.p/user, .p/user]
    items: .p/user * 2
    total: 2

  posts:
    id: .t/integer
    misc: .t/my_type

  attachments: .f/images.png  # 读取 images.png 中的数据
```

## Project

如果一个请求 host 及协议匹配下面的 url，应用将会处理些请求而不是直接转发。比如可以在 history 中看到相关的请求数据。并可以通过定义的 apis 来生成 mock 数据

```
project:
  url: https?://xjz\.pw # 这是一个正则表达式
```


## 系统类型

**基础类型**

* integer
* float
* string: 短文本
* text: 长文本
* boolean

**扩展类型**

* name
* email
* avatar
* username
* hex_color
* color_name
* domain
* url
* date: 日期字符串, 格式: "YYYY-mm-dd"
* datetime: 日期时间字符串, 格式: "YYYY-mm-dd HH:MM:DD"
* markdown: markdown 文本


## 定制类型


|field|required|value|desc|
|-----|--------|-----|----|
|regexp|true, 如果 items 为空|正则字符串|按此正则来生成随机数据或是验证数据|
|items|true, 如果 items 为空|String 或 Array|随机选择一个。为字符串时支持计数器变量 `i`, 比如值为 `"num: %{i}"` 时，自动替换 `%{i}` 为当前计数器的值|
|prefix|false|String 或 Array|如果存在，将转换 items 中的值为字符，并从这里面随机选取一个作为前缀|
|suffix|false|String 或 Array|如果存在，将转换 items 中的值为字符，并从这里面随机选取一个作为后缀|


示例

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


## Partials 局部模板

只是一个数据模板，可以放置任何数据。可以在 response 或 api 中通过 `.p/{name}` 来引用

```
partials:
  user: # 模板名，使用 .p/user 来引用
    a: 1
```

## API 响应体

类似局部模板，但它有固定的响应体数据结构


|field|required|value|desc|
|-----|--------|-----|----|
|desc|false| String||
|http_code|false|默认为 200||
|headers|false|hash|http headers|
|headers['content_type']|false|string|返回的数据格式|
|data|true|任何, String, Hash, 或其它的|API 将会返回这些数据|


```
responses:
  show_user: # 响应体名字
    desc: xyz # 描述可选
    http_code: 200 # 默认 http 状态码为 200
    data: # 响应体返回的数据内容，这里可以为任何的数据结构
      id: .t/integer
      name: .t/name

  response_name:
    desc: xxx
    http_code: 400
    headers: # 也可以定制响应体的 header 
      a: 1
      content_type: application/json # 定义 data 的返回格式
    data: 
      id: .t/integer
```

如果不定义headers['content_type']，将自动根据客户端的 accept 来识别。

下面为支持的 content_type

* `application/json`
* `application/xml`
* `text/csv`
* `text/plain`
* `application/grpc`


## APIs

所有的接口都在这里，如果请求方法和路径匹配，将会直接按 response['success'] 中对应的定义来生成数据并返回，而不是去请求服务器。

|field|required|value|desc|
|-----|--------|-----|----|
|title|true|||
|method|true|get, post, put...|http request method|
|path|true|regexp, /api/v1/users|request path|
|labels|false|array|对应 labels 的插件将会应用到此接口|
|desc|false|string|接口描述，会在文档中显示|
|query|false|hash, like {page: ".t/integer"}|请求的 query 参数定义|
|body|false|like query|请求的 body 参数定义|
|params|false|like query|请求的参数定义, 调用时 query 或 body 任一存在对应参数|
|response|true|hash, see response|接口可以返回的数据|


如下所示，当发送请求通过代理时 `curl http://xjz.pw/api/v1/users --proxy localhost:9898`，将直接返回 `.r/list_users` 定义的数据结构

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

**API 参数及响应**

使用前缀 `.` 来为定义字段参数，这些参数在文档生成及数据检查时将会用到。可以在 query, body, params 及 responses 中使用。

```
response:
  list_users:
    data: .p/user * 2

  invalid_page:
    http_code: 400
    data:
      error: Invalid page
      .error.desc: 错误描述

apis:
  - title: Get all users
    desc: more desc of this API
    method: POST
    path: /api/v1/users
    labels: ['auth']
    query: # 检查 query 中的数据
      page: 1
      .page.desc: xxx # 文档中对 page 参数的描述
      .page.optional: true # 设置字段为可选的。所以字段默认为必须的

      page_size: .t/integer
      .page_size: # 等同 .page_size.desc + .page_size.optioanl 的定义
        desc: Max 100
        optional: true
    body: # 请求 body 中的数据字段
      token: .t/string
    params: # query 或 body 任一存在对应的字段
      token: .t/string

    response:
      success: .r/list_users
      invalid_page: .r/invalid_page
```

**字段参数:**

* desc: 字段描述，用于文档生成
* optional: true, false. 默认值为 false，文档及数据检查时使用

**TODO:**

* optional: true, false, { 'if' => field_name }, { 'unless' => field_name }
* required: true, false, { 'if' => field_name }, { 'unless' => field_name }
* rejected: true, false, { 'if' => field_name }, { 'unless' => field_name }


## Project

如果请求匹配项目的 host ，代码将会记录请求或是为请求生成数据

|field|required|value|desc|
|-----|--------|-----|----|
|host|true|regexp string||
|grpc|false|hash||
|grpc['dir']|true|./protobuf_dir|protobuf 在项目中的路径，相对于当前目录|
|grpc['protoc_args']|false|string, example "-I./path/to/include"|使用 protoc 时的定制参数，除了*_out|
|grpc['proto_files']|false|protobuf 文件匹配器, 默认为 '**/*.proto'|只会编译匹配的文件|

```
project:
  host: .+\.xjz\.pw # regexp
  grpc:
    dir: protos
```


## Plugins

|field|required|value|desc|
|-----|--------|-----|----|
|labels|true|array|这个插件将会应用到有这些 labels 的 apis 上|
|template|false|hash|API 模板, 它相当于一个 api 定义的默认值|

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

`GET /api/v1/users` 将会基于 auth 和 paging 模板. 所以这个接口将会需要 `token`, `page` 和 `page_size` 参数. 当然, 这个接口同样会有 `invalid_token` 的响应数据.


## 引用

* 正则表达式规范: https://ruby-doc.org/core-2.6.2/Regexp.htm
