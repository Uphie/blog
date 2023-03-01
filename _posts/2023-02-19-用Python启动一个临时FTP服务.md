---
title: 用 Python 启动一个临时 FTP 服务
author: Uphie
date: 2023-02-19 19:13:00 +0800
categories: [技术]
tags: [python,ftp]
math: true
toc: true
---

有时候需要将本机的一些文件传递给其他电脑，但考虑到安全性、隐私性，不想通过微信、QQ 等第三方工具传输，又不想特地找专门的软件，那么这时候可以通过 Python 临时开启一个 FTP 服务，即用即开。

先安装 `pyftpdlib` 包：
```shell
pip install pyftpdlib
```

创建文件 ftp_server.py
```python

from pyftpdlib.authorizers import DummyAuthorizer
from pyftpdlib.handlers import FTPHandler
from pyftpdlib.servers import FTPServer
 
# 新建一个用户组
authorizer = DummyAuthorizer()
# 将用户名、密码、服务目录、权限 添加到里面
authorizer.add_user("uphie", "uphie.studio", "/Users/uphie/Desktop", perm="elr")  # adfmw
 
handler = FTPHandler
handler.authorizer = authorizer
# 开启服务器，对所有来源的主机开放，端口21
server = FTPServer(("0.0.0.0", 21), handler)
server.serve_forever()
```

启动 ftp 服务：
```shell
python3 ftp_server.py
```

假设你的 IP 地址是 `192.168.0.10`

在其他电脑上，打开 `ftp://192.168.0.10:21` 即可访问文件了。