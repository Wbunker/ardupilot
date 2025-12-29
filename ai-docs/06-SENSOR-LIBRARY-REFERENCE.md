# ArduPilot Sensor Library Reference

A comprehensive guide to all sensor libraries in ArduPilot, their interfaces, and how to use them.

## Table of Contents
1. [Inertial/Motion Sensors](#1-inertialmotion-sensors)
2. [Position/Navigation Sensors](#2-positionnavigation-sensors)
3. [Range/Distance Sensors](#3-rangedistance-sensors)
4. [Airflow Sensors](#4-airflow-sensors)
5. [Power/Battery Monitoring](#5-powerbattery-monitoring)
6. [State Estimation (AHRS)](#6-state-estimation-ahrs)
7. [Peripheral Sensors](#7-peripheral-sensors)
8. [Vehicle-Specific Sensors](#8-vehicle-specific-sensors)
9. [Quick Reference Table](#9-quick-reference-table)

---

## 1. Inertial/Motion Sensors

### AP_InertialSensor (IMU)

**Location**: `libraries/AP_InertialSensor/`

**Purpose**: Provides gyroscope and accelerometer readings for attitude estimation.

**Singleton Access**:
```cpp
AP_InertialSensor *ins = AP_InertialSensor::get_singleton();
// Or via AP namespace:
AP_InertialSensor &ins = AP::ins();
```

**Key Interface Functions**:

| Function | Returns | Description |
|----------|---------|-------------|
| `init(sample_rate_hz)` | void | Initialize sensors at specified rate |
| `get_gyro()` | `Vector3f&` | Rotation rates in rad/s (X, Y, Z) |
| `get_gyro(i)` | `Vector3f&` | Gyro from sensor instance i |
| `get_accel()` | `Vector3f&` | Acceleration in m/s² (X, Y, Z) |
| `get_accel(i)` | `Vector3f&` | Accel from sensor instance i |
| `get_delta_angle(delta, dt)` | bool | Integrated angle since last call |
| `get_delta_velocity(delta, dt)` | bool | Integrated velocity since last call |
| `get_gyro_health()` | bool | Primary gyro healthy |
| `get_gyro_health(i)` | bool | Gyro i healthy |
| `get_accel_health()` | bool | Primary accel healthy |
| `get_accel_health(i)` | bool | Accel i healthy |
| `get_gyro_count()` | uint8_t | Number of gyro sensors |
| `get_accel_count()` | uint8_t | Number of accel sensors |
| `calibrating()` | bool | Currently calibrating |
| `get_primary_gyro()` | uint8_t | Primary gyro instance |
| `get_primary_accel()` | uint8_t | Primary accel instance |
| `get_accel_clip_count(i)` | uint32_t | Clipping events on accel i |
| `periodic()` | void | Call from main loop |

**Supported Backends** (30+):
- ICM20689, ICM20602, ICM20948, ICM42688
- BMI055, BMI088, BMI160, BMI270
- MPU6000, MPU9250
- LSM9DS0, LSM9DS1
- ADIS16xxx series
- DroneCAN, External AHRS, SITL

**Example Usage**:
```cpp
AP_InertialSensor &ins = AP::ins();

// Initialize at 400Hz
ins.init(400);

// In main loop
ins.periodic();

// Read data
Vector3f gyro = ins.get_gyro();      // rad/s
Vector3f accel = ins.get_accel();    // m/s²

if (ins.get_gyro_health() && ins.get_accel_health()) {
    // Process valid data
    float roll_rate = gyro.x;
    float pitch_rate = gyro.y;
    float yaw_rate = gyro.z;
}
```

---

## 2. Position/Navigation Sensors

### AP_GPS

**Location**: `libraries/AP_GPS/`

**Purpose**: GNSS position, velocity, and time data.

**Singleton Access**:
```cpp
AP_GPS &gps = AP::gps();
```

**Key Interface Functions**:

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize GPS system |
| `update()` | void | Update GPS state (call at 10Hz+) |
| `status()` | GPS_Status | Fix status (NO_GPS, NO_FIX, 2D, 3D, DGPS, RTK_FLOAT, RTK_FIXED) |
| `status(i)` | GPS_Status | Status of instance i |
| `location()` | `Location&` | Current location (lat/lon/alt) |
| `location(i)` | `Location&` | Location from instance i |
| `ground_speed()` | float | Speed over ground (m/s) |
| `ground_course()` | float | Course over ground (degrees) |
| `velocity()` | `Vector3f&` | 3D velocity NED (m/s) |
| `num_sats()` | uint8_t | Number of satellites |
| `num_sats(i)` | uint8_t | Satellites for instance i |
| `hdop()` | uint16_t | Horizontal DOP × 100 |
| `vdop()` | uint16_t | Vertical DOP × 100 |
| `horizontal_accuracy(hacc)` | bool | Horizontal accuracy in m |
| `vertical_accuracy(vacc)` | bool | Vertical accuracy in m |
| `speed_accuracy(sacc)` | bool | Speed accuracy in m/s |
| `have_vertical_velocity()` | bool | Has vertical velocity |
| `gps_yaw_deg(i, yaw)` | bool | GPS-derived heading |
| `num_sensors()` | uint8_t | Number of GPS units |
| `primary_sensor()` | uint8_t | Primary GPS instance |
| `time_week()` | uint16_t | GPS week number |
| `time_week_ms()` | uint32_t | ms into GPS week |
| `last_fix_time_ms()` | uint32_t | Time of last fix |

**GPS Status Enum**:
```cpp
enum GPS_Status {
    NO_GPS = 0,           // No GPS detected
    NO_FIX = 1,           // No position fix
    GPS_OK_FIX_2D = 2,    // 2D fix
    GPS_OK_FIX_3D = 3,    // 3D fix
    GPS_OK_FIX_3D_DGPS = 4,      // Differential GPS
    GPS_OK_FIX_3D_RTK_FLOAT = 5, // RTK float
    GPS_OK_FIX_3D_RTK_FIXED = 6, // RTK fixed (cm accuracy)
};
```

**Supported Backends** (20+):
- u-blox (M8, F9P, etc.)
- Septentrio SBF
- Swift Navigation SBP
- Trimble/Ashtech
- NMEA generic
- DroneCAN
- SITL

**Example Usage**:
```cpp
AP_GPS &gps = AP::gps();

gps.update();

if (gps.status() >= AP_GPS::GPS_OK_FIX_3D) {
    Location loc = gps.location();
    float speed = gps.ground_speed();
    float course = gps.ground_course();
    uint8_t sats = gps.num_sats();

    // lat/lon in degrees×10^7
    int32_t lat = loc.lat;
    int32_t lng = loc.lng;
    int32_t alt_cm = loc.alt; // cm above sea level
}
```

---

### Compass (AP_Compass)

**Location**: `libraries/AP_Compass/`

**Purpose**: Magnetometer readings for heading reference.

**Singleton Access**:
```cpp
Compass &compass = AP::compass();
```

**Key Interface Functions**:

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize compass |
| `read()` | bool | Read and update values |
| `get_field()` | `Vector3f&` | Magnetic field (milligauss) |
| `get_field(i)` | `Vector3f&` | Field from instance i |
| `calculate_heading(dcm)` | float | Heading in radians |
| `calculate_heading(dcm, i)` | float | Heading from instance i |
| `healthy()` | bool | Primary compass healthy |
| `healthy(i)` | bool | Compass i healthy |
| `get_count()` | uint8_t | Number of compasses |
| `get_declination()` | float | Magnetic declination (rad) |
| `set_declination(dec)` | void | Set declination |
| `use_for_yaw()` | bool | Used for yaw estimation |
| `use_for_yaw(i)` | bool | Instance i used for yaw |
| `set_offsets(i, offsets)` | void | Set calibration offsets |
| `get_offsets(i)` | `Vector3f&` | Get calibration offsets |
| `available()` | bool | Compass enabled & initialized |

**Supported Backends**:
- HMC5843, HMC5883
- AK8963, AK09916
- LIS3MDL, LIS2MDL
- IST8310, IST8308
- QMC5883L
- BMM150
- RM3100
- DroneCAN, External AHRS

**Example Usage**:
```cpp
Compass &compass = AP::compass();

if (compass.read() && compass.healthy()) {
    Vector3f field = compass.get_field();

    // Calculate heading (need attitude DCM matrix)
    Matrix3f dcm;
    float heading = compass.calculate_heading(dcm);
}
```

---

### AP_Baro (Barometer)

**Location**: `libraries/AP_Baro/`

**Purpose**: Atmospheric pressure for altitude estimation.

**Singleton Access**:
```cpp
AP_Baro &baro = AP::baro();
```

**Key Interface Functions**:

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize barometer |
| `calibrate(save)` | void | Calibrate ground pressure |
| `update()` | void | Update readings |
| `get_pressure()` | float | Pressure in Pascals |
| `get_pressure(i)` | float | Pressure from instance i |
| `get_temperature()` | float | Temperature in °C |
| `get_temperature(i)` | float | Temperature from instance i |
| `get_altitude()` | float | Altitude in meters (rel to cal) |
| `get_altitude(i)` | float | Altitude from instance i |
| `get_climb_rate()` | float | Climb rate in m/s |
| `healthy()` | bool | Primary sensor healthy |
| `healthy(i)` | bool | Sensor i healthy |
| `num_instances()` | uint8_t | Number of barometers |
| `get_ground_pressure()` | float | Calibrated ground pressure |
| `get_ground_temperature()` | float | Ground temperature °C |

**Supported Backends** (20+):
- BMP085, BMP180, BMP280, BMP388
- MS5611, MS5607, MS5837
- SPL06, DPS280
- LPS22H, LPS25H
- DroneCAN
- SITL

**Example Usage**:
```cpp
AP_Baro &baro = AP::baro();

baro.update();

if (baro.healthy()) {
    float pressure = baro.get_pressure();    // Pa
    float temp = baro.get_temperature();     // °C
    float alt = baro.get_altitude();         // m
    float climb = baro.get_climb_rate();     // m/s
}
```

---

## 3. Range/Distance Sensors

### RangeFinder

**Location**: `libraries/AP_RangeFinder/`

**Purpose**: Distance measurement (lidar, sonar, radar).

**Singleton Access**:
```cpp
RangeFinder *rf = AP::rangefinder();
```

**Key Interface Functions**:

| Function | Returns | Description |
|----------|---------|-------------|
| `init(orientation)` | void | Initialize with default orientation |
| `update()` | void | Update all rangefinders |
| `num_sensors()` | uint8_t | Number of sensors |
| `distance_orient(rot)` | float | Distance for orientation (m) |
| `status_orient(rot)` | Status | Status for orientation |
| `has_orientation(rot)` | bool | Has sensor with orientation |
| `has_data_orient(rot)` | bool | Valid data for orientation |
| `max_distance_orient(rot)` | float | Max range for orientation (m) |
| `min_distance_orient(rot)` | float | Min range for orientation (m) |
| `signal_quality_pct_orient(rot)` | int8_t | Signal quality 0-100 |
| `ground_clearance_orient(rot)` | float | Ground clearance (m) |
| `get_pos_offset_orient(rot)` | `Vector3f&` | Position offset |
| `last_reading_ms(rot)` | uint32_t | Time of last reading |
| `prearm_healthy(msg, len)` | bool | Pre-arm check |

**Status Enum**:
```cpp
enum class Status {
    NotConnected = 0,
    NoData = 1,
    OutOfRangeLow = 2,
    OutOfRangeHigh = 3,
    Good = 4,
};
```

**Supported Backends** (40+):
- LightWare SF02, SF10, SF11, SF40C, SF45B
- Benewake TFmini, TF02, TF03, TFLuna
- Garmin Lidar-Lite
- MaxBotix sonar
- TeraRanger
- Leddar
- DroneCAN
- PWM input
- Analog
- SITL

**Example Usage**:
```cpp
RangeFinder *rf = AP::rangefinder();

rf->update();

// Get downward-facing distance
if (rf->has_data_orient(ROTATION_PITCH_270)) {
    float dist = rf->distance_orient(ROTATION_PITCH_270); // meters
    RangeFinder::Status status = rf->status_orient(ROTATION_PITCH_270);

    if (status == RangeFinder::Status::Good) {
        // Valid distance reading
    }
}
```

---

### AP_Proximity

**Location**: `libraries/AP_Proximity/`

**Purpose**: 360° obstacle detection for avoidance.

**Singleton Access**:
```cpp
AP_Proximity *prx = AP::proximity();
```

**Key Interface Functions**:

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize proximity |
| `update()` | void | Update all sensors |
| `get_status()` | Status | Overall status |
| `num_sensors()` | uint8_t | Number of sensors |
| `get_horizontal_distances(array)` | bool | Get distances by angle |
| `get_closest_object(angle, dist)` | bool | Closest obstacle |
| `get_obstacle_count()` | uint8_t | Number of obstacles |
| `get_obstacle(num, vec)` | bool | Vector to obstacle |
| `distance_max_m()` | float | Maximum detection range |
| `distance_min_m()` | float | Minimum detection range |
| `get_upward_distance(dist)` | bool | Distance above |
| `sensor_present()` | bool | Sensor present |
| `sensor_enabled()` | bool | Sensor enabled |
| `sensor_failed()` | bool | Sensor failed |

**Supported Backends**:
- RPLidar A2
- LightWare SF40C, SF45B
- TeraRanger Tower/Tower EVO
- Cygbot D1
- LD06
- DroneCAN
- MAVLink
- SITL

**Example Usage**:
```cpp
AP_Proximity *prx = AP::proximity();

prx->update();

if (prx->get_status() == AP_Proximity::Status::Good) {
    float angle, distance;
    if (prx->get_closest_object(angle, distance)) {
        // Obstacle at 'angle' degrees, 'distance' meters away
    }
}
```

---

## 4. Airflow Sensors

### AP_Airspeed

**Location**: `libraries/AP_Airspeed/`

**Purpose**: Differential pressure for airspeed measurement.

**Singleton Access**:
```cpp
AP_Airspeed *airspeed = AP::airspeed();
```

**Key Interface Functions**:

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize airspeed |
| `calibrate(in_startup)` | void | Calibrate zero offset |
| `update()` | void | Update readings |
| `get_airspeed()` | float | Indicated airspeed (m/s) |
| `get_airspeed(i)` | float | Airspeed from instance i |
| `get_raw_airspeed()` | float | Unfiltered airspeed |
| `get_differential_pressure()` | float | Pressure difference (Pa) |
| `get_temperature(temp)` | bool | Probe temperature (°C) |
| `healthy()` | bool | Primary sensor healthy |
| `healthy(i)` | bool | Sensor i healthy |
| `enabled()` | bool | Airspeed enabled |
| `use()` | bool | Use for control |
| `get_airspeed_ratio()` | float | Calibration ratio |
| `last_update_ms()` | uint32_t | Time of last update |

**Supported Backends**:
- MS4525DO, MS5525
- DLVR (pressure sensors)
- SDP3x
- NMEA
- Analog
- DroneCAN
- SITL

**Example Usage**:
```cpp
AP_Airspeed *airspeed = AP::airspeed();

airspeed->update();

if (airspeed->healthy() && airspeed->enabled()) {
    float ias = airspeed->get_airspeed();  // m/s
    float temp;
    if (airspeed->get_temperature(temp)) {
        // temp in °C
    }
}
```

---

### AP_OpticalFlow

**Location**: `libraries/AP_OpticalFlow/`

**Purpose**: Ground-relative velocity using optical sensor.

**Singleton Access**:
```cpp
AP_OpticalFlow *flow = AP::opticalflow();
```

**Key Interface Functions**:

| Function | Returns | Description |
|----------|---------|-------------|
| `init(log_bit)` | void | Initialize optical flow |
| `update()` | void | Update readings |
| `enabled()` | bool | Flow enabled |
| `healthy()` | bool | Sensor healthy |
| `quality()` | uint8_t | Surface quality 0-255 |
| `flowRate()` | `Vector2f&` | Raw flow rate (rad/s) |
| `bodyRate()` | `Vector2f&` | IMU-corrected flow (rad/s) |
| `last_update()` | uint32_t | Last update time (ms) |
| `get_pos_offset()` | `Vector3f&` | Sensor position offset |

**Supported Backends**:
- PX4Flow
- PMW3901 (Pixart)
- CXOF
- HereFlow (DroneCAN)
- UPFLOW
- MAVLink
- SITL

**Example Usage**:
```cpp
AP_OpticalFlow *flow = AP::opticalflow();

flow->update();

if (flow->healthy() && flow->quality() > 50) {
    Vector2f flow_rate = flow->flowRate();    // rad/s
    Vector2f body_rate = flow->bodyRate();    // rad/s (corrected)
}
```

---

## 5. Power/Battery Monitoring

### AP_BattMonitor

**Location**: `libraries/AP_BattMonitor/`

**Purpose**: Battery voltage, current, and capacity monitoring.

**Singleton Access**:
```cpp
AP_BattMonitor &battery = AP::battery();
```

**Key Interface Functions**:

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize monitors |
| `read()` | void | Update readings (call at 10Hz) |
| `num_instances()` | uint8_t | Number of batteries |
| `voltage()` | float | Primary voltage (V) |
| `voltage(i)` | float | Voltage of battery i |
| `current_amps(current, i)` | bool | Current draw (A) |
| `consumed_mah(mah, i)` | bool | Consumed capacity (mAh) |
| `consumed_wh(wh, i)` | bool | Consumed energy (Wh) |
| `capacity_remaining_pct(pct, i)` | bool | Remaining capacity % |
| `time_remaining(secs, i)` | bool | Time remaining (s) |
| `pack_capacity_mah(i)` | int32_t | Total capacity (mAh) |
| `healthy()` | bool | All monitors healthy |
| `healthy(i)` | bool | Monitor i healthy |
| `get_temperature(temp, i)` | bool | Battery temp (°C) |
| `has_failsafed()` | bool | Failsafe triggered |

**Failsafe Enum**:
```cpp
enum class Failsafe : uint8_t {
    None = 0,
    Unhealthy,
    Low,
    Critical
};
```

**Supported Backends**:
- Analog (voltage divider, current sense)
- SMBus smart batteries
- DroneCAN
- FuelFlow
- Generator/EFI
- INA2xx, INA3221 (power monitors)
- Scripting

**Example Usage**:
```cpp
AP_BattMonitor &battery = AP::battery();

battery.read();

if (battery.healthy()) {
    float voltage = battery.voltage();
    float current;
    if (battery.current_amps(current)) {
        // current in Amps
    }

    uint8_t pct;
    if (battery.capacity_remaining_pct(pct)) {
        // pct = remaining percentage
    }
}
```

---

### AP_ESC_Telem

**Location**: `libraries/AP_ESC_Telem/`

**Purpose**: ESC telemetry (RPM, current, temperature, voltage).

**Singleton Access**:
```cpp
AP_ESC_Telem *esc = AP_ESC_Telem::get_singleton();
```

**Key Interface Functions**:

| Function | Returns | Description |
|----------|---------|-------------|
| `get_rpm(esc_index, rpm)` | bool | ESC RPM |
| `get_temperature(esc_index, temp)` | bool | ESC temp (centi-°C) |
| `get_motor_temperature(esc_index, temp)` | bool | Motor temp |
| `get_current(esc_index, amps)` | bool | ESC current (A) |
| `get_voltage(esc_index, volts)` | bool | ESC voltage (V) |
| `get_consumption_mah(esc_index, mah)` | bool | Consumed (mAh) |
| `get_average_motor_rpm()` | float | Average RPM |
| `get_average_motor_frequency_hz()` | float | Average Hz |
| `get_num_active_escs()` | uint8_t | Active ESC count |
| `get_active_esc_mask()` | uint32_t | Bitmask of active ESCs |
| `update()` | void | Update (call at 10Hz) |

**Example Usage**:
```cpp
AP_ESC_Telem *esc = AP_ESC_Telem::get_singleton();

esc->update();

for (uint8_t i = 0; i < 4; i++) {
    float rpm;
    if (esc->get_rpm(i, rpm)) {
        // rpm for motor i
    }

    int16_t temp;
    if (esc->get_temperature(i, temp)) {
        // temp in centi-degrees C
    }
}
```

---

## 6. State Estimation (AHRS)

### AP_AHRS

**Location**: `libraries/AP_AHRS/`

**Purpose**: Fused attitude, position, and velocity estimates.

**Singleton Access**:
```cpp
AP_AHRS &ahrs = AP::ahrs();
```

**Key Interface Functions**:

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize AHRS |
| `update()` | void | Update estimates |
| `get_gyro()` | `Vector3f&` | Corrected gyro (rad/s) |
| `get_gyro_drift()` | `Vector3f&` | Gyro drift estimate |
| `get_rotation_body_to_ned()` | `Matrix3f&` | Rotation matrix |
| `get_quaternion()` | `Quaternion&` | Attitude quaternion |
| `roll` | float | Roll angle (rad) |
| `pitch` | float | Pitch angle (rad) |
| `yaw` | float | Yaw angle (rad) |
| `roll_sensor` | int32_t | Roll (centi-degrees) |
| `pitch_sensor` | int32_t | Pitch (centi-degrees) |
| `yaw_sensor` | int32_t | Yaw (centi-degrees) |
| `get_location(loc)` | bool | Current position |
| `get_velocity_NED(vel)` | bool | Velocity NED (m/s) |
| `groundspeed_vector()` | Vector2f | Ground speed (m/s) |
| `get_hagl(hagl)` | bool | Height above ground (m) |
| `healthy()` | bool | AHRS healthy |
| `wind_estimate()` | `Vector3f&` | Wind estimate (m/s) |
| `get_error_rp()` | float | Roll/pitch error |
| `get_error_yaw()` | float | Yaw error |
| `airspeed_EAS(airspeed)` | bool | Equivalent airspeed |
| `get_EAS2TAS()` | float | EAS to TAS ratio |

**Example Usage**:
```cpp
AP_AHRS &ahrs = AP::ahrs();

ahrs.update();

// Attitude (radians)
float roll = ahrs.roll;
float pitch = ahrs.pitch;
float yaw = ahrs.yaw;

// Position
Location loc;
if (ahrs.get_location(loc)) {
    // Valid position
}

// Velocity
Vector3f vel;
if (ahrs.get_velocity_NED(vel)) {
    float north = vel.x;
    float east = vel.y;
    float down = vel.z;
}
```

---

## 7. Peripheral Sensors

### AP_TemperatureSensor

**Location**: `libraries/AP_TemperatureSensor/`

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize sensors |
| `update()` | void | Update readings |
| `get_temperature(temp, i)` | bool | Temperature (°C) |
| `healthy(i)` | bool | Sensor i healthy |
| `num_instances()` | uint8_t | Number of sensors |

**Backends**: TSYS01, TSYS03, MCP9600, MAX31865, MLX90614, Analog, DroneCAN

---

### AP_RPM

**Location**: `libraries/AP_RPM/`

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize RPM sensors |
| `update()` | void | Update readings |
| `get_rpm(instance, rpm)` | bool | RPM value |
| `get_signal_quality(i)` | float | Signal quality |
| `num_sensors()` | uint8_t | Number of sensors |

**Backends**: PWM input, GPIO pin, EFI, ESC telemetry, DroneCAN, Harmonic notch

---

### AP_Beacon

**Location**: `libraries/AP_Beacon/`

**Purpose**: Indoor positioning using beacons.

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize |
| `update()` | void | Update readings |
| `healthy()` | bool | System healthy |
| `get_vehicle_position_ned(pos, acc)` | bool | Position from origin (m) |
| `get_origin(loc)` | bool | System origin |
| `count()` | uint8_t | Number of beacons |
| `beacon_distance(i)` | float | Distance to beacon i (m) |
| `beacon_position(i)` | Vector3f | Beacon i position (m) |

**Backends**: Pozyx, Marvelmind, Nooploop

---

### AP_VisualOdom

**Location**: `libraries/AP_VisualOdom/`

**Purpose**: Visual odometry from cameras.

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize |
| `enabled()` | bool | VO enabled |
| `healthy()` | bool | Sensor healthy |
| `quality()` | int8_t | Quality (-1=fail, 0-100) |
| `get_pos_offset()` | `Vector3f&` | Camera offset |
| `get_delay_ms()` | uint16_t | Sensor delay |

**Backends**: Intel T265, VOXL, MAVLink

---

### AP_WindVane

**Location**: `libraries/AP_WindVane/`

**Purpose**: Wind direction and speed for sailboats.

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize |
| `update()` | void | Update readings |
| `enabled()` | bool | Wind vane enabled |
| `get_apparent_wind_direction_rad()` | float | Apparent direction (rad) |
| `get_true_wind_direction_rad()` | float | True direction (rad) |
| `get_apparent_wind_speed()` | float | Apparent speed (m/s) |
| `get_true_wind_speed()` | float | True speed (m/s) |
| `get_current_tack()` | Tack | Port or starboard |

**Backends**: Analog, NMEA, Airspeed, RPM, Home heading

---

### AP_WheelEncoder

**Location**: `libraries/AP_WheelEncoder/`

**Purpose**: Wheel rotation for rovers.

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize |
| `update()` | void | Update readings |
| `healthy(i)` | bool | Encoder i healthy |
| `get_distance(i)` | float | Distance traveled (m) |
| `get_delta_angle(i)` | float | Rotation (rad) |
| `get_rate(i)` | float | Angular rate (rad/s) |
| `get_wheel_radius(i)` | float | Wheel radius (m) |

**Backends**: Quadrature encoder, SITL

---

## 8. Vehicle-Specific Sensors

### AP_LeakDetector (ArduSub)

**Location**: `libraries/AP_LeakDetector/`

| Function | Returns | Description |
|----------|---------|-------------|
| `init()` | void | Initialize |
| `update()` | bool | Update (returns leak status) |
| `get_status()` | bool | Leak detected |
| `set_detect()` | void | Set external leak detection |

**Backends**: Analog, Digital GPIO

---

## 9. Quick Reference Table

### All Sensor Libraries with Primary Functions

| Library | Access | init | update | get_value | healthy |
|---------|--------|------|--------|-----------|---------|
| **AP_InertialSensor** | `AP::ins()` | `init(rate)` | `periodic()` | `get_gyro()`, `get_accel()` | `get_gyro_health()` |
| **AP_GPS** | `AP::gps()` | `init()` | `update()` | `location()`, `ground_speed()` | `status() >= 3` |
| **Compass** | `AP::compass()` | `init()` | `read()` | `get_field()`, `calculate_heading()` | `healthy()` |
| **AP_Baro** | `AP::baro()` | `init()` | `update()` | `get_pressure()`, `get_altitude()` | `healthy()` |
| **RangeFinder** | `AP::rangefinder()` | `init(rot)` | `update()` | `distance_orient(rot)` | `status_orient()` |
| **AP_Proximity** | `AP::proximity()` | `init()` | `update()` | `get_closest_object()` | `get_status()` |
| **AP_Airspeed** | `AP::airspeed()` | `init()` | `update()` | `get_airspeed()` | `healthy()` |
| **AP_OpticalFlow** | `AP::opticalflow()` | `init()` | `update()` | `flowRate()`, `bodyRate()` | `healthy()` |
| **AP_BattMonitor** | `AP::battery()` | `init()` | `read()` | `voltage()`, `current_amps()` | `healthy()` |
| **AP_AHRS** | `AP::ahrs()` | `init()` | `update()` | `roll`, `pitch`, `yaw` | `healthy()` |
| **AP_TemperatureSensor** | `get_singleton()` | `init()` | `update()` | `get_temperature()` | `healthy()` |
| **AP_RPM** | N/A | `init()` | `update()` | `get_rpm()` | via return |
| **AP_Beacon** | `AP::beacon()` | `init()` | `update()` | `get_vehicle_position_ned()` | `healthy()` |
| **AP_VisualOdom** | `AP::visualodom()` | `init()` | N/A | via `handle_*` | `healthy()` |
| **AP_WindVane** | `get_singleton()` | `init()` | `update()` | `get_*_wind_*()` | `enabled()` |
| **AP_WheelEncoder** | `get_singleton()` | `init()` | `update()` | `get_distance()` | `healthy()` |
| **AP_LeakDetector** | N/A | `init()` | `update()` | `get_status()` | via return |
| **AP_ESC_Telem** | `get_singleton()` | N/A | `update()` | `get_rpm()`, `get_*()` | via return |

### Common Pattern

All ArduPilot sensor libraries follow this pattern:

```cpp
// 1. Get singleton
SensorClass *sensor = SensorClass::get_singleton();
// or
SensorClass &sensor = AP::sensor();

// 2. Initialize (once at startup)
sensor.init();

// 3. Update in main loop
void loop() {
    sensor.update();  // or sensor.read(), sensor.periodic()

    // 4. Check health
    if (sensor.healthy()) {
        // 5. Read data
        auto value = sensor.get_value();
    }
}
```

---

## See Also

- `ai-docs/05-AP_BARO-DEEP-DIVE.md` - Deep dive into one sensor library
- `ai-docs/02-CODEBASE-ARCHITECTURE.md` - Overall architecture
- Official docs: https://ardupilot.org/dev/docs/apmcopter-programming-libraries.html
