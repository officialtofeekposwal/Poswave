# Desktop releases and the update check

How a desktop build gets its version, how its files are named on the releases page, and how the
app finds out a newer one exists.

## 1. One version, stamped from one place

- The version is a plain string such as `1.0.0`. No leading `v`.
- `desktopApp/build.gradle.kts` reads it from `-PPoswave.version=...` and falls back to a default (`appVersion`).
- The app reads the same value at runtime from the `poswave.version` system property.

## 2. File names on the releases page

The release workflow renames build output to these patterns, where `<v>` is the version:

| Platform | File |
|---|---|
| Windows | `Poswave-<v>-windows-x64-setup.exe` |
| Windows | `Poswave-<v>-windows-x64.msi` |
| Windows | `Poswave-<v>-windows-x64-portable.zip` |
| Linux | `Poswave-<v>-linux-x86_64.AppImage` |
| Linux | `Poswave-<v>-linux-amd64.deb` |
| Linux | `Poswave-<v>-linux-x86_64.rpm` |
| macOS | `Poswave-<v>-macos-arm64.dmg` |
| macOS | `Poswave-<v>-macos-x64.dmg` |

macOS builds are unsigned. Clear the quarantine flag with `xattr -cr /Applications/Poswave.app`.
