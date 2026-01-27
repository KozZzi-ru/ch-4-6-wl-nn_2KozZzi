# ch-4-6-wl-nn_2KozZzi - НАДО ТЕСТИРОВАТЬ на ChBLK, с моим кеймапом и генерацией раскладки в charybdis.svg

4-6 wireless zmk  
KozZzi adopted

Версия прошивки с плавным скролом  
Переделана под МК2 (BLACK), навален мой кеймаппинг

---

## Изменения в бранче `feature/badjeff-scroll-layers`

### Переход на badjeff/zmk-pmw3610-driver с поддержкой Input Processors

В этом бранче прошивка **переключена с inorichi/zmk-pmw3610-driver на badjeff/zmk-pmw3610-driver**, который поддерживает **ZMK Input Processors** для гибкой настройки скорости скролла на разных слоях.

#### Основные преимущества badjeff драйвера:

✅ **Поддержка Input Processors** — возможность настройки разной скорости скролла на разных слоях  
✅ **Настройки через device tree** — всё конфигурируется в `.overlay` файлах  
✅ **Разделение sampling и reporting rate** — лучшая отзывчивость  
✅ **Совместимость с split клавиатурами**  

#### Настройки скорости скролла по слоям:

| Слой | Название | Скорость скролла | Input Processor |
|------|------------------|---------------------|------------------|
| 0 | QWERTY (base) | **Стандартная** (100%) | `&zip_scroll_scaler 1 1` |
| 1 | num_and_fun | **В 3 раза медленнее** (33%) | `&zip_scroll_scaler 1 3` |
| 3 | snipe | Стандартная (100%) | `&zip_scroll_scaler 1 1` |

#### Как это работает:

- **Базовый слой (QWERTY):** Скролл работает с обычной скоростью
- **Слой 1 (num_and_fun):** При удержании клавиши **Calculator** скорость скролла **автоматически замедляется в 3 раза**
- **Слой 3 (snipe):** Режим точного наведения, стандартная скорость

#### Применение:

Медленный скролл полезен для:
- Точной навигации по документам
- Построчного просмотра кода
- Работы с таблицами и спредшитами
- Чтения длинных текстов

---

### Изменённые файлы

#### 1. `config/west.yml`

Переключение с `inorichi/zmk-pmw3610-driver` на `badjeff/zmk-pmw3610-driver`:

```yaml
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/petejohanson
    - name: badjeff  # НОВЫЙ remote
      url-base: https://github.com/badjeff
  projects:
    - name: zmk
      remote: zmkfirmware
      revision: feat/pointers-move-scroll
      import: app/west.yml
    - name: zmk-pmw3610-driver
      remote: badjeff  # ИЗМЕНЕНО: было inorichi
      revision: main
```

**Что изменилось:**
- Remote `inorichi` заменён на `badjeff`
- Драйвер теперь берётся из https://github.com/badjeff/zmk-pmw3610-driver

---

#### 2. `config/boards/shields/charybdis/charybdis_right.overlay`

Полная переделка конфигурации трекбола:

```devicetree
#include "charybdis.dtsi"
#include <zephyr/dt-bindings/input/input-event-codes.h>

// ... (остальные настройки pinctrl, kscan)

&spi0 {
    trackball: trackball@0 {
        status = "okay";
        compatible = "pixart,pmw3610-alt";  // ИЗМЕНЕНО!
        reg = <0>;
        spi-max-frequency = <2000000>;
        irq-gpios = <&gpio0 6 (GPIO_ACTIVE_LOW | GPIO_PULL_UP)>;
        cpi = <600>;  // ДОБАВЛЕНО
        
        // НОВЫЕ параметры для badjeff драйвера
        evt-type = <INPUT_EV_REL>;
        x-input-code = <INPUT_REL_X>;
        y-input-code = <INPUT_REL_Y>;
    };
};

/ {
  /* ВАЖНО: Определение scroll scaler Input Processor */
  zip_scroll_scaler: zip_scroll_scaler {
    compatible = "zmk,input-processor-scaler";
    #input-processor-cells = <2>;
    type = <INPUT_EV_REL>;
    codes = <INPUT_REL_WHEEL>, <INPUT_REL_HWHEEL>;
  };

  trackball_listener {
    compatible = "zmk,input-listener";
    device = <&trackball>;

    /* Базовый слой 0: стандартная скорость */
    base_layer {
      layers = <0>;
      input-processors = <&zip_scroll_scaler 1 1>;
    };

    /* Слой 1: скорость скролла в 3 раза медленнее */
    slow_scroll_layer {
      layers = <1>;
      input-processors = <&zip_scroll_scaler 1 3>;
    };

    /* Слой 3: режим snipe */
    snipe_layer {
      layers = <3>;
      input-processors = <&zip_scroll_scaler 1 1>;
    };
  };
};
```

**Что изменилось:**
- `compatible`: `"pixart,pmw3610"` → `"pixart,pmw3610-alt"`
- Добавлен `cpi = <600>`
- Добавлены `evt-type`, `x-input-code`, `y-input-code`
- **ВАЖНО:** Добавлено определение `zip_scroll_scaler` (обязательно для работы!)
- Добавлен `trackball_listener` с настройками по слоям
- Удалены старые `scroll-layers`, `snipe-layers`

---

#### 3. `config/boards/shields/charybdis/charybdis_right.conf`

Обновление конфигурации для badjeff драйвера:

```conf
CONFIG_SPI=y
CONFIG_INPUT=y
CONFIG_NFCT_PINS_AS_GPIOS=y
CONFIG_ZMK_EXT_POWER=y

# badjeff PMW3610-ALT driver
CONFIG_PMW3610_ALT=y

# Минимальный интервал отчётов
CONFIG_PMW3610_ALT_REPORT_INTERVAL_MIN=12

# Дополнительная задержка при инициализации (nice_nano_v2)
CONFIG_PMW3610_ALT_INIT_POWER_UP_EXTRA_DELAY_MS=300
```

**Что изменилось:**
- `CONFIG_PMW3610` → `CONFIG_PMW3610_ALT`
- Удалены старые параметры: `CONFIG_PMW3610_CPI`, `CONFIG_PMW3610_ORIENTATION_90`, `CONFIG_PMW3610_SCROLL_TICK` и т.д.
- Добавлены новые параметры для badjeff драйвера

---

#### 4. `README.md`

- Добавлена полная документация по Input Processors
- Примеры настройки скорости
- Инструкции по кастомизации

---

### Техническая реализация Input Processors

#### Что такое `zip_scroll_scaler`?

**`zip_scroll_scaler`** — это встроенный ZMK Input Processor, который масштабирует скорость скролла.

**Синтаксис:**
```devicetree
&zip_scroll_scaler <multiplier> <divisor>
```

**Формула:** `Скорость = (Базовая скорость × multiplier) ÷ divisor`

**Примеры:**
- `&zip_scroll_scaler 1 1` — стандартная скорость (100%)
- `&zip_scroll_scaler 1 2` — половинная скорость (50%)
- `&zip_scroll_scaler 1 3` — скорость в 3 раза медленнее (33%)
- `&zip_scroll_scaler 1 5` — скорость в 5 раз медленнее (20%)
- `&zip_scroll_scaler 2 1` — удвоенная скорость (200%)
- `&zip_scroll_scaler 3 1` — утроенная скорость (300%)

---

### Настройка под свои предпочтения

#### Изменение скорости скролла

Редактируйте `charybdis_right.overlay` в секции `trackball_listener`:

**Сделать медленнее:**
```devicetree
slow_scroll_layer {
  layers = <1>;
  input-processors = <&zip_scroll_scaler 1 5>;  // в 5 раз медленнее
};
```

**Сделать быстрее:**
```devicetree
fast_scroll_layer {
  layers = <2>;
  input-processors = <&zip_scroll_scaler 2 1>;  // в 2 раза быстрее
};
```

**Добавить новый слой:**
```devicetree
custom_layer {
  layers = <4>;
  input-processors = <&zip_scroll_scaler 1 10>;  // очень медленный
};
```

#### Изменение CPI (чувствительности)

Редактируйте `charybdis_right.overlay` в секции `trackball`:

```devicetree
trackball: trackball@0 {
    cpi = <800>;  // Измените значение (200-3200)
};
```

#### Добавление инверсии осей

Для инверсии осей X/Y добавьте в секцию `trackball`:

```devicetree
trackball: trackball@0 {
    cpi = <600>;
    invert-x;  // Инверсия оси X
    invert-y;  // Инверсия оси Y
    swap-xy;   // Поменять оси местами
};
```

---

### Ссылки на документацию

**ZMK Input Processors:**
- [Input Processors Overview](https://zmk.dev/docs/keymaps/input-processors)
- [Scaler Input Processor](https://zmk.dev/docs/keymaps/input-processors/scaler)
- [Input Processor Usage](https://zmk.dev/docs/keymaps/input-processors/usage)

**badjeff/zmk-pmw3610-driver:**
- [GitHub Repository](https://github.com/badjeff/zmk-pmw3610-driver)
- [README с примерами](https://github.com/badjeff/zmk-pmw3610-driver/blob/main/README.md)

---

### Проверка работы

После сборки и прошивки клавиатуры:

1. **Базовый слой (0):** Скролл работает с обычной скоростью
2. **Удерживайте Calculator (слой 1):** Скорость скролла должна замедлиться в 3 раза
3. **Отпустите Calculator:** Скорость восстановится к нормальной

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

## Кредиты

Интеграция основана на:
- [keymap-drawer](https://github.com/caksoylar/keymap-drawer) by [@caksoylar](https://github.com/caksoylar)
- [badjeff/zmk-pmw3610-driver](https://github.com/badjeff/zmk-pmw3610-driver) by [@badjeff](https://github.com/badjeff)
