# androidstudio-tools

It lets AndroidIDE install the Android SDK **build-tools**, **platform-tools**,
**platforms**, **NDK** and **JDK** from a single manifest, with support for
**more versions** and for **32-bit arm** devices (`arm`), not just `aarch64`.

- **Manifest (single source of truth):**
  ```
  https://raw.githubusercontent.com/Ahmedgeebalkareem/androidstudio-tools/main/manifest.json
  ```
- **Archives** live in this repository's [Releases](../../releases).

---

## Usage

The IDE passes the manifest to its setup scripts with `-m`:

```bash
# Full first-time setup (SDK + platform-tools + cmdline-tools + build-tools)
idesetup.sh -m https://raw.githubusercontent.com/Ahmedgeebalkareem/androidstudio-tools/main/manifest.json

# Install a single component
sdk-install.sh -m <manifest-url> -t build-tools    -v 34.0.4 -y
sdk-install.sh -m <manifest-url> -t platform-tools -v 34.0.4 -y
sdk-install.sh -m <manifest-url> -t sdk            -v 35     -y
sdk-install.sh -m <manifest-url> -t ndk            -v 27.0.12077973 -y
sdk-install.sh -m <manifest-url> -t cmdline-tools  -y
sdk-install.sh -m <manifest-url> -t jdk            -v 11 -y
```

`-v` accepts any version listed below. `sdk` takes an Android **API level**
(`33`–`37`); `ndk` takes the full NDK revision; `jdk` accepts `11` (tarball) or
`17`/`21` (installed via the Termux package manager).

## Available packages

### build-tools / platform-tools

`idesetup.sh` installs build-tools and platform-tools as a pair, so every
version is published for both.

| Version | aarch64 | arm | x86_64 |
|---|---|---|---|
| 30.0.3 | ✅ | ✅ | — |
| 31.0.0 | ✅ | ✅ | — |
| 32.0.0 | ✅ | ✅ | — |
| 32.1.0 | ✅ | ✅ | — |
| 33.0.0 | ✅ | ✅ | — |
| 33.0.1 | ✅ | ✅ | — |
| 33.0.2 | ✅ | ✅ | — |
| 33.0.3 | ✅ | ✅ | — |
| 34.0.0 | ✅ | ✅ | — |
| 34.0.1 | ✅ | ✅ | — |
| 34.0.3 | ✅ | ✅ | — |
| 34.0.4 | ✅ | ✅ | ✅ |
| 35.0.2 | ✅ | ✅ | — |
| 36.0.0 | ✅ | ✅ | — |
| 37.0.0 | ✅ | ✅ | — |

### android_sdk (platform packages)

Architecture independent — the same bundle is listed for every ABI.

| API level | Android |
|---|---|
| 33 | Android 13 |
| 34 | Android 14 |
| 35 | Android 15 |
| 36 | Android 16 |
| 37 | Android 17 |

### NDK

| Revision | aarch64 | arm |
|---|---|---|
| 25.0.8775105 | ✅ | ✅ |
| 27.0.12077973 | ✅ | ✅ |
| 28.0.13004108 | ✅ | ✅ |
| 30.0.16248370 | ✅ | ✅ |

### JDK

| Version | aarch64 | arm | Install method |
|---|---|---|---|
| 11 | ✅ | ✅ | tarball from this manifest |
| 17 | ✅ | ✅ | Termux `openjdk-17` package |
| 21 | ✅ | ✅ | Termux `openjdk-21` package |

## Manifest structure

```jsonc
{
  "android_sdk": "<legacy single URL>",
  "android_sdk_versions": { "aarch64": { "_33": "…", "_37": "…" }, "arm": { … } },

  "cmdline_tools": "<single URL>",

  "build_tools": {
    "aarch64": { "_34_0_4": "<url>", "_37_0_0": "<url>" },
    "arm":     { "_34_0_4": "<url>", ... },
    "x86_64":  { "_34_0_4": "…" }
  },
  "platform_tools": { /* same keys as build_tools */ },

  "ndk": { "aarch64": { "_27_0_12077973": "<url>" }, "arm": { … } },
  "jdk": { "aarch64": { "_11": "<url>" }, "arm": { "_11": "<url>" } }
}
```

- Keys are `_`-escaped versions: `35.0.2` → `_35_0_2`, `33` → `_33`.
- Components are grouped per CPU ABI: `aarch64`, `arm`, `x86_64`.
- Legacy single-string keys (`android_sdk`, `cmdline_tools`) are kept so older
  clients keep working; newer clients prefer the `<tool>_versions` maps.

## Notes

- **aarch64** — real, current native builds.
- **arm (32-bit)** — the tools (aapt2, aidl, zipalign, adb, …) are CPU-native
  ELF binaries, and 32-bit arm packages only ever existed up to 34.0.4. The
  older 30.x–33.x entries and the newer 35.x–37.x entries are therefore
  **relabelled** from a real arm package; `aapt2` is replaced at install time by
  Termux's PIE `aapt2`, and `d8`/`apksigner` are Java, so this is transparent to
  AGP.
- **x86_64** — only the upstream 34.0.4 build exists.
- **android_sdk** — the platform trees come from Google's SDK `platform-*.zip`
  archives, repackaged into the `android-sdk/` layout AndroidIDE expects.
- **ndk** — community Android-host builds
  ([HomuHomu833/android-ndk-custom](https://github.com/HomuHomu833/android-ndk-custom)),
  installed under `sdk/ndk/<revision>`.
- **jdk** — JDK 11 is a standalone tarball (extracted to
  `$PREFIX/lib/jvm/java-11-openjdk`); 17/21 come from Termux packages.

## Credits

Based on and compatible with
[AndroidIDEOfficial/androidide-tools](https://github.com/AndroidIDEOfficial/androidide-tools)
(manifest format, `34.0.4`, `33.0.1`/`33.0.3`, JDK 11, android-sdk and
cmdline-tools assets) and
[HomuHomu833/android-ndk-custom](https://github.com/HomuHomu833/android-ndk-custom)
(NDK Android-host builds). All rights belong to their respective owners.

## License

The manifest and packaging scripts in this repository are provided as-is.
The downloaded tools (Android SDK, NDK, OpenJDK, …) remain under their
respective upstream licenses.
