# Robot v2

STM32F407 Discovery based robot with dual DC motor control via L298N driver and FreeRTOS.

## Hardware

| Component | Details |
|---|---|
| MCU board | STM32F407 Discovery |
| Motor driver | L298N (dual H-bridge) |
| Motors | 2x DC motor |
| Motor power | 12V battery pack (separate) |
| Board power | USB power bank |

## Pin connections

### Motor A — L298N Channel A
| L298N pin | STM32 pin | Function |
|---|---|---|
| IN1 | PB0 | Forward |
| IN2 | PB1 | Reverse |
| ENA | jumper | Always enabled (full speed) |

### Motor B — L298N Channel B
| L298N pin | STM32 pin | Function |
|---|---|---|
| IN3 | PC4 | Forward |
| IN4 | PC5 | Reverse |
| ENB | jumper | Always enabled (full speed) |

> **Note:** L298N VSS (5V logic) and GND must be connected to the STM32 board.
> Motor VCC (12V) is powered separately from the battery pack.
> GND of 12V battery and GND of power bank must be joined together.

## Software

- **STM32CubeMX** generated HAL initialization
- **FreeRTOS** (CMSIS-RTOS V2) with two tasks:
  - `defaultTask` — UART status output every 500ms, blinks LD4
  - `motorTask` — motor control sequence

### Motor task sequence
1. Wait 1 second (power supply settling)
2. Both motors forward for 2 seconds
3. Both motors stop
4. Task sleeps indefinitely

### UART debug output (USART2, 115200 baud)
```
Motor: waiting
Motor: forward
Motor: stopped
```

## Known issues and fixes

### PB4/PB5 → PC4/PC5 pin change
Originally IN3/IN4 were on PB4/PB5. PB4 is the JTAG JTRST pin — it has an internal
pull-up after reset, causing motor B to briefly spin before `MX_GPIO_Init()` reconfigures
it. Moved to PC4/PC5 (plain GPIO with no special reset state) to fix this.

### Duplicate `MOTOR_GPIO_Port` define
After the pin change, code had two conflicting defines:
```c
#define MOTOR_GPIO_Port GPIOB  // motor A
#define MOTOR_GPIO_Port GPIOC  // motor B — overwrote the first one
```
The second define silently overrode the first, so motor A (PB0/PB1) received no signals.
Fixed by using separate defines per channel: `MOTOR_A_GPIO_Port` and `MOTOR_B_GPIO_Port`.

### Power bank auto-shutoff
The STM32F407 Discovery alone draws ~100mA. Most power banks cut off below 200–500mA,
causing periodic resets (red power LED blinks, motors restart every ~30–60 seconds).

**Solutions:**
- Use a power bank with "always on" / low-current mode
- Add a 47–100 Ohm resistor between 5V and GND to increase idle current draw
- Use a dedicated LiPo + 5V boost converter without auto-off

## Project structure

```
Robot_v2/
├── Core/
│   ├── Inc/main.h          — pin definitions
│   └── Src/main.c          — main logic, motor task, FreeRTOS setup
├── Drivers/
│   └── STM32F4xx_HAL_Driver/
├── Middlewares/
│   ├── Third_Party/FreeRTOS/
│   └── ST/STM32_USB_Host_Library/
```
