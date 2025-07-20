# Template Changelog

## Version 1.0.0-TEMPLATE

### Changes from ESP32MQTTBase
- **Converted to Template**: Removed BMS-specific code and replaced with placeholders
- **Added Placeholders**: 
  - `PutInputCodeHere` - For input reading and parsing logic
  - `PutOutputFormattingForMqttHere` - For MQTT output formatting
- **Generic Naming**: 
  - Changed `BMS_READER` tag to `DATA_READER`
  - Renamed `bms_topic` to `data_topic` 
  - Updated web interface labels
  - Changed default topic from "BMS/JKBMS" to "Data/Template"
- **Template Documentation**: Added README_TEMPLATE.md with usage instructions
- **Retained Features**: Preserved all WiFi, MQTT, OTA, and configuration functionality

### Template Structure
- `input_data_t` - Generic data structure (replace with your data fields)
- `read_input_data()` - Template for reading from your input sources
- `parse_input_data()` - Template for parsing your protocol/data format
- `publish_input_data_mqtt()` - Template for MQTT JSON formatting
- `template_read_cmd[]` - Example command structure

### Usage
1. Search for `PutInputCodeHere` placeholders and implement your input logic
2. Search for `PutOutputFormattingForMqttHere` placeholders and customize MQTT output
3. Update data structures, constants, and default values for your use case
4. Test with your specific hardware and protocols

### Compatibility
- Compatible with ESP32 DevKit V1 and similar boards
- Uses PlatformIO build system
- Requires ESP-IDF framework
