# Open questions and minimum next evidence

## Scope clarification

Erlla's Custom LUTS and this research are not the same feature.

```text
Erlla:
  photo-oriented native JSON look parameters
  custom "Custom LUTs" menu
  third-party appFw/CPAPP implementation

This research:
  video-mode PPLUT1–4
  CUBE/user LUT through PPLUT
  official provider/candidate/commit/sequence path
```

Do not use Erlla's photo JSON LUT results as evidence that video PPLUT is
implemented or understood.

## The main blocker

A7M4 has `model::sequence_set_lut_data` and its registry entry, but no static
code xref to the registry/constructor. The dynamic loader/dispatcher that
instantiates type ID `0x1BC` is unresolved.

Without that, no responsible PPLUT patch can be designed.

## Minimum evidence needed

### A. Positive runtime sample (highest value)

From an A7C2 or FX2 (official firmware, read-only):

1. Confirm movie-mode PPLUT1–4 menu appears and is selectable.
2. Record whether PPLUT selection requires Log Shooting, Cine EI, or other mode.
3. Read-only capture before/after PPLUT selection:
   - relevant backup properties;
   - service-mode files;
   - UART/serial logs if available.
4. Look for:
   ```text
   0x4155 / 0x4156 / 0x4157
   sequence_set_lut_data
   type ID 0x1B7 (A7C2) or 0x1B6/0x1B8 neighbours
   mpr::SetLutData
   ```
5. Report the tested camera's appFw/CPAPP hashes.

### B. Dynamic dispatcher anchor

Any of:

```text
a section-scanning sequence loader symbol
a registration call or init-array function
a runtime trace showing who reads the sequence registry
a type-ID dispatch table with a code reference
```

### C. CP path proof

```text
PPLUT selection -> which AP message/sequence
-> mpr::SetLutData or another API
-> CP LUT/gamma/grid consumer
```

### D. Product-gate semantics

```text
what exactly menu_item LS=0 removes
what Log Shooting model-diff LS omission changes at runtime
whether any official product class exposes PPLUT on the same hardware
```

## Useful experiments on A7M4 only

Read-only:

```text
service-mode file/backup reads (already done partially)
official UI behavior
```

Destructive / high-risk:

```text
debug custom firmware to observe registry loader
```

The latter requires a full eMMC/Fastboot backup and explicit recovery plan.
Do not attempt as a first step.

## Known negative results to avoid repeating

```text
date_time_add_seconds wrapper is not the sequence request
0x11A7 is Gamma0, not PPLUT
single menu-catalogue byte does not make PPLUT usable
validator mask alone is not the PPLUT gate
handler table alone is not sufficient
A7M4 /log is not readable through service mode
```

## What a contributor should NOT send

```text
camera backup
firmware image / DAT
decryption keys
serial number
personal account data
photos with personal metadata unless stripped
```

A short menu video, property diff, UART log, and hashes are sufficient.
