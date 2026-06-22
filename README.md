<div align="center">

# macOS Background Removal for OBS - Clean / Protected Region

**AI background removal for OBS on macOS that doesn't eat your microphone.**

A fork of [gxalpha/obs-mac-backgroundremoval](https://github.com/gxalpha/obs-mac-backgroundremoval) that adds a **Protected Region** so static objects in front of you (like a desk or boom mic) stop getting cut out along with the background.

[Download](https://github.com/andrewleejenkins/obs-mac-clean-backgroundremoval/releases/latest) ·
[Installation](#installation) ·
[How to use](#how-to-use) ·
[Why this exists](#why-this-fork-exists) ·
[Build from source](#building-from-source)

</div>

---

## Overview

This plugin adds an OBS **effect filter** that removes a person's background on macOS using Apple's built-in [Vision](https://developer.apple.com/documentation/vision) person-segmentation API. No green screen, no cloud service, no NVIDIA GPU - it runs entirely on-device using the same engine macOS uses for Portrait mode.

It works on Apple Silicon and Intel Macs, on any OBS source (not just webcams), and requires **OBS 31.1 or later** on **macOS 12 (Monterey) or later**.

### What this fork adds

Apple's Vision API segments **people only**. Anything in front of you that it doesn't recognize as part of a person - most commonly a microphone - drops below the segmentation threshold and gets erased right along with your background. No amount of tweaking the Threshold or Quality settings brings it back, because the model fundamentally does not consider the object to be a person.

This fork fixes that with a **Protected Region**: a user-positioned rectangle that is always forced to stay in the foreground, no matter what the segmentation model decides. Park it over your mic and the mic stays put. The box has feathered edges and is composited at the same alpha stage as the segmentation mask, so the result still looks clean instead of like a hard pasted rectangle.

---

## Installation

> Releases are **code-signed and notarized** with an Apple Developer ID, so they install with no Gatekeeper warning. (Older builds were unsigned and required an "Open Anyway" step - that is no longer needed.)

1. **Quit OBS completely** (Cmd-Q, not just closing the window).
2. **If you already have a previous version installed, uninstall it first** - see [Upgrading](#upgrading-from-a-previous-version) below. This step matters.
3. **Download** the latest `obs-mac-backgroundremoval-x.y.z-macos-universal.pkg` from the [Releases page](https://github.com/andrewleejenkins/obs-mac-clean-backgroundremoval/releases/latest).
4. **Open the `.pkg`** and run through the installer.
5. **Open OBS.** Add the filter to a source (see [How to use](#how-to-use)).

The plugin installs to `~/Library/Application Support/obs-studio/plugins/`.

### Upgrading from a previous version

This is important. If you already have the original `obs-mac-backgroundremoval` (or any earlier build) installed, **uninstall it before installing the new one.** The macOS installer will sometimes report "Installation successful" but **silently skip overwriting an existing older bundle** - so you end up still running the old version with none of the new controls. The tell-tale symptom: you install, reopen OBS, and the filter looks exactly the same (no "Keep a protected region" option).

To do a clean upgrade, **quit OBS first**, then run:

```sh
rm -rf "$HOME/Library/Application Support/obs-studio/plugins/obs-mac-backgroundremoval.plugin"
```

Then install the new `.pkg` and reopen OBS. Your existing scene filters keep working and gain the Protected Region option - filter settings are stored in your OBS scene collection, not in the plugin, so nothing is lost.

To confirm which version is actually loaded, you can check:

```sh
grep -i Protect "$HOME/Library/Application Support/obs-studio/plugins/obs-mac-backgroundremoval.plugin/Contents/Resources/locale/en-US.ini"
```

If that prints `Protect.*` lines, you are on this fork. If it prints nothing, the old version is still installed.

### Uninstalling

Quit OBS, then delete `~/Library/Application Support/obs-studio/plugins/obs-mac-backgroundremoval.plugin` and reopen OBS.

---

## How to use

1. In OBS, right-click your camera (or any) source → **Filters**.
2. Under **Effect Filters**, click **+** and add **macOS Background Removal**.
3. Your background disappears immediately. Now tune it:

### The controls

| Control | What it does |
| --- | --- |
| **Threshold** | How confident Vision must be that a pixel is "person" before keeping it. Higher = tighter cutout, lower = softer/looser edge. Default `0.9`. |
| **Quality** | `Fast` / `Balanced` / `Accurate`. Higher quality is cleaner but uses more CPU/GPU. `Accurate` is recommended on Apple Silicon. |
| **Keep a protected region (e.g. microphone)** | Master on/off for the protected box. **On by default.** |
| **Crop left / right / top / bottom edge in** | Each slider pulls that edge of the box inward from the frame border. `0` = the edge sits at the frame border; higher = pulled further in. Drag all four to box in your mic. |
| **Protected region: edge feather** | Softens the box edge so it blends into the cutout instead of showing a hard rectangle. |
| **Show region outline** | Draws a bright magenta outline around the box in the preview so you can position it. **Turn this off before going live** - the outline renders on your output too. |

### Dialing in the mic

The defaults box in the **bottom-center**, where a desk or boom mic usually sits. To fit it to your setup:

1. Turn on **Show region outline** so you can see the box.
2. Drag **Crop left / right / top / bottom** until the magenta box **just** covers your mic and nothing more. (Bottom defaults to `0` so the box reaches the bottom of the frame, where mics come up from.)
3. Add a small amount of **feather** (try `0.02` to `0.05`) to soften the seam.
4. **Turn Show region outline back off** before you stream or record.

> **Keep the box as small as possible.** Anything that enters it - including your hand if it drops low - will also show the real background inside the box. A snug box around just the mic is invisible in practice.

If you ever want the original, vanilla behavior with no protected region, just toggle **Keep a protected region** off.

---

## Why this fork exists

I use this plugin for streaming and recording, and it is genuinely excellent - except my black microphone kept getting deleted because Apple's segmentation model only knows how to find people. Rather than switch to a heavier ONNX-based matting plugin, I added a tiny, surgical fix: force one fixed rectangle to always stay visible. It is the right tool for a static object in a fixed position, and it keeps the plugin fast and native.

Full credit for the original plugin goes to **Sebastian Beckmann** ([gxalpha](https://github.com/gxalpha/obs-mac-backgroundremoval)). This fork only adds the Protected Region on top of his work.

---

## Building from source

The build system is the standard [OBS Plugin Template](https://github.com/obsproject/obs-plugintemplate); its instructions apply here.

**Requirements:** macOS with Xcode (full install, not just Command Line Tools) and CMake 3.28+.

```sh
git clone https://github.com/andrewleejenkins/obs-mac-clean-backgroundremoval.git
cd obs-mac-clean-backgroundremoval

# Build a universal (arm64 + x86_64) plugin
.github/scripts/build-macos

# Produce an installer .pkg
.github/scripts/package-macos
```

The repository's **GitHub Actions** workflows also build and package the plugin for macOS, Windows, and Linux automatically on every push. **Pushing a version tag** like `0.3.1` builds the universal package and publishes a GitHub Release:

```sh
git tag 0.3.1
git push origin 0.3.1
```

### Project layout

| Path | Purpose |
| --- | --- |
| `src/plugin-main.m` | The filter: Vision request, mask handling, properties, and the Protected Region logic. |
| `data/alpha_mask.effect` | The shader that composites the segmentation mask and the protected box. |
| `data/locale/en-US.ini` | UI strings. |
| `buildspec.json` | Plugin metadata and dependency pins. |

---

## Troubleshooting

- **"The filter isn't in the list."** Make sure you're on OBS 31.1+ and that you quit and reopened OBS after installing. Look under **Effect Filters**, not Audio filters.
- **"Edges flicker or lag a little."** Segmentation runs on a background thread to keep OBS smooth, so the mask trails the video by a frame or two. Raise **Quality** for a cleaner edge.
- **"My mic is still cut off."** Turn on **Keep a protected region** and drag the box over the mic. If the mic moves around, make the box a little larger or reposition it.
- **"Something else inside the box shows the background."** That's expected - the box always shows the real image. Make the box smaller so it covers only the static object.

---

## Credits and License

This plugin is licensed under the **GNU General Public License, Version 2**. See the [`LICENSE`](LICENSE) file for the full text.

- Original plugin by **Sebastian Beckmann** - <https://github.com/gxalpha/obs-mac-backgroundremoval>
- Early implementation referenced **pkv** from OBS, who built a similar NVIDIA-GPU filter.
- Build system based on the [OBS Plugin Template](https://github.com/obsproject/obs-plugintemplate).
- Protected Region fork by **Andrew Lee Jenkins**.

---

## About the author

Built and maintained by **Andrew Lee Jenkins**.

### Projects

- **Warpsite** - fast, modern websites for service businesses · <https://warpsite.dev>
- **Seedly CRM** - the CRM built for growing local businesses · <https://seedlycrm.com>
- **Personal site** - <https://andrewleejenkins.com>

### Connect

<!-- SOCIALS:START -->
- **X / Twitter** - <https://x.com/_cultvibez>
- **LinkedIn** - <https://www.linkedin.com/in/andrew-lee-jenkins/>
- **Facebook** - <https://www.facebook.com/AndrewLeeJenkins>
<!-- SOCIALS:END -->

If this plugin saved your microphone, a ⭐ on the repo is appreciated.
