# Requirements Document

## Introduction

The DIOT IoT Firmware is an embedded system that runs on ESP32-based M5Stack devices equipped with IMU (Inertial Measurement Unit) sensors. The firmware collects motion data from accelerometers and gyroscopes, processes this data in real-time, and publishes it to an MQTT broker for consumption by other system components. The device operates autonomously, managing WiFi connectivity, MQTT communication, and sensor sampling with configurable intervals.

## Glossary

- **IMU Device**: The ESP32-based M5Stack hardware device equipped with accelerometer and gyroscope sensors
- **MQTT Broker**: A message broker server that receives and distributes sensor data messages using the MQTT protocol
- **Accelerometer Data**: Three-axis (x, y, z) measurements of linear acceleration in meters per second squared
- **Gyroscope Data**: Three-axis (x, y, z) measurements of angular velocity in degrees per second
- **Sampling Window**: The time period during which the system collects and aggregates sensor readings before publishing
- **Device Identifier**: A unique Ethereum-style address that identifies the specific IMU Device
- **Sensor Topic**: The MQTT topic path where the IMU Device publishes its sensor data messages

## Requirements

### Requirement 1

**User Story:** As a system operator, I want the IMU Device to connect to WiFi automatically on startup, so that the device can communicate with the MQTT Broker without manual intervention.

#### Acceptance Criteria

1. WHEN the IMU Device powers on, THE IMU Device SHALL attempt to connect to the configured WiFi network
2. WHILE the IMU Device is attempting WiFi connection, THE IMU Device SHALL retry the connection every 500 milliseconds
3. IF the IMU Device fails to connect to WiFi after 40 attempts, THEN THE IMU Device SHALL restart itself
4. WHEN the IMU Device successfully connects to WiFi, THE IMU Device SHALL log the assigned IP address to the serial output

### Requirement 2

**User Story:** As a system operator, I want the IMU Device to maintain a persistent MQTT connection, so that sensor data can be reliably transmitted to the MQTT Broker.

#### Acceptance Criteria

1. WHEN the IMU Device establishes WiFi connectivity, THE IMU Device SHALL connect to the MQTT Broker using the configured credentials
2. WHILE the IMU Device is not connected to the MQTT Broker, THE IMU Device SHALL attempt reconnection every 5 seconds
3. WHEN the IMU Device successfully connects to the MQTT Broker, THE IMU Device SHALL subscribe to the command topic for receiving instructions
4. IF the IMU Device loses connection to the MQTT Broker during operation, THEN THE IMU Device SHALL automatically attempt to reconnect
5. IF the IMU Device loses WiFi connectivity during operation, THEN THE IMU Device SHALL re-establish WiFi connection before reconnecting to the MQTT Broker

### Requirement 3

**User Story:** As a data consumer, I want the IMU Device to sample sensor data at high frequency, so that rapid motion changes are captured accurately.

#### Acceptance Criteria

1. WHEN the IMU Device is operational, THE IMU Device SHALL sample the accelerometer and gyroscope every 100 milliseconds
2. WHEN the IMU Device samples the accelerometer, THE IMU Device SHALL track the maximum absolute value for each axis during the sampling window
3. WHEN the IMU Device samples the gyroscope, THE IMU Device SHALL store the most recent reading for each axis
4. WHILE sampling sensor data, THE IMU Device SHALL continue processing even if individual sensor reads fail

### Requirement 4

**User Story:** As a data consumer, I want the IMU Device to publish aggregated sensor data periodically, so that I receive regular updates without overwhelming the network.

#### Acceptance Criteria

1. WHEN 5 seconds have elapsed since the last publish, THE IMU Device SHALL publish a sensor data message to the Sensor Topic
2. WHEN publishing sensor data, THE IMU Device SHALL format the message as valid JSON containing the Device Identifier, sensor name, description, and sensor readings
3. WHEN publishing accelerometer data, THE IMU Device SHALL include the maximum absolute values recorded for x, y, and z axes during the sampling window
4. WHEN publishing gyroscope data, THE IMU Device SHALL include the most recent readings for x, y, and z axes
5. WHEN a sensor data message is successfully published, THE IMU Device SHALL reset the accelerometer maximum values to zero for the next sampling window
6. WHEN a sensor data message is successfully published, THE IMU Device SHALL log the published JSON payload to the serial output
7. IF a sensor data message fails to publish, THEN THE IMU Device SHALL log the failure and the attempted payload to the serial output

### Requirement 5

**User Story:** As a system integrator, I want the IMU Device to format sensor data in a standardized JSON structure, so that downstream systems can parse and process the data consistently.

#### Acceptance Criteria

1. WHEN the IMU Device creates a sensor data message, THE IMU Device SHALL include a root-level "address" field containing the Device Identifier
2. WHEN the IMU Device creates a sensor data message, THE IMU Device SHALL include a root-level "name" field containing the sensor name
3. WHEN the IMU Device creates a sensor data message, THE IMU Device SHALL include a root-level "description" field containing the device description
4. WHEN the IMU Device creates a sensor data message, THE IMU Device SHALL include a nested "data" object containing "accel" and "gyro" objects
5. WHEN the IMU Device formats accelerometer data, THE IMU Device SHALL include "x", "y", and "z" fields within the "accel" object with values formatted to 2 decimal places
6. WHEN the IMU Device formats gyroscope data, THE IMU Device SHALL include "x", "y", and "z" fields within the "gyro" object with values formatted to 2 decimal places
7. WHEN the IMU Device serializes the sensor data message, THE IMU Device SHALL produce valid JSON that can be parsed by standard JSON parsers

### Requirement 6

**User Story:** As a developer, I want the IMU Device to provide serial logging of its operations, so that I can monitor device behavior and troubleshoot issues during development and deployment.

#### Acceptance Criteria

1. WHEN the IMU Device performs WiFi connection attempts, THE IMU Device SHALL log connection status to the serial output
2. WHEN the IMU Device performs MQTT connection attempts, THE IMU Device SHALL log connection status and error codes to the serial output
3. WHEN the IMU Device publishes sensor data, THE IMU Device SHALL log the publish result and payload to the serial output
4. WHEN the IMU Device receives messages on subscribed topics, THE IMU Device SHALL log the topic and message content to the serial output

### Requirement 7

**User Story:** As a system operator, I want the IMU Device to handle configuration through compile-time constants, so that device behavior can be customized for different deployment environments.

#### Acceptance Criteria

1. THE IMU Device SHALL support configuration of WiFi SSID and password through compile-time constants
2. THE IMU Device SHALL support configuration of MQTT Broker address, username, and password through compile-time constants
3. THE IMU Device SHALL support configuration of Device Identifier, sensor name, and description through compile-time constants
4. THE IMU Device SHALL support configuration of the Sensor Topic path through compile-time constants
5. THE IMU Device SHALL support configuration of sampling interval and publish interval through compile-time constants
