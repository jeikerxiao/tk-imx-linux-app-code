# ATK I.MX6ULL - 18_tslib C APP 示例说明

## 1. 目录功能概述

本示例演示如何使用 tslib 库进行触摸屏应用编程。

tslib 是一个开源的触摸屏校准和过滤库，可以将原始触摸数据转换为标准坐标。

适用于电容触摸屏的坐标获取、手势识别等场景。示例涵盖单点触摸和多点触摸两种模式。

**适用硬件外设**：电容触摸屏

**示例文件对应功能**：
- `ts_read.c`：使用 tslib 读取单点触摸坐标
- `ts_read_mt.c`：使用 tslib 读取多点触摸坐标

## 2. 目录内源码文件说明

| 文件名 | 类型 | 作用说明 |
|:---|:---|:---|
| ts_read.c | C 源文件 | 单点触摸示例，使用 ts_read() 读取触摸坐标 |
| ts_read_mt.c | C 源文件 | 多点触摸示例，使用 ts_read_mt() 读取多点坐标 |

## 3. 核心 C 代码逻辑拆解分析

### 3.1 tslib 初始化

```c
struct tsdev *ts = ts_setup(NULL, 0);
if (NULL == ts) {
    fprintf(stderr, "ts_setup error");
    exit(EXIT_FAILURE);
}
```

`ts_setup()` 函数自动完成触摸屏设备的打开和配置。`NULL` 参数表示使用默认设备（通常是 `/dev/input/event0`），`0` 表示使用默认配置。

### 3.2 单点触摸读取

```c
struct ts_sample samp;
if (0 > ts_read(ts, &samp, 1)) {
    fprintf(stderr, "ts_read error");
    ts_close(ts);
    exit(EXIT_FAILURE);
}
if (samp.pressure) {  // 按压力 > 0
    printf("按下(%d, %d)\n", samp.x, samp.y);
}
```

### 3.3 多点触摸读取

```c
struct ts_sample_mt *mt_ptr = NULL;
ts_read_mt(ts, &mt_ptr, max_slots, 1);
for (i = 0; i < max_slots; i++) {
    if (mt_ptr[i].valid) {
        printf("slot<%d>, 按下(%d, %d)\n", mt_ptr[i].slot, mt_ptr[i].x, mt_ptr[i].y);
    }
}
```

### 3.4 涉及的 API

| API | 作用 |
|:---|:---|
| `ts_setup()` | 初始化触摸屏设备 |
| `ts_read()` | 读取单点触摸数据 |
| `ts_read_mt()` | 读取多点触摸数据 |
| `ts_close()` | 关闭触摸屏设备 |

## 4. 交叉编译完整操作步骤

```bash
# 进入示例目录
cd 18_tslib

# 编译（需要链接 tslib 库）
arm-linux-gnueabihf-gcc -o ts_read ts_read.c -lts
arm-linux-gnueabihf-gcc -o ts_read_mt ts_read_mt.c -lts

# 如 tslib 安装在自定义路径，需指定路径
arm-linux-gnueabihf-gcc -o ts_read ts_read.c -I/path/to/tslib/include -L/path/to/tslib/lib -lts
```

## 5. 开发板部署流程

```bash
# 确保开发板已安装 tslib
# 设置环境变量（添加到 /etc/profile 或 ~/.bashrc）
export TSLIB_FBDEVICE=/dev/fb0
export TSLIB_TSDEVICE=/dev/input/event0
export TSLIB_CONFFILE=/etc/ts.conf
export TSLIB_CALIBFILE=/etc/pointercal
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib

# 运行程序
./ts_read
./ts_read_mt
```

## 6. 完整运行验证操作

### 6.1 单点触摸测试

```bash
./ts_read

# 触摸屏幕时输出
按下(300, 200)
移动(320, 210)
松开
```

### 6.2 多点触摸测试

```bash
./ts_read_mt

# 两点同时触摸时输出
max_slots: 5
slot<0>, 按下(300, 200)
slot<1>, 按下(500, 300)
```

## 7. 常见报错 FAQ

### 7.1 ts_setup 失败

```bash
# 检查环境变量是否正确
echo $TSLIB_TSDEVICE

# 检查设备节点是否存在
ls /dev/input/event*

# 检查 tslib 配置文件
ls /etc/ts.conf
```

### 7.2 坐标不准确

**原因**：触摸屏未校准。

**解决方案**：
```bash
# 运行校准程序
ts_calibrate

# 确保校准文件存在
cat /etc/pointercal
```

### 7.3 找不到 libts.so

```bash
# 检查动态库路径
ldconfig -p | grep libts

# 手动指定库路径
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/tslib/lib
```
