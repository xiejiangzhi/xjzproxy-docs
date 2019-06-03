---
title: XJZProxy
description: API documentation itself is the API, develop and test APIs by documenting the APIs.
lang: en-US
---

## Features

* local API Mock based on YAML document, support dynamically generate data
* Automatically generate and export API document.
* Automatically compare request and response data by document, and show differences
* Generate corresponding error statistics
* HTTP/HTTPS/HTTP2/GRPC Proxy


## Workflow

![workflow](/imgs/workflow.png)


## The problem we encountered

Usually after the requirements are determined, we will first write an interface document to the front end, and then the front end will be developed according to the interface defined by the document.

During this time we may encounter many problems:

1. Structured loose or unstructured document format, difficult to manage, collaborate, and troublesome to write
2. Only documents, users themselves mocking various data is very troublesome, sometimes it will be developed after the real interface is ready.
3. The request parameter format is not in accordance with the document. It works normally under the example, and there may be problems on the production environment.
4. The backend interface is not in the document format, and the front end may not be processed in certain situations.
5. The interface has been changed, but the document has not been changed. Over time, the document is ineffective.


## Our solution

1. Write structured documents using YAML, and the data structures can be reused. An interface can even be as simple as a few lines
2. After the document is completed, you are equivalent to having a local backend server. All interfaces in the document can be called directly.
3. When sending a request defined in the document, we will check by the documentation whether the request parameters are consistent with the document definition. If there is a mismatch, there will be a corresponding prompt
4. When connecting to a real server, we will help you check if the return format of the data is consistent with the document definition. If there is a mismatch, there will be a corresponding prompt
5. The above method allows the document to actively participate in the development, and will urge the user to update the document.


## Quick Start

[Here](/quick-start) you can quickly learn how to define interface documentation.

An interface can be as simple as a few lines

```yaml
project:
  host: xjz.pw

apis:
  - title: Get a user information
    method: GET
    path: /api/v1/users/\d+
    response:
      success:
        data:
          id: 1
          name: User 1
```


## Preview

![app-1](/imgs/app-1.png)

![app-2](/imgs/app-2.png)


## Document Specification and Examples

[SPEC](https://github.com/xiejiangzhi/xjzproxy-docs/blob/master/SPEC.md)

[Examples](https://github.com/xiejiangzhi/xjzproxy-docs)


