# VoxPro 1.8.5 — Classic Macintosh preservation and reverse engineering

Engineering record of restoring **Audion Laboratories VoxPro 1.8.5**, a classic Macintosh broadcast audio editor, to usable operation on original Power Macintosh hardware.

## Confirmed working baseline (November 3, 2026)

| Component | Result |
|---|---|
| Hardware | Power Macintosh 6500/275, built-in RCA audio input |
| OS | **System 7.6.1**, booted from a SCSI Zip 100 disk |
| Virgin VoxPro 1.8.5 | Launches, renders the full interface, accepts playback password `audion`, plays at varying speeds, and scrubs forward/reverse via mouse and keys |
| `VoxPro_185_NATIVE_PresenceOnly_FULL.sit` | **Recording and playback confirmed working on the physical PM6500**, with no perceptible lag reported |
| Mac OS 8.6 | Virgin and multiple patched builds drew incomplete UI and failed with Type 10 / MacsBug “Unimplemented Instruction” at PC `00000000` |
| Mac OS 8.1 | Proposed comparison; **not yet tested or established** |

**Most important finding:** startup compatibility and MicroGuard recording authorization were **distinct issues**. A working 7.6.1 baseline resolved the original startup failure; the previously generated native *presence-only* build then enabled functional recording/playback. The exact 8.6 incompatibility root cause is not proven.

## Repository guide

- [Engineering timeline and test matrix](docs/ENGINEERING_HISTORY.md) — original media, SheepShaver test stages, real-Mac MacsBug traces, System 7.6.1 breakthrough, what is proven versus hypothesized.
- [Ghidra and MicroGuard analysis](docs/GHIDRA_MICROGUARD.md) — PowerPC PEF, CodeWarrior symbols, `Zc68` 68K function table, ADB traps and timing protocol, byte-level patches.
- [MacsBug and compatibility investigation](docs/MACSBUG_AND_COMPATIBILITY.md) — original crash signatures, stack-crawl cautions, OS testing and next debugging targets.
- [Archive descriptions and validation](RELEASE_NOTES.md) — the two archival StuffIt files, sizes, SHA-256 checksums, installation notes and tested behavior.

## Archival binaries

These are the intended full installation archives:

| File | Purpose | SHA-256 |
|---|---|---|
| `VoxPro_185_ORIGINAL_FULL.sit` | Unmodified original VoxPro installation preserved in a classic StuffIt archive | `79fdc2c0cdc9264d86c2af470ab10b62751e89bb223367668f7b37b301c0d38f` |
| `VoxPro_185_NATIVE_PresenceOnly_FULL.sit` | Original installation with narrowly scoped native PPC MicroGuard presence diagnostic | `4beb020f946db6196d939cdd5aa503e5ea6b6e6c150363265264c05354650746` |

Each archive is **107,789,420 bytes** (about **102.8 MiB**), larger than GitHub's 100 MiB per-file repository limit. These should be published as **GitHub Release assets**, rather than normal Git blobs. Documentation in the repository does **not** by itself mean release attachments are present; check the Releases page.

**Preserve and test original copies.** Do not assume a presence-only patch emulates a real MicroGuard dongle or establishes the legitimacy of any license. This repository documents the owner's preservation and interoperability experiments. Original VoxPro and MicroGuard remain the rights of their respective owners; redistribution permissions must be respected.

## Recovery and research details

- Original media: `HD10_512  VoxPro folder.hda`, an HFS Macintosh disk image.
- Main application: `VoxPro 185`, PowerPC CFM/PEF executable with both data and resource forks.
- Tools: Ghidra 12.2, MacsBug 6.6.3, SheepShaver, HFS/PEF/StuffIt analysis scripts.
- CPU architecture: native PowerPC with a dynamically loaded 68K `Zc68` MicroGuard resource and Mixed Mode calls.
- Audio recovery: archived project material was accessible for playback; full project-format documentation remains an independent preservation task.

## Current status

**Operational success, not a complete emulation.** Original-hardware recording/playback with the specific *PresenceOnly* build under System 7.6.1 is established by user testing. Long-duration recording, file reopening, and a complete feature-by-feature editing regression remain future validation items.

Do not replace the working baseline with the later, independently developed `Security()` call-site patch or the SheepShaver sound-input diagnostic builds without a separate experiment.
