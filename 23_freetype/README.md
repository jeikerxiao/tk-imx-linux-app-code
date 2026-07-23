# ATK I.MX6ULL - 23_freetype C APP 示例说明

## 1. 目录功能概述

本示例演示如何使用 FreeType 库在 I.MX6ULL 开发板上进行字体渲染和显示。

程序通过 FreeType 提供的 API 将文字渲染为位图，然后通过 FrameBuffer 接口显示到 LCD 屏幕上。

示例涵盖单字显示和文本显示两种模式。

**适用硬件外设**：LCD 显示屏

**示例文件对应功能**：
- `freetype_test.c`：FreeType 字体渲染与显示主程序
- `show_char.c`：字符点阵显示示例

## 2. 目录内源码文件说明

| 文件名 | 类型 | 作用说明 |
|:---|:---|:---|
| freetype_test.c | C 源文件 | FreeType 字体渲染与 FrameBuffer 显示示例 |
| show_char.c | C 源文件 | 字符点阵显示示例 |

## 3. 核心 C 代码逻辑拆解分析

### 3.1 FreeType 初始化

```c
#include <ft2build.h>
#include FT_FREETYPE_H

FT_Library library;
FT_Face face;

// 初始化 FreeType 库
FT_Init_FreeType(&library);

// 加载字体文件
FT_New_Face(library, "/usr/share/fonts/truetype/arial.ttf", 0, &face);

// 设置字体大小
FT_Set_Pixel_Sizes(face, 24, 0);
```

### 3.2 字符渲染

```c
// 加载字符
FT_Load_Char(face, 'A', FT_LOAD_RENDER);

// 获取位图数据
FT_Bitmap bitmap = face->glyph->bitmap;

// 将位图数据写入 FrameBuffer
for (int y = 0; y < bitmap.rows; y++) {
    for (int x = 0; x < bitmap.width; x++) {
        if (bitmap.buffer[y * bitmap.pitch + x]) {
            // 在 (x, y) 位置绘制像素
            draw_pixel(x + offset_x, y + offset_y, color);
        }
    }
}
```

### 3.3 涉及的库 API

| API | 作用 |
|:---|:---|
| `FT_Init_FreeType()` | 初始化 FreeType 库 |
| `FT_New_Face()` | 加载字体文件 |
| `FT_Set_Pixel_Sizes()` | 设置字体大小 |
| `FT_Load_Char()` | 加载字符 |
| `FT_Load_Glyph()` | 加载字形 |

## 4. 交叉编译完整操作步骤

```bash
# 进入示例目录
cd 23_freetype

# 编译 FreeType 示例
arm-linux-gnueabihf-gcc -o freetype_test freetype_test.c -lfreetype

# 编译字符点阵示例
arm-linux-gnueabihf-gcc -o show_char show_char.c
```

## 5. 开发板部署流程

```bash
# 确保开发板已安装 FreeType
# 查看动态库
ldconfig -p | grep freetype

# 运行程序
./freetype_test
./show_char
```

## 6. 完整运行验证操作

```bash
./freetype_test

# 预期效果
# LCD 屏幕显示渲染后的文字
```

## 7. 常见报错 FAQ

### 7.1 找不到 libfreetype.so

```bash
# 检查动态库路径
ldconfig -p | grep freetype

# 手动指定库路径
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/freetype/lib
```

### 7.2 字体渲染失败

**原因**：字体文件路径错误或字体格式不支持。

**解决方案**：
确保字体文件存在且路径正确。
