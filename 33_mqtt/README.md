# ATK I.MX6ULL - 33_mqtt C APP 示例说明

## 1. 目录功能概述

本示例演示如何在 I.MX6ULL 开发板上使用 MQTT 协议进行物联网通信。

程序通过 MQTT 客户端库连接到 MQTT 服务器，实现消息的发布和订阅。

**适用硬件外设**：以太网接口

**示例文件对应功能**：
- `mqtt_prj/mqttClient.c`：MQTT 客户端主程序
- `mqtt_prj/CMakeLists.txt`：CMake 构建脚本
- `mqtt_prj/cmake/arm-linux-setup.cmake`：交叉编译器配置

## 2. 目录内源码文件说明

| 文件名 | 类型 | 作用说明 |
|:---|:---|:---|
| mqtt_prj/mqttClient.c | C 源文件 | MQTT 客户端示例 |
| mqtt_prj/CMakeLists.txt | CMake 脚本 | 项目构建配置 |
| mqtt_prj/cmake/arm-linux-setup.cmake | CMake 脚本 | 交叉编译器配置 |

## 3. 核心 C 代码逻辑拆解分析

### 3.1 MQTT 客户端初始化

```c
#include <MQTTClient.h>

MQTTClient client;
MQTTClient_create(&client, "tcp://broker.hivemq.com:1883", "client_id",
                  MQTTCLIENT_PERSISTENCE_NONE, NULL);

MQTTClient_connectOptions conn_opts = MQTTClient_connectOptions_initializer;
conn_opts.keepAliveInterval = 20;
conn_opts.cleansession = 1;
MQTTClient_connect(client, &conn_opts);
```

### 3.2 消息订阅

```c
// 设置消息回调
MQTTClient_setCallbacks(client, NULL, connlost, msgarrvd, delivered);

// 订阅主题
MQTTClient_subscribe(client, "test/topic", 0);

// 消息接收回调
int msgarrvd(void *context, char *topicName, int topicLen, MQTTClient_message *message) {
    printf("Message arrived\n");
    printf("Topic: %s\n", topicName);
    printf("Message: %.*s\n", message->payloadlen, (char*)message->payload);
    MQTTClient_freeMessage(&message);
    MQTTClient_free(topicName);
    return 1;
}
```

### 3.3 消息发布

```c
MQTTClient_message pubmsg = MQTTClient_message_initializer;
pubmsg.payload = "Hello, MQTT!";
pubmsg.payloadlen = strlen(pubmsg.payload);
pubmsg.qos = 0;
pubmsg.retained = 0;

MQTTClient_publishMessage(client, "test/topic", &pubmsg, &token);
MQTTClient_waitForCompletion(client, token, 10000L);
```

### 3.4 涉及的库 API

| API | 作用 |
|:---|:---|
| `MQTTClient_create()` | 创建 MQTT 客户端 |
| `MQTTClient_connect()` | 连接到服务器 |
| `MQTTClient_setCallbacks()` | 设置回调函数 |
| `MQTTClient_subscribe()` | 订阅主题 |
| `MQTTClient_publishMessage()` | 发布消息 |
| `MQTTClient_waitForCompletion()` | 等待发送完成 |
| `MQTTClient_disconnect()` | 断开连接 |
| `MQTTClient_destroy()` | 销毁客户端 |

## 4. 交叉编译完整操作步骤

```bash
# 进入示例目录
cd 33_mqtt/mqtt_prj

# 创建构建目录
mkdir build && cd build

# 配置 CMake
cmake .. -DCMAKE_TOOLCHAIN_FILE=../cmake/arm-linux-setup.cmake

# 编译
make
```

## 5. 开发板部署流程

```bash
# 确保 MQTT 库已安装
# 查看动态库
ldconfig -p | grep mqtt

# 运行程序
./mqttClient
```

## 6. 完整运行验证操作

```bash
# 运行 MQTT 客户端
./mqttClient

# 预期效果
# 客户端连接到 MQTT 服务器
# 订阅并接收消息
# 发布消息到指定主题
```

## 7. 常见报错 FAQ

### 7.1 连接失败

```bash
# 检查网络连接
ping broker.hivemq.com

# 检查 MQTT 服务器地址和端口
```

### 7.2 订阅失败

**原因**：主题格式错误或权限不足。

**解决方案**：
确保主题格式正确，并且有足够的权限。

### 7.3 消息发送失败

**原因**：网络不稳定或服务器未响应。

**解决方案**：
确保网络连接稳定，检查服务器状态。
