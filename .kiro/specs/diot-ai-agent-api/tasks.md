# Implementation Plan

## Status: Implementation Complete - Testing Phase

The core implementation of the D.I.O.T AI Agent API is complete. All major features have been implemented including Express server, LLM configuration, wallet clients, IoT sensor tools, LangGraph workflow, and health monitoring. The following tasks focus on testing and validation to ensure correctness.

---

## Testing Tasks

- [ ] 1. Set up testing infrastructure
  - Create `diot-agent/package.json` with test dependencies (Jest, fast-check)
  - Configure Jest for ES modules support
  - Create test directory structure: `diot-agent/__tests__/`
  - _Requirements: All requirements need testing validation_

- [ ] 2. Implement core utility tests
  - [ ] 2.1 Create unit tests for sensor data formatting
    - Test `summarizeSensorData` function with various data shapes
    - Test unit suffix application (°C, %, hPa, ppb, ppm)
    - Test handling of null/undefined values
    - Test nested object formatting
    - _Requirements: 4.3, 4.4, 4.5, 4.6, 4.7_

  - [ ]* 2.2 Write property test for sensor data unit formatting
    - **Property 4: Sensor data unit formatting**
    - **Validates: Requirements 4.3, 4.4, 4.5, 4.6, 4.7**
    - Generate random sensor data objects
    - Verify correct unit suffixes appear in formatted output
    - Run 100 iterations minimum

  - [ ] 2.3 Create unit tests for accelerometer analysis
    - Test maximum value calculation with known datasets
    - Test abnormal detection with values above/below ±3 threshold
    - Test handling of string, integer, and float accelerometer values
    - Test NaN and invalid value handling
    - _Requirements: 5.2, 5.3, 5.4_

  - [ ]* 2.4 Write property test for anomaly detection threshold
    - **Property 5: Anomaly detection threshold**
    - **Validates: Requirements 5.4, 5.5**
    - Generate random accelerometer reading arrays
    - Verify abnormal flag matches presence of values exceeding ±3
    - Run 100 iterations minimum

  - [ ]* 2.5 Write property test for maximum value calculation
    - **Property 6: Maximum value calculation**
    - **Validates: Requirements 5.2**
    - Generate random accelerometer datasets
    - Verify calculated max values are truly the maximum across all readings
    - Run 100 iterations minimum

- [ ] 3. Implement authentication and API tests
  - [ ] 3.1 Create unit tests for API key validation
    - Test valid API key acceptance
    - Test invalid API key rejection with 401 status
    - Test missing API key rejection with 401 status
    - Test that valid requests proceed to handler
    - _Requirements: 1.4_

  - [ ]* 3.2 Write property test for authentication enforcement
    - **Property 1: Authentication enforcement**
    - **Validates: Requirements 1.4**
    - Generate random request objects with/without valid API keys
    - Verify 401 status for invalid/missing keys
    - Verify non-401 status for valid keys
    - Run 100 iterations minimum

- [ ] 4. Implement configuration and initialization tests
  - [ ] 4.1 Create unit tests for LLM configuration
    - Test Ollama initialization with correct model name
    - Verify temperature, maxRetries, keepAlive, numCtx parameters
    - Mock ChatOllama constructor to verify parameters
    - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5_

  - [ ]* 4.2 Write property test for LLM configuration consistency
    - **Property 2: LLM configuration consistency**
    - **Validates: Requirements 2.1, 2.2, 2.3, 2.4, 2.5**
    - Test multiple initialization scenarios
    - Verify configuration parameters remain consistent
    - Run 100 iterations minimum

  - [ ] 4.3 Create unit tests for wallet client setup
    - Test wallet client creation with private keys
    - Verify Base chain configuration
    - Verify HTTP transport setup
    - Mock viem functions to avoid actual blockchain calls
    - _Requirements: 3.1, 3.2, 3.3, 3.4_

  - [ ]* 4.4 Write property test for wallet client initialization
    - **Property 3: Wallet client initialization**
    - **Validates: Requirements 3.1, 3.2, 3.3, 3.4**
    - Verify exactly two wallet clients are created
    - Verify correct configuration for each
    - Run 100 iterations minimum

- [ ] 5. Implement workflow and routing tests
  - [ ] 5.1 Create unit tests for LangGraph routing logic
    - Test `shouldContinue` function with tool_calls present
    - Test `shouldContinue` function with no tool_calls
    - Verify routing to "tool" node when tool_calls exist
    - Verify routing to END when no tool_calls
    - _Requirements: 7.4, 7.5_

  - [ ]* 5.2 Write property test for tool routing correctness
    - **Property 7: Tool routing correctness**
    - **Validates: Requirements 7.4, 7.5**
    - Generate random model responses with/without tool_calls
    - Verify correct routing decision
    - Run 100 iterations minimum

  - [ ] 5.3 Create unit tests for response extraction
    - Test extraction from tool messages with JSON content
    - Test extraction from tool messages with plain text
    - Test extraction from model messages when no tool messages
    - Test JSON parsing error handling
    - _Requirements: 8.5, 8.6, 8.7_

  - [ ]* 5.4 Write property test for response extraction priority
    - **Property 9: Response extraction priority**
    - **Validates: Requirements 8.5, 8.6, 8.7**
    - Generate various message state configurations
    - Verify tool messages prioritized over model messages
    - Verify JSON parsing when applicable
    - Run 100 iterations minimum

- [ ] 6. Implement thread ID and state management tests
  - [ ] 6.1 Create unit tests for thread ID generation
    - Test that generated IDs are valid UUID v4 format
    - Test that multiple calls generate different IDs
    - Use uuid validation library
    - _Requirements: 7.1_

  - [ ]* 6.2 Write property test for thread ID uniqueness
    - **Property 12: Thread ID uniqueness**
    - **Validates: Requirements 7.1**
    - Generate large number of thread IDs (100-1000)
    - Verify all are valid UUID v4
    - Verify all are unique
    - Run 100 iterations minimum

  - [ ]* 6.3 Write integration test for conversation state persistence
    - **Property 8: Conversation state persistence**
    - **Validates: Requirements 7.7**
    - Send multiple messages with same thread_id
    - Verify message history accumulates correctly
    - Verify MemorySaver maintains state across invocations

- [ ] 7. Implement error handling tests
  - [ ] 7.1 Create unit tests for tool error handling
    - Test error catching in fetchIoTSensorData
    - Test error catching in fetchIoTSensorTop10Analysis
    - Mock fetch failures and verify error responses
    - Verify error logging occurs
    - _Requirements: 10.1, 10.2, 10.3_

  - [ ]* 7.2 Write property test for error handling graceful degradation
    - **Property 10: Error handling graceful degradation**
    - **Validates: Requirements 10.1, 10.2, 10.3**
    - Simulate various tool execution failures
    - Verify structured error responses returned
    - Verify application doesn't crash
    - Run 100 iterations minimum

- [ ] 8. Implement health check tests
  - [ ] 8.1 Create unit tests for health check logic
    - Test health check makes request to PERMANENT_URL
    - Test health check includes X-API-Key header
    - Test LLM health verification on API success
    - Mock fetch and LLM invoke calls
    - _Requirements: 9.2, 9.3, 9.4, 9.5_

  - [ ]* 8.2 Write property test for health check completeness
    - **Property 11: Health check completeness**
    - **Validates: Requirements 9.4, 9.5**
    - Verify both API and LLM are tested
    - Verify success only reported when both pass
    - Run 100 iterations minimum

- [ ] 9. Checkpoint - Ensure all tests pass
  - Run complete test suite
  - Verify all property tests run minimum 100 iterations
  - Fix any failing tests
  - Ensure test coverage meets requirements
  - Ask the user if questions arise

---

## Notes

- The implementation is feature-complete and matches the design document
- All 10 requirements have been implemented in the code
- Testing tasks focus on validation and correctness verification
- Property-based tests use fast-check library with 100+ iterations
- Optional tasks (marked with *) provide comprehensive test coverage
- Core functionality is working; tests ensure long-term correctness
