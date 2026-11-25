# Official PPLUT differential: A7M4 vs A7C2/FX2 (2026-11-25)

Read-only static comparison and official documentation check. No camera access,
no firmware modification, no flashing.

## Official docs

- A7M4 official Picture Profile page documents PP1–PP11 and S-Log via
  PP7/PP8/PP9; it does not document PPLUT1–4.
  https://helpguide.sony.net/ilc/2110/v1/en/contents/TP1000649066.html
- A7C2 official Picture Profile page explicitly documents:
  `PPLUT1–PPLUT4 (only when shooting movies)` and the `Manage User LUTs`
  registration path.
  https://helpguide.sony.net/ilc/2360/v1/en/contents/0412D_picture_profile.html

Thus on official documentation:

```text
A7M4: no PPLUT1..4
A7C2: PPLUT1..4 is an official movie-mode feature
```

## Official binary symbol differential

Files:

```text
A7M4 1e1b9202eaa69198f20df03dbfdab2f103b206cc4ebcd2e7bd817e6a710018a2
A7C2 2ea02f7ecbc5bb917316fe8ef4ce5ffd87eb66efaae78ba2ce11420552df539b
FX2  b111be7d3cc36bd031e8a67e36d2b870d6833c1c8326a5b26b72e1b38619a17f
```

Search results for exact printable strings:

| symbol | A7M4 | A7C2 | FX2 |
|---|---|---|---|
| `get_name_resid_select_lut` | absent | present | absent |
| `m_monitor_lut_on_msg` | absent | present | present |
| `m_user_lut_binary_data_msg` | absent | present | present |
| `m_lut_embedded_msg` | absent | present | present |
| `PPLUT1..4` labels | present | present | present |
| `set_pplut_link` | present | present | present |
| `vdf_disp_set_monitor_lut` | present | present | present |
| `to_remote_event_monitor_lut_setting_all_line` | present | present | present |

The A7M4 lacks the UI/model message-member family that the PPLUT-capable
models carry. It still has lower-level PPLUT labels, link keys, and CP/VDF
monitor-LUT functions, so this is not a total absence of LUT code.

A7C2 `get_name_resid_select_lut` is a virtual method:

```text
virtual void menu::util::util_instance::get_name_resid_select_lut(
    model::value_id_t, libra::ResID&, dict_t&)
```

A7C2/FX2 `m_monitor_lut_on_msg` is a member name adjacent to:

```text
m_user_lut_binary_data_msg
m_lut_embedded_msg
m_base_picture_setting_msg
```

These names are part of a UI/model message/member table, not merely
application strings.

## UI/model virtual interface differential

`menu::util::util_instance` RTTI-signature strings:

A7M4 has:

```text
get_name_resid_fps_select
get_name_resid_raw_file_type
get_name_resid_exposure_index
get_name_resid_ae_level_offset
...
```

A7M4 lacks these A7C2 signatures:

```text
get_name_resid_select_lut
get_name_resid_picture_profile
get_name_resid_iptc_slot_info
```

A7C2 has all three. FX2 also lacks `get_name_resid_select_lut` and
`get_name_resid_picture_profile` in this exact form, so this is a
model-specific UI implementation difference, not a universal PPLUT
prerequisite.

The stronger shared positive evidence remains the `.data.rel.ro`
member-name family:

```text
m_monitor_lut_on_msg
m_user_lut_binary_data_msg
m_lut_embedded_msg
```

present on A7C2 and FX2, absent on A7M4.

## Interpretation

Supported:

```text
A7M4 official firmware omits UI/model LUT message members present in A7C2/FX2.
A7M4 still contains lower-level PPLUT labels and monitor-LUT functions.
PPLUT availability is therefore a feature/profile-layer difference, not a
single hidden Boolean in the main PP validator.
```

Not established:

```text
exact consumer of these missing members
whether porting only the UI/message layer would make PPLUT usable
whether A7M4 CP/LUT hardware path accepts PPLUT1..4 commit
```

## Consequence for "official way"

No official A7M4 setting or menu path exposes PPLUT1–4. The official differential
shows a real UI/model feature omission relative to PPLUT-capable models. This
supports the conclusion that there is no simple official enablement flag; making
PPLUT usable would require porting/modifying UI and likely commit paths, which
is outside official stock behavior.

No patch or camera write follows from this report.
