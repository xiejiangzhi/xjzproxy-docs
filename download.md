---
title: XJZProxy
description: The document is the interface, using document development and testing interfaces.
lang: en-US
---

{% assign url_prefix = site.download_cdn_url | append: '/' | append: site.app_version | append: '/XJZProxy-' | append: site.app_version %}
{% assign version_id = site.app_version | replace: '.', '_' %}

## XJZProxy {{ site.app_version }}

**Changes**

* Support Windows 10
* Show version in application title
* Faster to start
* Fixed some bugs


**Download**

* <a href="{{ url_prefix | append: '.dmg' }}" target='_blank' id='gat_download_osx_{{ version_id }}'>MacOS-x86_64</a>
* <a href="{{ url_prefix | append: '-amd64.deb' }}" target='_blank' id='gat_download_linux_{{ version_id }}'>Ubuntu-x86_64 16/18</a>
* <a href="{{ url_prefix | append: '-win10.7z' }}" target='_blank' id='gat_download_win10_{{ version_id }}'>Windows 10</a>


## Release log & old versions

<a href="https://github.com/xiejiangzhi/xjzproxy-docs/releases" target='_blank' id='gat_show_releases'>Releases</a>

