# ATK I.MX6ULL - 20_libjpeg C APP 示例说明

## 1. 目录功能概述

本示例演示如何使用 libjpeg 库在 I.MX6ULL 开发板上进行 JPEG 图像解码和显示。

程序通过 libjpeg 提供的 API 将 JPEG 图像解码为 RGB 像素数据，然后通过 FrameBuffer 接口显示到 LCD 屏幕上。

**适用硬件外设**：LCD 显示屏

**示例文件对应功能**：
- `show_jpeg_image.c`：JPEG 图像解码与显示主程序

## 2. 目录内源码文件说明

| 文件名 | 类型 | 作用说明 |
|:---|:---|:---|
| show_jpeg_image.c | C 源文件 | JPEG 图像解码与 FrameBuffer 显示示例 |

## 3. 核心 C 代码逻辑拆解分析

### 3.1 libjpeg 解码流程

```c
// 创建 JPEG 解压对象
struct jpeg_decompress_struct cinfo;
struct jpeg_error_mgr jerr;
cinfo.err = jpeg_std_error(&jerr);
jpeg_create_decompress(&cinfo);

// 指定数据源
FILE *infile = fopen(filename, "rb");
jpeg_stdio_src(&cinfo, infile);

// 读取头部信息
jpeg_read_header(&cinfo, TRUE);

// 开始解压
jpeg_start_decompress(&cinfo);

// 逐行读取解压后的数据
while (cinfo.output_scanline < cinfo.output_height) {
    jpeg_read_scanlines(&cinfo, buffer, 1);
    // 将 RGB 数据写入 FrameBuffer
}

// 完成解压
jpeg_finish_decompress(&cinfo);
jpeg_destroy_decompress(&cinfo);
```

### 3.2 涉及的库 API

| API | 作用 |
|:---|:---|
| `jpeg_create_decompress()` | 创建解压对象 |
| `jpeg_stdio_src()` | 设置输入文件 |
| `jpeg_read_header()` | 读取图像头部 |
| `jpeg_start_decompress()` | 开始解压 |
| `jpeg_read_scanlines()` | 读取解压后的扫描线 |
| `jpeg_finish_decompress()` | 完成解压 |
| `jpeg_destroy_decompress()` | 销毁解压对象 |

## 4. 交叉编译完整操作步骤

```bash
# 进入示例目录
cd 20_libjpeg

# 编译（需要链接 libjpeg 库）
arm-linux-gnueabihf-gcc -o show_jpeg_image show_jpeg_image.c -ljpeg

# 如 libjpeg 安装在自定义路径
arm-linux-gnueabihf-gcc -o show_jpeg_image show_jpeg_image.c \
    -I/path/to/jpeg/include -L/path/to/jpeg/lib -ljpeg
```

## 5. 开发板部署流程

```bash
# 确保开发板已安装 libjpeg
# 查看动态库
ldconfig -p | grep libjpeg

# 运行程序
./show_jpeg_image image.jpg
```

## 6. 完整运行验证操作

```bash
./show_jpeg_image image.jpg

# 预期效果
# LCD 屏幕显示 JPEG 图像
```

## 7. 常见报错 FAQ

### 7.1 找不到 libjpeg.so

```bash
# 检查动态库路径
ldconfig -p | grep libjpeg

# 手动指定库路径
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/jpeg/lib
```

### 7.2 JPEG 解码失败

**原因**：JPEG 文件格式不正确或文件损坏。

**解决方案**：
使用 `file` 命令检查文件格式：
```bash
file image.jpg
```
