# Product Requirements Document — Heliostat

## 1. Product name

**Heliostat**

STM32-based automatic light-tracking platform.

---

# 2. Product idea

The Heliostat is an embedded system that detects the direction of the
strongest light source and automatically changes the orientation of a
mechanical platform.

The system uses four light sensors arranged around a central point and two
servo motors responsible for two-axis positioning.

The main controller is STM32F411RET6.

---

# 3. Problem

A fixed-position platform cannot automatically follow a moving light source.

The purpose of the Heliostat is to determine the direction of the strongest
light using multiple sensors and automatically orient the platform toward it.

The project demonstrates a practical embedded control system rather than
only testing an individual microcontroller peripheral.

---

# 4. Product goal

The goal is to create a working prototype that:

1. measures light intensity from four directions;
2. determines the direction in which the light intensity is greater;
3. controls two axes of mechanical movement;
4. continuously adjusts the platform orientation;
5. stores selected configuration or operational data in external Flash memory;
6. provides diagnostic information through UART.

---

# 5. Target hardware

## Main controller

NUCLEO-F411RE / STM32F411RET6.

## Sensors and actuators

- 4 × GM5528 LDR;
- 2 × SG92R servo;
- 1 × W25Q64 SPI Flash.

---

# 6. Functional requirements

## FR-01 — Light measurement

The system shall periodically read four analog light sensor signals using
the STM32 ADC.

## FR-02 — Sensor comparison

The firmware shall compare the measurements from the four sensors.

## FR-03 — Direction determination

The firmware shall determine the direction in which the measured light
intensity is greater.

## FR-04 — Azimuth control

The system shall control one servo motor responsible for horizontal
orientation.

## FR-05 — Elevation control

The system shall control a second servo motor responsible for vertical
orientation.

## FR-06 — Continuous tracking

The system shall repeat the measurement and positioning process continuously.

## FR-07 — Flash storage

The system shall provide communication with external W25Q64 Flash through SPI
for non-volatile data storage.

## FR-08 — Diagnostics

The system shall provide diagnostic output through USART2.

---

# 7. Hardware interfaces

| Function | Interface | STM32 resource |
|---|---|---|
| LDR1 | ADC | ADC1_IN10 / PC0 |
| LDR2 | ADC | ADC1_IN11 / PC1 |
| LDR3 | ADC | ADC1_IN12 / PC2 |
| LDR4 | ADC | ADC1_IN13 / PC3 |
| Azimuth servo | PWM | TIM4_CH1 / PB6 |
| Elevation servo | PWM | TIM4_CH2 / PB7 |
| W25Q64 clock | SPI | SPI2_SCK / PB13 |
| W25Q64 data to MCU | SPI | SPI2_MISO / PB14 |
| W25Q64 data from MCU | SPI | SPI2_MOSI / PB15 |
| W25Q64 selection | GPIO | CS# |
| Diagnostics | UART | USART2 |

---

# 8. Software requirements

The firmware shall be generated and configured using STM32CubeMX and built
using the project Makefile and ARM GNU Embedded Toolchain.

The firmware shall contain initialization for:

- ADC1;
- TIM4 PWM;
- SPI2;
- USART2;
- required GPIO.

The project shall initially verify peripheral initialization before full
sensor and Flash drivers are implemented.

---

# 9. Operating algorithm

The expected high-level algorithm is:

1. Initialize the microcontroller.
2. Initialize GPIO.
3. Initialize ADC1.
4. Initialize TIM4 PWM.
5. Initialize SPI2.
6. Initialize USART2.
7. Read four LDR values.
8. Compare the sensor values.
9. Determine the direction of stronger illumination.
10. Adjust azimuth servo.
11. Adjust elevation servo.
12. Optionally save required data to W25Q64.
13. Output diagnostic information.
14. Repeat the cycle.

---

# 10. Minimum Viable Product

The MVP shall demonstrate:

- successful STM32 initialization;
- successful ADC initialization;
- successful PWM initialization;
- successful SPI initialization;
- four sensor inputs;
- two servo outputs;
- continuous light-direction tracking;
- UART diagnostic output.

External Flash functionality may initially be demonstrated by successful SPI
initialization and identification before implementing the complete storage
logic.

---

# 11. Non-functional requirements

## Reliability

The system should continue operating continuously without requiring a reset
during normal operation.

## Maintainability

Peripheral configuration shall be stored in the CubeMX `.ioc` file and the
source code shall remain organized in the generated firmware structure.

## Reproducibility

The project shall be buildable using the provided Docker/Makefile environment.

## Documentation

Hardware documentation, product requirements and the final laboratory report
shall be stored in the `docs` directory.

---

# 12. Development environment

The project uses:

- STM32CubeMX;
- STM32 HAL;
- ARM GNU Embedded Toolchain;
- GNU Make;
- Docker;
- Visual Studio Code;
- Wokwi for simulation.

---

# 13. Testing

The project shall be tested at several levels.

### Initialization test

Verify that ADC1, TIM4, SPI2 and USART2 initialize successfully.

### UART test

Verify diagnostic messages in the serial terminal.

### ADC test

Verify that the four sensor channels produce changing values when the
simulated or physical illumination changes.

### PWM test

Verify that changing the calculated target position changes the servo PWM
output.

### SPI test

Verify communication with the W25Q64 device.

### System test

Verify that the platform changes its orientation according to the relative
light intensity measured by the four sensors.

---

# 14. Project limitations

The first prototype is focused on demonstrating the embedded control system.

The following items are outside the minimum scope:

- weatherproof mechanical construction;
- industrial-grade positioning accuracy;
- autonomous power generation;
- GPS;
- remote Internet control;
- advanced solar-power optimization.

---

# 15. Possible future extensions

Possible future versions may include:

- solar panel mounting;
- automatic calibration;
- better filtering of sensor noise;
- configurable tracking sensitivity;
- storage of calibration values in Flash;
- OLED or LCD status display;
- remote monitoring;
- wireless communication;
- weather sensors;
- energy optimization.

---

# 16. Definition of Done

The Heliostat project is considered complete when:

- the firmware builds successfully;
- the required peripherals initialize successfully;
- four light sensors can be read;
- two servos can be controlled;
- SPI communication with W25Q64 is demonstrated;
- the system can determine the direction of stronger light;
- the servos react to the calculated direction;
- diagnostic UART output works;
- the repository contains the required source code and documentation;
- the final laboratory report is added to the repository;
- the project can be demonstrated in the selected environment.

---

# 17. Project success criterion

The main success criterion is a working two-axis light-tracking prototype that
uses multiple sensor measurements to automatically change its orientation
toward the stronger light source.
