# ESP32 BH1750 Light Sensor MQTT Template

## Overview
This is an ESP32-based template project for creating light sensor monitoring systems. It provides a complete foundation for collecting ambient light data from a BH1750 I2C sensor and publishing it to MQTT. The template includes web-based configuration, over-the-air updates, and robust connectivity with automatic recovery features.

**Note**: This is a template project derived from ESP32-MQTT-Weather. Use this as a starting point for your own ESP32 IoT projects with sensor data collection and MQTT publishing capabilities.
## Features
- **BH1750 Light Sensor**: High-precision ambient light measurement via I2C
- **MQTT Publishing**: Automatic data publishing with configurable intervals
- **Web Configuration**: Captive portal for easy WiFi and MQTT setup
- **Over-the-Air Updates**: Remote firmware and file system updates
- **System Monitoring**: WiFi signal strength, CPU temperature, and system metrics
- **Watchdog Protection**: Automatic recovery from system failures
- **Persistent Configuration**: Settings stored in ESP32 NVS (Non-Volatile Storage)

## Hardware Components
- **ESP32 Development Board** (tested with ESP32 DevKit V1)
- **BH1750 Light Sensor Module** (I2C interface)
- **Power Supply**: 5V USB or 3.3V-5V power source

## BH1750 Sensor Specifications
- **Communication**: I2C interface (address 0x23)
- **Resolution**: 16-bit ADC, 1 lux resolution
- **Range**: 1 - 65535 lux
- **Accuracy**: ±20% typical

## Wiring Connections
| BH1750 Pin | ESP32 Pin | Purpose                    |
|------------|-----------|----------------------------|
| VCC        | 3.3V      | Power supply               |
| GND        | GND       | Ground                     |
| SCL        | GPIO22    | I2C Clock                  |
| SDA        | GPIO21    | I2C Data                   |
| ADDR       | GND       | I2C Address (0x23)         |

| Function      | ESP32 Pin     | Purpose                           |
|---------------|---------------|-----------------------------------|
| Status LED    | GPIO2         | System status and MQTT heartbeat |
| Boot Button   | GPIO0         | Enter configuration mode          |

## Light Level Reference
The BH1750 sensor measures ambient light in lux units. Here are some typical readings:

| Light Condition              | Approximate Lux Range |
|------------------------------|----------------------|
| Darkness                     | 0 - 1 lux           |
| Moonlight                    | 0.1 - 1 lux         |
| Indoor lighting (dim)        | 50 - 200 lux        |
| Office lighting              | 300 - 500 lux       |
| Bright office               | 500 - 1000 lux      |
| Overcast day (outdoor)      | 1000 - 5000 lux     |
| Full daylight (outdoor)     | 10000 - 25000 lux   |
| Direct sunlight             | 25000 - 100000 lux  |

*Note: These values are approximate and can vary based on specific conditions and sensor calibration.*
- For troubleshooting, monitor the serial log output via USB.

## MQTT Data Structure

The ESP32 publishes light sensor data in JSON format to the configured MQTT topic. The message includes both system metrics and BH1750 sensor readings.

### Data Message Format
```json
{
  // ESP32 processor metrics
  "processor": {
    "WiFiRSSI": 75,                    // WiFi signal strength (% - 0-100%)
    "IPAddress": "192.168.1.150",      // Current IP address
    "CPUTemperature": 32.0,            // ESP32 CPU temperature (°C - placeholder)
    "SoftwareVersion": "1.3.0",        // Firmware version
    "ChipID": "A1B2C3D4E5F6",         // ESP32 unique chip identifier
    "WDTRestartCount": 2               // Watchdog restart counter
  },
  
  // BH1750 light sensor data
  "data": {
    "sunlight": 1250.5,                    // Light intensity in lux (float)
    "sensor_ok": true                      // Sensor status (true = successful read)
  }
}
```

### Data Fields Description

#### Processor Metrics (System Information)
- **WiFiRSSI**: WiFi signal strength as percentage (0-100%)
- **IPAddress**: Current IP address assigned by DHCP
- **CPUTemperature**: ESP32 internal temperature (placeholder implementation)
- **SoftwareVersion**: Firmware version (auto-generated from CMakeLists.txt VERSION field)
- **ChipID**: Unique ESP32 chip identifier (MAC-based)
- **WDTRestartCount**: Number of watchdog-triggered restarts (reliability metric)

#### Light Sensor Data
- **sunlight**: Light intensity in lux units (float, 0.0 - 65535.0)
- **sensor_ok**: Boolean indicating successful sensor reading

### Data Publishing Behavior
- **Successful Reading**: MQTT message published with current sensor data
- **Failed Reading**: No MQTT message published (prevents invalid data)
- **Watchdog Protection**: ESP32 resets if no successful MQTT publish within timeout period
## Configuration and Setup

### Initial Setup
1. **Hardware Connection**: Wire the BH1750 sensor to ESP32 as shown in the wiring table
2. **Power On**: Connect ESP32 to power source via USB
3. **Configuration Mode**: 
   - Hold the boot button (GPIO0) during power-on for 10 seconds
   - ESP32 will create a WiFi access point named "ESP32-CONFIG-XXXX"
4. **Web Configuration**:
   - Connect to the ESP32 access point
   - Open any website in your browser (captive portal will redirect)
   - Configure WiFi credentials and MQTT broker settings
   - Set device name and sampling interval
5. **Normal Operation**: ESP32 will restart and begin collecting/publishing data

### Configuration Parameters
- **WiFi SSID/Password**: Network credentials for internet connectivity
- **MQTT Broker URL**: MQTT server address (e.g., mqtt://192.168.1.100)
- **Data Topic**: MQTT topic for sensor data publishing
- **Sample Interval**: Time between sensor readings (milliseconds)

### Web Interface Features
- **System Information**: View current configuration and system status
- **OTA Updates**: Upload new firmware or file system images
- **Configuration Export**: Download current settings as JSON
- **Reset Options**: Factory reset or watchdog counter reset

## Applications and Use Cases
- **Smart Home**: Automatic lighting control based on ambient light
- **Greenhouse Monitoring**: Track natural light levels for plant growth
- **Energy Management**: Solar panel efficiency monitoring
- **Building Automation**: HVAC and lighting optimization
- **Environmental Monitoring**: Indoor/outdoor light level logging
- **Photography**: Light meter for optimal exposure settings
- **Circadian Lighting**: Adjust artificial lighting based on natural light cycles

## MQTT Integration Examples

### Home Assistant Integration
```yaml
# configuration.yaml
sensor:
  - platform: mqtt
    name: "ESP32 Light Sensor"
    state_topic: "sensors/light/data"
    value_template: "{{ value_json.data.sunlight }}"
    unit_of_measurement: "lux"
    device_class: "illuminance"
    
  - platform: mqtt
    name: "ESP32 WiFi Signal"
    state_topic: "sensors/light/data"
    value_template: "{{ value_json.processor.WiFiRSSI }}"
    unit_of_measurement: "%"
    
binary_sensor:
  - platform: mqtt
    name: "ESP32 Light Sensor Status"
    state_topic: "sensors/light/data"
    value_template: "{{ value_json.data.sensor_ok }}"
    payload_on: true
    payload_off: false
```

### InfluxDB Line Protocol
```influxdb
light_sensor,device=ESP32-Light-Sensor,location=living_room sunlight=1250.5,wifi_rssi=75,sensor_ok=1 1642234567890000000
```

### Node-RED Flow Example
```json
[
  {
    "id": "mqtt_in",
    "type": "mqtt in",
    "topic": "sensors/light/data",
    "broker": "mqtt_broker"
  },
  {
    "id": "json_parse",
    "type": "json"
  },
  {
    "id": "light_switch",
    "type": "switch",
    "property": "payload.data.sunlight",
    "rules": [
      {"t": "lt", "v": "300", "vt": "num"},
      {"t": "gte", "v": "300", "vt": "num"}
    ]
  }
]
```

## Troubleshooting

### Common Issues

#### Sensor Not Reading
- **Check Wiring**: Verify BH1750 connections (VCC, GND, SCL, SDA)
- **I2C Address**: Ensure ADDR pin is connected to GND (address 0x23)
- **Power Supply**: BH1750 requires 2.4V - 3.6V (use ESP32 3.3V pin)
- **Pull-up Resistors**: Most BH1750 modules include built-in pull-ups

#### WiFi Connection Issues
- **Signal Strength**: Check WiFi signal strength in system info
- **Credentials**: Verify SSID and password in configuration
- **Network**: Ensure 2.4GHz WiFi (ESP32 doesn't support 5GHz)
- **Firewall**: Check if network blocks MQTT traffic

#### MQTT Publishing Problems
- **Broker Connection**: Verify MQTT broker URL and port
- **Network Connectivity**: Ensure ESP32 can reach MQTT broker
- **Topic Permissions**: Check if broker requires authentication
- **QoS Settings**: Verify MQTT quality of service configuration

**Common MQTT Connection Errors:**
```
E (xxxxx) esp-tls: [sock=xx] select() timeout
E (xxxxx) transport_base: Failed to open a new connection
E (xxxxx) mqtt_client: Error transport connect
```
**Solutions:**
- **Check MQTT Broker URL**: Ensure format is `mqtt://IP_ADDRESS` or `mqtt://IP_ADDRESS:PORT`
  - ✅ `mqtt://192.168.1.100` (uses default port 1883)
  - ✅ `mqtt://192.168.1.100:1883` (explicit port)
  - ❌ `192.168.1.100` (missing mqtt:// protocol)
- **Network Reachability**: Ping MQTT broker from same network as ESP32
- **Firewall Rules**: Check if port 1883 is blocked by router/firewall
- **Broker Status**: Verify MQTT broker service is running and accepting connections
- **Authentication**: If broker requires login, ensure credentials are configured
- **URL Format**: Use IP address instead of hostname if DNS issues suspected

#### Web Interface Not Accessible
- **Configuration Mode**: Hold boot button during power-on for 10 seconds
- **AP Mode**: Look for "ESP32-CONFIG-XXXX" WiFi network
- **Browser Issues**: Try different browser or clear cache
- **Firewall**: Disable firewall/antivirus temporarily

### Diagnostic Information
- **Serial Monitor**: Connect via USB for detailed log output
- **System Info**: Check `/sysinfo.json` endpoint for system status
- **LED Indicator**: GPIO2 LED blinks on successful MQTT publish
- **Watchdog Counter**: Monitor restart count for stability issues

### Factory Reset
1. Hold boot button during power-on for 10 seconds
2. Connect to ESP32-CONFIG-XXXX WiFi network
3. Access web interface
4. Use "Factory Reset" option to clear all settings

## Technical Specifications

### Power Consumption
- **ESP32**: ~80mA active, ~10µA deep sleep
- **BH1750**: ~120µA active, <1µA standby
- **Total System**: ~85-100mA during normal operation

### Performance
- **Sensor Resolution**: 16-bit (65,536 levels)
- **Measurement Time**: ~120ms (high resolution mode)
- **I2C Speed**: 100kHz (configurable up to 400kHz)
- **MQTT Publish Rate**: Configurable (default: 5 seconds)
- **WiFi Reconnection**: Automatic with exponential backoff

### Memory Usage
- **Flash**: ~1.5MB firmware, ~500KB file system
- **RAM**: ~200KB free heap during operation
- **NVS**: <4KB for configuration storage

## Development and Customization

### Build Environment
- **Framework**: ESP-IDF v4.4 or later
- **Platform**: PlatformIO or Arduino IDE
- **Compiler**: Xtensa GCC cross-compiler
- **Dependencies**: ESP-IDF components (WiFi, MQTT, NVS, I2C)

### Version Management

The project implements **automatic version management** with a single source of truth approach to eliminate version synchronization issues and ensure consistency across all project components.

#### Architecture Overview
```
CMakeLists.txt (VERSION "X.Y.Z") 
    ↓ (CMake configure_file)
include/project_version.h.in (template)
    ↓ (Auto-generated during build)
include/project_version.h (header with constants)
    ↓ (Included by source code)
All project components use same version
```

#### Key Files and Roles

**Source of Truth:**
- **`CMakeLists.txt`**: Contains `project(ESP32-MQTT-TEMPLATE VERSION "X.Y.Z")` 
  - **THIS IS THE ONLY FILE YOU SHOULD EDIT FOR VERSION CHANGES**
  - Format: `project(ESP32-MQTT-TEMPLATE VERSION "1.4.0")`
  - Supports semantic versioning (MAJOR.MINOR.PATCH)

**Template System:**
- **`include/project_version.h.in`**: CMake template file
  - Contains `#define PROJECT_VERSION "@PROJECT_VERSION@"`
  - Do not edit manually - CMake processes this file
  - Template placeholders are replaced during build

**Generated Files (DO NOT EDIT MANUALLY):**
- **`include/project_version.h`**: Auto-generated header
  - Created by CMake during build process
  - Contains actual version constants
  - Included by C/C++ source files
  - Regenerated on every build if CMakeLists.txt changes

#### Automated Integration Points

The version system automatically integrates with all project components:

**1. Firmware Boot Logs:**
```
I (1234) ESP32-MQTT-TEMPLATE: Software Version: 1.4.0
I (1235) ESP32-MQTT-TEMPLATE: Build Date: Jul 23 2025
```

**2. MQTT JSON Messages:**
```json
{
  "processor": {
    "SoftwareVersion": "1.4.0",
    "BuildDate": "Jul 23 2025",
    ...
  },
  "data": { ... }
}
```

**3. Web Interface:**
- System information page displays current version
- About page shows version and build information
- OTA update pages reference current version

**4. System API Endpoints:**
```json
GET /sysinfo.json
{
  "firmware_version": "1.4.0",
  "build_date": "Jul 23 2025",
  ...
}
```

#### How to Update Version

**Step 1: Edit CMakeLists.txt**
```cmake
# Change this line only:
project(ESP32-MQTT-TEMPLATE VERSION "1.5.0")
```

**Step 2: Build Project**
```bash
pio run
# or
cmake --build build
```

**Step 3: Automatic Propagation**
- CMake detects version change
- Regenerates `include/project_version.h`
- All source files pick up new version
- Build output shows new version
- Runtime logs display new version

#### Version Verification

**Check Current Version:**
- **Build logs**: Version appears in compilation output
- **Serial monitor**: Boot sequence shows version
- **Web interface**: System information page
- **MQTT messages**: Published in processor metrics
- **Source code**: `PROJECT_VERSION` constant

**Build-Time Verification:**
```bash
# Check generated header
cat include/project_version.h

# Verify CMake configuration
cmake --build build --verbose
```

#### Benefits of This System

✅ **Single Source of Truth**: Only one place to update versions  
✅ **Automatic Synchronization**: No manual file editing required  
✅ **Build-Time Generation**: Version always matches build  
✅ **Runtime Availability**: Version accessible in all code  
✅ **Zero Maintenance**: No version file management needed  
✅ **Error Prevention**: Eliminates version mismatch issues  

#### Troubleshooting Version Issues

**Problem: Old version still showing after update**
```bash
# Solution: Clean and rebuild
pio run -t clean
pio run
```

**Problem: Version header not found**
```bash
# Solution: Ensure template file exists
ls include/project_version.h.in
```

**Problem: CMake not detecting version change**
```bash
# Solution: Force CMake reconfiguration
rm -rf .pio/build
pio run
```

#### Version History

The project maintains semantic versioning (MAJOR.MINOR.PATCH):
- **MAJOR**: Breaking changes or major feature additions
- **MINOR**: New features, backward compatible
- **PATCH**: Bug fixes, minor improvements

Example progression:
- `1.0.0` - Initial BH1750 implementation
- `1.1.0` - Added web configuration interface
- `1.1.1` - Fixed MQTT connection timeout
- `1.2.0` - Added OTA update capability
- `1.3.0` - Enhanced sensor error handling
- `1.4.0` - Implemented automatic version management

### Adding Additional Sensors
The codebase is designed for easy expansion:

```c
// Add new sensor to data structure
typedef struct {
    float sunlight;     // BH1750 light sensor
    float temperature;  // Additional temperature sensor
    float humidity;     // Additional humidity sensor
    bool sensors_ok;    // Combined sensor status
} input_data_t;
```

### Customizing MQTT Topics
```c
// Multiple topic publishing
esp_mqtt_client_publish(client, "sensors/light/lux", lux_string, 0, 1, 0);
esp_mqtt_client_publish(client, "sensors/light/status", status_string, 0, 1, 0);
esp_mqtt_client_publish(client, "sensors/system/wifi", wifi_string, 0, 1, 0);
```

## License and Attribution
This project is based on the ESP32MQTTBase template and has been adapted for BH1750 light sensor applications. 

- **Original Template**: ESP32 MQTT Data Template
- **Sensor Integration**: BH1750 I2C Light Sensor
- **Current Version**: Specialized for ambient light monitoring applications

For technical support or contributions, please refer to the project documentation and issue tracker.
