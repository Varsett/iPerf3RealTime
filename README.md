# iPerf3 Real-Time Monitor — Руководство пользователя
### (English manual below)


PowerShell-инструмент с GUI для запуска тестов iPerf3 через ADB
и отображения Bitrate, Jitter и Packet Loss в реальном времени
на трёх отдельных графиках. Поддерживает режимы UDP и TCP.

![]()

---

## Требования

- Windows 10 или новее
- PowerShell 5.1 (встроен в Windows)
- `adb.exe` по пути из `-ToolsPath` или в системном PATH
- Бинарник iPerf3 на Android-устройстве: `/data/local/tmp/iperf3.18`
- Сервер iPerf3 запущен локально: `iperf3.exe -s`

---

## Быстрый старт

```cmd
set host=10.0.0.30
powershell -ExecutionPolicy Bypass -File "iperf3_realtime.ps1" ^
  -ServerIP %host% -Protocol -u -Direction -R -Duration 300 -Interval 0.1 -Bitrate 100
```

---

## Режимы TCP и UDP

| | UDP (`-Protocol -u`) | TCP (по умолчанию, без `-Protocol`) |
|---|---|---|
| График Bitrate | Активен | Активен |
| График Jitter | Активен | Затемнён, "N/A for TCP" |
| График Loss | Активен | Затемнён, "N/A for TCP" |
| LIVE STATS Loss/Jitter | Числовые значения | "N/A" |
| Заголовок окна | Bitrate, Loss, Jitter | Только Bitrate |
| Цветовая индикация заголовка | Активна | Не применяется |

Режим TCP полезен для стресс-теста чистой пропускной способности,
где iPerf3 не выдаёт джиттер и потери пакетов.

---

## Лаунчер (CMD-файл)

Все параметры необязательны кроме `-ServerIP`. Пустые переменные CMD
безопасно игнорируются — используются встроенные значения iPerf3.

```cmd
@echo off
set host=10.0.0.30
set toolspath=d:\Tools\ADB
set profilename=Stress Test 300s
set protocol=-u
set direction=-R
set bitrate=100
set duration=300
set interval=0.1
set streams=1
set port=
set buflen=
set socketsize=
set logfile=d:\logs\test.csv
set savepath=d:\logs

set "AnalyzerScript=d:\tools\iPerf3VisualAnalyzer.v3.24.ps1"
set "AnalyzerArgs=-AView reverse"

powershell -ExecutionPolicy Bypass -File "iperf3_realtime.ps1" ^
  -ServerIP %host% ^
  -ToolsPath %toolspath% ^
  -ProfileName %profilename% ^
  -Port %port% ^
  -Protocol %protocol% ^
  -Direction %direction% ^
  -Bitrate %bitrate% ^
  -Duration %duration% ^
  -Interval %interval% ^
  -Buflen %buflen% ^
  -Socketsize %socketsize% ^
  -Streams %streams% ^
  -LogFile %logfile% ^
  -SavePath %savepath% ^
  -VisualAnalyzerScript "%AnalyzerScript%" ^
  -VisualAnalyzerArgs "%AnalyzerArgs%"
```

---

## Параметры

| Параметр | Описание | По умолчанию |
|---|---|---|
| `-ServerIP` | IP-адрес сервера iPerf3 | **Обязательный** |
| `-ToolsPath` | Папка с `adb.exe` | `adb` в системном PATH |
| `-ProfileName` | Имя теста в панели TEST INFO | — |
| `-Port` | Порт сервера iPerf3 | 5201 |
| `-Protocol` | пусто = TCP, `-u` = UDP | TCP |
| `-Direction` | `` = обычный, `-R` = реверс | обычный |
| `-Bitrate` | Целевой битрейт Мбит/с, `0` = без ограничений | 1 Мбит/с |
| `-Duration` | Длительность теста в секундах | 10 |
| `-Interval` | Интервал отчётов в секундах | 1 |
| `-Buflen` | Размер пакета/буфера в байтах (флаг iPerf `-l`) | 1460 |
| `-Socketsize` | Размер сокетного окна в МБ (флаг iPerf `-w`) | 1M |
| `-Streams` | Количество параллельных потоков | 1 |
| `-Extra` | Любые дополнительные флаги iPerf3 | — |
| `-LogFile` | Путь к файлу живого CSV-лога | нет |
| `-SavePath` | Папка для CSV и PNG | папка скрипта |
| `-LossWarnPct` | Потери для жёлтого заголовка (%, UDP) | 2.0 |
| `-LossCritPct` | Потери для красного заголовка (%, UDP) | 5.0 |
| `-ThreshLoss` | Линия порога потерь на графике, % (UDP) | 1.0 |
| `-ThreshJitter` | Линия порога джиттера на графике, мс (UDP) | 0.2 |
| `-ThreshBitrateRel` | Порог битрейта — % от целевого | 50 |
| `-ScrollWindowSec` | Ширина окна автопрокрутки в секундах | 10 |
| `-MaxPoints` | Максимум точек данных в памяти | 36000 |
| `-VisualAnalyzerScript` | Полный путь к скрипту анализатора `.ps1` | — |
| `-VisualAnalyzerArgs` | Дополнительные аргументы для анализатора | — |
| `-CpuDivisor` | Ручной делитель CPU% | авто |
| `-KillServerOnFinish` | `1` = убить локальный `iperf3.exe` и окно `IPERF_TEST` после теста | — |
| `-KillServerOnFinish` | `1` = убить `iperf3.exe` + окно `IPERF_TEST` при завершении | — |
| `-DoneFlagFile` | Файл-маркер при завершении/прерывании (экспериментально) | — |
| `-AutoExit` | Секунды до автозакрытия после завершения; `0` = сразу, пусто = никогда | — |

---

## Графики

Три отдельных графика на общей оси X (прошедшее время в секундах).
У каждого своя ось Y, сетка и линия порога.

### Bitrate (верхний)

| Линия | Цвет | Описание |
|---|---|---|
| Bitrate | Синяя сплошная | Текущий битрейт Мбит/с |
| Avg Bitrate | Зелёная пунктирная | Скользящее среднее (кнопка Show Avg) |
| Threshold | Фиолетовая пунктирная | Bitrate / 2 |

### Jitter (средний) — только UDP

| Линия | Цвет | Описание |
|---|---|---|
| Jitter | Жёлтая сплошная | Джиттер в мс |
| Threshold | Фиолетовая пунктирная | ThreshJitter мс (по умолчанию 0.2) |

В режиме TCP график затемнён, заголовок "Jitter (ms) - N/A for TCP".

### Packet Loss (нижний) — только UDP

| Линия | Цвет | Описание |
|---|---|---|
| Loss | Красная сплошная | Потери пакетов % |
| Threshold | Фиолетовая пунктирная | ThreshLoss % (по умолчанию 1.0) |

В режиме TCP график затемнён, заголовок "Loss (%) - N/A for TCP".

Все threshold-линии показывают надпись **"Threshold"** у правого края
и не влияют на ось X и автопрокрутку.

---

## Кнопки управления

| Кнопка | Описание |
|---|---|
| **Stop** | Немедленно завершает процесс ADB / iPerf3 |
| **Export CSV** | Сохраняет данные в CSV. **Недоступна во время теста** |
| **Save PNG** | Скриншот всего окна в PNG |
| **Visual Analyzer** | Запускает скрипт анализатора после завершения теста. Видна только при заданном `-VisualAnalyzerScript` |
| **Autoscroll** | Вкл (синяя): следует последним N секундам. Выкл (серая): вся история |
| **Show Avg** | Показать/скрыть зелёную линию среднего битрейта |
| **Legend** | Показать/скрыть легенду графика (скрыта по умолчанию) |
| **?** | Открывает встроенную справку |

Файлы сохраняются в `-SavePath` или рядом со скриптом.
Формат: `iperf3_YYYYMMDD_HHMMSS.csv / .png`

---

## Visual Analyzer

Кнопка **Visual Analyzer** появляется после завершения теста
если указан `-VisualAnalyzerScript`. Запускает:

```
powershell.exe -NoLogo -NoProfile -ExecutionPolicy Bypass
    -WindowStyle Hidden
    -File "<VisualAnalyzerScript>"
    -CsvPath "<LogFile>"
    [токены VisualAnalyzerArgs]
```

Путь к CSV-логу подставляется автоматически из `-LogFile`.
Окно анализатора открывается без консольного окна.

**Настройка в CMD:**
```cmd
set "AnalyzerScript=d:\tools\iPerf3VisualAnalyzer.v3.24.ps1"
set "AnalyzerArgs=-AView reverse"

powershell ... ^
  -LogFile %logfile% ^
  -VisualAnalyzerScript "%AnalyzerScript%" ^
  -VisualAnalyzerArgs "%AnalyzerArgs%"
```

**Важно:** не включайте в `-VisualAnalyzerArgs` флаги `-ExecutionPolicy`,
`-NoLogo`, `-NoProfile`, `-WindowStyle` — они добавляются автоматически.
Дублирование помешает запуску анализатора.

**Совет:** передавайте `-AllLogs` в `-VisualAnalyzerArgs` чтобы анализатор
показывал все лог-файлы в текущей директории, а не только лог текущего теста.

---

## Панель TEST INFO (правая колонка, верх)

| Цвет | Значение |
|---|---|
| Бирюзовый | Параметр задан вручную |
| Белый + (Default) | Используется значение iPerf3 по умолчанию |

| Поле | Описание |
|---|---|
| Profile | Имя теста из `-ProfileName` |
| Protocol | TCP или UDP |
| Direction | Direct или Reverse |
| Bitrate | Целевой битрейт или Unlimited |
| Duration | Длительность теста |
| Interval | Интервал отчётов |
| Streams | Количество потоков |
| Socket size | Размер сокетного окна |
| Buflen | Размер пакета/буфера |

---

## Панель LIVE STATS (правая колонка, низ)

Значения превысившие порог становятся **красными**.
В режиме TCP поля Loss и Jitter показывают "N/A".

| Поле | Цвет | Краснеет когда |
|---|---|---|
| Status | Зелёный | — |
| Min | Голубой | Min Bitrate ниже Bitrate/2 |
| Max | Голубой | — |
| Avg | Голубой | — |
| Avg Loss | Красный | — (N/A для TCP) |
| Max Loss | Красный | Превышает `-ThreshLoss` (N/A для TCP) |
| Avg Jitter | Жёлтый | — (N/A для TCP) |
| Max Jitter | Жёлтый | Превышает `-ThreshJitter` (N/A для TCP) |
| Points | Серый | — |
| iPerf3 CPU | Динамический | См. градации ниже |

---

## Загрузка CPU iPerf3

Опрашивает `iperf3.exe` каждые ~1 секунду через счётчики производительности WMI.

| Загрузка | Оценка | Цвет |
|---|---|---|
| > 30% | Неприемлемый | Красный |
| > 25% | Очень высокий | Красный |
| > 20% | Высокий | Красный |
| > 15% | Повышенный | Оранжевый |
| > 10% | Средний | Жёлтый |
| > 5% | Нормальный | Зелёный |
| > 1% | Хороший | Голубой |
| 0–1% | Отличный | Голубой |

**Внимание:** загрузка выше **20%** делает результаты недостоверными.
Держите ниже **15%** для достоверных измерений.

На системах с HyperThreading значение нормализуется автоматически.
Если показания отличаются от Диспетчера задач — используйте `-CpuDivisor N`
(например `-CpuDivisor 3` для 2-ядерного/4-поточного CPU).

---

## Цвет заголовка окна (только UDP)

| Цвет | Условие |
|---|---|
| Белый | Потери ниже `-LossWarnPct` (по умолчанию 2%) |
| Жёлтый | Потери на уровне `-LossWarnPct` или выше |
| Красный | Потери на уровне `-LossCritPct` или выше (по умолчанию 5%) |

В режиме TCP заголовок показывает только Bitrate без цветовой индикации.

---

## Автозавершение (KillServerOnFinish)

`-KillServerOnFinish 1` заставляет скрипт по завершении теста:
1. Остановить локальный процесс `iperf3.exe` (сервер, запущенный с `-s`)
2. Закрыть окно CMD с заголовком `IPERF_TEST*` (через `taskkill`)
3. Скрыть своё консольное окно (окно с графиками остаётся открытым)

Полезно когда сервер запускался из того же CMD-сценария:
```cmd
start "IPERF_TEST" cmd /c "iperf3.exe -s"
powershell ... -KillServerOnFinish 1
```

---

## Автоматизация серии тестов (AutoExit)

`-AutoExit N` автоматически закрывает окно графиков через N секунд после
завершения теста — процесс скрипта завершается, и CMD-сценарий может
перейти к следующему тесту без ручного вмешательства.

| Значение | Поведение |
|---|---|
| (пусто, по умолчанию) | Окно остаётся открытым до закрытия вручную |
| `0` | Окно закрывается сразу при завершении теста |
| `N` (секунды) | Окно закрывается через N секунд после завершения |

В сочетании с `-KillServerOnFinish 1` получается полностью автоматизированная
серия последовательных тестов:

```cmd
for %%P in (10 50 100 200) do (
    start "IPERF_TEST" cmd /c "iperf3.exe -s"
    powershell -ExecutionPolicy Bypass -File "iperf3_realtime.ps1" ^
      -ServerIP %host% -Protocol -u -Bitrate %%P -Duration 60 ^
      -ProfileName "Test_%%P_Mbps" ^
      -LogFile "d:\logs	est_%%P.csv" ^
      -KillServerOnFinish 1 ^
      -AutoExit 5
)
```

Каждый тест выполняется, показывает результат 5 секунд, затем автоматически
закрывается, и цикл переходит к следующему битрейту.

---

## Выходные файлы

| Файл | Когда | Столбцы |
|---|---|---|
| `iperf3_YYYYMMDD_HHMMSS.csv` | Кнопка Export CSV | Time, Bitrate, Loss, Jitter |
| `iperf3_YYYYMMDD_HHMMSS.png` | Кнопка Save PNG | Скриншот окна |
| Пользовательский лог | Живой во время теста | Тот же CSV-формат (`-LogFile`) |

---

## Объём данных

`-MaxPoints 36000` при интервале `-Interval 0.1` = до **1 часа** в памяти.
Статистика вычисляется через O(1) аккумуляторы — производительность
не зависит от длительности теста.

---

## Решение проблем

**ADB не запускается** — проверьте путь к `adb.exe` и выполните `adb devices`.

**Нет данных на графике (TCP)** — ось X показывает время; активен только
график Bitrate в режиме TCP — это ожидаемое поведение.

**Нет данных на графике (UDP)** — убедитесь что iPerf3 есть на устройстве,
сервер запущен, и установлен `-Protocol -u`.

**Export CSV недоступна** — нажмите Stop или дождитесь окончания теста.

**CPU показывает "not found"** — запустите `iperf3.exe -s` до старта скрипта.

**CPU отличается от Диспетчера задач** — используйте `-CpuDivisor N`.

**Visual Analyzer не запускается** — проверьте путь в `-VisualAnalyzerScript`.
Не включайте флаги запуска PowerShell в `-VisualAnalyzerArgs`.
---

---
# iPerf3 Real-Time Monitor — User Guide

A PowerShell GUI tool for running iPerf3 tests over ADB and visualizing
Bitrate, Jitter, and Packet Loss in real time on three separate charts.
Supports both UDP and TCP modes.

---

## Requirements

- Windows 10 or later
- PowerShell 5.1 (built into Windows)
- `adb.exe` accessible via `-ToolsPath` or on system PATH
- iPerf3 binary on the Android device: `/data/local/tmp/iperf3.18`
- iPerf3 server running locally: `iperf3.exe -s`

---

## Quick Start

```cmd
set host=10.0.0.30
powershell -ExecutionPolicy Bypass -File "iperf3_realtime.ps1" ^
  -ServerIP %host% -Protocol -u -Direction -R -Duration 300 -Interval 0.1 -Bitrate 100
```

---

## TCP vs UDP Mode

| | UDP (`-Protocol -u`) | TCP (default, no `-Protocol`) |
|---|---|---|
| Bitrate chart | Active | Active |
| Jitter chart | Active | Dimmed, shows "N/A for TCP" |
| Loss chart | Active | Dimmed, shows "N/A for TCP" |
| LIVE STATS Loss/Jitter | Numeric values | "N/A" |
| Title bar | Bitrate, Loss, Jitter | Bitrate only |
| Title color coding | Active | Not applicable |

TCP mode is useful for raw throughput stress testing where jitter and
packet loss are not reported by iPerf3.

---

## Launcher (CMD file)

All parameters are optional except `-ServerIP`. Empty CMD variables are safely
ignored — iPerf3 built-in defaults are used instead.

```cmd
@echo off
set host=10.0.0.30
set toolspath=d:\Tools\ADB
set profilename=Stress Test 300s
set protocol=-u
set direction=-R
set bitrate=100
set duration=300
set interval=0.1
set streams=1
set port=
set buflen=
set socketsize=
set logfile=d:\logs\test.csv
set savepath=d:\logs

set "AnalyzerScript=d:\tools\iPerf3VisualAnalyzer.v3.24.ps1"
set "AnalyzerArgs=-AView reverse"

powershell -ExecutionPolicy Bypass -File "iperf3_realtime.ps1" ^
  -ServerIP %host% ^
  -ToolsPath %toolspath% ^
  -ProfileName %profilename% ^
  -Port %port% ^
  -Protocol %protocol% ^
  -Direction %direction% ^
  -Bitrate %bitrate% ^
  -Duration %duration% ^
  -Interval %interval% ^
  -Buflen %buflen% ^
  -Socketsize %socketsize% ^
  -Streams %streams% ^
  -LogFile %logfile% ^
  -SavePath %savepath% ^
  -VisualAnalyzerScript "%AnalyzerScript%" ^
  -VisualAnalyzerArgs "%AnalyzerArgs%"
```

---

## Parameters

| Parameter | Description | Default |
|---|---|---|
| `-ServerIP` | iPerf3 server IP address | **Required** |
| `-ToolsPath` | Folder containing `adb.exe` | `adb` on system PATH |
| `-ProfileName` | Test name shown in TEST INFO panel | — |
| `-Port` | iPerf3 server port | 5201 |
| `-Protocol` | empty = TCP, `-u` = UDP | TCP |
| `-Direction` | `` = normal, `-R` = reverse | normal |
| `-Bitrate` | Target bitrate in Mbps, `0` = unlimited | 1 Mbps |
| `-Duration` | Test duration in seconds | 10 |
| `-Interval` | Reporting interval in seconds | 1 |
| `-Buflen` | Packet/buffer size in bytes (iPerf `-l` flag) | 1460 |
| `-Socketsize` | Socket window size in MB (iPerf `-w` flag) | 1M |
| `-Streams` | Number of parallel streams | 1 |
| `-Extra` | Any extra raw iPerf3 flags | — |
| `-LogFile` | Path to live CSV log file | none |
| `-SavePath` | Folder for CSV and PNG exports | script folder |
| `-LossWarnPct` | Packet loss % for yellow title warning (UDP) | 2.0 |
| `-LossCritPct` | Packet loss % for red title alert (UDP) | 5.0 |
| `-ThreshLoss` | Loss threshold line on chart, % (UDP) | 1.0 |
| `-ThreshJitter` | Jitter threshold line on chart, ms (UDP) | 0.2 |
| `-ThreshBitrateRel` | Bitrate threshold as % of target | 50 |
| `-ScrollWindowSec` | Autoscroll window width in seconds | 10 |
| `-MaxPoints` | Maximum data points kept in memory | 36000 |
| `-VisualAnalyzerScript` | Full path to the analyzer `.ps1` script | — |
| `-VisualAnalyzerArgs` | Extra arguments passed to the analyzer | — |
| `-CpuDivisor` | Manual CPU% divisor override | auto |
| `-KillServerOnFinish` | `1` = kill local `iperf3.exe` and `IPERF_TEST` window on finish | — |
| `-KillServerOnFinish` | `1` = kill `iperf3.exe` + `IPERF_TEST` window on finish | — |
| `-DoneFlagFile` | Marker file written on finish/abort (experimental) | — |
| `-AutoExit` | Seconds before auto-close after finish; `0` = immediately, empty = never | — |

---

## Charts

Three separate charts share the same X axis (elapsed time in seconds).
Each has its own Y axis, grid, and threshold line.

### Bitrate (top)

| Line | Color | Description |
|---|---|---|
| Bitrate | Blue solid | Current throughput in Mbps |
| Avg Bitrate | Green dashed | Rolling average (toggle with Show Avg) |
| Threshold | Purple dashed | Bitrate / 2 |

### Jitter (middle) — UDP only

| Line | Color | Description |
|---|---|---|
| Jitter | Yellow solid | Jitter in ms |
| Threshold | Purple dashed | ThreshJitter ms (default 0.2) |

In TCP mode this chart is dimmed and titled "Jitter (ms) - N/A for TCP".

### Packet Loss (bottom) — UDP only

| Line | Color | Description |
|---|---|---|
| Loss | Red solid | Packet loss % |
| Threshold | Purple dashed | ThreshLoss % (default 1.0) |

In TCP mode this chart is dimmed and titled "Loss (%) - N/A for TCP".

All threshold lines show a **"Threshold"** label at the right edge and do not
affect the X axis or autoscroll behavior.

---

## Controls

| Button | Description |
|---|---|
| **Stop** | Immediately terminates the ADB / iPerf3 process |
| **Export CSV** | Saves all data to CSV. **Disabled during test** |
| **Save PNG** | Screenshots the entire window to PNG |
| **Visual Analyzer** | Launches the analyzer script after test completes. Visible only when `-VisualAnalyzerScript` is set |
| **Autoscroll** | ON (blue): follows last N seconds. OFF (grey): full history |
| **Show Avg** | Toggle the green rolling average Bitrate line |
| **Legend** | Toggle the chart legend on/off (hidden by default) |
| **?** | Opens the built-in help window |

Files are saved to `-SavePath` (or next to the script).
Format: `iperf3_YYYYMMDD_HHMMSS.csv / .png`

---

## Visual Analyzer

The **Visual Analyzer** button appears after the test finishes if
`-VisualAnalyzerScript` was specified at launch. It runs:

```
powershell.exe -NoLogo -NoProfile -ExecutionPolicy Bypass
    -WindowStyle Hidden
    -File "<VisualAnalyzerScript>"
    -CsvPath "<LogFile>"
    [VisualAnalyzerArgs tokens]
```

The CSV log path is appended automatically from `-LogFile`.
The analyzer window opens without a console window.

**CMD setup:**
```cmd
set "AnalyzerScript=d:\tools\iPerf3VisualAnalyzer.v3.24.ps1"
set "AnalyzerArgs=-AView reverse"

powershell ... ^
  -LogFile %logfile% ^
  -VisualAnalyzerScript "%AnalyzerScript%" ^
  -VisualAnalyzerArgs "%AnalyzerArgs%"
```

**Note:** do not include `-ExecutionPolicy`, `-NoLogo`, `-NoProfile`, or
`-WindowStyle` in `-VisualAnalyzerArgs` — they are added automatically.
Duplicates will prevent the analyzer from launching.

**Tip:** pass `-AllLogs` in `-VisualAnalyzerArgs` to show all log files
in the current directory, not just the one from this test run.

---

## TEST INFO Panel (top right)

| Color | Meaning |
|---|---|
| Aqua | Parameter was set manually |
| White + (Default) | iPerf3 built-in default is used |

| Field | Description |
|---|---|
| Profile | Test name from `-ProfileName` |
| Protocol | TCP or UDP |
| Direction | Direct or Reverse |
| Bitrate | Target bitrate or Unlimited |
| Duration | Test duration |
| Interval | Reporting interval |
| Streams | Parallel stream count |
| Socket size | Socket window size |
| Buflen | Packet/buffer size |

---

## LIVE STATS Panel (bottom right)

Values turn **red** when they exceed their threshold.
In TCP mode, Loss and Jitter fields show "N/A".

| Field | Color | Turns red when |
|---|---|---|
| Status | Green | — |
| Min | Cyan | Min Bitrate drops below Bitrate/2 |
| Max | Cyan | — |
| Avg | Cyan | — |
| Avg Loss | Red | — (N/A for TCP) |
| Max Loss | Red | Exceeds `-ThreshLoss` (N/A for TCP) |
| Avg Jitter | Yellow | — (N/A for TCP) |
| Max Jitter | Yellow | Exceeds `-ThreshJitter` (N/A for TCP) |
| Points | Grey | — |
| iPerf3 CPU | Dynamic | See CPU grades below |

---

## iPerf3 CPU Load

Polls `iperf3.exe` CPU usage every ~1 second via WMI performance counters.

| Load | Grade | Color |
|---|---|---|
| > 30% | Unacceptable | Red |
| > 25% | Very High | Red |
| > 20% | High | Red |
| > 15% | Elevated | Orange |
| > 10% | Medium | Yellow |
| > 5% | Normal | Green |
| > 1% | Good | Cyan |
| 0–1% | Excellent | Cyan |

**Warning:** CPU above **20%** makes results unreliable — iPerf3 reduces
reported throughput and may show artificial packet loss. Keep below **15%**.

On HyperThreading systems the value is normalized automatically.
If readings differ from Task Manager, use `-CpuDivisor N`
(e.g. `-CpuDivisor 3` for a 2-core/4-thread CPU).

---

## Title Bar Color (UDP only)

| Color | Condition |
|---|---|
| White | Loss below `-LossWarnPct` (default 2%) |
| Yellow | Loss at or above `-LossWarnPct` |
| Red | Loss at or above `-LossCritPct` (default 5%) |

In TCP mode the title shows Bitrate only with no color coding.

---

## Automated Cleanup (KillServerOnFinish)

`-KillServerOnFinish 1` makes the script, on test completion:
1. Stop the local `iperf3.exe` process (the server started with `-s`)
2. Close the CMD window titled `IPERF_TEST*` (best-effort via `taskkill`)
3. Hide its own console window (the chart window stays open)

This is useful when the server was launched from the same CMD batch:
```cmd
start "IPERF_TEST" cmd /c "iperf3.exe -s"
powershell ... -KillServerOnFinish 1
```

---

## Batch Test Automation (AutoExit)

`-AutoExit N` automatically closes the chart window N seconds after the test
finishes, so the script process exits and a CMD batch can continue to the
next test without manual intervention.

| Value | Behavior |
|---|---|
| (empty, default) | Window stays open until closed manually |
| `0` | Window closes immediately when the test finishes |
| `N` (seconds) | Window closes N seconds after the test finishes |

Combine with `-KillServerOnFinish 1` for fully automated sequential testing:

```cmd
for %%P in (10 50 100 200) do (
    start "IPERF_TEST" cmd /c "iperf3.exe -s"
    powershell -ExecutionPolicy Bypass -File "iperf3_realtime.ps1" ^
      -ServerIP %host% -Protocol -u -Bitrate %%P -Duration 60 ^
      -ProfileName "Test_%%P_Mbps" ^
      -LogFile "d:\logs	est_%%P.csv" ^
      -KillServerOnFinish 1 ^
      -AutoExit 5
)
```

Each test runs, displays results for 5 seconds, then closes automatically
and the loop proceeds to the next bitrate.

---

## Output Files

| File | Trigger | Columns |
|---|---|---|
| `iperf3_YYYYMMDD_HHMMSS.csv` | Export CSV button | Time, Bitrate, Loss, Jitter |
| `iperf3_YYYYMMDD_HHMMSS.png` | Save PNG button | Full window screenshot |
| Custom log file | Live during test | Same CSV format (`-LogFile`) |

---

## Data Capacity

`-MaxPoints 36000` with `-Interval 0.1` = up to **1 hour** in memory.
Statistics use O(1) running accumulators — performance is constant
regardless of test duration.

---

## Troubleshooting

**ADB fails** — check `adb.exe` path and run `adb devices`.

**No chart data (TCP)** — verify the X axis shows time; only the Bitrate
chart is active in TCP mode, this is expected.

**No chart data (UDP)** — verify iPerf3 binary on device, server running,
and `-Protocol -u` is set.

**Export CSV greyed out** — click Stop or wait for test to finish.

**CPU shows "not found"** — start `iperf3.exe -s` before launching the script.

**CPU differs from Task Manager** — use `-CpuDivisor N` to override.

**Visual Analyzer does not launch** — check `-VisualAnalyzerScript` path.
Do not include PS execution flags in `-VisualAnalyzerArgs`.