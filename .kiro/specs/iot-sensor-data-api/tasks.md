# Implementation Plan

## Status: Implementation Complete - Testing Phase

The core API functionality has been implemented in `diot-api/index.js`. The following tasks focus on adding comprehensive testing coverage to ensure correctness and reliability.

---

## Testing Infrastructure

- [ ] 1. Set up testing framework and project structure
  - Create `package.json` with test dependencies (Jest, fast-check, supertest)
  - Configure Jest for Node.js environment
  - Set up test directory structure (`__tests__/`)
  - Configure test database path (use temporary file for tests)
  - Add npm test scripts for running tests
  - _Requirements: All requirements need testing coverage_

---

## Property-Based Testing

- [ ] 2. Implement core data persistence property tests
  - **Property 5: Sensor data persistence** - For any valid sensor reading posted, it should be stored and retrievable
  - **Validates: Requirements 6.1**
  - Create generators for sensor addresses (hex strings)
  - Create generators for sensor readings (address, name, data objects)
  - Test that POST then GET returns the same data
  - Run 100 iterations minimum

- [ ] 3. Implement chronological ordering property test
  - **Property 8: Chronological ordering** - For any sequence of readings, they should be stored in order received
  - **Validates: Requirements 6.6**
  - Generate multiple readings for same address
  - POST them in sequence
  - Verify GET /api/sensors/latestTop returns them in correct order
  - Run 100 iterations minimum

- [ ] 4. Implement latest reading retrieval property test
  - **Property 12: Latest reading retrieval** - For any sensor with readings, GET latest should return the most recent
  - **Validates: Requirements 7.1, 7.6**
  - Generate random number of readings (1-20)
  - POST all readings
  - Verify GET /api/sensors/latest returns the last one posted
  - Run 100 iterations minimum

- [ ] 5. Implement top 10 readings property test
  - **Property 17: Top 10 readings retrieval** - For any sensor, GET latestTop should return last 10 (or all if fewer)
  - **Validates: Requirements 8.1, 8.7**
  - Generate random number of readings (0-30)
  - POST all readings
  - Verify GET /api/sensors/latestTop returns correct slice
  - Test both cases: <10 readings and >10 readings
  - Run 100 iterations minimum

- [ ] 6. Implement validation property tests
  - **Property 7: Required field validation** - For any POST missing required fields, return 400
  - **Validates: Requirements 6.4**
  - Generate readings with missing address, name, or data
  - Verify all return 400 status
  - **Property 13: Missing address parameter** - For any GET without address param, return 400
  - **Validates: Requirements 7.2, 8.2**
  - Run 100 iterations minimum

- [ ] 7. Implement error handling property tests
  - **Property 15: Non-existent sensor handling** - For any non-existent address, return 404
  - **Validates: Requirements 7.4, 8.4**
  - Generate random addresses not in database
  - Verify GET requests return 404
  - **Property 24: Address in 404 errors** - Error message should include the requested address
  - **Validates: Requirements 10.3**
  - Run 100 iterations minimum

- [ ] 8. Implement data format property tests
  - **Property 9: Default description handling** - For any reading without description, use "No description"
  - **Validates: Requirements 6.7**
  - Generate readings with and without description field
  - Verify stored data has correct description value
  - **Property 1: Sensor initialization** - For any new sensor address, initialize empty array
  - **Validates: Requirements 4.5**
  - Run 100 iterations minimum

---

## Unit Testing

- [ ] 9. Write unit tests for server initialization
  - Test server starts on port 3000
  - Test environment variables loaded correctly
  - Test startup logs include expected URLs
  - Test middleware stack configured in correct order (CORS → JSON → x402)
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 2.1, 2.2, 2.3_

- [ ] 10. Write unit tests for public endpoints
  - Test GET /api/sensors returns array of addresses
  - Test GET /api/sensors with empty database returns empty array
  - Test GET /api/sensors with multiple sensors returns all addresses
  - Test POST /api/sensors with valid data returns 200
  - Test POST /api/sensors creates new sensor entry
  - Test POST /api/sensors appends to existing sensor
  - _Requirements: 5.1, 5.2, 5.3, 6.1, 6.2, 6.6_

- [ ] 11. Write unit tests for protected endpoints
  - Mock x402 middleware for these tests
  - Test GET /api/sensors/latest with valid address returns latest reading
  - Test GET /api/sensors/latest response includes all fields (timestamp, name, description, data)
  - Test GET /api/sensors/latestTop returns array with "last" property
  - Test GET /api/sensors/latestTop with 5 readings returns all 5
  - Test GET /api/sensors/latestTop with 15 readings returns last 10
  - _Requirements: 7.1, 7.6, 7.7, 7.8, 8.1, 8.6, 8.7, 8.8, 8.9_

- [ ] 12. Write unit tests for error cases
  - Test 400 errors for missing required fields
  - Test 404 errors for non-existent sensors
  - Test 404 errors for sensors with no data
  - Test 500 errors for database failures (mock FileSync to throw)
  - Test error messages are descriptive
  - Test errors are logged to console
  - _Requirements: 6.4, 7.2, 7.4, 7.5, 8.2, 8.4, 8.5, 10.1, 10.2, 10.3_

- [ ] 13. Write unit tests for logging behavior
  - Test query parameters are logged (Requirements 5.6, 10.5)
  - Test address parameters are logged (Requirements 7.3, 8.3, 10.6)
  - Test new readings are logged with all details (Requirements 6.9, 10.4)
  - Test database errors are logged (Requirements 10.1)
  - _Requirements: 5.6, 6.9, 7.3, 8.3, 10.1, 10.4, 10.5, 10.6_

---

## Integration Testing

- [ ] 14. Write integration tests for end-to-end flows
  - Test complete flow: POST reading → GET /api/sensors (verify address in list) → GET latest (verify data)
  - Test multiple sensors: POST to 3 different addresses → GET /api/sensors → verify all 3 addresses
  - Test multiple readings: POST 15 readings → GET latestTop → verify correct 10 returned
  - Test CORS headers present in all responses
  - Test JSON parsing works for complex data objects
  - _Requirements: Multiple requirements - end-to-end validation_

---

## x402 Payment Integration Testing

- [ ] 15. Write tests for x402 route configuration
  - Test x402RouteConfigs has correct pricing for /api/sensors/latest ($0.001)
  - Test x402RouteConfigs has correct pricing for /api/sensors/latestTop ($0.005)
  - Test x402RouteConfigs has correct network (base) for both routes
  - Test x402RouteConfigs has input schemas with required address parameter
  - Test x402RouteConfigs has descriptions for both routes
  - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5, 3.6, 3.7, 3.8_

- [ ] 16. Write integration tests for payment middleware
  - Set up test wallets with Base testnet ETH
  - Test protected endpoints reject requests without payment (402 status)
  - Test protected endpoints accept requests with valid payment
  - Test payment amount matches route price
  - Test payments sent to RECEIVING_ADDRESS
  - Note: This requires actual blockchain interaction or sophisticated mocking
  - _Requirements: 9.1, 9.2, 9.3, 9.4, 9.5_

---

## Test Execution & Quality

- [ ] 17. Checkpoint - Ensure all tests pass
  - Run full test suite with `npm test`
  - Verify all property tests run 100+ iterations
  - Check test coverage report (aim for 80%+ coverage)
  - Fix any failing tests
  - Document any known limitations or edge cases
  - Ensure all tests pass, ask the user if questions arise

---

## Documentation & Deployment Readiness

- [ ] 18. Create testing documentation
  - Document how to run tests locally
  - Document test database setup
  - Document x402 payment testing with test wallets
  - Add testing section to README
  - Document any test environment variables needed
  - _Requirements: Supporting all requirements through documentation_

