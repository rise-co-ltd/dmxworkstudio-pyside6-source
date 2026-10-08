# Qt for Python (PySide6) source for DMX Work Studio

This repository provides the complete corresponding source for the modified
**PySide6 and shiboken6** libraries that RISE CO., LTD. distributes with the
Windows version of **DMX Work Studio**, as required by the GNU Lesser General
Public License version 3 (LGPL-3.0).

Japanese summary is at the end of this file.

## What was modified

The PySide6 and shiboken6 libraries included in the Windows distribution of
DMX Work Studio were built by RISE CO., LTD. from the official
**Qt for Python 6.11.1** source, with the following modification made on
**2026-10-04**:

- The Python Limited API level used for the build was raised from 3.10
  (`Py_LIMITED_API=0x030a0000`) to 3.12 (`0x030c0000`).
  This is four one-line changes in the build configuration. No C++ or Python
  source code of PySide6 / shiboken6 was changed.

| File in `pyside-setup-everywhere-src-6.11.1` | Change |
|---|---|
| `sources/shiboken6/libshiboken/CMakeLists.txt` | `0x030a0000` → `0x030c0000` |
| `sources/shiboken6_generator/data/Shiboken6ToolsMacros.cmake` | `0x030a0000` → `0x030c0000` |
| `sources/pyside6/plugins/designer/CMakeLists.txt` | `0x030a0000` → `0x030c0000` |
| `wheel_artifacts/pyproject.toml.base` | wheel tag `cp310` → `cp312` |

The exact diff is in [`patches/pyside-setup-6.11.1_limited-api-3.12.diff`](patches/pyside-setup-6.11.1_limited-api-3.12.diff).

**Why:** with the 3.10 Limited API, every function that returns `None`
increments the reference count of `Py_None` directly. On Windows, long-running
use of DMX Work Studio can make that count overflow and break the receive path
(reported upstream as [PYSIDE-3424](https://qt-project.atlassian.net/browse/PYSIDE-3424)).
Building with the 3.12 Limited API uses immortal-object aware code and the
count no longer grows.

**The Qt libraries (`Qt6*.dll`) are unmodified official binaries from
The Qt Company.**

## Contents

| Path | What it is |
|---|---|
| `patches/pyside-setup-6.11.1_limited-api-3.12.diff` | The modification (the only change to the library build) |
| `patches/pyside-setup-6.11.1_create-wheels-runtime-only.diff` | Packaging step only: build just the `shiboken6` and `PySide6_Essentials` wheels. Does not change any built library |
| `patches/psutil-7.2.2_limited-api-3.12.diff` | For reference: the same Limited API change for psutil 7.2.2 (BSD-3-Clause), also shipped with DMX Work Studio for Windows |
| `BUILD.md` | Step-by-step build instructions |
| `LICENSES/` | LGPL-3.0 and GPL-3.0 texts |
| Releases | Copies of the original upstream source archives and the built wheels, per DMX Work Studio version |

## Original source

- Qt for Python 6.11.1:
  <https://download.qt.io/official_releases/QtForPython/pyside6/PySide6-6.11.1-src/>
  (`pyside-setup-everywhere-src-6.11.1.zip`)
- Git: <https://code.qt.io/cgit/pyside/pyside-setup.git/tag/?h=v6.11.1>

A copy of the original archive is attached to this repository's Releases so the
source stays available for as long as the corresponding DMX Work Studio version
is distributed.

## Built wheels shipped with DMX Work Studio (Windows)

| Wheel | SHA-256 |
|---|---|
| `shiboken6-6.11.1-cp312-abi3-win_amd64.whl` | `52ee5d6c189029a6f81b13547ba1b18cab124c9888c42e6ea9375f7658c752ea` |
| `pyside6_essentials-6.11.1-cp312-abi3-win_amd64.whl` | `92269a7efe3a23ca617d68c25b645eb8b10f56489aaf8058f53f0d73d7d9fbe3` |
| `psutil-7.2.2-cp312-abi3-win_amd64.whl` | `6f6c6954778c416fbb0f0175f06316b2abc0596b10151f12fb533428fce885a4` |

## Replacing the libraries

Under the LGPL you may replace the PySide6 / shiboken6 libraries in DMX Work
Studio with your own build (for example, the official PyPI wheels of the same
version, or your own build from this source). Note that the official wheels are
built with the 3.10 Limited API, so replacing them **removes the PYSIDE-3424
workaround** described above.

## License

PySide6 and shiboken6 are licensed by The Qt Company under the LGPL-3.0
(with GPL-3.0 as referenced by it). See [`LICENSES/`](LICENSES/).
The patch files in this repository are provided under the same terms.

## Contact

RISE CO., LTD. — info@rise-japan.co.jp

---

## 日本語の概要

このリポジトリは、有限会社ライズ（RISE CO., LTD.）が Windows 版 DMX Work Studio に同梱している
PySide6・shiboken6（Qt for Python）の「対応するソース」を、LGPL-3.0 に基づいて公開するものです。

- 公式の Qt for Python 6.11.1 のソースから組み直しています。
- 変更点は、組み立ての設定で Python の Limited API の水準を 3.10 から 3.12 に上げた 4 行だけです
  （2026-10-04）。PySide6 の本体のプログラムは書き換えていません。
- 目的は、Windows で長時間使った時に受信が壊れる不具合（PYSIDE-3424）を避けることです。
- Qt 本体（Qt6*.dll）は The Qt Company の公式のものをそのまま使っています。
- 公式版に差し替えることもできますが、その場合は上の不具合の回避が外れます。
