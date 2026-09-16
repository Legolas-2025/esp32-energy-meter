# Troubleshooting Guide

This comprehensive guide helps you diagnose and resolve common issues with the ESP32 Energy Meter project.

## 🔍 Quick Diagnosis

### System Health Check
Before diving into specific issues, verify these basics:
- [ ] ESP32 powers on and boots successfully
- [ ] WiFi connection is established
- [ ] OLED display shows "Energy Meter" title
- [ ] Home Assistant integration is working
- [ ] Energy meter readings are stable

### First Steps for Any Issue
1. **Check Serial Logs**: Monitor ESP32 output for error messages
2. **Verify Connections**: Ensure all hardware connections are secure
3. **Test Individual Components**: Test ESP32, OLED, and energy meter separately
4. **Check Power Supply**: Verify stable power to all components

### ESPHome version
Always report your ESPHome version when troubleshooting:
```bash
esphome version
```
This project requires **ESPHome 2026.9.0 or newer**. See
[`docs/configuration-guide.md`](configuration-guide.md#esphome-202690-migration)
for the migration notes.

## ⚡ Power and Boot Issues

### ESP32 Not Powering On
**Symptoms**: No LED indicators, no serial output

**Diagnosis**:
```bash
# Check USB cable
esphome logs esp32-energy-meter.yaml --serial /dev/ttyUSB0
```

**Solutions**:
- **USB Cable**: Use data cable (not charge-only cable)
- **Power Supply**: Ensure 5V power supply provides adequate current (≥1A)
- **Reset Button**: Press reset button on ESP32
- **Power Connections**: Verify 3.3V and GND connections

### Boot Loop or Continuous Restart
**Symptoms**: ESP32 restarts repeatedly, no stable operation

**Diagnosis**:
```bash
# Monitor boot sequence
esphome logs esp32-energy-meter.yaml --serial /dev/ttyUSB0
```

**Common Causes & Solutions**:
- **Insufficient Power**: Upgrade to higher current power supply
- **Loose Connections**: Secure all wire connections
- **Memory Issues**: Reduce font loading or increase stack size
- **Watchdog Reset**: Check for infinite loops in code

```yaml
# Fix memory issues
esp32:
  board: esp32dev
  framework:
    type: esp-idf
  psram:
    mode: octal
```

## 📶 WiFi Connection Issues

### Cannot Connect to WiFi
**Symptoms**: No WiFi icon, "Fallback Hotspot" appears

**Diagnosis**:
```yaml
# Check WiFi configuration
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: true  # Skip scanning
```

**Solutions**:
- **Credentials**: Verify SSID and password in secrets.yaml
- **Network Issues**: Check router status and other device connectivity
- **Signal Strength**: Move ESP32 closer to WiFi access point
- **Security**: Ensure WPA2/WPA3 security (avoid WEP)

### Intermittent WiFi Connection
**Symptoms**: WiFi drops frequently, connectivity problems

**Diagnosis**:
```yaml
# Monitor signal strength
sensor:
  - platform: wifi_signal
    name: "WiFi Signal Strength"
    update_interval: 10s
```

**Solutions**:
- **Signal Strength**: Improve WiFi coverage or move closer to router
- **Power Management**: Disable WiFi power saving
- **Channel Interference**: Change WiFi channel on router
- **Hardware Issues**: Check ESP32 WiFi antenna

```yaml
# Optimize WiFi settings
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: true
  output_power: 12.0
  power_save_mode: NONE
```

### WiFi Status Shows "Disconnected" Despite Connection
**Symptoms**: Data flowing to Home Assistant but WiFi icon shows disconnected

**Diagnosis**: This is caused by the generic `platform: status` check

**Solution**: Use enhanced WiFi signal strength monitoring (already implemented):
```yaml
binary_sensor:
  - platform: template
    name: "WiFi Connection Status"
    id: connection_status
    lambda: !lambda
      return id(wifi_signal_strength).state > -70;
```

## 📱 OLED Display Issues

### Display Shows Nothing
**Symptoms**: Blank screen, no text or graphics

**Diagnosis**:
```yaml
# Check I2C configuration
i2c:
  sda: 21
  scl: 22

display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x64"
    address: 0x3C
```

**Solutions**:
- **I2C Address**: Default is 0x3C, try 0x3D if multiple I2C devices
- **Wiring**: Verify SDA (GPIO21) and SCL (GPIO22) connections
- **Power**: Ensure 3.3V and GND connections to OLED
- **Library**: Update ESPHome to a version ≥ 2026.9.0

```bash
# Test I2C device detection
esphome logs esp32-energy-meter.yaml --serial /dev/ttyUSB0
```

### Display Shows Garbled Text or Wrong Characters
**Symptoms**: Corrupted text, strange symbols

**Diagnosis**: Font loading or encoding issues

**Solutions**:
- **Font Loading**: Verify Google Fonts connectivity
- **Character Encoding**: Check font glyphs configuration
- **Memory Issues**: Reduce font sizes or count

```yaml
# Simplify font loading
font:
  - file:
      type: gfonts
      family: Arial
    id: arial_16
    size: 16
```

### Display Refreshes Too Slowly
**Symptoms**: Laggy display updates, delayed readings

**Solutions**:
- **Update Interval**: Reduce display update interval
- **Display Logic**: Optimize lambda function efficiency

```yaml
display:
  update_interval: 1s  # Faster updates
```

### OLED Burn-in Protection Not Working
**Symptoms**: Static text causing screen damage over time

**Diagnosis**: Time synchronization issues

**Solutions**:
```yaml
# Ensure time platform is configured
time:
  - platform: homeassistant
    id: homeassistant_time

# Verify burn-in protection logic
lambda: !lambda |-
  time_t now = id(homeassistant_time).now().timestamp;
  if (now % 1800 == 0) {  // Every 30 minutes
    it.clear();
    return;
  }
```

## 📊 Energy Meter Communication Issues

### No Modbus Communication
**Symptoms**: All sensors show 0, no data from energy meter

**Diagnosis**:
```yaml
# Check Modbus configuration
uart:
  id: mod_bus
  tx_pin: 17
  rx_pin: 16
  baud_rate: 4800
  stop_bits: 1

modbus_controller:
  - id: jsymk
    address: 0x1  # Default JSY address
```

**Solutions**:
- **Wiring**: Verify RS485 A+ and B- connections
- **Address**: Check energy meter address (usually 0x1)
- **Termination**: Add 120Ω termination resistor for long cables
- **Power**: Ensure energy meter is powered on

```bash
# Monitor Modbus communication
esphome logs esp32-energy-meter.yaml --serial /dev/ttyUSB0
```

### Intermittent Sensor Readings
**Symptoms**: Readings appear and disappear randomly

**Diagnosis**: Communication errors or timing issues

**Solutions**:
```yaml
# Optimize communication settings (ESPHome >= 2026.9.0)
modbus:
  id: modbus1
  turnaround_time: 100ms          # More time between commands (on the `modbus:` hub)

  modbus_controller:
    - id: jsymk
      address: 0x1
      modbus_id: modbus1
      update_interval: 5s          # Slower, more reliable
      # NOTE: `command_throttle` was deprecated; use `turnaround_time` above.
```

### Incorrect Sensor Values
**Symptoms**: Readings are wrong but consistent

**Diagnosis**: Calibration or scaling issues

**Solutions**:
```yaml
# Apply calibration corrections
sensor:
  - platform: modbus_controller
    # ... other config ...
    filters:
      - multiply: 0.0001       # Correct scaling
      - offset: -5.0           # Apply offset
      - calibrate_linear:
          - 0.0 -> 0.0
          - 240.0 -> 242.1     # Correct for systematic error
```

## 🏠 Home Assistant Integration Issues

### Device Not Auto-Discovered
**Symptoms**: ESP32 not appearing in Home Assistant

**Solutions**:
- **Network**: Ensure ESP32 and HA are on same network
- **API**: Verify API encryption key is correct
- **Firewall**: Check if network firewall blocks mDNS

### Entities Missing or Not Updating
**Symptoms**: Some sensors don't appear or show "unknown"

**Diagnosis**:
```bash
# Check ESP32 logs for API connection issues
esphome logs esp32-energy-meter.yaml --device 192.168.1.100
```

**Solutions**:
- **API Key**: Ensure correct API key in HA
- **Update Interval**: Check sensor update intervals
- **State Class**: Ensure energy sensors have proper state class

```yaml
# Proper state classes for energy
sensor:
  - platform: modbus_controller
    # ... other config ...
    state_class: total          # For cumulative energy
    device_class: energy        # For energy measurements
```

### Energy Dashboard Not Working
**Symptoms**: Energy sources not recognized

**Solutions**:
- **State Classes**: Ensure sensors have `state_class: total`
- **Device Classes**: Use `device_class: energy`
- **Statistics**: Enable statistics for energy calculations

## 🔧 Configuration Issues

### Compilation Errors
**Symptoms**: ESPHome compilation fails

**Common Issues**:
- **YAML Syntax**: Validate YAML file
- **Sensor IDs**: Ensure all referenced IDs exist
- **Dependencies**: Update ESPHome to **≥ 2026.9.0**

```bash
# Validate configuration
esphome config esp32-energy-meter.yaml

# Clean build
esphome run esp32-energy-meter.yaml --clean
```

### Validation Error: `register_count` has been removed

**Symptoms**: Starting with **ESPHome 2026.9.0**, `esphome config` (or
`esphome run`) fails with:

```
Failed config
sensor.modbus_controller: [source esp32-energy-meter.yaml:156]
  'register_count' has been removed; the number of registers to read is
  now derived from 'value_type' (or 'response_size' for RAW values and
  text sensors). To make one request span extra registers up to the next
  sensor, set 'reuse_previous_range: true' on the NEXT sensor instead;
  for RAW or text block reads set 'response_size' to the byte count; to
  force multi-register writes set 'use_write_multiple: true'.
```

(You may also see a `WARNING [modbus_controller] 'command_throttle' no
longer has any effect and will be removed in 2027.2.0. Command spacing
is handled by the 'modbus' component - use 'turnaround_time' there
instead.`)

**Diagnosis**: The configuration was written for ESPHome **< 2026.9.0**
and still uses two deprecated keys:

1. `register_count` on every `modbus_controller` sensor
2. `response_size` on every typed (non-RAW, non-text) sensor
3. `command_throttle` on the `modbus_controller` hub

**Fix**: Upgrade the configuration to the new sizing model.

**Before** (ESPHome < 2026.9.0):
```yaml
modbus_controller:
  - id: jsymk
    address: 0x1
    modbus_id: modbus1
    update_interval: 3s
    command_throttle: 50ms      # ← deprecated

sensor:
  - platform: modbus_controller
    modbus_controller_id: jsymk
    id: power2
    address: 0x0052
    register_type: holding
    value_type: U_DWORD
    filters:
      - multiply: 0.0001
    register_count: 1           # ← removed
    response_size: 4            # ← removed for typed sensors
```

**After** (ESPHome ≥ 2026.9.0):
```yaml
modbus:
  id: modbus1
  turnaround_time: 50ms        # ← pacing moves to the `modbus:` hub
  modbus_controller:
    - id: jsymk
      address: 0x1
      modbus_id: modbus1
      update_interval: 3s

sensor:
  - platform: modbus_controller
    modbus_controller_id: jsymk
    id: power2
    address: 0x0052
    register_type: holding
    value_type: U_DWORD         # → 4 bytes / 2 registers automatically
    filters:
      - multiply: 0.0001
```

**Apply the fix** to every sensor that still has `register_count` /
`response_size` lines. In this repository the equivalent changes are
already applied — simply pull the latest `esp32-energy-meter.yaml`.

The full migration walkthrough lives in
[`configuration-guide.md`](configuration-guide.md#esphome-202690-migration).

### OTA Update Fails
**Symptoms**: Cannot update firmware over WiFi

**Solutions**:
- **Network**: Ensure stable WiFi connection
- **OTA Password**: Verify correct OTA password
- **IP Address**: Use correct ESP32 IP address

```bash
# Force OTA update
esphome run esp32-energy-meter.yaml --upload-port 192.168.1.100
```

### Memory Issues
**Symptoms**: ESP32 restarts, unstable operation

**Diagnosis**:
```yaml
# Monitor memory usage
sensor:
  - platform: template
    name: "Free Heap"
    lambda: !lambda
      return ESP.getFreeHeap();
```

**Solutions**:
- **Reduce Fonts**: Limit font loading
- **Enable PSRAM**: Use external RAM if available
- **Optimize Code**: Simplify lambda functions

## 📋 Hardware-Specific Issues

### JSY Energy Meter Problems
**Symptoms**: No communication with JSY meter

**Solutions**:
- **Address**: Default address is usually 0x1
- **Wiring**: Check RS485 A+ (Data+) and B- (Data-)
- **Termination**: Add termination resistor for long cables
- **Power**: Ensure meter is properly powered

### RS485 Module Issues
**Symptoms**: No or corrupted Modbus communication

**Solutions**:
- **Direction Control**: Some modules need RE/DE pin control
- **Power**: Ensure proper 3.3V power supply
- **Isolation**: Consider optoisolated RS485 modules

### ESP32 Board Variations
**Symptoms**: Different behavior on different ESP32 boards

**Solutions**:
- **Pin Assignment**: Verify pin numbers for your specific board
- **Flash Mode**: Check if different flash mode needed

## 🔍 Advanced Diagnostics

### Serial Debug Commands
```bash
# Monitor all ESP32 output
esphome logs esp32-energy-meter.yaml --serial /dev/ttyUSB0

# Monitor specific component
esphome logs esp32-energy-meter.yaml --log-level=DEBUG

# Check configuration without building
esphome config esp32-energy-meter.yaml
```

### Network Diagnostics
```bash
# Ping ESP32
ping 192.168.1.100

# Check network connectivity
nmap -p 8266 192.168.1.100  # ESP32 API port
```

### Hardware Testing
```bash
# I2C scanner (add to configuration temporarily)
i2c:
  - id: i2c_component
    sda: 21
    scl: 22
    scan: True  # Scan for I2C devices

# UART testing
uart:
  - id: uart_test
    tx_pin: 17
    rx_pin: 16
    baud_rate: 115200  # Higher baud for testing
```

## 📞 Getting Help

### Before Asking for Help
1. **Check this guide** for common solutions
2. **Review serial logs** for error messages
3. **Test individual components** separately
4. **Document the issue** with steps to reproduce

### Information to Include
- **ESP32 board type** and specifications
- **Energy meter model** and version
- **ESPHome version** (`esphome version`) — must be ≥ 2026.9.0
- **Home Assistant version**
- **Relevant log excerpts**
- **Configuration file** (with secrets removed)

### Community Resources
- [ESPHome Discord](https://discord.gg/esphome)
- [Home Assistant Community Forum](https://community.home-assistant.io/)
- [GitHub Issues](https://github.com/esphome/esphome/issues)

### Useful Tools
- **ESP32 Serial WiFi Terminal** - For wireless serial monitoring
- **Network Analyzer** - For WiFi diagnostics
- **Modbus Poll** - For Modbus testing
- **I2C Scanner** - For I2C device detection

## 🔄 Prevention and Maintenance

### Regular Maintenance
- **Monitor logs** weekly for warning signs
- **Check connections** quarterly for corrosion
- **Update firmware** monthly for security patches
- **Backup configurations** after major changes

### System Monitoring
```yaml
# Add system health monitoring
sensor:
  - platform: uptime
    name: "Uptime"

  - platform: wifi_signal
    name: "WiFi Signal"
    update_interval: 60s

  - platform: template
    name: "Free Memory"
    lambda: !lambda
      return ESP.getFreeHeap();
```

### Performance Optimization
- **Update intervals**: Balance responsiveness vs. stability
- **Memory usage**: Monitor heap usage regularly
- **Connection quality**: Keep WiFi signal above -70 dBm

This troubleshooting guide should help you resolve most issues with the ESP32 Energy Meter project. For persistent problems, consider creating a GitHub issue with detailed information about your setup and the specific problem you're experiencing.
