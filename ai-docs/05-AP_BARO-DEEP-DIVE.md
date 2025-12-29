# AP_Baro Library Deep Dive

This document explains how the AP_Baro barometer library works, serving as a model for understanding ArduPilot's sensor library architecture.

## File Organization

```
libraries/AP_Baro/
├── AP_Baro.h              # Frontend class definition (user-facing API)
├── AP_Baro.cpp            # Frontend implementation
├── AP_Baro_Backend.h      # Abstract base class for all drivers
├── AP_Baro_Backend.cpp    # Common backend functionality
├── AP_Baro_config.h       # Compile-time feature flags
│
├── AP_Baro_BMP280.h/cpp   # BMP280 sensor driver
├── AP_Baro_BMP388.h/cpp   # BMP388 sensor driver
├── AP_Baro_MS5611.h/cpp   # MS5611 sensor driver
├── AP_Baro_SITL.h/cpp     # Simulation driver
├── AP_Baro_DroneCAN.h/cpp # CAN bus sensor driver
├── ... (20+ more drivers)
│
├── AP_Baro_atmosphere.cpp # Atmospheric model calculations
├── AP_Baro_Logging.cpp    # DataFlash logging
├── AP_Baro_Wind.cpp       # Wind compensation
│
├── examples/              # Example programs
│   └── BARO_generic/      # Basic usage example
└── tests/                 # Unit tests
```

## Architecture: Frontend/Backend Pattern

ArduPilot uses a **Frontend/Backend** pattern for all sensor libraries:

```
┌─────────────────────────────────────────────────────────────┐
│                    Vehicle Code                              │
│            (ArduCopter, ArduPlane, etc.)                    │
│                                                              │
│    float alt = baro.get_altitude();                         │
│    float pressure = baro.get_pressure();                    │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                   AP_Baro (Frontend)                         │
│                                                              │
│  - Single instance (singleton)                              │
│  - Provides unified API to vehicle code                     │
│  - Manages multiple sensor instances                        │
│  - Handles calibration, altitude calculation                │
│  - Selects primary sensor                                   │
└─────────────────────────┬───────────────────────────────────┘
                          │
          ┌───────────────┼───────────────┐
          │               │               │
          ▼               ▼               ▼
┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│ AP_Baro_    │   │ AP_Baro_    │   │ AP_Baro_    │
│ Backend     │   │ Backend     │   │ Backend     │
│ (BMP280)    │   │ (MS5611)    │   │ (SITL)      │
└──────┬──────┘   └──────┬──────┘   └──────┬──────┘
       │                 │                 │
       ▼                 ▼                 ▼
┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│   AP_HAL    │   │   AP_HAL    │   │  Simulated  │
│  I2C/SPI    │   │  I2C/SPI    │   │   Values    │
│   Device    │   │   Device    │   │             │
└─────────────┘   └─────────────┘   └─────────────┘
```

## Key Classes Explained

### 1. AP_Baro (Frontend) - `AP_Baro.h`

The frontend is what vehicle code interacts with:

```cpp
class AP_Baro {
public:
    // Singleton access - THE way to get the baro object
    static AP_Baro *get_singleton(void);

    // Lifecycle
    void init(void);           // Called once at startup
    void update(void);         // Called every loop iteration
    void calibrate(bool save); // Calibrate on ground

    // Primary readings (from best sensor)
    float get_pressure(void);     // Pascals
    float get_temperature(void);  // Celsius
    float get_altitude(void);     // Meters (relative to calibration)
    float get_climb_rate(void);   // m/s (positive = up)

    // Health
    bool healthy(void);           // Is primary sensor working?
    bool healthy(uint8_t i);      // Is sensor i working?

    // Multi-sensor support
    uint8_t num_instances(void);  // How many sensors?
    float get_pressure(uint8_t i); // Pressure from sensor i

    // For backends to register themselves
    uint8_t register_sensor(void);

private:
    AP_Baro_Backend *drivers[BARO_MAX_DRIVERS]; // Up to 3 drivers

    struct sensor_data {
        float pressure;
        float temperature;
        float altitude;
        bool healthy;
        // ... more fields
    } sensors[BARO_MAX_INSTANCES];  // Up to 3 sensors
};
```

**Key insight**: The frontend stores the actual sensor data. Backends push data *to* the frontend.

### 2. AP_Baro_Backend (Base Class) - `AP_Baro_Backend.h`

Every sensor driver inherits from this:

```cpp
class AP_Baro_Backend {
public:
    AP_Baro_Backend(AP_Baro &baro);

    // MUST implement: copy accumulated data to frontend
    virtual void update() = 0;

protected:
    AP_Baro &_frontend;  // Reference to the frontend

    // Call this to send data to frontend
    void _copy_to_frontend(uint8_t instance,
                           float pressure,
                           float temperature);

    HAL_Semaphore _sem;  // Thread safety
};
```

### 3. Concrete Backend (e.g., BMP280) - `AP_Baro_BMP280.h/cpp`

A real sensor driver:

```cpp
class AP_Baro_BMP280 : public AP_Baro_Backend {
public:
    AP_Baro_BMP280(AP_Baro &baro, AP_HAL::Device &dev);

    // Required: transfer data to frontend
    void update() override;

    // Factory method for auto-detection
    static AP_Baro_Backend *probe(AP_Baro &baro, AP_HAL::Device &dev);

private:
    bool _init(void);           // Configure the sensor
    void _timer(void);          // Periodic read callback (runs in timer thread)
    void _update_pressure(int32_t raw);
    void _update_temperature(int32_t raw);

    AP_HAL::Device *_dev;       // I2C or SPI device handle
    uint8_t _instance;          // Which sensor slot we registered

    // Accumulated readings (written by _timer, read by update)
    float _pressure_sum;
    uint32_t _pressure_count;
    float _temperature;

    // Calibration data read from sensor
    int16_t _t2, _t3, _p2, ...;
};
```

## How It All Works Together

### Initialization Flow

```
1. Vehicle calls: barometer.init()

2. Frontend (AP_Baro::init) probes for hardware:
   - Try SITL backend if simulating
   - Try DroneCAN sensors
   - Try board-specific sensors (from hwdef)
   - Probe external I2C buses

3. For each detected sensor (e.g., BMP280):
   a. AP_Baro_BMP280::probe() is called
   b. Creates new AP_Baro_BMP280 instance
   c. Backend calls _init():
      - Read sensor chip ID
      - Read calibration constants
      - Configure sensor mode
      - _instance = _frontend.register_sensor()  // Get slot number
      - Register periodic timer callback
   d. Backend added to drivers[] array

4. Vehicle calls: barometer.calibrate()
   - Takes ground pressure readings
   - Sets reference for altitude calculations
```

### Runtime Data Flow

```
┌────────────────────────────────────────────────────────────────┐
│                    Timer Thread (50Hz)                          │
│                                                                 │
│  AP_Baro_BMP280::_timer() {                                    │
│      // Read raw sensor data via I2C/SPI                       │
│      dev->read_registers(BMP280_REG_DATA, buf, 6);             │
│                                                                 │
│      // Convert raw to pressure/temp                           │
│      _update_temperature(raw_temp);                            │
│      _update_pressure(raw_press);                              │
│                                                                 │
│      // Accumulate (with semaphore)                            │
│      WITH_SEMAPHORE(_sem);                                     │
│      _pressure_sum += calculated_pressure;                     │
│      _pressure_count++;                                        │
│  }                                                             │
└────────────────────────────────────────────────────────────────┘
                               │
                               │ (runs asynchronously)
                               │
┌────────────────────────────────────────────────────────────────┐
│                    Main Thread (10Hz)                           │
│                                                                 │
│  // Vehicle main loop calls:                                   │
│  barometer.update();                                           │
│                                                                 │
│  // Frontend calls each backend's update():                    │
│  AP_Baro_BMP280::update() {                                    │
│      WITH_SEMAPHORE(_sem);                                     │
│      avg_pressure = _pressure_sum / _pressure_count;           │
│      _copy_to_frontend(_instance, avg_pressure, _temperature); │
│      _pressure_sum = 0;                                        │
│      _pressure_count = 0;                                      │
│  }                                                             │
│                                                                 │
│  // Now vehicle can read:                                      │
│  alt = barometer.get_altitude();                               │
└────────────────────────────────────────────────────────────────┘
```

## Using AP_Baro in Your Code

### Example: Basic Usage

From `examples/BARO_generic/BARO_generic.cpp`:

```cpp
#include <AP_Baro/AP_Baro.h>
#include <AP_HAL/AP_HAL.h>

const AP_HAL::HAL &hal = AP_HAL::get_HAL();

// Create the barometer object (usually done once globally)
static AP_Baro barometer;

void setup() {
    hal.console->printf("Barometer library test\n");

    // Initialize - probes hardware, registers backends
    barometer.init();

    // Calibrate - establishes ground reference
    barometer.calibrate();
}

void loop() {
    // Call update() regularly to get fresh data
    barometer.update();

    // Check health before using
    if (!barometer.healthy()) {
        hal.console->printf("Sensor not healthy!\n");
        return;
    }

    // Read values
    float pressure    = barometer.get_pressure();     // Pascals
    float temperature = barometer.get_temperature();  // Celsius
    float altitude    = barometer.get_altitude();     // Meters
    float climb_rate  = barometer.get_climb_rate();   // m/s

    hal.console->printf("Pressure: %.2f Pa, Temp: %.2f C, Alt: %.2f m\n",
                        pressure, temperature, altitude);
}
```

### Example: Multiple Sensors

```cpp
void read_all_sensors() {
    barometer.update();

    uint8_t num_sensors = barometer.num_instances();

    for (uint8_t i = 0; i < num_sensors; i++) {
        if (barometer.healthy(i)) {
            float pressure = barometer.get_pressure(i);
            float temp     = barometer.get_temperature(i);
            float alt      = barometer.get_altitude(i);

            hal.console->printf("Sensor %d: P=%.1f Pa, T=%.1f C, Alt=%.1f m\n",
                                i, pressure, temp, alt);
        }
    }

    // Primary sensor (auto-selected or user-configured)
    hal.console->printf("Primary: Alt = %.1f m\n", barometer.get_altitude());
}
```

### Example: In a Vehicle (How ArduCopter Uses It)

```cpp
// In ArduCopter/Copter.h
class Copter : public AP_Vehicle {
    AP_Baro barometer;  // Member variable
    // ...
};

// In ArduCopter/system.cpp - called once at startup
void Copter::init_ardupilot() {
    barometer.init();
    barometer.calibrate();
    // ...
}

// In ArduCopter/Copter.cpp - scheduler task
void Copter::read_barometer() {
    barometer.update();

    // Climb rate used for altitude hold
    float climb_rate = barometer.get_climb_rate();

    // Altitude used for terrain following, etc.
    float altitude = barometer.get_altitude();
}
```

## Key Design Patterns to Learn

### 1. Singleton Pattern
```cpp
// Get the global baro instance anywhere in the code
AP_Baro &baro = AP::baro();

// Or using the class method
AP_Baro *baro = AP_Baro::get_singleton();
```

### 2. Probe/Factory Pattern
Backends have a static `probe()` method that:
- Attempts to detect the sensor
- Returns a new instance if found, nullptr if not
- Allows auto-detection of hardware

```cpp
// The frontend tries to probe various sensors:
AP_Baro_Backend *backend = AP_Baro_BMP280::probe(*this, *dev);
if (backend != nullptr) {
    _add_backend(backend);  // Found one!
}
```

### 3. Thread-Safe Data Accumulation
Backends use semaphores because:
- Timer callback runs in a different thread
- Main loop reads data from a different thread

```cpp
// In timer callback (timer thread)
WITH_SEMAPHORE(_sem);
_pressure_sum += press;
_pressure_count++;

// In update() (main thread)
WITH_SEMAPHORE(_sem);
avg = _pressure_sum / _pressure_count;
_copy_to_frontend(_instance, avg, _temperature);
```

### 4. HAL Device Abstraction
Backends don't care if it's I2C or SPI:

```cpp
// Works for both I2C and SPI devices
_dev->read_registers(REG_DATA, buf, 6);
_dev->write_register(REG_CONFIG, value);
```

## Parameters Exposed

The library exposes user-configurable parameters:

| Parameter | Description |
|-----------|-------------|
| `BARO_PRIMARY` | Which sensor to use as primary (0, 1, 2) |
| `BARO_EXT_BUS` | External I2C bus to probe |
| `BARO_ALT_OFFSET` | Manual altitude offset (meters) |
| `BARO_FLTR_RNG` | Noise filter range (%) |
| `BARO1_GND_PRESS` | Calibrated ground pressure |

## Adding Your Own Baro Driver

To add support for a new barometer (e.g., "MySensor"):

1. **Create header** `AP_Baro_MySensor.h`:
```cpp
#pragma once
#include "AP_Baro_Backend.h"

class AP_Baro_MySensor : public AP_Baro_Backend {
public:
    AP_Baro_MySensor(AP_Baro &baro, AP_HAL::Device &dev);
    void update() override;
    static AP_Baro_Backend *probe(AP_Baro &baro, AP_HAL::Device &dev);
private:
    bool _init();
    void _timer();
    AP_HAL::Device *_dev;
    uint8_t _instance;
    float _pressure_sum;
    uint32_t _pressure_count;
    float _temperature;
};
```

2. **Implement** `AP_Baro_MySensor.cpp`:
   - Read chip ID in probe()
   - Read calibration in _init()
   - Read raw data in _timer()
   - Average and copy in update()

3. **Add to probe list** in `AP_Baro.cpp`:
```cpp
probe_i2c_dev(AP_Baro_MySensor::probe, bus, MY_SENSOR_I2C_ADDR);
```

4. **Add config flag** in `AP_Baro_config.h`:
```cpp
#ifndef AP_BARO_MYSENSOR_ENABLED
#define AP_BARO_MYSENSOR_ENABLED 1
#endif
```

## Summary

The AP_Baro library demonstrates ArduPilot's sensor library pattern:

1. **Frontend** provides a unified API and manages multiple backends
2. **Backend base class** defines the interface all drivers must implement
3. **Concrete backends** handle specific sensor hardware
4. **Auto-detection** via probe() factory methods
5. **Thread safety** with semaphores for async data collection
6. **HAL abstraction** for hardware independence

This same pattern is used by:
- `AP_InertialSensor` (IMU)
- `AP_Compass` (magnetometer)
- `AP_GPS` (GNSS)
- `AP_RangeFinder` (distance sensors)
- And many more...

Understanding AP_Baro gives you the template for understanding all ArduPilot sensor libraries!
