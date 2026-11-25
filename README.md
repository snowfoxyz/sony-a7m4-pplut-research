# sony-a7m4-pplut-research

Read-only research notes on Sony A7M4 6.02 PPLUT and Log Shooting behavior.

## Documents

- [Research status (2026-11-22)](RESEARCH-STATUS-2026-11-22.md)
- [Static analysis summary (2026-11-22)](docs/STATIC-ANALYSIS-2026-11-22.md)
- **[2026-11-25 update: findings, sequence registry, open questions](docs/2026-11-25/README.md)**
- [Contributing / evidence requests](CONTRIBUTING.md)

## Scope: JSON photo LUT vs video PPLUT/CUBE LUT

This repository is about the **video-mode PPLUT1–4 / CUBE LUT** path, not
Erlla's photo-oriented Custom LUTS implementation.

```text
Erlla Custom LUTS:
  custom menu
  camera-look v2 JSON native parameters
  photo-oriented color path
  third-party appFw + CPAPP modifications

This research:
  Sony PPLUT1–4 in Movie Picture Profile
  CUBE/user LUT selection through PPLUT
  official provider/candidate/commit/sequence binding
```

Erlla's custom menu appearing in movie mode does not make it PPLUT.

## Scope and safety

- Static, read-only analysis only.
- No firmware images, partition data, keys, device identifiers, or flashing procedures.
- Addresses and key values are research leads, not ready-to-apply patches.
- No camera I/O, property writes, firmware modification, or flashing was performed.

## License

MIT — see [LICENSE](LICENSE).
