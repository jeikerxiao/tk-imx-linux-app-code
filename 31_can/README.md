# ATK I.MX6ULL - 31_can C APP 示例说明

## 1. 目录功能概述

本示例演示如何在 I.MX6ULL 开发板上使用 SocketCAN 进行 CAN 总线通信。

程序通过 Linux SocketCAN 接口实现 CAN 数据帧的发送和接收。

**适用硬件外设**：CAN 总线接口

**示例文件对应功能**：
- `can_read.c`：CAN 数据接收示例
- `can_write.c`：CAN 数据发送示例

## 2. 目录内源码文件说明

| 文件名 | 类型 | 作用说明 |
|:---|:---|:---|
| can_read.c | C 源文件 | CAN 数据接收示例 |
| can_write.c | C 源文件 | CAN 数据发送示例 |

## 3. 核心 C 代码逻辑拆解分析

### 3.1 CAN 接口初始化

```c
#include <linux/can.h>
#include <linux/can/raw.h>

// 创建 socket
int s = socket(PF_CAN, SOCK_RAW, CAN_RAW);

// 获取 can0 接口索引
struct ifreq ifr;
strcpy(ifr.ifr_name, "can0");
ioctl(s, SIOCGIFINDEX, &ifr);

// 绑定地址
struct sockaddr_can addr;
addr.can_family = AF_CAN;
addr.can_ifindex = ifr.ifr_ifindex;
bind(s, (struct sockaddr *)&addr, sizeof(addr));
```

### 3.2 CAN 数据发送

```c
struct can_frame frame;
frame.can_id = 0x123;
frame.can_dlc = 8;
frame.data[0] = 0x11;
frame.data[1] = 0x22;
// ...
frame.data[7] = 0x88;

write(s, &frame, sizeof(struct can_frame));
```

### 3.3 CAN 数据接收

```c
struct can_frame frame;
read(s, &frame, sizeof(struct can_frame));

printf("ID: 0x%X\n", frame.can_id);
printf("DLC: %d\n", frame.can_dlc);
for (int i = 0; i < frame.can_dlc; i++) {
    printf("Data[%d]: 0x%02X\n", i, frame.data[i]);
}
```

### 3.4 涉及的 Linux 系统调用与 API

| 系统调用/API | 作用 |
|:---|:---|
| `socket()` | 创建 CAN socket |
| `ioctl()` | 获取接口索引 |
| `bind()` | 绑定 CAN 接口 |
| `write()` | 发送 CAN 数据帧 |
| `read()` | 接收 CAN 数据帧 |
| `close()` | 关闭 socket |

## 4. 交叉编译完整操作步骤

```bash
# 进入示例目录
cd 31_can

# 编译接收示例
arm-linux-gnueabihf-gcc -o can_read can_read.c

# 编译发送示例
arm-linux-gnueabihf-gcc -o can_write can_write.c
```

## 5. 开发板部署流程

```bash
# 确保 CAN 接口已启用
ip link set can0 up type can bitrate 500000

# 运行接收示例
./can_read

# 运行发送示例
./can_write
```

## 6. 完整运行验证操作

```bash
# 在开发板 A 上运行接收
./can_read

# 在开发板 B 上运行发送
./can_write

# 预期效果
# 开发板 A 接收并显示 CAN 数据
```

## 7. 常见报错 FAQ

### 7.1 CAN 接口不存在

```bash
# 检查 CAN 接口
ip link show can0

# 启用 CAN 接口
ip link set can0 up type can bitrate 500000
```

### 7.2 发送失败

**原因**：CAN 接口未启用或总线未连接。

**解决方案**：
确保 CAN 接口已启用，并且总线已正确连接。

### 7.3 接收不到数据

**原因**：总线上没有数据或配置错误。

**解决方案**：
```bash
# 检查 CAN 接口状态
cat /proc/net/can/stat

# 使用 candump 工具查看
sudo apt-get install can-utils
candump can0
```
