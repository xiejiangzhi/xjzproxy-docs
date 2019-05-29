---
title: XJZProxy
description: The document is the interface, using document development and testing interfaces.
lang: en-US
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

A tool lets you use the documentation to check the interface, use the document to accept the request and return the defined data.

After creating the interface document, the user can access the defined interface directly through the proxy. If you use `GRPC`, you don't even need to define the interface documentation one by one. Just specify the protobuf in place and then call the `GPRC` interface directly.


[Here](/quick-start) you can quickly learn how to define interface documentation. After completing the simple definition, we can help you generate a document that looks like it is not wrong. Documents can be managed by git on the team.

Documents are no longer a loose contract. Developers and interface consumers don't have to wait until all interfaces are complete to know that the interface call is ok.

This tool allows documents to be involved in your development process. The developer guarantees that its interface is the same parameter and return data format as the document. The user directly requests the interface according to the document and ensures that the calling parameters of the user are in accordance with the document convention.


## Preview

![app-1](./imgs/app-1.png)

![app-2](./imgs/app-2.png)


## Document Specification and Examples

[SPEC](https://github.com/xiejiangzhi/xjzproxy-docs/blob/master/SPEC.md)

[Examples](https://github.com/xiejiangzhi/xjzproxy-docs)


