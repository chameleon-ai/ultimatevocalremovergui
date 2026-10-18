# Fedora / Linux adaptation changelog

This changelog documents the Fedora/Linux-specific modifications in the [odeneriksson Ultimate Vocal Remover GUI fork](https://github.com/odeneriksson/ultimatevocalremovergui). It is **not** the upstream UVR release changelog. Changes in this fork have been vibe-coded with assistance from ChatGPT and should be independently reviewed and tested.

## 2026-10-18 — Offline-by-default Model Download Center

### Startup and network behavior
- Disable the known automatic application update / online-data refresh calls at startup on Linux.
- Do not automatically refresh online data merely because Settings is opened on Linux.
- Preserve explicit online actions for refreshing model catalogues, updating model parameters, and downloading models.
- Leave the intended startup behavior on Windows and macOS unchanged.

### Model Download Center
- Rename **Application Download Center** to **Model Download Center** on Linux.
- Make selection of **VR Arch**, **MDX-Net**, and **Demucs** independent of an online refresh; selecting a model family should not itself contact the network.
- Show a meaningful *Model list not loaded* state before any catalogue is available.
- Make **Refresh List** explicitly retrieve and populate the model catalogue, with more accurate success/failure feedback. When no family is selected, use VR Arch as the initial selection after refreshing.
- Prevent browsing MDX-Net models from automatically fetching model configuration files. Fetch required configuration files as part of an explicitly requested download instead.
- Avoid an automatic application-data refresh after a model download on Linux.

### Persistent offline catalogue
- Save the successfully refreshed model catalogue to `gui_data/model_download_catalog.json`.
- Load that cache on subsequent launches without fetching the catalogue again, so the available model lists survive program restarts and can be browsed offline.
- Treat the JSON file as generated local cache data; it should be excluded from Git using `.gitignore`.
- A first-time installation with no cached catalogue still requires one manual **Refresh List** while online to populate download choices.

### Observed verification and limitations
- The user confirmed that the model catalogue is available after a manual refresh and remains available after restarting UVR with the cache fix applied.
- Earlier testing identified occasional external TCP connection attempts during startup even after the application-level offline changes. Their source has not been conclusively identified. **Offline by default is not a guarantee of network isolation.**
- Full end-to-end validation of every model download, every audio separation method, and a fresh Fedora installation has not been established by these tests.

See [OFFLINE_MODE.md](OFFLINE_MODE.md) for the operational behavior, testing procedure, and network limitations.

## 2026-10-18 — Initial Fedora 44 GUI adaptation

### Compatibility
- On Linux, prefer a system-installed `tkdnd` package rather than the bundled platform binary. Retain the platform-specific bundled loading paths for Windows and macOS.
- Remove the hard Tk 8.6 requirement in the dark theme to allow newer Tk versions.

### GUI scaling and layout
- Add the configurable `UVR_UI_SCALE` factor (default `1.6`, clamped to `1.0`–`2.5`) for the legacy Tk `place`-based interface and its window and section dimensions.
- Rework input/output path controls and option placements for the scaled interface.
- Adjust Fedora checkbutton styling and placement to prevent clipped labels.
- Reposition the Start Processing, Settings and Stop controls, console, and progress bar using a `<Configure>`-driven footer layout.
- Preserve existing `<Configure>` event handlers with `add="+"`.

### Help and status output
- Display hover help inside the main window, avoid duplicate coordinate scaling, and delay hover hints by 500 ms.
- Suppress the upstream application-update notification on Linux. **The initial adaptation did not yet disable its automatic refresh; that behavior was changed by the later offline-by-default work above.**

### Original patch scope
- The initial GUI patch excluded changes under `models/**/model_data/*.json` and did not include Python dependency or installation-file changes.
- Its original verification covered patch structure and Python syntax, not a complete independent Fedora GUI and audio-processing test.

## Installation and development notes

- Follow [FEDORA_INSTALL.md](FEDORA_INSTALL.md) for Fedora setup, including installation of third-party Python dependencies **inside an activated `.venv`**; Fedora system packages are installed separately with `dnf`.
- This fork's changes have been produced with ChatGPT-assisted vibe coding. The original UVR project and its contributors retain credit for upstream work. Review changes and test them before relying on them.
