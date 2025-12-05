# Implementation Plan

- [x] 1. Set up project structure and sensor integration
  - Create React Native Expo application with sensor support
  - Install expo-sensors package and configure dependencies
  - Set up basic UI layout with SafeAreaView and ScrollView
  - _Requirements: 1.1, 1.2, 1.3, 1.4_

- [x] 2. Implement sensor availability detection
  - Query device for accelerometer, gyroscope, magnetometer, and device motion availability
  - Store availability state for conditional rendering
  - Display notification when no sensors are available
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5_

- [x] 3. Implement sensor data collection
  - Subscribe to accelerometer updates at 500ms intervals
  - Subscribe to gyroscope updates at 500ms intervals
  - Subscribe to magnetometer updates at 500ms intervals
  - Subscribe to device motion updates at 500ms intervals
  - Apply low-pass filter (alpha=0.8) for gravity estimation from accelerometer
  - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5_

- [x] 4. Implement real-time sensor display
  - Display accelerometer X, Y, Z values formatted to 3 decimal places
  - Display gyroscope X, Y, Z values formatted to 3 decimal places
  - Display magnetometer X, Y, Z values formatted to 3 decimal places
  - Display device motion acceleration including gravity formatted to 3 decimal places
  - Display device motion rotation (alpha, beta, gamma) formatted to 3 decimal places
  - Display estimated gravity X, Y, Z values formatted to 3 decimal places
  - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5, 3.6_

- [x] 5. Implement device information display
  - Display device address (hexadecimal identifier)
  - Display device name
  - Display device description
  - _Requirements: 4.1, 4.2, 4.3_

- [x] 6. Implement data transmission service
  - Transmit sensor data immediately on application start
  - Set up 10-second interval for periodic transmission
  - Build transmission payload with device info, timestamp, and sensor data
  - Use HTTP POST with JSON content type
  - Include null values for unavailable sensors
  - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5, 5.6, 5.7, 5.8, 5.9_

- [x] 7. Implement error handling for network failures
  - Display error notification on transmission failure (ToastAndroid)
  - Continue sensor collection despite transmission failures
  - Maintain normal transmission interval after failures
  - _Requirements: 6.1, 6.2, 6.3_

- [x] 8. Implement lifecycle management
  - Clean up sensor listeners on component unmount
  - Properly unsubscribe all active listeners to prevent memory leaks
  - _Requirements: 7.1, 7.3_

- [ ] 9. Refactor for proper lifecycle handling across app states
- [ ] 9.1 Add AppState listener for background/foreground detection
  - Import AppState from react-native
  - Subscribe to AppState changes in useEffect
  - Detect when app moves to background or foreground
  - _Requirements: 7.1, 7.2_

- [ ] 9.2 Implement sensor cleanup on app backgrounding
  - Unsubscribe all sensor listeners when app goes to background
  - Clear interval for data transmission
  - Prevent memory leaks during background state
  - _Requirements: 7.1, 7.3_

- [ ] 9.3 Implement sensor reinitialization on app foregrounding
  - Reinitialize sensor subscriptions when app returns to foreground
  - Restart data transmission interval
  - Restore normal operation after backgrounding
  - _Requirements: 7.2_

- [ ] 10. Create data processing utilities module
- [ ] 10.1 Extract gravity estimation function to utils
  - Move applyLowPassFilter to src/core/utils.js
  - Add JSDoc documentation for the function
  - Export for reuse and testing
  - _Requirements: 2.5_

- [ ] 10.2 Create sensor data formatter utility
  - Create formatSensorValue function for 3 decimal places
  - Create buildTransmissionPayload function
  - Add validation for sensor data structure
  - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5, 3.6, 8.1, 8.2, 8.3, 8.4, 8.5, 8.6_

- [ ] 11. Improve error handling and user notifications
- [ ] 11.1 Create cross-platform notification utility
  - Replace ToastAndroid with cross-platform solution (Alert or expo-notifications)
  - Ensure error notifications work on both iOS and Android
  - _Requirements: 6.1_

- [ ] 11.2 Add error logging for debugging
  - Log transmission errors with timestamps
  - Log sensor initialization failures
  - Add console warnings for missing sensors
  - _Requirements: 6.1, 6.2_

- [ ] 12. Set up testing infrastructure
- [ ] 12.1 Install testing dependencies
  - Install Jest and React Native Testing Library
  - Install fast-check for property-based testing
  - Configure Jest for React Native environment
  - Create test setup files

- [ ]* 12.2 Write property test for gravity estimation
  - **Property 3: Gravity estimation low-pass filter**
  - **Validates: Requirements 2.5**
  - Generate random accelerometer and gravity values
  - Verify calculation follows formula: gravity[n] = 0.8 * gravity[n-1] + 0.2 * acceleration[n]
  - Test with edge cases (zero values, extreme values)
  - Run minimum 100 iterations

- [ ]* 12.3 Write property test for decimal formatting
  - **Property 4: Sensor value decimal formatting**
  - **Validates: Requirements 3.1, 3.2, 3.3, 3.4, 3.5, 3.6**
  - Generate random floating-point numbers
  - Verify all formatted values have exactly 3 decimal places
  - Test with various magnitudes and signs
  - Run minimum 100 iterations

- [ ]* 12.4 Write property test for transmission payload completeness
  - **Property 7: Transmission payload completeness**
  - **Validates: Requirements 5.3, 5.4, 5.5, 5.6, 5.7, 5.9**
  - Generate random device info and sensor data combinations
  - Verify all required fields present (address, name, description, timestamp, sensors)
  - Verify null handling for unavailable sensors
  - Run minimum 100 iterations

- [ ]* 12.5 Write property test for payload structure validation
  - **Property 13: Transmission payload structure validation**
  - **Validates: Requirements 8.1, 8.2, 8.3, 8.4, 8.5, 8.6**
  - Generate random sensor data
  - Verify vector objects have x, y, z numeric properties
  - Verify rotation objects have alpha, beta, gamma numeric properties
  - Verify no extra properties included
  - Run minimum 100 iterations

- [ ]* 12.6 Write unit tests for data processing utilities
  - Test gravity estimation with specific known values
  - Test decimal formatting edge cases (very small, very large, negative)
  - Test payload construction with complete and partial sensor data
  - Test null handling for unavailable sensors

- [ ]* 12.7 Write unit tests for sensor manager
  - Mock sensor availability detection
  - Test subscription lifecycle management
  - Verify cleanup on unmount

- [ ]* 12.8 Write unit tests for transmission service
  - Mock fetch API calls
  - Test HTTP request format (POST, JSON content type)
  - Test error handling for network failures
  - Test interval timing with fake timers

- [ ]* 12.9 Write component tests for UI
  - Test sensor data display rendering
  - Test device information display
  - Test error notification display
  - Test "no sensors" message when none available

- [ ] 13. Configuration and environment setup
- [ ] 13.1 Create environment configuration file
  - Move hardcoded API URL to environment variable
  - Move device address, name, description to configuration
  - Support different environments (dev, staging, production)
  - _Requirements: 5.1, 5.2_

- [ ] 13.2 Add device address generation on first launch
  - Generate unique hexadecimal device identifier
  - Store in AsyncStorage for persistence
  - Load on app startup
  - _Requirements: 4.1, 5.3_

- [ ] 14. Final checkpoint - Ensure all tests pass
  - Run all unit tests and verify they pass
  - Run all property-based tests and verify they pass
  - Test on physical device (iOS and Android)
  - Verify sensor data transmission to API endpoint
  - Ensure all tests pass, ask the user if questions arise
