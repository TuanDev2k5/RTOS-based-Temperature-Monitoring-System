# RTOS-based Temperature Monitoring System

This project builds a multitasking temperature monitoring system using the STM32F405RGTx microcontroller and the FreeRTOS real-time operating system. The system integrates a user interface on a touch LCD screen, the CAN network communication standard, and power management mechanisms (Sleep/Wakeup).

The project is implemented within the framework of the Embedded Systems Practice course – University of Science (VNU-HCM).

----

## Feature Overview

- **Real-time Monitoring:** Collects data from the microcontroller's internal temperature sensor (Internal ADC).
- **HMI (Human-Machine Interface):** Direct display and interaction on a 2.8" Resistive Touch LCD (ST7789 IC) via the SPI standard.
- **CAN Bus Communication:** Handles the transmission/reception of temperature data frames (Standard ID: `0x123`) between two network nodes (CAN1 and CAN2).
- **Multitasking Management:** Utilizes FreeRTOS to independently operate tasks: sensor data collection, UI updates, and peripheral event handling.
- **Power Management:** Optimizes energy usage with Sleep mode (WFI), supporting wake-up via external interrupt (EXTI).

----

## Hardware Requirements

- **Microcontroller (MCU):** STM32F405RGTx (Cortex-M4 Core)
- **Display:** 2.8-inch Resistive Touch LCD (ST7789)
- **Transceiver:** 2 CAN Bus communication modules (TJA1050 or equivalent)

### Pin Mapping

| Functional Block | MCU Pin | Detailed Description |
| :--- | :--- | :--- |
| **SPI1 (LCD & Touch)** | `PA5`, `PA6`, `PA7` | SCK, MISO, MOSI |
| **LCD Control** | `PB6`, `PB7`, `PB8` | Backlight (BL), Chip Select (CS), Data/Command (DC) |
| **Touch Control** | `PA9`, `PB4` | Touch Panel Chip Select (TP_CS), Touch Interrupt (TP_IRQ) |
| **CAN Bus 1** | `PA11`, `PA12` | CAN1_RX, CAN1_TX |
| **CAN Bus 2** | `PB12`, `PB13` | CAN2_RX, CAN2_TX |
| **Wakeup** | `PA0` | Hardware Push Button (External Interrupt EXTI) |

----

## Software Architecture

The system is designed based on FreeRTOS with 3 threads running in parallel, exchanging data via a Message Queue:

1. `Temp_Task`: Samples the internal temperature ADC channel every 1000ms, calculates the temperature (°C), and pushes the data into `TempQueue`.
2. `UI_Task`: Receives data from `TempQueue` and renders the value on the LCD screen.
3. `Touch_Task`: Polls interrupts and reads touch coordinates.
   - **PLAY Event:** Packages the temperature payload, transmits it via CAN1, and checks the receive buffer of CAN2.
   - **PAUSE Event:** Suspends SysTick and HAL Tick, and calls the `WFI` (Wait For Interrupt) instruction to put the MCU into Sleep state.

----

## Build & Flash Guide

The project's peripherals are configured via STM32CubeMX and can be compiled using GNU Make or STM32CubeIDE.

----

## System Demonstration


https://github.com/user-attachments/assets/a6d22e86-2f6c-4f1d-8631-b3e8de9ed841


----

### 1. Build
Requires the system to have `arm-none-eabi-gcc` and `make` installed. Run the following commands in the terminal:
```bash
cd Debug
make
