# AGENTS.md

GBSC-Pro video upscaler firmware + tooling. Four independently built components; there is no root build, no test suite, and no CI. Verify changes by building the affected component.

## Layout

- `gbs-control/` — ESP8266 firmware (PlatformIO/Arduino). Main logic in `gbs-control.ino`; Pro extensions in `pro/`; TV5725 preset dumps in `ntsc_*.h` / `pal_*.h`.
- `adv-controller/` — HC32F460 firmware controlling ADV7280/ADV7391 (ARM GCC Makefile).
- `gbsc-pro-flasher/` — Python PySide6 GUI/CLI flasher for both MCUs.
- `adv-manager/` — Python PyQt6 serial register debugger for the ADV chips.
- `hardware/` — schematic, BOM, Gerbers, datasheets, Pico ADV CLI simulator.
- `gbs-control/public/` — TypeScript/web UI source; the firmware serves a generated gzipped blob.

## Build / verify

ESP8266 (primary):
```bash
cd gbs-control
python -m platformio run          # pio is not on PATH here; use python -m platformio
python -m platformio run -t upload
```
Artifact: `gbs-control/.pio/build/gbsc-pro/firmware.bin`.

HC32F460:
```bash
cd adv-controller && make         # make debug | make clean | make size
```
Needs `arm-none-eabi-gcc` + GNU make (on Windows, make via Git Bash/MSYS2). Output: `adv-controller/build/{release,debug}/`.

Web UI (source of the blob embedded in firmware):
```bash
cd gbs-control/public
npm install
npm run build                     # tsc -> index.js, build.js -> ../webui.html, html2h.sh -> ../webui_html.h
```
`html2h.sh` needs bash + gzip + xxd + sed; on Windows use Git Bash/WSL or replicate its gzip/xxd steps. Editing `webui.html` or `webui_html.h` alone has no effect: the firmware includes `webui_html.h` (gzip PROGMEM), and `index.js` is generated from `index.ts`.

Tools:
```bash
python gbsc-pro-flasher/gbsc_flasher.py [--list|--adv|--esp] [port] [firmware.bin]
cd adv-manager && pip install -r requirements.txt && python main.py   # needs hardware mod + serial
```

## Firmware gotchas

- PlatformIO config is unusual: `src_dir = ./`, `lib_dir = ./src/`; the `.ino` sits at project root and `src/` holds vendored libraries. Fix `platformio.ini` if either moves.
- `pro/` is compiled directly into the ESP8266 image. `gbs-control.ino` includes `webui_html.h` (around line 9843).
- Output modes are selected by `PresetPreference` in `gbs-control/options.h` and branched inside `applyPresets()` (`gbs-control.ino` ~line 4590). Preset headers are raw TV5725 register dumps (`// s<segment>_<register>` comments), written by `writeProgramArrayNew()`; post-load tweaks are in `doPostPresetLoadSteps()`.
- Adding a resolution/menu option touches several surfaces: enum + `applyPresets()` + serial/web command + OLED menu (`pro/menu/`, `menu-registry.h`) + TV OSD (`pro/osd/`, `osd-registry.h`) + web UI. Missing one makes the option unreachable.
- HD inputs (720p/1080i/1080p) are forced into HDMI bypass via `setOutModeHdBypass()`; only SD/ED inputs (240p/480i/480p/576i/576p) are scaled through presets. `applyPresets()` does not control output timing for HD sources.
- Output timing is non-standard: fixed display-clock codes 0x25..0xA5 and large blanking. `set_htotal()` / `set_vtotal()` are runtime timing helpers. Verify any timing change on real hardware.
- Persistent state: `/preferencesv2.txt` and `/preset_ntsc_*.<slot>` in LittleFS. Firmware version: `GBS_FW_VERSION` in `pro/gbs-control-pro.h`.
- Flash is nearly full (~82% used at v2.4.1 on the 4M1M layout). Recheck the size report after adding code.

## Conventions

- Match existing style; `.clang-format` exists in `gbs-control/` and `adv-controller/`.
- No tests, linters, or CI are configured. A clean build is the minimum verification; behaviour needs the real device.
- This checkout is a local fork: `upstream` → `brisma/gbsc-pro`, work on branch `gbsc-custom`, no `origin` yet. Do not push unless asked.
