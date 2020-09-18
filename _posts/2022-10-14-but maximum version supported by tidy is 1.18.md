---
title: go.mod file indicates go 1.19, but maximum version supported by tidy is 1.18
author: Uphie
date: 2022-10-14 21:13:00 +0800
categories: [技术]
tags: [go]
math: true
toc: true
---

在项目目录下执行 `go mod tidy` 失败，提示以下内容：
```
go: go.mod file indicates go 1.19, but maximum version supported by tidy is 1.18
```

原因是项目中 go.mod 中的 Go 版本为1.19，但笔者本机的 Go 版本为1.18，版本低了。

解决方案：到 [https://golang.google.cn/dl/](https://golang.google.cn/dl/) 下载新版本升级本机 Go。