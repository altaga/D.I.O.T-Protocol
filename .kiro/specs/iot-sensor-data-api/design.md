# IoT Sensor Data API - Design Document

## Overview

The IoT Sensor Data API is a Node.js/Express backend service that provides REST endpoints for storing and retrieving IoT sensor readings. The system integrates x402 micropayment protocol to monetize premium data access while keeping basic discovery endpoints free. Data is persisted using LowDB, a lightweight JSON-based database suitable for small to medium-scale IoT deployments.

**Key Design Decisions:**
- **Express.js Framework**: Chosen for its simplicity, extensive middleware ecosystem, and excellent support for REST APIs
- **LowDB for Persistence**: Lightweight, file-based JSON storage eliminates database server overhead while providing sufficient performance for IoT data ingestion rates
- **x402 Micropayments**: Enables monetization of sensor data access with sub-cent pricing, making it economically viable for both providers and consumers
- **Base Network**: Coinbase's Layer 2 solution provides low transaction fees suitable for micropayments
- **Middleware-First Architecture**: Payment verification happens at the middleware layer, keeping route handlers focused on business logic

## Architecture

### System Components

```
┌─────────────────┐
│  IoT Devices    │
│  (POST data)    │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│         Express Application              │
│  ┌───────────────────────────────────┐  │
│  │   Middleware Stack                │  │
│  │  - CORS                           │  │
│  │  - JSON Body Parser               │  │
│  │  - x402 Payment Verification      │  │
│  └───────────────────────────────────┘  │
│  ┌───────────────────────────────────┐  │
│  │   Route Handlers                  │  │
│  │  - GET /api/sensors (public)      │  │
│  │  - POST /api/sensors (public)     │  │
│  │  - GET /api/sensors/latest (paid) │  │
│  │  - GET /api/sensors/latestTop     │  │
│  │    (paid)                         │  │
│  └───────────────────────────────────┘  │
└──────────────┬──────────────────────────┘
               │
               ▼
        ┌──────────────┐
        │    LowDB     │
        │  (db.json)   │
        └──────────────┘
```

### Request Flow

**Public Endpoints (No Payment):**
1. Client → CORS → JSON Parser → Route Handler → LowDB → Response

**Protected Endpoints (Payment Required):**
1. Client → CORS → JSON Parser → x402 Middleware
2. x402 Middleware → Coinbase Facilitator (payment verification)
3. Payment Valid → Route Handler → LowDB → Response
4. Payment Invalid → 402 Payment Required Response

### Data Flow

**Sensor Data Ingestion:**
- IoT Device → POST /api/sensors → Validate Input → Generate Timestamp → Store in LowDB → Confirm

**Sensor Data Retrieval:**
- Client → Payment → GET /api/sensors/latest?address=X → Fetch from LowDB → Return Latest Reading

## Components and Interfaces

### 1. Express Server

**Responsibilities:**
- Initialize HTTP server on port 3000
- Load environment variables
- Configure middleware stack
- Register route handlers
- Log startup information

**Configuration:**
```javascript
{
  port: 3000,
  env: {
    RECEIVING_ADDRESS: string,  // Wallet for receiving payments
    DB_FILE: string             // Path to LowDB file
  }
}
```

### 2. Middleware Stack

**CORS Middleware:**
- Enables cross-origin requests from web clients
- Allows all origins (can be restricted in production)

**JSON Body Parser:**
- Parses incoming JSON request bodies
- Makes data available in `req.body`

**x402 Payment Middleware:**
- Verifies blockchain micropayments before protected routes
- Configuration:
  - `receivingAddress`: Wallet address for payments
  - `routeConfigs`: Price and metadata for each protected route
  - `facilitator`: Coinbase x402 service instance

### 3. x402 Route Configuration

**Purpose:** Define pricing, network, and metadata for protected endpoints

**Structure:**
```javascript
{
  "GET /api/sensors/latest": {
    price: "$0.001",
    network: "base",
    config: {
      description: "Get latest sensor reading",
      inputSchema: {
        type: "object",
        properties: {
          address: { type: "string" }
        },
        required: ["address"]
      }
    }
  },
  "GET /api/sensors/latestTop": {
    price: "$0.005",
    network: "base",
    config: {
      description: "Get last 10 sensor readings with analysis",
      inputSchema: {
        type: "object",
        properties: {
          address: { type: "string" }
        },
        required: ["address"]
      }
    }
  }
}
```

**Design Rationale:**
- Latest reading priced lower ($0.001) for frequent polling use cases
- Top 10 analysis priced higher ($0.005) as it provides more data and processing
- Input schemas enable client-side validation and API documentation

### 4. Database Layer (LowDB)

**Responsibilities:**
- Persist sensor readings to JSON file
- Provide synchronous read/write access
- Initialize empty collections for new sensors

**Data Structure:**
```javascript
{
  "sensors": {
    "0xSensorAddress1": [
      {
        timestamp: 1234567890,
        name: "Sensor Name",
        description: "Sensor Description",
        data: {
          temperature: 25.5,
          humidity: 60,
          // ... other sensor fields
        }
      }
    ],
    "0xSensorAddress2": [ /* readings */ ]
  }
}
```

**Design Rationale:**
- Sensor address as key enables O(1) lookup
- Array of readings maintains chronological order
- Timestamp enables time-based queries and analysis
- Flexible `data` object accommodates different sensor types

### 5. Route Handlers

#### GET /api/sensors (Public)

**Purpose:** List all available sensor addresses for discovery

**Input:** None

**Output:**
```javascript
{
  status: "success",
  addresses: ["0xAddr1", "0xAddr2", ...]
}
```

**Error Handling:**
- 500: Database read error

#### POST /api/sensors (Public)

**Purpose:** Ingest sensor readings from IoT devices

**Input:**
```javascript
{
  address: "0xSensorAddress",
  name: "Sensor Name",
  description: "Optional description",  // defaults to "No description"
  data: {
    temperature: 25.5,
    humidity: 60,
    // ... sensor-specific fields
  }
}
```

**Output:**
```javascript
{
  status: "success",
  message: "Sensor data stored successfully"
}
```

**Validation:**
- 400: Missing required fields (address, name, data)

**Error Handling:**
- 500: Database write error

#### GET /api/sensors/latest (Protected - $0.001)

**Purpose:** Retrieve most recent reading for a sensor

**Input:** Query parameter `address`

**Output:**
```javascript
{
  status: "success",
  reading: {
    timestamp: 1234567890,
    name: "Sensor Name",
    description: "Description",
    data: { /* sensor data */ }
  }
}
```

**Validation:**
- 400: Missing address parameter
- 404: Sensor address not found
- 404: Sensor has no data

**Error Handling:**
- 500: Database read error

#### GET /api/sensors/latestTop (Protected - $0.005)

**Purpose:** Retrieve last 10 readings for historical analysis

**Input:** Query parameter `address`

**Output:**
```javascript
{
  status: "success",
  last: [
    { timestamp, name, description, data },
    // ... up to 10 readings
  ]
}
```

**Validation:**
- 400: Missing address parameter
- 404: Sensor address not found
- 404: Sensor has no data

**Behavior:**
- Returns all readings if fewer than 10 exist
- Uses array slice(-10) to get last 10 elements

**Error Handling:**
- 500: Database read error

## Data Models

### Sensor Reading

**Schema:**
```javascript
{
  timestamp: Number,      // Unix timestamp (milliseconds)
  name: String,          // Sensor display name
  description: String,   // Optional description (default: "No description")
  data: Object          // Flexible sensor data object
}
```

**Constraints:**
- `timestamp`: Generated server-side using Date.now()
- `name`: Required, non-empty string
- `description`: Optional, defaults to "No description"
- `data`: Required, must be valid JSON object

**Common Data Fields (varies by sensor type):**
```javascript
{
  temperature: Number,    // Celsius
  humidity: Number,       // Percentage
  pressure: Number,       // hPa
  tvoc: Number,          // Total Volatile Organic Compounds
  eco2: Number,          // Equivalent CO2
  accelerometer: {
    x: Number,
    y: Number,
    z: Number
  }
}
```

### Database Schema

**Root Structure:**
```javascript
{
  sensors: {
    [sensorAddress: string]: SensorReading[]
  }
}
```

**Indexing Strategy:**
- Primary key: Sensor address (blockchain address or unique identifier)
- No secondary indexes (LowDB limitation)
- Chronological ordering maintained by array insertion order

**Scalability Considerations:**
- File-based storage suitable for <10,000 sensors
- For larger deployments, consider migrating to MongoDB or PostgreSQL
- Current design supports easy migration (sensor address as key)


## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property Reflection

After analyzing all acceptance criteria, several redundancies were identified:
- Criteria 6.5 duplicates 4.5 (sensor initialization)
- Criteria 8.2, 8.3, 8.4, 8.5 duplicate validation/logging from Requirement 7
- Criteria 10.4, 10.5, 10.6 duplicate logging requirements from earlier sections

These redundancies have been consolidated into comprehensive properties below.

### Core Functional Properties

**Property 1: Sensor initialization consistency**
*For any* sensor address that does not exist in the database, when accessed, the system should initialize an empty array for that address.
**Validates: Requirements 4.5**

**Property 2: Sensor list completeness**
*For any* database state, when GET /api/sensors is called, the returned addresses should exactly match all keys in the sensors collection.
**Validates: Requirements 5.1**

**Property 3: Successful list retrieval format**
*For any* successful GET /api/sensors request, the response should have status code 200 and contain an array of addresses.
**Validates: Requirements 5.3**

**Property 4: Query parameter logging**
*For any* request that includes query parameters, those parameters should be logged to the console.
**Validates: Requirements 5.6, 10.5**

**Property 5: Sensor data persistence**
*For any* valid sensor reading posted to /api/sensors, the reading should be stored and retrievable from the database.
**Validates: Requirements 6.1**

**Property 6: Timestamp generation**
*For any* sensor reading stored, the system should generate and include a timestamp field.
**Validates: Requirements 6.3**

**Property 7: Required field validation**
*For any* POST request to /api/sensors missing address, name, or data fields, the system should return status code 400 with an error message.
**Validates: Requirements 6.4**

**Property 8: Chronological ordering**
*For any* sequence of sensor readings posted to the same address, they should be stored in the order received (chronological).
**Validates: Requirements 6.6**

**Property 9: Default description handling**
*For any* sensor reading posted without a description field, the system should store it with description "No description".
**Validates: Requirements 6.7**

**Property 10: Successful storage response**
*For any* valid sensor reading successfully stored, the system should return status code 200 with a success status.
**Validates: Requirements 6.8**

**Property 11: Reading ingestion logging**
*For any* new sensor reading received, the system should log the reading details (address, name, description, data, timestamp) to the console.
**Validates: Requirements 6.9, 10.4**

**Property 12: Latest reading retrieval**
*For any* sensor address with at least one reading, GET /api/sensors/latest should return the most recently added reading (last element in array).
**Validates: Requirements 7.1, 7.6**

**Property 13: Missing address parameter validation**
*For any* GET request to /api/sensors/latest or /api/sensors/latestTop without an address query parameter, the system should return status code 400 with an error message.
**Validates: Requirements 7.2, 8.2**

**Property 14: Address parameter logging**
*For any* request to protected endpoints with an address query parameter, the address should be logged to the console.
**Validates: Requirements 7.3, 8.3, 10.6**

**Property 15: Non-existent sensor handling**
*For any* sensor address that does not exist in the database, GET requests to /api/sensors/latest or /api/sensors/latestTop should return status code 404 with an error message.
**Validates: Requirements 7.4, 8.4**

**Property 16: Successful latest reading response**
*For any* successful retrieval of the latest reading, the system should return status code 200 with the reading data including timestamp, name, description, and data fields.
**Validates: Requirements 7.7, 7.8**

**Property 17: Top 10 readings retrieval**
*For any* sensor address with readings, GET /api/sensors/latestTop should return the last 10 readings (or all readings if fewer than 10 exist).
**Validates: Requirements 8.1, 8.7**

**Property 18: Successful top 10 response format**
*For any* successful retrieval of top 10 readings, the system should return status code 200 with a "last" property containing an array of readings, each with timestamp, name, description, and data fields.
**Validates: Requirements 8.8, 8.9**

### Payment Verification Properties

**Property 19: Payment requirement enforcement**
*For any* request to /api/sensors/latest or /api/sensors/latestTop without valid payment, the x402 middleware should reject the request.
**Validates: Requirements 9.1, 9.2**

**Property 20: Valid payment acceptance**
*For any* request to protected endpoints with valid payment matching the route price, the x402 middleware should allow the request to proceed to the route handler.
**Validates: Requirements 9.3, 9.5**

**Property 21: Payment routing**
*For any* successful payment verification, the payment should be sent to the configured RECEIVING_ADDRESS.
**Validates: Requirements 9.4**

### Error Handling Properties

**Property 22: Database error handling**
*For any* database operation that throws an error, the system should catch the error, log it to the console, and return status code 500.
**Validates: Requirements 10.1**

**Property 23: Descriptive error messages**
*For any* invalid input from a client, the system should return an error message that describes what was invalid.
**Validates: Requirements 10.2**

**Property 24: Address in 404 errors**
*For any* 404 error due to sensor address not found, the error message should include the requested address.
**Validates: Requirements 10.3**

## Error Handling

### Error Categories

**1. Client Errors (4xx)**

**400 Bad Request:**
- Missing required fields in POST /api/sensors (address, name, data)
- Missing address query parameter in GET /api/sensors/latest
- Missing address query parameter in GET /api/sensors/latestTop
- Invalid JSON in request body

**Response Format:**
```javascript
{
  status: "error",
  message: "Descriptive error message"
}
```

**402 Payment Required:**
- Missing or invalid x402 payment for protected endpoints
- Payment amount doesn't match route price
- Payment verification fails

**Response Format:** (Handled by x402 middleware)
```javascript
{
  error: "Payment required",
  price: "$0.001",
  network: "base",
  receivingAddress: "0x..."
}
```

**404 Not Found:**
- Sensor address does not exist in database
- Sensor address exists but has no readings
- Route not found

**Response Format:**
```javascript
{
  status: "error",
  message: "Sensor address {address} not found" // or "No data available"
}
```

**2. Server Errors (5xx)**

**500 Internal Server Error:**
- Database read/write failures
- File system errors (LowDB)
- Unexpected exceptions in route handlers

**Response Format:**
```javascript
{
  status: "error",
  message: "Internal server error"
}
```

**Error Logging:**
- All errors logged to console with stack traces
- Includes request context (method, path, query params)
- Timestamp for debugging

### Error Recovery

**Database Errors:**
- LowDB operations are synchronous, so errors are immediate
- No automatic retry logic (file system errors are typically fatal)
- Recommendation: Monitor disk space and file permissions

**Payment Errors:**
- x402 middleware handles payment verification failures
- No retry logic at API level (client must retry with valid payment)
- Payment failures logged for audit purposes

**Validation Errors:**
- Fail fast with descriptive error messages
- No partial data storage (atomic operations)
- Client responsible for correcting input and retrying

## Testing Strategy

### Dual Testing Approach

This project will use both unit testing and property-based testing to ensure comprehensive coverage:

- **Unit tests** verify specific examples, edge cases, and integration points
- **Property-based tests** verify universal properties that should hold across all inputs
- Together they provide comprehensive coverage: unit tests catch concrete bugs, property tests verify general correctness

### Unit Testing

**Framework:** Jest (Node.js standard)

**Test Organization:**
```
diot-api/
  __tests__/
    server.test.js          # Server initialization
    routes.test.js          # Route handlers
    database.test.js        # LowDB operations
    middleware.test.js      # x402 integration
```

**Unit Test Coverage:**

1. **Server Initialization Tests:**
   - Server starts on port 3000
   - Environment variables loaded correctly
   - Startup logs include expected URLs
   - Middleware stack configured in correct order

2. **Route Handler Tests:**
   - GET /api/sensors returns sensor list
   - POST /api/sensors stores valid readings
   - GET /api/sensors/latest returns latest reading
   - GET /api/sensors/latestTop returns up to 10 readings

3. **Validation Tests:**
   - Missing required fields return 400
   - Invalid JSON returns 400
   - Non-existent sensors return 404
   - Empty sensor arrays return 404

4. **Database Tests:**
   - New sensor addresses initialized with empty array
   - Readings appended in chronological order
   - Default description applied when missing
   - Database file created if not exists

5. **Integration Tests:**
   - End-to-end flow: POST reading → GET latest
   - Multiple readings for same sensor
   - Multiple sensors in database
   - CORS headers present in responses

**Mocking Strategy:**
- Mock x402 middleware for route tests (payment verification tested separately)
- Use in-memory or temporary file for LowDB in tests
- Mock console.log for logging verification

### Property-Based Testing

**Framework:** fast-check (JavaScript property-based testing library)

**Configuration:**
- Minimum 100 iterations per property test
- Seed-based reproducibility for failed tests
- Shrinking enabled to find minimal failing cases

**Property Test Tagging:**
Each property-based test must include a comment with this format:
```javascript
// Feature: iot-sensor-data-api, Property X: [property description]
```

**Property Test Coverage:**

Each correctness property listed in the Correctness Properties section must be implemented as a single property-based test. The tests will generate random inputs to verify the properties hold universally.

**Example Property Test Structure:**
```javascript
// Feature: iot-sensor-data-api, Property 5: Sensor data persistence
test('Property 5: Any valid sensor reading should be stored and retrievable', () => {
  fc.assert(
    fc.property(
      fc.record({
        address: fc.hexaString({ minLength: 40, maxLength: 42 }),
        name: fc.string({ minLength: 1 }),
        data: fc.object()
      }),
      async (reading) => {
        // POST reading
        const postResponse = await request(app)
          .post('/api/sensors')
          .send(reading);
        
        expect(postResponse.status).toBe(200);
        
        // GET latest reading
        const getResponse = await request(app)
          .get('/api/sensors/latest')
          .query({ address: reading.address });
        
        expect(getResponse.status).toBe(200);
        expect(getResponse.body.reading.name).toBe(reading.name);
      }
    ),
    { numRuns: 100 }
  );
});
```

**Generators for Property Tests:**

1. **Sensor Address Generator:**
   - Valid blockchain addresses (0x + 40 hex chars)
   - Random hex strings for testing

2. **Sensor Reading Generator:**
   - Required fields: address, name, data
   - Optional field: description
   - Various data object structures (temperature, humidity, accelerometer, etc.)

3. **Query Parameter Generator:**
   - Valid addresses
   - Missing addresses (for validation tests)
   - Invalid formats

4. **Database State Generator:**
   - Empty database
   - Single sensor with varying number of readings
   - Multiple sensors with readings
   - Sensors with empty arrays

**Property Test Priorities:**

High Priority (Core Functionality):
- Property 5: Sensor data persistence
- Property 8: Chronological ordering
- Property 12: Latest reading retrieval
- Property 17: Top 10 readings retrieval

Medium Priority (Validation & Error Handling):
- Property 7: Required field validation
- Property 13: Missing address parameter validation
- Property 15: Non-existent sensor handling
- Property 22: Database error handling

Lower Priority (Logging & Formatting):
- Property 4: Query parameter logging
- Property 11: Reading ingestion logging
- Property 14: Address parameter logging

### Test Execution

**Local Development:**
```bash
npm test                    # Run all tests
npm test -- --watch        # Watch mode
npm test -- --coverage     # Coverage report
```

**CI/CD Pipeline:**
- Run all tests on every commit
- Require 80% code coverage minimum
- Property tests must pass 100 iterations
- No failing tests allowed in main branch

### Testing x402 Payment Integration

**Approach:**
- Unit tests mock x402 middleware to test route logic independently
- Integration tests use test wallets with Base testnet
- Property tests verify payment amounts match route configurations

**Test Wallets:**
- Create dedicated test wallets for CI/CD
- Fund with testnet ETH
- Rotate keys periodically for security

**Payment Verification Tests:**
- Valid payment allows access
- Invalid payment returns 402
- Payment amount matches route price
- Payment sent to correct receiving address

### Manual Testing Checklist

Before deployment, manually verify:
- [ ] Server starts and logs correct URLs
- [ ] Public endpoints accessible without payment
- [ ] Protected endpoints require payment
- [ ] IoT device can POST sensor data
- [ ] Client can retrieve latest reading with payment
- [ ] Client can retrieve top 10 readings with payment
- [ ] Database persists across server restarts
- [ ] CORS allows requests from expected origins
- [ ] Error messages are descriptive and helpful
- [ ] Logs provide sufficient debugging information

### Performance Testing

**Load Testing:**
- Simulate 100 concurrent IoT devices posting data
- Measure response times for GET /api/sensors/latest
- Verify database file size growth is manageable
- Test with 1000+ sensors and 10,000+ readings

**Scalability Limits:**
- LowDB suitable for <10,000 sensors
- File size should stay under 100MB for good performance
- Consider migration to PostgreSQL/MongoDB if limits exceeded

**Monitoring:**
- Track response times for all endpoints
- Monitor database file size
- Alert on error rate spikes
- Track payment success/failure rates
