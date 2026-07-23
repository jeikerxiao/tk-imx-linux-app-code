# ATK I.MX6ULL - 17_input C APP 示例说明

## 1. 目录功能概述

本示例演示 Linux Input 输入子系统的应用编程，涵盖输入事件的读取、按键检测、多点触摸以及触摸屏坐标解析等核心功能。

程序通过读取 `/dev/input/eventX` 设备节点，获取来自键盘、触摸屏等输入设备的事件数据。

适用于正点原子 I.MX6ULL 开发板配套的触摸屏及按键设备。

**适用硬件外设**：电容触摸屏、物理按键

**示例文件对应功能**：
- `read_input.c`：基础输入事件读取，打印所有输入事件
- `read_key.c`：按键事件解析，识别按下、松开、长按三种状态
- `read_slot.c`：查询触摸屏支持的最多触摸点数（slot 数量）
- `read_ts.c`：单点触摸坐标解析，打印触摸、移动、松开事件
- `read_mt.c`：多点触摸（Multi-Touch）坐标解析，支持多个触摸点同时检测

## 2. 目录内源码文件说明

| 文件名 | 类型 | 作用说明 |
|:---|:---|:---|
| read_input.c | C 源文件 | 基础输入事件读取示例，循环读取并打印所有输入事件 |
| read_key.c | C 源文件 | 按键事件处理示例，解析 EV_KEY 事件并判断按下/松开/长按 |
| read_slot.c | C 源文件 | 查询触摸屏支持的最大触摸点数 |
| read_ts.c | C 源文件 | 单点触摸坐标解析，处理 EV_ABS 和 EV_SYN 事件 |
| read_mt.c | C 源文件 | 多点触摸坐标解析，支持多手指同时触摸检测 |

## 3. 核心 C 代码逻辑拆解分析

### 3.1 输入事件数据结构

```c
struct input_event in_ev = {0};
```

`input_event` 结构体定义（`<linux/input.h>`）：
```c
struct input_event {
    struct timeval time;   // 事件发生的时间戳
    __u16 type;            // 事件类型（EV_KEY、EV_ABS、EV_SYN 等）
    __u16 code;            // 事件编码（按键码、ABS 坐标码等）
    __s32 value;           // 事件值（按键状态、坐标值等）
};
```

### 3.2 打开输入设备

```c
fd = open(argv[1], O_RDONLY);  // 以只读方式打开输入设备节点
```

设备节点通常为 `/dev/input/event0`、`/dev/input/event1` 等，对应不同的输入设备。可通过 `cat /proc/bus/input/devices` 查看设备映射关系。

### 3.3 基础输入事件读取（read_input.c）

```c
for (;;) {
    if (sizeof(struct input_event) !=
        read(fd, &in_ev, sizeof(struct input_event))) {
        perror("read error");
        exit(-1);
    }
    printf("type:%d code:%d value:%d\n",
            in_ev.type, in_ev.code, in_ev.value);
}
```

循环读取 `input_event` 结构体，打印事件类型、编码和值。

### 3.4 按键事件解析（read_key.c）

```c
if (EV_KEY == in_ev.type) {  // 按键事件
    switch (in_ev.value) {
    case 0:
        printf("code<%d>: 松开\n", in_ev.code);
        break;
    case 1:
        printf("code<%d>: 按下\n", in_ev.code);
        break;
    case 2:
        printf("code<%d>: 长按\n", in_ev.code);
        break;
    }
}
```

- `value=0`：按键松开
- `value=1`：按键按下
- `value=2`：按键长按（重复触发）

### 3.5 单点触摸解析（read_ts.c）

```c
case EV_ABS:  // 绝对坐标事件
    switch (in_ev.code) {
    case ABS_MT_TRACKING_ID:
        // 触摸点 ID，0 表示按下，-1 表示松开
    case ABS_MT_POSITION_X:
        x = in_ev.value;  // X 坐标
    case ABS_MT_POSITION_Y:
        y = in_ev.value;  // Y 坐标
    }
    break;
case EV_SYN:  // 同步事件
    if (SYN_REPORT == in_ev.code) {
        // 数据包同步，解析完整的触摸事件
    }
    break;
```

### 3.6 多点触摸解析（read_mt.c）

```c
// 获取触摸屏支持的最大触摸点数
ioctl(fd, EVIOCGABS(ABS_MT_SLOT), &slot);
max_slots = slot.maximum + 1 - slot.minimum;

// 解析 ABS_MT_SLOT 获取当前触摸点索引
// 解析 ABS_MT_POSITION_X/Y 获取各点坐标
// 解析 ABS_MT_TRACKING_ID 判断触摸点创建/删除
```

### 3.7 涉及的 Linux 系统调用与 API

| 系统调用/API | 作用 | IO 模型 |
|:---|:---|:---|
| `open()` | 打开输入设备节点 | 阻塞 IO |
| `read()` | 读取输入事件数据 | 阻塞 IO |
| `close()` | 关闭设备 | - |
| `ioctl()` | 获取输入设备信息 | - |
| `EVIOCGABS()` | 获取 ABS 类参数 | - |

## 4. 交叉编译完整操作步骤

```bash
# 进入示例目录
cd 17_input

# 编译所有示例
arm-linux-gnueabihf-gcc -o read_input read_input.c
arm-linux-gnueabihf-gcc -o read_key read_key.c
arm-linux-gnueabihf-gcc -o read_slot read_slot.c
arm-linux-gnueabihf-gcc -o read_ts read_ts.c
arm-linux-gnueabihf-gcc -o read_mt read_mt.c
```

## 5. 开发板部署流程

```bash
# SCP 传输
scp read_input read_key read_slot read_ts read_mt root@192.168.1.50:/home/root/

# 或 NFS 挂载运行
mount -t nfs -o nolock,tcp 192.168.1.100:/home/alientek/nfsroot /mnt
cd /mnt/17_input
chmod +x read_*
```

## 6. 完整运行验证操作

### 6.1 查看输入设备映射

```bash
# 列出所有输入设备
cat /proc/bus/input/devices

# 或使用 evtest 工具查看
# apt-get install evtest
evtest /dev/input/event0
```

### 6.2 基础输入测试

```bash
# 读取输入设备事件（替换为实际的设备节点）
./read_input /dev/input/event0

# 预期输出（触摸屏幕时）
type:3 code:47 value:0
type:3 code:53 value:300
type:3 code:54 value:200
type:0 code:0 value:0
```

### 6.3 按键测试

```bash
./read_key /dev/input/event0

# 按下按键时输出
code<103>: 按下

# 松开按键时输出
code<103>: 松开

# 长按时输出
code<103>: 长按
```

### 6.4 触摸坐标测试

```bash
./read_ts /dev/input/event0

# 按下屏幕
按下(300, 200)

# 移动手指
移动(320, 210)

# 松开手指
松开
```

## 7. 常见报错 FAQ

### 7.1 设备节点不存在

```bash
# 检查是否有输入设备节点
ls /dev/input/event*

# 如不存在，检查内核是否启用了输入子系统
# make menuconfig -> Device Drivers -> Input device support
```

### 7.2 读取数据失败

**原因**：缓冲区大小不匹配或设备被其他进程占用。

**解决方案**：
```bash
# 检查是否有其他进程占用该设备
lsof /dev/input/event0

# 确保每次读取的 size 与 input_event 结构体大小一致
```

### 7.3 多点触摸无法识别

**原因**：触摸屏驱动不支持多点触摸，或设备树配置错误。

**排查步骤**：
```bash
# 检查触摸屏驱动是否加载
lsmod | grep -i touch

# 查看触摸屏支持的最大触摸点数
./read_slot /dev/input/event0
# 预期输出：max_slots: 5

# 检查内核是否启用了多点触摸支持
# CONFIG_TOUCHSCREEN_FT5X06 等
```
