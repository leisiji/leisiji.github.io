---
title: 阿里云物联网套件的体验
---

# 移植

我用的是 ESP8266，所以乐鑫官方已经有移植好的固件，如果以后有其他的平台，我再去填坑吧。
在 git 里面执行以下，就可以吧 SDK 下载到本地：
<!-- more -->
```
$ git clone https://github.com/espressif/esp8266-aliyun.git
// 注意 clone 后以下的步骤
$ cd esp8266-aliyun
$ git submodule update --init --recursive
```

# SDK

SDK 的大体结构：
```
output/release/
+-- bin
+-- include
|   +-- exports
|   |   +-- iot_export_coap.h
|   |   +-- iot_export_errno.h
|   |   +-- iot_export_http.h
|   |   +-- iot_export_mqtt.h
|   |   +-- iot_export_ota.h
|   |   +-- iot_export_shadow.h
|   +-- imports
|   |   +-- iot_import_coap.h
|   |   +-- iot_import_config.h
|   |   +-- iot_import_dtls.h
|   |   +-- iot_import_ota.h
|   +-- iot_export.h
|   +-- iot_import.h
+-- lib
|   +-- libiot_sdk.a
+-- src
    +-- coap-example.c
    +-- http-example.c
    +-- Makefile
    +-- mqtt-example.c
```
a 文件是库文件，头文件对应的 c 代码编译成了 a 文件。

> 所以比较重要的是读懂 Makefile，不然无法编译自己的驱动（也就是 a 文件），而且还会出现很多的 bug，学习的话在[这里](https://www.jianshu.com/p/bdab2af57c17)入门吧。

| 文件|	说明|
|----|
|`include/iot_import.h`	| 这个头文件中列出为SDK适配新硬件平台时, 需要实现的平台抽象层函数, 以 `HAL_*()` 的方式命名; 编写平台抽象层实现时, 包含此头文件即可 |
| `include/imports/iot_import_*.h` |	这组头文件按功能模块分列各模块依赖的平台抽象层函数, 例如 `iot_import_coap.h`中列出用CoAP协议通信需要实现的那些, 若不使用CoAP, 可直接忽略该文件|
| `include/iot_export.h`|	这个头文件中列出SDK能提供的所有用户级别API, 以 `IOT_*()` 的方式命名, 也就是供用户调用编写业务应用程序的函数|
| `include/exports/iot_export_*.h` |	这组头文件按功能模块分列各模块提供的用户级别API, 例如 `iot_export_coap.h` 中列出用CoAP协议通信时可用的那些, 若不使用CoAP, 可直接忽略该文件|
|`lib/libiot_sdk.a`|	这个二进制的压缩库文件就是编译好的所谓"物联网套件设备端SDK", 它配合上面的 `include/iot_export.h` 文件, 分别提供API的接口实现和接口声明, 供用户link到自己的业务应用程序中|
| `src/Makefile`	| 示例用Makefile, 演示得到 `lib/libiot_sdk.a` 之后, 如何在SDK之外链接它使用起来|


# 连接服务器
参考自[这里](https://help.aliyun.com/document_detail/65255.html?spm=5176.doc30530.6.691.taxvHT)

## 设备端代码修改
修改 `aliyun_config.h`。注意下面的 XX 的修改：
```
#define PRODUCT_KEY             "XX"  // type:string
#define DEVICE_NAME             "XX"  // type:string
#define DEVICE_SECRET           "XX"  // type:string

#define WIFI_SSID       "XX"       // type:string, your AP/router SSID to config your device networking
#define WIFI_PASSWORD   "XX"       // type:string, your AP/router password
```

设备上传数据在 mqtt.c 中的 `mqttclient()` 完成：
```
u8_t temperature;
u8_t humidity;
u8_t *data;

/* 将数据写入 publish message*/
/* readBoth 是一个用 DHT11 读取温度和湿度的函数 */
data = readBoth(2);
humidity = *data;
temperature = *(data+1);
cJSON *msg_pub_json = cJSON_CreateObject();
cJSON_AddNumberToObject(msg_pub_json,"temperature",temperature);
cJSON_AddNumberToObject(msg_pub_json,"humidity",humidity);
msg_pub = cJSON_Print(msg_pub_json);
cJSON_Delete(msg_pub_json);
msg_len = strlen(msg_pub);

// 上传的数据是 payload
topic_msg.payload = (void *)msg_pub;
...
rc = IOT_MQTT_Publish(pclient, TOPIC_DATA, &topic_msg);
```
上面主要用到 cJSON 的库，所以在[这里](http://blog.csdn.net/xukai871105/article/details/17094113)学习一下 cJSON 吧。
`cJSON_Print()` 是传输和保存 JSON 格式数据的一个方法，JSON 是传输的格式，而不是文件格式。



## 创建产品和设备
首先在产品管理 -> 设备管理 -> 设备属性中添加两个属性：

|属性	|属性值|	描述|
|---|
|tag|	云栖小镇 2号楼3层007S|	设备所在位置|
|deviceISN|	T20180102XnbKjmoAnUb|	设备序列号| 

接着创建并订阅Topic：这里我们选择温湿度计产品，在左侧消息通信下创建一个 Topic 为 `/productKey/${deviceName}/data`，设备操作权限设置：发布。


## 创建并启用规则引擎
一条完整的规则包括基本信息，处理数据，转发数据三部分，其中转发数据支持配置多个转发动作。

我们从设备本身信息中抽取设备名(deviceName)。
从温湿度采集设备上报数据消息的 payload 中获取温度值(temperature)和湿度值(humidity)。
添加以下的规则引擎（注意去掉 SELECT 和 FROM）：
```
SELECT
//attribute('tag') as tag,
//attribute('deviceISN') as isn,
// 上面这两项可以根据以后的需要再加上，作为不同区域的不同设备
deviceName() as deviceName,
temperature,
humidity,
timestamp('yyyy-MM-dd HH:mm:ss') as time
FROM
"/此处为产品productKey/+/data"
```

> 注意：规则引擎的字段填入 `deviceName() as deviceName,temperature,humidity`，topic 填入产品名称和 `+/data`，一定不能够忘记 `+/`。我之前就是被这个坑了，导致很久都找不出不能转发服务器的原因。



