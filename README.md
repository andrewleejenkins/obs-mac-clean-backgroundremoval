# macOS Background Removal for OBS - Clean / Protected Region fork

## Introduction

This is an OBS filter that removes a person's background on macOS using Apple's
built-in [Vision](https://developer.apple.com/documentation/vision) person
segmentation API. It is a fork of Sebastian Beckmann's excellent
[obs-mac-backgroundremoval](https://github.com/gxalpha/obs-mac-backgroundremoval).

**What this fork adds: a "Protected Region".**

Apple's Vision API segments *people only*. Anything in front of you that is not
recognized as part of a person, such as a desk or boom microphone, falls below
the segmentation threshold and gets cut out along with the background. No
threshold or quality setting can bring it back, because the model simply does
not consider the object to be a person.

This fork adds a user-defined rectangle that is always forced to stay in the
foreground, regardless of what the segmentation model decides. Park it over your
microphone (or any other static object in front of you) and it stops
disappearing. The box has feathered edges so it blends cleanly, and it is
composited at the same alpha stage as the mask so the result stays smooth.

This plugin requires OBS 31.1 or later.

## The new controls

Open the filter's properties and you'll find, below the existing Threshold and
Quality controls:

| Control | What it does |
| --- | --- |
| **Keep a protected region (e.g. microphone)** | Master on/off for the feature. On by default. |
| **Protected region: left / top** | Top-left corner of the box, as a fraction of the frame (0 = left/top edge, 1 = right/bottom edge). |
| **Protected region: width / height** | Size of the box as a fraction of the frame. |
| **Protected region: edge feather** | Softens the box edge so it blends into the cutout instead of showing a hard rectangle. |

The defaults place the box bottom-center, where a desk or boom mic usually sits.
Watch the OBS preview and nudge **left/top/width/height** until the box just
covers your mic, then add a touch of **feather** to soften the seam. Keep the box
as small as possible: anything else that enters it (for example your hand) will
also show the real background, so you want it covering only the mic.

> Tip: if you want zero protected region (vanilla behaviour), just turn the
> toggle off.

## Getting started (install)

Download and run the installer from the
[releases](https://github.com/andrewleejenkins/obs-mac-clean-backgroundremoval/releases)
page. It is not signed or notarized, so follow these steps:

- Open the `.pkg` file you downloaded. You'll get a warning; select "Done" (do
  **not** move it to trash).
- Open System Settings → "Privacy & Security" → "Security". Near the bottom
  you'll see that the `.pkg` was blocked.
- Click "Open Anyway" and confirm with your administrator login, then run the
  installer again.

This fork keeps the same plugin name and identifier as the original, so it
installs as a **drop-in replacement**. If you already have the original
`obs-mac-backgroundremoval` installed, this will overwrite it and your existing
scene filters will keep working (and gain the protected-region option). If you
prefer, remove the old plugin first from
`~/Library/Application Support/obs-studio/plugins/`.

The filter is found in the filters window under "Effect Filters". It can be used
on any type of source, not just Video Capture Devices.

## Building from source

The build system is the standard
[OBS Plugin Template](https://github.com/obsproject/obs-plugintemplate); its
build instructions apply here. In short, on macOS:

```sh
# Requires Xcode + CMake
.github/scripts/build-macos
.github/scripts/package-macos
```

The GitHub Actions workflows in `.github/workflows/` also build and package the
plugin for macOS, Windows, and Linux automatically on every push and tag. To cut
a release, push a tag (the project follows the OBS template's release flow).

## License and Thanks

This plugin is licensed under the terms of the GNU General Public License,
Version 2. You can find the full text in the `LICENSE` file.

All credit for the original plugin goes to **Sebastian Beckmann**
([gxalpha](https://github.com/gxalpha/obs-mac-backgroundremoval)). This fork only
adds the protected-region feature on top of his work.

Huge credits also go to pkv from OBS, who implemented a similar filter in OBS for
NVIDIA GPUs; his code was referenced heavily in the original plugin's early
stages.

The build system is based on the
[OBS Plugin Template](https://github.com/obsproject/obs-plugintemplate).
