# ArduPilot Sensor Code Patterns

Common patterns and examples for working with ArduPilot sensors.

## Table of Contents
1. [Basic Sensor Usage](#basic-sensor-usage)
2. [Vehicle Integration](#vehicle-integration)
3. [Multi-Instance Handling](#multi-instance-handling)
4. [Health and Failsafe](#health-and-failsafe)
5. [Calibration](#calibration)
6. [Data Fusion Examples](#data-fusion-examples)
7. [Custom Backend Implementation](#custom-backend-implementation)

---

## Basic Sensor Usage

### IMU - Read Gyro and Accelerometer
```cpp
#include <AP_InertialSensor/AP_InertialSensor.h>

void read_imu() {
    AP_InertialSensor &ins = AP::ins();

    // Get rotation rates (rad/s)
    Vector3f gyro = ins.get_gyro();
    float roll_rate = gyro.x;
    float pitch_rate = gyro.y;
    float yaw_rate = gyro.z;

    // Get acceleration (m/s²)
    Vector3f accel = ins.get_accel();
    float accel_x = accel.x;  // Forward
    float accel_y = accel.y;  // Right
    float accel_z = accel.z;  // Down

    // For integration, use delta values
    Vector3f delta_angle;
    float dt;
    if (ins.get_delta_angle(delta_angle, dt)) {
        // delta_angle contains integrated rotation since last call
    }
}
```

### GPS - Get Position and Velocity
```cpp
#include <AP_GPS/AP_GPS.h>

void read_gps() {
    AP_GPS &gps = AP::gps();

    // Check fix quality first
    if (gps.status() < AP_GPS::GPS_OK_FIX_3D) {
        return;  // No valid 3D fix
    }

    // Get position
    Location loc = gps.location();
    double lat_deg = loc.lat * 1.0e-7;
    double lon_deg = loc.lng * 1.0e-7;
    float alt_m = loc.alt * 0.01f;  // Convert cm to m

    // Get velocity
    Vector3f vel = gps.velocity();  // NED in m/s
    float ground_speed = gps.ground_speed();  // m/s
    float course = gps.ground_course();  // degrees

    // Get accuracy estimates
    float hacc, vacc, sacc;
    if (gps.horizontal_accuracy(hacc)) {
        // hacc in meters
    }
    if (gps.vertical_accuracy(vacc)) {
        // vacc in meters
    }

    // Satellite info
    uint8_t num_sats = gps.num_sats();
    uint16_t hdop = gps.hdop();  // × 100
}
```

### Barometer - Altitude and Climb Rate
```cpp
#include <AP_Baro/AP_Baro.h>

void read_baro() {
    AP_Baro &baro = AP::baro();

    baro.update();

    if (!baro.healthy()) {
        return;
    }

    float pressure_pa = baro.get_pressure();
    float temp_c = baro.get_temperature();
    float altitude_m = baro.get_altitude();  // Relative to calibration
    float climb_rate = baro.get_climb_rate();  // m/s, positive up
}
```

### Compass - Magnetic Heading
```cpp
#include <AP_Compass/AP_Compass.h>
#include <AP_AHRS/AP_AHRS.h>

void read_compass() {
    Compass &compass = AP::compass();

    if (!compass.read() || !compass.healthy()) {
        return;
    }

    // Raw magnetic field (milligauss)
    Vector3f field = compass.get_field();

    // Calculate heading (need attitude for tilt compensation)
    AP_AHRS &ahrs = AP::ahrs();
    Matrix3f dcm;
    ahrs.get_rotation_body_to_ned().to_euler(nullptr, nullptr, nullptr);
    float heading_rad = compass.calculate_heading(ahrs.get_rotation_body_to_ned());
    float heading_deg = degrees(heading_rad);
}
```

### RangeFinder - Distance Measurement
```cpp
#include <AP_RangeFinder/AP_RangeFinder.h>

void read_rangefinder() {
    RangeFinder *rf = AP::rangefinder();
    if (rf == nullptr) return;

    rf->update();

    // Downward-facing sensor (most common)
    if (rf->has_data_orient(ROTATION_PITCH_270)) {
        float distance_m = rf->distance_orient(ROTATION_PITCH_270);
        RangeFinder::Status status = rf->status_orient(ROTATION_PITCH_270);

        if (status == RangeFinder::Status::Good) {
            // Valid reading
            float max_range = rf->max_distance_orient(ROTATION_PITCH_270);
            float min_range = rf->min_distance_orient(ROTATION_PITCH_270);
        }
    }

    // Forward-facing sensor
    if (rf->has_data_orient(ROTATION_NONE)) {
        float forward_dist = rf->distance_orient(ROTATION_NONE);
    }
}
```

### Battery Monitor
```cpp
#include <AP_BattMonitor/AP_BattMonitor.h>

void read_battery() {
    AP_BattMonitor &battery = AP::battery();

    battery.read();

    if (!battery.healthy()) {
        return;
    }

    // Primary battery
    float voltage = battery.voltage();

    float current;
    if (battery.current_amps(current)) {
        // current in Amps
    }

    float mah;
    if (battery.consumed_mah(mah)) {
        // mah consumed
    }

    uint8_t remaining_pct;
    if (battery.capacity_remaining_pct(remaining_pct)) {
        // remaining_pct is 0-100
    }

    // Check failsafe
    if (battery.has_failsafed()) {
        // Handle failsafe
    }
}
```

---

## Vehicle Integration

### Typical Sensor Init in Vehicle
```cpp
// In Copter.cpp or similar vehicle file

void Copter::init_ardupilot() {
    // Initialize sensors in correct order
    ins.init(scheduler.get_loop_rate_hz());
    barometer.init();
    compass.init();
    gps.init();

    // Calibrate barometer (blocking)
    barometer.calibrate();

    // Wait for GPS (non-blocking check in loop)
}
```

### Scheduler Task for Sensor Updates
```cpp
// Define task table
const AP_Scheduler::Task Copter::scheduler_tasks[] = {
    // Function              Rate(Hz)  Max Time(us)
    SCHED_TASK(read_AHRS,         400,    100),
    SCHED_TASK(update_GPS,         50,    200),
    SCHED_TASK(read_barometer,     10,     50),
    SCHED_TASK(update_compass,     10,    100),
    SCHED_TASK(read_rangefinder,   20,     50),
    SCHED_TASK(read_battery,       10,     50),
    // ...
};

void Copter::read_barometer() {
    barometer.update();
}

void Copter::update_GPS() {
    gps.update();
}
```

---

## Multi-Instance Handling

### Iterate All GPS Instances
```cpp
void check_all_gps() {
    AP_GPS &gps = AP::gps();

    uint8_t num = gps.num_sensors();
    for (uint8_t i = 0; i < num; i++) {
        if (gps.status(i) >= AP_GPS::GPS_OK_FIX_3D) {
            Location loc = gps.location(i);
            // Process each GPS
        }
    }

    // Get primary (blended or selected)
    uint8_t primary = gps.primary_sensor();
}
```

### Select Best Barometer
```cpp
void use_best_baro() {
    AP_Baro &baro = AP::baro();

    float best_alt = 0;
    bool found = false;

    for (uint8_t i = 0; i < baro.num_instances(); i++) {
        if (baro.healthy(i)) {
            if (!found) {
                best_alt = baro.get_altitude(i);
                found = true;
            }
        }
    }
}
```

---

## Health and Failsafe

### Pre-arm Sensor Checks
```cpp
bool sensors_prearm_check(char *msg, uint8_t msg_len) {
    // Check IMU
    AP_InertialSensor &ins = AP::ins();
    if (!ins.get_gyro_health() || !ins.get_accel_health()) {
        snprintf(msg, msg_len, "IMU not healthy");
        return false;
    }

    // Check barometer
    AP_Baro &baro = AP::baro();
    if (!baro.healthy()) {
        snprintf(msg, msg_len, "Baro not healthy");
        return false;
    }

    // Check compass
    Compass &compass = AP::compass();
    if (compass.enabled() && !compass.healthy()) {
        snprintf(msg, msg_len, "Compass not healthy");
        return false;
    }

    // Check GPS (if required)
    AP_GPS &gps = AP::gps();
    if (gps.status() < AP_GPS::GPS_OK_FIX_3D) {
        snprintf(msg, msg_len, "GPS no 3D fix");
        return false;
    }

    // Check rangefinder (if configured)
    RangeFinder *rf = AP::rangefinder();
    if (rf != nullptr && rf->num_sensors() > 0) {
        if (!rf->prearm_healthy(msg, msg_len)) {
            return false;
        }
    }

    return true;
}
```

### Handle Sensor Failure In-Flight
```cpp
void handle_sensor_failure() {
    AP_AHRS &ahrs = AP::ahrs();

    // AHRS health includes EKF checks
    if (!ahrs.healthy()) {
        // Switch to safe mode or land
    }

    // GPS failure
    AP_GPS &gps = AP::gps();
    if (gps.status() < AP_GPS::GPS_OK_FIX_3D) {
        // Switch to non-GPS mode (altitude hold, etc.)
    }

    // Barometer failure
    AP_Baro &baro = AP::baro();
    if (!baro.healthy()) {
        // Use GPS altitude if available
    }
}
```

---

## Calibration

### Accelerometer Calibration
```cpp
// Simple level calibration
void simple_accel_cal() {
    AP_InertialSensor &ins = AP::ins();

    // Vehicle must be level and still
    ins.init_accel();
}
```

### Compass Calibration
```cpp
void start_compass_cal() {
    Compass &compass = AP::compass();

    // Start calibration for all compasses
    for (uint8_t i = 0; i < compass.get_count(); i++) {
        compass.start_calibration(i);
    }
}

void update_compass_cal() {
    Compass &compass = AP::compass();

    // Call periodically during calibration
    compass.compass_cal_update();

    // Check if complete
    for (uint8_t i = 0; i < compass.get_count(); i++) {
        auto status = compass.get_calibration_status(i);
        // Handle status
    }
}
```

### Barometer Calibration
```cpp
void calibrate_baro() {
    AP_Baro &baro = AP::baro();

    // Must be on ground, vehicle stationary
    // This is blocking
    baro.calibrate(true);  // true = save to EEPROM
}
```

---

## Data Fusion Examples

### Combine Baro and Rangefinder for Altitude
```cpp
float get_altitude_estimate() {
    AP_Baro &baro = AP::baro();
    RangeFinder *rf = AP::rangefinder();

    float baro_alt = baro.get_altitude();

    // Use rangefinder when close to ground
    if (rf != nullptr && rf->has_data_orient(ROTATION_PITCH_270)) {
        float rf_dist = rf->distance_orient(ROTATION_PITCH_270);
        float max_rf = rf->max_distance_orient(ROTATION_PITCH_270);

        if (rf->status_orient(ROTATION_PITCH_270) == RangeFinder::Status::Good) {
            // Blend based on altitude
            if (rf_dist < max_rf * 0.7f) {
                // Trust rangefinder more when low
                return rf_dist;
            }
        }
    }

    return baro_alt;
}
```

### Use AHRS for Fused State
```cpp
void get_vehicle_state() {
    AP_AHRS &ahrs = AP::ahrs();

    // Attitude (already fused from IMU, compass, GPS)
    float roll = ahrs.roll;
    float pitch = ahrs.pitch;
    float yaw = ahrs.yaw;

    // Position (fused EKF estimate)
    Location loc;
    if (ahrs.get_location(loc)) {
        // Valid position
    }

    // Velocity (fused from GPS, IMU, baro)
    Vector3f vel;
    if (ahrs.get_velocity_NED(vel)) {
        float vn = vel.x;  // North
        float ve = vel.y;  // East
        float vd = vel.z;  // Down
    }

    // Height above ground
    float hagl;
    if (ahrs.get_hagl(hagl)) {
        // hagl in meters
    }
}
```

---

## Custom Backend Implementation

### Backend Class Template
```cpp
// AP_Baro_MyDevice.h
#pragma once

#include "AP_Baro_Backend.h"

class AP_Baro_MyDevice : public AP_Baro_Backend {
public:
    AP_Baro_MyDevice(AP_Baro &baro, AP_HAL::Device &dev);

    // Required: copy data to frontend
    void update() override;

    // Factory method for probe
    static AP_Baro_Backend *probe(AP_Baro &baro, AP_HAL::Device &dev);

private:
    bool _init();
    void _timer();  // Called from timer thread

    AP_HAL::Device *_dev;
    uint8_t _instance;

    // Thread-safe data accumulation
    float _pressure_sum;
    uint32_t _pressure_count;
    float _temperature;
};
```

### Backend Implementation
```cpp
// AP_Baro_MyDevice.cpp
#include "AP_Baro_MyDevice.h"

extern const AP_HAL::HAL &hal;

AP_Baro_MyDevice::AP_Baro_MyDevice(AP_Baro &baro, AP_HAL::Device &dev)
    : AP_Baro_Backend(baro)
    , _dev(&dev)
{
}

AP_Baro_Backend *AP_Baro_MyDevice::probe(AP_Baro &baro, AP_HAL::Device &dev) {
    AP_Baro_MyDevice *sensor = NEW_NOTHROW AP_Baro_MyDevice(baro, dev);
    if (!sensor || !sensor->_init()) {
        delete sensor;
        return nullptr;
    }
    return sensor;
}

bool AP_Baro_MyDevice::_init() {
    WITH_SEMAPHORE(_dev->get_semaphore());

    // Check device ID
    uint8_t id;
    if (!_dev->read_registers(REG_ID, &id, 1) || id != EXPECTED_ID) {
        return false;
    }

    // Configure device
    _dev->write_register(REG_CONFIG, CONFIG_VALUE);

    // Register with frontend
    _instance = _frontend.register_sensor();

    // Set up periodic timer (50Hz)
    _dev->register_periodic_callback(20000,
        FUNCTOR_BIND_MEMBER(&AP_Baro_MyDevice::_timer, void));

    return true;
}

void AP_Baro_MyDevice::_timer() {
    // Read raw data from device
    uint8_t buf[6];
    if (!_dev->read_registers(REG_DATA, buf, sizeof(buf))) {
        return;
    }

    // Convert raw to pressure/temp
    float pressure = convert_pressure(buf);
    float temp = convert_temperature(buf);

    // Accumulate with thread safety
    WITH_SEMAPHORE(_sem);
    _pressure_sum += pressure;
    _pressure_count++;
    _temperature = temp;
}

void AP_Baro_MyDevice::update() {
    WITH_SEMAPHORE(_sem);

    if (_pressure_count == 0) {
        return;
    }

    // Average and copy to frontend
    float avg_pressure = _pressure_sum / _pressure_count;
    _copy_to_frontend(_instance, avg_pressure, _temperature);

    // Reset accumulators
    _pressure_sum = 0;
    _pressure_count = 0;
}
```

### Register Backend in Frontend
```cpp
// In AP_Baro.cpp init()
void AP_Baro::init() {
    // ... existing probes ...

    // Add your device probe
    probe_i2c_dev(AP_Baro_MyDevice::probe, bus, MY_DEVICE_I2C_ADDR);
}
```
