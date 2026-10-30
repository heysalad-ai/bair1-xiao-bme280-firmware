# Bair1 XIAO ESP32-S3 — BME280 and two analogue inputs

**Backend deployed and verified: `https://app.bair1.live/api/readings`. Device flashing remains ON HOLD until Peter explicitly approves a specific board for the bench test.** You may review the source, prepare configuration and compile now. This package does not authorize flashing hardware.

**生产后端已部署并验证。设备烧录仍暂停：在 Peter 明确批准具体电路板之前，请勿烧录。现在可以审核代码、准备配置并编译。**

[中文厂家说明](README_CN.md) · [Verification results](VERIFICATION.md)

This public package contains no Wi-Fi password or API key. The upload endpoint is deliberately blank in the header; configure it as shown below, and keep the private configured copy out of GitHub.

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

## Prepare the production configuration; flashing remains on hold

The production endpoint is `https://app.bair1.live/api/readings`. Peter must privately provide a dedicated Bair1 developer API key with `write:readings`. Do not put credentials in GitHub, an issue, a screenshot or a shared copy of the header. Do not use AWS, Auth0, Notehub or administrator credentials in firmware.

The production backend now supports every accepted two-second reading and duplicate-safe retries. Use the production endpoint above, not a protected staging URL. Prepare a private header with your Wi-Fi details and the privately supplied developer key:

```cpp
static const char WIFI_SSID[] = "<factory Wi-Fi name>";
static const char WIFI_PASSWORD[] = "<private Wi-Fi password>";
static const char BAIR1_API_KEY[] = "<private Bair1 write:readings key>";
static const char BAIR1_URL[] = "https://app.bair1.live/api/readings";
```

These are placeholders, not working credentials. Configuring or compiling the files does not lift the flashing hold. Public copies must keep credentials empty.

The sender uses JSON POST with `Authorization: Bearer <developer-key>`, verifies TLS certificates, and requires internet time (NTP). A saved reading returns `ok:true`, `stored:true`, `outcome:"stored"`. A safe retry returns `ok:true`, `stored:false`, `outcome:"duplicate"`. Only these acknowledgements remove a queued reading. A 409 conflict or any other failed response retains it. The older server's throttled `stored:false` response is not a successful delivery.

## Factory network check (no device flashing required)

From the factory network, verify DNS and outbound HTTPS (TCP 443) to `app.bair1.live`. An unauthenticated request should reach the API and return HTTP 401:

```sh
curl -i https://app.bair1.live/api/readings
```

The device also needs NTP (UDP 123) to `pool.ntp.org` or `time.cloudflare.com`. Access from the factory network in China has not been tested. Report connectivity failures; do not disable TLS verification to work around them. The production certificate chain was verified using the exact public root certificate bundled in the firmware on 30 October 2026; ESP32 hardware TLS/time behavior still needs the approved bench test.

Common responses: 401/403 means check the key/permissions; 404 means check device ownership with Peter; 409 means an event identity was reused with different data; 503 means retain and retry. A permanent rejection can block the queue. Do not change device identity to bypass ownership.

## Data and limits

Every two seconds the sketch reads temperature (°C), humidity (%RH), pressure (hPa), estimated altitude (m), raw ADC values, calibrated ADC pin voltage and calculated input voltage for both channels. AQI is null because no calibrated AQI/particulate measurement is available. Raw ADC and calibrated voltage use successive conversions.

The upload task is separate from sampling. Its RAM queue holds 120 readings (about four minutes). It drops new readings when full, loses queued readings on power loss/reboot, and does not queue readings collected before NTP time is available. Such readings still print to serial. A permanently rejected first event blocks later uploads; HTTP 401/403 requires correcting credentials and restarting. This is not a lossless field logger. The production backend retains accepted readings for 14 days after first receipt.

## Approved one-device bench test

Once Peter authorizes flashing and provides an approved server/key:

1. Confirm board, wiring, divider ratios and input voltages; compile and flash exactly one identified board.
2. Compare BME280 readings and both analogue voltages with reference instruments.
3. Confirm server read-back of capture time, sequence, BME fields and both ADC channels. An HTTP acknowledgement alone does not validate sensor accuracy.
4. Briefly interrupt Wi-Fi, restore it, and confirm backlog drain without duplicate readings.
5. Check startup/NTP, restart and buffer-full behavior; report any missing sequences and `droppedSamples`.
6. Share firmware version, build result and a redacted test summary. Keep device IDs, keys and location data private.

No board has been flashed or physically validated as part of the software verification. Hardware measurements, real Wi-Fi/TLS/NTP behavior and power-loss recovery still require the approved bench test.
