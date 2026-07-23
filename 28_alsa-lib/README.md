# ATK I.MX6ULL - 28_alsa-lib C APP 示例说明

## 1. 目录功能概述

本示例演示如何在 I.MX6ULL 开发板上使用 ALSA（Advanced Linux Sound Architecture）库进行音频采集和播放。

程序通过 ALSA API 实现音频的录制、播放、混音等功能。

**适用硬件外设**：音频输出设备（如扬声器、耳机）、音频输入设备（如麦克风）

**示例文件对应功能**：
- `pcm_playback.c`：音频播放示例
- `pcm_playback_async.c`：异步音频播放示例
- `pcm_playback_poll.c`：使用 poll 的音频播放示例
- `pcm_playback_ctl.c`：音频播放控制示例
- `pcm_playback_mixer.c`：音频混音器示例
- `pcm_capture.c`：音频采集示例
- `pcm_capture_async.c`：异步音频采集示例
- `pcm_capture_poll.c`：使用 poll 的音频采集示例
- `mic_in_config.sh`：麦克风输入配置脚本

## 2. 目录内源码文件说明

| 文件名 | 类型 | 作用说明 |
|:---|:---|:---|
| pcm_playback.c | C 源文件 | 基础音频播放示例 |
| pcm_playback_async.c | C 源文件 | 异步音频播放示例 |
| pcm_playback_poll.c | C 源文件 | 使用 poll 的音频播放示例 |
| pcm_playback_ctl.c | C 源文件 | 音频播放控制示例 |
| pcm_playback_mixer.c | C 源文件 | 音频混音器示例 |
| pcm_capture.c | C 源文件 | 音频采集示例 |
| pcm_capture_async.c | C 源文件 | 异步音频采集示例 |
| pcm_capture_poll.c | C 源文件 | 使用 poll 的音频采集示例 |
| mic_in_config.sh | Shell 脚本 | 麦克风输入配置脚本 |

## 3. 核心 C 代码逻辑拆解分析

### 3.1 ALSA 播放初始化

```c
#include <alsa/asoundlib.h>

snd_pcm_t *handle;
snd_pcm_open(&handle, "default", SND_PCM_STREAM_PLAYBACK, 0);

snd_pcm_hw_params_t *params;
snd_pcm_hw_params_malloc(&params);
snd_pcm_hw_params_any(handle, params);
snd_pcm_hw_params_set_access(handle, params, SND_PCM_ACCESS_RW_INTERLEAVED);
snd_pcm_hw_params_set_format(handle, params, SND_PCM_FORMAT_S16_LE);
snd_pcm_hw_params_set_channels(handle, params, 2);
snd_pcm_hw_params_set_rate_near(handle, params, &rate, &dir);
snd_pcm_hw_params(handle, params);
snd_pcm_hw_params_free(params);
```

### 3.2 音频播放

```c
// 播放音频数据
snd_pcm_writei(handle, buffer, frames);

// 等待播放完成
snd_pcm_drain(handle);

// 关闭设备
snd_pcm_close(handle);
```

### 3.3 音频采集

```c
// 打开采集设备
snd_pcm_open(&handle, "default", SND_PCM_STREAM_CAPTURE, 0);

// 读取音频数据
snd_pcm_readi(handle, buffer, frames);
```

### 3.4 涉及的库 API

| API | 作用 |
|:---|:---|
| `snd_pcm_open()` | 打开 PCM 设备 |
| `snd_pcm_hw_params_malloc()` | 分配硬件参数 |
| `snd_pcm_hw_params_any()` | 初始化硬件参数 |
| `snd_pcm_hw_params_set_access()` | 设置访问模式 |
| `snd_pcm_hw_params_set_format()` | 设置数据格式 |
| `snd_pcm_hw_params_set_channels()` | 设置通道数 |
| `snd_pcm_hw_params_set_rate_near()` | 设置采样率 |
| `snd_pcm_hw_params()` | 应用硬件参数 |
| `snd_pcm_writei()` | 写入播放数据 |
| `snd_pcm_readi()` | 读取采集数据 |
| `snd_pcm_drain()` | 等待播放完成 |
| `snd_pcm_close()` | 关闭 PCM 设备 |

## 4. 交叉编译完整操作步骤

```bash
# 进入示例目录
cd 28_alsa-lib

# 编译播放示例
arm-linux-gnueabihf-gcc -o pcm_playback pcm_playback.c -lasound

# 编译采集示例
arm-linux-gnueabihf-gcc -o pcm_capture pcm_capture.c -lasound

# 编译其他示例
arm-linux-gnueabihf-gcc -o pcm_playback_async pcm_playback_async.c -lasound
arm-linux-gnueabihf-gcc -o pcm_playback_poll pcm_playback_poll.c -lasound
arm-linux-gnueabihf-gcc -o pcm_playback_ctl pcm_playback_ctl.c -lasound
arm-linux-gnueabihf-gcc -o pcm_playback_mixer pcm_playback_mixer.c -lasound
arm-linux-gnueabihf-gcc -o pcm_capture_async pcm_capture_async.c -lasound
arm-linux-gnueabihf-gcc -o pcm_capture_poll pcm_capture_poll.c -lasound
```

## 5. 开发板部署流程

```bash
# 确保 ALSA 库已安装
# 查看动态库
ldconfig -p | grep asound

# 运行播放示例
./pcm_playback audio.wav

# 运行采集示例
./pcm_capture output.raw
```

## 6. 完整运行验证操作

```bash
# 播放音频
./pcm_playback test.wav

# 采集音频
./pcm_capture recorded.raw

# 配置麦克风输入
chmod +x mic_in_config.sh
./mic_in_config.sh
```

## 7. 常见报错 FAQ

### 7.1 找不到 libasound.so

```bash
# 检查动态库路径
ldconfig -p | grep asound

# 手动指定库路径
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/alsa/lib
```

### 7.2 音频播放失败

**原因**：音频设备被占用或 ALSA 配置错误。

**解决方案**：
```bash
# 检查音频设备
aplay -l

# 检查 ALSA 配置
cat /proc/asound/cards
```

### 7.3 采集音频无声

**原因**：麦克风未正确配置。

**解决方案**：
```bash
# 运行麦克风配置脚本
./mic_in_config.sh

# 检查录音设备
arecord -l
```
