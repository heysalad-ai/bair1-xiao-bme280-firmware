# Verification — 30 October 2026

This package was prepared for manufacturer review. Production deployment and device flashing remain on hold.

- The XIAO ESP32-S3 sketch compiled with its networking code enabled using synthetic configuration: 1,041,172 bytes flash (31%), 48,120 bytes static RAM (14%). Public configuration values remain empty.
- The actual sketch's payload and acknowledgement functions compiled natively against ArduinoJson. Actual hosted staging replies for stored / duplicate / conflict were accepted / accepted / rejected correctly.
- An isolated hosted Bair1 staging backend retained 600/600 out-of-order backlog readings and 30/30 readings sent at approximately two-second intervals. BME280, both ADC channels, raw payload and capture times were checked.
- Staging checks passed for retry deduplication, conflicts, authentication, ownership and mixed old/new history. An ingestion-paused fallback kept history readable and preserved duplicate handling after resumption.
- Local backend checks passed: 5 unit/configuration tests, 11 integration tests, production build and lint with zero errors (25 existing warnings).

These are software/protocol results. The public package has not been flashed or tested on the physical board. It does not validate sensor accuracy, wiring, voltage dividers, Wi-Fi/TLS/NTP on the ESP32, or lossless acquisition through outages/restarts. See README.md for the four-minute RAM-buffer limit and required bench checks.

The existing production endpoint must not be used until its backend update is approved and deployed. Do not infer production readiness from these staging results.
