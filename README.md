ESP Thread Border Router release builds for the ESP32-S3 + ESP32-H2 dev board.

This repo only contains GitHub Actions workflows that fetch `esp-idf` and `espressif/esp-thread-br`, apply the 4MB/8MB and Wi-Fi/Ethernet options, then publish a release with ready-to-flash binaries.

## Build a release

1. Open Actions → `Build Thread Border Router Release`.
2. Run workflow and pick:
   - `idf_ref` (required, no default)
   - `flash_size` (`4mb` or `8mb`)
   - `mode` (`wifi` or `ethernet`)

The workflow publishes a GitHub Release with two artifacts:
- `*-single.tar.gz` (single merged image)
- `*-split.tar.gz` (split binaries + `flasher_args.json`)

## Flashing

Single image:

```bash
esptool.py --chip esp32s3 --port /dev/ttyUSB0 write_flash 0x0 esp_ot_br_full.bin
```

Split:

```bash
esptool.py --chip esp32s3 --port /dev/ttyUSB0 \
  write_flash @flasher_args.json
```
