# ATK I.MX6ULL - 22_lcd_vertical_display C APP 示例说明

## 1. 目录功能概述

本示例演示如何在 I.MX6ULL 开发板上实现 LCD 竖屏显示。

通过调整 FrameBuffer 的坐标映射关系，将原本横屏的显示内容旋转为竖屏显示。

**适用硬件外设**：LCD 显示屏（支持横竖屏切换）

**示例文件对应功能**：
- `lcd_vertical_display.c`：竖屏显示主程序

## 2. 目录内源码文件说明

| 文件名 | 类型 | 作用说明 |
|:---|:---|:---|
| lcd_vertical_display.c | C 源文件 | LCD 竖屏显示示例 |

## 3. 核心 C 代码逻辑拆解分析

### 3.1 竖屏坐标转换

```c
// 原始横屏坐标 (x, y)
// 竖屏坐标 (y, width - x)
// 通过调整内存地址计算实现旋转

void draw_pixel_vertical(int x, int y, unsigned int color) {
    int new_x = y;
    int new_y = width - x - 1;
    screen_base[new_y * height + new_x] = color;
}
```

### 3.2 FrameBuffer 操作

```c
// 打开 FrameBuffer 设备
fd = open("/dev/fb0", O_RDWR);

// 获取屏幕信息
ioctl(fd, FBIOGET_VSCREENINFO, &fb_var);

// 映射帧缓冲区
screen_base = mmap(NULL, screen_size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);

// 竖屏显示内容
for (int y = 0; y < height; y++) {
    for (int x = 0; x < width; x++) {
        // 计算旋转后的坐标
        draw_pixel_vertical(x, y, color);
    }
}
```

## 4. 交叉编译完整操作步骤

```bash
# 进入示例目录
cd 22_lcd_vertical_display

# 编译
arm-linux-gnueabihf-gcc -o lcd_vertical_display lcd_vertical_display.c
```

## 5. 开发板部署流程

```bash
# 运行程序
./lcd_vertical_display

# 预期效果
# LCD 屏幕显示竖屏内容
```

## 6. 完整运行验证操作

```bash
./lcd_vertical_display

# 预期效果
# 屏幕内容从横屏旋转为竖屏显示
```

## 7. 常见报错 FAQ

### 7.1 显示内容错位

**原因**：坐标计算错误。

**解决方案**：
检查坐标转换公式是否正确。

### 7.2 屏幕显示异常

**原因**：FrameBuffer 参数设置错误。

**解决方案**：
```bash
# 检查 FrameBuffer 信息
cat /sys/class/graphics/fb0/virtual_size
cat /sys/class/graphics/fb0/bits_per_pixel
```
