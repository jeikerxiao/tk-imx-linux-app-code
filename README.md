# 正点原子 I.MX6ULL Linux C APP 综合示例工程

## 1. 项目整体概述

本工程为正点原子（ATK）I.MX6ULL 开发板配套的纯用户态 Linux C 上层应用示例源码集合，不包含任何内核驱动代码。示例覆盖基础 IO 控制、GPIO 操作、Input 输入子系统、触摸屏、LCD 显示、图像编解码、音频采集与播放、串口通信、摄像头采集、网络 Socket 通信、CAN 总线以及 MQTT 物联网通信等嵌入式 Linux 应用开发核心技术领域。

所有示例均基于 Linux 标准系统调用和 POSIX API 实现，适配正点原子 I.MX6ULL 开发板硬件环境，可直接用于学习、验证和二次开发。

| 项目属性 | 说明 |
|---------|------|
| 芯片平台 | NXP I.MX6ULL（Cortex-A7，单核 792MHz） |
| 操作系统 | Linux 4.x / 5.x |
| 编译环境 | ATK I.MX6ULL 交叉编译器（arm-linux-gnueabihf-gcc） |
| 开发板 | 正点原子 I.MX6ULL ALPHA / Mini 开发板 |
| 代码定位 | 纯用户态 Linux C 应用，不涉及内核驱动开发 |

## 2. 开发环境前置准备

### 2.1 交叉编译器安装

推荐使用正点原子提供的交叉编译器工具链：

```bash
# 下载并解压交叉编译器到 /opt 目录
sudo tar -xvzf gcc-linaro-5.4.1-2017.05-x86_64_arm-linux-gnueabihf.tar.gz -C /opt/

# 配置环境变量，编辑 ~/.bashrc，追加以下内容
export PATH=$PATH:/opt/gcc-linaro-5.4.1-2017.05-x86_64_arm-linux-gnueabihf/bin
export CROSS_COMPILE=arm-linux-gnueabihf-
export ARCH=arm

# 使环境变量生效
source ~/.bashrc

# 验证交叉编译器版本
arm-linux-gnueabihf-gcc --version
```

### 2.2 内核源码导出头文件

```bash
# 在内核源码目录执行，导出编译所需头文件
make ARCH=arm headers_install INSTALL_HDR_PATH=/path/to/headers

# 编译时通过 -I 参数指定头文件路径
arm-linux-gnueabihf-gcc -I/path/to/headers/include ...
```

### 2.3 NFS 共享环境搭建

```bash
# Ubuntu/Debian 安装 NFS 服务
sudo apt-get install nfs-kernel-server

# 编辑 /etc/exports，添加共享目录（假设共享目录为 /home/alientek/nfsroot）
/home/alientek/nfsroot *(rw,sync,no_root_squash)

# 重启 NFS 服务
sudo exportfs -a
sudo systemctl restart nfs-kernel-server

# 开发板端挂载 NFS
mount -t nfs -o nolock,tcp 192.168.1.100:/home/alientek/nfsroot /mnt
```

### 2.4 开发板网络配置

```bash
# 配置静态 IP（开发板端执行）
ifconfig eth0 192.168.1.50 netmask 255.255.255.0

# 或者通过 DHCP 自动获取
udhcpc -i eth0
```

## 3. 工程一级目录总览

| 序号 | APP 示例模块名称 | 详细文档入口 |
|:---:|:---|:---|
| 1 | LED 控制（sysfs） | [点击查看完整文档](./15_led/README.md) |
| 2 | GPIO 输入输出与中断 | [点击查看完整文档](./16_gpio/README.md) |
| 3 | Input 输入子系统 | [点击查看完整文档](./17_input/README.md) |
| 4 | TSlib 触摸屏库 | [点击查看完整文档](./18_tslib/README.md) |
| 5 | LCD 显示（FrameBuffer） | [点击查看完整文档](./19_lcd/README.md) |
| 6 | libjpeg 图像显示 | [点击查看完整文档](./20_libjpeg/README.md) |
| 7 | libpng 图像显示 | [点击查看完整文档](./21_libpng/README.md) |
| 8 | LCD 竖屏显示 | [点击查看完整文档](./22_lcd_vertical_display/README.md) |
| 9 | FreeType 字体渲染 | [点击查看完整文档](./23_freetype/README.md) |
| 10 | PWM 控制（sysfs） | [点击查看完整文档](./24_pwm/README.md) |
| 11 | V4L2 摄像头采集 | [点击查看完整文档](./25_v4l2_camera/README.md) |
| 12 | UART 串口通信 | [点击查看完整文档](./26_uart/README.md) |
| 13 | Watchdog 看门狗 | [点击查看完整文档](./27_watchdog/README.md) |
| 14 | ALSA 音频采集与播放 | [点击查看完整文档](./28_alsa-lib/README.md) |
| 15 | Socket 网络通信 | [点击查看完整文档](./30_socket/README.md) |
| 16 | CAN 总线通信 | [点击查看完整文档](./31_can/README.md) |
| 17 | MQTT 客户端 | [点击查看完整文档](./33_mqtt/README.md) |

## 4. 通用交叉编译流程

### 4.1 单个 C 文件编译

```bash
# 使用交叉编译器直接编译单个 C 文件
arm-linux-gnueabihf-gcc -o led led.c

# 带调试信息的编译
arm-linux-gnueabihf-gcc -g -o led led.c

# 优化级别设置（-O0 关闭优化，-O2 标准优化）
arm-linux-gnueabihf-gcc -O2 -o led led.c
```

### 4.2 链接第三方库

```bash
# 链接动态库（以 libjpeg 为例）
arm-linux-gnueabihf-gcc -o show_jpeg show_jpeg.c -ljpeg

# 指定库路径和头文件路径
arm-linux-gnuabihf-gcc -o show_jpeg show_jpeg.c \
    -I/home/alientek/tools/jpeg/include \
    -L/home/alientek/tools/jpeg/lib \
    -ljpeg
```

### 4.3 清理编译产物

```bash
# 删除可执行文件和中间文件
rm -f led *.o

# 或者使用 Makefile 的 clean 目标
make clean
```

## 5. 开发板通用运行部署方法

### 5.1 NFS 挂载运行

```bash
# 开发板端执行，挂载 NFS 共享目录
mount -t nfs -o nolock,tcp 192.168.1.100:/home/alientek/nfsroot /mnt

# 进入挂载目录并运行程序
cd /mnt
chmod +x led
./led on
```

### 5.2 SCP 上传文件

```bash
# 主机端执行，将编译好的程序上传到开发板
scp led root@192.168.1.50:/home/root/

# 开发板端执行，赋予执行权限并运行
chmod +x /home/root/led
/home/root/led on
```

### 5.3 后台运行与日志查看

```bash
# 后台运行程序
./uart_test --dev=/dev/ttymxc2 --type=read &

# 查看后台进程
ps | grep uart_test

# 查看系统日志
dmesg | tail -20

# 重定向输出到日志文件
./led on > /tmp/led.log 2>&1
```

## 6. 统一调试手段

### 6.1 dmesg 查看底层设备信息

```bash
# 查看内核日志，排查设备驱动加载情况
dmesg | grep -i "led\|gpio\|input"

# 实时监控内核日志
dmesg -w
```

### 6.2 GDB 远程调试

```bash
# 开发板端启动 gdbserver
gdbserver :2345 ./led on

# 主机端使用交叉 GDB 连接
cross-gdb ./led
(gdb) target remote 192.168.1.50:2345
(gdb) continue
```

### 6.3 strace 追踪系统调用

```bash
# 安装 strace（如未安装）
apt-get install strace

# 追踪程序的系统调用
strace -o trace.log ./led on

# 只追踪特定的系统调用
strace -e open,read,write ./led on
```

### 6.4 printf 打印调试

```bash
# 开启程序内的调试打印（如支持）
./app -v

# 使用 stderr 输出调试信息
fprintf(stderr, "debug: fd=%d\n", fd);
```

## 7. 工程使用注意事项

1. **设备节点权限**：部分示例需要 root 权限才能访问 `/dev/fb0`、`/dev/input/eventX` 等设备节点，运行前请确保当前用户有相应权限，或使用 `sudo` 执行。

2. **32 位 ARM 编译**：所有示例均面向 32 位 ARM 架构（armv7-a）编译，请确保交叉编译器前缀为 `arm-linux-gnueabihf-`。

3. **用户态无法操作硬件寄存器**：本工程所有示例均通过 Linux 标准接口（sysfs、设备文件、库函数）操作硬件，不直接访问物理寄存器地址。如需直接操作寄存器，需在内核空间编写驱动程序。

4. **区分 APP 与内核驱动开发边界**：
   - 用户态 APP：通过 `open()`、`read()`、`write()`、`ioctl()` 等系统调用与内核交互
   - 内核态驱动：直接操作寄存器、处理中断、管理内存
   - 本工程仅涉及用户态 APP 开发，不涉及内核驱动

5. **第三方库依赖**：部分示例（如 libjpeg、libpng、tslib、alsa-lib）需要先在开发板上安装对应的库文件，运行前请确保相关动态库已正确部署到 `/lib` 或 `/usr/lib` 目录。

6. **设备树匹配**：所有设备节点路径（如 `/dev/fb0`、`/dev/input/event0`、`/dev/ttymxc2`）均基于正点原子 I.MX6ULL 出厂设备树配置，如使用自定义设备树，请根据实际配置修改代码中的设备路径。