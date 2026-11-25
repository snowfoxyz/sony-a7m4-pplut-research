# Toward implementing PPLUT on A7M4: evidence frontier (2026-11-25)

Read-only research only. No camera write, no firmware modification, no flash.

## 1. Target

Make Movie-mode Picture Profile PPLUT1–4 selectable on A7M4 6.02, ideally by
understanding and reusing the official mechanisms rather than adding a fake
menu.

## 2. Verified official baseline

Official documentation:

- A7M4 Picture Profile page: PP1–PP11 only; no PPLUT1–4.
  https://helpguide.sony.net/ilc/2110/v1/en/contents/TP1000649066.html
- A7C2 Picture Profile page: explicit `PPLUT1–PPLUT4 (only when shooting
  movies)`, `Manage User LUTs`.
  https://helpguide.sony.net/ilc/2360/v1/en/contents/0412D_picture_profile.html

Camera:

- A7M4 official 6.02 UI check: no PPLUT1–4 in Picture Profile or Fn menu.
- Read-only service-mode access works, but `/log` runtime files are absent in
  service mode (`GET_FILE2 0x10000006`), so camera logs cannot expose the
  menu/provider.

## 3. Binary layers

### Present in A7M4

```text
PPLUT1..4 value labels
PPLUT HAITA and LOG_SHOOTING_OFF x MOVIE_MODE model-diff labels
set_pplut_link (keys 0x1F8C..0x1F8F)
selector-11 PPLUT-style renderer 0x226926C
scr7053 / scr7275 presenter classes
vdf_disp_set_monitor_lut
get_select_lut_by_pp
to_remote_event_monitor_lut_setting_all_line
CPAPP monitor/video/3D LUT resources
```

### Missing in A7M4 vs A7C2/FX2

UI/model member-name table:

```text
m_monitor_lut_on_msg
m_user_lut_binary_data_msg
m_lut_embedded_msg
```

A7C2-only `menu::util::util_instance` signatures:

```text
get_name_resid_select_lut
get_name_resid_picture_profile
get_name_resid_iptc_slot_info
```

A7C2 and FX2 both have the three `m_*_lut_*_msg` members; A7M4 has none.

### Not a single switch

```text
A7M4 menu_item LS cell = 0 (TYPE_MENU_ITEM_LS)
Historical single-byte probe to 25 made PPLUT rows appear but grey/unusable
Main PP key 0x1185 validator rejects 12..15
FX2 PPLUT works even though its main validator also rejects 12..15
```

So a catalogue byte alone is insufficient; visibility and usefulness are
separate.

## 4. What implementation would require

Minimum layers:

```text
A. menu catalogue entry / visibility for PPLUT1..4
B. UI name provider and model/message members
C. main PP candidate provider and click/commit path
D. CP/LUT application path for selected PPLUT
```

Current state:

```text
A: partial evidence (historical catalogue probe shows rows possible but grey)
B: A7M4 missing member family present in A7C2/FX2
C: provider/commit still unlocated in all three models
D: A7M4 CP/Custom-LUT and VDF monitor-LUT pieces exist, binding unproven
```

No single patch or known official setting is justified.

## 5. Next evidence to collect

1. A7C2 `get_name_resid_select_lut` / `get_name_resid_picture_profile`
   consumer: vtable/reflection record, function body, menu resource IDs.
2. A7C2/FX2 `m_monitor_lut_on_msg` owning class and message dispatch; compare
   with A7M4 equivalent classes.
3. A7C2 main PP menu catalogue record for PPLUT1–4 and its selection event;
   compare with A7M4 LS catalogue.
4. Concrete A7M4 CP command reached by PPLUT selection (if any); do not assume
   `vdf_disp_set_monitor_lut` is the PPLUT recording path.
5. Only after A–D are evidenced: design a minimal reversible patch, with full
   eMMC backup and rollback. No direct camera write before that.

## 6. Safety

```text
No camera write
No property write
No firmware modification or flash
No custom firmware use as evidence
No PPLUT feasibility claim yet
```
