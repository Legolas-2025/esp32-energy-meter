# Advanced Features Guide

This guide covers advanced features and customization options for your ESP32 Energy Meter project.

## 🚀 Overview

The ESP32 Energy Meter project includes several advanced features that can be enabled and customized based on your specific requirements:

- **OLED Burn-in Protection**: Prevents screen damage from static content
- **Enhanced WiFi Monitoring**: Accurate connectivity status monitoring
- **Custom Display Layouts**: Flexible display configuration
- **Advanced Sensor Filtering**: Noise reduction and smoothing
- **MQTT Integration**: Alternative to Home Assistant API
- **Remote Configuration**: Dynamic parameter adjustment

## 🔄 OLED Burn-in Protection

### Current Implementation
The project includes automatic screen clearing to prevent OLED burn-in:

```yaml
lambda: !lambda |-
  // Time integration for burn-in protection
  time_t now = id(homeassistant_time).now().timestamp;
  
  // Clear screen every 30 minutes to prevent OLED burn-in
  if (now % 1800 == 0) {  // Every 30 minutes
    it.clear();
    return;  // Exit early to prevent drawing during clear
  }
  
  // Rest of display logic...
```

### Customizable Parameters
```yaml
# Modify burn-in protection interval
lambda: !lambda |-
  time_t now = id(homeassistant_time).now().timestamp;
  
  // Custom interval: every 10 minutes (600 seconds)
  if (now % 600 == 0) {
    it.clear();
    return;
  }
  
  // Alternative: Screen saver mode after 60 minutes
  static bool screen_saver = false;
  if (now > 3600 && !screen_saver) {
    screen_saver = true;
    // Display logo or blank screen
  }
```

### Advanced Burn-in Protection
```yaml
# Enhanced burn-in protection with rotation
lambda: !lambda |-
  time_t now = id(homeassistant_time).now().timestamp;
  
  // Clear screen every 15 minutes
  if (now % 900 == 0) {
    it.clear();
    return;
  }
  
  // Rotate display content every 30 minutes to distribute wear
  static int rotation_counter = 0;
  if (now % 1800 == 0) {
    rotation_counter++;
    it.clear();
  }
  
  // Draw based on rotation
  switch (rotation_counter % 3) {
    case 0:
      // Standard layout
      it.printf(it.get_width()/2, 38, id(baloo_32_700), TextAlign::CENTER, "%.1f W", id(power2).state);
      break;
    case 1:
      // Compact layout
      it.printf(it.get_width()/2, 20, id(baloo_18_700), TextAlign::CENTER, "Power: %.1fW", id(power2).state);
      it.printf(it.get_width()/2, 40, id(baloo_18_700), TextAlign::CENTER, "Voltage: %.1fV", id(voltage2).state);
      break;
    case 2:
      // Detailed layout with additional info
      it.printf(it.get_width()/2, 20, id(baloo_18_700), TextAlign::CENTER, "P:%.1fW V:%.1fV", id(power2).state, id(voltage2).state);
      it.printf(it.get_width()/2, 40, id(baloo_18_700), TextAlign::CENTER, "I:%.4fA", id(current2).state);
      break;
  }
```

## 📶 Enhanced WiFi Monitoring

### Current WiFi Status Logic
```yaml
binary_sensor:
  - platform: template
    name: "WiFi Connection Status"
    id: connection_status
    lambda: !lambda
      return id(wifi_signal_strength).state > -70;  // Connected if signal strength > -70 dBm
```

### Advanced WiFi Monitoring
```yaml
# Multiple WiFi network monitoring
wifi:
  networks:
    - ssid: !secret wifi_ssid
      password: !secret wifi_password
      priority: 1
    - ssid: !secret backup_wifi_ssid
      password: !secret backup_wifi_password
      priority: 0.5

# Enhanced WiFi status with multiple criteria
binary_sensor:
  - platform: template
    name: "WiFi Connection Status"
    id: connection_status
    lambda: !lambda
      // Check signal strength, connection time, and error rate
      return (id(wifi_signal_strength).state > -75) && 
             (id(uptime_sensor).state > 60) && 
             (id(wifi_error_count).state < 5);
             
  - platform: template
    name: "WiFi Quality"
    id: wifi_quality
    lambda: !lambda
      float signal = id(wifi_signal_strength).state;
      if (signal > -50) return "Excellent";
      if (signal > -60) return "Good";  
      if (signal > -70) return "Fair";
      if (signal > -80) return "Poor";
      return "Very Poor";
```

### WiFi Optimization
```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: true         # Skip network scanning
  output_power: 12.0         # Maximum transmission power
  # Enable WiFi power saving
  power_save_mode: LIGHT
```

## 📱 Custom Display Layouts

### Dynamic Display Content
```yaml
# Conditional display based on data availability
lambda: !lambda |-
  time_t now = id(homeassistant_time).now().timestamp;
  
  // Burn-in protection
  if (now % 1800 == 0) {
    it.clear();
    return;
  }
  
  it.clear();
  
  // Header with time
  it.printf(0, 0, id(baloo_18_500), TextAlign::TOP_LEFT, 
           "%02d:%02d", 
           id(homeassistant_time).now().hour, 
           id(homeassistant_time).now().minute);
  
  // Main power display
  if (!isnan(id(power2).state)) {
    it.printf(it.get_width()/2, 30, id(baloo_32_700), TextAlign::CENTER, 
             "%.1f W", id(power2).state);
  } else {
    it.printf(it.get_width()/2, 30, id(baloo_32_700), TextAlign::CENTER, "No Data");
  }
  
  // Secondary information
  if (!isnan(id(voltage2).state)) {
    it.printf(0, it.get_height() + 12, id(baloo_18_700), TextAlign::BOTTOM_LEFT, 
             "%.1f V", id(voltage2).state);
  }
  
  if (!isnan(id(current2).state)) {
    it.printf(it.get_width(), it.get_height() + 12, id(baloo_18_700), TextAlign::BOTTOM_RIGHT, 
             "%.4f A", id(current2).state);
  }
  
  // WiFi status with signal strength
  float signal = id(wifi_signal_strength).state;
  if (!isnan(signal)) {
    if (signal > -70) {
      it.print(it.get_width(), 0, id(icons_18), TextAlign::TOP_RIGHT, "\ue63e");
    } else {
      it.print(it.get_width(), 0, id(icons_18), TextAlign::TOP_RIGHT, "\ue648");
    }
  }
```

### Custom Font Loading
```yaml
# Additional fonts for different display modes
font:
  - file:
      type: gfonts
      family: Roboto+Mono
      weight: 400
    id: mono_14
    size: 14
  
  - file:
      type: gfonts
      family: Material+Icons
    id: material_icons_24
    size: 24
    glyphs: ["\ue63e", "\ue648", "\ue8d5", "\ue84f"]  # WiFi, WiFi off, battery, temperature
  
  - file:
      type: gfonts
      family: JetBrains+Mono
      weight: 700
    id: jetbrains_mono_12
    size: 12
```

### Display Themes
```yaml
# Display theme switching based on time of day
lambda: !lambda |-
  time_t now = id(homeassistant_time).now().timestamp;
  int hour = id(homeassistant_time).now().hour;
  
  // Night mode (10 PM - 6 AM)
  bool night_mode = (hour >= 22) || (hour <= 6);
  
  if (now % 1800 == 0) {
    it.clear();
    return;
  }
  
  it.clear();
  
  // Night mode: dimmer colors, larger text
  if (night_mode) {
    it.set_brightness(0.3);  // Assuming display supports brightness
    it.print(-1, -4, id(baloo_18_500), "Night Mode");
    it.printf(it.get_width()/2, 35, id(baloo_32_700), TextAlign::CENTER, "%.1f W", id(power2).state);
  } else {
    it.set_brightness(1.0);
    it.print(-1, -4, id(baloo_18_500), "Energy Meter");
    it.printf(it.get_width()/2, 38, id(baloo_32_700), TextAlign::CENTER, "%.1f W", id(power2).state);
  }
  
  // Rest of display logic...
```

## 🔧 Advanced Sensor Filtering

### Exponential Moving Average
```yaml
# Smooth out noisy readings
sensor:
  - platform: modbus_controller
    modbus_controller_id: jsymk
    id: power2_smoothed
    name: "Power 2 (Smoothed)"
    address: 0x0052
    unit_of_measurement: "W"
    register_type: holding
    value_type: U_DWORD
    filters:
      - multiply: 0.0001
      - exponential_moving_average:
          alpha: 0.1      # Lower values = more smoothing
          beta: 0.9
```

### Outlier Detection
```yaml
# Remove extreme outliers
sensor:
  - platform: modbus_controller
    modbus_controller_id: jsymk
    id: power2_filtered
    name: "Power 2 (Filtered)"
    address: 0x0052
    unit_of_measurement: "W"
    register_type: holding
    value_type: U_DWORD
    filters:
      - multiply: 0.0001
      - delta: 100.0      # Remove changes > 100W
      - clamp:
          min: 0.0
          max: 10000.0    # Clamp to reasonable range
```

### Calibration Filters
```yaml
# Apply calibration corrections
sensor:
  - platform: modbus_controller
    modbus_controller_id: jsymk
    id: voltage2_calibrated
    name: "Voltage 2 (Calibrated)"
    address: 0x0050
    unit_of_measurement: "V"
    register_type: holding
    value_type: U_DWORD
    filters:
      - multiply: 0.0001
      - offset: -2.3      # Apply measured offset
      - calibrate_linear:
          - 0.0 -> 0.0
          - 240.0 -> 242.1  # Correct for systematic error
```

## 📡 MQTT Integration

### MQTT Configuration
```yaml
# Alternative to Home Assistant API
mqtt:
  broker: 192.168.1.100
  port: 1883
  username: !secret mqtt_username
  password: !secret mqtt_password
  topic_prefix: energy_meter
  
sensor:
  - platform: modbus_controller
    # ... other config ...
    mqtt:
      state_topic: "energy_meter/power_2/state"
      value_template: "{{ value }}"
```

### MQTT Sensors
```yaml
# Send sensor data to MQTT topics
sensor:
  - platform: mqtt_sensormqtt:
      name: "ESP32 Power"
      state_topic: "energy_meter/power"
      unit_of_measurement: "W"
      value_template: "{{ value | float(0) }}"
```

## 🌍 Remote Configuration

### HTTP Server
```yaml
# Simple web server for remote configuration
web_server:
  port: 80
  auth:
    username: admin
    password: !secret web_password
```

### API Endpoints
```yaml
# Custom API for external control
api:
  services:
    - service: set_update_interval
      variables:
        interval: int
      then:
        - lambda: >-
            id(jsymk).set_update_interval(interval);
```

### Over-the-Air Updates
```yaml
# Enhanced OTA with progress monitoring
ota:
  - platform: esphome
    password: !secret ota_password
    id: ota_component
    platformio_options:
      board_build.extra_flags:
        - -DESP32_OTA_PROGRESS_PIN=2
        - -DESP32_OTA_PROGRESS_LED=4
```

## 📊 Data Logging and Analytics

### Local Data Storage
```yaml
# Store data locally for analysis
text_sensor:
  - platform: template
    name: "Last Update"
    id: last_update
    lambda: !lambda
      return to_string(id(homeassistant_time).now().strftime("%Y-%m-%d %H:%M:%S"));
```

### Custom Analytics
```yaml
# Calculate energy statistics
sensor:
  - platform: template
    name: "Peak Power Today"
    id: peak_power_today
    unit_of_measurement: "W"
    state_class: measurement
    lambda: !lambda
      static float max_power = 0.0;
      float current_power = id(power2).state;
      if (current_power > max_power) {
        max_power = current_power;
      }
      // Reset at midnight
      if (id(homeassistant_time).now().hour == 0 && id(homeassistant_time).now().minute == 0) {
        max_power = 0.0;
      }
      return max_power;
```

### Power Factor Monitoring
```yaml
# Advanced power quality monitoring
sensor:
  - platform: template
    name: "Power Quality Score"
    id: power_quality_score
    unit_of_measurement: "%"
    lambda: !lambda
      float voltage = id(voltage2).state;
      float frequency = id(frequency).state;
      float pf = id(powerfactor2).state;
      
      int score = 100;
      
      // Voltage quality (220-240V)
      if (voltage < 220) score -= (220 - voltage) * 2;
      if (voltage > 240) score -= (voltage - 240) * 2;
      
      // Frequency quality (49-51 Hz)
      if (frequency < 49) score -= (49 - frequency) * 10;
      if (frequency > 51) score -= (frequency - 51) * 10;
      
      // Power factor quality (>0.9)
      if (pf < 0.9) score -= (0.9 - pf) * 50;
      
      return clamp(score, 0, 100);
```

## 🔄 Conditional Updates

### Smart Update Intervals
```yaml
# Adaptive update intervals based on power consumption
sensor:
  - platform: template
    id: dynamic_update_interval
    lambda: !lambda
      float power = id(power2).state;
      if (power > 1000) return 1.0;      // High power: update every 1s
      if (power > 100) return 2.0;       // Medium power: update every 2s
      return 5.0;                         // Low power: update every 5s
```

### Conditional Display Updates
```yaml
# Only update display when values change significantly
lambda: !lambda |-
  static float last_power = -1.0;
  static float last_voltage = -1.0;
  static float last_current = -1.0;
  
  float current_power = id(power2).state;
  float current_voltage = id(voltage2).state;
  float current_current = id(current2).state;
  
  bool need_update = false;
  
  // Check for significant changes (>5% or >50W)
  if (fabs(current_power - last_power) > max(last_power * 0.05, 50.0)) {
    need_update = true;
  }
  
  // Update display only if needed
  if (need_update || millis() % 30000 < 1000) {  // Force update every 30s
    last_power = current_power;
    last_voltage = current_voltage;
    last_current = current_current;
    
    // Display update logic...
  }
```

## 🔒 Security Enhancements

### Secure Communication
```yaml
# Enable HTTPS and secure connections
web_server:
  port: 443
  ssl_certificate: /config/cert.pem
  ssl_private_key: /config/key.pem
  
mqtt:
  broker: 192.168.1.100
  port: 8883
  username: !secret mqtt_username
  password: !secret mqtt_password
  certificate: /config/ca.pem
```

### Access Control
```yaml
# API authentication
api:
  encryption:
    key: !secret api_key
  services:
    - service: set_calibration
      variables:
        offset: float
      then:
        - lambda: >-
            if (id(homeassistant_time).now().hour < 6 || 
                id(homeassistant_time).now().hour > 22) {
              // Only allow calibration during daytime
              return;
            }
```

## 🎯 Performance Optimization

### Memory Management
```yaml
esp32:
  board: esp32dev
  framework:
    type: esp-idf
  psram:
    mode: octal
    speed: 80mhz
    
# Optimize heap usage
substitutions:
  update_interval: "5s"
  font_cache_size: "3"
```

### CPU Optimization
```yaml
# Optimize for low power consumption
esp32:
  board: esp32dev
  board_flash_mode: qio
  board_flash_freq: 80m
  board_flash_size: 4MB
  
# Power management
deep_sleep:
  run_duration: 30s     # Active for 30s
  sleep_duration: 60s   # Sleep for 60s
```

## 📈 Custom Dashboard Widgets

### Power Usage Widget
```yaml
# Custom widget for real-time power display
web_server:
  index: index.html
  
# Custom HTML for power display
frontend:
  themes:
    default:
      primary-color: "#03a9f4"
      accent-color: "#ff5722"
```

## 🔧 Troubleshooting Advanced Features

### Debug Logging
```yaml
logger:
  level: DEBUG
  logs:
    modbus: DEBUG
    sensor: DEBUG
    wifi: DEBUG
```

### Performance Monitoring
```yaml
sensor:
  - platform: uptime
    name: "Uptime"
    id: uptime_sensor
    
  - platform: template
    name: "Free Heap"
    lambda: !lambda
      return ESP.getFreeHeap();
    unit_of_measurement: "bytes"
    
  - platform: template
    name: "WiFi Errors"
    id: wifi_error_count
    lambda: !lambda
      // Count WiFi connection errors
      static int error_count = 0;
      return error_count;
```

For more advanced features and customization options, refer to the [ESPHome Documentation](https://esphome.io/) and experiment with the configuration to suit your specific needs.