# PPLUT: sequence registry entry for sequence_set_lut_data (2026-11-25)

Read-only static analysis. No camera write, no firmware modification.

## Verified sequence registry entries

### A7M4

`.data.rel.ro` table around `0x58498E8`:

```text
0x58498E8  0x1A000001B8
0x58498F0  name 0x50AB60E
0x58498F8  0x27000001B9
0x5849900  name 0x50AB628
0x5849908  0x1A000001BA
0x5849910  name 0x50AB64F  sequence_set_embedded_lut
0x5849918  0x16000001BB
0x5849920  name 0x50AB669  sequence_set_lut_data
0x5849928  0x16000001BC
0x5849930  name 0x50AB67F  sequence_set_disp_lut
0x5849938  0x22000001BD
0x5849940  name 0x50AB695  sequence_cube_files_list_activate
```

Thus A7M4 registers:

```text
sequence_set_lut_data -> type ID 0x1BC
```

### A7C2

`.data.rel.ro` table around `0x58D79C8`:

```text
0x58D79E8  name 0x4C06F47  sequence_set_lut_data
0x58D79F0  0x16000001B7
```

Thus A7C2 registers:

```text
sequence_set_lut_data -> type ID 0x1B7
```

## Additional A7M4 metadata table

A7M4 also has a `.rodata` descriptor table around `0x4FED3C4`:

```text
0x4FED3C4  0x16000001BC
0x4FED3CC  0x0000000000030D1
```

This pairs the same type ID with an offset/length-like metadata word.

A7C2 does not expose the same immediate `0x16000001B7` in `.rodata`; its
registry entry lives in `.data.rel.ro`.

## Consumer status

Full `.text` ADRP scans for:

```text
A7M4 .data.rel.ro page 0x5849000 -> 0 hits
A7C2 .data.rel.ro page 0x58D7000 -> 0 hits
```

and the A7M4 `.rodata` page `0x4FED000` -> 0 hits.

So the registry exists, but the consumer is not a direct ADRP/BL edge. It is
likely:

```text
dynamic/section-scanning sequence loader
or a runtime registration path
```

matching the earlier unresolved type-registry consumer boundary.

## Interpretation

A7M4 is not missing the sequence class or its registration:

```text
model::sequence_set_lut_data exists
mpr::SetLutData exists
run() exists
sequence_set_lut_data is registered with ID 0x1BC
```

What remains unresolved is the dynamic dispatcher that instantiates/executes
the registered sequence, and the PPLUT UI selection that would feed it.

This is a concrete implementation target:

```text
type ID 0x1BC + sequence registry + run() -> mpr::SetLutData
```

but the consumer edge is still dynamic.

## Next

1. Find the sequence registry loader/dispatcher (section scan / registration
   call), not a direct table xref.
2. On an A7C2 positive sample, observe which path instantiates type `0x1B7`.
3. Map PPLUT selection to A7M4 type `0x1BC` sequence input.
4. Only then design a minimal reversible integration.

No patch, no camera write, no flash.
