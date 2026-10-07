<p align="center">
  <img src="docs/images/logo.svg" width="72" height="72" alt="FaceForge BDO mark">
</p>

<h1 align="center">FaceForge BDO</h1>

<p align="center"><strong>Offline Black Desert photo-to-preset tool.</strong></p>

<p align="center">
  Windows-first standalone EXE. Measured version-20 byte layout, user-supplied slider calibration, single-screen workflow.<br>
  No game injection, process memory access, macros, or cloud face upload.
</p>

<p align="center">
  <a href="https://github.com/SenjuWoo/FaceForge-BDO/actions/workflows/ci.yml"><img src="https://github.com/SenjuWoo/FaceForge-BDO/actions/workflows/ci.yml/badge.svg" alt="CI"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-e4a64f?labelColor=0d1319" alt="MIT License"></a>
  <a href="https://github.com/SenjuWoo/FaceForge-BDO/releases/tag/v0.7.1"><img src="https://img.shields.io/badge/release-v0.7.1-69bdd1?labelColor=0d1319" alt="v0.7.1"></a>
</p>

<p align="center">
  <a href="https://github.com/SenjuWoo/FaceForge-BDO/releases/latest">Download</a>
  ·
  <a href="#quick-start">Quick start</a>
  ·
  <a href="#honest-limits">Honest limits</a>
  ·
  <a href="CHANGELOG.md">Changelog</a>
</p>

## What it does

FaceForge BDO reads Black Desert customization format **version 20** files (fixed **924-byte** records), decrypts the Thin-ICE ciphertext, edits plaintext bytes that have been measured on a real corpus, re-encrypts, and validates by re-parsing before export or save.

Photo matching drives only **calibrated** sliders from local face measurements. Everything else (class, face type, hair, makeup, colours, body, uncalibrated sliders) is copied byte-for-byte from your starting preset.

## Quick start

Latest release: **[v0.7.1](https://github.com/SenjuWoo/FaceForge-BDO/releases/tag/v0.7.1)**.

```text
FaceForge BDO 0.7.1 - STANDALONE.exe
```

Double-click it. The EXE opens a dedicated FaceForge BDO desktop window (WebView2) and keeps its private token-protected service bound to `127.0.0.1` only. Close the window when finished.

**Requirements:** Windows 10/11 x64 and the [Microsoft Edge WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/) (usually already installed).

### Create Face

1. Choose a front-facing target photo (analyzed locally — nothing is uploaded).
2. Choose a starting preset that already works in game.
3. Click **Create Preset** — calibrated controls are written from measured proportions; uncalibrated bytes stay on the base.
4. Save to the BDO Customization folder or download the file.

The main screen always shows how many of the **13** catalogue controls are calibrated. It never pretends uncalibrated sliders can be inferred from a photo alone.

If you calibrated on 0.7.0, re-learn `chin_width` and `nose_length`. Old `forehead_height` entries are ignored.

### Calibrate (Learn)

For each control, make a base save, then drag **exactly one** named in-game slider to maximum and save again. FaceForge diffs the plaintext, records the single moved byte, and stores it in:

```text
%LOCALAPPDATA%\FaceForge BDO\slidermap.json
```

Ambiguous diffs (more than one candidate byte) are refused and listed instead of guessed. A `slidermap.json` next to the EXE seeds a fresh install so one person’s calibration can ship with the release.

### Merge

Optional panel under the main workflow. Mixes only the slider region (bytes 98–220) between same-class presets. Cross-class donors are refused.

### Safe saving

- Detects the usual `Documents\Black Desert\Customization` folder, including common OneDrive layouts
- Scans only files that pass strict preset validation
- Saves through a temporary file and validates before installation
- Creates a timestamped backup before replacing an existing file

## Face measurement (0.7.1+)

Landmarks still come from the bundled MediaPipe Face Landmarker. Geometry is handled by **Skyrim FaceForge’s** measurement pipeline, vendored as `web/js/skyrim-face.js`:

- head-pose estimation and perspective / foreshortening correction
- mirror averaging and per-measurement trust fading
- **baselines** from real neutral heads (not invented min/max windows)

Each BDO slider position is the **deviation from baseline**, mid at 50, tanh-compressed so strong faces approach end stops without pinning.

## Honest limits

- Photo matching only changes **calibrated** controls. Without a slidermap, Create Preset cannot invent face geometry from a photo
- Hair, makeup, colours, body, class, and face type always come from the starting preset
- A wrong calibration offset would corrupt later outputs; Learn refuses multi-byte diffs instead of guessing
- Cross-class merge is blocked because classes can use incompatible meshes and valid ranges
- Game updates may introduce a new preset format version requiring a new parser and map
- The app does not attach to the Black Desert process or automate its UI

## Security model

- The server binds to IPv4 loopback only
- Every API request requires a random per-launch 256-bit token
- The token is delivered in the URL fragment, then retained for the session
- Request bodies are size-limited
- Generated binaries are parsed again before download or save
- Existing files are backed up before replacement
- Photos and presets stay on this PC; nothing is uploaded by FaceForge BDO

## Build from source

Go 1.22+, Node.js 20+ (JavaScript tests), PowerShell 7 or Windows PowerShell 5.1.

```powershell
.\build.ps1
.\package.ps1
```

Output: `artifacts\FaceForge BDO 0.7.1 - STANDALONE.exe`. The committed `web/js/skyrim-face.js` is the pre-bundled analyzer.

## Credits

FaceForge BDO source is MIT. Bundled MediaPipe components use Apache License 2.0. See [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md) and [`licenses/`](licenses/). Measurement geometry is vendored from [FaceForge](https://github.com/SenjuWoo/FaceForge).

Black Desert and Pearl Abyss are trademarks of their owners. This project is unofficial.

## License

[MIT](LICENSE)

## Measured binary map (v20)

Full notes: [`LAYOUT.md`](LAYOUT.md). ICE 8-byte blocks are a **cipher** unit only. FaceForge never treats whole blocks as single features.

| Region | Plaintext bytes | Notes |
| --- | ---: | --- |
| Version header | 0–3 | Little-endian `20` |
| Character name | 8–39 | UTF-16LE, NUL-padded |
| Class id | 80 | Matches filename class prefix on album downloads |
| Face / body sliders | 98–220 | Almost all 0–100; offsets 106, 109, 112 are never written |
| Zero separator | 221–232 | Zero in the research corpus |
| Style / colour / hair / makeup | 233+ | Copied from the base; not photo-driven |

### Controls catalogue (13)

| Control | Metric |
| --- | --- |
| Face length | `faceAspect` |
| Cheekbone width | `cheekWidth` |
| Jaw width | `jawWidth` |
| Chin width | `chinWidth` |
| Chin length | `lowerFace` |
| Eye size | `eyeOpenness` |
| Eye spacing | `eyeSpacing` |
| Eye angle | `eyeTilt` |
| Eyebrow height | `browHeight` |
| Nose width | `noseWidth` |
| Nose length | `noseLength` |
| Mouth width | `mouthWidth` |
| Lip thickness | `lipFullness` |
