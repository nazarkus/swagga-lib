# Swagga Lib · Legendary Edition

UI-библиотека для Roblox. Переписана из статичного G2L-дампа в
полноценную code-first библиотеку: элементы создаются вызовами,
а не клонированием заготовок.

```
✓ 504 автотеста · 54 набора · 0 провалов
✓ 245/245 публичных методов покрыто тестами
✓ 23 модуля · 12 000 строк · 0 синтаксических ошибок
```

**Два готовых файла:**

| Файл | Для чего |
|------|----------|
| `release/swagga.lua` | Обычное использование — чистая библиотека |
| `release/swagga_selftest.lua` | То же + самопроверка: запустил и сразу видно, всё ли работает |

Оба собираются автоматически при каждом `./verify.sh` или `python3 build.py`.

```lua
local Swagga = loadstring(game:HttpGet("ССЫЛКА/swagga.lua"))()

local Window = Swagga:CreateWindow({ Name = "Моё меню" })
local Tab    = Window:CreateTab({ Name = "Аимбот", Icon = "crosshair" })

Tab:CreateToggle({
    Name = "Включить",
    Keybind = Enum.KeyCode.E,
    Callback = function(state) print(state) end,
})
```

---

## Содержание

| | |
|---|---|
| [Установка](#установка) | [Элементы](#элементы) |
| [Окно](#окно) | [Бинды](#бинды) |
| [Вкладки](#вкладки) | [Настройки](#настройки) |
| [Темы](#темы) | [Анимации](#анимации) |
| [Сохранение](#сохранение) | [Уведомления](#уведомления) |
| [Полный API](#полный-справочник-api) | [Разработка](#разработка) |

---

## Установка

**Вариант 1 — через loadstring** (обычный для экзекуторов):

```lua
local Swagga = loadstring(game:HttpGet("ССЫЛКА_НА_swagga.lua"))()
```

**Вариант 2 — как ModuleScript** в Roblox Studio:

```lua
local Swagga = require(path.to.SwaggaLib)
```

Библиотека сама определяет окружение и подстраивается:

| Возможность | Есть | Нет |
|-------------|------|-----|
| `gethui()` | GUI прячется туда | CoreGui → PlayerGui |
| `writefile` | Конфиги на диск | Хранение в памяти |
| `syn.protect_gui` | Защита от детекта | Пропускается |
| `UIShadow` | Нативные тени | 9-slice картинки |
| Раздельные углы | Нативно | Общий радиус |

---

## Окно

```lua
local Window = Swagga:CreateWindow({
    Name = "Swagga Lib",
    Subtitle = "Build 2.0.0",

    Theme = "Midnight",       -- Midnight · Obsidian · Crimson · Aurora · Violet · Frost
    Accent = Color3.fromRGB(0, 167, 126),
    Size = UDim2.fromOffset(780, 620),
    MinSize = Vector2.new(560, 400),

    ToggleKey = Enum.KeyCode.RightShift,
    SidebarCollapsed = false,

    ConfigFolder = "SwaggaLib",
    ConfigSaving = true,
    AutoLoad = true,

    AnimationSpeed = 1,       -- 0.25 … 3
    ReducedMotion = false,
    Settings = true,          -- встроенный оверлей настроек
    Welcome = true,
})
```

### Методы окна

| Метод | Описание |
|-------|----------|
| `Window:CreateTab(config)` | Создать вкладку |
| `Window:SelectTab(tab)` | Выбрать: объект, индекс или имя |
| `Window:ToggleSidebar()` | Свернуть / развернуть сайдбар |
| `Window:ToggleMorePanel(force?)` | Панель «…» |
| `Window:Minimize()` · `:Restore()` · `:Toggle()` | Видимость |
| `Window:Close()` | Закрыть с анимацией |
| `Window:SetTitle(text)` · `:SetSubtitle(text)` | Заголовки |
| `Window:AddMoreItem(text, icon, fn)` | Свой пункт в меню «…» |

У окна продублированы методы библиотеки — удобно, если не хочется
хранить ссылку на `Swagga` в переменной:

```lua
Window:Notify{...}    Window:Get(flag)     Window:Set(flag, v)
Window:SaveConfig(n)  Window:LoadConfig(n) Window:Search(q)
Window:SetTheme(n)    Window:SetAccent(c)
```

---

## Вкладки

```lua
local Tab = Window:CreateTab({
    Name = "Аимбот",
    Icon = "crosshair",   -- имя Lucide, число assetId или "rbxassetid://…"
    Order = 1,
})
```

Сайдбар сворачивается в режим «только иконки» — при наведении
появляется тултип с названием. Активная вкладка отмечена скользящим
индикатором, который переезжает пружиной с эффектом «резинки».

### Иконки

```lua
Icon = "crosshair"                 -- Lucide (~1500 иконок)
Icon = 10723345094                 -- числовой assetId
Icon = "rbxassetid://10723345094"  -- готовый URI
```

Lucide подгружается лениво — если иконки по имени не используются,
сетевого запроса не будет. При неудаче работает встроенный набор
из ~90 иконок.

---

## Элементы

### Переключатель

```lua
local toggle = Tab:CreateToggle({
    Name = "Включить аимбот",
    Description = "ПКМ — назначить бинд",
    Icon = "target",
    Default = false,
    Keybind = Enum.KeyCode.E,
    KeybindMode = "Hold",        -- Toggle · Hold · Always
    Flag = "aimEnabled",
    Callback = function(state) end,
})
```

### Слайдер

```lua
Tab:CreateSlider({
    Name = "Плавность",
    Min = 0, Max = 100, Default = 35,
    -- либо Range = { 0, 100 }
    Decimals = 1,
    Increment = 5,       -- шаг
    Suffix = "%",
    Prefix = "~",
    Callback = function(value) end,
})
```

Заливка и ручка двигаются пружиной — при быстром перетаскивании
картинка не отстаёт ступеньками. Клик по числу превращает его
в поле ввода.

### Выпадающий список

```lua
-- Одиночный выбор
Tab:CreateDropdown({
    Name = "Точка прицеливания",
    Options = { "Head", "Torso", "HumanoidRootPart" },
    Default = "Head",
    Callback = function(part) end,
})

-- Множественный — включается на уровне функции
Tab:CreateDropdown({
    Name = "Проверки цели",
    Options = { "Видимость", "Живой", "Не команда" },
    MultiSelect = true,
    Default = { "Видимость", "Живой" },
    MaxSelections = 5,       -- необязательно
    Callback = function(list) end,
})
```

Список раскрывается **вниз и физически раздвигает элементы под собой**:
меняется высота самой карточки, а `UIListLayout` двигает соседей.
Опции появляются каскадом. Поиск включается автоматически от 8 опций
и работает с кириллицей.

Методы: `:Set(v)` · `:Get()` · `:Refresh(opts, keepSelection?)` ·
`:Add(opt)` · `:Remove(opt)` · `:Toggle(force?)`

### Палитра цвета

```lua
Tab:CreateColorPicker({
    Name = "Цвет ESP",
    Default = Color3.fromRGB(0, 167, 126),
    Alpha = true,        -- ползунок прозрачности
    Rainbow = true,      -- кнопка радужного режима
    Callback = function(color, alpha) end,
})
```

Внутри: HSV-квадрат (три слоя — оттенок, белый, чёрный), полоса
оттенка, альфа с шахматной подложкой, поля HEX и RGB с валидацией,
12 приглушённых пресетов, режим Rainbow. Раскрывается такой же
push-анимацией, как дропдаун.

Методы: `:Set(color)` · `:Get()` · `:SetWithAlpha(c, a)` ·
`:GetAlpha()` · `:SetRainbow(bool)` · `:Toggle(force?)`

### Бинд

```lua
Tab:CreateKeybind({
    Name = "Паника",
    Default = Enum.KeyCode.End,
    Mode = "Toggle",        -- Toggle · Hold · Always
    HoldThreshold = 0.2,
    Callback = function(state) end,
    OnChanged = function(newKey) end,
})
```

### Поле ввода

```lua
Tab:CreateInput({
    Name = "Дистанция",
    Numeric = true,      -- только числа
    Min = 1, Max = 5000, Default = 1000,
    Step = 50,           -- шаг колеса мыши и стрелок
    Decimals = 0,
    MaxLength = 20,      -- считается в символах, не байтах
    Placeholder = "Введите значение",
    Callback = function(value) end,
})
```

### Кнопка

```lua
Tab:CreateButton({
    Name = "Выполнить",
    Icon = "zap",
    Style = "primary",   -- default · primary · danger
    Callback = function() end,
})
```

При нажатии — ripple-волна из точки клика.

### Остальные

```lua
Tab:CreateSection("Заголовок группы")
Tab:CreateLabel({ Name = "Текст", Type = "success" })  -- success/warning/danger/info
Tab:CreateParagraph({ Name = "Заголовок", Content = "Многострочный текст" })
Tab:CreateDivider()
Tab:CreateStat({ Name = "FPS", Default = 60, Suffix = " к/с" })
```

### Общий API элемента

```lua
element:Set(value, skipCallback?)   -- задать значение
element:Get()                       -- прочитать
element:OnChanged(function(v) end)  -- подписка
element:SetEnabled(false)           -- заблокировать
element:SetVisible(false)           -- скрыть
element:SetName("Новое имя")
element:SetDescription("Описание")
element:SetKeybind(Enum.KeyCode.G, "Hold")
element:SetKeybindMode("Always")
element:MoveToTop() / :MoveToBottom() / :MoveTo(index)
element:Destroy()
```

### Алиасы

`Add*` работает как `Create*` (`AddToggle`, `AddSlider`, …).
Также есть `CreateColourPicker` и `CreateTextbox`.

---

## Бинды

Два способа назначить клавишу.

**1. Правой кнопкой мыши по элементу** — открывается контекстное меню:

- Назначить / изменить бинд
- Режим: Toggle · Hold · Always
- Убрать бинд
- Копировать значение
- Сбросить к значению по умолчанию

При записи экран затемняется, появляется карточка «Нажмите клавишу»
с пульсирующей рамкой. `Esc` — отмена, `Backspace` — убрать бинд.
Поддерживаются клавиши и кнопки мыши (ЛКМ/ПКМ/СКМ).

**2. Отдельный элемент** `Tab:CreateKeybind{...}`.

### Режимы

| Режим | Поведение |
|-------|-----------|
| `Toggle` | Нажатие переключает состояние |
| `Hold` | Активно, пока клавиша зажата (с порогом от случайного тапа) |
| `Always` | Срабатывает каждый кадр при зажатой клавише |

Все бинды обрабатывает **один** слушатель `UserInputService`, а не
соединение на каждый элемент — при сотне элементов это заметно
экономит ресурсы. Ввод в текстовые поля игнорируется.

```lua
Swagga.Binds:GetAll()        -- список всех назначенных
Swagga.Binds:IsHeld(key)     -- зажата ли клавиша
Swagga.Binds:SetEnabled(b)   -- временно отключить все бинды
Swagga.Binds:ClearAll()      -- снять все
```

---

## Настройки

Настройки открываются **оверлеем поверх окна** — так же, как
`Background.MoreTabs` в оригинальном Swagga Lib. Это не вкладка
сайдбара.

Открытие: кнопка **«…»** в топбаре → «Настройки». Закрытие: крестик
или `Esc`.

Внутри — горизонтальная навигация со скользящим подчёркиванием:

| Раздел | Содержимое |
|--------|-----------|
| **Тема** | Пресет, 6 быстрых акцентов, свой HEX |
| **Интерфейс** | Скорость анимаций, упрощённые анимации, сайдбар, сброс окна |
| **Клавиши** | Клавиша меню, список биндов, сброс всех |
| **Конфиги** | Сохранить / загрузить / удалить / список |
| **О библиотеке** | Версия, среда, отчёт, закрыть меню |

### Свои разделы

```lua
local S = Swagga.Settings
local sec = S:AddSection("Моё", "star")

S:AddHeading(sec, "Группа")
S:AddToggle(sec,  { Name = "Тумблер", Default = false, Callback = fn })
S:AddSlider(sec,  { Name = "Ползунок", Min = 0, Max = 100, Default = 50, Callback = fn })
S:AddChoice(sec,  { Name = "Выбор", Options = { "A", "B" }, Default = "A", Callback = fn })
S:AddInput(sec,   { Name = "Поле", Default = "текст", Callback = fn })
S:AddButton(sec,  { Name = "Кнопка", Icon = "zap", Callback = fn })
```

Каждый handle умеет `:Get()` и `:Set(value)`.

---

## Темы

Шесть встроенных: `Midnight` (по умолчанию — палитра оригинала),
`Obsidian`, `Crimson`, `Aurora`, `Violet`, `Frost` (светлая).

```lua
Swagga:SetTheme("Violet")
Swagga:SetAccent(Color3.fromRGB(255, 100, 0))
```

Смена происходит в рантайме: все зарегистрированные объекты плавно
перекрашиваются твином. Элементы привязываются к **токенам**
(`Accent`, `Element`, `TextPrimary`), а не к сырым цветам, поэтому
любой элемент корректно выглядит в любой теме.

Палитра приглушённая — без кислотного неона. Есть тест, который
не даёт вернуть чистые `#FF0000`-подобные цвета.

### Своя тема

```lua
Swagga.Theme.apply({
    Name = "Моя",
    Background = Color3.fromRGB(20, 20, 24),
    Surface    = Color3.fromRGB(15, 15, 18),
    Element    = Color3.fromRGB(26, 26, 32),
    Accent     = Color3.fromRGB(120, 200, 255),
    -- недостающие токены берутся из Midnight
})
```

### Токены

`Background` · `Surface` · `Element` · `ElementHover` · `ElementActive` ·
`Divider` · `Stroke` · `TextPrimary` · `TextSecondary` · `TextMuted` ·
`Accent` · `AccentHover` · `AccentMuted` · `AccentSoft` ·
`Danger` · `Warning` · `Success` · `Info` · `GlowA` · `GlowB` · `Shadow`

---

## Анимации

```lua
Swagga:SetAnimationSpeed(0.6)          -- 0.25 (медленно) … 3 (быстро)
Swagga.Motion.SetReducedMotion(true)   -- мгновенное применение
```

Оба параметра есть в настройках.

**Пружины вместо твинов** там, где цель меняется на лету: слайдер под
курсором, высота дропдауна при фильтрации, индикатор вкладки, ширина
сайдбара. Реализована аналитически решённая критически демпфированная
пружина — устойчива при лагах и не дёргается при смене цели на полпути.

| Пресет | Время | Применение |
|--------|-------|-----------|
| `Micro` | 0.12 s | ховер иконки |
| `Fast` | 0.18 s | ховер элемента |
| `Normal` | 0.28 s | смена состояния |
| `Slow` | 0.42 s | раскрытие дропдауна |
| `Slower` | 0.55 s | открытие окна, сайдбар |
| `Lazy` | 0.75 s | фоновые градиенты |

---

## Сохранение

Значения сохраняются автоматически с задержкой 0.6 с — иначе
перетаскивание слайдера писало бы файл сотни раз в секунду.

```lua
Swagga:SaveConfig("мойпресет")
Swagga:LoadConfig("мойпресет")
Swagga.Save:ListConfigs()
Swagga.Save:DeleteConfig("мойпресет")
```

Файлы: `<ConfigFolder>/configs/<имя>.json`,
настройки библиотеки — `<ConfigFolder>/settings.json`.

Корректно переживают сериализацию: `Color3` (в HEX), `Enum` (по имени),
таблицы мультивыбора, бинды с режимами и — отдельно проверено —
значение `false`.

Отключить сохранение элемента: `ForgetState = true`.

### Флаги

```lua
Tab:CreateToggle({ Name = "Аим", Flag = "aimEnabled" })

Swagga:Get("aimEnabled")        -- корректно вернёт false, а не nil
Swagga:Set("aimEnabled", true)
```

Если `Flag` не задан, он выводится из имени. Для русских названий
используется UTF-8-безопасный алгоритм: обычный `%w` кириллицу
отбрасывает, из-за чего все русские элементы получили бы одинаковый
ключ и перетирали друг другу настройки.

---

## Уведомления

```lua
Swagga:Notify({
    Title = "Готово",
    Content = "Настройки применены",
    Icon = "check",
    Type = "success",       -- success · warning · danger · info
    Duration = 4,
    Actions = {
        { Text = "Да",  Callback = function() end },
        { Text = "Нет", Callback = function() end },
    },
})
```

Тосты живут в отдельной `ScreenGui`, поэтому видны даже когда меню
свёрнуто. Наведение мыши ставит отсчёт на паузу. Возвращаемый handle
умеет `:Close()`, `:SetTitle()`, `:SetContent()`.

---

## Полный справочник API

### Библиотека

```lua
Swagga:CreateWindow(config)      -- создать окно
Swagga:Notify(config)            -- уведомление
Swagga:Get(flag) / :Set(flag, v) -- значения по флагу
Swagga:SetTheme(name)            -- сменить тему
Swagga:SetAccent(color)          -- сменить акцент
Swagga:SetAnimationSpeed(mult)   -- скорость анимаций
Swagga:SaveConfig(name) / :LoadConfig(name)
Swagga:GetElements()             -- все элементы
Swagga:Search(query)             -- поиск (кириллица поддерживается)
Swagga:Destroy()                 -- полная выгрузка (алиас :Unload())
```

### Подсистемы

```lua
Swagga.Theme    -- темы: apply, setAccent, get, getPresetNames, serialize
Swagga.Motion   -- анимации: tween, spring, drive, stagger, SetSpeed
Swagga.Icons    -- иконки: resolve, apply, exists, setLucideEnabled
Swagga.Util     -- утилиты: create, hex, toHex, upper, lower, slug, janitor
Swagga.Compat   -- среда: IsExecutor, HasUIShadow, GetGuiParent, Report
Swagga.Binds    -- бинды: GetAll, IsHeld, SetEnabled, ClearAll
Swagga.Save     -- сохранение: ListConfigs, DeleteConfig, Collect, Apply
Swagga.Settings -- оверлей настроек: Open, Close, Toggle, AddSection
```

### Горячие клавиши

| Клавиша | Действие |
|---------|----------|
| `RightShift` (настраивается) | Показать / скрыть меню |
| `ПКМ` по элементу | Меню бинда и действий |
| `Esc` | Отмена записи бинда · закрыть настройки |
| `Backspace` | Убрать бинд (в режиме записи) |

---

## Разработка

```
swagga/
├── src/                    исходники (23 модуля)
│   ├── init.luau           точка входа, публичный API
│   ├── Compat.luau         детект окружения и возможностей движка
│   ├── Motion.luau         анимации: твины, пружины, стаггер
│   ├── Theme.luau          темы и живая перекраска
│   ├── Window.luau         окно, сайдбар, драг, ресайз
│   ├── SettingsOverlay.luau оверлей настроек (как MoreTabs)
│   ├── Element.luau        базовый класс + ПКМ-меню
│   └── elements/           конкретные элементы
├── tests/
│   ├── harness.luau        тестовый фреймворк
│   ├── mock.luau           мок Roblox API
│   ├── suite_core.luau     окно, вкладки, сайдбар, настройки
│   ├── suite_elements.luau все 12 типов элементов
│   ├── suite_systems.luau  темы, анимации, сохранение, поиск
│   ├── suite_stress.luau   спам-клики, гонки, утечки
│   ├── suite_visual.luau   верность оригиналу
│   ├── suite_regression.luau 23 теста на исправленные баги
│   └── suite_api.luau      покрытие оставшегося API
├── release/
│   ├── swagga.lua          готовая библиотека (для использования)
│   └── swagga_selftest.lua она же + 85 самопроверок (для отладки)
├── example.lua             демо-меню
├── selftest.lua            встроенный самотест (79 проверок)
└── preview.html            визуальное превью в браузере
```

### Файл с самопроверкой

`release/swagga_selftest.lua` — та же библиотека, но в конце прогоняет
**85 проверок** прямо в игре: окно, вкладки, все элементы, бинды, темы,
конфиги, три панели, шторка-анимация, мягкость линий.

Результат виден тремя способами:

- в консоли — построчно `[+]` / `[-]`;
- уведомлением в самом меню — сводка «пройдено / провалено»;
- кнопкой **«Скопировать отчёт самотеста»** во вкладке меню.

Удобно, когда нужно быстро убедиться, что на конкретном экзекуторе
ничего не отвалилось. Для обычной работы бери `swagga.lua`.

### Команды

```bash
./verify.sh          # полная проверка: сборка → статика → тесты → пример → самотест
./runtests.sh        # только тесты (504 шт.)
./runtests.sh --quick # без пересборки
./check.sh           # статический анализ
python3 build.py 2.0.0  # собрать release/swagga.lua
```

`verify.sh` возвращает ненулевой код при любой проблеме — годится для CI.

### Как устроено тестирование

Тесты исполняют **собранный бандл** в мок-окружении Roblox
(`tests/mock.luau` — Instance, Enum, TweenService, ввод, файловая
система). То есть проверяется ровно тот код, который отправляется
в игру, а не исходники.

Каждый набор получает **свежий экземпляр** библиотеки — состояние
не перетекает между тестами.

```
504 теста · 54 набора · 245/245 методов покрыто
```

---

## Найденные и исправленные ошибки

Тесты выявили **23 настоящих бага** до того, как они попали в релиз.
На каждый есть регрессионный тест в `suite_regression.luau`.

| # | Баг | Последствие |
|---|-----|-------------|
| 1 | Пружина: в формуле критического демпфирования дважды учитывалась скорость | Значение улетало до 1.83 при цели 1.0 |
| 2 | `%w` отбрасывает кириллицу в `slug()` | Все русские элементы делили один ключ конфига |
| 3 | `a and b or c` на булевых | `Swagga:Get()` возвращал `nil` вместо `false` |
| 4 | `ContextMenu.Open` — поле затирало метод | ПКМ падало: «attempt to call a boolean value» |
| 5 | `deepCopy` использовал `type()` вместо `typeof()` | `Color3` превращался в обычную таблицу |
| 6 | Вкладка настроек создавалась первой | Забирала фокус у пользовательской |
| 7 | Таймер тостов в блокирующем `while task.wait` | Висящий поток |
| 8 | `CreateWindow` настраивал `library`, а возвращал `window` | 10 из 16 вызовов публичного API падали |
| 9 | `false` писался в JSON как `null` | Выключенный тумблер «залипал» включённым |
| 10 | Отложенный автосейв не отменялся при загрузке | Перетирал только что загруженный конфиг |
| 11 | `while task.wait(2)` в примере | Поток жил вечно, держал мёртвые ссылки |
| 12 | Градиент насыщенности висел на самом квадрате | Цвета палитры выглядели выцветшими |
| 13 | Дропдаун закрывал только дропдауны | Палитра и список висели открытыми одновременно |
| 14 | Настройки были вкладкой сайдбара | Не соответствовало оригиналу |
| 15 | `SelectTab` не сбрасывал строку при промахе | Падало на `tab:_setActive()` |
| 16 | RGB-парсер не понимал минус | `"999,-50,300"` не разбирался |
| 17 | Автосейв просыпался после `Destroy` | Падал на `CurrentConfig == nil` |
| 18 | `Theme.bind` красил тумблер без учёта состояния | Включённый терял акцент при смене темы |
| 19 | То же для вкладок | Активная вкладка теряла подсветку |
| 20 | Подчёркивание считалось через `AbsolutePosition` | Залипало в углу — layout ещё не посчитан |
| 21 | Handle-и настроек требовали `Set(_, v)` | Неудобная сигнатура |
| 22 | `MaxLength` резал по байтам | Рвал кириллицу пополам |
| 23 | Кислотные неоновые цвета | Выглядело дёшево на тёмном фоне |

---

## Что нового относительно исходника

| Было | Стало |
|------|-------|
| G2L-дамп, 434 инстанса, ничего нельзя создать в рантайме | Code-first: `Tab:CreateToggle{...}` |
| Пустой контейнер `Tabs`, 2 кнопки-заготовки | Полноценные вкладки со скроллом и иконками |
| Нет ни одного рабочего элемента | 12 типов элементов |
| `require(ReplicatedStorage.Anims)` — падало в экзекуторе | Автономная сборка |
| Glow только картинками 9-slice | Нативный `UIShadow` + автооткат |
| Тема захардкожена | 6 тем + смена акцента в рантайме |
| Нет сохранения | Автосейв + слоты конфигов |
| Нет тестов | 463 автотеста |

**Использованы возможности движка 2026 года:**

- **`UIShadow`** (полный релиз — июнь 2026) — нативные тени и свечение,
  быстрее 9-slice картинок, корректно повторяют скругление углов.
- **Индивидуальные радиусы углов** `UICorner.TopLeftRadius` и др. —
  топбар скруглён только сверху, сайдбар только слева-снизу.

Обе фичи с автоматическим откатом через `Compat.luau`, поэтому
библиотека одинаково работает и на свежем клиенте, и на старом.

---

## Лицензия

Свободное использование. Оригинальный дизайн — **opiumbyte**.
