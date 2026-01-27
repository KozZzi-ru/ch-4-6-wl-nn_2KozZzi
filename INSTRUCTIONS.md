# 📚 ПОЛНОЕ ПОСОБИЕ: Настройка замедленного скролла на отдельном слое в ZMK

## 🎯 Цель
Настроить трекбол так, чтобы при переходе на определённый слой клавиатуры скорость скролла автоматически изменялась (замедлялась или ускорялась).

---

## 📋 ОГЛАВЛЕНИЕ

1. [Предварительный анализ](#1-предварительный-анализ)
2. [Выбор драйвера](#2-выбор-драйвера)
3. [Подготовка файлов](#3-подготовка-файлов)
4. [Настройка west.yml](#4-настройка-westyml)
5. [Настройка overlay файла](#5-настройка-overlay-файла)
6. [Настройка конфигурационного файла](#6-настройка-конфигурационного-файла)
7. [Проверка перед сборкой](#7-проверка-перед-сборкой)
8. [Устранение типовых ошибок](#8-устранение-типовых-ошибок)
9. [Тестирование прошивки](#9-тестирование-прошивки)
10. [Тонкая настройка](#10-тонкая-настройка)

---

## 1. ПРЕДВАРИТЕЛЬНЫЙ АНАЛИЗ

### ✅ ШАГ 1.1: Проверь текущую конфигурацию

**Определи используемый драйвер:**

```bash
# Проверь config/west.yml
cat config/west.yml | grep "zmk-pmw3610-driver" -A 2
```

**Возможные варианты:**
- `inorichi/zmk-pmw3610-driver` — **НЕ поддерживает** Input Processors
- `badjeff/zmk-pmw3610-driver` — **поддерживает** Input Processors

### ✅ ШАГ 1.2: Определи номера слоёв

**Проверь charybdis.keymap:**

```bash
# Найди определения слоёв
grep -n "^        [A-Za-z_]* {" config/charybdis.keymap
```

**Запиши номера слоёв:**
- Слой 0: QWERTY (базовый)
- Слой 1: num_and_fun (цифры/функции)
- Слой 2: auto_mouse
- Слой 3: snipe_layer
- И т.д.

### ✅ ШАГ 1.3: Определи клавишу активации слоя

**Найди, какая клавиша активирует нужный слой:**

```bash
# Ищи lt 1 (layer-tap для слоя 1)
grep "lt 1" config/charybdis.keymap
```

**Пример:** `&lt 1 C_AL_CALCULATOR` → клавиша Calculator активирует слой 1

### 📝 ЧЕКЛИСТ:
- [ ] Определён текущий драйвер
- [ ] Записаны номера всех слоёв
- [ ] Определены клавиши активации слоёв
- [ ] Известна желаемая скорость скролла для каждого слоя

---

## 2. ВЫБОР ДРАЙВЕРА

### 🔍 Сравнение драйверов

| Критерий | inorichi | badjeff |
|----------|----------|---------||
| **Input Processors** | ❌ Нет | ✅ Есть |
| **Разная скорость скролла на слоях** | ❌ Нет | ✅ Есть |
| **Стабильность** | ✅ Проверен | ⚠️ Новее |
| **Настройка** | Kconfig (.conf) | Device Tree (.overlay) |
| **Энергосбережение** | ✅ Smart algorithm | ⚠️ Меньше опций |

### ⚠️ КРИТИЧНОЕ РЕШЕНИЕ

**Если используешь inorichi → МИГРАЦИЯ ОБЯЗАТЕЛЬНА**

Без миграции на badjeff **невозможно** реализовать разную скорость скролла на слоях!

### ✅ ШАГ 2.1: Решение о миграции

**ЕСЛИ драйвер = inorichi:**
```
✅ Переходим на badjeff (см. раздел 4)
```

**ЕСЛИ драйвер = badjeff:**
```
✅ Пропускаем раздел 4, переходим к разделу 5
```

---

## 3. ПОДГОТОВКА ФАЙЛОВ

### 📂 Структура репозитория

```
config/
├── west.yml                              # Зависимости
├── charybdis.keymap                      # Раскладка (не трогаем)
├── charybdis.conf                        # Общие настройки
└── boards/shields/charybdis/
    ├── charybdis_right.overlay          # ✏️ ГЛАВНЫЙ ФАЙЛ ДЛЯ ПРАВКИ
    └── charybdis_right.conf             # ✏️ Конфигурация драйвера
```

### ✅ ШАГ 3.1: Создай резервную копию

```bash
# Создай новую ветку
git checkout -b feature/scroll-speed-layers

# Скопируй важные файлы
cp config/west.yml config/west.yml.backup
cp config/boards/shields/charybdis/charybdis_right.overlay config/boards/shields/charybdis/charybdis_right.overlay.backup
cp config/boards/shields/charybdis/charybdis_right.conf config/boards/shields/charybdis/charybdis_right.conf.backup
```

### 📝 ЧЕКЛИСТ:
- [ ] Создана новая git-ветка
- [ ] Сделаны резервные копии файлов
- [ ] Коммит перед началом изменений

---

## 4. НАСТРОЙКА west.yml

### ✅ ШАГ 4.1: Открой west.yml

```yaml
# config/west.yml
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/petejohanson
    - name: inorichi  # ← СТАРЫЙ ДРАЙВЕР
      url-base: https://github.com/inorichi
  projects:
    - name: zmk
      remote: zmkfirmware
      revision: feat/pointers-move-scroll
      import: app/west.yml
    - name: zmk-pmw3610-driver
      remote: inorichi  # ← МЕНЯЕМ НА badjeff
      revision: main
  self:
    path: config
```

### ✅ ШАГ 4.2: Замени на badjeff

```yaml
# config/west.yml
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/petejohanson
    - name: badjeff  # ✏️ ИЗМЕНЕНО
      url-base: https://github.com/badjeff
  projects:
    - name: zmk
      remote: zmkfirmware
      revision: feat/pointers-move-scroll
      import: app/west.yml
    - name: zmk-pmw3610-driver
      remote: badjeff  # ✏️ ИЗМЕНЕНО
      revision: main
  self:
    path: config
```

### ⚠️ ВАЖНО:
- Название `zmk-pmw3610-driver` **НЕ МЕНЯЕТСЯ**
- Меняется только `remote: badjeff`
- `revision: main` остаётся без изменений

### 📝 ЧЕКЛИСТ:
- [ ] remote изменён на `badjeff`
- [ ] Проект всё ещё называется `zmk-pmw3610-driver`
- [ ] Файл сохранён
- [ ] Сделан коммит: `git commit -m "feat: switch to badjeff/zmk-pmw3610-driver"`

---

## 5. НАСТРОЙКА OVERLAY ФАЙЛА

### ✅ ШАГ 5.1: Открой charybdis_right.overlay

```bash
vim config/boards/shields/charybdis/charybdis_right.overlay
```

### ✅ ШАГ 5.2: Добавь include для input codes

**В НАЧАЛО файла после других includes:**

```devicetree
#include "charybdis.dtsi"
#include <zephyr/dt-bindings/input/input-event-codes.h>  // ✏️ ДОБАВИТЬ
```

### ✅ ШАГ 5.3: Найди секцию &spi0

```devicetree
&spi0 {
    status = "okay";
    compatible = "nordic,nrf-spim";
    pinctrl-0 = <&spi0_default>;
    pinctrl-1 = <&spi0_sleep>;
    pinctrl-names = "default", "sleep";
    cs-gpios = <&gpio0 20 GPIO_ACTIVE_LOW>;

    trackball: trackball@0 {
        status = "okay";
        compatible = "pixart,pmw3610";  // ← СТАРЫЙ ДРАЙВЕР
        reg = <0>;
        spi-max-frequency = <2000000>;
        irq-gpios = <&gpio0 6 (GPIO_ACTIVE_LOW | GPIO_PULL_UP)>;

        /* optional features */
        scroll-layers = <0>;   // ← СТАРЫЙ ПОДХОД
        snipe-layers = <3>;    // ← СТАРЫЙ ПОДХОД
    };
};
```

### ✅ ШАГ 5.4: Обнови секцию trackball

```devicetree
&spi0 {
    status = "okay";
    compatible = "nordic,nrf-spim";
    pinctrl-0 = <&spi0_default>;
    pinctrl-1 = <&spi0_sleep>;
    pinctrl-names = "default", "sleep";
    cs-gpios = <&gpio0 20 GPIO_ACTIVE_LOW>;

    trackball: trackball@0 {
        status = "okay";
        compatible = "pixart,pmw3610-alt";  // ✏️ ИЗМЕНЕНО
        reg = <0>;
        spi-max-frequency = <2000000>;
        irq-gpios = <&gpio0 6 (GPIO_ACTIVE_LOW | GPIO_PULL_UP)>;
        
        // ✏️ НОВЫЕ ПАРАМЕТРЫ
        cpi = <600>;  // Чувствительность (200-3200)
        
        evt-type = <INPUT_EV_REL>;
        x-input-code = <INPUT_REL_X>;
        y-input-code = <INPUT_REL_Y>;
        
        // ✏️ УДАЛЕНЫ: scroll-layers, snipe-layers
    };
};
```

### ✅ ШАГ 5.5: Добавь Input Processor Definition

**СРАЗУ ПОСЛЕ секции &spi0, добавь:**

```devicetree
/ {
  /* Определение scroll scaler Input Processor */
  zip_scroll_scaler: zip_scroll_scaler {
    compatible = "zmk,input-processor-scaler";
    #input-processor-cells = <2>;
    type = <INPUT_EV_REL>;
    codes = <INPUT_REL_WHEEL>, <INPUT_REL_HWHEEL>;
  };

  trackball_listener {
    compatible = "zmk,input-listener";
    device = <&trackball>;

    /* Базовый слой 0: стандартная скорость скролла */
    base_layer {
      layers = <0>;
      input-processors = <&zip_scroll_scaler 1 1>;
    };

    /* Слой 1: скорость скролла уменьшена в 3 раза */
    slow_scroll_layer {
      layers = <1>;
      input-processors = <&zip_scroll_scaler 1 3>;
    };

    /* Слой 3: стандартная скорость */
    snipe_layer {
      layers = <3>;
      input-processors = <&zip_scroll_scaler 1 1>;
    };
  };
};
```

### 🎯 ФОРМУЛА СКОРОСТИ

```
Скорость = (Базовая × multiplier) ÷ divisor

&zip_scroll_scaler <multiplier> <divisor>
```

**Примеры:**
- `<1 1>` → 100% (стандарт)
- `<1 2>` → 50% (в 2 раза медленнее)
- `<1 3>` → 33% (в 3 раза медленнее)
- `<1 5>` → 20% (в 5 раз медленнее)
- `<2 1>` → 200% (в 2 раза быстрее)
- `<3 1>` → 300% (в 3 раза быстрее)

### ⚠️ КРИТИЧНЫЕ МОМЕНТЫ

1. **Определение ОБЯЗАТЕЛЬНО до использования:**
   ```devicetree
   // ✅ ПРАВИЛЬНО
   zip_scroll_scaler: zip_scroll_scaler { ... };
   trackball_listener { input-processors = <&zip_scroll_scaler 1 3>; };
   
   // ❌ ОШИБКА: undefined node label
   trackball_listener { input-processors = <&zip_scroll_scaler 1 3>; };
   ```

2. **Номера слоёв должны совпадать с keymap:**
   ```devicetree
   // Если в keymap слой называется num_and_fun и это слой 1
   slow_scroll_layer {
     layers = <1>;  // ← СОВПАДАЕТ
   };
   ```

3. **Можно настроить несколько слоёв:**
   ```devicetree
   trackball_listener {
     layer_0 { layers = <0>; input-processors = <&zip_scroll_scaler 1 1>; };
     layer_1 { layers = <1>; input-processors = <&zip_scroll_scaler 1 3>; };
     layer_2 { layers = <2>; input-processors = <&zip_scroll_scaler 2 1>; };
     layer_3 { layers = <3>; input-processors = <&zip_scroll_scaler 1 1>; };
   };
   ```

### 📝 ЧЕКЛИСТ:
- [ ] `compatible` изменён на `"pixart,pmw3610-alt"`
- [ ] Добавлены `cpi`, `evt-type`, `x-input-code`, `y-input-code`
- [ ] Удалены `scroll-layers`, `snipe-layers`
- [ ] Добавлено определение `zip_scroll_scaler`
- [ ] Настроен `trackball_listener` с нужными слоями
- [ ] Номера слоёв совпадают с keymap
- [ ] Файл сохранён

---

## 6. НАСТРОЙКА КОНФИГУРАЦИОННОГО ФАЙЛА

### ✅ ШАГ 6.1: Открой charybdis_right.conf

```bash
vim config/boards/shields/charybdis/charybdis_right.conf
```

### ✅ ШАГ 6.2: Замени конфигурацию драйвера

**БЫЛО (inorichi):**
```conf
CONFIG_PMW3610=y
CONFIG_PMW3610_CPI=1500
CONFIG_PMW3610_CPI_DIVIDOR=4
CONFIG_PMW3610_ORIENTATION_90=y
CONFIG_PMW3610_SNIPE_CPI=800
CONFIG_PMW3610_SCROLL_TICK=32
CONFIG_PMW3610_INVERT_X=y
CONFIG_PMW3610_SMART_ALGORITHM=y
# ... и т.д.
```

**СТАЛО (badjeff):**
```conf
CONFIG_SPI=y
CONFIG_INPUT=y
CONFIG_NFCT_PINS_AS_GPIOS=y

CONFIG_ZMK_EXT_POWER=y

# badjeff PMW3610-ALT driver
CONFIG_PMW3610_ALT=y

# Минимальный интервал отчётов (в мс)
CONFIG_PMW3610_ALT_REPORT_INTERVAL_MIN=12

# Дополнительная задержка при инициализации (nice_nano_v2 fix)
CONFIG_PMW3610_ALT_INIT_POWER_UP_EXTRA_DELAY_MS=300

# Smart algorithm для улучшенного отслеживания (опционально)
# CONFIG_PMW3610_ALT_SMART_ALGORITHM=y

# Отладка (раскомментировать при проблемах)
# CONFIG_PMW3610_ALT_LOG_LEVEL_DBG=y
```

### ⚠️ ВАЖНЫЕ ЗАМЕЧАНИЯ

1. **CONFIG_ZMK_POINTING НЕ СУЩЕСТВУЕТ** в ветке feat/pointers-move-scroll
   - Если добавишь — получишь ошибку сборки
   - Не копируй из примеров других прошивок слепо!

2. **CONFIG_ZMK_MOUSE** — опционально
   - Нужен только если используешь mouse emulation behaviors
   - Для трекбола НЕ ОБЯЗАТЕЛЕН

3. **CPI теперь в .overlay, а не в .conf**
   - В старом драйвере: `CONFIG_PMW3610_CPI=1500`
   - В новом: `cpi = <600>;` в overlay

### 📝 ЧЕКЛИСТ:
- [ ] Заменён `CONFIG_PMW3610` на `CONFIG_PMW3610_ALT`
- [ ] Удалены ВСЕ старые параметры `CONFIG_PMW3610_*`
- [ ] НЕ добавлен `CONFIG_ZMK_POINTING`
- [ ] Добавлены минимальные настройки badjeff
- [ ] Файл сохранён
- [ ] Сделан коммит

---

## 7. ПРОВЕРКА ПЕРЕД СБОРКОЙ

### ✅ ШАГ 7.1: Проверь синтаксис overlay

```bash
# Установи dtc (Device Tree Compiler) если нет
sudo apt-get install device-tree-compiler  # Debian/Ubuntu
brew install dtc  # macOS

# Проверь синтаксис (НЕ ПОЛНАЯ ПРОВЕРКА, но базовые ошибки найдёт)
dtc -I dts -O dtb config/boards/shields/charybdis/charybdis_right.overlay -o /dev/null
```

### ✅ ШАГ 7.2: Проверь структуру файлов

```bash
# Убедись, что все файлы на месте
ls -la config/west.yml
ls -la config/boards/shields/charybdis/charybdis_right.overlay
ls -la config/boards/shields/charybdis/charybdis_right.conf

# Проверь, что в overlay есть определение zip_scroll_scaler
grep "zip_scroll_scaler: zip_scroll_scaler" config/boards/shields/charybdis/charybdis_right.overlay
```

### ✅ ШАГ 7.3: Сверь номера слоёв

```bash
# Найди слои в keymap
grep -n "^        [A-Za-z_]* {" config/charybdis.keymap | head -10

# Пример вывода:
# 123:        QWERTY {           # Слой 0
# 234:        num_and_fun {      # Слой 1
# 345:        auto_mouse {       # Слой 2
# 456:        snipe_layer {      # Слой 3

# Проверь, что в overlay используются те же номера
grep "layers = <" config/boards/shields/charybdis/charybdis_right.overlay
```

### ✅ ШАГ 7.4: Проверочный чеклист

**Файл west.yml:**
- [ ] remote = `badjeff`
- [ ] url-base = `https://github.com/badjeff`
- [ ] project name = `zmk-pmw3610-driver`

**Файл charybdis_right.overlay:**
- [ ] Есть `#include <zephyr/dt-bindings/input/input-event-codes.h>`
- [ ] `compatible = "pixart,pmw3610-alt"`
- [ ] Добавлены `cpi`, `evt-type`, `x-input-code`, `y-input-code`
- [ ] Определён `zip_scroll_scaler` ПЕРЕД `trackball_listener`
- [ ] В `trackball_listener` настроены все нужные слои
- [ ] Номера слоёв совпадают с keymap

**Файл charybdis_right.conf:**
- [ ] `CONFIG_PMW3610_ALT=y`
- [ ] НЕТ `CONFIG_ZMK_POINTING`
- [ ] НЕТ старых `CONFIG_PMW3610_*` параметров
- [ ] Добавлены минимальные настройки badjeff

---

## 8. УСТРАНЕНИЕ ТИПОВЫХ ОШИБОК

### ❌ ОШИБКА #1: undefined node label 'zip_scroll_scaler'

**Сообщение:**
```
devicetree error: /trackball_listener/base_layer: undefined node label 'zip_scroll_scaler'
```

**Причина:** Определение `zip_scroll_scaler` отсутствует или идёт ПОСЛЕ использования

**Решение:**
```devicetree
/ {
  /* СНАЧАЛА определяем */
  zip_scroll_scaler: zip_scroll_scaler {
    compatible = "zmk,input-processor-scaler";
    #input-processor-cells = <2>;
    type = <INPUT_EV_REL>;
    codes = <INPUT_REL_WHEEL>, <INPUT_REL_HWHEEL>;
  };

  /* ПОТОМ используем */
  trackball_listener {
    base_layer {
      input-processors = <&zip_scroll_scaler 1 1>;
    };
  };
};
```

---

### ❌ ОШИБКА #2: attempt to assign the value 'y' to the undefined symbol ZMK_POINTING

**Сообщение:**
```
error: Aborting due to Kconfig warnings
/__w/.../charybdis_right.conf:3: warning: attempt to assign the value 'y' to the undefined symbol ZMK_POINTING
```

**Причина:** `CONFIG_ZMK_POINTING` не существует в feat/pointers-move-scroll

**Решение:** Удали строку `CONFIG_ZMK_POINTING=y` из .conf файла

---

### ❌ ОШИБКА #3: compatible 'pixart,pmw3610-alt' has unknown vendor prefix

**Сообщение:**
```
node '/soc/spi@40003000/trackball@0' compatible 'pixart,pmw3610-alt' has unknown vendor prefix 'pixart'
```

**Причина:** Это НЕ ошибка, а предупреждение. Можно игнорировать.

**Действие:** Проверь, что сборка продолжается дальше.

---

### ❌ ОШИБКА #4: Сборка зависает или падает без ошибок

**Возможные причины:**
1. GitHub Actions timeout
2. Кэш сборки повреждён
3. West dependencies не обновлены

**Решение:**
```yaml
# В .github/workflows/build.yml добавь:
- name: Clear west cache
  run: |
    rm -rf ~/.west
    rm -rf zephyr/.cache

- name: West update
  run: west update
```

---

### ❌ ОШИБКА #5: Прошивка собралась, но трекбол не работает

**Возможные причины:**
1. Неправильная ориентация/инверсия осей
2. Неправильный CPI
3. GPIO пины не совпадают с железом

**Диагностика:**

```devicetree
// Добавь в overlay для отладки
trackball: trackball@0 {
    cpi = <600>;
    
    // Попробуй разные комбинации:
    invert-x;
    // invert-y;
    // swap-xy;
};
```

**Проверь GPIO пины:**
```devicetree
irq-gpios = <&gpio0 6 (GPIO_ACTIVE_LOW | GPIO_PULL_UP)>;
cs-gpios = <&gpio0 20 GPIO_ACTIVE_LOW>;
```

---

## 9. ТЕСТИРОВАНИЕ ПРОШИВКИ

### ✅ ШАГ 9.1: Собери прошивку

```bash
# Локально (если есть ZMK окружение)
west build -p -b nice_nano_v2 -- -DSHIELD=charybdis_right

# Или через GitHub Actions
git add .
git commit -m "feat: add variable scroll speed on layers"
git push origin feature/scroll-speed-layers
```

### ✅ ШАГ 9.2: Прошей клавиатуру

1. Скачай UF2 файлы из GitHub Actions Artifacts
2. Переведи контроллер в режим bootloader
3. Скопируй `charybdis_right-nice_nano_v2-zmk.uf2` на диск контроллера
4. Дождись перезагрузки

### ✅ ШАГ 9.3: Базовая проверка

**Тест 1: Курсор двигается**
- Двигай трекбол → курсор должен перемещаться
- Если курсор не двигается или двигается "криво" → см. ошибку #5

**Тест 2: Скролл работает на базовом слое**
- На базовом слое (слой 0) крути трекбол
- Страница должна скроллиться с обычной скоростью

**Тест 3: Скролл замедляется на слое 1**
- Удержи клавишу активации слоя 1 (например, Calculator)
- Крути трекбол
- Скорость скролла должна уменьшиться в 3 раза (или как настроил)

**Тест 4: Скорость восстанавливается**
- Отпусти клавишу слоя 1
- Крути трекбол
- Скорость должна вернуться к обычной

### ✅ ШАГ 9.4: Измерь скорость

**Практический тест:**
```
1. Открой текстовый редактор с длинным документом
2. Засеки: сколько строк прокручивается за 1 полный оборот трекбола
3. Базовый слой: например, 15 строк
4. Слой 1 (медленный): должно быть ~5 строк (если делитель 3)
5. Соотношение: 15 / 5 = 3 ✅
```

### 📝 ЧЕКЛИСТ ТЕСТОВ:
- [ ] Курсор перемещается корректно
- [ ] Скролл работает на базовом слое
- [ ] Скролл замедляется на настроенном слое
- [ ] Скорость восстанавливается при возврате на базовый слой
- [ ] Соотношение скоростей соответствует настройкам

---

## 10. ТОНКАЯ НАСТРОЙКА

### 🎯 Подбор оптимальной скорости

**Метод итераций:**

```devicetree
// Итерация 1: Слишком медленно?
slow_scroll_layer {
  layers = <1>;
  input-processors = <&zip_scroll_scaler 1 5>;  // 20% скорости
};

// Итерация 2: Слишком быстро?
slow_scroll_layer {
  layers = <1>;
  input-processors = <&zip_scroll_scaler 1 2>;  // 50% скорости
};

// Итерация 3: Найден баланс
slow_scroll_layer {
  layers = <1>;
  input-processors = <&zip_scroll_scaler 1 3>;  // 33% скорости ✅
};
```

### 🎯 Настройка CPI (чувствительности курсора)

```devicetree
trackball: trackball@0 {
    // Низкий CPI = медленный точный курсор
    cpi = <400>;   // Для дизайна, CAD, точной работы
    
    // Средний CPI = баланс
    cpi = <600>;   // Универсальный вариант
    
    // Высокий CPI = быстрый курсор
    cpi = <1200>;  // Для больших экранов, быстрой навигации
};
```

### 🎯 Добавление ускоренного скролла

```devicetree
trackball_listener {
  // Базовый слой: обычная скорость
  base_layer {
    layers = <0>;
    input-processors = <&zip_scroll_scaler 1 1>;
  };

  // Слой 1: медленный скролл
  slow_scroll_layer {
    layers = <1>;
    input-processors = <&zip_scroll_scaler 1 3>;
  };

  // Слой 2: БЫСТРЫЙ скролл ✨
  fast_scroll_layer {
    layers = <2>;
    input-processors = <&zip_scroll_scaler 3 1>;  // В 3 раза быстрее!
  };

  // Слой 3: обычная скорость
  snipe_layer {
    layers = <3>;
    input-processors = <&zip_scroll_scaler 1 1>;
  };
};
```

**Применение:** Быстрая прокрутка длинных документов, логов, кода.

### 🎯 Настройка ориентации трекбола

**Если курсор двигается "неправильно":**

```devicetree
trackball: trackball@0 {
    cpi = <600>;
    
    // Попробуй разные комбинации:
    invert-x;        // Инверсия по X
    // invert-y;     // Инверсия по Y
    // swap-xy;      // Поменять оси местами
};
```

**Систематический подбор:**

| Проблема | Решение |
|----------|---------||
| Влево/вправо перепутаны | `invert-x;` |
| Вверх/вниз перепутаны | `invert-y;` |
| X и Y поменяны местами | `swap-xy;` |
| Диагональное движение | `swap-xy;` + `invert-x;` или `invert-y;` |

---

## 📋 ФИНАЛЬНЫЙ ЧЕКЛИСТ

### Перед коммитом:
- [ ] west.yml переключён на badjeff
- [ ] overlay: compatible = "pixart,pmw3610-alt"
- [ ] overlay: добавлены evt-type, x/y-input-code
- [ ] overlay: определён zip_scroll_scaler
- [ ] overlay: настроен trackball_listener
- [ ] conf: CONFIG_PMW3610_ALT=y
- [ ] conf: НЕТ CONFIG_ZMK_POINTING
- [ ] conf: удалены старые CONFIG_PMW3610_*
- [ ] Номера слоёв совпадают с keymap
- [ ] Файлы сохранены и закоммичены

### После сборки:
- [ ] Прошивка собралась без ошибок
- [ ] Курсор работает корректно
- [ ] Базовый скролл функционирует
- [ ] Медленный скролл активируется на нужном слое
- [ ] Скорость возвращается при смене слоя
- [ ] Соотношение скоростей соответствует настройкам

### Документация:
- [ ] Обновлён README с описанием настроек
- [ ] Указаны номера слоёв и их назначение
- [ ] Описана формула zip_scroll_scaler
- [ ] Добавлены примеры настройки

---

## 🎓 ВЫВОДЫ И РЕКОМЕНДАЦИИ

### ✅ Что работает идеально:
1. **badjeff/zmk-pmw3610-driver** — надёжный выбор для Input Processors
2. **Формула `1/N`** для замедления — интуитивно понятна
3. **Device Tree конфигурация** — чище и проще, чем Kconfig

### ⚠️ На что обратить внимание:
1. **Миграция драйвера** — требует полной переконфигурации
2. **Ориентация трекбола** — может потребовать ручной настройки
3. **Энергопотребление** — badjeff менее оптимизирован по умолчанию

### 🚀 Дальнейшее развитие:
1. Добавь **ускоренный скролл** для быстрой навигации
2. Настрой **разный CPI** для разных задач
3. Экспериментируй с **точной калибровкой** скоростей под свои привычки

---

**🎯 Этот опыт применим к любой ZMK прошивке с трекболом PMW3610!**
