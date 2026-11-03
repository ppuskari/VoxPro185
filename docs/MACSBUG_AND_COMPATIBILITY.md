# MacsBug 6.6.3 — the PC-zero fault and operating-system compatibility

## Known original-hardware environment

- Power Macintosh 6500/275, 128 MiB RAM.
- Mac OS 8.6, 256-color display, RCA input, 44.1 kHz sound configuration.
- Later boot: System 7.6.1 from a SCSI Zip 100 disk; VoxPro executable residing on a different drive.

## Initial Mac OS 8.6 symptoms

The **virgin** VoxPro 1.8.5 program partially painted its UI and often remained at a watch cursor. Mouse click could end in a Type 10 dialog or MacsBug “Unimplemented Instruction at `00000000`.” Later patches, including a native presence-only variant, did not establish a working 8.6 startup.

These observations are **not sufficient** to identify the failing component. The classic Type 10 class concerns F-line instruction traps, while “Unimplemented Instruction at 00000000” indicates a zero-valued execution address at the observed exception; do not assume all exceptions have the same cause without a preserved first-fault context.

## MicroGuard execution in MacsBug

Manual 68K tracing established that the embedded `Zc68` resource had loaded and was executing:

- `CountADBs` saw two entries on the test setup: ADB mouse (address 3) and keyboard (address 2).
- `GetIndADB` enumerated devices, but they did not match the protection key's required descriptor/handler fields.
- `initOldKey()` returned `0x2711` during the observed attempts, meaning “not found/matching” in context (exact official symbolic error label unverified).
- The 68K procedure returned through the Mixed Mode transition (`0xFE02`) into PPC code; the PC-zero fault happened later.
- Therefore, failure to find a MicroGuard device **did not prove** the Type 10 startup crash was in the 68K ADB enumeration path.

MacsBug trace addresses were runtime values affected by load placement. Ghidra's static PPC addresses (often `1004xxxx`) are not interchangeable with physical PM6500 runtime addresses (`17xxxxxx` observed during separate sessions).

## Stack crawl and false-lead handling

At a later PC-zero crash, MacsBug warned that the expected current stack base could not be found. The `SC` stack scan surfaced apparent return addresses that were **not validated call-stack frames**:

- Candidate `0723389C`: address in RAM, not recognized as a known heap/procedure; disassembly resembled arbitrary data.
- Candidate `000046A0`: identified inside the MacsBug system heap/module, not VoxPro application code.
- ROM location `FFC16574` was annotated `_VRemove+0074` but was not shown to be causal.
- Additional 68K candidates near `005CA7C6` were not sufficiently verified.

A stack scanner reporting an address is not proof it contains the last valid caller. No fix was applied based on those addresses.

## System 7.6.1 isolation result

On **November 3, 2026**, the operator booted a System 7.6.1 SCSI Zip100 volume and ran the **same virgin VoxPro installation** from the other drive. It initialized and accepted the password `audion` for playback; variable-speed playback and mouse/keyboard bidirectional scrubbing worked.

The prior `VoxPro_185_NATIVE_PresenceOnly_FULL.sit` was then tested under System 7.6.1 and **recorded and played back with no perceptible lag**.

**Conclusion justified by testing:** the runtime environment difference matters greatly; the original 8.6 Type 10/PC-zero failure is not inherent to the Virgin application under all classic Mac OS installations. **Not yet proven:** whether 8.6 itself, particular extensions, Sound Manager versions, the Mixed Mode Manager, a driver, or a callback causes failure.

## Practical regression sequence

1. Preserve the functioning System 7.6.1 Zip100 boot volume and both original/presence-only binaries.
2. Confirm safe playback with the virgin original, keeping historic session files backed up.
3. Use a fresh expendable audio session to record, save, quit, relaunch and replay in the presence-only build.
4. Only after the golden case passes, compare a *clean* 8.1 or 8.6 installation with identical audio/extension settings; document differences instead of stacking further security patches.
5. If a future Mac OS 8.6 investigation resumes, capture the **first** exception, actual 68K/PPC context, registers and verified caller rather than trying to patch a guessed stack return address.

The goal is a reproducible operating environment, not a claim that every Mac OS 8.x release is inherently unsupported.
