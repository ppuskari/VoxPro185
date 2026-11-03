# VoxPro 1.8.5 — archival files and release description

**Release objective:** preserve the original 1990s Macintosh installation and a minimal, native-PowerPC **presence-only** research build that has been demonstrated recording and playing back on actual Power Macintosh 6500 hardware under System 7.6.1.

## Downloads / file identities

### `VoxPro_185_ORIGINAL_FULL.sit`

- **Size:** 107,789,420 bytes (about 102.8 MiB).
- **SHA-256:** `79fdc2c0cdc9264d86c2af470ab10b62751e89bb223367668f7b37b301c0d38f`
- Original/virgin VoxPro 1.8.5 installation in classic StuffIt format, including the Mac application data and resource forks and installation data, derived from the preserved HFS disk image.
- **Hardware result:** on a PM6500/275 booted with System 7.6.1, original program launched, unlocked playback with the operator password, played at different speeds and scrubbed in either direction with keyboard or mouse. Original binary recording authorization was not established as working without hardware dongle.

### `VoxPro_185_NATIVE_PresenceOnly_FULL.sit`

- **Size:** 107,789,420 bytes (about 102.8 MiB).
- **SHA-256:** `4beb020f946db6196d939cdd5aa503e5ea6b6e6c150363265264c05354650746`
- Retains the original installation but modifies one PPC function entry for the native `MicroGuardSecurity::wasDonglePresent()` presence diagnostic.
- **Original eight PowerPC bytes:** `7C 08 02 A6 93 E1 FF FC`
- **Replacement bytes:** `38 60 00 01 4E 80 00 20` (`li r3,1; blr`).
- **Patch location:** original HDA offset `0x337D1C`; original PowerPC code offset `0x4901C`; runtime Ghidra address `0x1004901C`.
- The 68K `Zc68` resource and native sound-input routines remain unmodified.
- **Confirmed physical-hardware result:** PM6500/275 + System 7.6.1: **recording and playback work, with no noticeable lag** as reported by the owner on November 3, 2026. This does not assert full feature coverage, long-run reliability, or faithful USB/ADB dongle emulation.

## Exact binary comparison

The two full StuffIt archives have identical lengths. Byte-level comparison found **12 differing bytes total**: the 8-byte PPC presence-only patch and 4 bytes of StuffIt integrity metadata. The second archive is *not* the later `Security()` call-site patch (`1004A34C`) and is *not* one of the SheepShaver sound-input diagnostic builds.

## Install and test

1. Keep both `.sit` files and original media backed up. Verify SHA-256 before using them.
2. Expand a chosen StuffIt archive on a classic Macintosh with a suitable StuffIt Expander version, preserving resource forks and Finder file types/creators. Do not flatten files through a transfer path that loses the resource fork.
3. For a known-good baseline, boot the Power Macintosh 6500 with **System 7.6.1** (the owner used a SCSI Zip 100 disk), then launch the original program.
4. Use a separate expendable copy for the presence-only research build. Test record, save, quit, relaunch, and playback using disposable audio data.
5. Mac OS 8.6 produced incomplete UI and Type 10/PC-zero exceptions on the test system; do not assume it is a supported working environment.

## Known limitations

- Long-duration recording, edit operations, project reopening, automatic calibration across devices, and later OS compatibility have not all been regression-tested.
- The presence-only patch changes one application check; it **does not reproduce MicroGuard memory responses** and does not alter the 68K ADB handler.
- The exact Mac OS 8.6 compatibility bug remains unidentified.
- Original software and third-party licensing rights remain with their holders. Confirm rights before redistributing commercial archives publicly.

## Release publication note

GitHub blocks regular repository files greater than **100 MiB**. These `.sit` files are **102.8 MiB each** and belong as **GitHub Releases binary assets**, not tracked normal blobs. This Markdown is the release description and manifest; it is not evidence that the GitHub Release has been created or the assets attached.
