# Requirements Document

## Introduction

The IoT Sensor Data API is a backend service that stores and serves IoT sensor readings through a REST API protected by x402 micropayments. The system uses Express.js for the web server, LowDB for JSON-based data persistence, and the x402-express middleware for blockchain-based payment verification. The API provides both free public endpoints for listing sensors and paid endpoints for accessing detailed sensor data.

## Glossary

- **IoT Sensor Data API**: The backend service that stores and serves sensor readings
- **x402 Payment Middleware**: Express middleware that verifies blockchain micropayments before allowing access to protected routes
- **LowDB**: A lightweight JSON database for Node.js that persists data to a file
- **Sensor Address**: A unique blockchain address or identifier for an IoT sensor device
- **Sensor Reading**: A timestamped data point containing sensor measurements (temperature, humidity, accelerometer, etc.)
- **Protected Route**: An API endpoint that requires x402 payment verification before access
- **Public Route**: An API endpoint that is freely accessible without payment
- **Facilitator**: The Coinbase x402 service that handles payment verification
- **Receiving Address**: The blockchain wallet address that receives payments for API access
- **Route Config**: Configuration object defining price, network, and metadata for x402 protected routes

## Requirements

### Requirement 1: Express Server Initialization

**User Story:** As a system administrator, I want the API server to start and listen on a configured port, so that clients can access the IoT sensor data endpoints.

#### Acceptance Criteria

1. WHEN the application starts THEN the IoT Sensor Data API SHALL listen on port 3000
2. WHEN the server starts successfully THEN the IoT Sensor Data API SHALL log the server URL to the console
3. WHEN the server starts THEN the IoT Sensor Data API SHALL log the public list route URL
4. WHEN the server starts THEN the IoT Sensor Data API SHALL log an example protected route URL
5. WHEN the application initializes THEN the IoT Sensor Data API SHALL load environment variables from the .env file

### Requirement 2: Middleware Configuration

**User Story:** As the system, I want to configure necessary middleware, so that the API can handle CORS, parse JSON, and verify payments.

#### Acceptance Criteria

1. WHEN the Express app initializes THEN the IoT Sensor Data API SHALL enable CORS middleware for cross-origin requests
2. WHEN the Express app initializes THEN the IoT Sensor Data API SHALL enable JSON body parsing middleware
3. WHEN the Express app initializes THEN the IoT Sensor Data API SHALL apply x402 payment middleware globally before route handlers
4. WHEN configuring payment middleware THEN the IoT Sensor Data API SHALL use the configured RECEIVING_ADDRESS
5. WHEN configuring payment middleware THEN the IoT Sensor Data API SHALL use the x402RouteConfigs for route pricing
6. WHEN configuring payment middleware THEN the IoT Sensor Data API SHALL use the Coinbase facilitator for payment verification

### Requirement 3: x402 Route Configuration

**User Story:** As a service provider, I want to define pricing and metadata for protected routes, so that clients know the cost and requirements for accessing sensor data.

#### Acceptance Criteria

1. WHEN defining the "GET /api/sensors/latest" route THEN the IoT Sensor Data API SHALL set the price to "$0.001"
2. WHEN defining the "GET /api/sensors/latest" route THEN the IoT Sensor Data API SHALL set the network to "base"
3. WHEN defining the "GET /api/sensors/latest" route THEN the IoT Sensor Data API SHALL include a description of the endpoint functionality
4. WHEN defining the "GET /api/sensors/latest" route THEN the IoT Sensor Data API SHALL specify an input schema requiring an "address" string parameter
5. WHEN defining the "GET /api/sensors/latestTop" route THEN the IoT Sensor Data API SHALL set the price to "$0.005"
6. WHEN defining the "GET /api/sensors/latestTop" route THEN the IoT Sensor Data API SHALL set the network to "base"
7. WHEN defining the "GET /api/sensors/latestTop" route THEN the IoT Sensor Data API SHALL include a description of the endpoint functionality
8. WHEN defining the "GET /api/sensors/latestTop" route THEN the IoT Sensor Data API SHALL specify an input schema requiring an "address" string parameter

### Requirement 4: Database Initialization and Access

**User Story:** As the system, I want to initialize and access the LowDB database, so that sensor data can be persisted and retrieved.

#### Acceptance Criteria

1. WHEN a route handler needs database access THEN the IoT Sensor Data API SHALL create a FileSync adapter pointing to the configured DB_FILE path
2. WHEN a route handler needs database access THEN the IoT Sensor Data API SHALL initialize a LowDB instance with the adapter
3. WHEN the database file path is "/home/ubuntu/server/db.json" THEN the IoT Sensor Data API SHALL use that path for data persistence
4. WHEN accessing sensor data THEN the IoT Sensor Data API SHALL use the "sensors" key as the root collection
5. WHEN a sensor address does not exist THEN the IoT Sensor Data API SHALL initialize an empty array for that address

### Requirement 5: Public Sensor List Endpoint

**User Story:** As a client application, I want to retrieve a list of all sensor addresses, so that I can discover available sensors without payment.

#### Acceptance Criteria

1. WHEN a GET request is made to "/api/sensors" THEN the IoT Sensor Data API SHALL return a list of all unique sensor addresses
2. WHEN retrieving the sensor list THEN the IoT Sensor Data API SHALL extract keys from the "sensors" collection
3. WHEN the sensor list is retrieved successfully THEN the IoT Sensor Data API SHALL return status code 200 with the address array
4. WHEN an error occurs retrieving the sensor list THEN the IoT Sensor Data API SHALL return status code 500 with an error message
5. WHEN an error occurs THEN the IoT Sensor Data API SHALL log the error to the console
6. WHEN the request includes query parameters THEN the IoT Sensor Data API SHALL log the query parameters

### Requirement 6: Sensor Data Ingestion Endpoint

**User Story:** As an IoT device, I want to submit sensor readings to the API, so that my data is stored and available for retrieval.

#### Acceptance Criteria

1. WHEN a POST request is made to "/api/sensors" with valid data THEN the IoT Sensor Data API SHALL store the sensor reading
2. WHEN receiving a POST request THEN the IoT Sensor Data API SHALL extract address, name, description, and data from the request body
3. WHEN storing a sensor reading THEN the IoT Sensor Data API SHALL generate a timestamp using Date.now()
4. WHEN the request is missing address, name, or data fields THEN the IoT Sensor Data API SHALL return status code 400 with an error message
5. WHEN the sensor address does not exist in the database THEN the IoT Sensor Data API SHALL initialize an empty array for that address
6. WHEN storing a reading THEN the IoT Sensor Data API SHALL append the reading to the sensor's array with timestamp, name, description, and data
7. WHEN the description field is missing THEN the IoT Sensor Data API SHALL use "No description" as the default value
8. WHEN the reading is stored successfully THEN the IoT Sensor Data API SHALL return status code 200 with a success status
9. WHEN a new reading is received THEN the IoT Sensor Data API SHALL log the reading details to the console

### Requirement 7: Latest Sensor Reading Endpoint (Protected)

**User Story:** As a paying client, I want to retrieve the most recent sensor reading for a specific address, so that I can access current sensor data.

#### Acceptance Criteria

1. WHEN a GET request is made to "/api/sensors/latest" with valid payment THEN the IoT Sensor Data API SHALL return the latest sensor reading
2. WHEN the request is missing the "address" query parameter THEN the IoT Sensor Data API SHALL return status code 400 with an error message
3. WHEN the address query parameter is provided THEN the IoT Sensor Data API SHALL log the address to the console
4. WHEN the sensor address does not exist in the database THEN the IoT Sensor Data API SHALL return status code 404 with an error message
5. WHEN the sensor address exists but has no data THEN the IoT Sensor Data API SHALL return status code 404 with an error message
6. WHEN retrieving the latest reading THEN the IoT Sensor Data API SHALL return the last element from the sensor's data array
7. WHEN the latest reading is found THEN the IoT Sensor Data API SHALL return status code 200 with the reading data
8. WHEN returning the reading THEN the IoT Sensor Data API SHALL include timestamp, name, description, and data fields

### Requirement 8: Latest Top 10 Readings Endpoint (Protected)

**User Story:** As a paying client, I want to retrieve the last 10 sensor readings for a specific address, so that I can analyze recent sensor history.

#### Acceptance Criteria

1. WHEN a GET request is made to "/api/sensors/latestTop" with valid payment THEN the IoT Sensor Data API SHALL return the last 10 sensor readings
2. WHEN the request is missing the "address" query parameter THEN the IoT Sensor Data API SHALL return status code 400 with an error message
3. WHEN the address query parameter is provided THEN the IoT Sensor Data API SHALL log the address to the console
4. WHEN the sensor address does not exist in the database THEN the IoT Sensor Data API SHALL return status code 404 with an error message
5. WHEN the sensor address exists but has no data THEN the IoT Sensor Data API SHALL return status code 404 with an error message
6. WHEN retrieving the top 10 readings THEN the IoT Sensor Data API SHALL use array slice with -10 to get the last 10 elements
7. WHEN fewer than 10 readings exist THEN the IoT Sensor Data API SHALL return all available readings
8. WHEN the readings are found THEN the IoT Sensor Data API SHALL return status code 200 with a "last" property containing the array
9. WHEN returning the readings THEN the IoT Sensor Data API SHALL include all reading fields (timestamp, name, description, data)

### Requirement 9: Payment Verification

**User Story:** As a service provider, I want to verify x402 payments before serving protected data, so that only paying clients can access premium endpoints.

#### Acceptance Criteria

1. WHEN a request is made to "/api/sensors/latest" without valid payment THEN the x402 middleware SHALL reject the request
2. WHEN a request is made to "/api/sensors/latestTop" without valid payment THEN the x402 middleware SHALL reject the request
3. WHEN a valid payment is provided THEN the x402 middleware SHALL allow the request to proceed to the route handler
4. WHEN payment verification succeeds THEN the payment SHALL be sent to the configured RECEIVING_ADDRESS
5. WHEN the payment amount matches the route price THEN the x402 middleware SHALL approve the request

### Requirement 10: Error Handling and Logging

**User Story:** As a developer, I want comprehensive error handling and logging, so that I can debug issues and monitor API usage.

#### Acceptance Criteria

1. WHEN an error occurs during database operations THEN the IoT Sensor Data API SHALL catch the error and log it to the console
2. WHEN a client provides invalid input THEN the IoT Sensor Data API SHALL return a descriptive error message
3. WHEN a sensor address is not found THEN the IoT Sensor Data API SHALL return a 404 error with the address in the message
4. WHEN a new sensor reading is received THEN the IoT Sensor Data API SHALL log the reading details including address, name, description, data, and timestamp
5. WHEN query parameters are received THEN the IoT Sensor Data API SHALL log the query parameters
6. WHEN an address parameter is provided THEN the IoT Sensor Data API SHALL log the address value
