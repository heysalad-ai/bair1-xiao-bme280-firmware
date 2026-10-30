# Verification — production backend deployed, 30 October 2026

**Backend availability is confirmed. Device flashing remains on hold pending Peter's explicit approval.**

- The reviewed backend was deployed to `https://app.bair1.live/api/readings` on Node 24 after a production-configured preflight.
- Preflight retained 12/12 distinct readings. After activation, the public production endpoint retained 30/30 readings sent at approximately two-second intervals. Retry deduplication, changed-content conflicts, authentication, ownership, capture times, BME280 and both ADC channels passed read-back checks.
- A production-configured ingestion-paused fallback was built and tested without activating it on live domains. It reads both old and new storage formats and refuses new uploads with HTTP 503. History and duplicate receipts remain intact.
- The firmware's actual acknowledgement function accepted real production stored/duplicate replies and rejected conflict replies.
- The live TLS certificate chain verified using only the public root certificate bundled with this package. This is a host-side TLS check; physical ESP32 TLS/NTP behavior remains untested.
- Earlier isolated staging retained 600/600 out-of-order backlog samples and 30/30 paced samples. Local checks passed: 5 unit/configuration tests and 11 integration tests; build and type checking passed; lint had zero errors and 25 pre-existing warnings.
- The two firmware source files are unchanged from the compiled review package: 1,041,172 bytes flash (31%) and 48,120 bytes static RAM (14%), with networking enabled using synthetic configuration. Public credentials and endpoint remain empty; use the production URL in the private configuration as described in README.md.

No devices were flashed. Physical sensor accuracy, divider ratios, real Wi-Fi/NTP behavior, factory-network access from China and loss during outages/restarts still require an approved one-board bench test. The RAM-only buffer holds about four minutes; production retention is 14 days.
