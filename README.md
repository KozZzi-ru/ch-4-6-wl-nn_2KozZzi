# ch-4-6-wl-nn_2KozZzi - полностью рабочий, с моим кеймапом и генерацией раскладки в charybdis.svg

4-6 wireless zmk  
KozZzi adopted

Версия прошивки с плавным скролом  
Переделана под МЖ2 (BLACK), навален мой кеймаппинг

---

## Изменения в бранче `feature/layer-scroll-speed`

### Разная скорость скролла на разных слоях

Добавлена возможность **изменять скорость скролла трекбола в зависимости от активного слоя** с использованием ZMK Input Processors.

#### Настройки скорости по слоям:

| Слой | Название | Скорость скролла |
|------|------------------|---------------------|
| 0 | QWERTY | **Стандартная** (1:1) |
| 1 | num_and_fun | **В 3 раза медленнее** (1:3) |
| 2-5 | остальные | Стандартная (по умолчанию) |

#### Как это работает:

- **Базовый слой (QWERTY):** При работе на основном слое скролл работает с обычной скоростью
- **Слой 1 (num_and_fun):** При переключении на этот слой (удержанием клавиши Calculator) скролл замедляется в 3 раза

#### Применение:

Медленный скролл полезен для:
- Точной навигации по документам
- Просмотра кода (построчного скролла)
- Работы с таблицами
- Чтения длинных текстов

#### Техническая реализация:

Используется **ZMK Input Processor `zip_scroll_scaler`** — встроенный компонент для масштабирования скорости скролла.

**Синтаксис:**
```devicetree
&zip_scroll_scaler <multiplier> <divisor>
```

**Примеры:**
- `&zip_scroll_scaler 1 1` — стандартная скорость (100%)
- `&zip_scroll_scaler 1 2` — половинная скорость (50%)
- `&zip_scroll_scaler 1 3` — скорость в 3 раза медленнее (33%)
- `&zip_scroll_scaler 2 1` — удвоенная скорость (200%)

#### Измененные файлы:

**`config/boards/shields/charybdis/charybdis_right.overlay`**

Добавлена конфигурация `trackball_listener` с настройкой Input Processors для разных слоёв:

```devicetree
/ {
  trackball_listener {
    compatible = "zmk,input-listener";
    device = <&trackball>;

    /* Базовый слой (0 - QWERTY): стандартная скорость скролла */
    base_layer {
      layers = <0>;
      input-processors = <&zip_scroll_scaler 1 1>;
    };

    /* Слой 1 (num_and_fun): скорость скролла уменьшена в 3 раза */
    slow_scroll_layer {
      layers = <1>;
      input-processors = <&zip_scroll_scaler 1 3>;
    };
  };
};
```

#### Настройка под свои предпочтения:

Если вы хотите изменить скорость скролла:

1. **Сделать медленнее:** увеличьте divisor (третий параметр)
   - `<&zip_scroll_scaler 1 5>` — в 5 раз медленнее
   - `<&zip_scroll_scaler 1 10>` — в 10 раз медленнее

2. **Сделать быстрее:** увеличьте multiplier (второй параметр)
   - `<&zip_scroll_scaler 2 1>` — в 2 раза быстрее
   - `<&zip_scroll_scaler 3 1>` — в 3 раза быстрее

3. **Добавить настройки для других слоёв:**
   ```devicetree
   another_layer {
     layers = <2>;  // номер слоя
     input-processors = <&zip_scroll_scaler 1 5>;  // в 5 раз медленнее
   };
   ```

#### Ссылки на документацию:

- [ZMK Input Processors Overview](https://zmk.dev/docs/keymaps/input-processors)
- [ZMK Scaler Input Processor](https://zmk.dev/docs/keymaps/input-processors/scaler)
- [ZMK Input Processor Usage](https://zmk.dev/docs/keymaps/input-processors/usage)

---

## Изменения в бранче `add-svg-generation-and-packaging`

### SVG-схема в архиве прошивки

Теперь при сборке **SVG-схема раскладки добавляется в zip-архив с UF2-файлами прошивки!**

**Преимущества:**
- Открыв `firmware_BLK_SVG.zip`, сразу видно кеймап
- Не нужно искать SVG в репозитории
- Кеймап сохраняется вместе с прошивкой

**Порядок сборки:**

1. **Генерация SVG** (`keymap_images`) → `charybdis.svg`
2. **Сборка прошивки** (`build`) → `firmware_BLK.zip`
3. **Упаковка** (`package_with_keymap`) → `firmware_BLK_SVG.zip` (с SVG внутри)

**Артефакты GitHub Actions:**

- ✅ `firmware_BLK_SVG` - **полный пакет** (UF2 + SVG)
- 📄 `firmware_BLK` - только UF2-файлы
- 🎨 `keymap_raw_files` - только SVG/YAML

---

## Автоматическая генерация SVG-схемы клавиатуры

Теперь при каждом коммите автоматически генерируется визуальная схема раскладки клавиатуры с помощью [keymap-drawer](https://github.com/caksoylar/keymap-drawer).

---

## Добавленные файлы

### 1. `.github/workflows/draw_keymaps.yaml`

**Reusable workflow** для автоматической генерации SVG-схем клавиатуры.

**Что делает:**
- Запускается при каждой сборке прошивки
- Парсит `config/charybdis.keymap`
- Генерирует `charybdis.svg` и `charybdis.yaml` в папке `keymap-drawer/`
- Автоматически коммитит изменения

**Особенности:**
- Использует `amend_commit: true` - перезаписывает последний коммит, а не создает новый
- Поддерживает west modules для ZMK
- Кастомные биндинги мыши и макросов

### 2. `keymap-drawer/config.yaml`

**Конфигурация внешнего вида** генерируемой SVG-схемы.

**Настройки:**
- **Размеры клавиш:** 60x56 px
- **Разрыв между половинами:** 30 px
- **Тема:** auto (автоматическая смена светлой/тёмной)
- **Комбо-диаграммы:** отдельные, масштаб x2
- **Маппинг ZMK-клавиш:** сокращённые названия

### 3. `config/charybdis.json`

**Физическое описание раскладки** Charybdis 4x6.

**Что содержит:**
- **Координаты** каждой клавиши (x, y, rotation)
- **Два layout:** `default_transform` и `charybdis_6col_layout`
- **56 клавиш:** 48 основных + 8 тамбовых (с учётом трекбола)

**Назначение:**
- Используется `keymap-drawer` для определения позиций клавиш
- Автоматически обнаруживается workflow при наличии в `config/`

---

## Изменённые файлы

### `.github/workflows/build.yml`

**Изменения структуры:**

1. **Порядок джобов:**
   - `keymap_images` запускается **первым**
   - `build` ждёт `keymap_images`
   - `package_with_keymap` ждёт обоих

2. **Добавлен `destination: 'both'`:**
   - SVG сохраняется и в репозиторий, и в артефакты

3. **Новый джоб `package_with_keymap`:**
   - Скачивает `firmware_BLK.zip`
   - Скачивает `charybdis.svg`
   - Распаковывает zip
   - Добавляет SVG в папку с UF2
   - Запаковывает обратно
   - Загружает как `firmware_BLK_SVG`

---

## Результат

После каждого коммита в папке `keymap-drawer/` автоматически обновляются:

- **`charybdis.svg`** - визуальная схема раскладки
- **`charybdis.yaml`** - YAML-описание раскладки

### Просмотр схемы

Схему можно просматривать:
- Напрямую в GitHub: [`keymap-drawer/charybdis.svg`](keymap-drawer/charybdis.svg)
- Вставить в этот README:

```markdown
![Keymap](keymap-drawer/charybdis.svg)
```

---

## Как работает автогенерация

### Workflow последовательность:

1. **Push/Pull Request** → запуск GitHub Actions
2. **Draw keymaps** → генерация SVG
3. **Build ZMK firmware** → сборка прошивки
4. **Package with keymap** → объединение SVG + UF2

### Кастомные биндинги:

Workflow распознаёт кастомные ZMK-биндинги:
- **Мышь:** `&mkp LCLK`, `&mkp RCLK`, `&mkp MCLK`, `&mkp MB4`, `&mkp MB5`
- **Движение мыши:** `&mmv MOVE_UP`, `&mmv MOVE_DOWN`, `&mmv MOVE_LEFT`, `&mmv MOVE_RIGHT`
- **Скролл:** `&msc MOVE_UP`, `&msc MOVE_DOWN`, `&msc MOVE_LEFT`, `&msc MOVE_RIGHT`
- **Кастом:** `&HSplit`, `&VSplit`, `&caps_word`, `&mmv_slow`
- **Макросы:** `&phrase_proshu` ("ПРОШУ ПРОЩЕНИЯ"), `&phrase_spasibo` ("СПАСИБО!")

---

## Кредиты

Интеграция основана на [keymap-drawer](https://github.com/caksoylar/keymap-drawer) by [@caksoylar](https://github.com/caksoylar)

---

## Дополнительно

### Отключение amend_commit:

Если не хотите перезаписывать последний коммит, измените в `build.yml`:

```yaml
keymap_images:
  permissions:
    contents: write
  uses: ./.github/workflows/draw_keymaps.yaml
  with:
    amend_commit: false  # Создавать новый коммит
    destination: 'both'
    artifact_name: 'keymap_raw_files'
```

### Изменение настроек визуализации:

Редактируйте `keymap-drawer/config.yaml` для тонкой настройки:
- Размеры клавиш (`key_w`, `key_h`)
- Цветовые схемы (`dark_mode`)
- Шрифты и размеры текста (`glyph_tap_size`, `glyph_hold_size`)
- Маппинг клавиш (`zmk_keycode_map`)

### Добавление новых кастомных биндингов:

Для распознавания новых макросов или специальных биндингов добавьте их в `KEYMAP_raw_binding_map` в файле `.github/workflows/draw_keymaps.yaml`:

```yaml
KEYMAP_raw_binding_map: >
  {
    "&your_custom_macro": "DISPLAY NAME",
    "&another_binding": "SHORT NAME"
  }
```
