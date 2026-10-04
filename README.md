# Robot v2

Tracked mobile robot platform based on the **STM32F407 Discovery** and **FreeRTOS**.

> **Status:** chassis assembled; motor-control wiring, encoder integration and sensors are the next development steps.

![Robot V2 prototype](docs/Oct%204,%202026,%2009_45_28%20PM.jpg)

## Project goal

Robot V2 is a practical embedded-systems project for developing and testing:

- STM32 firmware in C
- FreeRTOS task-based architecture
- DC motor control
- encoder feedback
- UART / I2C / SPI communication
- sensor integration
- debugging with the onboard ST-LINK
- later communication with a Linux computer for higher-level control

## Current hardware prototype

The first tracked chassis has been assembled.

| Component | Current setup |
|---|---|
| MCU board | STM32F407G-DISC1 Discovery |
| Drive | Tracked differential drive |
| Motors | 2 × geared DC motors |
| Motor drivers | 2 × IBT-2 / BTS7960 H-bridge modules |
| Mechanical platform | Custom tracked chassis with removable electronics plate |
| Power | Battery-powered; power distribution still being finalized |
| Debug/programming | Onboard ST-LINK |

### Prototype views

![Robot V2 chassis](docs/Oct%204,%202026,%2009_45_14%20PM.jpg)

![Robot V2 top view](docs/Oct%204,%202026,%2009_45_50%20PM.jpg)

## Firmware

The project is generated with **STM32CubeMX / STM32CubeIDE** and currently uses:

- STM32 HAL
- FreeRTOS
- CMSIS-RTOS V2
- UART debugging
- onboard Discovery LEDs for diagnostics

The current firmware is still an early hardware-test version. Motor-control code will be adapted to the final dual-driver wiring as the chassis integration progresses.

## Current development steps

1. Finalize power distribution
2. Mount and connect the STM32F407 Discovery securely
3. Connect both motor drivers
4. Configure PWM and direction control
5. Add motor encoder inputs
6. Create separate FreeRTOS tasks for motor control and sensor processing
7. Add distance and orientation sensors
8. Add communication with the Linux computer
9. Implement closed-loop speed control

## Planned software architecture

```text
                Linux computer
                      |
               UART / USB / CAN
                      |
              STM32F407 Discovery
                      |
              +-------+-------+
              |               |
          Motor task      Sensor task
              |               |
       PWM / direction    I2C / GPIO
              |               |
        Motor drivers        Sensors
              |
           Motors
              |
           Encoders
```

The STM32 is intended to handle time-critical low-level control, while the Linux computer will later be used for higher-level functions such as the web interface, navigation and camera processing.

## Development notes

During early bench testing, several useful hardware/software issues were identified:

- GPIO reset states can briefly affect connected motor-control inputs.
- Special-function/JTAG pins should be avoided for motor-control signals unless intentionally configured.
- All controller and motor-driver grounds must share a common reference.
- Power banks may switch off when the load current is too low; a dedicated regulator is preferable for the final robot.

## Repository structure

```text
Robot_v2/
├── Core/
│   ├── Inc/
│   └── Src/
├── Drivers/
├── Middlewares/
│   └── Third_Party/FreeRTOS/
├── docs/
│   └── prototype photos
└── README.md
```

## Roadmap

- [x] Build tracked chassis
- [x] Install geared motors
- [x] Install motor-driver modules
- [x] Prepare STM32F407 Discovery firmware project
- [x] Enable FreeRTOS
- [x] Verify UART debug output
- [ ] Finalize STM32 mounting/interface board
- [ ] Wire both motor drivers
- [ ] Implement PWM motor control
- [ ] Integrate encoders
- [ ] Add distance sensors
- [ ] Add IMU
- [ ] Add Linux-to-STM32 communication
- [ ] Implement closed-loop drive control
