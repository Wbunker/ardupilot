# Getting Started as an ArduPilot Contributor

This guide will help you go from zero to making your first contribution to ArduPilot.

## Prerequisites

### Required Knowledge
- **C++ proficiency**: ArduPilot core is C++11/14
- **Git/GitHub workflow**: Forking, branching, pull requests
- **Basic embedded systems concepts**: Helpful but can learn as you go
- **Control theory basics**: For working on flight controllers

### System Requirements
- **Linux** (native or WSL on Windows) - strongly recommended
- **macOS** - fully supported
- **Windows WSL2** - good alternative to native Linux

## Step 1: Set Up Your Development Environment

### Clone the Repository
```bash
git clone --recursive https://github.com/ArduPilot/ardupilot.git
cd ardupilot
```

### Install Dependencies (Ubuntu/Debian)
```bash
# Run the setup script
Tools/environment_install/install-prereqs-ubuntu.sh -y

# Reload shell to pick up environment changes
. ~/.profile
```

### macOS Setup
```bash
# Install Homebrew if not present
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install dependencies
brew install gcc-arm-none-eabi gawk python3
pip3 install future empy pexpect
```

### Verify Installation
```bash
# Configure for simulation
./waf configure --board sitl

# Build ArduCopter
./waf copter

# Should produce: build/sitl/bin/arducopter
```

## Step 2: Run Your First Simulation

### Start SITL (Software In The Loop)
```bash
cd ArduCopter

# Start with map and console
sim_vehicle.py --map --console

# Or start fresh (wipe parameters)
sim_vehicle.py -w
```

### Basic SITL Commands (in MAVProxy console)
```
arm throttle          # Arm motors
mode GUIDED           # Switch to guided mode
takeoff 10            # Take off to 10m
mode LAND             # Land
disarm                # Disarm motors
```

### Connect with Ground Control Station
- SITL listens on UDP port 14550
- Open Mission Planner/QGroundControl and connect to `udp:127.0.0.1:14550`

## Step 3: Understand the Codebase

### Recommended Learning Path

1. **Read the Library Descriptions**
   - https://ardupilot.org/dev/docs/apmcopter-programming-libraries.html

2. **Run Example Sketches**
   ```bash
   ./waf configure --board sitl
   ./waf examples  # Build all examples

   # Run an example
   ./build/sitl/examples/INS_generic
   ```

3. **Study a Flight Mode**
   - Start with `ArduCopter/mode_loiter.cpp` - well-documented
   - Trace the code flow from `Mode::run()` through control loops

4. **Follow the Tutorial**
   - https://ardupilot.org/dev/docs/learning-the-ardupilot-codebase.html

### Key Files to Study First

| File | Why Important |
|------|---------------|
| `ArduCopter/Copter.h` | Main class definition |
| `ArduCopter/Copter.cpp` | Scheduler tasks, setup |
| `ArduCopter/mode.h` | Flight mode base class |
| `ArduCopter/mode_althold.cpp` | Simple mode to study |
| `libraries/AP_HAL/HAL.h` | Hardware abstraction interface |
| `libraries/AC_AttitudeControl/` | Core attitude controller |

## Step 4: Find Your First Contribution

### Good First Issues
- Check GitHub for "good first issue" label:
  https://github.com/ArduPilot/ardupilot/labels/GoodFirstIssue

### Types of Contributions

1. **Documentation Fixes**
   - Typos, unclear explanations
   - Add code comments
   - Wiki improvements

2. **Bug Fixes**
   - Start with simple, well-defined bugs
   - Look for issues with reproduction steps

3. **New Sensor Drivers**
   - Add support for new hardware
   - Follow existing drivers as templates

4. **Test Improvements**
   - Add missing unit tests
   - Improve SITL test coverage

5. **Code Cleanup**
   - Remove dead code
   - Improve code style consistency

## Step 5: Make Your Changes

### Create a Branch
```bash
git checkout -b feature/my-new-feature
```

### Coding Standards
- **4 spaces** for indentation (no tabs)
- **Unix line endings** (LF, not CRLF)
- Follow existing code style in the file you're editing
- No commented-out code in commits
- Write descriptive commit messages

### Commit Message Format
```
Subsystem: brief description

Longer explanation of what changed and why.
Keep lines under 72 characters.
```

Example:
```
ArduCopter: add new SUPER_LOITER flight mode

This mode combines altitude hold with automatic
obstacle avoidance. Activates proximity sensors
when available and adjusts position to maintain
safe distance from obstacles.
```

### Test Your Changes

1. **Build for SITL**
   ```bash
   ./waf configure --board sitl
   ./waf copter
   ```

2. **Run SITL Tests**
   ```bash
   # Run specific test
   ./Tools/autotest/autotest.py test.Copter.AltHold

   # Run all Copter tests
   ./Tools/autotest/autotest.py build.Copter test.Copter
   ```

3. **Build for Real Hardware**
   ```bash
   ./waf configure --board CubeBlack
   ./waf copter
   ```

## Step 6: Submit Your Pull Request

### Before Submitting
- [ ] Code builds without errors
- [ ] Follows coding standards
- [ ] Tests pass
- [ ] Commit messages are descriptive
- [ ] Branch is up-to-date with master

### Rebase onto Latest Master
```bash
git fetch origin
git rebase origin/master
```

### Push to Your Fork
```bash
git push -u origin feature/my-new-feature
```

### Create Pull Request
1. Go to https://github.com/ArduPilot/ardupilot
2. Click "New Pull Request"
3. Select your fork and branch
4. Fill out the PR template:
   - Clear description of changes
   - Testing performed
   - Which vehicles/boards affected

### PR Best Practices
- Keep changes small and focused
- One logical change per commit
- Respond promptly to review feedback
- Be patient - maintainers are volunteers

## Step 7: Join the Community

### Communication Channels

| Channel | Purpose |
|---------|---------|
| [Discussion Forum](https://discuss.ardupilot.org) | Q&A, help, announcements |
| [Discord](https://ardupilot.org/discord) | Real-time chat |
| [Weekly Dev Calls](https://ardupilot.org/dev/docs/ardupilot-mumble-server.html) | Voice meetings |
| [GitHub Issues](https://github.com/ArduPilot/ardupilot/issues) | Bug reports, features |

### Weekly Developer Calls
- Join to discuss ongoing work
- Present your planned changes
- Get feedback before implementation
- Label your PR with "DevCallTopic" to discuss

## Common Development Tasks

### Adding a New Parameter
```cpp
// In Parameters.cpp
// @Param: MY_PARAM
// @DisplayName: My Parameter
// @Description: Does something cool
// @Range: 0 100
// @User: Standard
AP_GROUPINFO("MY_PARAM", 1, ClassName, my_param, 50),
```

### Adding a New Flight Mode (Copter)

1. Create `mode_mymode.cpp` and `mode_mymode.h`
2. Add enum in `mode.h`
3. Register in `mode.cpp` switch statement
4. Add parameter for mode switch

### Adding a New Sensor Driver

1. Create `libraries/AP_MySensor/`
2. Implement `AP_MySensor.h` and `.cpp`
3. Add to vehicle's `wscript` dependencies
4. Instantiate in vehicle class

### Adding a MAVLink Message

1. Define in `modules/mavlink/message_definitions/`
2. Regenerate with `./modules/mavlink/pymavlink/tools/mavgen.py`
3. Implement handler in `GCS_MAVLink/`
4. Test with MAVProxy

## Debugging Tips

### Using GDB with SITL
```bash
# Build with debug symbols
./waf configure --board sitl --debug
./waf copter

# Run under GDB
gdb ./build/sitl/bin/arducopter
(gdb) run -S
```

### Log Analysis
- Use Mission Planner's log viewer
- MAVExplorer: `mavexplorer.py mylog.bin`
- Plot specific values from DataFlash logs

### Printf Debugging
```cpp
// Use GCS_SEND_TEXT for output visible in GCS
gcs().send_text(MAV_SEVERITY_INFO, "MyValue: %f", my_value);

// Use hal.console for SITL console output
hal.console->printf("Debug: %d\n", value);
```

## Resources

### Documentation
- **Developer Wiki**: https://ardupilot.org/dev/
- **Learning the Codebase**: https://ardupilot.org/dev/docs/learning-the-ardupilot-codebase.html
- **Contributing Guide**: https://ardupilot.org/dev/docs/contributing.html

### Video Tutorials
- Search YouTube for "ArduPilot development"
- Developer call recordings on YouTube

### Books/Papers
- ArduPilot uses Extended Kalman Filters - study EKF theory
- Control theory textbooks for flight control understanding

## Quick Reference Commands

```bash
# Build Commands
./waf configure --board <board>    # Configure build
./waf copter                       # Build Copter
./waf plane                        # Build Plane
./waf clean                        # Clean build
./waf list_boards                  # Show supported boards

# SITL Commands
sim_vehicle.py -v ArduCopter       # Run Copter simulation
sim_vehicle.py -v ArduPlane        # Run Plane simulation
sim_vehicle.py --map --console     # With visualization
sim_vehicle.py -w                  # Wipe parameters

# Testing
./Tools/autotest/autotest.py test.Copter     # Run Copter tests
./Tools/autotest/autotest.py --list          # List available tests

# Git Workflow
git checkout -b feature/name       # Create branch
git rebase origin/master           # Update branch
git push -u origin branch-name     # Push to fork
```

## Your Roadmap

### Month 1: Foundation
- [ ] Set up development environment
- [ ] Build and run SITL successfully
- [ ] Study codebase structure
- [ ] Join Discord/forum

### Month 2: First Contribution
- [ ] Find a good first issue
- [ ] Submit first PR (documentation/small fix)
- [ ] Get familiar with review process

### Month 3+: Deeper Involvement
- [ ] Tackle more complex issues
- [ ] Join weekly developer calls
- [ ] Propose new features
- [ ] Help review other PRs

Welcome to the ArduPilot community!
