# Implementation Plan

## Status: Core functionality implemented, testing and refinement needed

- [x] 1. Set up project structure and core interfaces
  - ESP32 M5Stack project structure created
  - WiFi and MQTT libraries integrated
  - IMU sensor library configured
  - _Requirements: 1.1, 2.1, 3.1_

- [x] 2. Implement WiFi connection management
  - WiFi connection with retry logic implemented
  - Device restart on persistent failure (40 attempts)
  - IP address logging to serial output
  - _Requirements: 1.1, 1.2, 1.3, 1.4_

- [x] 3. Implement MQTT client functionality
  - MQTT broker connection with credentials
  - Automatic reconnection on connection loss
  - Topic subscription for commands
  - WiFi prerequisite check before MQTT reconnect
  - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5_

- [x] 4. Implement sensor sampling logic
  - 100ms sampling interval for IMU data
  - Maximum absolute value tracking for accelerometer
  - Latest value storage for gyroscope
  - Graceful handling of sensor read failures
  - _Requirements: 3.1, 3.2, 3.3, 3.4_


- [x] 5. Implement JSON data formatting
  - JSON structure with address, name, description fields
  - Nested data object with accel and gyro sub-objects
  - 2 decimal place formatting for all numeric values
  - Valid JSON serialization
  - _Requirements: 4.2, 5.1, 5.2, 5.3, 5.4, 5.5, 5.6, 5.7_

- [x] 6. Implement periodic data publishing
  - 5-second publish interval
  - MQTT publish to configured topic
  - Success/failure logging to serial
  - Accelerometer max reset after successful publish
  - Gyroscope values persist across publishes
  - _Requirements: 4.1, 4.3, 4.4, 4.5, 4.6, 4.7_

- [x] 7. Implement serial logging
  - WiFi connection status logging
  - MQTT connection status and error codes
  - Publish success/failure with payload
  - Received message logging
  - _Requirements: 6.1, 6.2, 6.3, 6.4_

- [ ] 8. Refine configuration management
- [ ] 8.1 Consolidate configuration constants
  - Merge code.ino and code-arduino.ino into single implementation
  - Ensure all configuration follows design document naming
  - Add missing configuration constants (WIFI_RETRY_INTERVAL_MS, etc.)
  - _Requirements: 7.1, 7.2, 7.3, 7.4, 7.5_


- [ ] 9. Create property-based testing framework
- [ ] 9.1 Set up testing environment
  - Choose and configure property-based testing framework (RapidCheck or custom)
  - Create test harness for data formatting functions
  - Set up test execution with minimum 100 iterations per property
  - _Requirements: All testing requirements_

- [ ]* 9.2 Write property test for accelerometer maximum tracking
  - **Property 1: Accelerometer maximum tracking**
  - **Validates: Requirements 3.2**
  - Generate random sequences of accelerometer readings
  - Verify tracked maximum >= all individual readings for each axis
  - Test with values in range -20.0 to +20.0 m/s²

- [ ]* 9.3 Write property test for gyroscope latest value retention
  - **Property 2: Gyroscope latest value retention**
  - **Validates: Requirements 3.3**
  - Generate random sequences of gyroscope readings
  - Verify stored value equals most recent reading for each axis
  - Test with values in range -250.0 to +250.0 deg/s

- [ ]* 9.4 Write property test for JSON message completeness
  - **Property 3: JSON message completeness**
  - **Validates: Requirements 4.2, 5.1, 5.2, 5.3, 5.4**
  - Generate random sensor states
  - Verify JSON contains all required fields (address, name, description, data.accel, data.gyro)
  - Verify each accel/gyro object contains x, y, z fields


- [ ]* 9.5 Write property test for decimal precision formatting
  - **Property 4: Decimal precision formatting**
  - **Validates: Requirements 5.5, 5.6**
  - Generate random float values for sensor data
  - Verify JSON representation has exactly 2 decimal places
  - Test with edge cases (0.0, 0.01, very large values)

- [ ]* 9.6 Write property test for JSON serialization round-trip
  - **Property 5: JSON serialization round-trip**
  - **Validates: Requirements 5.7**
  - Generate random sensor states
  - Format to JSON, parse back, verify data preserved
  - Account for floating-point precision limits

- [ ]* 9.7 Write property test for published data integrity
  - **Property 6: Published data integrity**
  - **Validates: Requirements 4.3, 4.4**
  - Generate random sampling windows
  - Verify published accel values match tracked maximums
  - Verify published gyro values match most recent readings

- [ ]* 9.8 Write property test for state reset after publish
  - **Property 7: State reset after publish**
  - **Validates: Requirements 4.5**
  - Generate random sensor states
  - Simulate successful publish
  - Verify accel max values reset to zero
  - Verify gyro values remain unchanged


- [ ]* 10. Create unit tests for core functionality
- [ ]* 10.1 Write unit tests for WiFi connection logic
  - Test retry mechanism with mock WiFi
  - Test device restart trigger after 40 failed attempts
  - Test IP address logging
  - _Requirements: 1.1, 1.2, 1.3, 1.4_

- [ ]* 10.2 Write unit tests for MQTT connection logic
  - Test reconnection with 5-second intervals
  - Test topic subscription on successful connect
  - Test WiFi prerequisite check
  - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5_

- [ ]* 10.3 Write unit tests for sensor aggregation
  - Test maximum value tracking with known sequences
  - Test latest value storage with known sequences
  - Test state reset behavior
  - _Requirements: 3.2, 3.3, 4.5_

- [ ]* 10.4 Write unit tests for JSON formatting
  - Test JSON structure with known sensor values
  - Test decimal precision formatting
  - Test field presence and nesting
  - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5, 5.6_

- [ ] 11. Checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.


- [ ] 12. Enhance error handling and resilience
- [ ] 12.1 Improve WiFi error handling
  - Add detailed WiFi status code logging
  - Implement exponential backoff for retries (currently fixed 500ms)
  - Add WiFi signal strength monitoring
  - _Requirements: 1.1, 1.2, 1.3_

- [ ] 12.2 Improve MQTT error handling
  - Add detailed MQTT error code interpretation
  - Implement connection state tracking
  - Add publish retry logic for failed messages
  - Prevent state reset on publish failure
  - _Requirements: 2.2, 2.4, 4.7_

- [ ] 12.3 Improve sensor read error handling
  - Add sensor initialization validation
  - Log sensor read failures without crashing
  - Use last known good values on read failure
  - _Requirements: 3.4_

- [ ] 12.4 Add serial logging safety
  - Wrap serial operations in error handlers
  - Ensure core operations continue if logging fails
  - _Requirements: 6.1, 6.2, 6.3, 6.4_

- [ ] 13. Add documentation and comments
- [ ] 13.1 Add inline code documentation
  - Document configuration constants
  - Document function purposes and parameters
  - Add section comments for major code blocks
  - _Requirements: All_


- [ ] 13.2 Create deployment documentation
  - Document configuration steps for new deployments
  - Document WiFi and MQTT credential setup
  - Document device identifier assignment process
  - Document upload and testing procedures
  - _Requirements: 7.1, 7.2, 7.3, 7.4_

- [ ] 14. Final integration testing and validation
- [ ] 14.1 Perform hardware integration tests
  - Test cold start on actual M5Stack device
  - Test network interruption recovery
  - Test MQTT broker restart recovery
  - Test continuous operation for extended period (1+ hours)
  - _Requirements: All_

- [ ] 14.2 Validate sensor data accuracy
  - Compare published data with expected motion patterns
  - Verify JSON structure matches design specification
  - Verify data values are within realistic ranges
  - Test with various motion patterns (stationary, gentle, vigorous)
  - _Requirements: 3.1, 3.2, 3.3, 4.1, 4.2, 4.3, 4.4_

- [ ] 15. Final Checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.
