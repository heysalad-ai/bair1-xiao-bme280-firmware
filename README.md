# Bair1 XIAO ESP32-S3 — BME280 and two analogue inputs

**For manufacturer review. Device flashing and production uploads are ON HOLD until Peter approves the server and board test.** This public package contains no Wi-Fi password or API key. Sharing these files is not approval to flash a device or send data to production.

**供厂家审核：在 Peter 确认服务器和电路板测试之前，请勿烧录设备，也不要向生产服务器上传数据。此公开代码不包含 Wi-Fi 密码或 API 密钥。**

## The two Arduino files

Keep these files in the same folder named `Bair1_Xiao`:

- `Bair1_Xiao/Bair1_Xiao.ino` — sensor sampling and HTTPS sender.
- `Bair1_Xiao/bair1_config.h` — configuration and public TLS root certificate.

This is **one sketch plus one configuration header**, not separate firmware for two different board types. Multiple identical boards generate distinct device IDs automatically from their chip identity.

## Exact hardware

- Board: Seeed Studio XIAO ESP32-S3 (`esp32:esp32:XIAO_ESP32S3`).
- BME280: SDA D4, SCL D5; I²C address 0x76 or 0x77.
- Analogue inputs: D3 and D2. D6 is tied to D2 in the supplied sketch and stays INPUT, with no pull-up or output drive.
- Each channel uses an assumed 2:1 divider. **Confirm the actual circuit, ratio, ground reference and safe pin voltage before powering/testing.** These are neutral analogue measurements; neither channel is labelled gas concentration or battery level.
- Do not use older SPS30 wiring that places SCL on D3. This sketch uses D3 as an ADC input.

## Build before any approved upload

Arduino IDE or Arduino CLI; tested library versions:

| Component | Version |
|---|---|
| Espressif ESP32 board package | 3.3.11 |
| Adafruit BME280 | 2.3.0 |
| Adafruit Unified Sensor | 1.1.15 |
| Adafruit BusIO | 1.17.4 |
| ArduinoJson | 7.4.3 |

Open `Bair1_Xiao/Bair1_Xiao.ino`, select **XIAO_ESP32S3**, and compile. CLI equivalent:

```sh
arduino-cli compile --fqbn esp32:esp32:XIAO_ESP32S3 Bair1_Xiao
```

The checked-in `WIFI_SSID`, `WIFI_PASSWORD`, `BAIR1_API_KEY` and `BAIR1_URL` are empty. With missing configuration, the sketch prints measurements over serial and does not start uploading. Serial speed: 115200. Actual device flashing remains a separate approved step.

## Server configuration — after approval only

Peter must privately provide the approved endpoint and a dedicated Bair1 developer API key with `write:readings`. Do not put credentials in GitHub, an issue, a screenshot or a shared copy of the header. Do not use AWS, Auth0, Notehub or administrator credentials in firmware.

The intended production endpoint is `https://app.bair1.live/api/readings`, but the production backend has **not** been updated for this package. Do not fill that URL until the backend deployment has been reviewed and approved. The protected staging endpoint is for automated validation; it is not a public manufacturer testing service or a substitute for production approval.

The sender uses JSON POST with `Authorization: Bearer <developer-key>`, verifies TLS certificates, and requires internet time (NTP). A saved reading returns `ok:true`, `stored:true`, `outcome:"stored"`. A safe retry returns `ok:true`, `stored:false`, `outcome:"duplicate"`. Only these acknowledgements remove a queued reading. A 409 conflict or any other failed response retains it. The older server's throttled `stored:false` response is not a successful delivery.

## Data and limits

Every two seconds the sketch reads temperature (°C), humidity (%RH), pressure (hPa), estimated altitude (m), raw ADC values, calibrated ADC pin voltage and calculated input voltage for both channels. AQI is null because no calibrated AQI/particulate measurement is available. Raw ADC and calibrated voltage use successive conversions.

The upload task is separate from sampling. Its RAM queue holds 120 readings (about four minutes). It drops new readings when full, loses queued readings on power loss/reboot, and does not queue readings collected before NTP time is available. Such readings still print to serial. A permanently rejected first event blocks later uploads; HTTP 401/403 requires correcting credentials and restarting. This is not a lossless field logger. The candidate backend retains accepted readings for 14 days after first receipt.

## Approved one-device bench test

Once Peter authorizes flashing and provides an approved server/key:

1. Confirm board, wiring, divider ratios and input voltages; compile and flash exactly one identified board.
2. Compare BME280 readings and both analogue voltages with reference instruments.
3. Confirm server read-back of capture time, sequence, BME fields and both ADC channels. An HTTP acknowledgement alone does not validate sensor accuracy.
4. Briefly interrupt Wi-Fi, restore it, and confirm backlog drain without duplicate readings.
5. Check startup/NTP, restart and buffer-full behavior; report any missing sequences and `droppedSamples`.
6. Share firmware version, build result and a redacted test summary. Keep device IDs, keys and location data private.

No board has been flashed or physically validated as part of the software verification. Hardware measurements, real Wi-Fi/TLS/NTP behavior and power-loss recovery still require the approved bench test.
