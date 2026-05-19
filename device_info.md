# Beeline Fast 2 (Lenovo A2010 family, MediaTek MT6735M) — Device Info

Compiled from a 4PDA-AICP-rooted unit on 2026-05-19 for the purpose of seeding a postmarketOS / Alpine port. All per-device unique identifiers (serial, IMEI, MAC, modem NV) are redacted.

## Identity

| Property | Value |
|----------|-------|
| Marketing name | **Beeline Fast 2** (Russian carrier-branded device) |
| Underlying hardware | rebrand / OEM-relative of Lenovo A2010 family (A2010 / A2010-a / A5006) |
| Screen size | **~5.0"** (measured diagonal 12.5 cm), confirms Fast 2 not A2010 (which is 4.5") |
| SoC | MediaTek **MT6735M** (a.k.a. mt6735, mt6735m board variant) |
| CPU | 4× ARM Cortex-A53 — ARMv8, hardware 64-bit capable (stock runs armv7 userland — MTK BSP cost-cutting) |
| pmOS target architecture | **aarch64** (decision: mainline `mt6735-mainline/linux` is aarch64; no point staying on legacy 32-bit) |
| Kernel | Linux 3.10.65 (downstream MTK BSP) |
| Android (stock) | 5.1.1 — **currently running 4PDA-AICP custom ROM** (cm_A2010 base) |
| RAM | 1 GiB (959 808 KiB usable) |
| eMMC | 7.5 GiB (7 634 944 KiB), Toshiba 008G30 (manfid 0x11) |
| Display | ILI9806E controller, 480×854 FWVGA, 200 dpi MIPI-DSI panel (`lcm=1-ili9806e_fwvga_qc_vdo_tongxingda_for_a5006`) |
| SIM | dual-SIM (`persist.radio.multisim.config=dsds`) |
| Modem baseband | MOLY.LR9.W1444.MD.LWTG.MP.V16.P4 (2015-11-06) |
| Camera HAL | mt6735m |

Note: `ro.product.*` props show `LENOVO/cm_A2010` because the device is currently running the AICP custom ROM from a Lenovo A2010 base. Stock Beeline firmware would identify differently — this is a 4PDA-community port shared across the MT6735M-Lenovo-family.

## Boot chain / cmdline (sanitized)

```
console=tty0 console=ttyMT0,921600n1
root=/dev/ram
vmalloc=496M
slub_max_order=0 slub_debug=O
androidboot.hardware=mt6735
bootopt=64S3,32N2,32N2
lcm=1-ili9806e_fwvga_qc_vdo_tongxingda_for_a5006
fps=6658
vram=5832704
printk.disable_uart=1
bootprof.pl_t=9899 bootprof.lk_t=1504
boot_reason=0
androidboot.bootreason=power_key
gpt=1
```

* Serial console: `ttyMT0` (MediaTek UART) @ **921600 baud, 8N1**. No earlycon parameter — kernel may need patch for very-early debug.
* `gpt=1` confirms GPT layout (not legacy MTK pmt).
* `printk.disable_uart=1` — UART is silenced at runtime by default; for porting set this to 0.
* `vram=5832704` (5.5 MiB) — display framebuffer reservation.
* `lcm` value contains hardware-specific panel descriptor; `a5006` suffix is the panel calibration ID, not the device model.

## Partition layout (`/dev/block/platform/mtk-msdc.0/by-name/`)

No A/B slots (legacy MTK pre-Treble layout).

| Partition | mmcblk0pN | Size KiB | Notes |
|-----------|-----------|---------:|-------|
| `proinfo`     | p1  |    3072 | Calibration data + IMEI, unique per device |
| `nvram`       | p2  |    5120 | Modem NV with MAC/IMEI |
| `protect1`    | p3  |   10240 | Protected user data |
| `protect2`    | p4  |   10240 | Protected user data |
| `lk`          | p5  |     512 | LK (Little Kernel — second-stage bootloader) |
| `para`        | p6  |     512 | misc/recovery boot mode marker |
| `boot`        | p7  |   16384 | Android boot.img (kernel + DTB-appended at offset 0x5ae550 in current image + ramdisk) |
| `recovery`    | p8  |   16384 | recovery.img |
| `logo`        | p9  |    8192 | Boot logo |
| `expdb`       | p10 |   10240 | Exception DB |
| `seccfg`      | p11 |     512 | Secure config |
| `oemkeystore` | p12 |    2048 | OEM keystore |
| `secro`       | p13 |    6144 | secure ro |
| `keystore`    | p14 |    8192 | keystore |
| `tee1`        | p15 |    5120 | TEE OS |
| `tee2`        | p16 |    5120 | TEE OS backup (identical to tee1) |
| `frp`         | p17 |    1024 | Factory reset protection |
| `nvdata`      | p18 |   32768 | NV data |
| `metadata`    | p19 |   37888 | encryption metadata |
| `system`      | p20 | 1572864 | Android system (AICP currently) |
| `cache`       | p21 |  409600 | Android cache |
| `userdata`    | p22 | 5455360 | User data |
| `flashinfo`   | p23 |   16384 | Flash info |

eMMC HW boot partitions:

* `mmcblk0boot0` — 4 MiB, contains **MTK preloader** (first-stage bootloader). Critical for unbrick.
* `mmcblk0boot1` — 4 MiB, secondary boot (different content from boot0 on this device).

## Device tree

Stock kernel has the DTB **appended to the kernel inside `boot.img`** at offset `0x5ae550` (single 23 349-byte FDT blob).

* `model = "MT6735M"`, `compatible = ["mediatek,MT6735"]`

No DTBO partition (pre-Treble).

## Bootloader / root

* Bootloader: MediaTek BootROM → preloader → LK
* `ro.debuggable=0`, `ro.secure=0` — userdebug build, `adb root` works directly (adbd elevates to UID 0 without `su`)
* AICP includes `su` at `/system/xbin/su`, context `u:r:su:s0`
* For custom flashing: SP Flash Tool with scatter file (standard MTK route)
* No fastboot mode that lets you `getvar all` here — MTK fastboot is rudimentary

## Mainline kernel support

* MT6735 mainline driver work tracked at **gitlab.com/mt6735-mainline/linux** — community effort, partial coverage (UART, basic clocks, MMC; display, modem, WiFi, GPU still WIP)
* Downstream 3.18 kernel for Lenovo A2010: **github.com/muralivijay/android_kernel_lenovo_a2010**
* Peer SC9863A (not MT6735) Samsung A03 Core TWRP tree: **github.com/almondnguyen/twrp_device_samsung_a3core** — not directly applicable but useful for MT vs SP comparison

## Local backup (NOT redistribute)

* `mmcblk0_full.img` — bit-perfect 7456 MiB image, SHA-256 `e7889f5bea8eb0ba2693a3ff9bef3da3225e910df774301fd80bb56e5fd9fa20`
* `mmcblk0boot0.img` / `boot1.img` — MTK preloader + secondary boot (different)
* 23 individual partition images per `manifest.txt`

**Never redistribute publicly:** `proinfo`, `nvram`, `nvdata`, `protect1/2`, `secro`, `keystore`, `oemkeystore`, `tee1/2`, `seccfg`, `frp`, `metadata`, `mmcblk0boot0` (preloader contains DDR config that is per-device-batch tuned, though often shared across same hardware family), `userdata`.

Safe to publish: `device_info.md` (this file), extracted DTS source.

## Status of pmOS / Alpine port

* No upstream pmOS port for Beeline Fast 2 or Lenovo A2010 family.
* MT6735 has only **basic mainline support** as of 2026 — UART, I2C, MMC, basic clocks. Display, modem, WiFi need work.
* For the headless web-server use case: USB-OTG networking + eMMC + UART + basic clocks are the only must-haves. WiFi/modem/touch can be ignored.
* Starting point: clone `gitlab.com/mt6735-mainline/linux`, add board DTS based on the dumped `fdt_00` from boot.img.

## Useful refs

* `gitlab.com/mt6735-mainline/linux` — MT6735 mainline kernel effort
* `github.com/muralivijay/android_kernel_lenovo_a2010` — downstream 3.18 kernel
* postmarketOS Wiki, [Category:MediaTek](https://wiki.postmarketos.org/wiki/Category:MediaTek)
* SP Flash Tool — official MTK flashing route
