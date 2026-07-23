# ATK I.MX6ULL - 25_v4l2_camera C APP 示例说明

## 1. 目录功能概述

本示例演示如何使用 V4L2（Video for Linux 2）接口在 I.MX6ULL 开发板上进行摄像头视频采集。

程序通过 V4L2 API 配置摄像头参数、采集图像数据，并通过 FrameBuffer 接口显示到 LCD 屏幕上。

**适用硬件外设**：USB 摄像头（如 UVC 摄像头）

**示例文件对应功能**：
- `v4l2_camera.c`：V4L2 摄像头采集与显示主程序

## 2. 目录内源码文件说明

| 文件名 | 类型 | 作用说明 |
|:---|:---|:---|
| v4l2_camera.c | C 源文件 | V4L2 摄像头采集与 FrameBuffer 显示示例 |

## 3. 核心 C 代码逻辑拆解分析

### 3.1 V4L2 设备初始化

```c
#include <linux/videodev2.h>

// 打开 V4L2 设备
int fd = open("/dev/video0", O_RDWR);

// 查询设备能力
struct v4l2_capability cap;
ioctl(fd, VIDIOC_QUERYCAP, &cap);
printf("Driver: %s\n", cap.driver);

// 设置视频格式
struct v4l2_format fmt;
memset(&fmt, 0, sizeof(fmt));
fmt.type = V4L2_BUF_TYPE_VIDEO_CAPTURE;
fmt.fmt.pix.width = 640;
fmt.fmt.pix.height = 480;
fmt.fmt.pix.pixelformat = V4L2_PIX_FMT_MJPEG;
fmt.fmt.pix.field = V4L2_FIELD_INTERLACED;
ioctl(fd, VIDIOC_S_FMT, &fmt);
```

### 3.2 缓冲队列管理

```c
// 请求缓冲区
struct v4l2_requestbuffers req;
memset(&req, 0, sizeof(req));
req.count = 4;
req.type = V4L2_BUF_TYPE_VIDEO_CAPTURE;
req.memory = V4L2_MEMORY_MMAP;
ioctl(fd, VIDIOC_REQBUFS, &req);

// 映射缓冲区
void *buffers[4];
for (int i = 0; i < 4; i++) {
    struct v4l2_buffer buf;
    memset(&buf, 0, sizeof(buf));
    buf.type = V4L2_BUF_TYPE_VIDEO_CAPTURE;
    buf.memory = V4L2_MEMORY_MMAP;
    buf.index = i;
    ioctl(fd, VIDIOC_QUERYBUF, &buf);
    buffers[i] = mmap(NULL, buf.length, PROT_READ | PROT_WRITE,
                      MAP_SHARED, fd, buf.m.offset);
}

// 入队
for (int i = 0; i < 4; i++) {
    struct v4l2_buffer buf;
    memset(&buf, 0, sizeof(buf));
    buf.type = V4L2_BUF_TYPE_VIDEO_CAPTURE;
    buf.memory = V4L2_MEMORY_MMAP;
    buf.index = i;
    ioctl(fd, VIDIOC_QBUF, &buf);
}

// 启动采集
enum v4l2_buf_type type = V4L2_BUF_TYPE_VIDEO_CAPTURE;
ioctl(fd, VIDIOC_STREAMON, &type);
```

### 3.3 图像采集与显示

```c
// 出队
struct v4l2_buffer buf;
memset(&buf, 0, sizeof(buf));
buf.type = V4L2_BUF_TYPE_VIDEO_CAPTURE;
buf.memory = V4L2_MEMORY_MMAP;
ioctl(fd, VIDIOC_DQBUF, &buf);

// 处理图像数据
// 将 MJPEG 解码为 RGB
// 显示到 LCD

// 入队
ioctl(fd, VIDIOC_QBUF, &buf);
```

### 3.4 涉及的 Linux 系统调用与 API

| 系统调用/API | 作用 |
|:---|:---|
| `open()` | 打开 V4L2 设备 |
| `ioctl()` | V4L2 控制命令 |
| `mmap()` | 映射视频缓冲区 |
| `munmap()` | 解除映射 |
| `close()` | 关闭设备 |

## 4. 交叉编译完整操作步骤

```bash
# 进入示例目录
cd 25_v4l2_camera

# 编译
arm-linux-gnueabihf-gcc -o v4l2_camera v4l2_camera.c
```

## 5. 开发板部署流程

```bash
# 确保摄像头已连接
ls /dev/video*

# 运行程序
./v4l2_camera /dev/video0
```

## 6. 完整运行验证操作

```bash
# 查看摄像头设备
ls /dev/video*

# 运行程序
./v4l2_camera /dev/video0

# 预期效果
# LCD 屏幕显示摄像头实时画面
```

## 7. 常见报错 FAQ

### 7.1 找不到摄像头设备

```bash
# 检查 USB 设备
lsusb

# 检查内核日志
dmesg | grep -i video

# 加载 UVC 驱动
modprobe uvcvideo
```

### 7.2 视频格式不支持

**原因**：摄像头不支持请求的像素格式。

**解决方案**：
```bash
# 查看摄像头支持的格式
v4l2-ctl --list-formats

# 使用支持的格式
```

### 7.3 内存映射失败

**原因**：缓冲区大小不足或内存不足。

**解决方案**：
减少缓冲区数量或大小。
