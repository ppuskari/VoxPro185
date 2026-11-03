# Ghidra 12.2 — PowerPC PEF, 68K resource and MicroGuard analysis

This is a **static reverse-engineering record**, not a complete MicroGuard emulator.

## 1. Original binary layout

| Element | Confirmed value |
|---|---|
| Mac application name | `VoxPro 185` |
| PPC data fork | 417,319 bytes |
| Header | `Joy!peffpwpc` (PEF / Code Fragment Manager) |
| PEF section 0 | 385,540-byte uncompressed native PPC code, stored at data-fork offset `0x1900` |
| PEF section 1 | 25,367 stored bytes, expands to 33,360 bytes before relocations |
| PEF loader section 2 | 6,264 bytes; 349 imports from four libraries |
| Application resource | `'Zc68'` resource ID `0`, name `MicroGuardStandalone`, **4,366 bytes** |
| Ghidra 68K import | Raw binary, 68000 big-endian 32-bit, image base `0x00000000` |

Ghidra PPC memory addresses and raw PEF code-section-relative offsets differ. **Always verify instruction bytes** against original media before translating patch offsets into the HDA.

## 2. CodeWarrior symbols and native security methods

| Runtime Ghidra address | Meaning |
|---|---|
| `1004901C` | `MicroGuardSecurity::wasDonglePresent()` |
| `1004A338` | `MicroGuardSecurity::Security()` wrapper; calls `wasDonglePresent()` and checks the result |
| `1004A34C` | `Security()` call site to `wasDonglePresent()`; used in a **later independent** experimental patch |
| `1004A484` | `InitMicroGuardPowerPC()` |
| `1004A574` | `InitMicroGuard()` wrapper |
| `1003E99C` | One recording-related `.do_key()` method |
| `1003E9E4` | Another recording-related `.do_key()` method |

The startup path also queries `keyLevel()` (CodeWarrior PPC code-relative offset `0x49DF4`), storing a result in `r23` at code-relative `0x60B0`; precise key-level meanings were **not** established.

### 68K resource loading by the PPC code

At `1004A484`, PPC `InitMicroGuardPowerPC()`:

1. Forms the resource type `'Zc68'`.
2. Calls `GetResource('Zc68', 0)`.
3. Handles Resource Manager and Memory Manager errors.
4. Calls `MoveHHi` and `HLock` for the resource handle.
5. Passes the executable resource pointer and a 17-slot procedure table to `CallUniversalProc` at `1004A528` with parameter descriptor `0xC1`.

The PPC TOC base seen in Ghidra was `10066210`. The procedure vector begins `1006D008` and ends `1006D048`; the resource handle is at `1006D04C`.

## 3. Complete 17-entry procedure table

The 68K initializer at offset `0000` uses PC-relative `LEA` and `MOVE.L` to populate the caller's table. Two earlier guessed offsets were corrected from the full disassembly: entry 4 is **`0xCF8`** and entry 15 is **`0x2F0`**.

| Index | PPC vector address | Raw 68K offset |
|---:|---|---|
| 0 | `1006D008` | `0000010E` |
| 1 | `1006D00C` | `000000AC` |
| 2 | `1006D010` | `000000CA` |
| 3 | `1006D014` | `000008E4` |
| 4 | `1006D018` | `00000CF8` |
| 5 | `1006D01C` | `00000A94` |
| 6 | `1006D020` | `00000EBC` |
| 7 | `1006D024` | `00000DC8` |
| 8 | `1006D028` | `00000B38` |
| 9 | `1006D02C` | `00000776` |
| 10 | `1006D030` | `000005DC` |
| 11 | `1006D034` | `000006AA` |
| 12 | `1006D038` | `000004E8` |
| 13 | `1006D03C` | `00000C38` |
| 14 | `1006D040` | `00000FC8` |
| 15 | `1006D044` | `000002F0` |
| 16 | `1006D048` | `00000400` |

**Index 8** correlates with the PPC `GetMemory()` wrapper using `CallUniversalProc(...,0x3E61,...)`.

## 4. 68K ADB operations and callback structure

Recognized classic Mac OS A-line trap opcodes include:

| 68K opcode | Classic ADB/Toolbox call | Example location |
|---|---|---|
| `A077` | `CountADBs` | `0x0126` |
| `A078` | `GetIndADB` | `0x0130` |
| `A079` | `GetADBInfo` | `0x01E2` |
| `A07A` | `SetADBInfo` | `0x09C8` |
| `A07B` | `ADBReInit` | `0x0186` |
| `A07C` | `ADBOp` | `0x1104` |
| `A975` | `TickCount` | `0x0BDA`, `0x0BF8`, many others |
| `A02E` | `BlockMove` | `0x10AC` |

Raw Ghidra's CPU disassembler initially marked many A-line trap instructions as unknown. Do not treat these as corruption or runtime unimplemented-instruction exceptions: classic Mac OS traps are **intended** to be interpreted by the OS. Sequential disassembly can restart after a 2-byte trap. Replacing a trap with NOP in a **separate, clearly marked analysis copy** preserves instruction alignment but not trap side effects, so resulting pseudocode needs verification against assembly.

### `MG68_GetMemory` at `0x0B38`

The recovered control flow:

1. Validates device/address input (non-positive value leads to `0x2710`).
2. Prepares request structure via helper `0x1096`.
3. Uses common ADB operation function `0x09F4`.
4. Selects among operation constants `0xA15E`, `0xAE51`, `0x21DE` when mode is `0`, `0x100`, or `0x200`; the semantic meaning of these constants remains unresolved.
5. Clears an asynchronous completion word, reads `TickCount`, establishes a 20-tick deadline (~0.33 s at nominal 60 ticks/s), registers/initiates a request via `0x097A`, and polls for completion/error/timeout.
6. If successful, retrieves response data using helper `0x09F4` and returns a status word in `D0`.

At `0x10B8` a short 68K handler executes `MOVE.W #-1,(A2); RTS`, consistent with marking an asynchronous ADB completion word. `0x10FE` is an `ADBOp` assembly shim. Error constant `0x2712` appears on elapsed-tick timeout paths. A full end-to-end asynchronous callback ABI mapping is still pending.

## 5. Distinct patches — do not conflate

### A. Native presence-only (the build **successfully tested** on 7.6.1)

- **PPC target:** `wasDonglePresent()` at `1004901C`, code-relative `0x4901C`, original HDA offset `0x337D1C`.
- Changes **eight bytes**: `7C 08 02 A6 93 E1 FF FC` → `38 60 00 01 4E 80 00 20`.
- Assembly: `li r3,1; blr`.
- Original 68K MicroGuard ADB resource and Sound Manager code are unaltered.
- Shipped in `VoxPro_185_NATIVE_PresenceOnly_FULL.sit`, confirmed recording/playback on PM6500 with System 7.6.1.

### B. Separate `Security()` wrapper call-site test (not the golden build)

- **PPC call site:** `1004A34C`; original HDA absolute offset `0x33904C`.
- Changes **four bytes**: `4B FF EC D1` (branch/link) → `38 60 00 01` (`li r3,1`).
- This build produced further UI initialization but still faulted at PC zero on Mac OS 8.6. Its later StuffIt package is **not** the native PresenceOnly full archive.

### C. Other diagnostics

- `ADBOp` two-byte test: 68K `A07C` → `7000` (return `noErr` without hardware I/O), **not** a functional dongle or a recommended build.
- Key-level 4 experiments and SheepShaver sound-input Stages 1–7 are **separate** from the tested native presence-only release.

## 6. Methodology and cautions

Preserve `original HDA` and original executable. Verify both original opcode bytes and whole-file hashes. Analyze the PPC PEF using correct file mapping; never overwrite the original resource fork or mix up code-section offsets with HDA absolute offsets. Validate any StuffIt archive's fork CRCs and file type `APPL`. Record exact hardware, system, audio-input source and build names for each experiment.
