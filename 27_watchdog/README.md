# ATK I.MX6ULL - 27_watchdog C APP 示例说明

## 1. 目录功能概述

本示例演示如何在 I.MX6ULL 开发板上使用看门狗（Watchdog）。

程序通过操作 `/dev/watchdog` 设备节点，实现对系统运行的监控，防止程序死锁或系统崩溃。

**适用硬件外设**：I.MX6ULL 内置看门狗

**示例文件对应功能**：
- `watchdog_test.c`：看门狗测试主程序

## 2. 目录内源码文件说明

| 文件名 | 类型 | 作用说明 |
|:---|:---|:---|
| watchdog_test.c | C 源文件 | 看门狗操作示例 |

## 3. 核心 C 代码逻辑拆解分析

### 3.1 看门狗初始化

```c
#include <linux/watchdog.h>

int fd = open("/dev/watchdog", O_RDWR);
if (fd < 0) {
    perror("open watchdog failed");
    return -1;
}

// 设置超时时间（秒）
int timeout = 10;
ioctl(fd, WDIOC_SETTIMEOUT, &timeout);
```

### 3.2 喂狗操作

```c
// 定时喂狗
while (1) {
    // 执行正常任务
    do_something();

    // 喂狗
    ioctl(fd, WDIOC_KEEPALIVE, 0);
    // 或 write(fd, "\0", 1);

    sleep(5);  // 喂狗间隔必须小于超时时间
}
```

### 3.3 涉及的 Linux 系统调用与 API

| 系统调用/API | 作用 |
|:---|:---|
| `open()` | 打开看门狗设备 |
| `ioctl()` | 看门狗控制命令 |
| `write()` | 喂狗操作 |
| `close()` | 关闭设备（关闭后看门狗停止） |

## 4. 交叉编译完整操作步骤

```bash
# 进入示例目录
cd 27_watchdog

# 编译
arm-linux-gnueabihf-gcc -o watchdog_test watchdog_test.c
```

## 5. 开发板部署流程

```bash
# 确保看门狗设备存在
ls -l /dev/watchdog

# 运行程序
./watchdog_test
```

## 6. 完整运行验证操作

```bash
# 查看看门狗设备
ls -l /dev/watchdog

# 运行程序
./watchdog_test

# 预期效果
# 程序正常运行，系统不会重启
# 如果停止喂狗，系统会在超时后自动重启
```

## 7. 常见报错 FAQ

### 7.1 无法打开看门狗

```bash
# 检查设备节点
ls -l /dev/watchdog

# 检查内核配置
# CONFIG_WATCHDOG 必须启用
```

### 7.2 看门狗不起作用

**原因**：看门狗超时时间设置过长或喂狗间隔过短。

**解决方案**：
确保喂狗间隔小于超时时间。

### 7.3 系统意外重启

**原因**：程序未正确喂狗。

**解决方案**：
确保程序在正常运行时定期喂狗。
