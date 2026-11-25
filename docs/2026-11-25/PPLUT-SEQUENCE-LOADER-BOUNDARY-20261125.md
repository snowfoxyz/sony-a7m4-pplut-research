# PPLUT sequence-loader boundary (2026-11-25)

Read-only static analysis. No camera write, no firmware modification.

## Registry table

A7M4 `.data.rel.ro` sequence registry contains `{type_id, name}` entries:

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
```

A7C2 has the same table with shifted IDs:

```text
sequence_set_lut_data -> 0x16000001B7
```

## No static consumer found

Scans:

```text
A7M4 .text ADRP page 0x5849000 -> 0 hits
A7C2 .text ADRP page 0x58D7000 -> 0 hits
A7M4 .text ADRP page 0x4FED000 -> 0 hits
```

Qword reference scan to the table/entry also found no code/data pointer
besides the table's own relative relocation.

The constructor `0x3527098` is only referenced by the deleting destructor;
there is no direct factory BL caller.

## Conclusion

The sequence class, run() path, `mpr::SetLutData`, and registry entry all exist
in A7M4. What is missing is a **dynamically discovered sequence loader /
dispatcher** that consumes the registry section.

This is a real static boundary, not a missing address in a scanner.

## Implementation consequence

A minimal patch cannot be justified from static evidence alone:

```text
we know the target class/type ID
we do not know the instantiation/dispatch contract
```

To cross the boundary, one of these is required:

1. A7C2/FX2 positive runtime sample showing how type ID 0x1B7 is dispatched.
2. A debug custom firmware/QEMU that observes the appFw registry loader.
3. A trusted static anchor for the loader (e.g., a section-scanning symbol or
   registration call) from a new artifact.

Without one, further static scanning repeats negatives.

## Safety

No patch, no camera write, no flash. Any later experiment needs a full eMMC
backup and explicit approval.
