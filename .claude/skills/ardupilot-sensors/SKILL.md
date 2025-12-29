---
name: ardupilot-sensors
description: |
  Expert guidance for ArduPilot sensor libraries - IMU, GPS, compass, barometer, rangefinder,
  airspeed, optical flow, battery monitoring, proximity, and more. Use this skill when:
  (1) Writing code that reads sensor data in ArduPilot
  (2) Adding new sensor drivers or backends
  (3) Understanding sensor library architecture (frontend/backend pattern)
  (4) Debugging sensor health, calibration, or data flow issues
  (5) Integrating sensors with vehicle code or custom applications
  (6) Questions about AP_InertialSensor, AP_GPS, AP_Compass, AP_Baro, RangeFinder,
      AP_Airspeed, AP_OpticalFlow, AP_BattMonitor, AP_Proximity, AP_AHRS, or any AP_ sensor
---

# ArduPilot Sensors Expert

Expert knowledge for working with ArduPilot's sensor libraries.

## Quick Reference

All ArduPilot sensors follow the **Frontend/Backend** pattern:

```cpp
// 1. Get singleton
SensorClass &sensor = AP::sensor();  // or SensorClass::get_singleton()

// 2. Initialize (once at startup)
sensor.init();

// 3. Update in main loop
void loop() {
    sensor.update();  // or read(), periodic()

    if (sensor.healthy()) {
        auto value = sensor.get_value();
    }
}
```

## Core Sensor Libraries

| Library | Singleton | Primary Data |
|---------|-----------|--------------|
| AP_InertialSensor | `AP::ins()` | `get_gyro()`, `get_accel()` |
| AP_GPS | `AP::gps()` | `location()`, `ground_speed()` |
| Compass | `AP::compass()` | `get_field()`, `calculate_heading()` |
| AP_Baro | `AP::baro()` | `get_pressure()`, `get_altitude()` |
| RangeFinder | `AP::rangefinder()` | `distance_orient(rotation)` |
| AP_Airspeed | `AP::airspeed()` | `get_airspeed()` |
| AP_OpticalFlow | `AP::opticalflow()` | `flowRate()`, `bodyRate()` |
| AP_BattMonitor | `AP::battery()` | `voltage()`, `current_amps()` |
| AP_Proximity | `AP::proximity()` | `get_closest_object()` |
| AP_AHRS | `AP::ahrs()` | `roll`, `pitch`, `yaw`, `get_location()` |

## References

- **Complete API Reference**: See [references/sensor-api.md](references/sensor-api.md) for all functions
- **Code Patterns**: See [references/code-patterns.md](references/code-patterns.md) for implementation examples
- **Adding New Drivers**: See [references/new-driver-guide.md](references/new-driver-guide.md)

## Key Architecture Concepts

### Frontend/Backend Pattern

```
Vehicle Code  →  Frontend (AP_Baro)  →  Backend (AP_Baro_BMP280)  →  HAL (I2C/SPI)
                 - Unified API           - Hardware-specific         - Bus access
                 - Multi-sensor mgmt     - Timer callbacks          - Platform abstraction
                 - Calibration           - Data accumulation
```

### Thread Safety

Backends use semaphores for async data collection:
```cpp
// Timer thread (50Hz) - in backend
WITH_SEMAPHORE(_sem);
_pressure_sum += reading;
_pressure_count++;

// Main thread (10Hz) - in update()
WITH_SEMAPHORE(_sem);
_copy_to_frontend(_instance, _pressure_sum/_pressure_count, _temp);
```

### Health Checking

Always check sensor health before using data:
```cpp
if (sensor.healthy()) {
    // Safe to use data
}
```

## Common Tasks

### Read IMU Data
```cpp
AP_InertialSensor &ins = AP::ins();
Vector3f gyro = ins.get_gyro();    // rad/s
Vector3f accel = ins.get_accel();  // m/s²
```

### Get GPS Position
```cpp
AP_GPS &gps = AP::gps();
if (gps.status() >= AP_GPS::GPS_OK_FIX_3D) {
    Location loc = gps.location();
    float speed = gps.ground_speed();  // m/s
}
```

### Read Altitude
```cpp
AP_Baro &baro = AP::baro();
baro.update();
float alt = baro.get_altitude();      // meters relative to cal
float climb = baro.get_climb_rate();  // m/s
```

### Check Distance (Downward Lidar)
```cpp
RangeFinder *rf = AP::rangefinder();
if (rf->has_data_orient(ROTATION_PITCH_270)) {
    float dist = rf->distance_orient(ROTATION_PITCH_270);  // meters
}
```

### Monitor Battery
```cpp
AP_BattMonitor &bat = AP::battery();
float volts = bat.voltage();
float amps;
bat.current_amps(amps);
uint8_t pct;
bat.capacity_remaining_pct(pct);
```

## Scripts

- `scripts/find_sensor_backends.py` - List all backends for a sensor type
- `scripts/sensor_params.py` - Extract parameters for a sensor library

## File Locations

Sensor libraries are in `libraries/`:
- `libraries/AP_InertialSensor/` - IMU
- `libraries/AP_GPS/` - GPS
- `libraries/AP_Compass/` - Magnetometer
- `libraries/AP_Baro/` - Barometer
- `libraries/AP_RangeFinder/` - Distance sensors
- `libraries/AP_Airspeed/` - Airspeed
- `libraries/AP_OpticalFlow/` - Optical flow
- `libraries/AP_BattMonitor/` - Battery
- `libraries/AP_Proximity/` - 360° sensors
- `libraries/AP_AHRS/` - State estimation
