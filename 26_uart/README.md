# ATK I.MX6ULL - 26_uart C APP 示例说明

## 1. 目录功能概述

本示例演示如何在 I.MX6ULL 开发板上进行 UART 串口通信。

程序通过配置串口参数（波特率、数据位、停止位、校验位等），实现串口数据的发送和接收。

**适用硬件外设**：UART 串口（如 TTL 串口、RS232 等）

**示例文件对应功能**：
- `uart_test.c`：UART 串口通信主程序

## 2. 目录内源码文件说明

| 文件名 | 类型 | 作用说明 |
|:---|:---|:---|
| uart_test.c | C 源文件 | UART 串口通信示例 |

## 3. 核心 C 代码逻辑拆解分析

### 3.1 串口初始化

```c
#include <termios.h>

int fd = open("/dev/ttymxc0", O_RDWR | O_NOCTTY);

struct termios tty;
tcgetattr(fd, &tty);

// 设置波特率
cfsetospeed(&tty, B115200);
cfsetispeed(&tty, B115200);

// 8 数据位，无校验，1 停止位
tty.c_cflag &= ~PARENB;
tty.c_cflag &= ~CSTOPB;
tty.c_cflag &= ~CSIZE;
tty.c_cflag |= CS8;

// 设置原始模式
tty.c_lflag &= ~(ICANON | ECHO | ECHOE | ISIG);
tty.c_oflag &= ~OPOST;

tcsetattr(fd, TCSANOW, &tty);
```

### 3.2 数据收发

```c
// 发送数据
char *msg = "Hello, UART!\n";
write(fd, msg, strlen(msg));

// 接收数据
char buf[256];
int n = read(fd, buf, sizeof(buf));
if (n > 0) {
    buf[n] = '\0';
    printf("Received: %s\n", buf);
}
```

### 3.3 涉及的 Linux 系统调用与 API

| 系统调用/API | 作用 |
|:---|:---|
| `open()` | 打开串口设备 |
| `tcgetattr()` | 获取串口配置 |
| `tcsetattr()` | 设置串口配置 |
| `cfsetospeed()` | 设置输出波特率 |
| `cfsetispeed()` | 设置输入波特率 |
| `read()` | 读取数据 |
| `write()` | 写入数据 |
| `close()` | 关闭串口 |

## 4. 交叉编译完整操作步骤

```bash
# 进入示例目录
cd 26_uart

# 编译
arm-linux-gnueabihf-gcc -o uart_test uart_test.c
```

## 5. 开发板部署流程

```bash
# 确保串口设备存在
ls /dev/tty*

# 运行程序
./uart_test /dev/ttymxc0
```

## 6. 完整运行验证操作

```bash
# 查看串口设备
ls /dev/ttymxc*

# 运行程序
./uart_test /dev/ttymxc0

# 预期效果
# 串口发送数据并接收返回数据
```

## 7. 常见报错 FAQ

### 7.1 无法打开串口

```bash
# 检查串口设备是否存在
ls -l /dev/ttymxc0

# 检查权限
sudo chmod 666 /dev/ttymxc0
```

### 7.2 数据收发异常

**原因**：波特率或数据格式不匹配。

**解决方案**：
确保收发双方的波特率、数据位、停止位、校验位一致。

### 7.3 乱码

**原因**：串口参数配置错误或硬件连接问题。

**解决方案**：
检查串口接线，确保 TX、RX、GND 连接正确。
