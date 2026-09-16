# Configuration Guide

This guide walks you through configuring ESPHome for your ESP32 Energy Meter, from initial setup to advanced customization.

> **ESPHome 2026.9.0+ required.** This guide reflects the new Modbus sizing model introduced in ESPHome 2026.9.0. If you are upgrading an older configuration, see the [ESPHome 2026.9.0 Migration](#esphome-202690-migration) section below.

## 🚀 Quick Start

### Prerequisites
- ESP32 development board with energy meter hardware connected
- Computer with **ESPHome 2026.9.0 or newer** installed
- WiFi network credentials
- Home Assistant instance (optional but recommended)

### Installation Steps
1. [Install ESPHome](#installing-esphome)
2. [Configure Secrets](#configuring-secrets)
3. [Upload Configuration](#uploading-configuration)
4. [Integrate with Home Assistant](#home-assistant-integration)

## 🔧 Installing ESPHome

### Method 1: pip (Recommended)
```bash
pip install --upgrade esphome
esphome version   # confirm 2026.9.0 or newer
```

### Method 2: Docker
```bash
docker run -it --rm \
  -v "$PWD":/config \
  esphome/esphome run esp32-energy-meter.yaml
```

### Method 3: Home Assistant Add-on
1. Open Home Assistant
2. Go to Supervisor → Add-ons
3. Search for "ESPHome"
4. Click "Install"

> The HA ESPHome add-on bundles its own ESPHome version. Make sure the add-on is updated to a release that ships ESPHome **≥ 2026.9.0** before validating this configuration.

## 🔐 Configuring Secrets

Create a `secrets.yaml` file in your project directory:
```yaml
# WiFi Configuration
wifi_ssid: "YOUR_WIFI_NETWORK_NAME"
wifi_password: "YOUR_WIFI_PASSWORD"

# API Encryption Key (optional - will be auto-generated)
api_key: "GENERATED_API_KEY"

# OTA Password
ota_password: "YOUR_OTA_PASSWORD"

# MQTT Configuration (if using MQTT)
mqtt_broker_ip: "192.168.1.100"
mqtt_broker_port: 1883
mqtt_username: "your_mqtt_username"
mqtt_password: "your_mqtt_password"
```

### Security Best Practices
- Use strong passwords (at least 12 characters)
- Enable API encryption
- Use WPA2/WPA3 WiFi encryption
- Change default passwords immediately

## ⚙️ Basic Configuration

### File Structure
```
project/
├── esp32-energy-meter.yaml
├── secrets.yaml
└── .esphome/
```

### Core Configuration Sections

#### 1. Basic Device Settings
```yaml
esphome:
  name: esp32-energy-meter
  friendly_name: ESP32 Energy Meter

esp32:
  board: esp32dev
  framework:
    type: esp-idf
```

#### 2. Network Configuration
```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  manual_ip:
    static_ip: 192.168.1.100  # Choose available IP
    gateway: 192.168.1.1     # Your router IP
    subnet: 255.255.255.0
  ap:
    ssid: "ESP32-Energy-Meter Fallback Hotspot"
    password: "CHANGE_THIS_PASSWORD"
```

#### 3. API and OTA
```yaml
api:
  encryption:
    key: !secret api_key

ota:
  - platform: esphome
    password: !secret ota_password
```

## 📡 Hardware Configuration

### Modbus RTU Setup
```yaml
uart:
  id: mod_bus
  tx_pin: 17
  rx_pin: 16
  baud_rate: 4800
  stop_bits: 1

modbus:
  id: modbus1
  # `turnaround_time` (on the `modbus:` hub) replaced the deprecated
  # `command_throttle` on `modbus_controller` in ESPHome 2026.9.0.
  turnaround_time: 50ms
  modbus_controller:
    - id: jsymk
      address: 0x1               # JSY meter address
      modbus_id: modbus1
      update_interval: 3s        # Measurement update interval
      # NOTE: do NOT set `command_throttle` here — it is ignored and will
      #       emit a deprecation warning. Use `turnaround_time` above.
```

### I2C Display Setup
```yaml
i2c:
  sda: 21
  scl: 22

font:
  - file:
      type: gfonts
      family: Baloo+Bhaijaan+2
      weight: 500
    id: baloo_18_500
    size: 18
```

## 📱 Display Configuration

### OLED Display Lambda
```yaml
display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x64"
    address: 0x3C
    id: oled
    rotation: 180°
    update_interval: 3s
    lambda: !lambda |-
      // Burn-in protection
      time_t now = id(homeassistant_time).now().timestamp;
      if (now % 1800 == 0) {
        it.clear();
        return;
      }
      // Clear and draw content
      it.clear();
      it.print(-1, -4, id(baloo_18_500), "Energy Meter");
      // WiFi status
      if(id(connection_status).state == 1) {
        it.print(it.get_width(), 0, id(icons_18), TextAlign::TOP_RIGHT, "\ue63e");
      } else {
        it.print(it.get_width(), 0, id(icons_18), TextAlign::TOP_RIGHT, "\ue648");
      }
      // Power display
      it.printf(it.get_width()/2, 38, id(baloo_32_700), TextAlign::CENTER, "%.1f W", id(power2).state);
      it.printf(0, it.get_height() + 12, id(baloo_18_700), TextAlign::BOTTOM_LEFT, "%.1f V", id(voltage2).state);
      it.printf(it.get_width(), it.get_height() + 12, id(baloo_18_700), TextAlign::BOTTOM_RIGHT, "%.1f A", id(current2).state);
```

## 📊 Sensor Configuration

### WiFi Signal Strength Monitoring
```yaml
binary_sensor:
  - platform: template
    name: "WiFi Connection Status"
    id: connection_status
    lambda: !lambda
      return id(wifi_signal_strength).state > -70;

sensor:
  - platform: wifi_signal
    name: "WiFi Signal Strength"
    id: wifi_signal_strength
    update_interval: 10s
    unit_of_measurement: "dBm"
```

### Energy Meter Sensors

> **ESPHome ≥ 2026.9.0:** register count is derived from `value_type` (e.g.
> `U_DWORD` ⇒ 4 bytes / 2 registers). The `register_count` and `response_size`
> keys that used to appear here have been **removed** — leave them out.

```yaml
sensor:
  # Power measurement (Channel 2 - primary)
  - platform: modbus_controller
    modbus_controller_id: jsymk
    id: power2
    name: "Power 2"
    icon: mdi:lightning-bolt
    device_class: energy
    address: 0x0052
    unit_of_measurement: "W"
    register_type: holding
    value_type: U_DWORD        # → automatically reads 2 registers / 4 bytes
    accuracy_decimals: 1
    filters:
      - multiply: 0.0001

  # Voltage measurement
  - platform: modbus_controller
    modbus_controller_id: jsymk
    id: voltage2
    name: "Voltage 2"
    icon: mdi:alpha-v-box
    device_class: energy
    address: 0x0050
    unit_of_measurement: "V"
    register_type: holding
    value_type: U_DWORD
    accuracy_decimals: 1
    filters:
      - multiply: 0.0001

  # Current measurement
  - platform: modbus_controller
    modbus_controller_id: jsymk
    id: current2
    name: "Current 2"
    icon: mdi:current-ac
    device_class: energy
    address: 0x0051
    unit_of_measurement: "A"
    register_type: holding
    value_type: U_DWORD
    accuracy_decimals: 4            # Increased for low current readings
    filters:
      - multiply: 0.0001
```

## 🚀 Uploading Configuration

### First Upload
```bash
# Connect ESP32 via USB and upload
esphome run esp32-energy-meter.yaml
```

### Subsequent Updates
```bash
# Compile and upload (no USB connection required)
esphome run esp32-energy-meter.yaml --upload-port 192.168.1.100
```

### Advanced Options
```bash
# Enable verbose logging
esphome run esp32-energy-meter.yaml --log-level=debug

# Clean build
esphome run esp32-energy-meter.yaml --clean

# Upload via OTA (if IP is known)
esphome run esp32-energy-meter.yaml --upload-port 192.168.1.100
```

## 🏠 Home Assistant Integration

### Auto-Discovery
Once uploaded, the ESP32 will automatically appear in Home Assistant under "Devices & Services".

### Manual Integration
1. Go to Configuration → Devices & Services
2. Click "Add Integration"
3. Search for "ESPHome"
4. Enter the IP address: `192.168.1.100`
5. Enter API key when prompted

### Entity Naming
ESPHome will create entities like:
- `sensor.esp32_energy_meter_power_2`
- `sensor.esp32_energy_meter_voltage_2`
- `sensor.esp32_energy_meter_current_2`
- `binary_sensor.esp32_energy_meter_wifi_connection_status`

## 🔧 Advanced Configuration

### Custom Update Intervals
```yaml
modbus:
  id: modbus1
  turnaround_time: 25ms           # Reduced pause between commands

  modbus_controller:
    - id: jsymk
      address: 0x1
      modbus_id: modbus1
      update_interval: 2s         # Faster updates
```

### Display Customization
```yaml
display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x64"
    address: 0x3C
    id: oled
    rotation: 0°                  # Normal orientation
    update_interval: 1s           # Faster display updates
    lambda: !lambda |-
      // Custom display logic here
      it.clear();
      it.printf(0, 0, id(baloo_18_500), "Custom Label");
```

### Sensor Filtering
```yaml
sensor:
  - platform: modbus_controller
    # ... other config ...
    filters:
      - multiply: 0.0001           # Scale factor
      - offset: -5.0               # Calibration offset
      - exponential_moving_average:
          alpha: 0.2               # Smooth readings
      - heartbeat: 10s             # Periodic updates
```

## 📊 Performance Optimization

### Memory Management
```yaml
esp32:
  board: esp32dev
  framework:
    type: esp-idf
  psram:
    mode: octal                    # Enable PSRAM if available
```

### WiFi Optimization
```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: true               # Skip scanning
  output_power: 10.5               # Adjust power level
```

### Update Optimization
```yaml
modbus:
  id: modbus1
  turnaround_time: 100ms           # Prevent bus overload

  modbus_controller:
    - id: jsymk
      address: 0x1
      modbus_id: modbus1
      update_interval: 5s          # Balance responsiveness vs. stability
```

## 🆕 ESPHome 2026.9.0 Migration

This project was updated to be compatible with **ESPHome 2026.9.0**, which changed how `modbus_controller` sensors are sized and paced.

### What changed

| Setting | Old behaviour (≤ 2026.x) | New behaviour (≥ 2026.9.0) |
|---|---|---|
| Number of registers read | Explicit `register_count: N` per sensor | **Derived from `value_type`** (e.g. `U_DWORD` ⇒ 2 registers). `register_count` has been removed. |
| Per-sensor byte size | `response_size: N` on every sensor | Only valid for `RAW` values and text sensors. For typed sensors (`U_DWORD`, `S_WORD`, …) the size is implicit in the type. |
| Inter-command spacing | `command_throttle: 50ms` on `modbus_controller` | Moved to `turnaround_time: 50ms` on the `modbus:` hub. The old key still parses but **has no effect** and will be removed in 2027.2.0. |

### Before / after — typical sensor

```yaml
# Before (ESPHome < 2026.9.0)
- platform: modbus_controller
  modbus_controller_id: jsymk
  id: power2
  address: 0x0052
  register_type: holding
  value_type: U_DWORD
  filters:
    - multiply: 0.0001
  register_count: 1        # ← removed
  response_size: 4         # ← removed for non-RAW / non-text sensors
```

```yaml
# After (ESPHome ≥ 2026.9.0)
- platform: modbus_controller
  modbus_controller_id: jsymk
  id: power2
  address: 0x0052
  register_type: holding
  value_type: U_DWORD
  filters:
    - multiply: 0.0001
  # `value_type: U_DWORD` ⇒ 2 registers / 4 bytes automatically.
```

### Before / after — command pacing

```yaml
# Before (ESPHome < 2026.9.0)
modbus:
  id: modbus1
  modbus_controller:
    - id: jsymk
      command_throttle: 50ms
```

```yaml
# After (ESPHome ≥ 2026.9.0)
modbus:
  id: modbus1
  turnaround_time: 50ms                 # pacing moves to the hub
  modbus_controller:
    - id: jsymk
      # command_throttle removed
```

### Special cases

- **Reading more registers than `value_type` implies** (e.g. you want one
  request that covers two adjacent sensors): add `reuse_previous_range: true`
  on the *next* sensor instead of bumping `register_count`.
- **RAW or text block reads**: keep `response_size` and set it to the byte
  count of the payload.
- **Multi-register writes**: set `use_write_multiple: true` on the write
  sensor.

### Upgrading an existing install

1. Upgrade ESPHome: `pip install --upgrade esphome` (and update the Home
   Assistant ESPHome add-on, if you use it).
2. Pull the updated `esp32-energy-meter.yaml` from this repository.
3. Validate before flashing:
   ```bash
   esphome config esp32-energy-meter.yaml
   ```
4. Flash:
   ```bash
   esphome run esp32-energy-meter.yaml
   ```
   Wiring, secrets, and Home Assistant entities are unchanged.

See the [ESPHome Modbus controller docs](https://esphome.io/components/modbus_controller/) for the full reference.

## 🐛 Troubleshooting

### Common Issues

#### Compilation Errors
- **Check YAML syntax**: Use online YAML validators
- **Verify sensor IDs**: Ensure all referenced IDs exist
- **Update ESPHome**: `pip install --upgrade esphome` (must be ≥ 2026.9.0)

#### `register_count has been removed` validation error
See the dedicated entry in [`troubleshooting.md`](troubleshooting.md#validation-error-register_count-has-been-removed).

#### Connection Issues
- **Check IP conflicts**: Use static IP to avoid DHCP issues
- **Verify WiFi credentials**: Ensure correct SSID and password
- **Check firewall**: Ensure device can connect to local network

#### Sensor Reading Issues
- **Check Modbus address**: Default is usually `0x1`
- **Verify wiring**: Check RS485 A+ and B- connections
- **Check update intervals**: Don't set too aggressive

### Debug Commands
```bash
# Check device status
esphome config esp32-energy-meter.yaml

# Test connection
esphome logs esp32-energy-meter.yaml --device 192.168.1.100

# Check logs
esphome logs esp32-energy-meter.yaml --serial /dev/ttyUSB0
```

## 📝 Configuration Templates

### Minimal Configuration
```yaml
esphome:
  name: minimal-energy-meter
  friendly_name: Minimal Energy Meter

esp32:
  board: esp32dev

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

api:
  encryption:
    key: !secret api_key

ota:
  - platform: esphome
    password: !secret ota_password

uart:
  id: mod_bus
  tx_pin: 17
  rx_pin: 16
  baud_rate: 4800

modbus:
  id: modbus1
  turnaround_time: 50ms
  modbus_controller:
    - id: jsymk
      address: 0x1
      modbus_id: modbus1

sensor:
  - platform: modbus_controller
    modbus_controller_id: jsymk
    id: power
    name: "Power"
    address: 0x0052
    unit_of_measurement: "W"
    register_type: holding
    value_type: U_DWORD
    filters:
      - multiply: 0.0001
```

### Production Configuration
```yaml
# Full featured configuration with all sensors, display, and monitoring
# (see esp32-energy-meter.yaml for complete example)
```

## 🔗 Next Steps

After basic configuration:
1. [Hardware Setup](hardware-setup.md) - If not already completed
2. [Home Assistant Integration](Home-Assistant-Integration.md)
3. [Advanced Features](Advanced-Features.md)
4. [Troubleshooting](troubleshooting.md)

For specific hardware issues, refer to the [Hardware Setup Guide](hardware-setup.md).
