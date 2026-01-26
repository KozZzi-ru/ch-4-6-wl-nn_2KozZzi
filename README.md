# ch-4-6-wl-nn_2KozZzi (MK2 - Black)

4-6 wireless zmk firmware for Charybdis

KozZzi adopted

## Features

- Беспроводная прошивка ZMK для Charybdis 4x6
- Контроллер: nice!nano v2
- Трекбол: PMW3610
- Автоматическая генерация SVG-схемы раскладки клавиатуры

## Визуализация раскладки

При каждом push автоматически генерируется SVG-файл с визуальным отображением раскладки клавиатуры.
Файл сохраняется в папке `keymap-drawer/charybdis.svg`.

Благодарности:
- [keymap-drawer](https://github.com/caksoylar/keymap-drawer) от @caksoylar
- Базовая конфигурация из [charybdis-4-6-dongle-prospector-studio](https://github.com/KozZzi-ru/charybdis-4-6-dongle-prospector-studio)

## Изменения

### 2026-01-26: Добавлена автогенерация SVG раскладки

**Добавленные файлы:**
- `.github/workflows/draw_keymaps.yaml` - Reusable workflow для генерации SVG из keymap
- `keymap-drawer/config.yaml` - Конфигурация визуализации (размеры клавиш, цвета, стили)
- `keymap-drawer/charybdis.yaml` - Описание физического layout клавиатуры

**Изменения в существующих файлах:**
- `.github/workflows/build.yml` - Добавлен job `keymap_images`, который запускается после сборки прошивки

**Как это работает:**
1. После push кода запускается GitHub Actions workflow
2. Сначала собирается прошивка (job `build`)
3. Затем запускается `keymap_images` job, который:
   - Парсит файл `config/charybdis.keymap`
   - Использует конфигурацию из `keymap-drawer/config.yaml` и `keymap-drawer/charybdis.yaml`
   - Генерирует файлы `keymap-drawer/charybdis.svg` и `keymap-drawer/charybdis.yaml`
   - Автоматически коммитит изменения (amend к последнему коммиту)

**Примечание:** SVG обновляется автоматически при изменении keymap файла.
