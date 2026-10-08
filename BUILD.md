# Building PySide6 6.11.1 with the Python 3.12 Limited API (Windows x64)

These are the steps RISE CO., LTD. used on 2026-10-04 to build the PySide6 and
shiboken6 wheels shipped with DMX Work Studio for Windows.

- Target: Windows x64, CPython 3.14 (MSVC build), PySide6 6.11.1
- Modules built: Core, Gui, Widgets, Svg, Test
- Time: about 15 minutes once the tools are installed

## 1. Tools

| Tool | Notes |
|---|---|
| Visual Studio Build Tools 2022 (C++) | Workload `Microsoft.VisualStudio.Workload.VCTools`, components `VC.Tools.x86.x64` and `Windows11SDK.26100` |
| CMake, ninja and build helpers | In a separate build venv: `pip install cmake ninja aqtinstall py7zr setuptools==78.1.0 packaging==24.2 build==1.2.2.post1 wheel==0.46.3 mypy` |

`numpy==2.2.0` listed in pyside-setup's `requirements.txt` has no wheel for
Python 3.14; it is not needed for the build and can be removed.

## 2. Inputs

1. **Qt 6.11.1 (MSVC 2022, 64-bit)** from
   `https://download.qt.io/online/qtsdkrepository/windows_x86/desktop/qt6_6111/qt6_6111_msvc2022_64/qt.qt6.6111.win64_msvc2022_64/`:
   the `qtbase`, `qtsvg`, `qttools` and `qttranslations` `.7z` archives (verify each with its `.sha1`).
   Extract them into one folder `qt\` and write `qt\bin\qt.conf`:
   ```
   [Paths]
   Prefix=..
   ```
2. **libclang** (MSVC):
   `https://download.qt.io/development_releases/prebuilt/libclang/qt/libclang-llvmorg-20.1.0-windows-vs2022_64.7z`
3. **pyside-setup source**:
   `https://download.qt.io/official_releases/QtForPython/pyside6/PySide6-6.11.1-src/pyside-setup-everywhere-src-6.11.1.zip`
   (verify with `.sha256`; a copy is attached to this repository's Releases).

## 3. Apply the modification

From the root of the extracted `pyside-setup-everywhere-src-6.11.1`:

```
git apply <this repo>/patches/pyside-setup-6.11.1_limited-api-3.12.diff
```

After building, `build.ninja` for shiboken6 and pyside6 should contain only
`Py_LIMITED_API=0x030c0000` and no `0x030a0000`.

## 4. Build

With the MSVC environment loaded (`vcvars64.bat`):

```
set LLVM_INSTALL_DIR=<...>\libclang
set PATH=<...>\bvenv\Scripts;<...>\libclang\bin;<...>\qt\bin;%PATH%
cd <...>\pyside-setup-everywhere-src-6.11.1
python setup.py build --qtpaths=<...>\qt\bin\qtpaths6.exe --parallel=10 ^
  --module-subset=Core,Gui,Widgets,Svg,Test --ignore-git --standalone --limited-api=yes --skip-docs
```

## 5. Create the wheels

Optionally apply `patches/pyside-setup-6.11.1_create-wheels-runtime-only.diff`
so that only the `shiboken6` and `PySide6_Essentials` wheels are created, then:

```
python create_wheels.py --no-examples
```

Notes:
- If a path inside `package_for_wheels\build\...` reaches 260 characters, map the
  source folder to a short drive letter first (`subst Q: <source folder>`), run
  `create_wheels.py` from `Q:\`, then `subst Q: /D`.
- After a failed run, move away `package_for_wheels\*.egg-info` and
  `package_for_wheels\build\` before retrying; otherwise leftovers can end up in
  the next wheel.

## 6. Check

```
python -c "from PySide6.QtCore import QLibraryInfo; print(QLibraryInfo.build())"
```
The output should end with `[Python limited API 3.14.x]` (the official wheels show
`run time: 3.14.x built: 3.10.0`).

## psutil (reference)

psutil 7.2.2 for Windows is built with the same Limited API change
(`patches/psutil-7.2.2_limited-api-3.12.diff`) from the PyPI sdist:
load the MSVC environment, set `PYTHONUTF8=1`, `DISTUTILS_USE_SDK=1`, `MSSdk=1`, then
`pip wheel --no-build-isolation --no-deps .`
