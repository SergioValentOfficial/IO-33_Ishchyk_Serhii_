# Hardware Components

## 1. Project hardware overview

The Heliostat project is based on the STM32F411RE microcontroller and uses four
light-dependent resistors, two servo motors and external SPI Flash memory.

The main interfaces are:

- ADC — measurement of four light sensors;
- PWM — control of two servo motors;
- SPI — communication with external W25Q64 Flash memory;
- USART2 — diagnostic output.

### Main components

| Component | Quantity | Interface | Purpose |
|---|---:|---|---|
| GM5528 / GL5528 LDR | 4 | Analog / ADC | Light intensity measurement |
| SG92R | 2 | PWM | Mechanical positioning |
| W25Q64FV | 1 | SPI | Non-volatile data storage |
| STM32F411RET6 | 1 | ADC/PWM/SPI/UART/GPIO | Main controller |

---

# 2. GM5528 / GL5528 LDR

## Purpose

The heliostat uses four photoresistors to determine the direction of the
strongest light source.

The four sensors are positioned as a quadrant. The STM32 reads the four
analog voltage levels and compares them. The difference between sensor values
is used to determine how the platform should be moved.

The methodical documentation specifies GM5528. The available technical source
used for numerical characteristics is documentation for GL5528. The exact
component marking should therefore be verified before physical assembly.

## Interface

The LDR is an analog component and does not use a digital communication
protocol.

The photoresistor is connected as part of a voltage divider:

VCC -> fixed resistor -> ADC node -> LDR -> GND

The STM32 ADC measures the voltage at the middle point of the divider.

## Current pin mapping

| Sensor | STM32 pin | ADC channel |
|---|---|---|
| LDR1 | PC0 | ADC1_IN10 |
| LDR2 | PC1 | ADC1_IN11 |
| LDR3 | PC2 | ADC1_IN12 |
| LDR4 | PC3 | ADC1_IN13 |

## Characteristics of GL5528 from the used technical source

| Parameter | Value |
|---|---|
| Maximum voltage | 150 V |
| Maximum power | 100 mW |
| Operating temperature | -30...+70 °C |
| Peak wavelength | approximately 540 nm |
| Light resistance | 10–20 kOhm at the specified test condition |
| Dark resistance | approximately 1 MOhm |
| Rise time | approximately 20 ms |
| Fall time | approximately 30 ms |
| Number of terminals | 2 |

## Important considerations

- The LDR provides an analog resistance change.
- The ADC measures voltage, not resistance directly.
- The fixed resistor value must be selected according to the actual LDR range.
- For the heliostat, relative differences between the four sensors are more
  important than absolute lux measurement.
- The exact GM5528 characteristics must be verified against the physical
  component before final hardware assembly.

## Source

GL5528 technical documentation:

https://www.marutsu.co.jp/pc/i/2783924/

---

# 3. SG92R Servo Motor

## Purpose

Two SG92R servo motors are used to orient the heliostat.

One servo controls the azimuth axis and the second servo controls the
elevation axis.

## Interface

The servo is controlled using a PWM signal.

### Current pin mapping

| Axis | STM32 pin | Timer |
|---|---|---|
| Azimuth | PB6 | TIM4_CH1 |
| Elevation | PB7 | TIM4_CH2 |

## Main characteristics

| Parameter | Value |
|---|---|
| Model | TowerPro SG92R |
| Type | Digital micro servo |
| Nominal voltage in datasheet | 4.8 V |
| Operating range in available technical description | approximately 3–6 V |
| Stall torque at 4.8 V | approximately 2.5 kg·cm |
| Speed at 4.8 V | approximately 0.1 s / 60° |
| Weight | approximately 9 g |
| Dimensions | approximately 23 × 12.2 × 27 mm |
| Dead band | approximately 1 us |

## PWM control

A typical servo control signal uses a periodic PWM waveform.

A practical control example is:

- approximately 1 ms — one side of the position range;
- approximately 1.5 ms — center position;
- approximately 2 ms — opposite side.

The exact safe mechanical range depends on the particular servo and should
be checked before applying extreme positions.

## Power

The servo motor should not be powered directly from an STM32 GPIO.

The signal pin is controlled by the STM32 timer, while the servo receives
power from an appropriate supply.

The common ground between the STM32 and the servo power system is required.

## Sources

SG92R datasheet:

https://cdn-shop.adafruit.com/product-files/5592/C17481_SG92R_datasheet.pdf

Additional technical information:

https://www.adafruit.com/product/169

---

# 4. W25Q64FV SPI Flash

## Purpose

W25Q64FV is an external Serial NOR Flash memory.

The memory can be used to store:

- configuration data;
- calibration parameters;
- operation history;
- other non-volatile information.

The firmware driver is not implemented at this stage. This section documents
the hardware interface required for the future driver.

## Main characteristics

| Parameter | Value |
|---|---|
| Type | Serial NOR Flash |
| Capacity | 64 Mbit |
| Capacity in bytes | 8 MByte |
| Supply voltage | 2.7–3.6 V |
| Page size | 256 bytes |
| Number of pages | 32768 |
| Sector erase | 4 KB |
| Block erase | 32 KB / 64 KB |
| Interface | SPI |
| Maximum SPI frequency | up to approximately 104 MHz depending on mode and conditions |

## SPI signals

| Flash signal | STM32 |
|---|---|
| CLK | PB13 / SPI2_SCK |
| DO / MISO | PB14 / SPI2_MISO |
| DI / MOSI | PB15 / SPI2_MOSI |
| CS# | Dedicated GPIO |

## SPI configuration

The planned configuration is:

| Parameter | Value |
|---|---|
| Mode | Master |
| Direction | 2 Lines |
| Data size | 8 bit |
| First bit | MSB First |
| Clock polarity | Low |
| Clock phase | 1st Edge |
| NSS | Software |

## Important commands

The future Flash driver will need operations such as:

- Read JEDEC ID;
- Read data;
- Write Enable;
- Page Program;
- Sector Erase;
- Block Erase;
- Chip Erase;
- Read Status Register.

At this stage these commands are documented but the driver is not implemented.

## Power

W25Q64FV uses a 2.7–3.6 V supply.

The physical circuit must provide compatible voltage levels and correct
connection of the supply and ground.

Unused control pins such as WP# and HOLD# must be handled according to the
specific datasheet and final hardware design.

## Source

W25Q64FV datasheet:

https://cdn-shop.adafruit.com/product-files/3085/w25q64fv.pdf

---

# 5. STM32F411RET6 / NUCLEO-F411RE

## Purpose

STM32F411RET6 is the main microcontroller of the heliostat.

It reads the light sensors, calculates the required movement, generates PWM
signals for the servos and communicates with external Flash memory.

## Main characteristics

| Parameter | Value |
|---|---|
| MCU | STM32F411RET6 |
| CPU | Arm Cortex-M4 with FPU |
| Maximum frequency | 100 MHz |
| Flash | 512 KB |
| SRAM | 128 KB |
| ADC | 12-bit |
| Development board | NUCLEO-F411RE |
| Debug/programming | On-board ST-LINK/V2-1 |

## Peripheral allocation

| Peripheral | Purpose |
|---|---|
| ADC1 | Four LDR sensors |
| TIM4 CH1 | Azimuth servo |
| TIM4 CH2 | Elevation servo |
| SPI2 | W25Q64 Flash |
| USART2 | Diagnostic UART |
| GPIO | Flash CS and other digital signals |

## Current mapping

### ADC

| Pin | Function |
|---|---|
| PC0 | ADC1_IN10 |
| PC1 | ADC1_IN11 |
| PC2 | ADC1_IN12 |
| PC3 | ADC1_IN13 |

### PWM

| Pin | Function |
|---|---|
| PB6 | TIM4_CH1 |
| PB7 | TIM4_CH2 |

### SPI

| Pin | Function |
|---|---|
| PB13 | SPI2_SCK |
| PB14 | SPI2_MISO |
| PB15 | SPI2_MOSI |

### UART

USART2 is used for diagnostic output through the NUCLEO board's ST-LINK
Virtual COM Port.

## Sources

STM32F411RE:

https://www.st.com/en/microcontrollers-microprocessors/stm32f411re.html

STM32F411RE datasheet:

https://www.st.com/resource/en/datasheet/stm32f411re.pdf

STM32F411 reference manual:

https://www.st.com/resource/en/reference_manual/rm0383-stm32f411xce-advanced-armbased-32bit-mcus-stmicroelectronics.pdf

NUCLEO-64 user manual:

https://www.st.com/resource/en/user_manual/dm00105823.pdf

---

# 6. Complete system pin mapping

| Component | Pin | STM32 function | Purpose |
|---|---|---|---|
| LDR1 | signal | PC0 / ADC1_IN10 | Light measurement |
| LDR2 | signal | PC1 / ADC1_IN11 | Light measurement |
| LDR3 | signal | PC2 / ADC1_IN12 | Light measurement |
| LDR4 | signal | PC3 / ADC1_IN13 | Light measurement |
| Servo 1 | signal | PB6 / TIM4_CH1 | Azimuth |
| Servo 2 | signal | PB7 / TIM4_CH2 | Elevation |
| W25Q64 | CLK | PB13 / SPI2_SCK | SPI clock |
| W25Q64 | DO | PB14 / SPI2_MISO | Flash to MCU |
| W25Q64 | DI | PB15 / SPI2_MOSI | MCU to Flash |
| W25Q64 | CS# | GPIO | Flash selection |

---

# 7. Parameters transferred to STM32CubeMX

## ADC

ADC1 is configured for the four LDR channels:

- PC0 / ADC1_IN10;
- PC1 / ADC1_IN11;
- PC2 / ADC1_IN12;
- PC3 / ADC1_IN13.

## PWM

TIM4 is configured for:

- CH1 / PB6 — first servo;
- CH2 / PB7 — second servo.

## SPI

SPI2 is configured as:

- Master;
- Full Duplex / 2 Lines;
- 8-bit data;
- MSB First;
- CPOL Low;
- CPHA 1st Edge;
- Software NSS.

## UART

USART2 is used for diagnostic communication at 115200 8N1 in the current
firmware configuration.

---

# 8. Important hardware limitations

Before physical assembly, the following parameters must be verified:

1. Exact GM5528 component and its datasheet.
2. Fixed resistor values for the LDR voltage dividers.
3. Safe PWM range of the actual SG92R servos.
4. External power supply and maximum current for two servos.
5. Exact GPIO selected for W25Q64 CS#.
6. Handling of WP# and HOLD# on W25Q64.
7. Voltage compatibility of all connected components.

These parameters should not be invented without checking the actual hardware.

---

# 9. Step 5 conclusion

The main components required for the Heliostat project have been investigated.
Their interfaces, electrical parameters, power requirements, pin mapping and
role in the system have been documented.

The resulting hardware architecture uses three main interfaces:

- ADC for four light sensors;
- PWM for two servo motors;
- SPI for external Flash memory.

The documented parameters provide the basis for the STM32CubeMX configuration
and for subsequent firmware development.
