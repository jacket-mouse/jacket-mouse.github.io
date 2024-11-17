---
layout: post
title: 计算机网络-使用Socket完成一个简易Web服务器
---

## 实验内容

---

在本实验中，您将学习 Python 中 TCP 连接的套接字编程的基础知识：如何创建套接字，将其绑定到特定的地址和端口，以及发送和接收 HTTP 数据包。您还将学习一些 HTTP 首部格式的基础知识。
您将开发一个处理一个 HTTP 请求的 Web 服务器。您的 Web 服务器应该接受并解析 HTTP 请求，然后从服务器的文件系统获取所请求的文件，创建一个由响应文件组成的 HTTP 响应消息，前面是首部行，然后将响应直接发送给客户端。如果请求的文件不存在于服务器中，则服务器应该向客户端发送“404 Not Found”差错报文。

### 代码

---

在文件下面你会找到 Web 服务器的代码框架。您需要填写这个代码。而且需要在标有`# Fill in start` 和 `# Fill in end`的地方填写代码。另外，每个地方都可能需要不止一行代码。

### 运行服务器

---

将 HTML 文件（例如 HelloWorld.html）放在服务器所在的目录中。运行服务器程序。确认运行服务器的主机的 IP 地址（例如 128.238.251.26）。从另一个主机，打开浏览器并提供相应的 URL。例如： http://128.238.251.26:6789/HelloWorld.html

“HelloWorld.html”是您放在服务器目录中的文件。还要注意使用冒号后的端口号。您需要使用服务器代码中使用的端口号来替换此端口号。在上面的例子中，我们使用了端口号 6789. 浏览器应该显示 HelloWorld.html 的内容。如果省略“:6789”，浏览器将使用默认端口 80，只有当您的服务器正在端口 80 监听时，才会从服务器获取网页。

然后用客户端尝试获取服务器上不存在的文件。你应该会得到一个“404 Not Found”消息。

## 代码模版

---

```python
# import socket module
from socket import *
serverSocket = socket(AF_INET, SOCK_STREAM)
# Prepare a sever socket
# Fill in start
# Fill in end
while True:
    # Establish the connection
    print 'Ready to serve...'
    connectionSocket, addr =   # Fill in start  # Fill in end
    try:
        message =   # Fill in start  # Fill in end
        filename = message.split()[1]
        f = open(filename[1:])
        outputdata = # Fill in start  # Fill in end
        # Send one HTTP header line into socket
        # Fill in start
        # Fill in end

        # Send the content of the requested file to the client
        for i in range(0, len(outputdata)):
            connectionSocket.send(outputdata[i])
        connectionSocket.close()
    except IOError:
        # Send response message for file not found
        # Fill in start
        # Fill in end

        # Close client socket
        # Fill in start
        # Fill in end
serverSocket.close()
```

## 实现代码

---

```python
# import socket module
from socket import *
serverSocket = socket(AF_INET, SOCK_STREAM)
# Prepare a sever socket

server_port = 12000
serverSocket.bind(('', server_port))
serverSocket.listen(1)

while True:
    # Establish the connection
    print ('Ready to serve...')
    connectionSocket, addr = serverSocket.accept()
    try:
        message = connectionSocket.recv(1024).decode()
        filename = message.split()[1] # /HelloWorld.html
        # with open(filename[1:], "r") as f: # HelloWorld.html
        #     outputdata = [line.encode() for line in f]
        f = open(filename[1:])
        outputdata = f.read()
        # Send one HTTP header line into socket
        # 最后有一行空行，用来分割响应头和响应体
        # String
        response_headers = (
            "HTTP/1.1 200 OK\r\n"
            "Content-Type: text/html; charset=utf-8\r\n"
            "Connection: close\r\n"
            f"Content-Length: {len(outputdata)}\r\n"
            "\r\n"
        )

        connectionSocket.send(response_headers.encode())

        # Send the content of the requested file to the client
        for i in range(0, len(outputdata)):
            connectionSocket.send(outputdata[i].encode())
        connectionSocket.close()
    except IOError:
        # Send response message for file not found
        error = "HTTP/1.1 404 Not Found\r\n"
        connectionSocket.send(error.encode())

        # Close client socket
        connectionSocket.close()
serverSocket.close()
```

易错点：响应头的格式以及每行之后都要加上`\r\n`否则不起作用
