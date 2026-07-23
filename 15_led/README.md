# ATK I.MX6ULL - 15_led C APP 示例说明

## 1. 目录功能概述

本示例演示如何通过 Linux sysfs 文件系统控制 I.MX6ULL 开发板上的 LED 灯。

程序通过读写 `/sys/class/leds/sys-led/` 下的属性文件，实现 LED 的点亮、熄灭以及触发模式切换功能。

适用于正点原子 I.MX6ULL ALPHA / Mini 开发板，LED 设备节点由内核 LED 子系统驱动提供。

**适用硬件外设**：I.MX6ULL 开发板板载 LED（通常为红色或绿色系统指示灯）

## 2. 目录内源码文件说明

| 文件名 | 类型 | 作用说明 |
|:---|:---|:---|
| led.c | C 源文件 | LED 控制主程序，解析命令行参数并操作 sysfs 属性文件 |

**无 Makefile 和配置文件**，直接使用交叉编译器编译单个 C 文件即可。

## 3. 核心 C 代码逻辑拆解分析

### 3.1 关键宏定义与设备路径

```c
#define  LED_TRIGGER    "/sys/class/leds/sys-led/trigger"
#define  LED_BRIGHTNESS "/sys/class/leds/sys-led/brightness"
```

- `LED_TRIGGER`：LED 触发模式属性文件，可写入 `none`、`heartbeat`、`timer` 等触发方式
- `LED_BRIGHTNESS`：LED 亮度属性文件，写入 `0` 表示熄灭，`1` 表示点亮

### 3.2 文件打开与错误处理

```c
fd1 = open(LED_TRIGGER, O_RDWR);
if (0 > fd1) {
    perror("open error");
    exit(-1);
}
```

使用标准 POSIX `open()` 系统调用打开 sysfs 属性文件，返回文件描述符。如打开失败，通过 `perror()` 打印错误信息并退出。

### 3.3 LED 控制逻辑

```c
if (!strcmp(argv[1], "on")) {
    write(fd1, "none", 4);   // 先将触发模式设置为 none
    write(fd2, "1", 1);      // 点亮 LED
}
else if (!strcmp(argv[1], "off")) {
    write(fd1, "none", 4);   // 先将触发模式设置为 none
    write(fd2, "0", 1);      // LED 灭
}
else if (!strcmp(argv[1], "trigger")) {
    write(fd1, argv[2], strlen(argv[2]));  // 设置触发模式
}
```

- **点亮/熄灭**：先设置触发模式为 `none`（手动控制），再写入亮度值
- **触发模式切换**：支持 `heartbeat`（心跳闪烁）、`timer`（定时闪烁）等内核内置触发器

### 3.4 涉及的 Linux 系统调用

| 系统调用 | 作用 | IO 模型 |
|:---|:---|:---|
| `open()` | 打开 sysfs 属性文件 | 阻塞 IO |
| `write()` | 向属性文件写入控制字符串 | 阻塞 IO |
| `close()` | 关闭文件描述符 | - |

## 4. 交叉编译完整操作步骤

```bash
# 进入示例目录
cd 15_led

# 使用交叉编译器编译
arm-linux-gnueabihf-gcc -o led led.c

# 验证生成的可执行文件架构
file led
# 输出应包含：ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked
```

## 5. 开发板部署流程

### 5.1 SCP 传输方式

```bash
# 主机端执行，将编译好的程序上传到开发板
scp led root@192.168.1.50:/home/root/

# 开发板端执行，赋予执行权限
chmod +x /home/root/led
```

### 5.2 NFS 挂载方式

```bash
# 开发板端执行，挂载 NFS 共享目录
mount -t nfs -o nolock,tcp 192.168.1.100:/home/alientek/nfsroot /mnt

# 进入挂载目录
cd /mnt/15_led
chmod +x led
```

### 5.3 后台运行配置

```bash
# 前台运行（阻塞终端）
./led on

# 后台运行
./led on &

# 开机自启配置（可选）
# 编辑 /etc/rc.local，在 exit 0 之前添加
/home/root/led on
```

## 6. 完整运行验证操作

### 6.1 测试命令

```bash
# 点亮 LED
./led on

# 熄灭 LED
./led off

# 设置为心跳闪烁模式
./led trigger heartbeat

# 设置为定时器闪烁模式
./led trigger timer
```

### 6.2 预期输出效果

- 执行 `./led on` 后，开发板板载 LED 常亮
- 执行 `./led off` 后，LED 熄灭
- 执行 `./led trigger heartbeat` 后，LED 以心跳节奏闪烁（快闪两次，停顿）
- 执行 `./led trigger timer` 后，LED 按固定频率闪烁

### 6.3 查看当前触发模式

```bash
# 查看当前可用的触发模式及激活状态
cat /sys/class/leds/sys-led/trigger
# 输出示例：none [heartbeat] timer mmc0
# 方括号内的为当前激活的模式
```

## 7. 常见报错 FAQ

### 7.1 编译报错：找不到头文件

**现象**：
```
led.c:1:10: fatal error: stdio.h: No such file or directory
```

**解决方案**：
确保已正确安装交叉编译器并配置环境变量：
```bash
export PATH=$PATH:/opt/gcc-linaro-5.4.1-2017.05-x86_64_arm-linux-gnueabihf/bin
```

### 7.2 运行时找不到设备节点

**现象**：
```
open error: No such file or directory
```

**原因**：内核未启用 LED 子系统驱动，或设备树中未配置 `sys-led` 节点。

**解决方案**：
```bash
# 检查 sysfs 目录下是否存在 sys-led
ls /sys/class/leds/

# 如不存在，需在内核配置中启用 LED 支持
make menuconfig
# Device Drivers -> LED Support -> LED Class Support (选中)
# 重新编译内核并烧录
```

### 7.3 权限不足

**现象**：
```
open error: Permission denied
```

**解决方案**：
```bash
# 使用 root 用户运行
sudo ./led on

# 或修改设备节点权限
chmod 666 /sys/class/leds/sys-led/trigger
chmod 666 /sys/class/leds/sys-led/brightness
```

### 7.4 段错误（Segmentation fault）

**现象**：
```
Segmentation fault (core dumped)
```

**原因**：传参不正确，`argv[1]` 为空导致 `strcmp()` 访问非法内存。

**解决方案**：
确保传入正确的参数：
```bash
./led on    # 正确
./led       # 错误，缺少参数
```

### 7.5 LED 无响应

**现象**：执行命令后 LED 无任何变化。

**排查步骤**：
1. 确认 LED 硬件是否正常：`dmesg | grep led`
2. 确认触发模式是否被其他进程占用：`cat /sys/class/leds/sys-led/trigger`
3. 尝试手动写入测试：`echo 1 > /sys/class/leds/sys-led/brightness`
