# ArduPilot AI Documentation

This folder contains AI-generated documentation to help understand and contribute to the ArduPilot project.

## Contents

| Document | Description |
|----------|-------------|
| [01-ARDUPILOT-OVERVIEW.md](./01-ARDUPILOT-OVERVIEW.md) | High-level project overview, statistics, vehicle types |
| [02-CODEBASE-ARCHITECTURE.md](./02-CODEBASE-ARCHITECTURE.md) | Detailed code structure, layers, data flow |
| [03-GETTING-STARTED-CONTRIBUTOR.md](./03-GETTING-STARTED-CONTRIBUTOR.md) | Step-by-step guide to becoming a contributor |
| [04-LIBRARY-REFERENCE.md](./04-LIBRARY-REFERENCE.md) | Reference for all 153+ shared libraries |
| [05-AP_BARO-DEEP-DIVE.md](./05-AP_BARO-DEEP-DIVE.md) | Deep dive into sensor library architecture |
| [06-SENSOR-LIBRARY-REFERENCE.md](./06-SENSOR-LIBRARY-REFERENCE.md) | Complete sensor API reference |

## Quick Start

1. **New to ArduPilot?** Start with `01-ARDUPILOT-OVERVIEW.md`
2. **Want to understand the code?** Read `02-CODEBASE-ARCHITECTURE.md`
3. **Ready to contribute?** Follow `03-GETTING-STARTED-CONTRIBUTOR.md`
4. **Need library details?** Reference `04-LIBRARY-REFERENCE.md`

## Key Takeaways

### Languages
- **C++** (~95%): Core autopilot code
- **Python** (~3%): Build system, tools, testing
- **Lua** (~2%): User scripting

### Architecture
```
Vehicle Layer (ArduCopter, ArduPlane, Rover, ArduSub)
         │
         ▼
Shared Libraries (153+ in libraries/)
         │
         ▼
Hardware Abstraction Layer (AP_HAL_*)
         │
         ▼
Hardware / SITL Simulation
```

### Essential Commands
```bash
# Setup
git clone --recursive https://github.com/ArduPilot/ardupilot.git
./Tools/environment_install/install-prereqs-ubuntu.sh -y

# Build
./waf configure --board sitl
./waf copter

# Simulate
cd ArduCopter && sim_vehicle.py --map --console
```

### Community Resources
- Forum: https://discuss.ardupilot.org
- Discord: https://ardupilot.org/discord
- Wiki: https://ardupilot.org/dev/

---
*Generated: December 2025*
