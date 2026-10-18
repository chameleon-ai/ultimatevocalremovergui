# Offline by default (Fedora/Linux)

On Linux, Ultimate Vocal Remover starts without automatically refreshing the application's online update data or model download catalogues. Opening Settings does not automatically refresh them either.

This is an **application-level offline-by-default policy**, not a network sandbox. Manual downloads and refreshes remain available, and third-party dependencies or system integrations may still initiate network connections.

## Model Download Center

Under **Settings > Model Download Center** on Linux:

- **Refresh List** explicitly retrieves the remote model download catalogue. It is not fetched automatically when UVR starts or when Settings is opened.
- After a successful refresh, the catalogue is cached locally at `gui_data/model_download_catalog.json`.
- On subsequent launches, UVR loads the cached catalogue without requesting it from the network. The model lists remain usable without another refresh, including when offline.
- Selecting **VR Arch**, **MDX-Net**, or **Demucs** uses the currently loaded catalogue and should not initiate a network request.
- Before a catalogue has been loaded or cached, the model list indicates that it is not loaded. Use **Refresh List** once while online to populate it.
- **Update Model Parameters** explicitly retrieves model parameter metadata when requested.
- Downloading a model is an explicit online action. MDX model configuration files are retrieved as part of the chosen download, rather than while merely browsing the model list.

The catalogue cache is generated runtime data and should not be committed to Git. Add `gui_data/model_download_catalog.json` to `.gitignore` if it is not already ignored. Deleting the cache requires another manual refresh to repopulate the download lists.

## Other platforms

The Linux-specific changes are not intended to change the existing startup behavior on Windows or macOS.

## Network limitations

Offline by default means UVR's known automatic application refreshes are disabled on Linux. It does **not** guarantee that every dependency, native library, or operating-system integration is network-silent.

During testing, occasional outbound TCP connection attempts were observed even after the application-level changes. Their origin has not been established, so do not describe this mode as complete network isolation. Use an OS-level firewall or sandbox to enforce that requirement.

## Validation

Check Python syntax and patch whitespace:

```bash
python -m py_compile UVR.py
git diff --check
```

To inspect connection attempts during startup and while opening Settings:

```bash
UVR_UI_SCALE=1.6 strace -f -tt -e trace=connect \
  -o /tmp/uvr-connect.log python UVR.py

grep -E 'AF_INET|AF_INET6' /tmp/uvr-connect.log
```

Test the UI separately:

1. With no cached catalogue, open **Model Download Center**. Model selection must not trigger an automatic refresh.
2. While connected, click **Refresh List** and check that the available models appear under VR Arch, MDX-Net, and Demucs.
3. Exit UVR and confirm `gui_data/model_download_catalog.json` exists.
4. Restart UVR without refreshing. Verify that the saved lists are available immediately, including without internet access.
5. Check **Update Model Parameters** and a model download separately; both are user-initiated network operations.

A `connect()` attempt alone does not prove that a TLS connection completed or that application data was sent.
