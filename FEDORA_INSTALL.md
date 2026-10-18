# Installing Ultimate Vocal Remover GUI on Fedora 44

This guide is for the Fedora-adapted [odeneriksson/ultimatevocalremovergui](https://github.com/odeneriksson/ultimatevocalremovergui) fork, on **Fedora Linux 44 (x86_64)**. It describes a **source installation using an activated Python virtual environment (`.venv`)**, not a packaged RPM or AppImage. Install Fedora system packages with `dnf`, but install **all third-party Python packages inside `.venv`**, never into the system Python environment.

The Fedora GUI changes have been used on Fedora 44 with Python 3.14, Tcl/Tk 9 and GNOME/Wayland (via XWayland). **A completely fresh installation and every audio separation model have not been independently validated.** The upstream dependency file is unpinned, so a new installation can resolve different package versions over time.

## 1. Install Fedora packages

Open a terminal and install the system dependencies:

```bash
sudo dnf install git python3 python3-pip python3-tkinter \
    tk tkdnd ffmpeg libsndfile rubberband
```

**Important:** This fork loads the **system-installed `tkdnd`** package on Linux. Its loader expects the Tcl 9 package tree under `/usr/lib64/tcl9.0`. Do not rely on the older bundled `tkdnd` binary for Fedora 44. `dos2unix` is **not** required.

If `dnf` cannot find `ffmpeg`, verify that an appropriate multimedia repository is configured for your Fedora installation. Do not continue until `ffmpeg -version` works. Some environments may provide a limited FFmpeg build; codec availability depends on the installed package.

Check the system tools:

```bash
python3 --version
ffmpeg -version
test -d /usr/lib64/tcl9.0/tkdnd2.9.5 && echo 'tkdnd package directory found'
```

The last command is only an example of the package directory in Fedora 44; the installed `tkdnd` version may differ. You can check its files with `rpm -ql tkdnd`.

## 2. Clone this fork

Choose an installation directory. For example:

```bash
mkdir -p ~/Applications
cd ~/Applications
git clone https://github.com/odeneriksson/ultimatevocalremovergui.git
cd ultimatevocalremovergui
```

Check the revision being installed:

```bash
git log -1 --oneline
git status --short
```

The repository does not have to be installed as root. **Do not run `pip` or UVR with `sudo`.** Only the Fedora system-package installation in step 1 uses `sudo`.

## 3. Create and activate the Python virtual environment

Fedora 44 normally supplies Python 3.14. **Run the commands below from the repository directory.** Create `.venv` and activate it **before installing any Python packages**:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python --version
python -m pip --version
```

Your shell prompt should now start with `(.venv)`. Confirm that `python` comes from this repository's virtual environment:

```bash
which python
python -c 'import sys; print(sys.executable); print(sys.prefix != sys.base_prefix)'
```

The printed Python executable should end in `ultimatevocalremovergui/.venv/bin/python`, and the last line should say `True`. **Do not continue if the virtual environment is not active.** Keep the same terminal and activated environment for steps 4–6.

If you open a new terminal later, activate the environment again:

```bash
cd ~/Applications/ultimatevocalremovergui
source .venv/bin/activate
```

When finished, type `deactivate` to leave the environment. If you need to recreate it later, remove only this clone's `.venv` directory and repeat this step. Do not delete the system Python installation.

## 4. Install PyTorch inside `.venv` (CPU baseline)

The following is the **CPU-only configuration used during Fedora development**:

```bash
python -m pip install \
    torch==2.10.0 torchvision==0.25.0 \
    --index-url https://download.pytorch.org/whl/cpu
```

This example intentionally does **not** configure NVIDIA CUDA, AMD ROCm or other GPU acceleration. For a GPU build, use the platform-specific instructions at [pytorch.org](https://pytorch.org/get-started/locally/) and confirm compatibility with your Python version and hardware. Do not install a CUDA build merely because a GPU is present.

## 5. Install UVR's Python dependencies inside `.venv`

```bash
python -m pip install -r requirements.txt
python -m pip check
```

These commands assume `(.venv)` is visible in your shell prompt. The current `requirements.txt` is **not a lockfile**. If dependency installation fails, record the *first* error and the Python and pip versions. Do not force incompatible versions or run `sudo pip`.

Check the core imports:

```bash
python - <<'PY'
import tkinter
import torch
import librosa
import onnxruntime
import soundfile

print('Tk:', tkinter.TkVersion)
print('PyTorch:', torch.__version__)
print('Librosa:', librosa.__version__)
print('ONNX Runtime:', onnxruntime.__version__)
print('SoundFile:', soundfile.__version__)
PY
```

A successful import test verifies these libraries, **not** the whole application or audio model compatibility.

## 6. Verify TkDND and launch UVR

Run this optional Tk drag-and-drop test from your graphical desktop session:

```bash
python - <<'PY'
from gui_data.tkinterdnd2 import TkinterDnD
root = TkinterDnD.Tk()
print('TkDND:', root.TkdndVersion)
root.destroy()
PY
```

Then start UVR **in the same activated virtual environment**:

```bash
UVR_UI_SCALE=1.6 python UVR.py
```

`UVR_UI_SCALE` controls the Fedora GUI scaling adaptation. The fork defaults to `1.6` when the variable is omitted. For a smaller or larger interface, try another value in the supported range **1.0–2.5**, for example:

```bash
UVR_UI_SCALE=1.3 python UVR.py
```

The GUI uses Tk and can run through XWayland in a GNOME Wayland session. The scale setting is independent of GNOME's desktop scaling.

## 7. Add a model and perform a real separation test

The repository's application code is not sufficient by itself to guarantee a working separation: you also need the corresponding model files.

1. Open UVR and obtain or select a supported model using the available model-management controls.
2. Select a short input audio file and an output directory.
3. Select the matching separation method and the installed model.
4. Click **Start Processing**.
5. Confirm that the expected output audio files were written and can actually be played.

**Single-model Multi Stem Ensemble** is a separate feature under development. Do not assume that it is present in the fork's default branch; check the relevant feature branch or commit before testing it.

## Troubleshooting

### `Unable to load tkdnd library`

Confirm that the system package is installed and that the Tcl 9 package files are present:

```bash
rpm -q tkdnd tk python3-tkinter
rpm -ql tkdnd | grep -E 'pkgIndex.tcl|libtcl.*tkdnd'
```

If your Fedora version uses a different Tcl/Tk layout, this fork's Linux-specific TkDND search path may need updating. Do not copy an old prebuilt `.so` into the repository as a workaround.

### Python package conflicts or missing imports

First run `source .venv/bin/activate` from the repository directory if needed. Then:

```bash
python --version
python -m pip check
python -m pip list
```

Keep the full pip error output when reporting a failed installation. Since dependencies are not pinned, results can change between installations.

### The GUI is too small, too large or clipped

Start with the documented scaling factor:

```bash
UVR_UI_SCALE=1.6 python UVR.py
```

Try a different value in the supported range if needed. Report the Fedora version, GNOME session type, display resolution, scale factor and a complete screenshot when opening a layout issue.

### No sound files are produced

Check that a model has actually been installed, the selected architecture matches that model, the input file is readable, the output path is writable, and FFmpeg supports the input format. A visible GUI alone does not confirm working audio separation.

## Recording a known-good installation

With `.venv` activated, once UVR works, save the exact revision and resolved Python dependency versions for reproducibility:

```bash
git rev-parse HEAD
python -m pip freeze > fedora44-installed-packages.txt
```

The generated `fedora44-installed-packages.txt` describes **your local environment** and is not a substitute for a maintained, tested dependency lockfile. Review it before committing it to the project.

For the Fedora-specific source changes, see [CHANGELOG.fedora.md](CHANGELOG.fedora.md).
