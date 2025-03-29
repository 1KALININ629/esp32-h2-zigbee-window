# Подробная инструкция по установке и настройке проекта

Данная инструкция описывает детальные шаги по установке, настройке и запуску проекта умного окна на базе ESP32-C6 с поддержкой ZigBee.

## Содержание

1. [Предварительные требования](#1-предварительные-требования)
2. [Установка ESP-IDF](#2-установка-esp-idf)
3. [Клонирование проекта](#3-клонирование-проекта)
4. [Установка компонента ESP-ZB](#4-установка-компонента-esp-zb)
5. [Проверка структуры проекта](#5-проверка-структуры-проекта)
6. [Настройка окружения](#6-настройка-окружения)
7. [Сборка проекта](#7-сборка-проекта)
8. [Загрузка прошивки](#8-загрузка-прошивки)
9. [Подключение сервоприводов](#9-подключение-сервоприводов)
10. [Настройка ZigBee](#10-настройка-zigbee)
11. [Интеграция с умным домом](#11-интеграция-с-умным-домом)
12. [Обновление прошивки OTA](#12-обновление-прошивки-ota)
13. [Устранение неполадок](#13-устранение-неполадок)

## 1. Предварительные требования

Перед началом работы убедитесь, что у вас установлены:

- **Git** для клонирования репозиториев
- **Python** (версия 3.6 или выше) - [скачать](https://www.python.org/downloads/)
- **Драйверы для UART/USB** (CP210x для большинства плат ESP32-C6) - [скачать](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers)

## 2. Установка ESP-IDF

ESP-IDF (Espressif IoT Development Framework) - это официальная среда разработки для ESP32-C6.

### Для Windows:

1. Скачайте установщик ESP-IDF с [официального сайта](https://dl.espressif.com/dl/esp-idf/)
2. Запустите установщик и следуйте инструкциям
3. В процессе установки:
   - Выберите опцию "Express Installation"
   - Выберите ESP-IDF v5.0 или выше
   - Убедитесь, что выбрана опция "Install Python"
   - Выберите опцию "Register ESP-IDF in system PATH"

**Альтернативный способ для Windows (через PowerShell):**

```powershell
# Создайте папку для ESP-IDF
mkdir -p "$env:USERPROFILE\esp"
cd "$env:USERPROFILE\esp"

# Клонируйте ESP-IDF
git clone -b v5.0 --recursive https://github.com/espressif/esp-idf.git

# Запустите скрипт установки
cd esp-idf
./install.ps1
```

### Для Linux/macOS:

```bash
# Создайте папку для ESP-IDF
mkdir -p ~/esp
cd ~/esp

# Клонируйте ESP-IDF
git clone -b v5.0 --recursive https://github.com/espressif/esp-idf.git

# Запустите скрипт установки
cd ~/esp/esp-idf
./install.sh esp32c6
```

## 3. Клонирование проекта

Создайте директорию для проекта и клонируйте репозиторий:

```bash
# Для Windows (PowerShell)
mkdir -p "$env:USERPROFILE\projects"
cd "$env:USERPROFILE\projects"
git clone https://github.com/username/esp32-c6-zigbee-window.git
cd esp32-c6-zigbee-window

# Для Linux/macOS
mkdir -p ~/projects
cd ~/projects
git clone https://github.com/username/esp32-c6-zigbee-window.git
cd esp32-c6-zigbee-window
```

## 4. Установка компонента ESP-ZB

Для работы с ZigBee необходимо дополнительно установить компонент ESP-ZB:

```bash
# Создайте директорию components (если её ещё нет)
mkdir -p components
cd components

# Клонируйте компонент ESP-ZB
git clone --recursive https://github.com/espressif/esp-zigbee-sdk.git esp_zb

# Вернитесь в корневую директорию проекта
cd ..
```

## 5. Проверка структуры проекта

После клонирования и установки ESP-ZB, структура проекта должна выглядеть следующим образом:

```
esp32-c6-zigbee-window/
├── components/
│   └── esp_zb/
├── main/
│   ├── CMakeLists.txt
│   ├── esp_zigbee_lib.c
│   ├── esp_zigbee_lib.h
│   ├── main.c
│   ├── ota_update.c
│   ├── ota_update.h
│   ├── power_management.c
│   ├── power_management.h
│   ├── servo_control.c
│   ├── servo_control.h
│   ├── state_management.c
│   ├── state_management.h
│   ├── zigbee_handler.c
│   └── zigbee_handler.h
├── CMakeLists.txt
├── INSTALL.md
├── README.md
└── sdkconfig.defaults
```

Если какие-то файлы отсутствуют, создайте их с соответствующим содержимым, которое было предоставлено ранее.

## 6. Настройка окружения

Перед сборкой необходимо активировать окружение ESP-IDF:

### Для Windows (PowerShell):

```powershell
# Активировать окружение ESP-IDF
& "$env:USERPROFILE\esp\esp-idf\export.ps1"
```

### Для Linux/macOS:

```bash
# Активировать окружение ESP-IDF
. $HOME/esp/esp-idf/export.sh
```

### Важные изменения в файле esp_zigbee_lib.c

Отредактируйте файл `main/esp_zigbee_lib.c`, заменив импорты:

```c
// Замените эти строки:
#include "esp_zb_device.h"
#include "esp_zb_zcl.h"
#include "esp_zb_zcl_window_covering.h"

// На эти строки:
#include "esp_zb/esp_zb_device.h"
#include "esp_zb/esp_zb_zcl.h"
#include "esp_zb/esp_zb_zcl_window_covering.h"
```

## 7. Сборка проекта

Для сборки проекта выполните следующие команды:

```bash
# Установите целевую платформу (ESP32-C6)
idf.py set-target esp32c6

# Сгенерируйте конфигурацию (необязательно)
idf.py menuconfig

# Соберите проект
idf.py build
```

Процесс сборки может занять несколько минут. По завершении вы увидите сообщение о успешной сборке и расположение файла прошивки.

## 8. Загрузка прошивки

### Подключение платы

1. Подключите плату ESP32-C6 к компьютеру через USB-кабель
2. Определите порт COM (Windows) или /dev/ttyUSB* (Linux/macOS):
   - **Windows**: Проверьте в Диспетчере устройств → Порты (COM и LPT)
   - **Linux**: Выполните команду `ls /dev/ttyUSB*`
   - **macOS**: Выполните команду `ls /dev/cu.*`

### Загрузка прошивки

```bash
# Замените PORT на ваш порт (например, COM3 в Windows или /dev/ttyUSB0 в Linux)
idf.py -p PORT flash
```

### Проверка логов

```bash
# Запустите монитор для просмотра отладочных сообщений
idf.py -p PORT monitor
```

Для выхода из монитора нажмите Ctrl+].

### Полный процесс (сборка + загрузка + мониторинг)

```bash
idf.py -p PORT build flash monitor
```

## 9. Подключение сервоприводов

### Схема подключения

1. **Питание сервоприводов**:
   - VCC: Подключите к внешнему источнику питания 5V (НЕ к пину 5V платы ESP32-C6!)
   - GND: Подключите к GND внешнего источника и к GND платы ESP32-C6

2. **Сигнальные провода**:
   - Сервопривод ручки: Подключите к GPIO4 ESP32-C6
   - Сервопривод зазора: Подключите к GPIO5 ESP32-C6

![Схема подключения](https://i.ibb.co/sFF7DzL/servo-wiring.png)

### Примечания по питанию

- Сервоприводы MG996R требуют до 2.5A тока при максимальной нагрузке
- Всегда используйте отдельный источник питания для сервоприводов
- Соедините GND источника питания с GND платы ESP32-C6

## 10. Настройка ZigBee

### Первичная настройка

1. После первой загрузки прошивки устройство автоматически запустится в режиме ZigBee End Device
2. Для входа в режим сопряжения:
   - Удерживайте кнопку BOOT на плате ESP32-C6 в течение 3 секунд
   - Встроенный светодиод начнет мигать, указывая на активацию режима сопряжения

### Настройка ZigBee координатора

Для работы системы необходим ZigBee координатор. Рекомендуемые варианты:

1. **CC2652P/CC2652R** - Texas Instruments ZigBee координатор
2. **ConBee II / RaspBee II** - deCONZ координатор
3. **Sonoff ZBBridge-P** - координатор на базе EFR32MG21
4. **Еще один ESP32-C6** - настроенный как ZigBee координатор

Инструкция по настройке Zigbee2MQTT с координатором:
[https://www.zigbee2mqtt.io/guide/getting-started/](https://www.zigbee2mqtt.io/guide/getting-started/)

## 11. Интеграция с умным домом

### Home Assistant

1. Установите один из вариантов интеграции ZigBee:
   - [Zigbee2MQTT](https://www.zigbee2mqtt.io/guide/installation/01_linux.html)
   - [ZHA (ZigBee Home Automation)](https://www.home-assistant.io/integrations/zha/)

2. После подключения устройства в Home Assistant, оно будет отображаться как устройство покрытия окна (Window Covering).

3. Пример конфигурации `configuration.yaml` для Home Assistant:

```yaml
cover:
  - platform: template
    covers:
      living_room_window:
        friendly_name: "Окно в гостиной"
        device_class: window
        value_template: "{{ is_state('cover.zigbee_window_covering', 'open') }}"
        open_cover:
          service: cover.open_cover
          target:
            entity_id: cover.zigbee_window_covering
        close_cover:
          service: cover.close_cover
          target:
            entity_id: cover.zigbee_window_covering
        stop_cover:
          service: cover.stop_cover
          target:
            entity_id: cover.zigbee_window_covering
        set_cover_position:
          service: cover.set_cover_position
          target:
            entity_id: cover.zigbee_window_covering
          data:
            position: "{{ position }}"
```

### Интеграция с Яндекс Алисой

1. Установите [Yandex Smart Home](https://github.com/dmitry-k/yandex_smart_home) в Home Assistant
2. Настройте интеграцию согласно инструкции
3. Устройство будет доступно в Яндекс Алисе как "шторы"

## 12. Обновление прошивки OTA

### Подготовка файла прошивки

```bash
# Соберите проект с включенной поддержкой OTA
idf.py build
```

Файл прошивки будет расположен в:
`build/esp32-c6-zigbee-window.bin`

### Настройка HTTP-сервера

1. Разместите файл прошивки на HTTP-сервере
2. Убедитесь, что файл доступен по URL

### Запуск OTA-обновления

Обновление можно запустить через:

1. **Home Assistant**: Создайте автоматизацию с вызовом сервиса
2. **Zigbee2MQTT**: Используйте функцию OTA обновления
3. **HTTP-запрос**: Отправьте запрос на устройство

## 13. Устранение неполадок

### Ошибки сборки

#### Ошибка: "Component 'esp_zb' not found"

**Решение:**
- Проверьте, что директория `components/esp_zb` существует и содержит код компонента
- Убедитесь, что в `main/CMakeLists.txt` компонент указан в REQUIRES

#### Ошибка: "Cannot find esp_zb_device.h"

**Решение:**
- Измените пути импорта в `main/esp_zigbee_lib.c`:
```c
#include "esp_zb/esp_zb_device.h"
#include "esp_zb/esp_zb_zcl.h"
#include "esp_zb/esp_zb_zcl_window_covering.h"
```

### Ошибки прошивки

#### Ошибка: "Failed to connect to ESP32-C6"

**Решение:**
- Проверьте подключение кабеля
- Удерживайте кнопку BOOT при подключении платы
- Проверьте правильность указанного порта COM/USB

#### Ошибка: "A fatal error occurred: Invalid head of packet"

**Решение:**
- Переведите плату в режим загрузки: удерживайте кнопку BOOT, нажмите и отпустите кнопку RESET, затем отпустите BOOT
- Попробуйте другой USB-кабель (некоторые кабели поддерживают только зарядку)

### Ошибки ZigBee

#### Устройство не подключается к сети ZigBee

**Решение:**
- Убедитесь, что координатор работает и находится в режиме сопряжения
- Проверьте расстояние между устройством и координатором (не более 10-20 метров в помещении)
- Сбросьте настройки ZigBee: удерживайте кнопку BOOT 10 секунд

#### Сервоприводы не работают

**Решение:**
- Проверьте подключение питания (5V и достаточная мощность)
- Проверьте подключение сигнальных проводов к правильным пинам
- Убедитесь, что не возникает механического сопротивления

## Дополнительные ресурсы

- [Официальная документация ESP-IDF](https://docs.espressif.com/projects/esp-idf/en/v5.0/esp32c6/index.html)
- [Документация по ESP-ZB](https://docs.espressif.com/projects/esp-zigbee-sdk/en/latest/)
- [Документация по Zigbee2MQTT](https://www.zigbee2mqtt.io/)
- [Документация Home Assistant](https://www.home-assistant.io/docs/)

---

© 2024 Smart Home Projects 