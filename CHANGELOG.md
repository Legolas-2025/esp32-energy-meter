# Changelog
All notable changes to this project will be documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
## [2.1.0] - 2026-09-17
### Changed
- **ESPHome compatibility**: Configuration now requires **ESPHome 2026.9.0 or newer**.
  The deprecated `register_count` and `response_size` keys have been removed
  from non-RAW / non-text `modbus_controller` sensors, and `command_throttle`
  on the `modbus_controller` hub no longer has any effect. The configuration
  has been updated to match the new Modbus sizing model.
- **Modbus register sizing** is now derived automatically from `value_type`
  (e.g. `U_DWORD` ⇒ 4 bytes / 2 registers). The redundant
  `register_count: 1` and `response_size: 4` lines on every sensor have been
  removed.
- **Modbus command pacing** is now configured with `turnaround_time: 50ms`
  on the `modbus:` hub, replacing `command_throttle: 50ms` on
  `modbus_controller`.
### Fixed
- **Validation failure** on ESPHome 2026.9.0
  (`'register_count' has been removed`). The release no longer accepts the
  previous `register_count` / `response_size` pair on `value_type`-based
  sensors; the new file validates cleanly out of the box.
- **Second-channel current direction register** (`direction2`) was reading
  from `0x004E`, the same register as `direction1`, producing a duplicate
  of channel 1's direction bit. The address has been corrected to
  `0x0056` so the second channel reports its own current direction.
### Documentation
- Added an "ESPHome 2026.9.0 Migration" section to
  `docs/configuration-guide.md` describing the removed keys and the
  `turnaround_time` replacement.
- Added a new `register_count` validation error entry to
  `docs/troubleshooting.md` with copy-pasteable before/after snippets.

---
## [2.0.0] - 2025-12-20
### Added
- Enhanced WiFi connectivity monitoring using signal strength
- OLED burn-in protection with automatic screen clearing
- Professional Google Fonts integration for display
- Comprehensive hardware setup documentation
- Complete ESPHome configuration guide
- Extensive troubleshooting documentation
- Home Assistant integration tutorial
- Advanced features and customization guide
- Complete API reference documentation
- System health monitoring sensors
- Memory usage and uptime tracking
- Template sensors for power quality analysis
### Changed
- WiFi status from generic `platform: status` to signal strength-based monitoring
- Display update intervals optimized for performance (3 seconds)
- Modbus communication throttling improved (50ms)
- Font loading efficiency enhanced
- Documentation structure reorganized and expanded
- Security through anonymization of personal credentials
### Fixed
- False WiFi "disconnected" alerts after device boot
- OLED screen burn-in from static content display
- Missing Home Assistant entity state classes for energy dashboard
- Inadequate troubleshooting guidance for common issues
- Lack of professional documentation for community sharing
### Enhanced
- User experience through accurate connectivity status
- Display readability with professional typography
- System reliability through optimized communication intervals
- Documentation quality for educational value
- Community readiness through comprehensive guides
### Security
- Personal API keys and passwords anonymized
- Proper secret management system implementation
- Mandatory encryption for Home Assistant API
- Network security requirements documented
### Documentation
- Hardware assembly guide with safety warnings
- Step-by-step configuration tutorial
- 400+ lines of troubleshooting solutions
- Complete entity and service reference
- Home Assistant energy dashboard setup
- Advanced customization options
### Performance
- Optimized sensor update intervals (3s)
- Efficient Modbus command throttling
- Memory usage optimization
- Network communication improvements
---
## [1.0.0] - Original Project
### Original Features
- Basic ESP32 energy monitoring with JSY meter
- Simple OLED display for power readings
- Home Assistant API integration
- Modbus RTU communication
- Basic WiFi connectivity
### Original Author
- Giovanni Aggiustatutto
- Project: "DIY Smart Energy Meter With ESP32 + Home Assistant"
- License: Creative Commons BY-NC-SA 4.0
- Source: https://www.instructables.com/DIY-Smart-Energy-Meter-With-ESP32-Home-Assistant/
### Original Capabilities
- Dual channel power monitoring
- Voltage, current, and frequency measurement
- Energy consumption tracking
- Power factor calculation
- Basic Home Assistant integration

---
## Version Numbering
This project uses semantic versioning (SemVer):
- \*\*MAJOR\*\* version when you make incompatible API changes
- \*\*MINOR\*\* version when you add functionality in a backwards compatible manner
- \*\*PATCH\*\* version when you make backwards compatible bug fixes
### Version 2.0.0 Rationale
- \*\*MAJOR\*\*: Significant architectural improvements and new features
- \*\*Enhanced WiFi monitoring\*\*: Major reliability improvement
- \*\*OLED burn-in protection\*\*: New functionality
- \*\*Comprehensive documentation\*\*: Major value addition
- \*\*Professional presentation\*\*: Significant user experience improvement
### Migration from 1.0.0 to 2.0.0
1. Hardware remains compatible (no changes required)
2. Configuration file needs updating (new YAML)
3. Personal credentials need configuring (secrets.yaml)
4. Documentation benefits from new guides
5. Enhanced features become immediately available
### Migration from 2.0.0 to 2.1.0
1. **Upgrade ESPHome** to **2026.9.0 or newer** before flashing this configuration
2. No secrets changes required
3. No wiring changes required
4. Re-validate with `esphome config esp32-energy-meter.yaml` before uploading
5. Re-flash with `esphome run esp32-energy-meter.yaml`
### Future Versioning
- \*\*2.1.0\*\*: ESPHome 2026.9.0 compatibility (this release)
- \*\*2.2.0\*\*: Solar panel monitoring support
- \*\*3.0.0\*\*: Mobile application integration
- \*\*1.x.x\*\*: Will refer to original project versions
