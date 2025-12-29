# ArduPilot Sensor API Reference

Complete API reference for all sensor libraries.

## Table of Contents
1. [AP_InertialSensor](#ap_inertialsensor)
2. [AP_GPS](#ap_gps)
3. [Compass](#compass)
4. [AP_Baro](#ap_baro)
5. [RangeFinder](#rangefinder)
6. [AP_Proximity](#ap_proximity)
7. [AP_Airspeed](#ap_airspeed)
8. [AP_OpticalFlow](#ap_opticalflow)
9. [AP_BattMonitor](#ap_battmonitor)
10. [AP_AHRS](#ap_ahrs)
11. [Peripheral Sensors](#peripheral-sensors)

---

## AP_InertialSensor

**Location**: `libraries/AP_InertialSensor/`
**Singleton**: `AP::ins()` or `AP_InertialSensor::get_singleton()`

### Functions

| Function | Returns | Description |
|----------|---------|-------------|
| `init(sample_rate_hz)` | void | Initialize at specified rate (typically 400Hz) |
| `periodic()` | void | Call from main loop |
| `get_gyro()` | `const Vector3f&` | Angular rates in rad/s (primary) |
| `get_gyro(i)` | `const Vector3f&` | Angular rates from instance i |
| `get_accel()` | `const Vector3f&` | Acceleration in m/s² (primary) |
| `get_accel(i)` | `const Vector3f&` | Acceleration from instance i |
| `get_delta_angle(i, delta, dt)` | bool | Integrated angle since last call |
| `get_delta_velocity(i, delta, dt)` | bool | Integrated velocity since last call |
| `get_gyro_health()` | bool | Primary gyro healthy |
| `get_gyro_health(i)` | bool | Gyro i healthy |
| `get_accel_health()` | bool | Primary accel healthy |
| `get_accel_health(i)` | bool | Accel i healthy |
| `get_gyro_count()` | uint8_t | Number of gyros |
| `get_accel_count()` | uint8_t | Number of accels |
| `get_primary_gyro()` | uint8_t | Primary gyro instance |
| `get_primary_accel()` | uint8_t | Primary accel instance |
| `calibrating()` | bool | Currently calibrating |
| `get_accel_clip_count(i)` | uint32_t | Clipping events count |
| `get_gyro_offsets(i)` | `const Vector3f&` | Gyro calibration offsets |

### Supported Backends
ICM20689, ICM20602, ICM20948, ICM42688, BMI055, BMI088, BMI160, BMI270, MPU6000, MPU9250, LSM9DS0, LSM9DS1, ADIS16xxx, DroneCAN, ExternalAHRS, SITL

---

## AP_GPS

**Location**: `libraries/AP_GPS/`
**Singleton**: `AP::gps()` or `AP_GPS::get_singleton()`

### Status Enum
```cpp
enum GPS_Status {
    NO_GPS = 0,              // No GPS detected
    NO_FIX = 1,              // No position fix
    GPS_OK_FIX_2D = 2,       // 2D fix
    GPS_OK_FIX_3D = 3,       // 3D fix
    GPS_OK_FIX_3D_DGPS = 4,  // Differential GPS
    GPS_OK_FIX_3D_RTK_FLOAT = 5,  // RTK float
    GPS_OK_FIX_3D_RTK_FIXED = 6,  // RTK fixed (cm accuracy)
};
```

### Functions

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize GPS |
| `update()` | void | Update state (call at 10Hz+) |
| `status()` | GPS_Status | Primary fix status |
| `status(i)` | GPS_Status | Status of instance i |
| `location()` | `const Location&` | Current position (lat/lon/alt) |
| `location(i)` | `const Location&` | Position from instance i |
| `ground_speed()` | float | Speed over ground (m/s) |
| `ground_course()` | float | Course over ground (degrees) |
| `velocity()` | `const Vector3f&` | 3D velocity NED (m/s) |
| `num_sats()` | uint8_t | Satellite count |
| `num_sats(i)` | uint8_t | Satellites for instance i |
| `hdop()` | uint16_t | Horizontal DOP × 100 |
| `vdop()` | uint16_t | Vertical DOP × 100 |
| `horizontal_accuracy(hacc)` | bool | Horizontal accuracy (m) |
| `vertical_accuracy(vacc)` | bool | Vertical accuracy (m) |
| `speed_accuracy(sacc)` | bool | Speed accuracy (m/s) |
| `have_vertical_velocity()` | bool | Has vertical velocity |
| `gps_yaw_deg(i, yaw)` | bool | GPS heading (degrees) |
| `num_sensors()` | uint8_t | GPS count |
| `primary_sensor()` | uint8_t | Primary instance |
| `time_week()` | uint16_t | GPS week number |
| `time_week_ms()` | uint32_t | ms into GPS week |
| `last_fix_time_ms()` | uint32_t | Time of last fix |

### Location Structure
```cpp
struct Location {
    int32_t lat;   // Latitude in degrees × 10^7
    int32_t lng;   // Longitude in degrees × 10^7
    int32_t alt;   // Altitude in cm (AMSL or relative)
};
```

### Supported Backends
u-blox (M8, F9P), Septentrio SBF, Swift SBP, Trimble, NMEA, DroneCAN, MAVLink, SITL

---

## Compass

**Location**: `libraries/AP_Compass/`
**Singleton**: `AP::compass()` or `Compass::get_singleton()`

### Functions

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize compass |
| `read()` | bool | Read and update values |
| `get_field()` | `const Vector3f&` | Magnetic field (milligauss) |
| `get_field(i)` | `const Vector3f&` | Field from instance i |
| `calculate_heading(dcm)` | float | Heading in radians |
| `calculate_heading(dcm, i)` | float | Heading from instance i |
| `healthy()` | bool | Primary healthy |
| `healthy(i)` | bool | Instance i healthy |
| `get_count()` | uint8_t | Compass count |
| `get_declination()` | float | Magnetic declination (rad) |
| `set_declination(dec)` | void | Set declination |
| `use_for_yaw()` | bool | Used for yaw |
| `use_for_yaw(i)` | bool | Instance i used for yaw |
| `set_offsets(i, offsets)` | void | Set calibration offsets |
| `get_offsets(i)` | `const Vector3f&` | Get offsets |
| `available()` | bool | Enabled and initialized |

### Supported Backends
HMC5843, HMC5883, AK8963, AK09916, LIS3MDL, LIS2MDL, IST8310, IST8308, QMC5883L, BMM150, RM3100, DroneCAN, ExternalAHRS

---

## AP_Baro

**Location**: `libraries/AP_Baro/`
**Singleton**: `AP::baro()` or `AP_Baro::get_singleton()`

### Functions

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize barometer |
| `calibrate(save)` | void | Calibrate ground pressure |
| `update()` | void | Update readings |
| `get_pressure()` | float | Pressure (Pascals) |
| `get_pressure(i)` | float | Pressure from instance i |
| `get_temperature()` | float | Temperature (°C) |
| `get_temperature(i)` | float | Temperature from instance i |
| `get_altitude()` | float | Altitude (m, relative to cal) |
| `get_altitude(i)` | float | Altitude from instance i |
| `get_climb_rate()` | float | Climb rate (m/s) |
| `healthy()` | bool | Primary healthy |
| `healthy(i)` | bool | Instance i healthy |
| `num_instances()` | uint8_t | Barometer count |
| `get_ground_pressure()` | float | Calibrated ground pressure |
| `get_ground_temperature()` | float | Ground temperature (°C) |

### Supported Backends
BMP085, BMP180, BMP280, BMP388, BMP390, MS5611, MS5607, MS5837, SPL06, DPS280, LPS22H, LPS25H, DroneCAN, SITL

---

## RangeFinder

**Location**: `libraries/AP_RangeFinder/`
**Singleton**: `AP::rangefinder()` or `RangeFinder::get_singleton()`

### Status Enum
```cpp
enum class Status {
    NotConnected = 0,
    NoData = 1,
    OutOfRangeLow = 2,
    OutOfRangeHigh = 3,
    Good = 4,
};
```

### Common Orientations
```cpp
ROTATION_NONE          // Forward
ROTATION_PITCH_270     // Down (most common)
ROTATION_PITCH_90      // Up
ROTATION_YAW_180       // Backward
```

### Functions

| Function | Returns | Description |
|----------|---------|-------------|
| `init(orientation)` | void | Initialize with default orientation |
| `update()` | void | Update all rangefinders |
| `num_sensors()` | uint8_t | Sensor count |
| `distance_orient(rot)` | float | Distance for orientation (m) |
| `status_orient(rot)` | Status | Status for orientation |
| `has_orientation(rot)` | bool | Has sensor with orientation |
| `has_data_orient(rot)` | bool | Valid data for orientation |
| `max_distance_orient(rot)` | float | Max range (m) |
| `min_distance_orient(rot)` | float | Min range (m) |
| `signal_quality_pct_orient(rot)` | int8_t | Quality 0-100 |
| `ground_clearance_orient(rot)` | float | Ground clearance (m) |
| `get_pos_offset_orient(rot)` | `const Vector3f&` | Position offset |
| `last_reading_ms(rot)` | uint32_t | Last reading time |
| `prearm_healthy(msg, len)` | bool | Pre-arm check |

### Supported Backends (40+)
LightWare SF02/SF10/SF11/SF40C/SF45B, Benewake TFmini/TF02/TF03/TFLuna, Garmin Lidar-Lite, MaxBotix sonar, TeraRanger, Leddar, PWM, Analog, DroneCAN, SITL

---

## AP_Proximity

**Location**: `libraries/AP_Proximity/`
**Singleton**: `AP::proximity()`

### Status Enum
```cpp
enum class Status {
    NotConnected = 0,
    NoData,
    Good
};
```

### Functions

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize |
| `update()` | void | Update all sensors |
| `get_status()` | Status | Overall status |
| `num_sensors()` | uint8_t | Sensor count |
| `get_horizontal_distances(array)` | bool | Distances by angle |
| `get_closest_object(angle, dist)` | bool | Closest obstacle |
| `get_obstacle_count()` | uint8_t | Obstacle count |
| `get_obstacle(num, vec)` | bool | Vector to obstacle |
| `distance_max_m()` | float | Max detection range |
| `distance_min_m()` | float | Min detection range |
| `get_upward_distance(dist)` | bool | Distance above |
| `sensor_present()` | bool | Sensor present |
| `sensor_enabled()` | bool | Sensor enabled |
| `sensor_failed()` | bool | Sensor failed |

### Supported Backends
RPLidar A2, LightWare SF40C/SF45B, TeraRanger Tower, Cygbot D1, LD06, MR72, DroneCAN, MAVLink, SITL

---

## AP_Airspeed

**Location**: `libraries/AP_Airspeed/`
**Singleton**: `AP::airspeed()`

### Functions

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize |
| `calibrate(in_startup)` | void | Calibrate zero offset |
| `update()` | void | Update readings |
| `get_airspeed()` | float | Indicated airspeed (m/s) |
| `get_airspeed(i)` | float | Airspeed from instance i |
| `get_raw_airspeed()` | float | Unfiltered airspeed |
| `get_differential_pressure()` | float | Pressure diff (Pa) |
| `get_temperature(temp)` | bool | Probe temp (°C) |
| `healthy()` | bool | Primary healthy |
| `healthy(i)` | bool | Instance i healthy |
| `enabled()` | bool | Airspeed enabled |
| `use()` | bool | Use for control |
| `get_airspeed_ratio()` | float | Calibration ratio |
| `last_update_ms()` | uint32_t | Last update time |

### Supported Backends
MS4525DO, MS5525, DLVR, SDP3x, NMEA, Analog, DroneCAN, SITL

---

## AP_OpticalFlow

**Location**: `libraries/AP_OpticalFlow/`
**Singleton**: `AP::opticalflow()`

### Functions

| Function | Returns | Description |
|----------|---------|-------------|
| `init(log_bit)` | void | Initialize |
| `update()` | void | Update readings |
| `enabled()` | bool | Flow enabled |
| `healthy()` | bool | Sensor healthy |
| `quality()` | uint8_t | Surface quality 0-255 |
| `flowRate()` | `const Vector2f&` | Raw flow (rad/s) |
| `bodyRate()` | `const Vector2f&` | IMU-corrected flow (rad/s) |
| `last_update()` | uint32_t | Last update time (ms) |
| `get_pos_offset()` | `const Vector3f&` | Sensor offset |

### Supported Backends
PX4Flow, PMW3901, CXOF, HereFlow, UPFLOW, MAVLink, SITL

---

## AP_BattMonitor

**Location**: `libraries/AP_BattMonitor/`
**Singleton**: `AP::battery()`

### Failsafe Enum
```cpp
enum class Failsafe : uint8_t {
    None = 0,
    Unhealthy,
    Low,
    Critical
};
```

### Functions

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize |
| `read()` | void | Update readings (10Hz) |
| `num_instances()` | uint8_t | Battery count |
| `voltage()` | float | Primary voltage (V) |
| `voltage(i)` | float | Voltage of battery i |
| `current_amps(current, i)` | bool | Current (A) |
| `consumed_mah(mah, i)` | bool | Consumed (mAh) |
| `consumed_wh(wh, i)` | bool | Consumed (Wh) |
| `capacity_remaining_pct(pct, i)` | bool | Remaining % |
| `time_remaining(secs, i)` | bool | Time remaining (s) |
| `pack_capacity_mah(i)` | int32_t | Total capacity (mAh) |
| `healthy()` | bool | All healthy |
| `healthy(i)` | bool | Instance i healthy |
| `get_temperature(temp, i)` | bool | Temp (°C) |
| `has_failsafed()` | bool | Failsafe triggered |

### Supported Backends
Analog, SMBus smart batteries, DroneCAN, FuelFlow, Generator, INA2xx, INA3221, Scripting

---

## AP_AHRS

**Location**: `libraries/AP_AHRS/`
**Singleton**: `AP::ahrs()`

### Functions

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize |
| `update()` | void | Update estimates |
| `get_gyro()` | `const Vector3f&` | Corrected gyro (rad/s) |
| `get_gyro_drift()` | `const Vector3f&` | Drift estimate |
| `get_rotation_body_to_ned()` | `const Matrix3f&` | Rotation matrix |
| `get_quaternion()` | `Quaternion&` | Attitude quaternion |
| `roll` | float | Roll (rad) |
| `pitch` | float | Pitch (rad) |
| `yaw` | float | Yaw (rad) |
| `roll_sensor` | int32_t | Roll (centi-degrees) |
| `pitch_sensor` | int32_t | Pitch (centi-degrees) |
| `yaw_sensor` | int32_t | Yaw (centi-degrees) |
| `get_location(loc)` | bool | Current position |
| `get_velocity_NED(vel)` | bool | Velocity NED (m/s) |
| `groundspeed_vector()` | Vector2f | Ground speed (m/s) |
| `get_hagl(hagl)` | bool | Height AGL (m) |
| `healthy()` | bool | AHRS healthy |
| `wind_estimate()` | `const Vector3f&` | Wind (m/s) |
| `get_error_rp()` | float | Roll/pitch error |
| `get_error_yaw()` | float | Yaw error |
| `airspeed_EAS(airspeed)` | bool | Equivalent airspeed |
| `get_EAS2TAS()` | float | EAS to TAS ratio |

---

## Peripheral Sensors

### AP_TemperatureSensor
**Singleton**: `AP_TemperatureSensor::get_singleton()`

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize |
| `update()` | void | Update readings |
| `get_temperature(temp, i)` | bool | Temperature (°C) |
| `healthy(i)` | bool | Sensor healthy |
| `num_instances()` | uint8_t | Sensor count |

### AP_RPM
**Location**: `libraries/AP_RPM/`

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize |
| `update()` | void | Update readings |
| `get_rpm(i, rpm)` | bool | RPM value |
| `get_signal_quality(i)` | float | Signal quality |
| `num_sensors()` | uint8_t | Sensor count |

### AP_Beacon
**Singleton**: `AP::beacon()`

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize |
| `update()` | void | Update readings |
| `healthy()` | bool | System healthy |
| `get_vehicle_position_ned(pos, acc)` | bool | Position (m) |
| `get_origin(loc)` | bool | System origin |
| `count()` | uint8_t | Beacon count |
| `beacon_distance(i)` | float | Distance (m) |

### AP_VisualOdom
**Singleton**: `AP::visualodom()`

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize |
| `enabled()` | bool | VO enabled |
| `healthy()` | bool | Sensor healthy |
| `quality()` | int8_t | Quality (-1 to 100) |
| `get_pos_offset()` | `const Vector3f&` | Camera offset |

### AP_ESC_Telem
**Singleton**: `AP_ESC_Telem::get_singleton()`

| Function | Returns | Description |
|----------|---------|-------------|
| `get_rpm(i, rpm)` | bool | ESC RPM |
| `get_temperature(i, temp)` | bool | ESC temp (centi-°C) |
| `get_current(i, amps)` | bool | Current (A) |
| `get_voltage(i, volts)` | bool | Voltage (V) |
| `get_average_motor_rpm()` | float | Average RPM |
| `update()` | void | Update (10Hz) |

### AP_LeakDetector (ArduSub)

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize |
| `update()` | bool | Update and return status |
| `get_status()` | bool | Leak detected |

### AP_WindVane (Rover/Sailboat)
**Singleton**: `AP_WindVane::get_singleton()`

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize |
| `update()` | void | Update readings |
| `get_apparent_wind_direction_rad()` | float | Apparent dir (rad) |
| `get_true_wind_direction_rad()` | float | True dir (rad) |
| `get_apparent_wind_speed()` | float | Apparent speed (m/s) |
| `get_true_wind_speed()` | float | True speed (m/s) |

### AP_WheelEncoder (Rover)
**Singleton**: `AP_WheelEncoder::get_singleton()`

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize |
| `update()` | void | Update readings |
| `healthy(i)` | bool | Encoder healthy |
| `get_distance(i)` | float | Distance (m) |
| `get_rate(i)` | float | Rate (rad/s) |
