# ATK I.MX6ULL - 21_libpng C APP 示例说明

## 1. 目录功能概述

本示例演示如何使用 libpng 库在 I.MX6ULL 开发板上进行 PNG 图像解码和显示。

程序通过 libpng 提供的 API 将 PNG 图像解码为 RGB 像素数据，然后通过 FrameBuffer 接口显示到 LCD 屏幕上。

**适用硬件外设**：LCD 显示屏

**示例文件对应功能**：
- `show_png_image.c`：PNG 图像解码与显示主程序
- `setjmp.c`：setjmp/longjmp 非局部跳转示例（辅助理解 PNG 错误处理）

## 2. 目录内源码文件说明

| 文件名 | 类型 | 作用说明 |
|:---|:---|:---|
| show_png_image.c | C 源文件 | PNG 图像解码与 FrameBuffer 显示示例 |
| setjmp.c | C 源文件 | setjmp/longjmp 非局部跳转示例 |

## 3. 核心 C 代码逻辑拆解分析

### 3.1 libpng 解码流程

```c
// 创建 PNG 读取结构
png_structp png_ptr = png_create_read_struct(PNG_LIBPNG_VER_STRING, NULL, NULL, NULL);
png_infop info_ptr = png_create_info_struct(png_ptr);

// 设置错误处理
if (setjmp(png_jmpbuf(png_ptr))) {
    png_destroy_read_struct(&png_ptr, &info_ptr, NULL);
    fclose(fp);
    return -1;
}

// 设置数据源
FILE *fp = fopen(filename, "rb");
png_init_io(png_ptr, fp);

// 读取图像信息
png_read_info(png_ptr, info_ptr);
int width = png_get_image_width(png_ptr, info_ptr);
int height = png_get_image_height(png_ptr, info_ptr);

// 读取图像数据
png_read_image(png_ptr, row_pointers);

// 清理
png_read_end(png_ptr, NULL);
png_destroy_read_struct(&png_ptr, &info_ptr, NULL);
```

### 3.2 涉及的库 API

| API | 作用 |
|:---|:---|
| `png_create_read_struct()` | 创建 PNG 读取结构 |
| `png_create_info_struct()` | 创建 PNG 信息结构 |
| `png_init_io()` | 设置输入文件 |
| `png_read_info()` | 读取图像信息 |
| `png_read_image()` | 读取图像数据 |
| `png_read_end()` | 读取结束 |
| `png_destroy_read_struct()` | 销毁读取结构 |

## 4. 交叉编译完整操作步骤

```bash
# 进入示例目录
cd 21_libpng

# 编译 PNG 显示示例
arm-linux-gnueabihf-gcc -o show_png_image show_png_image.c -lpng

# 编译 setjmp 示例
arm-linux-gnueabihf-gcc -o setjmp setjmp.c
```

## 5. 开发板部署流程

```bash
# 确保开发板已安装 libpng
# 查看动态库
ldconfig -p | grep libpng

# 运行程序
./show_png_image image.png
```

## 6. 完整运行验证操作

```bash
./show_png_image image.png

# 预期效果
# LCD 屏幕显示 PNG 图像
```

## 7. 常见报错 FAQ

### 7.1 找不到 libpng.so

```bash
# 检查动态库路径
ldconfig -p | grep libpng

# 手动指定库路径
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/libpng/lib
```

### 7.2 PNG 解码失败

**原因**：PNG 文件格式不正确或文件损坏。

**解决方案**：
使用 `file` 命令检查文件格式：
```bash
file image.png
```
