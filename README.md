# BLDC Motor Control — STM32G431RB

UART-commanded dual DC/BLDC motor controller running on the NUCLEO-G431RB development board. Direction and speed of two independent motors are controlled by sending single ASCII characters over a serial link.

## Hardware

| Item | Value |
|------|-------|
| MCU | STM32G431RBTx (Cortex-M4, LQFP64) |
| Board | NUCLEO-G431RB |
| Clock | 16 MHz (HSI, no PLL) |
| Toolchain | STM32CubeIDE + STM32CubeMX 6.7.0 |
| Firmware pack | STM32Cube FW_G4 V1.5.1 |

### Pin Mapping

| Pin | Function | Description |
|-----|----------|-------------|
| PA0 | TIM2 CH1 (PWM) | Speed control output |
| PA2 | LPUART1 TX | ST-LINK Virtual COM Port |
| PA3 | LPUART1 RX | ST-LINK Virtual COM Port |
| PA5 | GPIO Output | LD2 (green LED) |
| PC4 | USART1 TX | External UART TX |
| PA10 | USART1 RX | External UART RX (command input) |
| PC8 | GPIO Output | Motor 1 — direction pin A |
| PC9 | GPIO Output | Motor 1 — direction pin B |
| PC5 | GPIO Output | Motor 2 — direction pin A |
| PC6 | GPIO Output | Motor 2 — direction pin B |
| PC13 | GPIO Input (EXTI) | B1 blue push button |

## Peripherals

- **USART1** — 9600 baud, 8N1. Receives motor command characters from an external host (e.g. Bluetooth module, USB-UART adapter, or another MCU).
- **LPUART1** — 9600 baud, 8N1 on the ST-LINK VCP. Available for debug output.
- **TIM2 CH1** — PWM on PA0. Prescaler = 15999, period = 100 at 16 MHz → 10 Hz PWM. Duty cycle is set directly via `CCR1` (0–100).

## Command Protocol

Send a single ASCII character to USART1 (9600 baud, 8N1) to control the motors.

### Motor 1 (PC8 / PC9)

| Character | Action |
|-----------|--------|
| `A` | Forward |
| `B` | Reverse |
| `S` | Stop |

### Motor 2 (PC5 / PC6)

| Character | Action |
|-----------|--------|
| `C` | Forward |
| `D` | Reverse |
| `F` | Stop |

### Global

| Character | Action |
|-----------|--------|
| `X` | Stop both motors |

### Speed Control

| Character | Duty Cycle (CCR1) |
|-----------|-------------------|
| `0` | 10% |
| `1` | 20% |
| `2` | 30% |
| `3` | 40% |
| `4` | 50% |
| `5` | 60% |
| `6` | 70% |
| `7` | 80% |
| `8` | 90% |
| `9` | 100% |

Speed commands apply to the PWM output shared by both motors. Send a speed command before or after a direction command — the PWM runs continuously once started.

## Building and Flashing

1. Open STM32CubeIDE and import the project (`File → Import → Existing Projects into Workspace`).
2. Select the `BLDC_MOTOR_CONTROL` directory.
3. Build with **Project → Build All** (or `Ctrl+B`).
4. Connect the NUCLEO board via USB.
5. Flash with **Run → Debug** or **Run → Run** using the built-in ST-LINK.

To regenerate peripheral init code from the `.ioc` file, open `final project.ioc` in STM32CubeMX and click **Generate Code**.

## Project Structure

```
BLDC_MOTOR_CONTROL/
├── Core/
│   ├── Inc/
│   │   ├── main.h              # Pin definitions and exported prototypes
│   │   ├── stm32g4xx_hal_conf.h
│   │   └── stm32g4xx_it.h
│   ├── Src/
│   │   ├── main.c              # Application logic and motor control loop
│   │   ├── stm32g4xx_hal_msp.c # HAL MSP (peripheral low-level init)
│   │   ├── stm32g4xx_it.c      # Interrupt handlers
│   │   ├── system_stm32g4xx.c  # System clock init
│   │   ├── syscalls.c          # Newlib syscall stubs
│   │   └── sysmem.c            # Heap implementation
│   └── Startup/
│       └── startup_stm32g431rbtx.s  # ARM startup assembly
├── Drivers/
│   ├── CMSIS/                  # ARM CMSIS headers
│   └── STM32G4xx_HAL_Driver/   # STM32 HAL library
├── final project.ioc           # STM32CubeMX project configuration
├── .project                    # STM32CubeIDE Eclipse project file
└── .cproject                   # CDT managed build configuration
```

## License

Driver code is licensed under ST's BSD-style license — see `Drivers/STM32G4xx_HAL_Driver/LICENSE.txt` and `Drivers/CMSIS/LICENSE.txt` for details.
