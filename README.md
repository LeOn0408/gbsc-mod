<p align="center">
    <img alt="gbsc-pro" src="assets/gbsc-pro%20logo.png" />
</p>

---

Custom firmware for the GBSC-Pro video upscaler by **Brisma**.

Converts retro console video signals to HDMI up to 1080p with low latency, advanced color processing, and extensive customization options.

Based on [RetroScaler/gbsc-pro](https://github.com/RetroScaler/gbsc-pro), which is a fork of [gbs-control](https://github.com/ramapcsx2/gbs-control) by ramapcsx2.

> **Fork note (`gbsc-mod`):** all changes below were developed and tested on a **GBSC (original gbs-control hardware), not on GBSC PRO**. The GBSC PRO-specific hardware features (ADV7280/ADV7391 AV module, etc.) were not verified on this fork's test unit.

## Fork changes

- **Geometry presets for 720p**: 16:9, 4:3 pillarbox (960 active pixels centered), Zoom and Overscan. Zoom/Overscan are experimental and session-only; after reboot the saved mode falls back to 16:9.
- **Image controls**: VDS Noise Reduction (Off/Low/Medium/High), peaking level (sharp+), ADC bandwidth (Auto/150/110/70/40 MHz), SOG level (+/-/auto), saturation, tint, RGB balance and persistent contrast. ADC/SOG are session-only and boot in auto.
- **Persistent picture settings**: black level (Y offset), contrast, saturation, tint and RGB balance survive reboots and preset reloads.
- **PS2 profile**: one click sets 720p, motion-adaptive deinterlace, line filter, peaking, NR Low, 4:3 pillarbox and full range for VGA.
- **Recovery**: Reset image and Safe reboot restore a known-good baseline (ADC/SOG auto, 16:9 geometry, no NR/peaking) and reboot when needed.
- **OTA over WiFi**: ArduinoOTA starts automatically after the station connects (hostname `gbsc-mod`); use the `gbsc-pro-ota` PlatformIO environment. No USB needed after the first flash.
- **HDMI Limited Range defaults to OFF**, correct for direct VGA/component output. Use HD/SD/ALL only for HDMI converters that flag limited range.
- Added `AGENTS.md` with build and firmware architecture notes.

## Links

[Manual](https://www.retroscaler.com/?page_id=480) | [Discord](https://discord.com/invite/2MMWRkVRbk) | [Video Tutorial](https://www.youtube.com/playlist?list=PLQ6X-Dl0NtDnZm5v3n3IgvNOSrBsg7s7T)

## Features

### Video Input
- **S-Video** and **Composite** (active processing via ADV7280)
- **RGB**: RGBS, RGsB, RGBHV with auto sync detection
- **Component** (YPbPr)
- **VGA** passthrough

### Video Output
- HDMI up to **1080p**
- Multiple output resolutions: 480p, 720p, 960p, 1024p, 1080p

### Image Processing
- **I2P (Interlace to Progressive)**: Motion-adaptive deinterlacing
- **ACE (Adaptive Contrast Enhancement)**: Dynamic contrast/saturation
- **Video Filters**: Shaping filter, comb filter, CTI for S-Video/Composite
- **Scanlines**: Adjustable intensity and style
- **BCSH**: Brightness, Contrast, Saturation, Hue control
- **ADC Gain**: Y/U/V gain with persistent settings
- **Color standard**: ITU-R BT.601 compliant

### User Interface
- **TV OSD**: On-screen display via STV9426 (4 themes: Classic, Dark, Light, Retro)
- **OLED Menu**: IR remote with key repeat support
- **Web UI**: WiFi-based control panel
- **36 Profile Slots**: A-Z, 0-9 with per-input settings
- **Developer menu**: Advanced hardware settings and I2C commands

### System
- **Auto-switching**: Automatic input detection
- **Low latency**: Real-time signal processing
- **Preset system**: Save/load complete configurations
- **Volume control**: PT2257 audio IC support

## Components

### [gbs-control/](gbs-control/)
**ESP8266 firmware** - Main controller running on the ESP8266. Handles the TV5725 video scaler, TV OSD (STV9426), OLED menu with IR remote, web interface, and communicates with the ADV controller via UART.

### [adv-controller/](adv-controller/)
**HC32F460 firmware** - Secondary controller managing the ADV7280 (video decoder for S-Video/Composite) and ADV7391 (video encoder). Receives commands from the ESP8266 and controls the ADV chips via I2C.

### [adv-manager/](adv-manager/)
**Python GUI** - Desktop application for debugging ADV7280/ADV7391 registers via serial connection. Useful for testing video processing settings without reflashing firmware.

### [gbsc-pro-flasher/](gbsc-pro-flasher/)
**Firmware Flash Tool** - Cross-platform GUI/CLI tool to flash both ESP8266 and HC32 firmware. Auto-detects connected devices and supports drag & drop. Works on Windows, macOS, and Linux.

### [hardware/](hardware/)
**Hardware resources** - PCB design files (Gerber), BOM, datasheets for all ICs, and an ADV CLI simulator for testing without physical hardware.

## Building

```bash
# ESP8266 firmware
cd gbs-control && pio run
# Windows / PlatformIO installed via pip: python -m platformio run

# OTA over WiFi (ArduinoOTA starts automatically after WiFi connect)
cd gbs-control
python -m platformio run -e gbsc-pro-ota -t upload --upload-port <device-ip>

# HC32 firmware
cd adv-controller && make

# Web interface (needs bash + gzip + xxd for scripts/html2h.sh)
cd gbs-control/public && npm install && npm run build
```

The web UI is embedded as a gzipped blob (`gbs-control/webui_html.h`). Editing `webui.html` or `webui_html.h` alone has no effect; edit `public/src/index.html.tpl` / `index.ts` and rebuild.

## Hardware

Pre-assembled kits available at [Official Store](https://www.aliexpress.com/item/3256808268415575.html).

Or [build it yourself](hardware/gerber/) using the open-source design files.

![](assets/product-01.jpg)

## Community

[Create an issue](https://github.com/brisma/gbsc-pro/issues/new) or join the [Discord](https://discord.com/invite/2MMWRkVRbk).

## Thanks

- [@ramapcsx2](https://github.com/ramapcsx2) for [gbs-control](https://github.com/ramapcsx2/gbs-control)

---

# Перевод на русский

Кастомная прошивка для видеоапскейлера **GBSC-Pro** от **Brisma**.

Преобразует сигналы ретро-консолей в HDMI до 1080p с низкой задержкой, продвинутой обработкой цвета и широкими возможностями настройки.

Основано на [RetroScaler/gbsc-pro](https://github.com/RetroScaler/gbsc-pro), который является форком [gbs-control](https://github.com/ramapcsx2/gbs-control) от ramapcsx2.

> **Примечание о форке (`gbsc-mod`):** все изменения ниже разработаны и проверены на **GBSC (оригинальное железо gbs-control), а не на GBSC PRO**. Аппаратные функции, специфичные для GBSC PRO (AV-модуль ADV7280/ADV7391 и т.д.), на тестовом экземпляре этого форка не проверялись.

## Изменения форка

- **Геометрические пресеты для 720p**: 16:9, 4:3 pillarbox (960 активных пикселей по центру), Zoom и Overscan. Zoom/Overscan экспериментальные и действуют только до перезагрузки — после ребута возвращается 16:9.
- **Настройки изображения**: VDS Noise Reduction (Off/Low/Medium/High), уровень peaking (sharp+), полоса ADC (Auto/150/110/70/40 МГц), уровень SOG (+/-/auto), насыщенность, tint, RGB-баланс и сохраняемый контраст. ADC/SOG действуют в рамках сессии, при загрузке — auto.
- **Сохраняемые настройки картинки**: уровень чёрного (Y offset), контраст, насыщенность, tint и RGB-баланс переживают перезагрузку и смену пресетов.
- **Профиль PS2**: одной кнопкой включает 720p, motion-adaptive деинтерлейс, line filter, peaking, NR Low, 4:3 pillarbox и full range для VGA.
- **Восстановление**: кнопки Reset image и Safe reboot возвращают безопасный baseline (ADC/SOG auto, геометрия 16:9, без NR/peaking) и при необходимости перезагружают GBS.
- **OTA по WiFi**: ArduinoOTA запускается автоматически после подключения к сети (hostname `gbsc-mod`); используйте окружение PlatformIO `gbsc-pro-ota`. После первой прошивки USB не нужен.
- **HDMI Limited Range по умолчанию OFF** — это правильно для прямого вывода VGA/компонента. HD/SD/ALL используйте только для HDMI-конвертеров, которые помечают сигнал как limited range.
- Добавлен `AGENTS.md` с заметками по сборке и архитектуре прошивки.

## Ссылки

[Руководство](https://www.retroscaler.com/?page_id=480) | [Discord](https://discord.com/invite/2MMWRkVRbk) | [Видеоинструкция](https://www.youtube.com/playlist?list=PLQ6X-Dl0NtDnZm5v3n3IgvNOSrBsg7s7T)

## Возможности

### Видеовход
- **S-Video** и **Composite** (активная обработка через ADV7280)
- **RGB**: RGBS, RGsB, RGBHV с автоопределением синхронизации
- **Component** (YPbPr)
- **VGA** passthrough

### Видеовыход
- HDMI до **1080p**
- Различные выходные разрешения: 480p, 720p, 960p, 1024p, 1080p

### Обработка изображения
- **I2P (Interlace to Progressive)**: motion-adaptive деинтерлейсинг
- **ACE (Adaptive Contrast Enhancement)**: динамический контраст/насыщенность
- **Видеофильтры**: shaping, comb, CTI для S-Video/Composite
- **Scanlines**: регулируемая интенсивность и стиль
- **BCSH**: яркость, контраст, насыщенность, оттенок
- **ADC Gain**: усиление Y/U/V с сохранением настроек
- **Цветовой стандарт**: соответствие ITU-R BT.601

### Интерфейс
- **TV OSD**: экранное меню через STV9426 (4 темы: Classic, Dark, Light, Retro)
- **OLED-меню**: ИК-пульт с поддержкой повторов
- **Web UI**: панель управления по WiFi
- **36 слотов профилей**: A-Z, 0-9 с настройками для каждого входа
- **Меню разработчика**: расширенные настройки железа и I2C-команды

### Система
- **Автопереключение**: автоматическое определение входа
- **Низкая задержка**: обработка сигнала в реальном времени
- **Система пресетов**: сохранение/загрузка полных конфигураций
- **Регулировка громкости**: поддержка аудиочипа PT2257

## Компоненты

### [gbs-control/](gbs-control/)
**Прошивка ESP8266** — основной контроллер на ESP8266. Управляет видеоскейлером TV5725, TV OSD (STV9426), OLED-меню с ИК-пультом, веб-интерфейсом и общается с ADV-контроллером по UART.

### [adv-controller/](adv-controller/)
**Прошивка HC32F460** — вторичный контроллер, управляющий ADV7280 (видеодекодер для S-Video/Composite) и ADV7391 (видеоэнкодер). Принимает команды от ESP8266 и управляет чипами ADV по I2C.

### [adv-manager/](adv-manager/)
**Python GUI** — настольное приложение для отладки регистров ADV7280/ADV7391 через последовательный порт. Полезно для проверки настроек обработки видео без перепрошивки.

### [gbsc-pro-flasher/](gbsc-pro-flasher/)
**Утилита прошивки** — кроссплатформенный GUI/CLI для прошивки ESP8266 и HC32. Автоматически определяет подключённые устройства, поддерживает drag & drop. Работает на Windows, macOS и Linux.

### [hardware/](hardware/)
**Аппаратные ресурсы** — файлы печатной платы (Gerber), BOM, даташиты на все микросхемы и симулятор ADV CLI для тестирования без реального железа.

## Сборка

```bash
# Прошивка ESP8266
cd gbs-control && pio run
# Windows / PlatformIO, установленный через pip: python -m platformio run

# OTA по WiFi (ArduinoOTA запускается автоматически после подключения к WiFi)
cd gbs-control
python -m platformio run -e gbsc-pro-ota -t upload --upload-port <device-ip>

# Прошивка HC32
cd adv-controller && make

# Веб-интерфейс (нужны bash + gzip + xxd для scripts/html2h.sh)
cd gbs-control/public && npm install && npm run build
```

Веб-интерфейс встроен в прошивку как gzip-блоб (`gbs-control/webui_html.h`). Редактирование только `webui.html` или `webui_html.h` ничего не меняет: правьте `public/src/index.html.tpl` / `index.ts` и пересобирайте.

## Железо

Готовые комплекты доступны в [официальном магазине](https://www.aliexpress.com/item/3256808268415575.html).

Или [соберите самостоятельно](hardware/gerber/) по открытым файлам проекта.

![](assets/product-01.jpg)

## Сообщество

[Создать issue](https://github.com/brisma/gbsc-pro/issues/new) или присоединиться к [Discord](https://discord.com/invite/2MMWRkVRbk).

## Благодарности

- [@ramapcsx2](https://github.com/ramapcsx2) за [gbs-control](https://github.com/ramapcsx2/gbs-control)
