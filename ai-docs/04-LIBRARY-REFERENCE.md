# ArduPilot Library Reference

A categorized reference of all 153+ libraries in the ArduPilot codebase.

## Control Libraries (AC_*)

These libraries implement control algorithms, primarily for Copter:

| Library | Description | Key Files |
|---------|-------------|-----------|
| **AC_AttitudeControl** | Core attitude (roll/pitch/yaw) controller | `AC_AttitudeControl.h/cpp` |
| **AC_PosControl** | 3D position controller | `AC_PosControl.h/cpp` |
| **AC_WPNav** | Waypoint navigation and spline paths | `AC_WPNav.h/cpp` |
| **AC_Loiter** | Loiter position holding | `AC_Loiter.h/cpp` |
| **AC_Circle** | Circle navigation | `AC_Circle.h/cpp` |
| **AC_PID** | PID controller implementations | `AC_PID.h/cpp`, `AC_P.h/cpp` |
| **AC_AutoTune** | Automatic PID tuning | `AC_AutoTune.h/cpp` |
| **AC_Avoidance** | Collision avoidance algorithms | `AC_Avoid.h/cpp` |
| **AC_Fence** | Geofencing boundaries | `AC_Fence.h/cpp` |
| **AC_PrecLand** | Precision landing with IR/vision | `AC_PrecLand.h/cpp` |
| **AC_InputManager** | Pilot input processing | `AC_InputManager.h/cpp` |
| **AC_Sprayer** | Agricultural sprayer control | `AC_Sprayer.h/cpp` |
| **AC_Autorotation** | Helicopter autorotation | `AC_Autorotation.h/cpp` |
| **AC_CustomControl** | Custom controller interface | `AC_CustomControl.h/cpp` |

## Sensor Libraries (AP_*)

### Inertial/Motion
| Library | Description |
|---------|-------------|
| **AP_InertialSensor** | IMU driver framework (accel/gyro) |
| **AP_Compass** | Magnetometer driver framework |
| **AP_Baro** | Barometric pressure sensors |
| **AP_OpticalFlow** | Optical flow sensors |
| **AP_GyroFFT** | FFT analysis for motor harmonics |
| **AP_AccelCal** | Accelerometer calibration |

### Position/Navigation
| Library | Description |
|---------|-------------|
| **AP_GPS** | GPS/GNSS receivers (30+ supported) |
| **AP_RangeFinder** | Distance sensors (lidar, sonar, radar) |
| **AP_Airspeed** | Airspeed sensors |
| **AP_Beacon** | Indoor positioning systems |
| **AP_VisualOdom** | Visual odometry systems |
| **AP_Proximity** | Proximity/360 sensors |

### Environmental
| Library | Description |
|---------|-------------|
| **AP_WindVane** | Wind direction sensing |
| **AP_Temperature** | Temperature sensors |

## State Estimation

| Library | Description |
|---------|-------------|
| **AP_AHRS** | Attitude/Heading Reference System interface |
| **AP_NavEKF2** | Extended Kalman Filter v2 |
| **AP_NavEKF3** | Extended Kalman Filter v3 (current default) |
| **AP_DAL** | Data Abstraction Layer for EKF |
| **AP_Declination** | Magnetic declination database |

## Motor/Actuator Control

| Library | Description |
|---------|-------------|
| **AP_Motors** | Base motor control + multicopter mixing |
| **AP_MotorsHeli** | Helicopter rotor control |
| **AP_MotorsTri** | Tricopter motor control |
| **SRV_Channel** | Servo channel management |
| **AP_BLHeli** | BLHeli ESC passthrough |
| **AP_ESC_Telem** | ESC telemetry |
| **AP_LandingGear** | Landing gear servo control |
| **AP_Winch** | Winch/tether control |
| **AP_Gripper** | Gripper mechanisms |
| **AP_Parachute** | Parachute deployment |

## Communication

### MAVLink/GCS
| Library | Description |
|---------|-------------|
| **GCS_MAVLink** | MAVLink protocol implementation |
| **AP_SerialManager** | Serial port allocation |

### Telemetry Protocols
| Library | Description |
|---------|-------------|
| **AP_Frsky_Telem** | FrSky SmartPort/S.Port |
| **AP_LTM_Telem** | LTM (Lightweight TeleMachinemachine) |
| **AP_Devo_Telem** | Devo protocol |
| **AP_IBus_Telem** | FlySky IBus telemetry |
| **AP_CRSF_Telem** | TBS Crossfire |
| **AP_MSP** | Betaflight MSP protocol |

### CAN Bus
| Library | Description |
|---------|-------------|
| **AP_CANManager** | CAN bus management |
| **AP_DroneCAN** | DroneCAN/UAVCAN protocol |

### Network
| Library | Description |
|---------|-------------|
| **AP_Networking** | Ethernet/WiFi networking |
| **AP_HAL_ESP32** | ESP32 WiFi/BLE |

## Navigation & Mission

| Library | Description |
|---------|-------------|
| **AP_Mission** | Mission command storage/execution |
| **AP_Rally** | Rally point management |
| **AP_Terrain** | Terrain following/avoidance |
| **AP_SmartRTL** | Smart Return-To-Launch |
| **AP_Follow** | Follow-me mode |
| **AP_Avoidance** | Inter-vehicle collision avoidance |

## Safety Systems

| Library | Description |
|---------|-------------|
| **AP_Arming** | Pre-arm checks and arming |
| **AP_AdvancedFailsafe** | Advanced failsafe behaviors |
| **AP_BattMonitor** | Battery monitoring and failsafe |
| **AP_LeakDetector** | Leak detection (submarines) |
| **AP_InternalError** | Internal error tracking |

## Vehicle-Specific

### Plane
| Library | Description |
|---------|-------------|
| **APM_Control** | Fixed-wing attitude control |
| **AP_L1_Control** | L1 guidance algorithm |
| **AP_TECS** | Total Energy Control System |
| **AP_Landing** | Automatic landing |
| **AP_SoarAlg** | Thermal soaring |

### Rover
| Library | Description |
|---------|-------------|
| **AR_AttitudeControl** | Rover attitude control |
| **AR_Motors** | Rover motor control |
| **AP_Torqeedo** | Torqeedo motor support |

### Helicopter
| Library | Description |
|---------|-------------|
| **AP_MotorsHeli_Single** | Single rotor heli |
| **AP_MotorsHeli_Dual** | Dual rotor heli |
| **AP_MotorsHeli_Quad** | Quad heli |

## Hardware Abstraction Layer

| Library | Description |
|---------|-------------|
| **AP_HAL** | Abstract HAL interface |
| **AP_HAL_ChibiOS** | ChibiOS RTOS implementation |
| **AP_HAL_Linux** | Linux implementation |
| **AP_HAL_ESP32** | ESP32 implementation |
| **AP_HAL_SITL** | Simulation implementation |
| **AP_HAL_Empty** | Stub implementation |
| **AP_HAL_QURT** | Qualcomm QURT implementation |

## Core Infrastructure

| Library | Description |
|---------|-------------|
| **AP_Common** | Common utilities, locations |
| **AP_Math** | Vector/matrix math |
| **AP_Param** | Parameter system |
| **AP_Scheduler** | Task scheduling |
| **AP_Vehicle** | Base vehicle class |
| **StorageManager** | Non-volatile storage |
| **AP_Logger** | DataFlash logging |
| **AP_BoardConfig** | Board-specific configuration |
| **AP_IOMCU** | IO co-processor communication |
| **AP_RTC** | Real-time clock |

## Filters & Math

| Library | Description |
|---------|-------------|
| **Filter** | Low-pass, notch filters |
| **AP_Math** | Vectors, matrices, quaternions |

## Peripherals & Accessories

| Library | Description |
|---------|-------------|
| **AP_Camera** | Camera trigger/control |
| **AP_Mount** | Gimbal/mount control |
| **AP_Notify** | LED/buzzer notifications |
| **AP_OSD** | On-screen display |
| **AP_Button** | Button input handling |
| **AP_Relay** | Relay control |
| **AP_RPM** | RPM sensing |
| **AP_RCMapper** | RC channel mapping |

## External Systems

| Library | Description |
|---------|-------------|
| **AP_ExternalAHRS** | External AHRS integration |
| **AP_ExternalControl** | External controller interface |
| **AP_ADSB** | ADS-B traffic avoidance |
| **AP_AIS** | AIS for maritime |
| **AP_EFI** | Electronic fuel injection |
| **AP_Generator** | Generator monitoring |

## Scripting

| Library | Description |
|---------|-------------|
| **AP_Scripting** | Lua scripting engine |
| **AP_Scripting/bindings** | Lua API bindings |

## Simulation

| Library | Description |
|---------|-------------|
| **SITL** | Software-in-the-loop framework |
| **SIM_*** | Vehicle physics models |

## File System

| Library | Description |
|---------|-------------|
| **AP_Filesystem** | Virtual filesystem |
| **AP_FlashStorage** | Flash storage driver |

## Miscellaneous

| Library | Description |
|---------|-------------|
| **AP_Stats** | Flight statistics |
| **AP_Tuning** | In-flight tuning |
| **AP_Volz** | Volz servo protocol |
| **AP_FETtecOneWire** | FETtec OneWire ESC |

## How Libraries Connect

```
┌─────────────────────────────────────────────────────────────┐
│                    Vehicle (Copter/Plane/etc)                │
│              Uses: AP_Vehicle, AP_Arming, AP_Mission         │
└─────────────────────────────┬───────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
   ┌────▼────┐          ┌────▼────┐          ┌────▼────┐
   │ Control │          │ Sensors │          │ Comms   │
   │Libraries│          │Libraries│          │Libraries│
   │ AC_*    │          │ AP_*    │          │ GCS_*   │
   └────┬────┘          └────┬────┘          └────┬────┘
        │                    │                    │
        └─────────────┬──────┴────────────────────┘
                      │
              ┌───────▼───────┐
              │    AP_HAL    │
              │  (Interface)  │
              └───────┬───────┘
                      │
    ┌─────────────────┼─────────────────┐
    │                 │                 │
┌───▼───┐        ┌───▼───┐        ┌───▼───┐
│ChibiOS│        │ Linux │        │ SITL  │
│  HAL  │        │  HAL  │        │  HAL  │
└───────┘        └───────┘        └───────┘
```

## Key Dependencies

Most libraries depend on:
- `AP_HAL` - Hardware access
- `AP_Param` - Configuration parameters
- `AP_Math` - Mathematical operations
- `AP_Common` - Common utilities

Control libraries additionally depend on:
- `AP_AHRS` - Attitude estimation
- `AP_InertialSensor` - IMU data
- `AP_Motors` or `SRV_Channel` - Actuator output
