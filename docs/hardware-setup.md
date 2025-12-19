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

## 🏭 Original Author's Recommended Hardware

The following specific components were tested and recommended by Giovanni Aggiustatutto in the original project:

### Recommended Hardware List
| Component | Model/Specification | Source | Purpose |
|-----------|-------------------|--------|---------|
| **Energy Meter** | JSY-MK-194G | [AliExpress](https://www.aliexpress.com/item/1005007369940517.html) | Main measurement device with Modbus RTU |
| **ESP32 Board** | ESP32 Wemos D1 Mini | General ESP32 supplier | Main controller board |
| **Power Supply** | Meanwell APV-8-5 5V 8W | Electronics supplier | 5V power supply for the system |
| **OLED Display** | SSD1306 I2C 0.96" | General electronics supplier | Local display for readings |

### Hardware Details

#### JSY-MK-194G Energy Meter
- **Protocol**: Modbus RTU over RS485
- **Channels**: Dual channel measurement capability
- **Accuracy**: High precision power monitoring
- **Communication**: RS485 interface for reliable data transmission
- **Features**: Voltage, current, power, energy, and frequency measurement

#### ESP32 Wemos D1 Mini
- **Board Type**: Compact ESP32 development board
- **Pins**: Suitable for I2C and UART communication
- **WiFi**: Built-in WiFi for Home Assistant connectivity
- **Memory**: Sufficient for ESPHome firmware and display operations

#### Meanwell APV-8-5 Power Supply
- **Output**: 5V DC, 8W maximum power
- **Efficiency**: High efficiency switching power supply
- **Protection**: Over voltage and over current protection
- **Reliability**: Industrial-grade power supply for continuous operation

#### SSD1306 0.96" OLED Display
- **Resolution**: 128x64 pixels
- **Interface**: I2C communication
- **Size**: Compact 0.96" display suitable for enclosure mounting
- **Visibility**: High contrast OLED for clear reading visibility

### Why These Components?

These specific components were chosen by the original author because they:
- **Provide reliable performance** in energy monitoring applications
- **Are readily available** from common electronics suppliers
- **Work seamlessly together** with the ESPHome firmware
- **Offer good value** for the performance provided
- **Have community support** and documentation available

### Alternative Options

While these specific components are recommended, the project is compatible with:
- **Other JSY energy meter models** with Modbus RTU capability
- **Various ESP32 development boards** (DevKit, NodeMCU, etc.)
- **Different 5V power supplies** with adequate current capacity
- **Other SSD1306 OLED displays** in various sizes

## 🔌 Pin Connections

### ESP32 Wemos D1 Mini Pinout (Recommended)
```
ESP32 Wemos D1 Mini Pin Layout:
         ┌─────────┐
     3V3 │ RST   A0 │
      D0 │ D1    D2 │
      D3 │ GND   D4 │
      5V │ 3V3   GND │
         └─────────┘

GPIO Pin Mapping (Recommended for this project):
D0  = GPIO16 ← RX for Modbus
D1  = GPIO22 ← I2C SCL  
D2  = GPIO21 ← I2C SDA
D3  = GPIO0  (Boot select - don't use)
D4  = GPIO2  (Status LED - built-in)
D5  = GPIO14 (Available for expansion)
D6  = GPIO12 (Available for expansion)  
D7  = GPIO13 (Available for expansion)
D8  = GPIO15 (Available for expansion)
A0  = ADC0   (Available for analog sensors)

Note: Pin labels on D1 Mini correspond to specific GPIO numbers
```

Alternative ESP32 DevKit V1 Pinout:
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
| **Power Supply** | | | |
| 5V Power Supply (+) | +5V | VCC | Main ESP32 power |
| 5V Power Supply (-) | GND | GND | Power ground |
| **Energy Meter Power** | | | |
| JSY Meter VCC | 3.3V | VCC | Energy meter power |
| JSY Meter GND | GND | GND | Energy meter ground |
| **Modbus RTU Communication** | | | |
| RS485 Module TX | GPIO16 | RX | Modbus RX (Pin 16) |
| RS485 Module RX | GPIO17 | TX | Modbus TX (Pin 17) |
| RS485 Module VCC | 3.3V | VCC | Module power |
| RS485 Module GND | GND | GND | Module ground |
| **I2C OLED Display** | | | |
| OLED SDA | GPIO21 | SDA | I2C Data (Pin 21) |
| OLED SCL | GPIO22 | SCL | I2C Clock (Pin 22) |
| OLED VCC | 3.3V | VCC | Display power |
| OLED GND | GND | GND | Display ground |

## 🔌 Detailed Wiring Guide

### Step 0: Power Supply Setup (Critical)

#### Power Distribution Overview
The original author's design uses a centralized power distribution approach:

```
Mains AC (L/N) 
    ↓
5V Power Supply (Meanwell APV-8-5)
    ↓
ESP32 +5V pin (main power)
    ↓
ESP32 3.3V pin (internal regulator)
    ↓
Energy Meter VCC, OLED VCC, RS485 Module VCC
```

#### 5V Power Supply Connections
```
5V Power Supply (Meanwell APV-8-5):
     ┌─────────┐
     │    +    │ ← Mains Line (L)
     │    -    │ ← Mains Neutral (N)
     │   5V+   │ ← ESP32 +5V pin
     │   GND   │ ← ESP32 GND pin
     └─────────┘
```

#### ESP32 Power Distribution
```
ESP32 Power Rails:
+5V pin  → Powers ESP32 main circuit
3.3V pin → Powers external devices:
           • JSY Energy Meter VCC
           • OLED Display VCC  
           • RS485 Module VCC
GND pin  → Common ground for all devices
```

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
     │   VCC   │ ← ESP32 3.3V pin
     │   GND   │ ← ESP32 GND pin
     │    L    │ ← Mains Line (AC Power)
     │    N    │ ← Mains Neutral (AC Power)
     │   A+    │ ← RS485 Data+
     │   B-    │ ← RS485 Data-
     │   CT1   │ ← Current Transformer 1
     │   CT2   │ ← Current Transformer 2
     └─────────┘
```

**Note**: The energy meter receives its operating power (VCC/GND) from the ESP32's 3.3V rail, while the mains power (L/N) is for measurement purposes only.

### Step 2: I2C OLED Display Setup

#### OLED Display Connections
```
SSD1306 128x64 OLED:
     ┌─────────┐
     │   VCC   │ ← 3.3V (ESP32)
     │   GND   │ ← GND (ESP32)
     │   SDA   │ ← GPIO21 (D2) ← I2C SDA
     │   SCL   │ ← GPIO22 (D1) ← I2C SCL
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
2. **Wire Power**: Connect VCC to ESP32 3.3V, GND to ESP32 GND
3. **Wire Modbus**: Connect A+ and B- wires to RS485 module
4. **Wire Mains**: Connect L and N for measurement (high voltage!)
5. **Wire Current**: Connect current transformers (if used)

### Step 5: Final Power Distribution Verification

#### Complete Power Chain (as per original author design):
```
Mains AC (120V/230V)
    ↓
5V Power Supply (+5V, GND)
    ↓
ESP32 Wemos D1 Mini (+5V, GND pins)
    ↓
ESP32 Internal 3.3V Regulator
    ↓
ESP32 3.3V pin supplies:
    ├── JSY Energy Meter (VCC, GND)
    ├── SSD1306 OLED Display (VCC, GND)
    └── RS485 Module (VCC, GND)
```

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
- **Check 5V power supply**: Verify mains connection and 5V output
- **Check power supply polarity**: Ensure +5V to ESP32 +5V pin, GND to GND
- **Check ESP32 3.3V**: Verify internal regulator is working (should power peripherals)
- **Check connections**: Verify all power connections match the original design

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
- [ ] Safety measures are in place

Once hardware is verified, proceed to the [Configuration Guide](configuration-guide.md) for ESPHome setup.
