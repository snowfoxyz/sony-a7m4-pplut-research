# Contributing

The most useful contribution is **new evidence**, not a patch or a repacked
firmware.

## Please do not upload

```text
official or modified firmware images
BODYDATA.DAT / DAT files
camera eMMC/Fastboot backups
decryption keys or signing material
camera serial numbers
personal photos with EXIF
```

These can be illegal to redistribute, can identify a camera, or can harm other
users if flashed.

## What is useful

### A7C2 / FX2 owners (highest value)

An official-firmware, read-only observation:

1. Camera model, firmware version, region if voluntary.
2. `appFw.so` and `CPAPP-B.BIN` SHA-256 (hashes only; no files).
3. Movie-mode PPLUT workflow:
   - Does PPLUT1–4 appear?
   - Is Log Shooting or Cine EI required?
   - What happens when selecting a PPLUT?
4. Read-only property/backup diff before/after selecting PPLUT.
5. UART/serial or service-mode log excerpt covering the selection.
6. If possible, any log lines containing:
   ```text
   0x4155 / 0x4156 / 0x4157
   sequence_set_lut_data
   SetLutData
   ```

Do **not** write camera properties or flash firmware for this request.

### A7M4 owners

Useful:

```text
official 6.02 menu behavior (already known: no PPLUT)
service-mode read-only file/backup dumps, with identifiers removed
UART logs during normal menu navigation
```

Not useful:

```text
another "PPLUT visible but grey" custom-firmware screenshot
without runtime/CP evidence
```

### Reverse engineers

Useful:

```text
sequence registry loader/dispatcher anchor
type ID 0x1BC dispatch evidence
PPLUT message 0x4155/0x4156/0x4157 consumer
proof of CP LUT/gamma/grid application
```

## Evidence quality

For each claim, please provide:

```text
file/version
SHA-256
static address or log line
how it was obtained (read-only / UART / service mode / disassembly)
what is observed vs inferred
```

## Safety

This project is read-only by default.

Any write/flash experiment must be:

```text
on a backed-up camera
with a tested recovery path
explicitly separated from evidence claims
```

A successful custom firmware run is not proof of official PPLUT support.
