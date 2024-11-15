---
layout: post
title: Docker-Mac(M1)上运行Ubuntu-AMD64
---

## QEMU

---

Docker 需要 QEMU 来在 ARM 架构上运行 AMD64 镜像。我们可以通过运行 multiarch/qemu-user-static 镜像来配置 QEMU：

```shell
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
```

这条命令会在 Docker 中配置多架构支持。

- 重启 Docker

## 下载并运行 AMD64 架构的 Ubuntu 镜像

---

- 拉取 AMD64 版本的 Ubuntu 镜像：

```shell
docker pull --platform linux/amd64 ubuntu:20.04
```

- 创建并运行容器：

```shell
docker run --platform linux/amd64 -it ubuntu:20.04
```

- 验证架构

在容器中运行以下命令以验证正在使用 AMD 架构的 Ubuntu：

```shell
uname -m
```

输出应为 x86_64，表明当前运行在 AMD64 模拟环境中。

## 目录挂载

---

要将本地文件或目录挂载到 Docker 容器中，可以使用 -v 或 --mount 参数。

- 使用 -v 选项挂载目录

```shell
docker run -v /path/to/local/file_or_directory:/path/in/container -it ubuntu
```

其中：

- /path/to/local/file_or_directory 是你本地文件或目录的路径。
- /path/in/container 是你希望在容器中挂载的位置。

例如，如果你想将本地的 /Users/leeson/Downloads 目录挂载到容器中的 /app 目录，可以运行：

```shell
docker run -v /Users/leeson/Downloads:/app -it ubuntu:20.04
```

- 使用 --mount 选项（推荐）

--mount 提供更直观的语法，适合更复杂的挂载情况。基本语法如下：

```shell
docker run --mount type=bind,source=/path/to/local/file_or_directory,target=/path/in/container -it ubuntu
```

- type=bind 指定挂载类型为绑定挂载。
- source 指定本地文件或目录路径。
- target 指定在容器中的挂载位置。

## 打开之前的容器

---

上述方案每次都会创建一个新的容器，想要使用之前的容器，则需要下面的操作：

- 查看现有的容器列表

首先，找到之前的容器。

查看所有容器（包括已停止的）：

```shell
docker ps -a
```

输出示例：

```shell
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
ae70bfe5e9a8 ubuntu "/bin/bash" 2 minutes ago Exited (0) 1 minute ago compassionate_brown
```

从中找到目标容器的 CONTAINER ID 或 NAMES。

- 重新启动容器

如果容器已停止，可以重新启动它：

```shell
docker start <CONTAINER_ID_or_NAME>
```

- 进入容器的终端

使用 docker exec 或 docker attach 进入目标容器。

方法 1：docker exec（推荐）

启动一个新的交互式终端，不会影响容器内原有的进程。

```shell
docker exec -it <CONTAINER_ID_or_NAME> /bin/bash
```

方法 2：docker attach

重新附加到容器的终端，会直接连接到容器的主进程。

```shell
docker attach <CONTAINER_ID_or_NAME>
```

注意：

- 如果你使用 docker attach，按 Ctrl+C 退出时可能会停止容器。
- 为避免上述情况，可以使用 docker exec 启动一个新的 Bash 会话。
