# ATK I.MX6ULL - 19_lcd C APP 示例说明

## 1. 目录功能概述

本示例演示 Linux FrameBuffer（帧缓冲）应用编程，涵盖 LCD 信息显示、基本图形绘制、BMP 图像显示等。

程序通过操作 `/dev/fb0` 设备节点，实现对 LCD 屏幕的直接控制和内容显示。

适用于正点原子 I.MX6ULL 开发板配套的 LCD 显示屏。

**适用硬件外设**：LCD 显示屏（通常为 4.3 寸或 7 寸 RGB LCD，分辨率 800x480 或 1024x600）

**示例文件对应功能**：
- `lcd_info.c`：获取并显示 LCD 屏幕的分辨率、像素格式、行字节数等信息
- `lcd_test.c`：FrameBuffer 基本图形绘制示例，包含画点、画线、画矩形、填充颜色等
- `bmp_show.c`：BMP 图像文件解析与显示
- `image.bmp`：测试用的 BMP 图像文件

## 2. 目录内源码文件说明

| 文件名 | 类型 | 作用说明 |
|:---|:---|:---|
| lcd_info.c | C 源文件 | 读取 FrameBuffer 设备信息并打印 |
| lcd_test.c | C 源文件 | FrameBuffer 基本绘图示例 |
| bmp_show.c | C 源文件 | BMP 图像解析与显示 |
| image.bmp | 二进制文件 | 测试用 BMP 图像 |

## 3. 核心 C 代码逻辑拆解分析

### 3.1 FrameBuffer 设备操作基础

```c
// 打开 FrameBuffer 设备
fd = open("/dev/fb0", O_RDWR);

// 获取可变和固定屏幕信息
ioctl(fd, FBIOGET_VSCREENINFO, &fb_var);
ioctl(fd, FBIOGET_FSCREENINFO, &fb_fix);

// 将显示缓冲区映射到进程地址空间
screen_base = mmap(NULL, screen_size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
```

### 3.2 LCD 信息显示（lcd_info.c）

```c
struct fb_fix_screeninfo fb_fix;
struct fb_var_screeninfo fb_var;

ioctl(fd, FBIOGET_VSCREENINFO, &fb_var);
ioctl(fd, FBIOGET_FSCREENINFO, &fb_fix);

printf("分辨率: %d*%d\n", fb_var.xres, fb_var.yres);
printf("像素深度 bpp: %d\n", fb_var.bits_per_pixel);
printf("一行字节数: %d\n", fb_fix.line_length);
```

### 3.3 基本图形绘制（lcd_test.c）

```c
// 画点
screen_base[y * width + x] = color;

// 填充矩形区域
for (y = start_y; y <= end_y; y++) {
    for (x = start_x; x <= end_x; x++) {
        screen_base[y * width + x] = color;
    }
}
```

### 3.4 BMP 图像显示（bmp_show.c）

```c
// 解析 BMP 文件头
struct bmp_header header;
read(fd_bmp, &header, sizeof(header));

// 计算显示位置
for (y = 0; y < height; y++) {
    read(fd_bmp, buffer, width * bpp);
    memcpy(screen_base + y * width, buffer, width * bpp);
}
```

### 3.5 涉及的 Linux 系统调用与 API

| 系统调用/API | 作用 |
|:---|:---|
| `open()` | 打开 FrameBuffer 设备 |
| `ioctl()` | 获取屏幕信息 |
| `mmap()` | 映射帧缓冲区到进程空间 |
| `munmap()` | 解除映射 |
| `close()` | 关闭设备 |

## 4. 交叉编译完整操作步骤

```bash
# 进入示例目录
cd 19_lcd

# 编译所有示例
arm-linux-gnueabihf-gcc -o lcd_info lcd_info.c
arm-linux-gnueabihf-gcc -o lcd_test lcd_test.c
arm-linux-gnueabihf-gcc -o bmp_show bmp_show.c
```

## 5. 开发板部署流程

```bash
# SCP 传输
scp lcd_info lcd_test bmp_show image.bmp root@192.168.1.50:/home/root/

# NFS 挂载运行
mount -t nfs -o nolock,tcp 192.168.1.100:/home/alientek/nfsroot /mnt
cd /mnt/19_lcd
```

## 6. 完整运行验证操作

### 6.1 查看 LCD 信息

```bash
./lcd_info

# 预期输出
分辨率: 800*480
像素深度 bpp: 32
一行字节数: 3200
像素格式: ARGB8888
```

### 6.2 基本绘图测试

```bash
./lcd_test

# 预期效果
# 屏幕上显示红、绿、蓝、黄等颜色的方块和线条
```

### 6.3 BMP 图像显示

```bash
./bmp_show image.bmp

# 预期效果
# LCD 屏幕上显示 BMP 图像
```

## 7. 常见报错 FAQ

### 7.1 无法打开 /dev/fb0

```bash
# 检查设备节点是否存在
ls -l /dev/fb0

# 检查内核是否启用了 FrameBuffer 支持
# CONFIG_FB 和 CONFIG_FB_IMX6UL 必须启用
```

### 7.2 屏幕显示异常

**原因**：
1. 分辨率或像素格式不匹配
2. 内存映射地址错误
3. 像素数据格式错误

**排查步骤**：
```bash
# 确认屏幕信息
./lcd_info

# 检查内核日志
dmesg | grep -i fb
```

### 7.3 BMP 显示不正确

**原因**：BMP 格式不支持或像素格式不匹配。

**解决方案**：
确保 BMP 文件为 24 位或 32 位真彩色格式，且分辨率不超过屏幕分辨率。
