# DIOT Mobile Sensor Device - Design Document

## Overview

The DIOT Mobile Sensor Device is a React Native mobile application built with Expo that transforms smartphones into IoT sensor nodes. The application continuously monitors device hardware sensors (accelerometer, gyroscope, magnetometer, and device motion), displays real-time readings on screen, and transmits sensor data to a remote API endpoint at regular intervals.

The application follows a reactive architecture where sensor data flows through a state management system to both the UI layer (for real-time display) and the network layer (for periodic transmission). The design emphasizes reliability, graceful error handling, and proper resource management during application lifecycle events.

## Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Mobile Application                    │
│  ┌────────────────────────────────────────────────────┐ │
│  │              UI Layer (React Native)               │ │
│  │  - Real-time sensor display                        │ │
│  │  - Device information display                      │ │
│  │  - Error notifications                             │ │
│  └────────────────┬───────────────────────────────────┘ │
│                   │                                      │
│  ┌────────────────▼───────────────────────────────────┐ │
│  │         State Management (React Context)           │ │
│  │  - Sensor readings state                           │ │
│  │  - Device information state                        │ │
│  │  - Error state                                     │ │
│  └────────────────┬───────────────────────────────────┘ │
│                   │                                      │
│  ┌────────────────▼───────────────────────────────────┐ │
│  │            Sensor Collection Layer                 │ │
│  │  - Sensor availability detection                   │ │
│  │  - Periodic sensor reading (500ms)                 │ │
│  │  - Gravity estimation (low-pass filter)            │ │
│  └────────────────┬───────────────────────────────────┘ │
│                   │                                      │
│  ┌────────────────▼───────────────────────────────────┐ │
│  │          Network Transmission Layer                │ │
│  │  - Periodic data transmission (10s)                │ │
│  │  - HTTP POST with JSON payload                     │ │
│  │  - Error handling and retry logic                  │ │
│  └────────────────┬───────────────────────────────────┘ │
└────────────────────┼───────────────────────────────────┘
                     │
                     ▼
            ┌────────────────┐
            │   DIOT API     │
            │   Endpoint     │
            └────────────────┘
```

### Component Architecture

The application is structured into the following key components:

1. **Sensor Manager**: Handles sensor availability detection, subscription management, and data collection
2. **Data Processor**: Applies transformations (gravity estimation via low-pass filter) and formats sensor data
3. **State Provider**: Manages application state using React Context API
4. **Transmission Service**: Handles periodic data transmission to the API endpoint
5. **UI Components**: Displays real-time sensor readings and device information
6. **Lifecycle Manager**: Handles sensor cleanup and reinitialization during app lifecycle events

## Components and Interfaces

### Sensor Manager

**Responsibilities:**
- Detect available sensors on device startup
- Subscribe to sensor updates at 500ms intervals
- Unsubscribe from sensors during cleanup
- Provide sensor availability status

**Interface:**
```typescript
interface SensorManager {
  // Initialize and detect available sensors
  initializeSensors(): Promise<SensorAvailability>;
  
  // Subscribe to sensor updates
  subscribeToAccelerometer(callback: (data: AccelerometerData) => void): Subscription;
  subscribeToGyroscope(callback: (data: GyroscopeData) => void): Subscription;
  subscribeToMagnetometer(callback: (data: MagnetometerData) => void): Subscription;
  subscribeToDeviceMotion(callback: (data: DeviceMotionData) => void): Subscription;
  
  // Cleanup all subscriptions
  unsubscribeAll(): void;
}

interface SensorAvailability {
  accelerometer: boolean;
  gyroscope: boolean;
  magnetometer: boolean;
  deviceMotion: boolean;
}
```

### Data Processor

**Responsibilities:**
- Apply low-pass filter to accelerometer data for gravity estimation
- Format sensor readings to three decimal places
- Structure data for transmission payload

**Interface:**
```typescript
interface DataProcessor {
  // Apply low-pass filter with alpha = 0.8
  estimateGravity(
    currentAcceleration: Vector3D,
    previousGravity: Vector3D
  ): Vector3D;
  
  // Format sensor data for display
  formatSensorValue(value: number): string;
  
  // Build transmission payload
  buildTransmissionPayload(
    deviceInfo: DeviceInfo,
    sensorData: SensorData
  ): TransmissionPayload;
}
```

### State Provider

**Responsibilities:**
- Maintain current sensor readings
- Store device information
- Track error states
- Provide state updates to UI components

**Interface:**
```typescript
interface AppState {
  deviceInfo: DeviceInfo;
  sensorData: SensorData;
  sensorAvailability: SensorAvailability;
  error: string | null;
}

interface StateProvider {
  state: AppState;
  updateSensorData(data: Partial<SensorData>): void;
  setError(error: string | null): void;
}
```

### Transmission Service

**Responsibilities:**
- Transmit sensor data every 10 seconds
- Handle network errors gracefully
- Continue operation despite transmission failures

**Interface:**
```typescript
interface TransmissionService {
  // Start periodic transmission
  startTransmission(
    apiEndpoint: string,
    interval: number,
    getPayload: () => TransmissionPayload
  ): void;
  
  // Stop transmission
  stopTransmission(): void;
  
  // Send data to API
  sendData(
    apiEndpoint: string,
    payload: TransmissionPayload
  ): Promise<TransmissionResult>;
}

interface TransmissionResult {
  success: boolean;
  error?: string;
}
```

## Data Models

### Sensor Data Types

```typescript
// Three-axis vector for accelerometer, gyroscope, magnetometer
interface Vector3D {
  x: number;
  y: number;
  z: number;
}

// Rotation data from device motion
interface RotationData {
  alpha: number;  // Z-axis rotation
  beta: number;   // X-axis rotation
  gamma: number;  // Y-axis rotation
}

// Device motion includes both acceleration and rotation
interface DeviceMotionData {
  acceleration: Vector3D;
  rotation: RotationData;
}

// Complete sensor data structure
interface SensorData {
  accelerometer: Vector3D | null;
  gyroscope: Vector3D | null;
  magnetometer: Vector3D | null;
  deviceMotion: DeviceMotionData | null;
  gravityEstimation: Vector3D | null;
  timestamp: number;
}

// Device identification
interface DeviceInfo {
  address: string;      // Hexadecimal device identifier
  name: string;         // Device name
  description: string;  // Device description
}

// Transmission payload structure
interface TransmissionPayload {
  address: string;
  name: string;
  description: string;
  timestamp: number;
  sensors: {
    accelerometer: Vector3D | null;
    gyroscope: Vector3D | null;
    magnetometer: Vector3D | null;
    deviceMotion: DeviceMotionData | null;
    gravityEstimation: Vector3D | null;
  };
}
```

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*


### Property Reflection

After analyzing all acceptance criteria, several properties can be consolidated:

**Consolidations:**
- Properties 3.1-3.6 (decimal formatting) can be combined into one property about all sensor values being formatted to 3 decimal places
- Properties 2.1-2.4 (sensor collection intervals) can be combined into one property about all sensors collecting at 500ms intervals
- Properties 4.1-4.3 (device info display) can be combined into one property about all device info being displayed
- Properties 5.3-5.7 (payload contents) can be combined into one comprehensive property about payload structure
- Properties 8.1-8.6 (data structure validation) can be combined into one property about correct data structure formatting

**Redundancies:**
- Property 8.x (data structure) is largely redundant with Property 5.7 (payload completeness) - the structure validation is implied by correct payload construction
- Properties 1.1-1.4 (individual sensor queries) can be combined into a single startup example

### Correctness Properties

Property 1: Sensor availability detection at startup
*For any* device, when the application starts, the system should query all four sensor types (accelerometer, gyroscope, magnetometer, device motion) for availability
**Validates: Requirements 1.1, 1.2, 1.3, 1.4**

Property 2: Sensor collection interval consistency
*For any* available sensor, the system should collect sensor readings at intervals of approximately 500 milliseconds (±50ms tolerance)
**Validates: Requirements 2.1, 2.2, 2.3, 2.4**

Property 3: Gravity estimation low-pass filter
*For any* accelerometer reading, the gravity estimation should follow the low-pass filter formula: `gravity[n] = 0.8 * gravity[n-1] + 0.2 * acceleration[n]`
**Validates: Requirements 2.5**

Property 4: Sensor value decimal formatting
*For any* sensor value displayed in the UI, the value should be formatted to exactly three decimal places
**Validates: Requirements 3.1, 3.2, 3.3, 3.4, 3.5, 3.6**

Property 5: Device information display completeness
*For any* device information display, the UI should show all three required fields: device address, device name, and device description
**Validates: Requirements 4.1, 4.2, 4.3**

Property 6: Transmission interval consistency
*For any* running application instance, sensor data transmissions should occur at intervals of approximately 10 seconds (±1s tolerance)
**Validates: Requirements 5.2**

Property 7: Transmission payload completeness
*For any* data transmission, the payload should include device address, device name, device description, timestamp in milliseconds, and all available sensor readings (or null for unavailable sensors)
**Validates: Requirements 5.3, 5.4, 5.5, 5.6, 5.7, 5.9**

Property 8: HTTP transmission format
*For any* data transmission, the HTTP request should use POST method with Content-Type header set to "application/json"
**Validates: Requirements 5.8**

Property 9: Transmission failure resilience
*For any* transmission failure, the system should continue collecting sensor data without interruption and attempt the next transmission at the normal interval
**Validates: Requirements 6.2, 6.3**

Property 10: Error notification on transmission failure
*For any* transmission failure, the system should display an error notification to the user
**Validates: Requirements 6.1**

Property 11: Sensor cleanup on background
*For any* application backgrounding event, the system should unsubscribe all active sensor listeners to prevent memory leaks
**Validates: Requirements 7.1, 7.3**

Property 12: Sensor reinitialization on foreground
*For any* application foregrounding event, the system should reinitialize and resubscribe to all available sensor listeners
**Validates: Requirements 7.2**

Property 13: Transmission payload structure validation
*For any* transmission payload, vector data (accelerometer, gyroscope, magnetometer, gravity) should be objects with numeric x, y, z properties, and rotation data should be objects with numeric alpha, beta, gamma properties
**Validates: Requirements 8.1, 8.2, 8.3, 8.4, 8.5, 8.6**

## Error Handling

### Network Errors

**Strategy:** Graceful degradation with user notification

- Transmission failures do not interrupt sensor collection
- Display toast/alert notification to user on transmission failure
- Continue attempting transmissions at regular intervals
- Log errors for debugging purposes

**Implementation:**
```typescript
async function transmitData(payload: TransmissionPayload): Promise<void> {
  try {
    const response = await fetch(API_ENDPOINT, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload)
    });
    
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
  } catch (error) {
    // Display error to user
    showErrorNotification(`Transmission failed: ${error.message}`);
    
    // Log for debugging
    console.error('Transmission error:', error);
    
    // Continue normal operation - next transmission will be attempted
  }
}
```

### Sensor Unavailability

**Strategy:** Null-safe data handling with user notification

- Detect sensor availability at startup
- Display notification if no sensors are available
- Include null values in transmission payload for unavailable sensors
- UI displays only available sensor data

**Implementation:**
```typescript
function checkSensorAvailability(): SensorAvailability {
  const availability = {
    accelerometer: Accelerometer.isAvailableAsync(),
    gyroscope: Gyroscope.isAvailableAsync(),
    magnetometer: Magnetometer.isAvailableAsync(),
    deviceMotion: DeviceMotion.isAvailableAsync()
  };
  
  const hasAnySensor = Object.values(availability).some(v => v);
  
  if (!hasAnySensor) {
    showNotification('No supported sensors found on this device');
  }
  
  return availability;
}
```

### Lifecycle Management Errors

**Strategy:** Defensive cleanup with error boundaries

- Wrap sensor subscriptions in try-catch blocks
- Ensure cleanup happens even if errors occur
- Use React error boundaries to catch rendering errors
- Maintain subscription references for proper cleanup

**Implementation:**
```typescript
function cleanupSensors(): void {
  try {
    subscriptions.forEach(subscription => {
      if (subscription && typeof subscription.remove === 'function') {
        subscription.remove();
      }
    });
    subscriptions = [];
  } catch (error) {
    console.error('Cleanup error:', error);
    // Force clear subscriptions array even if removal fails
    subscriptions = [];
  }
}
```

## Testing Strategy

### Unit Testing

The application will use **Jest** as the testing framework with **React Native Testing Library** for component testing.

**Unit test coverage includes:**

1. **Data Processor Tests**
   - Gravity estimation calculation with various input values
   - Decimal formatting for edge cases (very small, very large, negative numbers)
   - Payload construction with complete and partial sensor data

2. **Sensor Manager Tests**
   - Sensor availability detection mocking
   - Subscription lifecycle management
   - Cleanup verification

3. **Transmission Service Tests**
   - HTTP request format validation
   - Error handling for network failures
   - Interval timing verification (using fake timers)

4. **UI Component Tests**
   - Sensor data display rendering
   - Device information display
   - Error notification display

### Property-Based Testing

The application will use **fast-check** for property-based testing in JavaScript/TypeScript.

**Configuration:**
- Minimum 100 iterations per property test
- Each property test must reference its corresponding design property using the format: `**Feature: diot-mobile-sensor-device, Property {number}: {property_text}**`

**Property test coverage:**

1. **Property 3: Gravity estimation low-pass filter**
   - Generate random accelerometer readings and previous gravity values
   - Verify the calculation follows the formula exactly
   - Test with edge cases (zero values, extreme values)

2. **Property 4: Sensor value decimal formatting**
   - Generate random floating-point numbers
   - Verify all formatted values have exactly 3 decimal places
   - Test with various magnitudes and signs

3. **Property 7: Transmission payload completeness**
   - Generate random device info and sensor data combinations
   - Verify all required fields are present in payload
   - Verify null handling for unavailable sensors

4. **Property 13: Transmission payload structure validation**
   - Generate random sensor data
   - Verify vector objects have x, y, z numeric properties
   - Verify rotation objects have alpha, beta, gamma numeric properties
   - Verify no extra properties are included

**Example property test structure:**
```typescript
import fc from 'fast-check';

describe('Property Tests', () => {
  test('Property 3: Gravity estimation low-pass filter', () => {
    /**
     * Feature: diot-mobile-sensor-device, Property 3: Gravity estimation low-pass filter
     * Validates: Requirements 2.5
     */
    fc.assert(
      fc.property(
        fc.record({
          x: fc.float(),
          y: fc.float(),
          z: fc.float()
        }),
        fc.record({
          x: fc.float(),
          y: fc.float(),
          z: fc.float()
        }),
        (acceleration, previousGravity) => {
          const result = estimateGravity(acceleration, previousGravity);
          const expected = {
            x: 0.8 * previousGravity.x + 0.2 * acceleration.x,
            y: 0.8 * previousGravity.y + 0.2 * acceleration.y,
            z: 0.8 * previousGravity.z + 0.2 * acceleration.z
          };
          
          expect(result.x).toBeCloseTo(expected.x, 10);
          expect(result.y).toBeCloseTo(expected.y, 10);
          expect(result.z).toBeCloseTo(expected.z, 10);
        }
      ),
      { numRuns: 100 }
    );
  });
});
```

### Integration Testing

Integration tests will verify end-to-end workflows:

1. **Sensor-to-UI Flow**
   - Mock sensor data updates
   - Verify UI displays updated values correctly
   - Verify decimal formatting in rendered output

2. **Sensor-to-Transmission Flow**
   - Mock sensor data collection
   - Verify transmission payload structure
   - Verify transmission timing

3. **Lifecycle Management Flow**
   - Simulate app backgrounding/foregrounding
   - Verify sensor cleanup and reinitialization
   - Verify no memory leaks

### Manual Testing Checklist

Due to the hardware-dependent nature of the application, manual testing on physical devices is essential:

- [ ] Test on iOS device with all sensors available
- [ ] Test on Android device with all sensors available
- [ ] Test on device with limited sensor availability
- [ ] Verify real-time UI updates during device movement
- [ ] Verify transmission to actual API endpoint
- [ ] Test app backgrounding and foregrounding behavior
- [ ] Verify error notifications appear on network failure
- [ ] Test with airplane mode to simulate network errors
- [ ] Verify no crashes during extended operation (30+ minutes)

## Implementation Considerations

### Technology Stack

- **Framework:** React Native 0.81.5 with Expo ~54
- **Sensors:** expo-sensors package
- **State Management:** React Context API
- **HTTP Client:** fetch API
- **Testing:** Jest + React Native Testing Library + fast-check

### Performance Considerations

1. **Sensor Update Frequency**
   - 500ms interval balances responsiveness with battery life
   - Expo sensors use native event throttling for efficiency

2. **State Updates**
   - Batch sensor updates to minimize re-renders
   - Use React.memo for sensor display components
   - Debounce UI updates if necessary

3. **Memory Management**
   - Properly cleanup sensor subscriptions
   - Avoid memory leaks during lifecycle transitions
   - Monitor subscription array size

4. **Network Efficiency**
   - 10-second transmission interval reduces network overhead
   - Single HTTP request per transmission
   - Payload size approximately 500-800 bytes

### Security Considerations

1. **API Endpoint**
   - Use HTTPS for data transmission
   - Store API endpoint in environment configuration
   - Validate API responses

2. **Device Identification**
   - Generate unique device address on first launch
   - Store device address securely (AsyncStorage)
   - Do not expose sensitive device information

3. **Data Privacy**
   - Sensor data is non-personally identifiable
   - Inform users about data collection
   - Provide opt-out mechanism if required

### Scalability Considerations

1. **Multiple Devices**
   - Each device operates independently
   - No device-to-device communication required
   - API endpoint must handle concurrent requests

2. **Extended Operation**
   - Application designed for continuous operation
   - Proper cleanup prevents resource exhaustion
   - Error recovery ensures long-term stability

3. **Future Enhancements**
   - Modular architecture allows adding new sensor types
   - Transmission service can be extended for batching
   - UI components can be enhanced without affecting core logic

## Design Rationale

### Why React Native with Expo?

- Cross-platform support (iOS and Android) from single codebase
- Expo provides unified sensor APIs with consistent behavior
- Rapid development and testing with Expo Go
- Strong community support and documentation

### Why 500ms Sensor Update Interval?

- Balances real-time responsiveness with battery efficiency
- Sufficient for most motion detection use cases
- Aligns with typical sensor sampling rates
- Reduces computational overhead

### Why 10-Second Transmission Interval?

- Reduces network overhead and battery consumption
- Provides near-real-time data for monitoring applications
- Allows for reasonable data buffering
- Balances freshness with efficiency

### Why Low-Pass Filter for Gravity Estimation?

- Separates gravitational acceleration from device motion
- Alpha value of 0.8 provides good balance between smoothness and responsiveness
- Standard technique in mobile sensor processing
- Enables more accurate motion detection

### Why React Context for State Management?

- Sufficient for single-screen application
- Avoids complexity of Redux or MobX
- Native React solution with no additional dependencies
- Easy to test and reason about

### Why Graceful Degradation for Network Errors?

- Sensor collection should not depend on network availability
- Users can still monitor local sensor readings
- Automatic retry on next interval prevents data loss
- Improves user experience during connectivity issues

## Deployment Considerations

### Build Configuration

- Configure API endpoint URL for production environment
- Set appropriate app permissions for sensor access
- Configure background execution if needed
- Set up proper app icons and splash screens

### App Store Requirements

- Request sensor permissions with clear explanations
- Provide privacy policy for data collection
- Test on minimum supported OS versions
- Ensure compliance with platform guidelines

### Monitoring and Debugging

- Implement logging for production debugging
- Track transmission success/failure rates
- Monitor sensor availability across devices
- Collect crash reports for stability improvements
