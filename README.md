# testMX — Виведення зображень на E-Paper дисплей (STM32L0538-Discovery)

Проєкт для мікроконтролера **STM32L053C8T6** на налагоджувальній платі **STM32L0538-Discovery**, призначений для виведення користувацьких монохромних графічних зображень на вбудований екран електронного паперу (**E-Paper Display / EPD**).

---

## 📋 Опис та особливості проекту

- **Мікроконтролер**: STM32L053C8T6 (ARM Cortex-M0+, 32 MHz, 64 KB Flash, 8 KB RAM, ultra-low-power).
- **Дисплей**: Вбудований E-Paper дисплей 2.04" (контролер GDE021A1 / GDEM0213B74, роздільна здатність 128×250 точок у портретній орієнтації).
- **Система збірки**: CMake + Ninja, STM32 HAL & BSP драйвери, підтримка VS Code (STM32 VS Code Extension) та STM32CubeIDE / IAR EWARM.
- **Підтримка альбомної орієнтації (Landscape)**: Власна функція `SUDO_EPD_DrawLandscape()` трансформує масив розміром **248×128** пікселів із поворотом на 90° для коректного горизонтального показу на екрані безпосередньо в буфер контролера EPD.

---

## 🎨 Генерація екранних артів та зображень (Screen Art)

Для швидкого перетворення будь-яких картинок, логотипів чи ілюстрацій у C-масив використовується онлайн-інструмент:

👉 **[image2cpp (онлайн-генератор бітмапів)](https://javl.github.io/image2cpp/)** — `https://javl.github.io/image2cpp/`

### ⚙️ Покрокова інструкція підготовки зображення

1. **Відкрийте сайт**: перейдіть за адресою [https://javl.github.io/image2cpp/](https://javl.github.io/image2cpp/).
2. **1. Select image**: 
   - Завантажте файл зображення (`PNG`, `JPG`, `BMP` або `SVG`).
3. **2. Image settings**:
   - **Canvas size**: встановіть **Width = `248`**, **Height = `128`** (це робоче вікно для функції `SUDO_EPD_DrawLandscape`).
   - **Scaling**: виберіть *Scale to fit, keeping proportions* (щоб зберегти пропорції) або *Stretch to fill*.
   - **Background color**: `White` (білий).
   - **Invert image colors**: перевірте попередній перегляд. Якщо лінії малюнка мають бути чорними на білому тлі, залиште як є, або інвертуйте за потреби.
   - **Brightness / Threshold**: відрегулюйте повзунок яскравості. Для фотографій та півтонів оберіть алгоритм дизерінгу (**Floyd-Steinberg**), для логотипів та піктограм — **Threshold**.
4. **3. Output settings**:
   - **Code output style**: `C / C++ code` (або `Arduino code`).
   - **Identifier**: назва масиву (наприклад, `epd_bitmap_my_art`).
   - **Draw mode**: ⚠️ **Обов'язково виберіть: `Horizontal - 1 bit per pixel`** *(код очікує горизонтальне пакування: 31 байт на рядок, по 8 біт на байт)*.
5. **4. Generate code**:
   - Натисніть кнопку **Generate code**.
   - Скопіюйте отриманий C-масив `const uint8_t epd_bitmap_my_art[] = { ... };`.

---

## 💻 Інтеграція нового арту в код

1. Відкрийте файл [`Core/Src/main.c`](file:///D:/ST/workspace/testMX/Core/Src/main.c).
2. Вставте згенерований масив у секцію `/* USER CODE BEGIN 0 */`:
   ```c
   /* USER CODE BEGIN 0 */
   // 'my_art', 248x128px
   const uint8_t epd_bitmap_my_art[] = {
       0x00, 0x00, 0x00, ...
   };
   /* USER CODE END 0 */
   ```
3. У функції `main()` (секція `/* USER CODE BEGIN 2 */`) передайте назву нового масиву у функцію виведення:
   ```c
   /* USER CODE BEGIN 2 */
   BSP_EPD_Init();

   // Виведення вашого зображення на E-Paper дисплей
   SUDO_EPD_DrawLandscape((uint8_t*)epd_bitmap_my_art);
   BSP_EPD_RefreshDisplay();
   /* USER CODE END 2 */
   ```
4. Після виклику `BSP_EPD_RefreshDisplay()` контролер дисплея виконає циклічне перемальовування частинок електронного чорнила (кілька секунд), після чого зображення залишиться на екрані навіть при повному відключенні живлення!

---

## 🛠️ Збірка та прошивка

### Варіант 1: Через VS Code (STM32 Extension / CMake Tools)
1. Відкрийте папку проекту у Visual Studio Code.
2. Переконайтеся, що встановлено розширення **STM32 VS Code Extension** або **CMake Tools**.
3. У нижньому рядку стану оберіть конфігурацію пресету (наприклад, `Debug`).
4. Натисніть **Build** (або `F7` / комбінацію `Ctrl+Shift+B`).
5. Для запуску та налагодження підключіть плату через USB ST-LINK та натисніть `F5` (конфігурація *"STM32Cube: Launch ST-Link GDB Server"*).

### Варіант 2: Через командний рядок (CMake + Ninja)
```bash
# Конфігурація через пресет (або напряму)
cmake --preset default

# Збірка проекту
cmake --build build/Debug

# Прошивка через STM32CubeProgrammer CLI (за наявності)
STM32_Programmer_CLI -c port=SWD -w build/Debug/testMX.elf -v -hardRst
```

---

## 🔌 Апаратне підключення та піни дисплея

На платі **STM32L0538-Discovery** EPD підключено за такою схемою:

| Сигнал дисплея | Пін STM32 | Опис |
|---|---|---|
| **EPD_CS** | `PA15` | Chip Select (активний низький рівень) |
| **EPD_DC** | `PB11` | Data / Command Control |
| **EPD_RESET** | `PB2` | Апаратний скид дисплея |
| **EPD_PWR_ENn**| `PB10`| Дозвіл живлення дисплея |
| **EPD_BUSY** | `PA8` | Сигнал зайнятості (Busy) від дисплея |
| **SPI2_SCK** | `PB13` | Тактовий сигнал SPI |
| **SPI2_MOSI**| `PB15` | Лінія передачі даних SPI |
| **LD_G** | `PB4` | Зелений світлодіод (блимає в нескінченному циклі `while(1)`) |
| **LD_R** | `PA5` | Червоний світлодіод |
| **B1** | `PA0` | Користувацька синя кнопка |

---

## 📂 Структура проекту

```text
testMX/
├── .vscode/               # Налаштування збірки та дебагу у VS Code
├── .settings/             # Конфігурація STM32 тулчейнів і бандлів
├── cmake/                 # Допоміжні скрипти та конфігурація CMake
├── Core/
│   ├── Inc/               # Заголовні файли (main.h, stm32l0xx_hal_conf.h тощо)
│   └── Src/               # Вихідні коди (main.c з функцією виведення арту)
├── Drivers/
│   ├── BSP/               # Драйвери плати та EPD дисплея
│   ├── CMSIS/             # Бібліотеки ARM Cortex-M CMSIS
│   └── STM32L0xx_HAL_Driver/ # STM32 HAL бібліотека
├── CMakeLists.txt         # Основний сценарій збірки CMake
├── STM32L053xx_FLASH.ld   # Скрипт компонувальника (Linker script)
├── testMX.ioc             # Проект конфігурації STM32CubeMX
└── README.md              # Документація та інструкція
```
