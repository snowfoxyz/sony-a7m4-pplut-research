# 2026-11-25 update: PPLUT / CUBE LUT research

Status: **paused at the dynamic sequence-loader boundary**

This folder extends the 2026-11-22 read-only research. The earlier documents
are retained for history; this update adds newer official-binary comparisons,
camera service-mode attempts, and the current implementation frontier.

## Scope: photo JSON LUT vs video PPLUT/CUBE LUT

These are different features:

| | Erlla Custom LUTS | This research |
|---|---|---|
| Target | Photo-oriented native **JSON** look parameters | Video-mode **PPLUT1–4** |
| Input | `camera-look` JSON native parameters | CUBE/user LUT through PPLUT |
| Menu | New custom "Custom LUTs" menu | Sony's missing PPLUT1–4 in Picture Profile |
| Implementation | Third-party appFw + CPAPP color wiring | Official provider/commit/sequence binding |
| Status | Custom firmware; runtime gate unresolved | Official path absent; binding unresolved |

Erlla's implementation is a custom photo-oriented JSON LUT system, not Sony's
video PPLUT. Its menu appearing in movie mode does not make it PPLUT.

## Key findings

- A7M4 official 6.02 has no PPLUT1–4; A7C2 official docs do (movie mode only).
- A7M4 still contains PPLUT labels, HAITA labels, `set_pplut_link`, a
  selector-11 PPLUT-style renderer, VDF monitor-LUT code, and CP LUT resources.
- A7M4 lacks the UI/model members `m_monitor_lut_on_msg`,
  `m_user_lut_binary_data_msg`, `m_lut_embedded_msg` present on A7C2/FX2.
- A7M4 message table `0x4154–0x4159` has NULL handlers; A7C2/FX2 have handlers.
- A7M4 has `model::sequence_set_lut_data`, `mpr::SetLutData`, and a registry
  entry for type ID `0x1BC`; A7C2 uses `0x1B7`.
- No static consumer/dispatcher xref for the sequence registry was found.
- Historical one-byte menu-catalogue probe made PPLUT rows visible but
  grey/unusable. A single product/menu gate is not sufficient.

## Files

- [FINDINGS.md](FINDINGS.md) — detailed evidence and addresses
- [OPEN-QUESTIONS.md](OPEN-QUESTIONS.md) — unresolved questions and next evidence
- [OFFICIAL-PPLUT-DIFFERENTIAL-20261125.md](OFFICIAL-PPLUT-DIFFERENTIAL-20261125.md)
- [PPLUT-MESSAGE-HANDLER-FRONTIER-20261125.md](PPLUT-MESSAGE-HANDLER-FRONTIER-20261125.md)
- [PPLUT-SEQUENCE-REGISTRY-20261125.md](PPLUT-SEQUENCE-REGISTRY-20261125.md)
- [PPLUT-SEQUENCE-LOADER-BOUNDARY-20261125.md](PPLUT-SEQUENCE-LOADER-BOUNDARY-20261125.md)
- [PPLUT-IMPLEMENTATION-FRONTIER-20261125.md](PPLUT-IMPLEMENTATION-FRONTIER-20261125.md)
- [CROSSMODEL-CONCLUSION-20261123.md](CROSSMODEL-CONCLUSION-20261123.md)
- [CAMERA-READONLY-ULOGIO-20261125.md](CAMERA-READONLY-ULOGIO-20261125.md)

## External update (2026-11-25)

`leogenot/sony-a7iv-feature-research` moved its roadmap to
`leogenot/sony-a7m4-custom-firmware-open-gate`.

Relevant points:

```text
new docs/CINEMA_FEATURES_WEB_RESEARCH.md
roadmap lists Log shooting mode / Flexible ISO as a Grade-D
  "Unlock, else build (mostly UI)" candidate
  next step: after the shutter-angle pilot
roadmap does not list PPLUT1–4
A7 IV research README still says Custom LUT runtime gate is unresolved
```

This supports treating Log Shooting as a possible precursor/unlock target, but
does not provide PPLUT provider/commit or sequence-loader evidence.

## Safety

No firmware, backup, key, serial number, or write/flash procedure is included.
All work described here was read-only.
