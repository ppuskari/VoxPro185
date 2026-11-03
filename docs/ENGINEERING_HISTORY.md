# Engineering history — VoxPro 1.8.5 restoration

Historical reconstruction from original binaries, analysis scripts, emulator tests, MacsBug/Ghidra sessions, and operator reports. Distinguish **observed**, **derived**, and **hypothesized** behavior.

## 1. Archival starting point

- Source image: `HD10_512  VoxPro folder.hda`, approximately 120 MiB HFS disk with the original installation.
- Main application `VoxPro 185` includes a native PowerPC PEF **data fork** and classic Macintosh **resource fork**.
- The source included historic audio/projects; some `PVXE`/`PVXM` material was extracted and replayed/converted for preservation.
- VoxPro 1.8.5 is historic commercial broadcaster software using MicroGuard ADB protection. The apparent lack of a readily archived newer “VoxPro 2.0” was an initial preservation motivation, not proof that such a version did not exist.
- Early tests showed an unavailable ADB dongle could cause **“295. Dongle memory error”** during Record/Insert, independently of audio configuration.

## 2. SheepShaver emulator: audio-input separation

SheepShaver's available sound setup did not match a real Power Macintosh recording device.

- Original/virgin program encountered sound-input initialization failures, including an error `-231`.
- **Stages 1–4:** explored and selectively neutralized failing Sound Input Manager setup paths to investigate whether the UI could initialize; these builds are diagnostic, not recommended native recordings.
- **Stage 3:** individually masked error reporting from `SetNumberChannels`, `SetInputSource`, `SetPlayThrough`, `SetSampleRateSM`, and `SetSampleSizeSM`; these were not hardware implementations.
- **Stage 5–6:** explored `dbin` / input-buffer initialization behavior with synthetic-buffer and status-handling experiments. Stage 6 initialized the interface, survived calibration, opened historical material and played audio while scrubbing.
- **Stage 7A:** combined Stage 6 audio work with the presence-only MicroGuard intervention. Record crossed its licensing gate but stalled in the emulator's nonfunctional recording-input path. This separated security-command gating from real capture hardware.
- **Stages 7B/7C:** investigated alternate key-level and combined interventions, not established as the preferred hardware build.

**Interpretation:** successful UI/playback in SheepShaver did not demonstrate working hardware capture. Recording success required real hardware and a compatible OS.

## 3. Native Power Macintosh + MacsBug

Hardware: Power Macintosh **6500/275**, 128 MiB RAM, built-in RCA sound input selected, 44.1 kHz audio environment, 256-color display.

Under **Mac OS 8.6**:

- Virgin program partially rendered windows, sometimes with a watch cursor; user interaction could produce Type 10 or “Unimplemented Instruction at `00000000`”.
- The native presence-only test variant also failed similarly, disproving any easy assumption that presence-test patch alone fixed native startup.
- MacsBug 6.6.3 stepping showed 68K MicroGuard code successfully enumerating ADB mouse and keyboard via `CountADBs` / `GetIndADB`, rejecting non-matching device descriptors, often returning `$2711` from key initialization.
- A valid 68K Mixed Mode return was observed before the later PC-zero crash; thus the dongle enumeration failure was **not proven** to be the exception source.
- A MacsBug stack scan contained false or unverified candidate return addresses. One address `0723389C` did not resolve to a known heap/module, and `000046A0` belonged to MacsBug. Neither was grounds for a binary patch.
- A later independently generated `Security()` call-site patch at Ghidra `1004A34C` rendered more recording-standby UI but still faulted at PC zero. This was a separate experiment from the earlier `PresenceOnly_FULL` build.

## 4. Ghidra static analysis

Ghidra 12.2 imported the native PPC PEF and raw 4,366-byte 68K `Zc68` resource. CodeWarrior symbols identified the security wrapper functions and prevented erroneous offset mapping. The 68K initializer at `0x0000` populated a **17-entry** callback/function table. The flow from PPC `InitMicroGuardPowerPC()` to 68K routines uses `GetResource('Zc68',0)`, `MoveHHi`, `HLock`, and `CallUniversalProc`.

The raw 68000 Ghidra language initially rendered Macintosh A-line Toolbox traps (e.g. `A975`, `A07C`, `A078`) as unknown instructions. Disassembly could be restarted after the two-byte trap; NOP substitutions were discussed **only as analysis-copy visual aids**, not valid runtime replacements of OS call semantics. See [GHIDRA_MICROGUARD.md](GHIDRA_MICROGUARD.md).

## 5. The successful OS compatibility experiment

**November 3, 2026:** booting from a **SCSI Zip 100 disk with System 7.6.1**, while launching the **virgin** VoxPro application from a different drive, produced an operational UI. The password `audion` unlocked playback; variable-speed playback and bidirectional scrubbing worked with mouse and keyboard.

Testing the **previously built** `VoxPro_185_NATIVE_PresenceOnly_FULL.sit` under this OS then gave **working recording and playback without noticeable lag**.

This is the present **golden hardware/software baseline**. The observation strongly implicates the Mac OS 8.6 runtime environment in the old startup failure but does not isolate the specific changed system component, trap, driver, or callback.

## 6. What remains open

- Exact OS compatibility boundary: 8.1/8.5/8.6 are not a completed controlled test series.
- Real cause of the PC-zero fault under 8.6: null pointer / callback / Mixed Mode / other library effects were hypotheses, not proven diagnoses.
- Original authorizing dongle protocol: ADB enumeration and memory transactions identified, but no complete faithful device emulation.
- Persistence and durability: long recordings, saving/reopening, session editing, export, edge-case audio input configurations.

For an archival release, preserve hashes and separate *virgin*, *native presence-only*, *SheepShaver audio-stage*, and *independent Security-call-site* builds rather than labeling them interchangeably.
