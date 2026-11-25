# Cross-model PPLUT gate differential conclusion

Date: 2026-11-23  
Scope: read-only static analysis and existing evidence audit; no camera, no firmware modification/flashing.

## 1. Module map (actual SHA-256)

| model | appFw.so SHA-256 | CPAPP-B.BIN SHA-256 |
|---|---|---|
| A7M4 official | `1e1b9202eaa69198f20df03dbfdab2f103b206cc4ebcd2e7bd817e6a710018a2` | `7b9b137d696e5972d20db7f9119076172ed8e0730805d7f0b4c31b87689e2c2b` |
| A7C2 | `2ea02f7ecbc5bb917316fe8ef4ce5ffd87eb66efaae78ba2ce11420552df539b` | `f5a477588698ef965748903b8959a39ce4f59ffd4aee4d4234ef1670f56a1428` |
| FX2 | `b111be7d3cc36bd031e8a67e36d2b870d6833c1c8326a5b26b72e1b38619a17f` | `eba723db657117a36d9593dacf640840aca893c0d6ee5d728de9d1996b8c5e29` |

Only `appFw.so` and `CPAPP-B.BIN` are present in the `components-*` working copies; other module hashes are not restated here.

## 2. Positive-sample assets shared by A7M4

Existing cross-model evidence (audited, hashes match current files):

- A7M4 selector-11 PPLUT renderer `0x226926C` is structurally homologous to FX2 `0x2526CD4`: 16 rows, PPLUT1..4 label token family, import-status labels, zeroed auxiliary/state arrays.
- A7M4 has PPLUT label table entries and a `scr7275_0004_presenter` class.
- A7M4 has a type-`0x144` registry row (callback/factory/marker) that reaches `scr7275_0004_presenter`.
- A7M4 and FX2 share the identical `w0==3 / PPLUT 12..15` selector-state predicate, and A7M4 names the 12..15 branch through `set_pplut_link`/keys `0x1F8C..0x1F8F`.

So the absence of PPLUT on A7M4 is **not** explained by a simple missing renderer, label set, presenter class, or state predicate.

## 3. Concrete differences found

### 3.1 Selector-11 template type

| model | selector-11 template | type | route |
|---|---|---|---|
| A7M4 | `0x3EAEBB8` | `0x146` | type-0x146 family -> `scr7053_0004_presenter` |
| A7C2 | `0x3DD1068` | `0x144` | type-0x144 family -> `scr7275_0004_presenter` |
| FX2 | `0x3FDCAC8` | `0x144` | type-0x144 family -> `scr7275_0004_presenter` |

A7M4 still has the type-`0x144` registry/class; its selector-11 template simply selects the `0x146` variant. This is a real UI-routing difference, not merely a missing class.

The template records were dumped directly: each model has exactly one selector-11
template, with identical `selector=11`, IDs `0x021C6242/0x021C6243`, `count=16`,
and `0` at `+0x14`. A7M4 differs only in `type=0x146` and its per-model UI
resource-key range (`0x868..` vs A7C2 `0x808..`, FX2 `0x80E..`). No second
selector-11 template exists in any model. Evidence:
`evidence-crossmodel-selector11-template-type-20261123.txt` (SHA-256
`78c1165d370e22465c489bb1972aad3d39f082e03b8f49b2213d35f3347403b5`).

A7M4's `scr7275_0004_presenter` refresh at `0x1EB74A0` is also
instruction-for-instruction identical in mnemonic shape to A7C2 `0x1E2BA00` and
FX2 `0x2160DB0`: 74 instructions each, and all three full mnemonic sequences
compare equal. A7M4 therefore contains the same-shaped PPLUT screen
implementation but does not bind selector 11 to it.

However, A7M4's type-`0x146` selector event is separately traced to PPLUT profile-link keys `0x1F8C..0x1F8F`, not to main PP key `0x1185`; it cannot be equated with the positive models' PPLUT screen route.

Important correction from an existing A7M4 evidence file
(`evidence-a7m4-type146-to-scr7053-selector11-vtable-correction-20261118.txt`):

```text
type-0x146 callback 0x1DF3920
  -> constructor 0x1DF3778 -> scr7053_0004_presenter
  -> vtable slot +0x10 = 0x1DF3B88
  -> 0x2269D28 -> dispatcher 0x22697B4
  -> selector-11 PPLUT renderer 0x226926C
```

That path also uses the same selector-11 static template `0x3EAEBB8` and runtime
root `0x07E820A0+0x1FC`. So A7M4 already has a registered
presenter-to-selector-11-PPLUT-renderer chain. The `0x146` vs `0x144` difference
is a presenter-class difference, **not** proof that A7M4 lacks a PPLUT screen
route.

The type-path constructors also share the same provider ABI:
A7M4 `0x1DF3778`, A7C2 `0x1D48038`, and FX2 `0x2082830` all store the dynamic
provider interface at `this+0x30`, call provider vtable slot `+0x30`, call slot
`+0x10` with row stride `0x10` and a count field (`[this+0x40]` on A7M4/A7C2,
`[this+0x28]` on FX2), and finally tail-call provider slot `+0x50`.
Evidence: `evidence-crossmodel-type-path-provider-abi-20261123.txt`
(SHA-256 `72e5d055a1d3ae8d78445f11e465d85796fab521c5a080d98d8e4dca5a1db367`).

Cross-model check: FX2 `0x20051C8` and A7C2 `0x1CC9AC8` also set typeinfo `23scr7053_0004_controller` (vtables `0x5AE4D18` / `0x5B14140`) with the same `+0x30` getter `ldr w0,[x0,#400]`. The filler key family shifts by exactly `+0x0A` per model: A7C2 `0x367..0x36C`, FX2 `0x371..0x376`, A7M4 `0x37B..0x380`. So raw key numbers are model-specific and cannot be compared directly; this shift is consistent with ordinal allocation, not a proven PPLUT gate. Evidence: `evidence-crossmodel-controller-keyfamily-20261123.txt` (SHA-256 `32e834f7e889154ba01067013df209f2f37c60fef5fd31dd82bf81cf851196fd`).

A7M4's **controller/workspace** class is identified statically (but this is not proven to be the `x1` provider): factory
`0x1D79C40` allocates `0x7614430+index*0x5000`, initializer `0x1D79AAC` sets
vptr `0x59A5C78` with typeinfo `scr7053_0004_controller`. Its `+0x30` getter is
`0x2265924` (`ldr w0,[x0,#400]`); its `+0x10` filler is `0x1AE02E8`, switching
over key families `0x100037B..0x1000380`. The concrete runtime instance/values
and A7C2/FX2 provider-class matching remain unresolved. Evidence:
`evidence-a7m4-scr7053-controller-provider-20261123.txt`
(SHA-256 `ad841ebbd6aab0cca6dad24bcba92823b4df055f3c189a5e09b7fc2ed11e9cf1`).

A numeric match initially suggested a provider-to-main-PP edge, but it was a
**hex/decimal confusion**: the documented main-PP ordinals are hex `0x895/0x896`,
i.e. decimal 2197/2198, while the controller filler key is hex `0x380`, decimal
896. `0x896 != 0x380`. The main-PP edge is **retracted**. Independently, the
`controller+0x170` sub-object itself resolves to typeinfo
`master_input_eventgp_MODEL_bt_streaming_screen_screengp_A_controller` (an
input-event controller), so this branch is not a PP value object either. Evidence:
`evidence-a7m4-controller-key380-mainpp-20261123.txt`
(SHA-256 `7dea549f7f61bb995b3613039dde6c298df738eaa7d374511f92f74942f7a0f0`).

A **PP-side getter/notification path** for the correct ordinals exists on A7M4: `0x1CC52D4` emits ordinal `0x896` (main PP `0x1185` family), with sibling wrappers `0x1CC51E0` (`0x895`) and `0x1CC5394` (`0x897`). The slot record handler `0x1C9C6D4` reads a fixed global cell `0x7FF2DF8` via `0x28FB9F8` and returns it; the wrapper then notifies by ordinal. The cell is a state mirror: copy function `0x2B1B61C` mirrors from source cell `0x84E9A50+0xD8` (0x84E9B28). A corrected multi-add scanner found 91 code references to base `0x84E9A50` in the `0x2B18xxxx` state-accessor family; the exact `+0xD8` writer is still not pinned down. The copy entry is referenced only via static function-table slot `0x56823D0`, whose owner/index semantics are unresolved. No direct user-selection setter to a PP property key was found. They are referenced only through data callback slots `0x595F040/0x5951C88/0x593DCB8` (no direct BL callers). This establishes a real PP value-read/state-mirror/notification path, but not the provider-to-value binding or a user setter/LUT commit. Evidence: `evidence-a7m4-pp-ordinal-event-wrappers-20261123.txt` (SHA-256 `958dd475292faa1b06be779bc2dc781dde824045ff58596d4e8d16ff14400da5`).

### 3.2 Known product-class gate is not a differentiator

The `scr7053` vtable `+0x68` callback is a shared product-class Boolean gate whose table contents align by product-class name across A7M4 and FX2.

- A7M4 `PRODUCT_MODEL_LS` index 26: both A/B entries zero.
- FX2 `PRODUCT_MODEL_LS` index 25: both A/B entries zero.

FX2 is PPLUT-capable while its LS table entries are also zero, so this known product-class gate cannot by itself explain FX2-vs-A7M4 PPLUT availability.

## 4. The unresolved link in the positive samples

The positive models do **not** provide a closed candidate-provider path either:

- FX2 selector-11 renderer reads a BSS composite at `0x84FD3A0+0x1FC`; exact static scans found no producer storing that BSS root. The composite is populated at runtime by an unlocated producer.
- A7C2 has PPLUT enum labels and a permissive route validator, but no proven menu candidate provider consuming the enum table.
- A7M4 has no live `view_list_pictureprofile` object in the current QEMU state; ppdump scanned all 222 rw regions and found no vptr. Statically, its controller/workspace class is identified as `scr7053_0004_controller` with key-family fill dispatch, but the old ABI audit shows this controller's `+0x30` returns a 32-bit state value and `+0x10` takes a context/key, not the constructor's expected provider contract (`+0x30` root pointer, `+0x10` count/stride/buffer). So the `x1` provider identity remains unknown.

Thus the candidate/menu provider boundary is unresolved in **all three** models; it is not a unique A7M4 deficiency demonstrated by static evidence.

### 4b. Static frontier (2026-11-23)

The PP state block `0x84E9A50` has 91 direct code references (corrected
multi-add scanner), but only reads of `+0xD8` were found among them; no direct
base-relative store to `0x84E9B28` exists. The writer is therefore behind a
computed alias, struct/memcpy, callback table, or runtime registration. This is
the precise static frontier. Evidence:
`evidence-a7m4-stateblock-d8-frontier-20261123.txt` (SHA-256 `2a2068bbb6bcaff3b4853d82a37cd82f28ed45efe68f33157242c3bc33a2a2eb`).

### 4c. Minimal snapshot fields

If runtime evidence is later supplied, the minimum is: model/firmware hash,
appFw base, PP menu state, `0x84E9B28` value, writer PC/stack/registers,
`scr7053_0004_controller` controller rows (not proven `x1` provider), PP ordinal `0x896`/key `0x1185`
before/after, and LUT commit fields.

## 5. Final classification

For the proposition "a single product/menu gate can enable usable PPLUT1–4":

**门控不够 (gate is not sufficient).**

Evidence:

- A7M4 is `PRODUCT_MODEL_LS`; `menu_item` LS cell `0x37B7C50 = 0`.
- Historical DOC-probe changed that cell to 25; the user's experiment log records
  PPLUT1–4 **appearing but grey/unusable with Log Shooting Off** (custom
  baseline, not official 6.02).
- Independent static policy: main PP key `0x1185` validator mask `0x4AB7CB0`,
  count 16, accepts 0..11 and **rejects 12..15**.

So visibility and operational validity are separate gates; a single catalogue or
product-ID change is not sufficient. The provider/candidate source and click/
commit handler remain unresolved too.

Caveat: the historical probe used a custom baseline and was not independently
repeated on official 6.02, so the exact official result is not proven. But the
"single gate sufficient" hypothesis is contradicted by the available evidence.

Therefore: **do not change a product ID / single catalogue byte / flash the
camera; no patchpoint is justified.**
Evidence: `evidence-final-gate-assessment-20261123.txt`.

## 6. Minimum evidence still needed

1. FX2/A7C2 menu-open runtime snapshot:
   - selector-11 row values/count/flags;
   - candidate provider object/array and its producer;
   - BSS composite `0x84FD3A0+0x1FC` contents and writer PC/stack.
2. A7M4 menu-open runtime snapshot:
   - live `view_list_pictureprofile` address/vptr;
   - `+0x2258[11]`, `+0x22B4`, `+0x22B0`;
   - candidate provider object and writer.
3. Runtime proof of what selector-11 template type `0x146` vs `0x144` actually instantiates on each model.
4. Product/capability state actually selected on FX2 vs A7M4 at runtime.

Without these, the "single product gate" hypothesis remains unverified, and a software-only bypass cannot be responsibly proposed.

## 7. Safety

No official firmware/camera modification; no patch/flash; no real camera I/O.


























