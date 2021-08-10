---
layout: posts
title: "[Express] Unhandled promise rejection 에러"
categories:
  - React
last_modified_at: 2021-08-10
author_profile: true
tags:
  - 웹 프레임워크
  - express
  - error handling
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---


```
 UnhandledPromiseRejectionWarning: Unhandled promise rejection. This error originated either by throwing inside of an async function without a catch block, or by rejecting a promise which was not handled with .catch(). 
 To terminate the node process on unhandled promise rejection, use the CLI flag `--unhandled-rejections=strict` (see https://
nodejs.org/api/cli.html#cli_unhandled_rejections_mode). (rejection id: 1)
(node:2265) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process
with a non-zero exit code.
```

엥 mongoose에서 스키마 잘못 적어서 그런거 같은데?
files: [file] 그거 지우고

api/workspace/index.js 에서
const User = ..../models/account
const File = .... 이거 지우니까 에러 없어짐..!

------------

[Error] In uploadFile: TypeError [ERR_INVALID_ARG_TYPE]: The "path" argument must be of type string or an instance of Buffer or URL. Received an instance of Object