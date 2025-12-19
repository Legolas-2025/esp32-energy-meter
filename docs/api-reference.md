# API Reference

Complete reference for all sensors, entities, and services in the ESP32 Energy Meter project.

## 📊 Sensor Entities

### Power Measurements

#### `sensor.esp32_energy_meter_power_1`
**Description**: Power consumption on Channel 1  
**Unit**: Watts (W)  
**Device Class**: power  
**Update Interval**: 3 seconds  

```yaml
- platform: modbus_controller
  modbus_controller_id: jsymk
  id: power1
  name: "Power 1"
  icon: mdi:lightning-bolt
  device_class: energy
  address: 0x004A
  unit_of_measurement: "W"
  register_type: holding
  value_type: U_DWORD
  accuracy_decimals: 1
  filters:
    - multiply: 0.0001
  register_count: 1
  response_size: 4
```

#### `sensor.esp32_energy_meter_power_2`
**Description**: Power consumption on Channel 2 (Primary)  
**Unit**: Watts (W)  
**Device Class**: power  
**Update Interval**: 3 seconds  
**Notes**: This is the primary power sensor used for display and monitoring

#### `sensor.esp32_energy_meter_power_factor_1`
**Description**: Power factor for Channel 1  
**Unit**: None (dimensionless)  
**Range**: 0.0 to 1.0  
**Update Interval**: 3 seconds  

#### `sensor.esp32_energy_meter_power_factor_2`
**Description**: Power factor for Channel 2  
**Unit**: None (dimensionless)  
**Range**: 0.0 to 1.0  
**Update Interval**: 3 seconds  

### Voltage Measurements

#### `sensor.esp32_energy_meter_voltage_1`
**Description**: Line voltage measurement for Channel 1  
**Unit**: Volts (V)  
**Device Class**: voltage  
**Typical Range**: 220-240V AC  
**Update Interval**: 3 seconds  

```yaml
- platform: modbus_controller
  modbus_controller_id: jsymk
  id: voltage1
  name: "Voltage"
  icon: mdi:alpha-v-box
  device_class: energy
  address: 0x0048
  unit_of_measurement: "V"
  register_type: holding
  value_type: U_DWORD
  accuracy_decimals: 1
  filters:
    - multiply: 0.0001
  register_count: 1
  response_size: 4
```

#### `sensor.esp32_energy_meter_voltage_2`
**Description**: Line voltage measurement for Channel 2  
**Unit**: Volts (V)  
**Device Class**: voltage  
**Typical Range**: 220-240V AC  
**Update Interval**: 3 seconds  

### Current Measurements

#### `sensor.esp32_energy_meter_current_1`
**Description**: Current flow measurement for Channel 1  
**Unit**: Amperes (A)  
**Device Class**: current  
**Accuracy Decimals**: 1  
**Update Interval**: 3 seconds  

#### `sensor.esp32_energy_meter_current_2`
**Description**: Current flow measurement for Channel 2  
**Unit**: Amperes (A)  
**Device Class**: current  
**Accuracy Decimals**: 4 (for low current readings)  
**Update Interval**: 3 seconds  

### Energy Measurements

#### `sensor.esp32_energy_meter_energy_1`
**Description**: Cumulative energy consumption for Channel 1  
**Unit**: Kilowatt-hours (kWh)  
**Device Class**: energy  
**State Class**: total  
**Update Interval**: 3 seconds  

#### `sensor.esp32_energy_meter_energy_2`
**Description**: Cumulative energy consumption for Channel 2  
**Unit**: Kilowatt-hours (kWh)  
**Device Class**: energy  
**State Class**: total  
**Update Interval**: 3 seconds  

### System Measurements

#### `sensor.esp32_energy_meter_frequency`
**Description**: AC line frequency  
**Unit**: Hertz (Hz)  
**Typical Range**: 49-51 Hz  
**Update Interval**: 3 seconds  

#### `sensor.esp32_energy_meter_wifi_signal_strength`
**Description**: WiFi signal strength in dBm  
**Unit**: dBm (decibels relative to milliwatt)  
**Range**: -100 to -30 dBm  
**Update Interval**: 10 seconds  
**Notes**: Values closer to -30 dBm indicate stronger signal

```yaml
- platform: wifi_signal
  name: "WiFi Signal Strength"
  id: wifi_signal_strength
  update_interval: 10s
  unit_of_measurement: "dBm"
```

### Advanced System Entities

#### `sensor.esp32_energy_meter_uptime`
**Description**: System uptime since last boot  
**Unit**: Seconds  
**Update Interval**: 30 seconds  

#### `sensor.esp32_energy_meter_free_heap`
**Description**: Available heap memory  
**Unit**: Bytes  
**Update Interval**: 60 seconds  
**Notes**: Monitor for memory leaks

## 📱 Binary Sensors

### Connection Status

#### `binary_sensor.esp32_energy_meter_wifi_connection_status`
**Description**: WiFi connectivity status  
**States**: 
- `on` (Connected) - Signal strength > -70 dBm
- `off` (Disconnected) - Signal strength ≤ -70 dBm

```yaml
- platform: template
  name: "WiFi Connection Status"
  id: connection_status
  lambda: !lambda
    return id(wifi_signal_strength).state > -70;
```

## 🎛️ Services

### Display Control Services

#### `esphome.display.update`
**Description**: Manually trigger display update  
**Parameters**: None  
**Usage**: 
```yaml
# Trigger manual display update
on_...:
  - display.update: oled
```

#### `esphome.display.clear`
**Description**: Clear the OLED display  
**Parameters**: None  
**Usage**:
```yaml
# Clear display
on_...:
  - display.clear: oled
```

### Modbus Communication Services

#### `esphome.modbus_controller.send`
**Description**: Send custom Modbus command  
**Parameters**:
- `address` (int): Modbus device address
- `command` (int): Command code
- `data` (list): Command data

### WiFi Services

#### `esphome.wifi.connect`
**Description**: Connect to WiFi network  
**Parameters**: None  
**Usage**: 
```yaml
# Force WiFi reconnection
on_...:
  - wifi.connect:
```

#### `esphome.wifi.disconnect`
**Description**: Disconnect from WiFi  
**Parameters**: None  

### System Services

#### `esphome.idle`
**Description**: Enter idle/low-power mode  
**Parameters**: None  

#### `esphome.restart`
**Description**: Restart ESP32  
**Parameters**: None  

#### `esphome.factory_reset`
**Description**: Factory reset to default settings  
**Parameters**: None  

## 🔧 Custom Entities

### Template Sensors

#### `sensor.esp32_energy_meter_power_quality_score`
**Description**: Calculated power quality score  
**Unit**: Percentage (0-100%)  
**Calculation**: Based on voltage, frequency, and power factor  
**Update Interval**: 30 seconds  

```yaml
- platform: template
  name: "Power Quality Score"
  id: power_quality_score
  unit_of_measurement: "%"
  lambda: !lambda
    // Complex calculation based on multiple factors
    return calculated_score;
```

#### `sensor.esp32_energy_meter_estimated_cost`
**Description**: Estimated energy cost  
**Unit**: Currency per day  
**Calculation**: Power consumption × electricity rate  
**Update Interval**: 60 seconds  

#### `sensor.esp32_energy_meter_daily_peak_power`
**Description**: Peak power consumption today  
**Unit**: Watts (W)  
**Reset**: Daily at midnight  
**Update Interval**: 60 seconds  

### Text Sensors

#### `text_sensor.esp32_energy_meter_wifi_quality`
**Description**: WiFi connection quality description  
**States**: 
- "Excellent" (-30 to -50 dBm)
- "Good" (-50 to -60 dBm)
- "Fair" (-60 to -70 dBm)
- "Poor" (-70 to -80 dBm)
- "Very Poor" (< -80 dBm)

#### `text_sensor.esp32_energy_meter_last_update`
**Description**: Timestamp of last successful sensor update  
**Format**: "YYYY-MM-DD HH:MM:SS"  

## 📋 Entity Categories

### Primary Sensors (Display)
These sensors are used for the OLED display:
- `sensor.esp32_energy_meter_power_2`
- `sensor.esp32_energy_meter_voltage_2`
- `sensor.esp32_energy_meter_current_2`

### System Monitoring
These sensors monitor system health:
- `sensor.esp32_energy_meter_wifi_signal_strength`
- `binary_sensor.esp32_energy_meter_wifi_connection_status`
- `sensor.esp32_energy_meter_uptime`

### Energy Analytics
These sensors provide enhanced data analysis:
- `sensor.esp32_energy_meter_energy_1`
- `sensor.esp32_energy_meter_energy_2`
- `sensor.esp32_energy_meter_power_factor_1`
- `sensor.esp32_energy_meter_power_factor_2`

### Historical Data
These sensors track historical trends:
- `sensor.esp32_energy_meter_daily_peak_power`
- `sensor.esp32_energy_meter_frequency`

## 🔌 Configuration Parameters

### Update Intervals

| Entity Type | Default Interval | Configurable |
|-------------|------------------|--------------|
| Power Sensors | 3 seconds | Yes |
| Voltage Sensors | 3 seconds | Yes |
| Current Sensors | 3 seconds | Yes |
| Energy Sensors | 3 seconds | Yes |
| Frequency | 3 seconds | Yes |
| WiFi Signal | 10 seconds | Yes |
| System Status | 30 seconds | Yes |

### Accuracy and Precision

| Measurement | Default Decimals | Range | Notes |
|-------------|------------------|-------|-------|
| Power | 1 | 0.1W | Sufficient for most applications |
| Voltage | 1 | 0.1V | Standard power line accuracy |
| Current (Ch1) | 1 | 0.1A | High current measurements |
| Current (Ch2) | 4 | 0.0001A | Low current precision |
| Energy | 1 | 0.1kWh | Cumulative accuracy |
| Power Factor | 3 | 0.001 | High precision needed |

### Filter Options

#### Built-in Filters
```yaml
filters:
  - multiply: 0.0001        # Scale factor
  - offset: -5.0            # Calibration offset
  - delta: 10.0             # Remove small changes
  - clamp:
      min: 0.0
      max: 10000.0          # Bound values
  - exponential_moving_average:
      alpha: 0.2            # Smoothing factor
  - heartbeat: 10s          # Periodic updates
  - throttle: 30s           # Limit update frequency
```

#### Custom Filters
```yaml
# Outlier removal
- outlier:
    radius: 50.0
    send_every: 4
    send_first_at: 2

# Calibration curves
- calibrate_linear:
    - 0.0 -> 0.0
    - 1000.0 -> 1025.0
    - 2000.0 -> 2040.0
```

## 📊 Home Assistant Integration

### Entity Mapping

| ESPHome Entity | Home Assistant Entity ID |
|----------------|-------------------------|
| Power 1 | `sensor.esp32_energy_meter_power_1` |
| Power 2 | `sensor.esp32_energy_meter_power_2` |
| Voltage 1 | `sensor.esp32_energy_meter_voltage_1` |
| Voltage 2 | `sensor.esp32_energy_meter_voltage_2` |
| Current 1 | `sensor.esp32_energy_meter_current_1` |
| Current 2 | `sensor.esp32_energy_meter_current_2` |
| Energy 1 | `sensor.esp32_energy_meter_energy_1` |
| Energy 2 | `sensor.esp32_energy_meter_energy_2` |
| WiFi Status | `binary_sensor.esp32_energy_meter_wifi_connection_status` |
| WiFi Signal | `sensor.esp32_energy_meter_wifi_signal_strength` |

### Energy Dashboard Configuration

```yaml
# Recommended energy dashboard entities
consumption:
  - sensor.esp32_energy_meter_energy_2  # Primary consumption
  - sensor.esp32_energy_meter_energy_1  # Secondary circuit

# Grid return (if applicable)
grid_return:
  - sensor.grid_energy  # Net metering
```

### Automation Examples

```yaml
# Power monitoring automation
automation:
  - id: high_power_alert
    trigger:
      - platform: numeric_state
        entity_id: sensor.esp32_energy_meter_power_2
        above: 3000
    action:
      - service: notify.mobile_app
        data:
          message: "High power usage: {{ states('sensor.esp32_energy_meter_power_2') }}W"
```

## 🔍 Monitoring and Diagnostics

### System Health Monitoring

```yaml
# Add to configuration for system monitoring
sensor:
  - platform: template
    name: "System Health"
    lambda: !lambda
      float heap = ESP.getFreeHeap();
      float signal = id(wifi_signal_strength).state;
      
      if (heap < 50000) return "Critical";
      if (heap < 100000) return "Low";
      if (signal < -80) return "Poor Signal";
      if (signal < -70) return "Fair Signal";
      return "Healthy";
```

### Performance Metrics

| Metric | Normal Range | Warning | Critical |
|--------|-------------|---------|----------|
| Free Heap | >150KB | 50-150KB | <50KB |
| WiFi Signal | >-60 dBm | -60 to -80 dBm | <-80 dBm |
| Power Factor | >0.9 | 0.7-0.9 | <0.7 |
| Voltage | 220-240V | 210-250V | <210V or >250V |
| Frequency | 49-51Hz | 48-52Hz | <48Hz or >52Hz |

## 🔧 Advanced Configuration

### Custom Entity Creation

```yaml
# Create custom entity with specific characteristics
sensor:
  - platform: modbus_controller
    modbus_controller_id: jsymk
    id: custom_power
    name: "Custom Power Reading"
    address: 0x0052
    unit_of_measurement: "W"
    register_type: holding
    value_type: U_DWORD
    accuracy_decimals: 2
    filters:
      - multiply: 0.0001
      - offset: -10.0
      - clamp:
          min: 0.0
          max: 50000.0
    on_value:
      - if:
          condition:
            lambda: return id(custom_power).state > 1000.0;
          then:
            - display.update: oled  # Force display update
```

### Entity Dependencies

```yaml
# Define entity dependencies for proper update order
sensor:
  - platform: template
    id: derived_calculation
    name: "Derived Value"
    lambda: !lambda
      // Use values from other sensors
      float power = id(power2).state;
      float voltage = id(voltage2).state;
      return voltage > 0 ? power / voltage : 0;
```

This API reference provides complete documentation for all entities and services in the ESP32 Energy Meter project. Use this as a guide for integration, automation, and customization.
