# ESP32 MQTT Template

This is a template project based on the ESP32MQTTBase project, designed to be used as a starting point for ESP32 projects that read data from various sources and publish it to MQTT.

## Template Overview

This template has been modified to remove BMS-specific code and replace it with placeholders marked with `PutInputCodeHere` and `PutOutputFormattingForMqttHere`. These placeholders indicate where you should add your specific input reading and output formatting code.

## Key Template Placeholders

### Input Code Placeholders (`PutInputCodeHere`)

1. **Data Structure Definition** (line ~123)
   - Replace `input_data_t` struct with your specific data structure
   - Add your actual data fields instead of the example fields

2. **Protocol Constants** (line ~138)
   - Define your protocol constants, commands, and communication parameters
   - Replace the template command with your actual communication protocol

3. **Input Reading Function** (line ~269)
   - Replace `read_input_data()` with your specific input reading logic
   - Add code to read from your sensors, devices, or data sources

4. **Data Parsing Function** (line ~295)
   - Replace `parse_input_data()` with your specific data parsing logic
   - Add protocol parsing, validation, and data extraction code

5. **Command Sending** (line ~1550)
   - Replace the template command sending with your actual device communication

### Output Formatting Placeholders (`PutOutputFormattingForMqttHere`)

1. **MQTT JSON Structure** (line ~1823)
   - Replace the example JSON structure with your specific data format
   - Customize the MQTT topic and message structure for your use case

## How to Use This Template

1. **Clone or copy this template project**
2. **Search for `PutInputCodeHere` placeholders** and replace with your input logic:
   - Define your data structures
   - Add your communication protocol
   - Implement your data reading and parsing functions
3. **Search for `PutOutputFormattingForMqttHere` placeholders** and replace with your output logic:
   - Customize the MQTT JSON structure
   - Add any specific data formatting requirements
4. **Update configuration**:
   - Modify default values in the code
   - Update the web interface labels if needed
5. **Test and validate**:
   - Verify your input reading works correctly
   - Check MQTT publishing with your data format

## Example Use Cases

This template can be adapted for:
- **Sensor monitoring**: Temperature, humidity, pressure sensors via I2C/SPI
- **Industrial protocols**: Modbus RTU/TCP, CAN bus, proprietary protocols
- **Serial device communication**: GPS modules, environmental sensors, scales
- **Network data collection**: HTTP APIs, TCP/UDP data sources
- **GPIO monitoring**: Digital/analog inputs, pulse counting

## Retained Features

The template preserves these useful features from the original project:
- **WiFi configuration**: Web-based WiFi setup with captive portal
- **MQTT connectivity**: Robust MQTT client with reconnection handling
- **Web interface**: Configuration management via web browser
- **OTA updates**: Over-the-air firmware and filesystem updates
- **Watchdog monitoring**: System stability monitoring and recovery
- **NVS configuration**: Persistent configuration storage
- **System monitoring**: CPU temperature, WiFi signal, IP address

## Configuration

The web interface provides configuration for:
- WiFi credentials (SSID/password)
- MQTT broker URL
- Data topic (customizable MQTT topic)
- Sample interval (data reading frequency)
- Device name (identifier in MQTT messages)
- Watchdog settings

## Building and Flashing

This project uses PlatformIO. To build and flash:

```bash
# Build the project
pio run

# Flash firmware and filesystem
pio run -t upload
pio run -t uploadfs

# Monitor serial output
pio device monitor
```

## File Structure

- `src/main.c` - Main application code with template placeholders
- `data/parameters.html` - Web interface for configuration
- `data/version.txt` - Version information displayed in web interface
- `platformio.ini` - PlatformIO project configuration
- `partitions.csv` - ESP32 partition table

## Support

This template is derived from the ESP32MQTTBase project. Refer to the original project documentation for detailed information about the underlying systems and configuration options.
