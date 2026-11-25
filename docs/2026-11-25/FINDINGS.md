# Findings

All addresses are static file/ELF virtual addresses, not runtime addresses.
A7M4 uses `PRODUCT_MODEL_LS`.

## 0. Scope distinction

This is about the **video-mode PPLUT1–4 / CUBE LUT** path, not Erlla's custom
photo-oriented JSON look implementation.

```text
Erlla Custom LUTS:
  new custom menu
  camera-look v2 JSON native parameters
  photo-oriented color path
  third-party appFw + CPAPP modifications

This research:
  Sony PPLUT1–4 in Movie Picture Profile
  CUBE/user LUT selection through PPLUT
  official provider/candidate/commit/sequence binding
```

Erlla's custom menu being visible in movie mode does not make it PPLUT.

## 1. Official baseline

A7M4 6.02:

```text
Menu: no PPLUT1–4
Official Picture Profile documents PP1–PP11
No official Log Shooting menu
```

A7C2:

```text
PPLUT1–PPLUT4 (only when shooting movies)
Manage User LUTs / Select LUT / Apply LUT
```

This is an official product/documentation difference, not merely a hidden
A7M4 setting.

## 2. What A7M4 still contains

```text
PPLUT1..PPLUT4 value labels
PPLUT HAITA / LOG_SHOOTING_OFF x MOVIE_MODE model-diff labels
set_pplut_link (keys 0x1F8C..0x1F8F)
selector-11 PPLUT-style renderer 0x226926C
scr7053 / scr7275 presenter classes
vdf_disp_set_monitor_lut
get_select_lut_by_pp
to_remote_event_monitor_lut_setting_all_line
CPAPP monitor/video/3D LUT resources
```

So A7M4 is not missing all LUT code.

## 3. What A7M4 lacks

UI/model member-name table:

```text
m_monitor_lut_on_msg
m_user_lut_binary_data_msg
m_lut_embedded_msg
```

A7C2 and FX2 have all three; A7M4 has none.

A7C2-only `menu::util::util_instance` signatures:

```text
get_name_resid_select_lut
get_name_resid_picture_profile
get_name_resid_iptc_slot_info
```

FX2 also lacks two of these exact signatures, so this is a model-specific UI
difference, not a universal prerequisite.

## 4. Message handler table

A7M4 `.data` table at `0x5D26DC8`:

```text
0x4153  handler 0
0x4154  handler 0
0x4155  handler 0
0x4156  handler 0
0x4157  handler 0
0x4158  handler 0
0x4159  handler 0
0x415A  handler 0x26CD7A0
0x415B  handler 0x26CD81C
```

A7C2 `.data` table at `0x5EE1E88`:

```text
0x4154  handler 0x253B098
0x4155  handler 0x253B164   # m_monitor_lut_on_msg
0x4156  handler 0x26AECD8   # m_user_lut_binary_data_msg
0x4157  handler 0x26AF05C   # m_lut_embedded_msg
```

FX2 `.data` table at `0x61329B8` has handlers for `0x4154..0x415C`.

A7M4 has handler infrastructure (its own 0x415A/0x415B handlers follow the same
wrapper shape) but no registrations for the LUT-related IDs.

## 5. Product/menu gate is not sufficient

A7M4 `PRM_FEATURE_MANAGER_menu_item` LS cell:

```text
0x37B7C50 = 0  (TYPE_MENU_ITEM_LS)
```

A historical experimental custom baseline changed only this byte:

```text
0x37B7C50: 0 -> 25
```

Observed result:

```text
PPLUT1–4 appeared in Picture Profile
but were grey / unusable
Log Shooting was not available
```

This is a custom-baseline result, not official 6.02, but it rules out the idea
that a single menu-catalogue switch is sufficient.

Additional policy:

```text
main PP key 0x1185 validator mask 0x4AB7CB0
accepts 0..11; rejects 12..15
```

FX2 supports PPLUT even though its main-route validator also rejects 12..15,
so the validator mask alone is not the functional PPLUT gate either.

## 6. Lower-level LUT application sequence exists

A7M4 contains `model::sequence_set_lut_data`:

```text
typeinfo name 0x4BE5F00: N5model21sequence_set_lut_dataE
vtable/table 0x59D6638
constructor-like 0x3527098
deleting destructor 0x35270AC
run() 0x3527170
```

`run()` reaches:

```text
0x35A904C -> 0x360CDA8 -> mpr::SetLutData
```

A7M4 PLT:

```text
0x1718040  mpr::SetLutData(setLutDataInParam const&)
```

A7C2 has the same class and a corresponding run path; its type ID differs.

## 7. Sequence registry

A7M4 `.data.rel.ro` around `0x58498E8` contains sequence `{type_id, name}`
entries:

```text
0x5849918  0x16000001BB  sequence_set_embedded_lut
0x5849920  name 0x50AB669 sequence_set_lut_data
0x5849928  0x16000001BC
0x5849930  name 0x50AB67F sequence_set_disp_lut
0x5849938  0x22000001BD
```

So A7M4 registers:

```text
sequence_set_lut_data -> type ID 0x1BC
```

A7C2 registers:

```text
sequence_set_lut_data -> type ID 0x1B7
```

A7M4 also has a `.rodata` metadata entry at `0x4FED3C4`:

```text
0x16000001BC + 0x0000000000030D1
```

## 8. Current hard boundary

The sequence registry is not referenced by any direct code xref:

```text
A7M4 .text ADRP page 0x5849000 -> 0 hits
A7C2 .text ADRP page 0x58D7000 -> 0 hits
A7M4 .text ADRP page 0x4FED000 -> 0 hits
```

The A7M4 constructor `0x3527098` is only referenced by its deleting destructor.
The vtable has only a self-relative relocation.

A7C2 shows the same pattern even though A7C2 supports PPLUT. Therefore the
loader/dispatcher is likely:

```text
dynamic / section-scanning
or a runtime registration path with no direct static xref
```

This is the current blocker.

## 9. Negative results / retractions

```text
0x3797480 is a date_time_add_seconds wrapper, not a sequence service request
0x11A7 is Gamma0 candidate key, not an alternate PPLUT route
single product/menu gate is not sufficient
validator mask alone is not the functional PPLUT gate
A7M4 handler table alone is not sufficient; handlers depend on model/UI state
no A7M4 natural input/binding log was obtained
A7M4 service mode does not expose /log runtime files
```

## 10. What this means

A7M4 has:

```text
lower-level PPLUT labels
LUT data sequence
SetLutData
handler infrastructure
model-state accessor infrastructure
```

A7M4 lacks / has not resolved:

```text
PPLUT main-PP candidate/provider
UI/model members
message handler registrations
dynamic sequence loader / dispatcher
proof of CP-side PPLUT application
```

This is a multi-layer product/UI/message integration gap, not a single hidden
flag.

## 11. External update (2026-11-25)

`leogenot/sony-a7iv-feature-research` updated its docs and moved the roadmap to
`leogenot/sony-a7m4-custom-firmware-open-gate`.

Relevant points:

```text
new docs/CINEMA_FEATURES_WEB_RESEARCH.md (2026-11-25)
roadmap lists Log shooting mode / Flexible ISO as a Grade-D
  "Unlock, else build (mostly UI)" candidate
  next step: after the shutter-angle pilot
roadmap does not list PPLUT1–4
A7 IV research README still says Custom LUT runtime gate is unresolved
```

This supports treating Log Shooting as a possible precursor/unlock target, but
it does not provide PPLUT provider/commit or sequence-loader evidence.
