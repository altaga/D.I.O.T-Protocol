# DIOT IoT Firmware Design Document

## Overview

The DIOT IoT Firmware is an embedded C++ application for ESP32-based M5Stack devices that collects motion sensor data and publishes it to an MQTT broker. The firmware manages WiFi connectivity, MQTT communication, and high-frequency sensor sampling with configurable intervals. It operates autonomously, aggregating accelerometer and gyroscope data before publishing formatted JSON messages to enable real-time IoT monitoring.

The system follows a simple event-driven architecture with three main operational phases:
1. **Initialization**: WiFi and MQTT connection establishment
2. **Sampling**: High-frequency sensor data collection (100ms intervals)
3. **Publishing**: Periodic aggregated data transmission (5-second intervals)

## Architecture

### System Components

```
┌─────────────────────────────────────────────────────────────┐
│                     ESP32 M5Stack Device                     │
│                                                               │
│  ┌──────────────┐      ┌──────────────┐                     │
│  │   WiFi       │      │    MQTT      │                     │
│  │  Manager     │─────▶│   Client     │                     │
│  └──────────────┘      └──────┬───────┘                     │
│                               │                              │
│  ┌──────────────┐             │                              │
│  │   Sensor     │             │                              │
│  │   Sampler    │─────────────┘                              │
│  └──────┬───────┘                                            │
│         │                                                     │
│  ┌──────▼───────┐      ┌──────────────┐                     │
│  │     IMU      │      │    Data      │                     │
│  │  Hardware    │─────▶│  Aggregator  │                     │
│  └──────────────┘      └──────────────┘                     │
│                                                               │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
                  ┌──────────────┐
                  │     MQTT     │
                  │    Broker    │
                  └──────────────┘
```

### Component Responsibilities

**WiFi Manager**
- Establishes and maintains WiFi connectivity
- Implements retry logic with exponential backoff
- Triggers device restart on persistent failures
- Logs connection status to serial output

**MQTT Client**
- Manages connection to MQTT broker
- Handles automatic reconnection on failures
- Publishes sensor data to configured topics
- Subscribes to command topics for remote control
- Logs all MQTT operations to serial output

**Sensor Sampler**
- Reads IMU data at 100ms intervals
- Aggregates accelerometer maximums per sampling window
- Captures latest gyroscope readings
- Handles sensor read failures gracefully

**Data Aggregator**
- Formats sensor data into JSON structure
- Maintains sampling window state
- Resets aggregated values after successful publish
- Ensures data consistency and validity

## Components and Interfaces

### Configuration Interface

All configuration is provided through compile-time constants defined in the firmware source:

```cpp
// WiFi Configuration
const char* WIFI_SSID = "network_name";
const char* WIFI_PASSWORD = "network_password";

// MQTT Configuration
const char* MQTT_BROKER = "broker.example.com";
const int MQTT_PORT = 1883;
const char* MQTT_USERNAME = "device_user";
const char* MQTT_PASSWORD = "device_pass";

// Device Configuration
const char* DEVICE_ADDRESS = "0x1234567890abcdef";
const char* SENSOR_NAME = "M5Stack IMU";
const char* SENSOR_DESCRIPTION = "Motion sensor device";
const char* SENSOR_TOPIC = "sensors/motion/data";

// Timing Configuration
const int SAMPLE_INTERVAL_MS = 100;
const int PUBLISH_INTERVAL_MS = 5000;
const int WIFI_RETRY_INTERVAL_MS = 500;
const int WIFI_MAX_RETRIES = 40;
const int MQTT_RETRY_INTERVAL_MS = 5000;
```

### WiFi Connection Interface

```cpp
// Initialize WiFi connection
void setupWiFi() {
  // Begin WiFi connection attempt
  // Retry every 500ms up to 40 times
  // Log connection status
  // Restart device on failure
}

// Check WiFi status
bool isWiFiConnected() {
  return WiFi.status() == WL_CONNECTED;
}
```

### MQTT Communication Interface

```cpp
// Initialize MQTT client
void setupMQTT() {
  // Configure MQTT broker connection
  // Set callback for incoming messages
}

// Maintain MQTT connection
void ensureMQTTConnection() {
  // Check connection status
  // Attempt reconnection if disconnected
  // Subscribe to command topics on connect
}

// Publish sensor data
bool publishSensorData(const char* jsonPayload) {
  // Publish to configured topic
  // Return success/failure status
  // Log result to serial
}
```

### Sensor Reading Interface

```cpp
// IMU sensor data structure
struct IMUData {
  float accel_x, accel_y, accel_z;  // m/s²
  float gyro_x, gyro_y, gyro_z;     // deg/s
};

// Read current sensor values
IMUData readIMU() {
  // Read accelerometer
  // Read gyroscope
  // Return combined data
}

// Aggregated sensor state
struct SensorState {
  float max_accel_x, max_accel_y, max_accel_z;
  float current_gyro_x, current_gyro_y, current_gyro_z;
};

// Update aggregated state with new reading
void updateSensorState(SensorState& state, const IMUData& reading) {
  // Update max accelerometer values
  // Update current gyroscope values
}

// Reset state after publish
void resetSensorState(SensorState& state) {
  // Reset max accelerometer values to 0
  // Keep gyroscope values (most recent)
}
```

### Data Formatting Interface

```cpp
// Format sensor data as JSON
String formatSensorJSON(const SensorState& state) {
  // Create JSON structure:
  // {
  //   "address": "0x...",
  //   "name": "sensor_name",
  //   "description": "sensor_desc",
  //   "data": {
  //     "accel": {"x": 0.00, "y": 0.00, "z": 0.00},
  //     "gyro": {"x": 0.00, "y": 0.00, "z": 0.00}
  //   }
  // }
  // Format floats to 2 decimal places
  // Return JSON string
}
```

## Data Models

### Sensor Reading Model

Raw sensor data captured from IMU hardware:

```cpp
struct IMUData {
  float accel_x;  // X-axis acceleration (m/s²)
  float accel_y;  // Y-axis acceleration (m/s²)
  float accel_z;  // Z-axis acceleration (m/s²)
  float gyro_x;   // X-axis angular velocity (deg/s)
  float gyro_y;   // Y-axis angular velocity (deg/s)
  float gyro_z;   // Z-axis angular velocity (deg/s)
};
```

### Aggregated Sensor State Model

Aggregated data maintained during sampling window:

```cpp
struct SensorState {
  // Maximum absolute accelerometer values during window
  float max_accel_x;
  float max_accel_y;
  float max_accel_z;
  
  // Most recent gyroscope readings
  float current_gyro_x;
  float current_gyro_y;
  float current_gyro_z;
  
  // Timing
  unsigned long last_sample_time;
  unsigned long last_publish_time;
};
```

### JSON Message Model

Published MQTT message structure:

```json
{
  "address": "string",      // Device identifier (Ethereum-style)
  "name": "string",         // Sensor name
  "description": "string",  // Device description
  "data": {
    "accel": {
      "x": 0.00,           // Max X acceleration (2 decimals)
      "y": 0.00,           // Max Y acceleration (2 decimals)
      "z": 0.00            // Max Z acceleration (2 decimals)
    },
    "gyro": {
      "x": 0.00,           // Current X angular velocity (2 decimals)
      "y": 0.00,           // Current Y angular velocity (2 decimals)
      "z": 0.00            // Current Z angular velocity (2 decimals)
    }
  }
}
```

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Acceptance Criteria Testing Prework


**Property Reflection:**

After reviewing all testable properties, several can be consolidated:
- Properties 5.1, 5.2, 5.3, 5.4 all test JSON structure and can be combined into one comprehensive property
- Properties 5.5 and 5.6 both test decimal formatting and can be combined
- Properties 4.3 and 4.4 both test data integrity in publishing and can be combined
- Property 5.7 (JSON round-trip) subsumes the structural validation in 5.1-5.4, so we'll keep the round-trip property as it's more comprehensive

The following properties provide unique validation value and will be included:

### Core Properties

**Property 1: Accelerometer maximum tracking**
*For any* sequence of accelerometer readings during a sampling window, the tracked maximum absolute value for each axis should be greater than or equal to all individual readings for that axis.
**Validates: Requirements 3.2**

**Property 2: Gyroscope latest value retention**
*For any* sequence of gyroscope readings, the stored value for each axis should equal the most recent reading received.
**Validates: Requirements 3.3**

**Property 3: JSON message completeness**
*For any* sensor state, the formatted JSON message should contain all required fields: "address", "name", "description", and nested "data" object with "accel" and "gyro" sub-objects, each containing "x", "y", "z" fields.
**Validates: Requirements 4.2, 5.1, 5.2, 5.3, 5.4**

**Property 4: Decimal precision formatting**
*For any* sensor values (accelerometer or gyroscope), the formatted JSON should represent each numeric value with exactly 2 decimal places.
**Validates: Requirements 5.5, 5.6**

**Property 5: JSON serialization round-trip**
*For any* valid sensor state, formatting the data to JSON and then parsing it should produce a valid object with all original data preserved (within floating-point precision limits).
**Validates: Requirements 5.7**

**Property 6: Published data integrity**
*For any* sampling window, the published accelerometer values should match the tracked maximum absolute values, and the published gyroscope values should match the most recent readings.
**Validates: Requirements 4.3, 4.4**

**Property 7: State reset after publish**
*For any* sensor state, after a successful publish operation, the maximum accelerometer values should be reset to zero while gyroscope values remain unchanged.
**Validates: Requirements 4.5**

## Error Handling

### WiFi Connection Failures

**Retry Strategy**
- Retry interval: 500ms between attempts
- Maximum retries: 40 attempts (20 seconds total)
- Failure action: Device restart to recover from persistent network issues

**Error Logging**
- Log each connection attempt with status
- Log final failure before restart
- Include WiFi status codes in logs

### MQTT Connection Failures

**Retry Strategy**
- Retry interval: 5 seconds between attempts
- Infinite retries (no maximum)
- Prerequisite: WiFi must be connected before MQTT retry

**Error Logging**
- Log each connection attempt with status
- Log MQTT error codes for debugging
- Log successful reconnection events

### Sensor Read Failures

**Resilience Strategy**
- Continue operation if individual sensor reads fail
- Use last known good values for failed reads
- Log sensor read errors to serial output

**Data Integrity**
- Don't publish data if all sensor reads fail
- Maintain sampling window timing even with failures
- Reset state appropriately after failures

### MQTT Publish Failures

**Retry Strategy**
- Log failure with attempted payload
- Continue sampling and aggregating data
- Retry publish on next interval (5 seconds)

**Data Loss Prevention**
- Don't reset aggregated state on publish failure
- Continue accumulating maximums across failed publishes
- Only reset state after confirmed successful publish

### Serial Logging Failures

**Graceful Degradation**
- Serial logging failures should not crash the device
- Continue core operations (sampling, publishing) even if logging fails
- No retry logic for serial output (fire-and-forget)

## Testing Strategy

### Unit Testing Approach

Given the embedded nature of this firmware and the Arduino/ESP32 platform, traditional unit testing is challenging. However, we can implement the following testing strategies:

**Testable Components**
- Data formatting functions (JSON generation)
- Aggregation logic (maximum tracking, latest value storage)
- State management (reset logic)
- Timing calculations

**Testing Framework**
- Use Arduino unit testing framework (AUnit or ArduinoUnit)
- Test on actual hardware or ESP32 simulator
- Focus on logic that doesn't require hardware I/O

**Example Unit Tests**
- Test JSON formatting with known sensor values
- Test maximum value tracking with sample data sequences
- Test state reset behavior
- Test decimal precision formatting
- Test JSON parsing of generated output

### Property-Based Testing Approach

**Testing Framework**
- For embedded C++, we'll use a custom property testing approach or adapt a lightweight framework
- Alternative: Test the data formatting logic in a separate C++ project with a full PBT library (e.g., RapidCheck)
- Minimum iterations: 100 per property test

**Property Test Implementation**
- Each property test must be tagged with: `// Feature: diot-iot-firmware, Property X: [property description]`
- Generate random sensor values within realistic ranges
- Verify properties hold across all generated inputs
- Test edge cases (zero values, maximum values, negative values)

**Property Test Coverage**
1. **Property 1 Test**: Generate random sequences of accelerometer readings, verify maximum tracking
2. **Property 2 Test**: Generate random sequences of gyroscope readings, verify latest value retention
3. **Property 3 Test**: Generate random sensor states, verify JSON contains all required fields
4. **Property 4 Test**: Generate random float values, verify 2 decimal place formatting
5. **Property 5 Test**: Generate random sensor states, verify JSON round-trip preserves data
6. **Property 6 Test**: Generate random sampling windows, verify published data matches aggregated state
7. **Property 7 Test**: Generate random sensor states, verify state reset behavior after publish

**Testing Constraints**
- Accelerometer values: -20.0 to +20.0 m/s² (realistic range for motion)
- Gyroscope values: -250.0 to +250.0 deg/s (realistic range for rotation)
- Test with edge cases: 0.0, very small values (0.01), very large values
- Test with negative and positive values for all axes

### Integration Testing

**Hardware Integration Tests**
- Verify WiFi connection on actual network
- Verify MQTT connection to test broker
- Verify sensor readings from actual IMU hardware
- Verify end-to-end data flow from sensor to MQTT broker

**Test Scenarios**
1. Cold start: Device powers on and establishes all connections
2. Network interruption: WiFi drops and recovers
3. MQTT broker restart: Connection lost and re-established
4. Continuous operation: Device runs for extended period (hours)
5. Sensor data validation: Published data matches expected motion

**Manual Testing**
- Monitor serial output for correct logging
- Verify MQTT messages received by broker
- Validate JSON structure and values
- Test with different motion patterns (stationary, gentle movement, vigorous shaking)

## Implementation Notes

### Platform-Specific Considerations

**ESP32 M5Stack**
- Use M5Stack library for hardware abstraction
- Use WiFi.h for network connectivity
- Use PubSubClient library for MQTT
- Use M5Stack IMU library for sensor access

**Memory Management**
- Minimize dynamic memory allocation
- Use stack allocation for sensor data structures
- Reuse buffers for JSON formatting
- Monitor heap fragmentation during long-term operation

**Timing Precision**
- Use millis() for timing (not delay())
- Account for timer overflow (every ~50 days)
- Non-blocking implementation for all operations
- Maintain sampling accuracy despite MQTT latency

### Performance Considerations

**Sampling Rate**
- 100ms sampling interval = 10 Hz
- Sufficient for human motion detection
- Low enough to avoid overwhelming the processor

**Publish Rate**
- 5-second publish interval = 0.2 Hz
- Balances data freshness with network efficiency
- 50 samples aggregated per publish

**Network Efficiency**
- JSON payload size: ~200-300 bytes
- MQTT QoS 0 (fire-and-forget) for performance
- No message retention on broker

### Security Considerations

**Credential Management**
- WiFi and MQTT credentials in compile-time constants
- No credential storage in EEPROM (reduces attack surface)
- Credentials must be updated via firmware reflash

**Network Security**
- MQTT over TCP (port 1883) - unencrypted
- For production: Consider MQTT over TLS (port 8883)
- Device authentication via MQTT username/password

**Physical Security**
- Device has no user interface for configuration
- Physical access required for firmware updates
- Serial output may expose sensitive information (disable in production)

### Deployment Considerations

**Firmware Updates**
- Over-the-air (OTA) updates not implemented in initial version
- Updates require USB connection and Arduino IDE
- Version tracking via compile-time constant

**Configuration Management**
- Each device requires custom firmware build with unique credentials
- Consider environment-specific build configurations
- Document configuration parameters for each deployment

**Monitoring and Debugging**
- Serial output provides real-time diagnostics
- MQTT broker logs show device connectivity
- Consider adding health check/heartbeat messages

## Future Enhancements

**Potential Improvements**
1. Over-the-air (OTA) firmware updates
2. Runtime configuration via MQTT commands
3. Local data buffering for offline operation
4. MQTT over TLS for encrypted communication
5. Additional sensor support (temperature, humidity)
6. Configurable sampling and publish intervals via MQTT
7. Device health metrics (uptime, memory usage, WiFi signal strength)
8. Multiple MQTT topic support for different data types
9. Data compression for reduced bandwidth
10. Local anomaly detection before publishing

