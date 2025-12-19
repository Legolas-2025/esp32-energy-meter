# Configuration Guide

This guide walks you through configuring ESPHome for your ESP32 Energy Meter, from initial setup to advanced customization.

## 🚀 Quick Start

### Prerequisites
- ESP32 development board with energy meter hardware connected
- Computer with ESPHome installed
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
pip install esphome
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
    gateway: 192.168.1.1      # Your router IP
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

modbus_controller:
  - id: jsymk
    address: 0x1              # JSY meter address
    modbus_id: modbus1
    update_interval: 3s       # Measurement update interval
    command_throttle: 50ms    # Min time between commands
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
    value_type: U_DWORD
    accuracy_decimals: 1
    filters:
      - multiply: 0.0001
    register_count: 1
    response_size: 4

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
    register_count: 1
    response_size: 4

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
    accuracy_decimals: 4  # Increased for low current readings
    filters:
      - multiply: 0.0001
    register_count: 1
    response_size: 4
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
modbus_controller:
  - id: jsymk
    address: 0x1
    modbus_id: modbus1
    update_interval: 2s       # Faster updates
    command_throttle: 25ms    # Reduced throttle
```

### Display Customization
```yaml
display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x64"
    address: 0x3C
    id: oled
    rotation: 0°              # Normal orientation
    update_interval: 1s       # Faster display updates
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
      - multiply: 0.0001      # Scale factor
      - offset: -5.0          # Calibration offset
      - exponential_moving_average:
          alpha: 0.2          # Smooth readings
      - heartbeat: 10s        # Periodic updates
```

## 📊 Performance Optimization

### Memory Management
```yaml
esp32:
  board: esp32dev
  framework:
    type: esp-idf
  psram:
    mode: octal              # Enable PSRAM if available
```

### WiFi Optimization
```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: true         # Skip scanning
  output_power: 10.5         # Adjust power level
```

### Update Optimization
```yaml
modbus_controller:
  - id: jsymk
    address: 0x1
    modbus_id: modbus1
    update_interval: 5s      # Balance responsiveness vs. stability
    command_throttle: 100ms  # Prevent bus overload
```

## 🐛 Troubleshooting

### Common Issues

#### Compilation Errors
- **Check YAML syntax**: Use online YAML validators
- **Verify sensor IDs**: Ensure all referenced IDs exist
- **Update ESPHome**: `pip install --upgrade esphome`

#### Connection Issues
- **Check IP conflicts**: Use static IP to avoid DHCP issues
- **Verify WiFi credentials**: Ensure correct SSID and password
- **Check firewall**: Ensure device can connect to local network

#### Sensor Reading Issues
- **Check Modbus address**: Default is usually 0x1
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
2. [Home Assistant Integration](wiki/Home-Assistant-Integration.md)
3. [Advanced Features](wiki/Advanced-Features.md)
4. [Troubleshooting](troubleshooting.md)

For specific hardware issues, refer to the [Hardware Setup Guide](hardware-setup.md).