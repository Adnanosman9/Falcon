<h1 align="center">
  Falcon
  <br>
</h1>

<h4 align="center">
A custom flight controller built from scratch for a 3D surveying drone, running a custom port of ArduPilot
</h4>

<div align="center">

![STM32](https://img.shields.io/badge/STM32H743VIT6-03234B?style=for-the-badge&logo=stmicroelectronics&logoColor=white)
![ArduPilot](https://img.shields.io/badge/ArduPilot-ED1C24?style=for-the-badge)
![KiCad](https://img.shields.io/badge/KiCad-314CB0?style=for-the-badge&logo=kicad&logoColor=white)

</div>

## Overview

I wanted to build a drone that could do proper 3D photogrammetry surveys. Off-the-shelf F4 flight controllers don't have enough processing power for the EKF configuration I need when fusing RTK GPS with IMU data at high rates. So I designed my own.

This is a flight controller built around the **STM32H743VIT6** , a 480MHz Cortex-M7 chip with 2MB flash and 1MB RAM. It runs a custom-compiled ArduPilot firmware with pin mappings specific to this board.

> ⚠️ PCB has not been manufactured yet. Firmware compiles and is ready to flash, but nothing has been tested on real hardware.

---

## Hardware Specs

| Component | Part |
| :--- | :--- |
| MCU | STM32H743VIT6 (480MHz, 2MB Flash) |
| IMU | BMI088 (6-axis, vibration-robust) |
| Barometer | BMP390 |
| Magnetometer | BMM350 |
| Flash Logging | FM25V02A (256KB FRAM) |
| SD Card | SDMMC1 4-bit interface |
| Power Input | MP1584EN buck → LP5907 3.3V LDO |
| USB | USB-C with USBLC6-2SC6 ESD protection |

---

## Why I Built It

I wanted to understand how flight controllers actually work, not just use one. Designing the schematic, figuring out why the VCAP pins need specific 2.2µF caps, getting the crystal load capacitors right, debugging the ArduPilot hwdef syntax , all of that taught me things I wouldn't have learned any other way.

Also, most hobbyist FCs are F4-based. The H7 gives me enough headroom to run ArduPilot's EKF3 with RTK GPS fusion without the processor struggling.

---

## How I Built It

### Schematic (KiCad)
- Designed the full power tree: LiPo → MP1584EN (5V buck) → LP5907 (3.3V LDO)
- VCAP1 and VCAP2 need exactly 2.2µF ceramic caps or the STM32H7 internal LDO becomes unstable, took a while to figure that out
- USB-C CC pins need 5.1kΩ pull-downs to GND or no charger recognizes the board as a device

### PCB Layout (KiCad)
- 4-layer board with bottom copper GND pour
- Crystal section has an isolated GND island underneath to avoid noise coupling into the oscillator
- DRC ended with 9 warnings, all on USB-C pad spacing (DRC settings issue, not a real problem). Zero footprint errors.

### Firmware (ArduPilot)
- Created a custom `hwdef.dat` that maps all STM32 pins to physical components: SPI for BMI088, I2C for BMP390 and BMM350, SDMMC1 for SD card, USART3 for GPS, PWM on TIM1
- Had to build a custom ChibiOS bootloader before the main firmware would compile
- Built on WSL (Ubuntu) using the `waf` build system and `arm-none-eabi-gcc`

---

## Current Status

- ✅ Schematic complete
- ✅ PCB layout complete, DRC clean
- ✅ Custom ArduPilot firmware builds successfully
- ⏳ PCB not manufactured yet
- ⏳ Not tested on real hardware

---

## How to Build Firmware

```bash
git clone --recurse-submodules https://github.com/ArduPilot/ardupilot.git
cd ardupilot
Tools/environment_install/install-prereqs-ubuntu.sh -y
. ~/.profile
mkdir libraries/AP_HAL_ChibiOS/hwdef/H7-DroneFC
cp Firmware/hwdef.dat libraries/AP_HAL_ChibiOS/hwdef/H7-DroneFC/
python Tools/scripts/build_bootloaders.py H7-DroneFC
./waf configure --board=H7-DroneFC
./waf copter
```

Output: `build/H7-DroneFC/bin/arducopter.bin`

---

## How to Flash

Connect an **ST-Link V2** to the SWD header:

| Header Pin | Signal |
| :--- | :--- |
| 1 | +3V3 |
| 2 | SWDIO (PA13) |
| 3 | SWCLK (PA14) |
| 4 | GND |

Flash `arducopter.bin` using STM32CubeProgrammer or Mission Planner (Load custom firmware).

---

## PWM Motor Outputs

| Output | MCU Pin | Timer |
| :--- | :--- | :--- |
| Motor 1 | PE9 | TIM1_CH1 |
| Motor 2 | PE11 | TIM1_CH2 |
| Motor 3 | PE13 | TIM1_CH3 |
| Motor 4 | PE14 | TIM1_CH4 |

---

## Repository Structure

```
H7-DroneFC/
├── PCB/     # KiCad schematic (.kicad_sch) and PCB layout (.kicad_pcb)
├── Firmware/     # hwdef.dat for ArduPilot custom board build
└── Images/       # Schematic screenshots and PCB renders
```

---

## ⚠️ Caution

Firmware has not been tested on real hardware yet. There may be issues with sensor initialization, SPI timing, or I2C addresses that only show up on actual hardware. Double check all pin assignments against the schematic before flashing and expect some debugging on first boot.

---

## Credits

- [ArduPilot](https://ardupilot.org/) for the open-source flight stack
- [KiCad](https://www.kicad.org/) for PCB design
- [Hack Club](https://hackclub.com/) for support

---

> GitHub [@Adnanosman9](https://github.com/Adnanosman9)
