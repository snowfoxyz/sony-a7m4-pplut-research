# PPLUT message-handler frontier (2026-11-25)

Read-only static analysis. No camera write, no firmware modification, no flash.

## Finding

A7M4 reserves the same LUT-related message IDs as A7C2/FX2, but does not
register handlers for them.

### A7M4 message table

Address: `0x5D26DC8` (`.data`)

Entries are 0x18 bytes: `{message_id, handler, reserved}`.

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
0x415C  handler 0
```

### A7C2 message table

Address: `0x5EE1E88` (`.data`)

```text
0x4153  handler 0
0x4154  handler 0x253B098
0x4155  handler 0x253B164
0x4156  handler 0x26AECD8
0x4157  handler 0x26AF05C
0x4158  handler 0
0x4159  handler 0
```

Corresponding names from the A7C2 reflection/member table:

```text
0x4155  m_monitor_lut_on_msg
0x4156  m_user_lut_binary_data_msg
0x4157  m_lut_embedded_msg
```

### FX2 message table

Address: `0x61329B8` (`.data`)

```text
0x4154  handler 0x2A16CC0
0x4155  handler 0x2A16DC0
0x4156  handler 0x2A16EC0
0x4157  handler 0x2A17044
0x4158  handler 0x2A17140
0x4159  handler 0x2A17240
0x415A  handler 0x2A173C0
0x415B  handler 0x2A174C0
0x415C  handler 0x2A17640
```

FX2 registers handlers for all these IDs; A7M4 registers only 0x415A/0x415B.

## A7C2 handler shapes

The A7C2 handlers are small message-dispatch wrappers:

```text
0x253B098  ID 0x4154 -> 0x253B048
0x253B164  ID 0x4155 -> 0x253B114
0x26AECD8  ID 0x4156 -> 0x26AEADC
0x26AF05C  ID 0x4157 -> 0x26AED54
```

They call common message/serialization helpers (`0x368312C`, `0x36EB21C`,
`0x36EB3F0`, `0x2E01874`) and then a per-message body. The 0x4156/0x4157
bodies index state objects and handle LUT binary/embedded data.

A7M4 has no corresponding handler entries, so even if an AP-side UI/message
were injected, these incoming LUT-related messages would have no registered
receiver in A7M4.

## Handler body inspection (A7C2)

### 0x4155 / 0x253B164

Body `0x253B114` is small. It only calls:

```text
0x24E0404
0x24E13D4
0x24E06C4
```

and writes a one-word state value. Bounded body shows no direct
`vdf_disp_set_monitor_lut`, no CP command construction, and no LUT payload
copy. It is a model/state notification handler, not the LUT data path itself.

### 0x4156 / 0x26AECD8

Body `0x26AEADC` is large and indexes model-state accessors:

```text
0x288E2B0 / 0x288E2C4 / 0x288E2D8 / 0x288E2EC
0x288E328 / 0x288E33C / 0x288E350 / 0x288E364
```

and calls model/UI helpers:

```text
0x26A7528 / 0x26A79E8 / 0x26A8750
0x1C18494 / 0x1C18588 / 0x1C18588
0x2587384
```

It iterates 0x18-byte records and updates per-slot fields. This is consistent
with user-LUT binary-data/model update.

### 0x4157 / 0x26AF05C

Body `0x26AED54` is the same family, larger and again indexes model state /
per-slot records.

### Bounded conclusion

The A7C2 handlers are not standalone CP senders. They depend on A7C2 model/UI
objects and per-slot state. Porting the handler table alone is therefore not
sufficient; the dependent state/accessor layout must exist in A7M4 or be
ported too.

No direct CP LUT command was observed in the bounded handler bodies.

## Interpretation

This is concrete evidence for a missing **receiver layer**, not just a hidden
menu flag:

```text
A7M4: IDs reserved, handlers NULL
A7C2/FX2: IDs have real handlers
```

It is consistent with the earlier findings:

```text
A7M4 lacks m_monitor_lut_on_msg / user LUT binary / embedded LUT members
A7M4 retains lower-level PPLUT labels and CP/VDF LUT resources
```

## What implementation would require

1. Port or reimplement the missing handler entries (0x4154–0x4159) and their
   per-message bodies.
2. Verify A7M4 has the object/state fields those bodies touch; if not, port
   those too.
3. Bind the AP-side menu/candidate/commit path to these message IDs.
4. Verify CP receives and acts on the selected PPLUT; do not assume the
   monitor LUT path equals the recording PPLUT path.

This is a multi-layer port, not a one-byte patch.

## Safety

No patch, no camera write, no firmware modification. Any later attempt needs a
full eMMC backup and explicit approval.
