# Requirements Document

## Introduction

The DIOT Mobile Sensor Device is a React Native mobile application that transforms smartphones into IoT sensor devices. The application continuously collects sensor data from the device's hardware sensors (accelerometer, gyroscope, magnetometer, and device motion) and transmits this data to a remote API endpoint at regular intervals. The system provides real-time visualization of sensor readings on the device screen and operates as a personal IoT data collection node within the DIOT ecosystem.

## Glossary

- **DIOT System**: The Decentralized Internet of Things system that manages IoT devices and sensor data
- **Sensor Device**: A mobile device running the DIOT Mobile Sensor Device application
- **Sensor Data**: Numerical readings from hardware sensors including accelerometer, gyroscope, magnetometer, and device motion
- **API Endpoint**: The remote server URL that receives sensor data transmissions
- **Device Address**: A unique hexadecimal identifier assigned to each Sensor Device
- **Gravity Estimation**: A calculated value derived from accelerometer data using a low-pass filter to isolate gravitational acceleration
- **Update Interval**: The time period between consecutive sensor readings
- **Transmission Interval**: The time period between consecutive data transmissions to the API Endpoint

## Requirements

### Requirement 1

**User Story:** As a device operator, I want the application to automatically detect available sensors on my device, so that I can utilize all compatible hardware capabilities without manual configuration.

#### Acceptance Criteria

1. WHEN the application starts THEN the DIOT System SHALL query the device for accelerometer availability
2. WHEN the application starts THEN the DIOT System SHALL query the device for gyroscope availability
3. WHEN the application starts THEN the DIOT System SHALL query the device for magnetometer availability
4. WHEN the application starts THEN the DIOT System SHALL query the device for device motion sensor availability
5. WHEN no sensors are available THEN the DIOT System SHALL display a notification message indicating no supported sensors were found

### Requirement 2

**User Story:** As a device operator, I want the application to continuously collect sensor data, so that I can monitor real-time device orientation and movement.

#### Acceptance Criteria

1. WHEN an accelerometer is available THEN the DIOT System SHALL collect three-axis acceleration data at 500 millisecond intervals
2. WHEN a gyroscope is available THEN the DIOT System SHALL collect three-axis rotation rate data at 500 millisecond intervals
3. WHEN a magnetometer is available THEN the DIOT System SHALL collect three-axis magnetic field data at 500 millisecond intervals
4. WHEN a device motion sensor is available THEN the DIOT System SHALL collect acceleration including gravity and rotation data at 500 millisecond intervals
5. WHEN accelerometer data is collected THEN the DIOT System SHALL apply a low-pass filter with alpha value 0.8 to estimate gravity components

### Requirement 3

**User Story:** As a device operator, I want to see real-time sensor readings on my screen, so that I can verify the device is functioning correctly and monitor current sensor values.

#### Acceptance Criteria

1. WHEN accelerometer data is available THEN the DIOT System SHALL display the X, Y, and Z acceleration values formatted to three decimal places
2. WHEN gyroscope data is available THEN the DIOT System SHALL display the X, Y, and Z rotation rate values formatted to three decimal places
3. WHEN magnetometer data is available THEN the DIOT System SHALL display the X, Y, and Z magnetic field values formatted to three decimal places
4. WHEN device motion data is available THEN the DIOT System SHALL display the acceleration including gravity values formatted to three decimal places
5. WHEN device motion data is available THEN the DIOT System SHALL display the alpha, beta, and gamma rotation values formatted to three decimal places
6. WHEN gravity estimation is calculated THEN the DIOT System SHALL display the estimated gravity X, Y, and Z values formatted to three decimal places

### Requirement 4

**User Story:** As a device operator, I want the application to display device identification information, so that I can verify which device is transmitting data.

#### Acceptance Criteria

1. WHEN the application displays device information THEN the DIOT System SHALL show the Device Address
2. WHEN the application displays device information THEN the DIOT System SHALL show the device name
3. WHEN the application displays device information THEN the DIOT System SHALL show the device description

### Requirement 5

**User Story:** As a system administrator, I want sensor data transmitted to the API endpoint at regular intervals, so that the backend system can process and store IoT device readings.

#### Acceptance Criteria

1. WHEN the application starts THEN the DIOT System SHALL transmit sensor data immediately to the API Endpoint
2. WHEN the application is running THEN the DIOT System SHALL transmit sensor data to the API Endpoint every 10 seconds
3. WHEN transmitting data THEN the DIOT System SHALL include the Device Address in the payload
4. WHEN transmitting data THEN the DIOT System SHALL include the device name in the payload
5. WHEN transmitting data THEN the DIOT System SHALL include the device description in the payload
6. WHEN transmitting data THEN the DIOT System SHALL include a timestamp in milliseconds since epoch in the payload
7. WHEN transmitting data THEN the DIOT System SHALL include all available sensor readings in the payload
8. WHEN transmitting data THEN the DIOT System SHALL use HTTP POST method with JSON content type
9. WHEN a sensor is unavailable THEN the DIOT System SHALL include null for that sensor's data in the transmission payload

### Requirement 6

**User Story:** As a device operator, I want the application to handle network errors gracefully, so that temporary connectivity issues do not crash the application.

#### Acceptance Criteria

1. WHEN a data transmission fails THEN the DIOT System SHALL display an error notification to the user
2. WHEN a data transmission fails THEN the DIOT System SHALL continue collecting sensor data without interruption
3. WHEN a data transmission fails THEN the DIOT System SHALL attempt the next scheduled transmission at the normal interval

### Requirement 7

**User Story:** As a device operator, I want the application to maintain accurate sensor readings during device lifecycle events, so that data collection remains consistent.

#### Acceptance Criteria

1. WHEN the application is backgrounded THEN the DIOT System SHALL properly clean up sensor listeners
2. WHEN the application returns to foreground THEN the DIOT System SHALL reinitialize sensor listeners
3. WHEN sensor listeners are removed THEN the DIOT System SHALL prevent memory leaks by unsubscribing all active listeners

### Requirement 8

**User Story:** As a developer, I want sensor data structured in a consistent format, so that the backend API can reliably parse and process incoming data.

#### Acceptance Criteria

1. WHEN sensor data is transmitted THEN the DIOT System SHALL format accelerometer data as an object with x, y, and z numeric properties
2. WHEN sensor data is transmitted THEN the DIOT System SHALL format gyroscope data as an object with x, y, and z numeric properties
3. WHEN sensor data is transmitted THEN the DIOT System SHALL format magnetometer data as an object with x, y, and z numeric properties
4. WHEN sensor data is transmitted THEN the DIOT System SHALL format gravity estimation as an object with x, y, and z numeric properties
5. WHEN sensor data is transmitted THEN the DIOT System SHALL format device motion data as an object containing gravity and rotation sub-objects
6. WHEN sensor data is transmitted THEN the DIOT System SHALL format rotation data as an object with alpha, beta, and gamma numeric properties
