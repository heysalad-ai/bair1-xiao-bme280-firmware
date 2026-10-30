# Bair1 厂家说明（中国）

**生产后端已部署并验证。设备烧录仍暂停。**

现在可以审核代码、确认电路、准备私有配置并编译。**只有收到 Peter 对具体电路板的明确烧录许可后，才可以烧录并进行单板测试。** 本代码包不包含 Wi-Fi 密码或 API 密钥；不要把填写后的配置文件上传到 GitHub、聊天群或工单。

## 文件与硬件

将以下两个文件放在同一个 `Bair1_Xiao` 文件夹中：

- `Bair1_Xiao.ino`：采样及 HTTPS 上传程序。
- `bair1_config.h`：配置及公开的 TLS 根证书。

这是**一个程序加一个配置头文件**，不是两种不同电路板的程序。目标板是 **Seeed Studio XIAO ESP32-S3**。每块板根据芯片标识自动生成不同设备 ID。

BME280 使用 D4（SDA）和 D5（SCL），地址 0x76 或 0x77。D3 和 D2 为模拟输入。原始电路中 D6 与 D2 相连，因此 D6 必须保持高阻 INPUT，不能输出或启用上下拉。两个分压比目前均按 2:1 设置；请先确认实际电路、共地和安全输入电压。不要使用旧版 SPS30 把 D3 作为 SCL 的接线。

## 编译与私有配置

Arduino 中选择 `XIAO_ESP32S3`。已验证版本：ESP32 3.3.11、Adafruit BME280 2.3.0、Unified Sensor 1.1.15、BusIO 1.17.4、ArduinoJson 7.4.3。也可以使用：

```sh
arduino-cli compile --fqbn esp32:esp32:XIAO_ESP32S3 Bair1_Xiao
```

生产接口：`https://app.bair1.live/api/readings`

请让 Peter 私下提供具有 `write:readings` 权限的专用 Bair1 开发者 API 密钥。在私有的 `bair1_config.h` 中填写 Wi-Fi 名称、密码、API 密钥及上述 URL。不要使用 AWS、Auth0、Notehub 或管理员凭据。公开文件中这些值为空；缺少配置时只输出串口数据，不启动上传。串口波特率为 115200。**配置或编译完成不代表可以烧录。**

## 网络与服务器返回值

工厂网络需要允许 DNS、访问 `app.bair1.live` 的 HTTPS/TCP 443，以及访问 `pool.ntp.org` 或 `time.cloudflare.com` 的 NTP/UDP 123。尚未在中国工厂网络进行连通性测试。可在电脑上执行以下只读检查，正常应返回 HTTP 401（因为没有提供密钥）：

```sh
curl -i https://app.bair1.live/api/readings
```

上传采用 JSON POST，认证头为 `Authorization: Bearer <开发者密钥>`。新记录返回 `ok:true, stored:true, outcome:"stored"`；重复发送同一记录返回 `ok:true, stored:false, outcome:"duplicate"`。这两种结果才算上传成功。

401/403：检查密钥及权限。404：请 Peter 检查设备归属。409：同一事件 ID 对应的数据发生变化。503：保留记录，稍后重试。不要通过更改设备 ID 绕过归属检查，也不要关闭 TLS 证书校验。

## 数据与限制

每两秒采集温度、湿度、气压、估算海拔，以及两路 ADC 原始值、引脚电压和计算输入电压。ADC 通道的实际物理用途尚待电路确认；不能直接称为气体浓度或电池电量。AQI 为 null。

内存队列可保存 120 条记录（约四分钟）。队列满时丢弃新记录；掉电或重启会丢失未上传记录；获得 NTP 时间前的数据只输出串口，不加入上传队列。持续被服务器拒绝的队首记录会阻塞后续上传。因此，该程序不是保证断网或断电期间零丢失的记录器。生产服务器按首次接收时间保留记录 14 天。

获得烧录许可后，请先测试一块指定板：检查传感器与万用表读数、服务器回读字段及时间、短暂断网后的队列恢复、重启和缓冲区满的行为。反馈编译结果、固件版本和脱敏测试摘要；不要公开密钥、设备标识或定位信息。

软件编译、生产接口和重试协议已验证，但尚未进行实际设备烧录或硬件精度、Wi-Fi、NTP、掉电行为验证。详见英文 `VERIFICATION.md`。
