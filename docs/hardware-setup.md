# Hardware Setup Guide

## ⚠️ Safety Warning

⚠️ **HIGH VOLTAGE**: This project involves working with mains electricity. Always disconnect power before making connections and use appropriate safety measures. The authors are not responsible for any damage or injury resulting from the use of this project.

## 📋 Required Components

### Core Hardware
| Component | Specification | Quantity | Notes |
|-----------|---------------|----------|-------|
| ESP32 Board | Any ESP32 variant | 1 | DevKit, WROOM, etc. |
| Energy Meter | JSY with Modbus RTU | 1 | Main measurement device |
| OLED Display | SSD1306 128x64 I2C | 1 | For local display |
| RS485 Module | TTL to RS485 converter | 1 | For Modbus communication |
| Power Supply | 3.3V/5V | 1 | USB power recommended |

### Optional Components
| Component | Purpose | Notes |
|-----------|---------|-------|
| 3D Printed Enclosure | Protection | Recommended for permanent installation |
| Current Clamps | Non-invasive measurement | For measuring current without direct connection |
| Voltage Transformers | Voltage isolation | If voltage measurement is required |

## 🔌 Pin Connections

### ESP32 Pinout
```
ESP32 DevKit V1 Pin Layout:
         ┌─────────┐
    3V3  │1       38│ GND
     EN  │2       37│ GPIO0
   SENSOR │3       36│ GPIO2
   VP/GPIO36 │4       35│ GPIO4
   VN/GPIO39 │5       34│ GPIO16 ← RX for Modbus
   GPIO34 │6       33│ GPIO17 ← TX for Modbus
   GPIO35 │7       32│ GPIO5
   GPIO32 │8       31│ GPIO18 ← I2C SCL
   GPIO33 │9       30│ GPIO19 ← I2C SDA
    5V   │10      29│ GPIO21
    GND  │11      28│ GND
    GND  │12      27│ GPIO3 (RX0)
    GPIO25│13      26│ GPIO1 (TX0)
   GPIO26│14      25│ GPIO22 ← SCL (Alt)
   GPIO27│15      24│ GPIO23 ← SDA (Alt)
         └─────────┘
```

### Connection Table

| Component | ESP32 Pin | Signal | Notes |
|-----------|-----------|--------|-------|
| **Modbus RTU** | | | |
| RS485 Module | GPIO17 | TX | UART TX |
| RS485 Module | GPIO16 | RX | UART RX |
| RS485 Module | 3.3V | VCC | Power |
| RS485 Module | GND | GND | Ground |
| **I2C OLED Display** | | | |
| OLED SDA | GPIO21 | SDA | I2C Data |
| OLED SCL | GPIO22 | SCL | I2C Clock |
| OLED VCC | 3.3V | VCC | Power |
| OLED GND | GND | GND | Ground |

## 🔌 Detailed Wiring Guide

### Step 1: RS485 Modbus Setup

#### RS485 Module Connections
```
RS485 Module Pinout:
     ┌─────────┐
     │   VCC   │ ← 3.3V (ESP32)
     │   GND   │ ← GND (ESP32)
     │   RE/DE │ ← Not connected (automatic)
     │    RE   │ ← Not connected
     │    DE   │ ← Not connected
     │    DI   │ ← GPIO17 (ESP32 TX)
     │   RO/RX │ ← GPIO16 (ESP32 RX)
     └─────────┘
     │   A+    │ ← JSY Meter A+ (or Data+)
     │   B-    │ ← JSY Meter B- (or Data-)
     │   GND   │ ← Optional: Earth ground
     └─────────┘
```

#### JSY Energy Meter Connections
```
JSY Energy Meter:
     ┌─────────┐
     │  Power  │ ← AC Power (L/N)
     │ Current │ ← CT Clamp connections
     │   A+    │ ← RS485 Data+
     │   B-    │ ← RS485 Data-
     │   GND   │ ← Optional ground
     └─────────┘
```

### Step 2: I2C OLED Display Setup

#### OLED Display Connections
```
SSD1306 128x64 OLED:
     ┌─────────┐
     │   VCC   │ ← 3.3V (ESP32)
     │   GND   │ ← GND (ESP32)
     │   SDA   │ ← GPIO21 (ESP32 SDA)
     │   SCL   │ ← GPIO22 (ESP32 SCL)
     └─────────┘
```

#### I2C Address Configuration
The default I2C address for SSD1306 is `0x3C`. If you have multiple I2C devices, you may need to use `0x3D` by connecting the address pin to VCC.

## 🔧 Hardware Assembly Steps

### Step 1: Prepare the ESP32
1. **Verify ESP32**: Check that your ESP32 board is working properly
2. **Install Headers**: Solder pin headers if not pre-installed
3. **Test Power**: Connect to USB and verify power-on LED

### Step 2: Install RS485 Module
1. **Position Module**: Place RS485 module on breadboard or PCB
2. **Connect Power**: Wire 3.3V and GND to ESP32
3. **Connect UART**: Wire TX/RX pins as shown in table above
4. **Connect to Meter**: Wire A+ and B- to JSY energy meter

### Step 3: Install OLED Display
1. **Position Display**: Mount OLED in visible location
2. **Wire I2C**: Connect SDA, SCL, VCC, and GND
3. **Test Communication**: Verify I2C detection (see troubleshooting)

### Step 4: Connect Energy Meter
1. **Power Off**: Ensure all power is disconnected
2. **Wire Modbus**: Connect A+ and B- wires to RS485 module
3. **Wire Power**: Connect AC power wires to meter (if required)
4. **Wire Current**: Connect current transformers (if used)

## 🏗️ Enclosure Recommendations

### 3D Printed Enclosure
- **Material**: ABS or PLA
- **Dimensions**: Minimum 100mm x 60mm x 30mm
- **Features Needed**:
  - Ventilation holes for heat dissipation
  - Cutouts for OLED display
  - Access holes for wiring
  - Mounting holes for PCB

### Commercial Enclosure
- **Type**: IP65 rated plastic or metal enclosure
- **Size**: DIN rail mount or wall mount
- **Features**:
  - Clear window for OLED display
  - Cable entry glands
  - Grounding lug

## 📏 Calibration and Testing

### Initial Power-On
1. **Connect Power**: Apply power to ESP32 via USB
2. **Check Serial**: Monitor serial output for boot messages
3. **Verify WiFi**: Check WiFi connection status
4. **Test Display**: Verify OLED shows "Energy Meter" title

### Modbus Communication Test
1. **Check Logs**: Look for "JSY energy meter protocol" messages
2. **Verify Address**: Ensure meter address is 0x1 (default)
3. **Monitor Communication**: Watch for read errors or timeouts

### Sensor Validation
1. **Check Values**: Verify all sensors show reasonable readings
2. **Verify Display**: Confirm power, voltage, current display correctly
3. **Test WiFi Icon**: Verify WiFi status indicator functionality

## 🔍 Troubleshooting Hardware Issues

### Common Problems

#### No Power to ESP32
- **Check USB cable**: Ensure data cable (not charge-only)
- **Check voltage**: Verify 3.3V and 5V rails
- **Check connections**: Verify all power connections

#### OLED Not Displaying
- **Check I2C address**: Default should be 0x3C
- **Check wiring**: Verify SDA/SCL connections
- **Check library**: Ensure SSD1306 library is loaded

#### Modbus Communication Failure
- **Check RS485 wiring**: Verify A+ and B- connections
- **Check address**: Ensure meter address matches configuration
- **Check termination**: Add 120Ω termination if long cables
- **Check power**: Verify meter is powered

#### Intermittent Readings
- **Check power supply**: Ensure stable power to all components
- **Check connections**: Verify all wire connections are secure
- **Check EMI**: Move away from high-voltage areas
- **Add filtering**: Use ferrite beads on power lines

### Diagnostic Tools
- **Serial Monitor**: Monitor ESP32 serial output
- **I2C Scanner**: Use I2C scan to find devices
- **Oscilloscope**: Check RS485 signal quality
- **Multimeter**: Verify voltage levels and continuity

## 📋 Final Checklist

Before proceeding to software configuration:

- [ ] ESP32 powers on and boots successfully
- [ ] OLED display shows "Energy Meter" title
- [ ] WiFi connection established
- [ ] Modbus communication working
- [ ] Energy meter readings are stable
- [ ] All connections are secure
- [ ] Enclosure (if used) is properly assembled
- [ ] Safety measures are in place

Once hardware is verified, proceed to the [Configuration Guide](configuration-guide.md) for ESPHome setup.