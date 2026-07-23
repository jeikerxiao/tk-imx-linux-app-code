# ATK I.MX6ULL - 30_socket C APP 示例说明

## 1. 目录功能概述

本示例演示如何在 I.MX6ULL 开发板上使用 Socket 进行网络通信。

程序通过 TCP/IP 协议实现客户端和服务端的数据收发。

**适用硬件外设**：以太网接口

**示例文件对应功能**：
- `socket_client.c`：TCP 客户端示例
- `socket_server.c`：TCP 服务端示例

## 2. 目录内源码文件说明

| 文件名 | 类型 | 作用说明 |
|:---|:---|:---|
| socket_client.c | C 源文件 | TCP 客户端示例 |
| socket_server.c | C 源文件 | TCP 服务端示例 |

## 3. 核心 C 代码逻辑拆解分析

### 3.1 服务端初始化

```c
#include <sys/socket.h>
#include <netinet/in.h>

// 创建 socket
int server_fd = socket(AF_INET, SOCK_STREAM, 0);

// 绑定地址
struct sockaddr_in addr;
addr.sin_family = AF_INET;
addr.sin_port = htons(8080);
addr.sin_addr.s_addr = INADDR_ANY;
bind(server_fd, (struct sockaddr *)&addr, sizeof(addr));

// 监听
listen(server_fd, 5);

// 接受连接
int client_fd = accept(server_fd, NULL, NULL);
```

### 3.2 客户端连接

```c
// 创建 socket
int client_fd = socket(AF_INET, SOCK_STREAM, 0);

// 连接服务端
struct sockaddr_in addr;
addr.sin_family = AF_INET;
addr.sin_port = htons(8080);
addr.sin_addr.s_addr = inet_addr("192.168.1.100");
connect(client_fd, (struct sockaddr *)&addr, sizeof(addr));
```

### 3.3 数据收发

```c
// 发送数据
char *msg = "Hello, Server!";
send(client_fd, msg, strlen(msg), 0);

// 接收数据
char buf[256];
recv(client_fd, buf, sizeof(buf), 0);
```

### 3.4 涉及的 Linux 系统调用与 API

| 系统调用/API | 作用 |
|:---|:---|
| `socket()` | 创建 socket |
| `bind()` | 绑定地址 |
| `listen()` | 监听连接 |
| `accept()` | 接受连接 |
| `connect()` | 连接服务端 |
| `send()` | 发送数据 |
| `recv()` | 接收数据 |
| `close()` | 关闭 socket |

## 4. 交叉编译完整操作步骤

```bash
# 进入示例目录
cd 30_socket

# 编译服务端
arm-linux-gnueabihf-gcc -o socket_server socket_server.c

# 编译客户端
arm-linux-gnueabihf-gcc -o socket_client socket_client.c
```

## 5. 开发板部署流程

```bash
# 确保网络连接正常
ifconfig

# 运行服务端
./socket_server

# 运行客户端（在另一台设备上）
./socket_client 192.168.1.100 8080
```

## 6. 完整运行验证操作

```bash
# 在开发板上运行服务端
./socket_server

# 在 PC 上运行客户端
./socket_client 192.168.1.100 8080

# 预期效果
# 客户端连接服务端并发送数据
# 服务端接收数据并返回响应
```

## 7. 常见报错 FAQ

### 7.1 连接失败

```bash
# 检查网络连接
ping 192.168.1.100

# 检查端口是否被占用
netstat -tlnp
```

### 7.2 端口被占用

**原因**：其他程序占用了该端口。

**解决方案**：
```bash
# 查找占用端口的进程
lsof -i :8080

# 终止进程
kill -9 <PID>
```

### 7.3 数据收发异常

**原因**：网络不稳定或数据格式错误。

**解决方案**：
确保网络连接稳定，检查数据格式是否正确。
