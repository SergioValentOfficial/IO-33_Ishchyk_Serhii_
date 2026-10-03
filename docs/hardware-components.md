ДОСЛІДЖЕННЯ КОМПОНЕНТІВ
Heliostat

Склад системи:

| Компонент | К-сть | Інтерфейс | Роль | Поточне підключення |
| --- | --- | --- | --- | --- |
| GM5528 / GL5528* | 4 | Аналоговий сигнал → ADC | Визначення напрямку світла | PC0–PC3 / ADC1_IN10–13 |
| SG92R | 2 | PWM | Азимут та елевація | PB6 / TIM4_CH1; PB7 / TIM4_CH2 |
| W25Q64FV** | 1 | SPI | Зовнішня NOR Flash | SPI2 PB13–PB15 + CS GPIO |
| STM32F411RET6 | 1 | ADC/PWM/SPI/UART/GPIO | Центральний контролер | NUCLEO-F411RE |

GM5528 / GL5528 — фоторезистор

Тип: CdS-фоторезистор (LDR), 2 виводи. У геліостаті чотири датчики розташовуються квадрантом. Кожен LDR разом із фіксованим резистором утворює дільник напруги; STM32 вимірює напругу середньої точки через ADC. Порівняння чотирьох показників дає інформацію про напрямок більшого освітлення.

Характеристики GL5528: 

| Параметр | Значення |
| --- | --- |
| Максимальна напруга | 150 V |
| Максимальна потужність | 100 mW |
| Робоча температура | −30…+70 °C |
| Пікова довжина хвилі | 540 nm |
| Опір при освітленні | 10–20 kΩ (10 lux у використаному описі) |
| Темновий опір | 1 MΩ |
| Час зростання | 20 ms |
| Час спадання | 30 ms |
| Корпус / габарит | близько 5.0 mm для вказаного GL5528 |
| Виводи | 2 |

Схема підключення

LDR сам по собі не підключається до ADC як цифровий сенсор. Потрібен дільник напруги:

VCC  > Rfixed > ADC node > LDR > GND

або еквівалентне дзеркальне включення. Вибір розташування LDR у дільнику визначає, чи зростатиме ADC-напруга при збільшенні освітленості.

Поточний ADC mapping:

| Датчик | STM32 pin | ADC channel |
| --- | --- | --- |
| LDR1 | PC0 | ADC1_IN10 |
| LDR2 | PC1 | ADC1_IN11 |
| LDR3 | PC2 | ADC1_IN12 |
| LDR4 | PC3 | ADC1_IN13 |

Цей mapping відповідає нашій поточній CubeMX-конфігурації.

Register map: N/A — GM5528 / GL5528 є пасивним аналоговим компонентом і не має цифрових регістрів.

Додаткове джерело: GL55 series datasheet, p. 1 — таблиця GL5528 (150 V, 8–20 kΩ, 1 MΩ, 540 nm, −30…+70 °C): https://datasheet.lcsc.com/datasheet/pdf/0ac3d43edf3703d98e2e4a16b4a936c8.pdf?productCode=C10080

Джерело:

SENBA SENSING TEC GL5528, опис/дані компонента:

https://www.marutsu.co.jp/pc/i/2783924/

Дані сторінки містять 150 V, 100 mW, −30…+70 °C, 540 nm, 10–20 kΩ, 1 MΩ та 20/30 ms.  Додатковий PDF GL55 series: p. 1.

SG92R — цифровий сервопривід

Кількість: 2. Один сервопривід використовується для азимуту, другий — для елевації. Керування виконується PWM-сигналом.

Основні характеристики:

| Параметр | Значення |
| --- | --- |
| Модель | TowerPro SG92R |
| Тип | Digital 9g servo |
| Номінальна напруга в datasheet | 4.8 V |
| Діапазон живлення | 3–6 V (за технічним описом Adafruit) |
| Stall torque при 4.8 V | 2.5 kg·cm |
| Швидкість при 4.8 V | 0.1 s / 60° |
| Маса | 9 g |
| Розмір | 23 × 12.2 × 27 mm (datasheet); розмір може відрізнятися між описами |
| Dead band | 1 μs |
| Шестерні | POM with carbon-fiber gear |
| Температура | 0…55 °C у datasheet |

PWM-керування:

Для типового керування сервоприводом використовується періодичний PWM-сигнал. У практичному прикладі Adafruit 1.5 ms відповідає центральному положенню, приблизно 1 ms — одному краю, 2 ms — іншому. Повний механічний діапазон конкретного сервопривода може бути меншим; вихід за безпечний діапазон імпульсів може пошкодити механіку.

Поточний timer mapping:

| Вісь | STM32 pin | Timer channel |
| --- | --- | --- |
| Азимут | PB6 | TIM4_CH1 |
| Елевація | PB7 | TIM4_CH2 |

Живлення

Servo VCC і GND підключаються до відповідного джерела живлення, а STM32 подає лише керуючий сигнал. Для двох сервоприводів потрібно врахувати струм при пуску та навантаженні; живити моторний ланцюг безпосередньо від GPIO не можна.

Register map: N/A — SG92R керується PWM-сигналом і не має цифрових регістрів.

Струм: точне значення робочого та пускового струму у використаному datasheet не наведено, тому для сервоприводів передбачається окреме джерело живлення.

SG92R datasheet, p. 1 — 4.8 V, 2.5 kg·cm, 0.1 s/60°, 9 g, 23×12.2×27 mm, 1 μs dead band.

Джерела:

SG92R datasheet:

https://cdn-shop.adafruit.com/product-files/5592/C17481_SG92R_datasheet.pdf

Додатковий технічний опис та приклад PWM:

https://www.adafruit.com/product/169

Характеристики SG92R підтверджуються datasheet: 4.8 V, 2.5 kg·cm, 0.1 s/60°, 9 g, 1 μs dead band.  Усі наведені параметри — p. 1.

W25Q64FV — зовнішня SPI NOR Flash

Кількість: 1. W25Q64FV — конкретний представник W25Q64-сімейства, який використовуємо для технічного опису. Пам'ять призначена для збереження журналу роботи та конфігураційних даних.

Основні характеристики:

| Параметр | Значення |
| --- | --- |
| Тип | Serial NOR Flash |
| Ємність | 64 Mbit = 8 MByte |
| Живлення | 2.7–3.6 V |
| Організація | 32,768 pages × 256 bytes |
| Page Program | до 256 bytes за операцію |
| Sector Erase | 4 KB |
| Block Erase | 32 KB або 64 KB |
| Chip Erase | вся мікросхема |
| SPI clock | до 104 MHz у відповідному режимі/напрузі; конкретна межа залежить від команди та VCC |
| Основні сигнали SPI | CS#, CLK, DI/MOSI, DO/MISO |
| Додаткові лінії | /WP та /HOLD у відповідних режимах |
| Робоча температура | −40…+85 °C для industrial range |

Принцип обміну:

У стандартному SPI режимі MCU формує CS#, CLK та MOSI; Flash повертає дані через MISO. Для запису потрібна послідовність команд, зокрема Write Enable та Page Program; перед повторним використанням області пам'яті потрібна відповідна erase-операція. На Кроці 5 ми фіксуємо протокол, але драйвер ще не реалізуємо.

Основні команди W25Q64FV:

| Команда | Код | Datasheet |
| --- | --- | --- |
| Read JEDEC ID | 9Fh | p. 61 |
| Write Enable | 06h | p. 25 |
| Read Data | 03h | p. 29 |
| Page Program | 02h | p. 44 |
| Sector Erase 4 KB | 20h | p. 47 |

Споживання W25Q64FV (datasheet, p. 73): Read Data 50 MHz — до 15 mA; Page Program — до 25 mA; Sector/Block Erase — до 25 mA; Power-down — до 25 μA.

Поточний SPI mapping:

| Сигнал | STM32 pin | SPI2 |
| --- | --- | --- |
| SCK | PB13 | SPI2_SCK |
| MISO | PB14 | SPI2_MISO |
| MOSI | PB15 | SPI2_MOSI |
| CS# | окремий GPIO | Software NSS / GPIO |

Поточна CubeMX-конфігурація:

| Параметр | Значення |
| --- | --- |
| Mode | Master |
| Direction | 2 Lines |
| Data size | 8 bit |
| First bit | MSB First |
| CPOL | Low |
| CPHA | 1 Edge |
| NSS | Software |

Джерело:

W25Q64FV datasheet:

https://cdn-shop.adafruit.com/product-files/3085/w25q64fv.pdf

У datasheet наведені 2.7–3.6 V, 32,768 pages × 256 bytes, 4 KB sector erase та до 104 MHz залежно від режиму.  General Description — p. 5; Standard SPI Instruction Set — p. 21; DC Electrical Characteristics — p. 73.

STM32F411RET6 / NUCLEO-F411RE

NUCLEO-F411RE — STM32 Nucleo-64 development board на STM32F411RET6. Плата має вбудований ST-LINK/V2-1 та Virtual COM Port, що зручно для UART-діагностики. ST підтверджує для STM32F411RE Cortex-M4 з FPU, до 100 MHz та 512 KB Flash.

Ресурси MCU, які потрібні геліостату:

| Ресурс | Використання в проєкті |
| --- | --- |
| ADC1 | 4 LDR; PC0–PC3 / ADC1_IN10–13 |
| TIM4 CH1/CH2 | 2 SG92R; PB6/PB7 |
| SPI2 | W25Q64; PB13/PB14/PB15 + CS GPIO |
| USART2 | CLI та діагностичний вивід |
| GPIO | CS Flash, LED та інші цифрові сигнали |

Ключові характеристики:

| Параметр | Значення |
| --- | --- |
| MCU | STM32F411RET6 |
| CPU | Arm Cortex-M4 with FPU/DSP |
| Максимальна частота | 100 MHz |
| Flash | 512 KB |
| SRAM | 128 KB |
| ADC | 12-bit ADC |
| Плата | NUCLEO-F411RE, Nucleo-64 |
| Програмування/налагодження | On-board ST-LINK/V2-1 |

Джерела:

STM32F411RE product page: https://www.st.com/en/microcontrollers-microprocessors/stm32f411re.html

STM32F411 datasheet: https://www.st.com/resource/en/datasheet/stm32f411re.pdf

STM32F411 reference manual RM0383: https://www.st.com/resource/en/reference_manual/rm0383-stm32f411xce-advanced-armbased-32bit-mcus-stmicroelectronics.pdf

NUCLEO-64 UM1724: https://www.st.com/resource/en/user_manual/dm00105823.pdf

ST прямо ідентифікує NUCLEO-F411RE як Nucleo-64 board з STM32F411RET6; сторінка документації містить UM1724 для Nucleo-64 (MB1136).
