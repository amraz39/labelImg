# labelImg (local fork — Windows EXE build)

This repository is based on **labelImg**, a graphical image annotation tool for labeling object bounding boxes.

- Official site: https://labelimg.org/
- Original source: https://github.com/HumanSignal/labelImg/

> **Note:** The original `HumanSignal/labelImg` repository was archived by its owner on **Feb 29, 2024** and is now read-only. This fork exists to keep the tool buildable/runnable locally, and to package it as a standalone Windows `.exe`.

All credit for the original application, design, and codebase goes to the original authors and contributors (TzuTa Lin and the HumanSignal/labelImg contributors).

---

## What was changed in this fork

No application logic was modified. The changes below only concern **building a standalone Windows executable** with PyInstaller, which the original project did not ship:

1. Added `labelimg.spec` — a PyInstaller spec file configured to:
   - Include the local `libs/` package correctly (`collect_submodules('libs')`, `datas=[('libs', 'libs')]`), since PyInstaller's static analysis doesn't always detect this local package on its own.
   - Disable UPX compression (`upx=False`), which was found to corrupt the bundled Python standard library and cause `ModuleNotFoundError: No module named 'json'` at runtime.
   - Bundle the app icon (`labelImg_icon.ico`).
2. Documented that `libs/resources.py` must be **regenerated locally** from `resources.qrc` via `pyrcc5` (it's a generated file, intentionally not committed to git — same as upstream).
3. Updated `.gitignore` to exclude local/build artifacts (`.venv/`, `build/`, `dist/`, generated `resources.py`, `__pycache__/`, etc.) so only source files needed to build the app are published.

---

## Prerequisites

- Python 3.11 (Windows)
- PowerShell

---

## Setup & Build Instructions

### 1. Create and activate a virtual environment

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

### 2. Install dependencies

```powershell
pip install -r requirements/requirements.txt
pip install pyqt5 lxml pyinstaller
```

### 3. Generate the Qt resources file

`libs/resources.py` is a generated file (compiled from `resources.qrc`) and is **not** committed to the repo. Generate it locally:

```powershell
pyrcc5 -o libs\resources.py resources.qrc
```

### 4. Run from source (sanity check)

```powershell
python labelImg.py
```

Confirm the app launches correctly before building the executable.

### 5. Build the executable

Build using the provided spec file (do **not** run `pyinstaller labelImg.py` directly — that regenerates a blank spec file and discards the configuration below):

```powershell
pyinstaller labelimg.spec
```

### 6. Run the built app

```powershell
.\dist\labelimg\labelimg.exe
```

The full app folder (`dist\labelimg\`) is what you distribute — it contains `labelimg.exe` alongside its `_internal` dependencies.

---

## Rebuilding after changes

If you edit any source files, clean old build artifacts first to avoid stale caches:

```powershell
Remove-Item build, dist -Recurse -Force
pyinstaller labelimg.spec
```

---

## `labelimg.spec` reference

```python
# -*- mode: python ; coding: utf-8 -*-
import os
from PyInstaller.utils.hooks import collect_submodules

project_root = os.path.abspath('.')
hiddenimports = collect_submodules('libs')

a = Analysis(
    ['labelimg.py'],
    pathex=[project_root],
    binaries=[],
    datas=[('libs', 'libs')],
    hiddenimports=hiddenimports,
    hookspath=[],
    hooksconfig={},
    runtime_hooks=[],
    excludes=[],
    noarchive=False,
    optimize=0,
)
pyz = PYZ(a.pure)
exe = EXE(
    pyz,
    a.scripts,
    [],
    exclude_binaries=True,
    name='labelimg',
    debug=False,
    bootloader_ignore_signals=False,
    strip=False,
    upx=False,
    console=False,
    disable_windowed_traceback=False,
    argv_emulation=False,
    target_arch=None,
    codesign_identity=None,
    entitlements_file=None,
    icon=['labelImg_icon.ico'],
)
coll = COLLECT(
    exe,
    a.binaries,
    a.datas,
    strip=False,
    upx=False,
    upx_exclude=[],
    name='labelimg',
)
```

---

## License

Original project licensed under the MIT License (see `LICENSE`). This fork retains the same license.
