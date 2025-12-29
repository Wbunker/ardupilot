# ArduPilot Codebase Architecture

## Directory Structure Overview

```
ardupilot/
├── ArduCopter/          # Multirotor/helicopter autopilot
├── ArduPlane/           # Fixed-wing autopilot
├── Rover/               # Ground vehicle autopilot
├── ArduSub/             # Underwater vehicle autopilot
├── AntennaTracker/      # Antenna tracking system
├── Blimp/               # Airship control
│
├── libraries/           # 153+ shared libraries (THE HEART OF THE PROJECT)
│   ├── AP_HAL*/         # Hardware Abstraction Layer implementations
│   ├── AC_*/            # Control libraries (Copter-focused)
│   ├── AP_*/            # ArduPilot common libraries
│   ├── AR_*/            # Rover-specific libraries
│   └── SITL/            # Software-in-the-loop simulation
│
├── Tools/               # Development utilities
│   ├── ardupilotwaf/    # Build system extensions
│   ├── autotest/        # Automated testing framework
│   ├── AP_Bootloader/   # Firmware bootloader
│   ├── AP_Periph/       # CAN peripheral firmware
│   └── Replay/          # Log replay tool
│
├── modules/             # Git submodules
│   ├── ChibiOS/         # Real-time OS for embedded
│   ├── DroneCAN/        # CAN protocol implementation
│   ├── mavlink/         # Communication protocol
│   └── waf/             # Build system
│
├── tests/               # Unit tests
├── docs/                # Documentation
└── .github/workflows/   # CI/CD pipelines
```

## Core Architecture Layers

### Layer 1: Vehicle Implementation (Top Level)

Each vehicle type has its own directory with vehicle-specific code:

```
ArduCopter/
├── Copter.h/cpp         # Main vehicle class
├── mode_*.cpp           # Flight mode implementations
├── control_*.cpp        # Control functions
├── motors.cpp           # Motor mixing
├── Parameters.h/cpp     # Vehicle parameters
├── defines.h            # Compile-time configuration
├── config.h             # Build options
└── wscript              # Build configuration
```

**Key Pattern**: Each vehicle inherits from `AP_Vehicle` base class and implements vehicle-specific logic.

### Layer 2: Shared Libraries (libraries/)

The libraries folder contains 153+ reusable components:

#### Control Libraries (AC_*)
| Library | Purpose |
|---------|---------|
| AC_AttitudeControl | Roll/pitch/yaw control |
| AC_PosControl | Position control (XY and Z) |
| AC_WPNav | Waypoint navigation |
| AC_Loiter | Loiter mode control |
| AC_AutoTune | Automatic PID tuning |
| AC_PID | PID controller implementations |
| AC_Fence | Geofencing |
| AC_Avoidance | Obstacle avoidance |
| AC_PrecLand | Precision landing |

#### Sensor Libraries (AP_*)
| Library | Purpose |
|---------|---------|
| AP_InertialSensor | IMU (accelerometer/gyro) |
| AP_Compass | Magnetometer |
| AP_Baro | Barometric pressure |
| AP_GPS | GPS/GNSS receivers |
| AP_RangeFinder | Distance sensors (lidar, sonar) |
| AP_OpticalFlow | Optical flow sensors |
| AP_Airspeed | Airspeed measurement |
| AP_Beacon | Indoor positioning beacons |

#### State Estimation
| Library | Purpose |
|---------|---------|
| AP_AHRS | Attitude/Heading Reference System |
| AP_NavEKF2 | Extended Kalman Filter v2 |
| AP_NavEKF3 | Extended Kalman Filter v3 |
| AP_DAL | Data Abstraction Layer (for EKF) |

#### Communication
| Library | Purpose |
|---------|---------|
| GCS_MAVLink | MAVLink protocol handler |
| AP_SerialManager | Serial port management |
| AP_Frsky_Telem | FrSky telemetry |
| AP_DroneCAN | CAN bus communication |
| AP_MSP | Betaflight MSP protocol |

#### Core Infrastructure
| Library | Purpose |
|---------|---------|
| AP_HAL | Hardware Abstraction Layer interface |
| AP_Param | Parameter system |
| AP_Scheduler | Task scheduling |
| AP_Logger | Dataflash logging |
| AP_Vehicle | Base vehicle class |
| AP_Common | Shared utilities |
| AP_Math | Math functions (vectors, matrices) |
| Filter | Signal filtering |

#### Motor/Actuator Control
| Library | Purpose |
|---------|---------|
| AP_Motors | Multicopter motor control |
| AP_MotorsHeli | Helicopter rotor control |
| SRV_Channel | Servo/PWM output |
| AP_BLHeli | BLHeli ESC passthrough |

### Layer 3: Hardware Abstraction Layer (AP_HAL)

The HAL provides a consistent interface across all supported hardware:

```
libraries/
├── AP_HAL/              # Abstract interface definitions
├── AP_HAL_ChibiOS/      # Embedded boards (Pixhawk, Cube, etc.)
├── AP_HAL_Linux/        # Linux SBCs (Raspberry Pi, Navio2)
├── AP_HAL_ESP32/        # ESP32 microcontrollers
├── AP_HAL_SITL/         # Software simulation
├── AP_HAL_QURT/         # Qualcomm processors
└── AP_HAL_Empty/        # Stub implementation
```

#### HAL Interface Components

```cpp
// libraries/AP_HAL/HAL.h
class HAL {
public:
    AP_HAL::UARTDriver*      serial[N];     // Serial ports
    AP_HAL::I2CDeviceManager* i2c_mgr;      // I2C bus
    AP_HAL::SPIDeviceManager* spi;          // SPI bus
    AP_HAL::RCInput*         rcin;          // RC input
    AP_HAL::RCOutput*        rcout;         // PWM output
    AP_HAL::Scheduler*       scheduler;      // Task scheduling
    AP_HAL::Storage*         storage;        // Persistent storage
    AP_HAL::GPIO*            gpio;          // GPIO pins
    // ... more interfaces
};
```

### Layer 4: Operating System / Hardware

- **ChibiOS**: Real-time OS for embedded (Pixhawk, etc.)
- **Linux**: For single-board computers
- **SITL**: Host OS for simulation

## Data Flow Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Vehicle Layer                             │
│  (ArduCopter/ArduPlane/Rover - Mission Logic, Flight Modes)     │
└───────────────────────────┬─────────────────────────────────────┘
                            │
           ┌────────────────┼────────────────┐
           │                │                │
    ┌──────▼──────┐  ┌─────▼──────┐  ┌──────▼──────┐
    │  Position   │  │  Attitude  │  │  Navigation │
    │  Control    │  │  Control   │  │  (Missions) │
    │ (AC_PosCtl) │  │(AC_AttCtl) │  │ (AC_WPNav)  │
    └──────┬──────┘  └─────┬──────┘  └──────┬──────┘
           │               │                │
           └───────────────┼────────────────┘
                           │
                    ┌──────▼──────┐
                    │  AP_Motors  │
                    │ (Motor Mix) │
                    └──────┬──────┘
                           │
┌──────────────────────────┼──────────────────────────────────────┐
│                          │         AP_HAL                        │
│   ┌──────────────────────▼───────────────────────────────────┐  │
│   │                     SRV_Channel                           │  │
│   │                   (PWM Output)                            │  │
│   └──────────────────────┬───────────────────────────────────┘  │
└──────────────────────────┼──────────────────────────────────────┘
                           │
                    ┌──────▼──────┐
                    │   ESCs /    │
                    │   Motors    │
                    └─────────────┘
```

## Sensor Data Flow

```
Physical Sensors
      │
      ▼
┌─────────────────────────────────────────────────────────────┐
│                         AP_HAL                               │
│  (I2C, SPI, Serial drivers for specific hardware)           │
└─────────────────────────┬───────────────────────────────────┘
                          │
      ┌───────────────────┼───────────────────┐
      │                   │                   │
┌─────▼─────┐      ┌─────▼─────┐      ┌──────▼──────┐
│ AP_Baro   │      │ AP_Compass│      │AP_InertialS │
│(Pressure) │      │ (Mag)     │      │   (IMU)     │
└─────┬─────┘      └─────┬─────┘      └──────┬──────┘
      │                  │                   │
      └──────────────────┼───────────────────┘
                         │
                  ┌──────▼──────┐
                  │   AP_AHRS   │
                  │   (State    │
                  │  Estimator) │
                  └──────┬──────┘
                         │
                  ┌──────▼──────┐
                  │ AP_NavEKF3  │
                  │ (Extended   │
                  │  Kalman     │
                  │  Filter)    │
                  └──────┬──────┘
                         │
              ┌──────────▼──────────┐
              │ Vehicle Position &  │
              │ Attitude Estimates  │
              └─────────────────────┘
```

## Scheduler System

ArduPilot uses a cooperative scheduler (`AP_Scheduler`) with prioritized tasks:

```cpp
// Example from ArduCopter
const AP_Scheduler::Task Copter::scheduler_tasks[] = {
    // Function           Rate(Hz)  Max Time(us)
    { rc_loop,               100,      130 },
    { throttle_loop,          50,       75 },
    { update_GPS,             50,      200 },
    { update_batt_compass,    10,      120 },
    { read_barometer,         10,       50 },
    { update_altitude,        10,      100 },
    { run_nav_updates,        50,      100 },
    { update_flight_mode,     50,       50 },
    // ... more tasks
};
```

## Parameter System

Parameters are stored persistently and accessible via MAVLink:

```cpp
// Example parameter declaration
const AP_Param::GroupInfo Copter::var_info[] = {
    // @Param: ANGLE_MAX
    // @DisplayName: Angle Max
    // @Description: Maximum lean angle in degrees
    // @Units: cdeg
    // @Range: 1000 8000
    AP_GROUPINFO("ANGLE_MAX", 1, Copter, aparm.angle_max, 4500),

    // More parameters...
    AP_GROUPEND
};
```

## Key Code Patterns

### Vehicle Class Structure
```cpp
class Copter : public AP_Vehicle {
public:
    friend class Mode;
    friend class ModeAltHold;
    // ... more friend classes for modes

    Copter();

    // Main entry points
    void setup() override;
    void loop() override;

private:
    // Subsystems
    AP_InertialSensor ins;
    AP_AHRS ahrs;
    AC_AttitudeControl attitude_control;
    AC_PosControl pos_control;
    AP_Motors *motors;

    // Current mode
    Mode *flightmode;

    // Parameters
    Parameters g;
};
```

### Flight Mode Pattern
```cpp
class ModeAltHold : public Mode {
public:
    bool init(bool ignore_checks) override;
    void run() override;

    bool requires_GPS() const override { return false; }
    bool has_manual_throttle() const override { return false; }

protected:
    const char *name() const override { return "ALT_HOLD"; }
    const char *name4() const override { return "ALTH"; }
};
```

## Hardware Definition Files

Board-specific configuration in `.hwdef` files:

```
# libraries/AP_HAL_ChibiOS/hwdef/CubeBlack/hwdef.dat
MCU STM32F7xx STM32F777xx
FLASH_SIZE_KB 2048

# Serial ports
SERIAL_ORDER OTG1 USART2 USART3 UART4 UART8 UART7 OTG2

# SPI devices
SPIDEV icm20689     SPI1 DEVID1  GYRO_CS   MODE3  2*MHZ  8*MHZ
SPIDEV icm20602     SPI1 DEVID2  GYRO2_CS  MODE3  2*MHZ  8*MHZ

# I2C
I2C_ORDER I2C2 I2C1

# PWM outputs
PWM_OUTPUT_NUM 14
```

## Simulation (SITL)

Software-in-the-loop allows testing without hardware:

```bash
# Run SITL
cd ArduCopter
sim_vehicle.py -w   # Wipe params and start fresh
sim_vehicle.py --map --console  # With map and console
```

SITL includes physics models for:
- Multirotors, planes, helicopters
- Rovers, boats, submarines
- Various sensor types
- Environmental factors (wind, etc.)

## Build System Integration

Each vehicle's `wscript` defines its dependencies:

```python
# ArduCopter/wscript
def build(bld):
    bld.ap_stlib(
        name='ArduCopter_libs',
        ap_vehicle='ArduCopter',
        ap_libraries=[
            'AC_AttitudeControl',
            'AC_InputManager',
            'AC_PosControl',
            'AC_WPNav',
            'AP_Motors',
            # ... 30+ more libraries
        ],
    )

    bld.ap_program(
        program_name='arducopter',
        use='ArduCopter_libs',
    )
```

## Testing Infrastructure

- **Unit Tests**: `tests/` directory
- **SITL Autotest**: `Tools/autotest/` - comprehensive automated testing
- **CI/CD**: GitHub Actions for all platforms
- **Log Replay**: `Tools/Replay/` for analyzing flight logs
