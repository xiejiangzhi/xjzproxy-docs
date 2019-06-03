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


## Description

Let the document tell you the parameters of the interface request, and whether the data returned by the server is consistent with the definition in the document. You can call the interface without getting the interface developed.

After creating the interface document, the user can access the defined interface directly through the proxy. If you use `GRPC`, you don't even need to define the interface documentation one by one. Just specify the protobuf in place and then call the `GPRC` interface directly.


With a simple definition, you can generate a nice API documentation. It can be version
controled by tools such as git.

Documents are no longer a loose contract. Developers and interface consumers don't have to wait until all interfaces are complete to know that the interface call is ok.

This tool makes API documentation part of your development process. With this tool, developers
can guarantee that the parameters and the returned data is always aligned with the
documentation. At the same time, the client can directly request the API according to the
documentation and ensure the requests are as documented.

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


