# ESP32 Energy Meter Project - GitHub Repository Summary

> **Latest release: v2.1.0 (2026-09-17)** — adds ESPHome 2026.9.0 compatibility
> (`register_count` / `response_size` removed, `command_throttle` → `turnaround_time`).
> See [CHANGELOG.md](CHANGELOG.md) for the full release history.

## 🎯 Project Overview

This GitHub repository contains a complete, production-ready implementation of a smart energy monitoring system based on ESP32 and ESPHome firmware. The project enhances the original work by Giovanni Aggiustatutto with several advanced features and optimizations.

## 📊 Original vs Enhanced Features

### Original Project (Giovanni Aggiustatutto)
- **Basic Energy Monitoring**: Power, voltage, current measurement
- **Home Assistant Integration**: Basic API connectivity
- **Simple Display**: Power readings on OLED
- **Standard Modbus**: Basic JSY energy meter communication

### Enhanced Features Added (v2.0.0)
- ✅ **Enhanced WiFi Monitoring**: Signal strength-based connection status
- ✅ **OLED Burn-in Protection**: Automatic screen clearing every 30 minutes
- ✅ **Professional Display**: Google Fonts integration with proper typography
- ✅ **Optimized Performance**: Configurable update intervals and command throttling
- ✅ **Advanced Sensor Filtering**: Noise reduction and calibration capabilities
- ✅ **System Health Monitoring**: Memory, uptime, and connectivity tracking
- ✅ **Professional Documentation**: Complete guides and troubleshooting
- ✅ **Home Assistant Optimization**: Proper state classes and device classes

### New in v2.1.0
- ✅ **ESPHome 2026.9.0 compatibility**: Configuration updated to the new Modbus
  sizing model — `value_type` now drives register count, and command pacing
  moved from `command_throttle` to `turnaround_time` on the `modbus:` hub.
- ✅ **Bug fix — `direction2` register**: The second channel's current-direction
  sensor was reading register `0x004E` (same as `direction1`). It now reads
  `0x0056`, so each channel reports its own direction bit.
- ✅ **New `register_count` troubleshooting entry** in
  `docs/troubleshooting.md` with copy-pasteable before/after YAML.
- ✅ **New "ESPHome 2026.9.0 Migration" section** in
  `docs/configuration-guide.md`.

## 🏗️ Repository Structure

```
esp32-energy-meter/
├── 📄 esp32-energy-meter.yaml          # Main ESPHome configuration (anonymized, requires ESPHome ≥ 2026.9.0)
├── 📄 README.md                        # Project overview and quick start
├── 📄 LICENSE                          # Creative Commons BY-NC-SA 4.0 license
├── 📄 PROJECT_SUMMARY.md               # This file - project overview
├── 📄 CHANGELOG.md                     # Release history (1.0.0, 2.0.0, 2.1.0)
├── 📁 docs/                           # Technical documentation
│   ├── 📄 hardware-setup.md           # Complete hardware assembly guide
│   ├── 📄 configuration-guide.md      # ESPHome configuration tutorial (with 2026.9.0 migration section)
│   ├── 📄 troubleshooting.md          # Comprehensive troubleshooting guide (with `register_count` fix entry)
│   ├── 📄 api-reference.md            # Complete API and entity reference
│   ├── 📄 Home-Assistant-Integration.md # HA setup and usage guide
│   └── 📄 Advanced-Features.md         # Customization and advanced features
```

## 🔧 Technical Specifications

### Hardware Requirements
- **ESP32 Wemos D1 Mini** (compact ESP32 development board - recommended)
- **JSY-MK-194G Energy Meter** with Modbus RTU over RS485
- **SSD1306 128x64 OLED Display** (I2C, address 0x3C)
- **RS485 to TTL Converter** for Modbus communication
- **Meanwell APV-8-5 5V 8W Power Supply** for system power

### Pin Configuration
```
UART (Modbus):
- TX: GPIO17
- RX: GPIO16

I2C (OLED):
- SDA: GPIO21
- SCL: GPIO22

Power:
- 5V: External power supply to ESP32 +5V pin (main power)
- 3.3V: ESP32 internal regulator powers OLED, RS485 module, and energy meter
```

### Communication Protocols
- **Modbus RTU**: 4800 baud, 8N1 (8 data bits, No parity, 1 stop bit)
- **I2C**: Standard 400kHz for OLED communication
- **WiFi**: 2.4GHz, WPA2/WPA3 security
- **Home Assistant API**: Encrypted communication

### Software Requirements (v2.1.0)
- **ESPHome ≥ 2026.9.0** — required for the new Modbus sizing model
- **Home Assistant** (optional but recommended) — for energy dashboard integration
- **Python ≥ 3.9** — for the `pip install esphome` toolchain

## 📊 Monitored Parameters

### Primary Measurements
| Parameter | Channel | Unit | Accuracy | Update Interval |
|-----------|---------|------|----------|-----------------|
| Power | 1&2 | W | 0.1W | 3s |
| Voltage | 1&2 | V | 0.1V | 3s |
| Current | 1 | A | 0.1A | 3s |
| Current | 2 | A | 0.0001A | 3s |
| Energy | 1&2 | kWh | 0.1kWh | 3s |
| Frequency | Single | Hz | 0.1Hz | 3s |

### System Monitoring
| Parameter | Unit | Update Interval | Purpose |
|-----------|------|----------------|---------|
| WiFi Signal | dBm | 10s | Connection quality |
| WiFi Status | Boolean | 10s | Connectivity indicator |
| System Uptime | seconds | 30s | Reliability tracking |
| Free Memory | bytes | 60s | Performance monitoring |

## 🎨 Display Features

### Current Implementation
- **Title**: "Energy Meter" (Google Fonts Baloo Bhaijaan 2, 18px)
- **WiFi Status**: Material Icons WiFi/WiFi-off indicators
- **Power Display**: Large centered power reading (32px font)
- **Voltage/Current**: Bottom corner display (18px font)
- **Rotation**: 180° for optimal viewing angle

### OLED Burn-in Protection
- **Automatic Clearing**: Every 30 minutes (1800 seconds)
- **Implementation**: Time-based clearing using Home Assistant time sync
- **Display Preservation**: Prevents permanent screen damage from static content

### Enhanced Typography
```yaml
font:
  - Baloo Bhaijaan 2 (500): Body text and titles
  - Baloo Bhaijaan 2 (700): Power display and emphasis
  - Material Symbols Outlined: WiFi status icons
```

## 🔧 WiFi Connectivity Enhancement

### Problem Solved
**Original Issue**: WiFi icon showed "disconnected" despite successful data transmission due to generic `platform: status` checking multiple connectivity types (WiFi, API, DNS, MQTT).

### Solution Implemented
```yaml
# Enhanced WiFi monitoring
binary_sensor:
  - platform: template
    name: "WiFi Connection Status"
    id: connection_status
    lambda: !lambda
      return id(wifi_signal_strength).state > -70;  # Signal-based check

sensor:
  - platform: wifi_signal
    name: "WiFi Signal Strength"
    id: wifi_signal_strength
    update_interval: 10s
```

### Benefits
- **Accurate Status**: Only shows disconnected when WiFi signal is actually weak
- **No False Positives**: Eliminates spurious disconnection alerts
- **Signal Monitoring**: Provides actual signal strength data for analysis
- **Configurable Threshold**: -70 dBm threshold can be adjusted as needed

## 🔌 Modbus Communication (v2.1.0)

### What changed in ESPHome 2026.9.0
ESPHome 2026.9.0 reworked how `modbus_controller` sensors are sized and paced.
The configuration in this repo was updated to match:

| Setting | Old (≤ 2026.x) | New (≥ 2026.9.0) |
|---|---|---|
| Register count | `register_count: N` per sensor | **Derived from `value_type`** (e.g. `U_DWORD` ⇒ 2 registers) |
| Per-sensor byte size | `response_size: N` on every sensor | Only valid for RAW and text sensors |
| Command pacing | `command_throttle: 50ms` on `modbus_controller` | `turnaround_time: 50ms` on the `modbus:` hub |

### Current Modbus setup
```yaml
modbus:
  id: modbus1
  turnaround_time: 50ms          # NEW: command pacing lives on the hub
  modbus_controller:
    - id: jsymk
      address: 0x1
      modbus_id: modbus1
      update_interval: 3s
      # NOTE: do NOT set `command_throttle` here — it has no effect in
      #       ESPHome >= 2026.9.0 and will be removed in 2027.2.0.
```

```yaml
# Typical sensor (no register_count / response_size needed)
sensor:
  - platform: modbus_controller
    modbus_controller_id: jsymk
    id: power2
    address: 0x0052
    register_type: holding
    value_type: U_DWORD          # → automatically reads 2 registers / 4 bytes
    filters:
      - multiply: 0.0001
```

## 🏠 Home Assistant Integration

### Auto-Discovery Features
- **Device Discovery**: Automatic appearance in Home Assistant
- **Entity Creation**: All sensors automatically recognized
- **Energy Dashboard**: Ready for energy monitoring configuration

### Entity Categories
1. **Primary Power Sensors**: Used for display and primary monitoring
2. **System Health**: WiFi, uptime, memory monitoring
3. **Energy Analytics**: Cumulative energy, power factors
4. **Historical Data**: Peak power, frequency tracking

### Energy Dashboard Ready
```yaml
# Recommended configuration
consumption:
  - sensor.esp32_energy_meter_energy_2  # Primary circuit
  - sensor.esp32_energy_meter_energy_1  # Secondary circuit

# Proper state classes
state_class: total      # For cumulative measurements
device_class: energy    # For energy entities
```

## 🔒 Security and Privacy

### Anonymization Applied
- **API Keys**: Placeholder values (CHANGE_THIS_TO_YOUR_API_KEY)
- **WiFi Credentials**: Uses secrets.yaml system (!secret wifi_ssid)
- **OTA Password**: Placeholder value (CHANGE_THIS_OTA_PASSWORD)
- **IP Addresses**: Generic placeholder (192.168.1.XXX)

### Security Best Practices
- **API Encryption**: Mandatory for Home Assistant communication
- **WiFi Security**: Requires WPA2/WPA3 encryption
- **OTA Protection**: Password-protected firmware updates
- **Secret Management**: External secrets.yaml file for credentials

## 📚 Documentation Quality

### Comprehensive Guides
1. **Hardware Setup**: Complete assembly instructions with safety warnings
2. **Configuration Guide**: Step-by-step ESPHome setup tutorial (with v2.1.0
   ESPHome 2026.9.0 migration section)
3. **Home Assistant Integration**: Detailed HA setup and usage guide
4. **Troubleshooting**: 500+ lines of common issues and solutions, including
   a dedicated `register_count` validation error entry
5. **API Reference**: Complete entity and service documentation
6. **Advanced Features**: Customization and optimization guide

### Professional Standards
- **Safety Warnings**: Prominent high-voltage warnings throughout
- **Code Examples**: Practical, tested configuration snippets
- **Troubleshooting**: Systematic diagnostic procedures
- **Community Support**: Clear paths for getting help

## ⚡ Performance Optimizations

### Communication Efficiency
```yaml
# Optimized Modbus settings (v2.1.0 — ESPHome >= 2026.9.0)
modbus:
  id: modbus1
  turnaround_time: 50ms       # Balanced responsiveness; lives on the `modbus:` hub
  modbus_controller:
    - id: jsymk
      update_interval: 3s     # Balanced responsiveness

# Display optimization
display:
  update_interval: 3s          # Matches sensor updates
  rotation: 180°               # Optimal viewing
```

### Memory Management
- **Font Loading**: Efficient Google Fonts integration
- **Sensor Filtering**: Built-in noise reduction
- **Update Intervals**: Configurable for performance balance

## 🌟 Key Achievements

### Technical Excellence
1. **Enhanced Reliability**: Solved WiFi false positive issues
2. **Professional Display**: Clean, readable interface with burn-in protection
3. **Comprehensive Monitoring**: Full system health tracking
4. **Home Assistant Optimization**: Proper entity configuration for energy dashboards
5. **Forward-compatible Modbus**: Updated to ESPHome 2026.9.0 sizing model

### Documentation Excellence
1. **Complete Hardware Guide**: From components to final assembly
2. **Professional Configuration**: Step-by-step tutorials
3. **Extensive Troubleshooting**: Common issues and solutions
4. **API Documentation**: Complete reference for all entities
5. **Migration Notes**: Dedicated ESPHome 2026.9.0 upgrade section

### Community Value
1. **Fork-Ready**: Anonymized configuration ready for immediate use
2. **Educational**: Comprehensive guides for learning ESPHome
3. **Scalable**: Configurable for different energy monitoring needs
4. **Safe**: Proper safety warnings and best practices

## 🎯 Target Audience

### Primary Users
- **DIY Enthusiasts**: Building energy monitoring systems
- **Home Automation Users**: Integrating with Home Assistant
- **Energy Conscious**: Monitoring household electricity usage
- **Technical Learners**: Understanding ESPHome and Modbus

### Skill Levels
- **Beginner**: Complete guides with step-by-step instructions
- **Intermediate**: Configuration customization and optimization
- **Advanced**: Custom sensors, filters, and automation

## 🔄 Versioning

| Version | Date | Highlights |
|---|---|---|
| **2.1.0** | 2026-09-17 | ESPHome 2026.9.0 compatibility, `direction2` register fix, expanded docs |
| 2.0.0 | 2025-12-20 | WiFi signal monitoring, OLED burn-in protection, full docs |
| 1.0.0 | Original | Giovanni Aggiustatutto's original project |

## 🔄 Migration Between Versions

### From v2.0.0 → v2.1.0
1. **Upgrade ESPHome** to **2026.9.0 or newer**:
   ```bash
   pip install --upgrade esphome
   esphome version   # must report 2026.9.0 or newer
   ```
2. **Replace** `esp32-energy-meter.yaml` with the v2.1.0 file.
3. **No secrets, wiring, or hardware changes** are required.
4. **Re-validate** before flashing:
   ```bash
   esphome config esp32-energy-meter.yaml
   ```
5. **Re-flash**:
   ```bash
   esphome run esp32-energy-meter.yaml
   ```
6. **Verify** that `direction2` now reads the channel-2 direction register
   (`0x0056`) instead of duplicating `direction1`. Confirm against your JSY
   meter's register map.

### From v1.0.0 → v2.0.0
1. **Backup Original**: Save existing configuration
2. **Update Hardware**: No hardware changes required
3. **Replace Configuration**: Use enhanced YAML file
4. **Add Secrets**: Configure personal WiFi credentials
5. **Test Integration**: Verify Home Assistant connectivity

## 🔄 Future Enhancement Opportunities

### Potential Additions
1. **MQTT Integration**: Alternative to Home Assistant API
2. **Cloud Connectivity**: Remote monitoring capabilities
3. **Mobile App**: Native mobile application
4. **Machine Learning**: Predictive energy analytics
5. **Solar Integration**: Solar panel monitoring support

### Hardware Expansions
1. **Multiple Energy Meters**: Support for additional circuits
2. **Environmental Sensors**: Temperature, humidity monitoring
3. **Relay Control**: Automatic load switching
4. **Data Logging**: Local storage for historical analysis

## 🏆 Project Impact

This enhanced ESP32 Energy Meter project represents a significant improvement over the original implementation, providing:

- **Enhanced User Experience**: Reliable WiFi monitoring and professional display
- **Production Readiness**: Comprehensive testing and documentation
- **Community Contribution**: Open-source sharing under Creative Commons license
- **Educational Value**: Complete learning resource for ESPHome and energy monitoring
- **Forward Compatibility**: Keeps pace with upstream ESPHome Modbus changes

The project successfully transforms a basic energy monitoring concept into a professional-grade, well-documented, and community-ready solution that maintains the spirit and license of the original work while adding substantial value through technical enhancements and comprehensive documentation.

---

**License**: Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International
**Original Attribution**: Based on work by Giovanni Aggiustatutto
**Enhanced By**: MiniMax Agent
**Repository Purpose**: Community sharing and educational resource
**Latest Stable Release**: [v2.1.0](https://github.com/Legolas-2025/esp32-energy-meter/releases/tag/v2.1.0)
