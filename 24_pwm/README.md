# ATK I.MX6ULL - 24_pwm C APP 示例说明

## 1. 目录功能概述

本示例演示如何通过 Linux sysfs 文件系统控制 I.MX6ULL 开发板上的 PWM（脉冲宽度调制）。

程序通过读写 `/sys/class/pwm/` 下的属性文件，实现 PWM 的周期、占空比控制。

**适用硬件外设**：PWM 输出引脚（如蜂鸣器、LED 亮度调节等）

**示例文件对应功能**：
- `pwm.c`：PWM 控制主程序

## 2. 目录内源码文件说明

| 文件名 | 类型 | 作用说明 |
|:---|:---|:---|
| pwm.c | C 源文件 | PWM 控制示例 |

## 3. 核心 C 代码逻辑拆解分析

### 3.1 PWM 设备操作

```c
// 导出 PWM
fd = open("/sys/class/pwm/pwmchip0/export", O_WRONLY);
write(fd, "0", 1);
close(fd);

// 设置周期
duty_fd = open("/sys/class/pwm/pwmchip0/pwm0/period", O_WRONLY);
write(duty_fd, "20000000", 8);  // 20ms = 50Hz
close(duty_fd);

// 设置占空比
duty_fd = open("/sys/class/pwm/pwmchip0/pwm0/duty_cycle", O_WRONLY);
write(duty_fd, "1000000", 7);  // 1ms
close(duty_fd);

// 使能 PWM
enable_fd = open("/sys/class/pwm/pwmchip0/pwm0/enable", O_WRONLY);
write(enable_fd, "1", 1);
close(enable_fd);
```

### 3.2 涉及的 Linux 系统调用

| 系统调用 | 作用 |
|:---|:---|
| `open()` | 打开 PWM 属性文件 |
| `write()` | 写入 PWM 参数 |
| `close()` | 关闭文件描述符 |

## 4. 交叉编译完整操作步骤

```bash
# 进入示例目录
cd 24_pwm

# 编译
arm-linux-gnueabihf-gcc -o pwm pwm.c
```

## 5. 开发板部署流程

```bash
# 运行程序
./pwm 0 20000000 1000000

# 参数说明
# 第一个参数：PWM chip 编号
# 第二个参数：周期（纳秒）
# 第三个参数：占空比（纳秒）
```

## 6. 完整运行验证操作

```bash
# 设置 PWM 频率 50Hz，占空比 5%
./pwm 0 20000000 1000000

# 预期效果
# 蜂鸣器发出声音或 LED 亮度变化
```

## 7. 常见报错 FAQ

### 7.1 设备节点不存在

```bash
# 检查 PWM 设备
ls /sys/class/pwm/

# 手动导出 PWM
echo 0 > /sys/class/pwm/pwmchip0/export
```

### 7.2 权限不足

```bash
# 修改权限
chmod -R 666 /sys/class/pwm/pwmchip0/pwm0/
```
