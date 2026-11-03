# Publishing VoxPro 1.8.5 to GitHub Releases

Both original full StuffIt archives are preserved and described in [RELEASE_NOTES.md](../RELEASE_NOTES.md).

GitHub's Git file-size limit is **100 MiB** per file. Both requested files are **107,789,420 bytes (102.8 MiB)**, so the full binary archives cannot be added as ordinary Git commits. They must be uploaded as **GitHub Release assets**.

## Verified archive manifest

```text
79fdc2c0cdc9264d86c2af470ab10b62751e89bb223367668f7b37b301c0d38f  VoxPro_185_ORIGINAL_FULL.sit
4beb020f946db6196d939cdd5aa503e5ea6b6e6c150363265264c05354650746  VoxPro_185_NATIVE_PresenceOnly_FULL.sit
```

Direct binary comparison found **12 changed bytes** between the two full StuffIt archives: eight PowerPC instruction bytes and four archive integrity bytes. No other archived byte differs.

## Finishing release publication

1. Confirm that you have appropriate redistribution permission for these commercial historical binaries.
2. Open [repository Releases](https://github.com/ppuskari/VoxPro185/releases).
3. The repository workflow `.github/workflows/prepare-release.yml` attempts to prepare **draft v1.0.0** when first added. If no draft exists, use **Draft a new release**, tag `v1.0.0` on `main`, title **VoxPro 1.8.5 — System 7.6.1 Native Preservation Baseline**, and use [RELEASE_NOTES.md](../RELEASE_NOTES.md) as the body.
4. Drag **VoxPro_185_ORIGINAL_FULL.sit** and **VoxPro_185_NATIVE_PresenceOnly_FULL.sit** into **Attach binaries by dropping them here**.
5. Confirm both filenames, sizes and hashes; then publish.

Do not attach the smaller APP_ONLY variants, Stage6/7 emulator diagnostic images, or the later `Security()` call-site test instead of the two requested archives.

## Optional single-line Windows PowerShell command

When GitHub CLI (`gh`) is already installed and authenticated, and both archives are in the current directory:

```powershell
gh release upload v1.0.0 "VoxPro_185_ORIGINAL_FULL.sit" "VoxPro_185_NATIVE_PresenceOnly_FULL.sit" --repo ppuskari/VoxPro185 --clobber
```

The draft release remains unpublished until the owner verifies and publishes it, with the two binary assets attached.
