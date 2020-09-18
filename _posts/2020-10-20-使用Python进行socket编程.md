---
title: 使用Python进行socket编程
author: Uphie
date: 2020-10-20 13:02:20 +0800
categories: [技术]
tags: [python,tcp,udp,socket]
math: true
toc: true
---

基于 TCP 的 Python socket 编程流程:

![socket编程流程](https://s1.ax1x.com/2020/10/20/BSgjk6.jpg)


<center style="font-size:12px;color:#C0C0C0;text-decoration:underline">若有误，欢迎读者指正。</center>

图中绿框部分在实际开发中可以用子线程或子进程进程处理。

Server端示例代码：
```python
import socket
import threading

server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(("0.0.0.0", 8000))
server.listen()


def handle(sck):
    while True:
        rcv_bytes = sck.recv(1024)
        rcv_data = rcv_bytes.decode("utf8")
        print("server got:", rcv_data)
        if rcv_data == "quit":
            sck.close()
            return
        resp_data = f"You said: {rcv_data}. Tell me 'quit' to quit if you want to finish ".encode("utf8")
        sck.send(resp_data)


if __name__ == '__main__':
    while True:
        print("server is accepting")
        sck, addr = server.accept()
        handle_th = threading.Thread(target=handle, args=(sck,))
        handle_th.start()
```

client端示例代码：
```python
import socket

# AF_INET 指 IPv4，SOCK_STREAM 指 TCP 协议
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 连接指定IP和端口号
client.connect(("127.0.0.1", 8000))

if __name__ == '__main__':
    while True:
        input_data = input("input something:")
        client.send(input_data.encode("utf8"))
        rcv_bytes = client.recv(1024)
        print(rcv_bytes.decode("utf8"))
```
