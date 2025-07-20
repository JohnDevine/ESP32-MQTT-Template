# ESP32 MQTT Data Template

## Overview
This is a template project for ESP32-based data collection and MQTT publishing systems. It has been derived from the ESP32MQTTBase project but converted to a generic template that can be easily adapted for various data sources and IoT applications.

## What This Template Provides
- **Generic data input framework** with clearly marked placeholders for your specific implementation
- **Robust MQTT connectivity** with automatic reconnection and publish confirmation
- **Web-based configuration** via captive portal for easy setup and management
- **Over-the-Air (OTA) updates** for both firmware and web interface files
- **System monitoring** including WiFi signal strength, CPU temperature, and uptime tracking
- **Watchdog functionality** for automatic system recovery from failures
- **Persistent configuration** storage using ESP32 NVS (Non-Volatile Storage)

## Template Placeholders

### Input Code (`PutInputCodeHere`)
Search for this marker to find locations where you should implement your specific data collection logic:

1. **Data Structure Definition**: Replace `input_data_t` with your data fields
2. **Protocol Constants**: Define your communication commands and constants  
3. **Data Reading Function**: Implement `read_input_data()` for your data source
4. **Data Parsing Function**: Implement `parse_input_data()` for your protocol
5. **Command Sending**: Replace template commands with your device communication

### Output Formatting (`PutOutputFormattingForMqttHere`)
Search for this marker to customize MQTT output:

1. **JSON Structure**: Modify the MQTT message format for your data
2. **Field Mapping**: Add your specific data fields to the JSON output
3. **Topic Strategy**: Customize MQTT topics for your use case

## Example Use Cases
This template can be adapted for:
- **Sensor Networks**: Temperature, humidity, pressure, air quality sensors
- **Industrial Monitoring**: Modbus devices, PLC communication, equipment status
- **Environmental Monitoring**: Weather stations, soil moisture, water level sensors
- **Energy Management**: Solar inverters, smart meters, battery systems
- **Building Automation**: HVAC monitoring, occupancy sensors, smart lighting
- **Asset Tracking**: GPS modules, RFID readers, proximity sensors

## Hardware Requirements
- **ESP32 Development Board** (tested with ESP32 DevKit V1)
- **Communication Interface** (depends on your data source):
  - RS485/Modbus: MAX485 module
  - I2C/SPI: Direct connection to sensors
  - Serial: UART connection
  - Network: WiFi (built-in) or Ethernet module
- **Power Supply**: 5V USB or appropriate power source for your application

## Pin Configuration
The template uses these pins by default (customize for your needs):

| Function      | ESP32 Pin     | Purpose                           |
|---------------|---------------|-----------------------------------|
| Status LED    | GPIO2         | System status and MQTT heartbeat |
| UART TX       | GPIO17        | Serial communication (if needed)  |
| UART RX       | GPIO16        | Serial communication (if needed)  |
| Boot Button   | GPIO0         | Enter configuration mode          |

*Note: Modify pin assignments in the code based on your specific hardware requirements.*
- For troubleshooting, monitor the serial log output via USB.

## MQTT Data Structure

The ESP32 publishes data in a structured JSON format to the configured MQTT topic. The template includes system metrics and placeholder sections for your custom data.

### Main Data Structure
```json
{
  // ESP32 processor metrics
  "processor": {
    "WiFiRSSI": 75,                    // WiFi signal strength (% - 0-100%)
    "IPAddress": "192.168.1.150",      // Current IP address
    "CPUTemperature": 32.0,            // ESP32 CPU temperature (°C - placeholder*)
    "SoftwareVersion": "0.0.1-TEMPLATE", // Firmware version
    "WDTRestartCount": 2               // Watchdog restart counter
  },
  
  // Your custom data will be added here
  // Replace PutOutputFormattingForMqttHere sections with your data fields
  "data": {
    // Example structure - customize for your application:
    // "temperature": 25.4,
    // "humidity": 60.2,
    // "status": "OK",
    // "sensor_id": 1
  },
  
  // System information (automatically included)
  "system": {
    "device_name": "ESP32-Template",   // Device identifier
    "uptime": 12345,                   // System uptime in seconds
    "heap_free": 234567,               // Available heap memory
    "timestamp": "2025-07-20T10:30:00Z" // ISO timestamp
  }
}
```

### Template Data Sections

#### 1. **Processor Metrics** (Always Included)
- WiFi signal strength and network information
- CPU temperature monitoring (placeholder implementation)
- Firmware version and system status
- Watchdog restart tracking for reliability monitoring

> **Note**: CPU temperature currently returns a placeholder value (25-45°C range). A full implementation would require enabling the ESP-IDF temperature sensor component in menuconfig.

#### 2. **Custom Data** (Your Implementation)
Replace the `PutOutputFormattingForMqttHere` placeholders with your specific data fields:
- Sensor readings (temperature, humidity, pressure, etc.)
- Device status and operational parameters  
- Measurement values and calculations
- Any application-specific information

#### 3. **System Information** (Automatically Included)
- Device identification and uptime tracking
- Memory usage monitoring
- Timestamp for data correlation
- System health indicators
  ## Template Usage Examples

### Example 1: Temperature Sensor Network

```c
// Replace input_data_t with your sensor data
typedef struct {
    float temperature_c;
    float humidity_percent;
    float pressure_hpa;
    int sensor_id;
} input_data_t;

// In PutInputCodeHere section:
esp_err_t read_input_data(input_data_t *data) {
    // I2C read from BME280 sensor
    data->temperature_c = bme280_read_temperature();
    data->humidity_percent = bme280_read_humidity();
    data->pressure_hpa = bme280_read_pressure();
    data->sensor_id = 1;
    return ESP_OK;
}

// In PutOutputFormattingForMqttHere section:
cJSON *sensor_json = cJSON_CreateObject();
cJSON_AddNumberToObject(sensor_json, "temperature_c", data->temperature_c);
cJSON_AddNumberToObject(sensor_json, "humidity_percent", data->humidity_percent);
cJSON_AddNumberToObject(sensor_json, "pressure_hpa", data->pressure_hpa);
cJSON_AddNumberToObject(sensor_json, "sensor_id", data->sensor_id);
```

### Example 2: Modbus RTU Device

```c
// Replace input_data_t with Modbus register data
typedef struct {
    uint16_t registers[10];
    float voltage;
    float current;
    uint16_t status_word;
} input_data_t;

// In PutInputCodeHere section:
esp_err_t read_input_data(input_data_t *data) {
    // Modbus RTU read holding registers
    modbus_read_holding_registers(1, 0, 10, data->registers);
    data->voltage = data->registers[0] / 100.0f;  // Scale factor
    data->current = data->registers[1] / 100.0f;
    data->status_word = data->registers[9];
    return ESP_OK;
}
```

### Example 3: GPS Tracker

```c
typedef struct {
    double latitude;
    double longitude;
    float altitude_m;
    float speed_kmh;
    uint8_t satellites;
    char timestamp[32];
} input_data_t;

// In PutInputCodeHere section:
esp_err_t read_input_data(input_data_t *data) {
    // Parse NMEA sentences from GPS module
    gps_parse_gga(&data->latitude, &data->longitude, &data->altitude_m);
    gps_parse_rmc(&data->speed_kmh, data->timestamp);
    data->satellites = gps_get_satellite_count();
    return ESP_OK;
}
```

## MQTT Integration Patterns

### Time Series Data (InfluxDB)
```json
{
  "measurement": "sensor_data",
  "tags": {
    "device": "ESP32-Template",
    "location": "room1"
  },
  "fields": {
    "temperature": 23.5,
    "humidity": 45.2
  },
  "timestamp": 1642234567890
}
```

### Home Assistant Discovery
```json
{
  "device": {
    "identifiers": ["esp32_template_001"],
    "name": "ESP32 Template Device",
    "model": "ESP32-Template",
    "manufacturer": "Custom"
  },
  "state_topic": "sensors/room1/data",
  "value_template": "{{ value_json.temperature }}"
}
```

## Advanced Features

### Captive Portal Customization
The template includes a web-based configuration portal. To customize:

1. **Edit HTML interface**: Modify `data/parameters.html`
2. **Add new parameters**: Update the configuration structure in main.c
3. **Custom styling**: Add CSS to the HTML file
4. **Additional pages**: Create new HTML files in the `data/` directory

### OTA Updates
The template supports Over-The-Air updates:

- **Firmware OTA**: Update code remotely via MQTT or HTTP
- **File System OTA**: Update web interface and configuration files
- **Secure updates**: Supports signed firmware for production use

### Multiple Data Sources
To handle multiple sensors or devices:

```c
typedef struct {
    input_data_t sensor1;
    input_data_t sensor2;
    input_data_t sensor3;
} multi_sensor_data_t;

// Read all sensors in sequence
esp_err_t read_all_sensors(multi_sensor_data_t *data) {
    read_sensor_1(&data->sensor1);
    read_sensor_2(&data->sensor2);
    read_sensor_3(&data->sensor3);
    return ESP_OK;
}
```

## Template Maintenance

### Keeping Up to Date
When updating the template for new projects:

1. **Preserve core functionality**: WiFi, MQTT, web portal
2. **Update placeholders**: Ensure markers are clear and complete
3. **Test thoroughly**: Verify all features work with minimal implementation
4. **Document changes**: Update README for any new features or requirements

### Creating Variants
For specialized use cases, consider creating template variants:

- **ESP32-Template-Sensor**: Pre-configured for I2C/SPI sensors
- **ESP32-Template-Modbus**: Pre-configured for RS485/Modbus RTU
- **ESP32-Template-LoRa**: Extended with LoRaWAN connectivity
- **ESP32-Template-Battery**: Optimized for battery-powered applications

## Migration from Original Project

If you're converting from the original JKBMS2ESPMQTT project:

1. **Backup your configuration**: Save WiFi and MQTT settings
2. **Note customizations**: Document any code changes you made
3. **Flash template**: Upload the template firmware
4. **Reconfigure**: Use captive portal to restore settings
5. **Implement your data source**: Replace template placeholders
6. **Test thoroughly**: Verify all functionality works as expected

## Performance Optimization

### Memory Management
- **Free heap monitoring**: Template includes heap usage reporting
- **JSON optimization**: Use streaming JSON for large payloads
- **Buffer management**: Adjust buffer sizes based on data requirements

### Power Consumption
For battery-powered applications:

```c
// Add deep sleep between readings
#include "esp_sleep.h"

void enter_deep_sleep(uint32_t sleep_time_ms) {
    esp_sleep_enable_timer_wakeup(sleep_time_ms * 1000);
    esp_deep_sleep_start();
}
```

### Network Optimization
- **MQTT QoS levels**: Choose appropriate quality of service
- **Connection pooling**: Reuse MQTT connections when possible
- **Compression**: Consider data compression for large payloads

## Security Considerations

### MQTT Security
- **TLS encryption**: Enable SSL/TLS for MQTT broker connections
- **Authentication**: Use username/password or certificate authentication
- **Topic permissions**: Restrict publish/subscribe permissions

### WiFi Security  
- **WPA3 support**: Use latest WiFi security protocols
- **Hidden networks**: Support for hidden SSID networks
- **Enterprise security**: WPA2-Enterprise support for corporate networks

### Configuration Security
- **Parameter encryption**: Encrypt sensitive configuration data
- **Access controls**: Restrict web interface access
- **Audit logging**: Log configuration changes

This template provides a comprehensive foundation for ESP32-based MQTT data collection projects while maintaining security and reliability best practices.
