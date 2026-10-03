# Hardware Components

## GM5528 LDR

Four photoresistors are used to determine the relative light intensity
from four directions. The analog outputs are connected to ADC inputs.

## SG92R Servo

Two servo motors provide mechanical positioning of the heliostat.
Their control signals are generated using PWM timers.

## W25Q64

W25Q64 is an external SPI Flash memory used for non-volatile storage.
The microcontroller communicates with it using SPI.

## STM32F411RE

STM32F411RE is the main microcontroller of the system.
It processes sensor measurements and generates control signals.
