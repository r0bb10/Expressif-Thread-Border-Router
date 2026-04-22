# ESP Thread Border Router Artifact Builder

This repo contains a GitHub Actions workflow for building ready-to-flash ESP Thread Border Router firmware for the official ESP32-S3 + ESP32-H2 Thread Border Router development board.

The workflow fetches Espressif's `esp-idf` and `espressif/esp-thread-br`, applies this repo's build options, patches ESP-IDF Wi-Fi reconnect handling for Wi-Fi builds, then uploads one merged full flash binary as a workflow artifact. It does not publish GitHub Releases.

## How to Build an Artifact

1. Clone or fork this repo into your GitHub account.
2. Push it to GitHub so Actions can run.
3. Open Actions -> `Build Thread Border Router Artifact`.
4. Select `Run workflow`.
5. Choose the build options.
6. Download the generated artifact from the completed workflow run.

## Workflow Options

- `idf_ref`: ESP-IDF ref to build against. Default: `v5.5.4`.
- `br_ref`: `espressif/esp-thread-br` ref to build against. Default: `v1.3`.
- `flash_size`: target flash size, either `8mb` or `4mb`.
- `mode`: backhaul mode, either `wifi` or `ethernet`.
- `wifi_setup`: Wi-Fi setup style, ignored when `mode=ethernet`.
- `wifi_ssid`: required only when `mode=wifi` and `wifi_setup=credentials`.
- `wifi_password`: optional when using credentials; leave empty for an open network.

Wi-Fi setup modes:

- `softap`: Wi-Fi is enabled, SoftAP provisioning is enabled, and no SSID/password is baked into the firmware.
- `credentials`: Wi-Fi is enabled, SoftAP provisioning is disabled, and the selected SSID/password are baked into the firmware.

Ethernet mode disables Wi-Fi entirely and does not require any Wi-Fi configuration.

## Wi-Fi Reconnect Patch

For Wi-Fi builds, the workflow applies `patches/idf-wifi-infinite-reconnect.patch` to ESP-IDF and sets `CONFIG_EXAMPLE_WIFI_CONN_MAX_RETRY=-1`.

This makes Wi-Fi reconnect retry indefinitely. It applies to both Wi-Fi SoftAP provisioning builds and Wi-Fi credential builds, so provisioned or baked credentials can recover from AP outages without stopping after the default retry limit.

## Artifact

The workflow uploads only one merged full binary:

- `esp-thread-br-<flash_size>-wifi-softap.bin`
- `esp-thread-br-<flash_size>-wifi-credentials.bin`
- `esp-thread-br-<flash_size>-ethernet.bin`

Split binaries and `flasher_args.json` are intentionally not uploaded.

## Flashing

Download the artifact from the workflow run, unzip it if your browser downloads the artifact as a zip, then flash the `.bin` at offset `0x0`:

```bash
esptool.py --chip esp32s3 --port /dev/ttyUSB0 write_flash 0x0 esp-thread-br-8mb-wifi-softap.bin
```

Replace the port and filename with the values for your system and selected build options.
