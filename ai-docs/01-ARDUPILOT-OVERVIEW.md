# ArduPilot Overview Report

**Generated**: December 2025
**Codebase Version**: Master branch (commit 733ddb2b81)

## Executive Summary

ArduPilot is the world's most advanced, full-featured, and reliable open source autopilot software. Under active development since 2010 by a diverse team of professional engineers, computer scientists, and community contributors, it controls almost any vehicle system imaginable.

## Project Statistics

| Metric | Value |
|--------|-------|
| Core Codebase | ~700,000 lines |
| C/C++ Source Files | ~3,292 files |
| Python Scripts | ~270 files |
| Lua Scripts | ~252 files |
| Shared Libraries | 153+ |
| Supported Boards | 100+ |
| License | GPLv3 |

## Supported Vehicle Types

### 1. ArduCopter (Multirotor/Helicopter)
- Multirotors: quadcopters, hexacopters, octocopters
- Traditional helicopters
- Single/dual rotors
- **Wiki**: https://ardupilot.org/copter/

### 2. ArduPlane (Fixed-Wing)
- Conventional fixed-wing aircraft
- VTOL aircraft (quadplanes)
- Gliders and sailplanes
- **Wiki**: https://ardupilot.org/plane/

### 3. Rover (Ground Vehicles)
- Wheeled rovers
- Tracked vehicles
- Boats and sailboats
- Balance bots
- **Wiki**: https://ardupilot.org/rover/

### 4. ArduSub (Underwater)
- ROVs (Remotely Operated Vehicles)
- Submarines
- **Wiki**: http://ardusub.com/

### 5. AntennaTracker
- Automated antenna tracking systems
- **Wiki**: https://ardupilot.org/antennatracker/

### 6. Blimp
- Airship/dirigible control

### 7. AP_Periph
- CAN peripheral devices (GPS units, compasses, airspeed sensors)

## Primary Languages

### C++ (Primary - ~95% of core code)
- All vehicle firmware
- All shared libraries
- Hardware abstraction layer
- Control algorithms

### Python (~270 files)
- Build system (WAF)
- Test automation
- Development tools
- SITL helpers
- Parameter generation

### Lua (~252 scripts)
- User scripting for custom behaviors
- Mission customization
- Sensor integration scripts

### Other
- XML: MAVLink message definitions
- Hardware definition files (.hwdef)
- Shell scripts for tooling

## Build System: WAF

ArduPilot uses **WAF** (Waf is a Python-based build system):

```bash
# Configure for a board
./waf configure --board CubeBlack

# Build vehicles
./waf copter    # Multicopter
./waf plane     # Fixed-wing
./waf rover     # Ground vehicle
./waf sub       # Underwater

# Build and upload
./waf --targets bin/arducopter --upload
```

## Key Documentation Resources

| Resource | URL |
|----------|-----|
| Main Wiki | https://ardupilot.org |
| Developer Wiki | https://ardupilot.org/dev/ |
| Discussion Forum | https://discuss.ardupilot.org |
| Discord | https://ardupilot.org/discord |
| GitHub | https://github.com/ArduPilot/ardupilot |

## Ground Control Stations

ArduPilot works with multiple ground control stations:
- **Mission Planner** (Windows - full featured)
- **QGroundControl** (Cross-platform)
- **MAVProxy** (Command-line, Python-based)
- **APM Planner 2** (Cross-platform)

## Communication Protocol

ArduPilot uses **MAVLink** (Micro Air Vehicle Link) for all communication:
- Serial and network transport
- Standardized message format
- Supported by all major GCS software
- Extensible for custom messages

## Project Governance

The project is maintained by volunteer developers with specific areas of responsibility:
- **Andrew Tridgell**: Plane, AntennaTracker, core architecture
- **Randy Mackay**: Copter, Rover, AntennaTracker
- **And many more** (see README.md for full list)

Weekly developer calls coordinate development efforts.

## Next Steps

See the following documents in this folder:
- `02-CODEBASE-ARCHITECTURE.md` - Detailed code structure
- `03-GETTING-STARTED-CONTRIBUTOR.md` - How to start contributing
