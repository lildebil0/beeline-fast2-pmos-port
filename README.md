# beeline-fast2-pmos-port

Pre-porting reconnaissance for **Beeline Fast 2** — a Russian carrier-branded handset, hardware-relative of the Lenovo A2010 family, running on **MediaTek MT6735M**. Targets a **postmarketOS / Alpine Linux** port — primary intent is a low-power web-server / always-on Linux box.

This repo contains only **sanitized hardware description** harvested from a unit running the 4PDA-community AICP custom ROM (already rooted). No per-device IDs, no proprietary firmware blobs.

## TL;DR

| | |
|---|---|
| SoC | MediaTek **MT6735M** (Cortex-A53 x4) |
| RAM / Storage | 1 GiB / 8 GB eMMC (Toshiba 008G30) |
| Display | ILI9806E MIPI-DSI, 480×854 FWVGA, ~5.0" |
| Stock Android | 5.1.1 (this unit runs 4PDA AICP / cm_A2010) |
| Mainline kernel SoC support | **partial** (gitlab.com/mt6735-mainline/linux — UART, MMC, clocks, no GPU/modem) |
| Mainline board support | **no** |
| Bootloader | MediaTek (preloader → LK), no slots, GPT layout |
| Hardware family | rebrand-relative of Lenovo A2010 / A2010-a / A5006 |
| pmOS port status | **not started** — this repo is step 0 |

## Why this device

* 4× Cortex-A53 @ 1.0 GHz, 1 GiB RAM — passively-cooled headless Linux candidate
* 8 GB eMMC tight but enough for nginx/Caddy + small payload
* Already rooted (AICP build) — no extra unlock dance
* MTK mainline has a community effort to track, so kernel work is partly bootstrapped

## What's in this repo

```
device_info.md              Sanitized full device write-up (drop into pmOS wiki).
extracted_dts/
  beeline_fast2_mt6735m.dts Decompiled DTS extracted from boot.img kernel append
                            (single FDT at offset 0x5ae550, 23 349 bytes,
                            model=MT6735M, compatible=mediatek,MT6735).
```

## What is NOT in this repo

* Full eMMC image (`mmcblk0_full.img`, 7.5 GiB)
* `proinfo`, `nvram`, `nvdata`, `protect1/2`, `secro`, `keystore`, `oemkeystore`, `tee1/2`, `seccfg`, `frp`, `metadata` — per-device identity & calibration
* `mmcblk0boot0` (MTK preloader, may be batch-specific DDR config)
* `userdata`, `system`, `boot`, `recovery` — stock-AICP code, can be re-extracted from any unit with this ROM

If you have a Fast 2 / A2010 / A5006 (same SoC family) and want to port, extract those from **your own** device with `dd`.

## Bring-up checklist for a porter

1. Unlock / root — many MTK devices are simple, but some have ARB. AICP installer typically handled this. If your unit is on stock, use SP Flash Tool with a stock scatter to get to a state where `adb root` works (userdebug build).
2. UART console — `ttyMT0` @ **921600 8N1** (note the unusual high baud!). MTK UART is normally exposed on test-pads or a USB-pin combo. `printk.disable_uart=1` in stock cmdline → set to `0` for porting.
3. Build initial pmOS image based on `mt6735-mainline/linux` kernel + DTS from `extracted_dts/`.
4. `fastboot boot pmos.img` won't work cleanly on MTK fastboot — instead, flash to `boot` partition with a scatter (SP Flash Tool) or via `dd` from a chroot. **Always keep a known-good `boot.img` backup before flashing.**
5. USB-OTG networking will work via `g_ether` / `g_ncm` once kernel is up.
6. WiFi (MT6630-class chip) and modem — out of scope for the planned use case.

## References

* MediaTek MT6735 mainline kernel: <https://gitlab.com/mt6735-mainline/linux>
* Lenovo A2010 downstream kernel (3.18): <https://github.com/muralivijay/android_kernel_lenovo_a2010>
* postmarketOS Wiki, [Category:MediaTek](https://wiki.postmarketos.org/wiki/Category:MediaTek)
* 4PDA AICP for A2010 family: <https://4pda.to/forum/index.php?showtopic=756039>
* SP Flash Tool (official MTK flasher)

## License

The DTS file in `extracted_dts/` is derived from the device's running boot image (AICP/cm). Its copyright belongs to the original authors (MediaTek / Lenovo / AICP contributors). It is reproduced here for hardware-description / interoperability purposes only.

Sanitized device documentation (`device_info.md`, this `README.md`) is released under **CC0 / public domain** by the contributor.
