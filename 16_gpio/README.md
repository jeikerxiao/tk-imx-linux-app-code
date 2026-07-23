# ATK I.MX6ULL - 16_gpio C APP 示例说明

## 1. 目录功能概述

本示例演示如何通过 Linux sysfs 文件系统对 I.MX6ULL 开发板的 GPIO 进行应用编程，涵盖 GPIO 输入、输出和中断三种典型使用场景。

程序通过读写 `/sys/class/gpio/` 目录下的属性文件，实现 GPIO 引脚的导出、方向配置、电平控制以及中断检测。

**适用硬件外设**：I.MX6ULL 开发板任意可用的 GPIO 引脚（如 KEY0、LED 等）

**示例文件对应功能**：
- `gpio_in.c`：GPIO 输入模式，读取引脚电平状态
- `gpio_out.c`：GPIO 输出模式，控制引脚输出高低电平
- `gpio_intr.c`：GPIO 中断检测，使用 poll 机制监听引脚状态变化

## 2. 目录内源码文件说明

| 文件名 | 类型 | 作用说明 |
|:---|:---|:---|
| gpio_in.c | C 源文件 | GPIO 输入示例，导出指定编号 GPIO 并读取其电平值 |
| gpio_out.c | C 源文件 | GPIO 输出示例，导出指定编号 GPIO 并设置输出电平 |
| gpio_intr.c | C 源文件 | GPIO 中断示例，配置引脚中断触发方式并使用 poll 监听 |

**无 Makefile 和配置文件**，直接使用交叉编译器编译即可。

## 3. 核心 C 代码逻辑拆解分析

### 3.1 GPIO 导出机制

```c
sprintf(gpio_path, "/sys/class/gpio/gpio%s", argv[1]);

if (access(gpio_path, F_OK)) {  // 目录不存在则导出
    fd = open("/sys/class/gpio/export", O_WRONLY);
    write(fd, argv[1], strlen(argv[1]));  // 写入 GPIO 编号
    close(fd);
}
```

通过向 `/sys/class/gpio/export` 写入 GPIO 编号完成引脚导出，导出后会在 `/sys/class/gpio/` 下生成对应编号的目录（如 `gpio1`）。

### 3.2 属性文件配置函数

```c
static int gpio_config(const char *attr, const char *val)
{
    char file_path[100];
    int fd;

    sprintf(file_path, "%s/%s", gpio_path, attr);
    fd = open(file_path, O_WRONLY);
    write(fd, val, strlen(val));
    close(fd);
    return 0;
}
```

封装了统一的属性文件写入函数，通过 `attr` 参数指定属性名（`direction`、`value`、`edge` 等），`val` 参数指定写入值。

### 3.3 GPIO 输入（gpio_in.c）

```c
// 配置为输入模式
gpio_config("direction", "in");
// 极性设置
gpio_config("active_low", "0");
// 配置为非中断方式
gpio_config("edge", "none");
// 读取电平值
read(fd, &val, 1);
printf("value: %c\n", val);
```

**关键属性说明**：
- `direction`：配置引脚方向，`in` 为输入，`out` 为输出
- `active_low`：极性设置，`0` 表示高电平有效，`1` 表示低电平有效
- `edge`：中断触发方式，`none` 为无中断，`rising` 为上升沿，`falling` 为下降沿，`both` 为双边沿

### 3.4 GPIO 输出（gpio_out.c）

```c
// 配置为输出模式
gpio_config("direction", "out");
// 控制输出电平
gpio_config("value", argv[2]);  // "0" 或 "1"
```

### 3.5 GPIO 中断（gpio_intr.c）

```c
// 配置为输入模式
gpio_config("direction", "in");
// 配置中断触发方式：双边沿
gpio_config("edge", "both");

// 使用 poll 监听中断事件
pfd.events = POLLPRI;  // 只关心高优先级数据可读（中断）
read(pfd.fd, &val, 1);  // 先读取一次清除状态
for (;;) {
    ret = poll(&pfd, 1, -1);  // 阻塞等待中断
    if (pfd.revents & POLLPRI) {
        lseek(pfd.fd, 0, SEEK_SET);  // 重置读位置
        read(pfd.fd, &val, 1);       // 读取当前电平
        printf("GPIO 中断触发<value=%c>\n", val);
    }
}
```

**poll 机制说明**：
- `POLLPRI`：高优先级数据可读，sysfs GPIO 中断的标准通知方式
- `lseek(pfd.fd, 0, SEEK_SET)`：每次读取前需将文件指针重置到开头，否则 poll 不会再次触发
- 阻塞 IO 模型，无中断时进程进入睡眠状态，不消耗 CPU

### 3.6 涉及的 Linux 系统调用与 API

| 系统调用/API | 作用 | IO 模型 |
|:---|:---|:---|
| `open()` | 打开 sysfs 属性文件 | 阻塞 IO |
| `write()` | 向属性文件写入配置 | 阻塞 IO |
| `read()` | 读取属性文件值 | 阻塞 IO |
| `close()` | 关闭文件描述符 | - |
| `poll()` | I/O 多路复用，监听中断事件 | 阻塞 IO |
| `lseek()` | 重置文件读位置 | - |
| `access()` | 检查文件/目录是否存在 | - |

## 4. 交叉编译完整操作步骤

```bash
# 进入示例目录
cd 16_gpio

# 分别编译三个示例程序
arm-linux-gnueabihf-gcc -o gpio_in gpio_in.c
arm-linux-gnueabihf-gcc -o gpio_out gpio_out.c
arm-linux-gnueabihf-gcc -o gpio_intr gpio_intr.c

# 验证生成的可执行文件
file gpio_in gpio_out gpio_intr
```

## 5. 开发板部署流程

### 5.1 SCP 传输方式

```bash
# 主机端执行
scp gpio_in gpio_out gpio_intr root@192.168.1.50:/home/root/

# 开发板端执行
chmod +x /home/root/gpio_in /home/root/gpio_out /home/root/gpio_intr
```

### 5.2 NFS 挂载方式

```bash
# 开发板端执行
mount -t nfs -o nolock,tcp 192.168.1.100:/home/alientek/nfsroot /mnt
cd /mnt/16_gpio
chmod +x gpio_in gpio_out gpio_intr
```

## 6. 完整运行验证操作

### 6.1 GPIO 输入测试（gpio_in）

```bash
# 导出 GPIO1（假设为按键引脚）并读取当前电平
./gpio_in 1

# 预期输出（按键未按下时为高电平）
value: 1

# 按键按下后再次执行
./gpio_in 1
# 预期输出
value: 0
```

### 6.2 GPIO 输出测试（gpio_out）

```bash
# 导出 GPIO5 并输出高电平
./gpio_out 5 1
# 连接 GPIO5 的 LED 应点亮

# 导出 GPIO5 并输出低电平
./gpio_out 5 0
# LED 应熄灭
```

### 6.3 GPIO 中断测试（gpio_intr）

```bash
# 导出 GPIO1 并配置为双边沿中断
./gpio_intr 1

# 按下按键，预期输出
GPIO 中断触发<value=0>

// 松开按键，预期输出
GPIO 中断触发<value=1>
```

**注意**：`gpio_intr` 程序会一直阻塞运行，按 `Ctrl+C` 终止。

### 6.4 GPIO 编号查询

```bash
# 查看当前已导出的 GPIO
cat /sys/class/gpio/gpio*/value

# 查看 GPIO 编号对应的物理引脚
# 参考正点原子 I.MX6ULL 开发板原理图或设备树定义
```

## 7. 常见报错 FAQ

### 7.1 设备节点不存在

**现象**：
```
open error: No such file or directory
```

**原因**：
1. GPIO 编号错误，该编号对应的 sysfs 目录不存在
2. 内核未启用 GPIO sysfs 接口支持

**解决方案**：
```bash
# 检查 sysfs 目录下是否有对应编号的 GPIO
cat /sys/kernel/debug/gpio

# 手动导出测试
echo 1 > /sys/class/gpio/export
ls /sys/class/gpio/gpio1/

# 如手动导出成功，检查 APP 是否有写入 /sys/class/gpio/export 的权限
```

### 7.2 权限不足

**现象**：
```
open error: Permission denied
```

**解决方案**：
```bash
# 以 root 用户运行
sudo ./gpio_out 5 1

# 修改 GPIO 目录权限（不推荐长期使用）
chmod -R 666 /sys/class/gpio/gpio*
```

### 7.3 GPIO 中断无响应

**现象**：执行 `gpio_intr` 后，按键按下/松开无中断输出。

**排查步骤**：
1. 确认 GPIO 已正确导出并配置为输入模式：
   ```bash
   cat /sys/class/gpio/gpio1/direction  # 应为 in
   cat /sys/class/gpio/gpio1/edge       # 应为 both
   ```
2. 确认按键硬件连接正确
3. 确认内核已启用 GPIO 中断支持：
   ```bash
   dmesg | grep -i gpio
   ```
4. 检查是否有其他程序占用该 GPIO

### 7.4 poll 返回超时

**现象**：
```
poll timeout.
```

**原因**：中断未触发或触发方式配置错误。

**解决方案**：
确保 `edge` 属性配置为 `rising`、`falling` 或 `both`：
```bash
echo both > /sys/class/gpio/gpio1/edge
```

### 7.5 段错误（Segmentation fault）

**现象**：
```
Segmentation fault (core dumped)
```

**原因**：命令行参数传入错误，如 `argv[1]` 为空或 GPIO 编号超出范围。

**解决方案**：
确保传入正确的参数：
```bash
./gpio_in 1    # 正确
./gpio_in      # 错误，缺少 GPIO 编号

./gpio_out 5 1  # 正确
./gpio_out 5    # 错误，缺少输出值
```
