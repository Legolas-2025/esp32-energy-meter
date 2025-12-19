# Home Assistant Integration Guide

This guide covers how to integrate your ESP32 Energy Meter with Home Assistant for comprehensive energy monitoring and automation.

## 🎯 Overview

Home Assistant provides a powerful platform for energy monitoring, allowing you to:
- Create detailed energy dashboards
- Set up energy usage alerts
- Integrate with utility companies
- Automate based on consumption
- Analyze trends and patterns

## 🔗 Integration Methods

### Method 1: Auto-Discovery (Recommended)
1. **Power on ESP32**: Device will broadcast presence on network
2. **Home Assistant Discovery**: Device appears under "Devices & Services"
3. **Add Integration**: Click "Configure" to add ESP32 Energy Meter
4. **Enter API Key**: Provide the API key when prompted

### Method 2: Manual Integration
1. Go to **Configuration** → **Devices & Services**
2. Click **Add Integration**
3. Search for **ESPHome**
4. Enter IP address: `192.168.1.XXX`
5. Enter API key

## 📊 Available Entities

### Power Monitoring Entities
```yaml
# Primary power sensors
sensor.esp32_energy_meter_power_1     # Channel 1 Power (W)
sensor.esp32_energy_meter_power_2     # Channel 2 Power (W)
sensor.esp32_energy_meter_voltage_1   # Channel 1 Voltage (V)
sensor.esp32_energy_meter_voltage_2   # Channel 2 Voltage (V)
sensor.esp32_energy_meter_current_1   # Channel 1 Current (A)
sensor.esp32_energy_meter_current_2   # Channel 2 Current (A)
sensor.esp32_energy_meter_energy_1    # Channel 1 Energy (kWh)
sensor.esp32_energy_meter_energy_2    # Channel 2 Energy (kWh)
```

### System Monitoring Entities
```yaml
# System status
sensor.esp32_energy_meter_wifi_signal_strength  # WiFi Signal (dBm)
binary_sensor.esp32_energy_meter_wifi_connection_status  # Connection Status
sensor.esp32_energy_meter_frequency              # AC Frequency (Hz)
sensor.esp32_energy_meter_power_factor_1         # Channel 1 Power Factor
sensor.esp32_energy_meter_power_factor_2         # Channel 2 Power Factor
```

## 🏠 Energy Dashboard Setup

### Step 1: Configure Energy Sources
1. Go to **Configuration** → **Energy**
2. Click **Add Consumption** under "Electricity Grid"
3. Select appropriate energy sensor
4. Set energy source details

### Step 2: Add Consumption Sources
```yaml
# Recommended configuration for energy dashboard
consumption:
  - sensor.esp32_energy_meter_energy_2     # Main consumption
  - sensor.esp32_energy_meter_energy_1     # Secondary circuit (optional)
```

### Step 3: Configure Sensors for Dashboard
```yaml
# Template sensors for better dashboard integration
template:
  sensors:
    # Daily energy consumption
    daily_energy_consumption:
      friendly_name: "Daily Energy Consumption"
      unit_of_measurement: "kWh"
      value_template: >-
        {% set today = states('sensor.date') %}
        {{ states('sensor.esp32_energy_meter_energy_2') | float(0) }}
      icon_template: mdi:flash
    
    # Real-time power usage
    current_power_usage:
      friendly_name: "Current Power Usage"
      unit_of_measurement: "W"
      value_template: "{{ states('sensor.esp32_energy_meter_power_2') | float(0) }}"
      icon_template: mdi:lightning-bolt
    
    # Power cost estimation
    estimated_power_cost:
      friendly_name: "Estimated Power Cost"
      unit_of_measurement: "$/day"
      value_template: >-
        {{ (states('sensor.esp32_energy_meter_power_2') | float(0) * 0.12 / 1000) | round(2) }}
      icon_template: mdi:currency-usd
```

## 📈 Creating Energy Cards

### Power Usage Card
```yaml
type: entities
entities:
  - entity: sensor.current_power_usage
    name: Current Power
    icon: mdi:lightning-bolt
  - entity: sensor.esp32_energy_meter_power_1
    name: Circuit 1 Power
  - entity: sensor.esp32_energy_meter_power_2
    name: Circuit 2 Power
show_header_toggle: false
title: Real-time Power Usage
```

### Energy Statistics Card
```yaml
type: statistics-graph
entities:
  - sensor.esp32_energy_meter_power_2
period: 24h
stat_types:
  - change
  - max
  - mean
title: 24 Hour Power Statistics
```

### Energy Gauge Card
```yaml
type: gauge
entity: sensor.current_power_usage
min: 0
max: 5000
severity:
  green: 0
  yellow: 2000
  red: 4000
unit: W
title: Current Power Usage
```

## 🔔 Energy Alerts and Automations

### High Power Usage Alert
```yaml
automation:
  - id: high_power_usage_alert
    alias: "High Power Usage Alert"
    description: "Alert when power usage exceeds threshold"
    trigger:
      - platform: numeric_state
        entity_id: sensor.esp32_energy_meter_power_2
        above: 3000  # 3000W threshold
    condition:
      - condition: time
        after: "08:00:00"
        before: "22:00:00"
    action:
      - service: notify.persistent_notification
        data:
          title: "High Power Usage Alert"
          message: "Current power usage is {{ states('sensor.esp32_energy_meter_power_2') }}W"
```

### Energy Cost Tracking
```yaml
automation:
  - id: daily_energy_report
    alias: "Daily Energy Report"
    description: "Send daily energy consumption report"
    trigger:
      - platform: time
        at: "23:59:00"
    condition:
      - condition: time
        weekday:
          - mon
          - wed
          - fri
    action:
      - service: notify.telegram
        data:
          message: >-
            Daily Energy Report:
            Power: {{ states('sensor.esp32_energy_meter_power_2') }}W
            Energy: {{ states('sensor.esp32_energy_meter_energy_2') }}kWh
```

### Power Quality Monitoring
```yaml
sensor:
  - platform: template
    sensors:
      power_quality_status:
        friendly_name: "Power Quality"
        value_template: >-
          {% set voltage = states('sensor.esp32_energy_meter_voltage_2') | float(0) %}
          {% set frequency = states('sensor.esp32_energy_meter_frequency') | float(0) %}
          {% set pf = states('sensor.esp32_energy_meter_power_factor_2') | float(1) %}
          
          {% if 220 <= voltage <= 240 and 49 <= frequency <= 51 and pf >= 0.9 %}
            Good
          {% elif 210 <= voltage <= 250 and 48 <= frequency <= 52 %}
            Fair
          {% else %}
            Poor
          {% endif %}
        icon_template: >-
          {% set status = states('sensor.power_quality_status') %}
          {% if status == 'Good' %}
            mdi:check-circle
          {% elif status == 'Fair' %}
            mdi:alert-circle
          {% else %}
            mdi:alert-octagon
          {% endif %}
```

## 🏠 Home Automation Examples

### Smart Appliance Control
```yaml
automation:
  - id: turn_off_high_consumption_device
    alias: "Auto Turn Off High Consumption Device"
    description: "Turn off appliances when power usage is too high"
    trigger:
      - platform: numeric_state
        entity_id: sensor.esp32_energy_meter_power_2
        above: 4000
    condition:
      - condition: state
        entity_id: switch.large_appliance
        state: "on"
    action:
      - service: switch.turn_off
        target:
          entity_id: switch.large_appliance
      - service: notify.mobile_app_phone
        data:
          message: "Auto turned off large appliance due to high power usage"
```

### Energy Saving Mode
```yaml
automation:
  - id: energy_saving_mode
    alias: "Energy Saving Mode"
    description: "Activate energy saving mode during peak hours"
    trigger:
      - platform: time
        at: "16:00:00"
    condition:
      - condition: time
        weekday:
          - mon
          - tue
          - wed
          - thu
          - fri
    action:
      - service: climate.set_temperature
        target:
          entity_id: climate.main_thermostat
        data:
          temperature: 20
      - service: light.turn_off
        target:
          entity_id: group.living_room_lights
```

## 📊 Advanced Dashboard Configuration

### Custom Energy Dashboard
```yaml
views:
  - title: Energy Dashboard
    path: energy_dashboard
    icon: mdi:flash
    cards:
      - type: energy-distribution
        title: Energy Distribution
        link_entity: sensor.esp32_energy_meter_energy_2
        link_type: consumption
        source_type: grid
        chart_type: pie
      
      - type: energy-usage-graph
        title: Energy Usage
        period: 24h
        start: "{{ now() - timedelta(days=1) }}"
        end: "{{ now() }}"
      
      - type: thermostat
        entity: climate.main_thermostat
        theme: default
      
      - type: grid
        columns: 2
        cards:
          - type: gauge
            entity: sensor.current_power_usage
            title: Current Power
            unit: W
            severity:
              green: 0
              yellow: 2000
              red: 4000
          
          - type: statistic
            name: Average Power
            entity: sensor.esp32_energy_meter_power_2
            period: day
            stat_type: mean
          
          - type: sensor
            entity: sensor.esp32_energy_meter_voltage_2
            name: Voltage
          
          - type: sensor
            entity: sensor.esp32_energy_meter_frequency
            name: Frequency
```

## 🔧 Configuration Tips

### Sensor Naming Conventions
```yaml
# Use descriptive names for better organization
sensor:
  - platform: modbus_controller
    name: "Kitchen Power"
    id: power_kitchen
  - platform: modbus_controller
    name: "Living Room Power"
    id: power_living_room
```

### Custom Device Classes
```yaml
# Use appropriate device classes for better integration
sensor:
  - platform: modbus_controller
    device_class: power          # For power measurements
    address: 0x0052
    unit_of_measurement: "W"
  
  - platform: modbus_controller
    device_class: voltage        # For voltage measurements
    address: 0x0050
    unit_of_measurement: "V"
  
  - platform: modbus_controller
    device_class: current        # For current measurements
    address: 0x0051
    unit_of_measurement: "A"
```

### State Classes for Energy
```yaml
# Proper state classes for energy measurements
sensor:
  - platform: modbus_controller
    address: 0x004B              # Energy register
    unit_of_measurement: "kWh"
    state_class: total           # For cumulative energy
    device_class: energy
```

## 📱 Mobile App Integration

### Mobile Card Configuration
```yaml
# Optimized for mobile viewing
cards:
  - type: entities
    title: Energy Overview
    entities:
      - sensor.current_power_usage
      - sensor.esp32_energy_meter_energy_2
      - sensor.esp32_energy_meter_voltage_2
      - binary_sensor.esp32_energy_meter_wifi_connection_status
    show_header_toggle: false
    
  - type: gauge
    title: Power Usage
    entity: sensor.current_power_usage
    min: 0
    max: 5000
    unit: W
```

## 🔄 Data Retention and History

### Recorder Configuration
```yaml
# In configuration.yaml
recorder:
  purge_keep_days: 365          # Keep 1 year of data
  commit_interval: 5            # Commit every 5 seconds
  
  exclude:
    entities:
      - binary_sensor.esp32_energy_meter_wifi_connection_status
```

### Statistics Configuration
```yaml
# Enable statistics for energy calculations
statistics:
  sensors:
    - entity_id: sensor.esp32_energy_meter_energy_2
      type: state
      name: "Energy Statistics"
```

## 🎛️ Lovelace UI Customization

### Energy Summary Card
```yaml
type: energy-summary
consumption:
  - sensor.esp32_energy_meter_energy_2
grid_return:
  - sensor.grid_energy
solar:
  - sensor.solar_energy
title: Energy Summary
```

### Power Flow Card
```yaml
type: energy-distribution
title: Power Flow
link_entity: sensor.esp32_energy_meter_power_2
link_type: consumption
source_type: grid
chart_type: pie
```

## 🔍 Troubleshooting Integration

### Common Issues

#### Entities Not Appearing
- **Check API key**: Ensure correct API key in Home Assistant
- **Verify network**: Confirm ESP32 and HA are on same network
- **Restart integration**: Try removing and re-adding ESP32 integration

#### Missing Energy Data
- **Check state class**: Ensure energy sensors have `state_class: total`
- **Verify device class**: Use `device_class: energy` for energy measurements
- **Check recorder**: Ensure statistics are being recorded

#### Dashboard Not Loading
- **Enable statistics**: Statistics must be enabled for energy dashboard
- **Check configuration**: Verify energy sources are properly configured
- **Restart Home Assistant**: Sometimes needed after major changes

### Debug Commands
```bash
# Check ESPHome device status
esphome logs esp32-energy-meter.yaml --device 192.168.1.100

# Monitor ESP32 serial output
esphome logs esp32-energy-meter.yaml --serial /dev/ttyUSB0

# Check Home Assistant logs
tail -f /config/home-assistant.log
```

## 📚 Additional Resources

- [Home Assistant Energy Documentation](https://www.home-assistant.io/docs/energy/)
- [ESPHome Documentation](https://esphome.io/)
- [JSY Energy Meter Manual](link-to-manual)

For advanced features and troubleshooting, see [Advanced Features](../wiki/Advanced-Features.md) and [Troubleshooting](../docs/troubleshooting.md).