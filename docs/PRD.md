# Heliostat

## Project description

The project is a heliostat system based on the STM32F411RE microcontroller.
The system determines the direction of the light source using four LDR sensors
and controls two servo motors to orient the platform toward the light.

## Target board

NUCLEO-F411RE
STM32F411RETx

## Main components

- 4x GM5528 LDR
- 2x SG92R servo motor
- W25Q64 SPI Flash memory

## Interfaces

- ADC - measurement of four light sensors
- PWM - control of two servo motors
- SPI - communication with W25Q64 Flash

## Firmware

The firmware is generated using STM32CubeMX and built with GNU Make
using the ARM GNU Embedded Toolchain.
