# Requirements Document

## Introduction

The D.I.O.T AI Agent API is a conversational AI service that enables users to interact with IoT sensor data through natural language. The system integrates LangChain/LangGraph for agent orchestration, Ollama for local LLM inference, and x402 micropayments for accessing sensor data endpoints. The agent provides intelligent analysis of sensor readings, detects anomalies, and presents data in a user-friendly format through a REST API.

## Glossary

- **D.I.O.T Agent**: The AI-powered conversational agent that processes user queries and interacts with IoT sensors
- **LangGraph**: A framework for building stateful, multi-actor applications with LLMs
- **Ollama**: Local LLM inference engine running the llama3.1:8b model
- **x402 Payment**: Blockchain-based micropayment protocol for API access using viem wallet clients
- **Tool Node**: A LangGraph component that executes specific functions (tools) based on LLM decisions
- **State Graph**: The workflow definition that orchestrates agent behavior and tool execution
- **Memory Saver**: Component that maintains conversation context across interactions
- **Sensor Address**: Blockchain address or identifier for an IoT sensor device
- **Accelerometer Reading**: Three-axis (x, y, z) motion sensor data measuring acceleration
- **Abnormal Reading**: Accelerometer values exceeding ±3, indicating excessive vibration or movement

## Requirements

### Requirement 1: Express API Server

**User Story:** As a client application, I want to interact with the AI agent through a REST API, so that I can integrate conversational IoT data access into my application.

#### Acceptance Criteria

1. WHEN the server starts THEN the D.I.O.T Agent SHALL listen on the configured port (default 8000)
2. WHEN a GET request is made to the root endpoint ("/") THEN the D.I.O.T Agent SHALL return a health status response with status code 200
3. WHEN a POST request is made to "/api/chat" with a message and optional context THEN the D.I.O.T Agent SHALL process the message and return an agent response
4. WHEN the server receives a request without a valid X-API-Key header THEN the D.I.O.T Agent SHALL reject the request with status code 401
5. WHEN the server receives a request with valid authentication THEN the D.I.O.T Agent SHALL parse the JSON body and forward it to the agent

### Requirement 2: LLM Configuration and Initialization

**User Story:** As the system, I want to initialize the Ollama LLM with appropriate parameters, so that the agent can generate intelligent responses efficiently.

#### Acceptance Criteria

1. WHEN the application starts THEN the D.I.O.T Agent SHALL initialize ChatOllama with model "llama3.1:8b"
2. WHEN the LLM is configured THEN the D.I.O.T Agent SHALL set temperature to 0.1 for consistent responses
3. WHEN the LLM is configured THEN the D.I.O.T Agent SHALL set maxRetries to 2 for fault tolerance
4. WHEN the LLM is configured THEN the D.I.O.T Agent SHALL set keepAlive to "24h" to maintain model in memory
5. WHEN the LLM is configured THEN the D.I.O.T Agent SHALL set numCtx to 25600 tokens for adequate context window

### Requirement 3: Wallet Client Setup for x402 Payments

**User Story:** As the system, I want to configure blockchain wallet clients, so that I can make micropayments to access IoT sensor data endpoints.

#### Acceptance Criteria

1. WHEN the application starts THEN the D.I.O.T Agent SHALL create a wallet client for the Base chain using AGENT_PRIVATE_KEY1
2. WHEN the application starts THEN the D.I.O.T Agent SHALL create a second wallet client for the Base chain using AGENT_PRIVATE_KEY2
3. WHEN a wallet client is created THEN the D.I.O.T Agent SHALL configure it with HTTP transport
4. WHEN a wallet client is created THEN the D.I.O.T Agent SHALL derive the account from the provided private key
5. WHEN making API calls THEN the D.I.O.T Agent SHALL use the appropriate wallet client for x402 payment wrapping

### Requirement 4: IoT Sensor Data Retrieval Tool

**User Story:** As a user, I want to query the latest sensor readings by address, so that I can understand current environmental conditions from my IoT devices.

#### Acceptance Criteria

1. WHEN the user requests sensor data with a valid address THEN the D.I.O.T Agent SHALL fetch data from the "/api/sensors/latest" endpoint using x402 payments
2. WHEN sensor data is retrieved successfully THEN the D.I.O.T Agent SHALL parse and summarize the sensor readings
3. WHEN summarizing sensor data THEN the D.I.O.T Agent SHALL format temperature values with "°C" units
4. WHEN summarizing sensor data THEN the D.I.O.T Agent SHALL format humidity values with "%" units
5. WHEN summarizing sensor data THEN the D.I.O.T Agent SHALL format pressure values with "hPa" units
6. WHEN summarizing sensor data THEN the D.I.O.T Agent SHALL format TVOC values with "ppb" units
7. WHEN summarizing sensor data THEN the D.I.O.T Agent SHALL format eCO2 values with "ppm" units
8. WHEN sensor data retrieval fails THEN the D.I.O.T Agent SHALL return an error status with a descriptive message
9. WHEN no sensor data is found THEN the D.I.O.T Agent SHALL return a message indicating no readings were found

### Requirement 5: IoT Sensor Top 10 Analysis Tool

**User Story:** As a user, I want to analyze the last 10 sensor readings for anomalies, so that I can detect abnormal device behavior or environmental conditions.

#### Acceptance Criteria

1. WHEN the user requests top 10 analysis with a valid address THEN the D.I.O.T Agent SHALL fetch data from the "/api/sensors/latestTop" endpoint using x402 payments
2. WHEN analyzing the last 10 readings THEN the D.I.O.T Agent SHALL calculate maximum accelerometer values for x, y, and z axes
3. WHEN analyzing accelerometer data THEN the D.I.O.T Agent SHALL parse string, integer, and float values correctly
4. WHEN any accelerometer value exceeds ±3 THEN the D.I.O.T Agent SHALL flag the reading as abnormal
5. WHEN abnormal readings are detected THEN the D.I.O.T Agent SHALL return an alert message indicating potential issues
6. WHEN all readings are within normal range THEN the D.I.O.T Agent SHALL return a confirmation message
7. WHEN no valid accelerometer data is found THEN the D.I.O.T Agent SHALL return an error message
8. WHEN the analysis completes THEN the D.I.O.T Agent SHALL return maximum values, abnormal detection status, and reading count

### Requirement 6: Fallback Greeting Tool

**User Story:** As a user, I want to receive a friendly greeting when I say hello, so that I understand how to interact with the agent.

#### Acceptance Criteria

1. WHEN the user sends a simple greeting like "hi" or "hello" THEN the D.I.O.T Agent SHALL invoke the fallback tool
2. WHEN the fallback tool is invoked THEN the D.I.O.T Agent SHALL return a friendly welcoming message
3. WHEN the fallback tool is invoked THEN the D.I.O.T Agent SHALL invite the user to interact with available features

### Requirement 7: LangGraph State Management

**User Story:** As the system, I want to maintain conversation state and context, so that multi-turn conversations are coherent and contextually aware.

#### Acceptance Criteria

1. WHEN a conversation starts THEN the D.I.O.T Agent SHALL generate a unique thread_id using UUID v4
2. WHEN processing a message THEN the D.I.O.T Agent SHALL use MessagesAnnotation for state management
3. WHEN the agent invokes the model THEN the D.I.O.T Agent SHALL append the response to the message history
4. WHEN the model response includes tool calls THEN the D.I.O.T Agent SHALL route to the tool node
5. WHEN the model response has no tool calls THEN the D.I.O.T Agent SHALL route to the END state
6. WHEN tool execution completes THEN the D.I.O.T Agent SHALL route to the end node
7. WHEN using the checkpointer THEN the D.I.O.T Agent SHALL persist conversation state with MemorySaver

### Requirement 8: Agent Workflow Orchestration

**User Story:** As the system, I want to orchestrate the agent workflow from user input to final response, so that queries are processed correctly through model inference and tool execution.

#### Acceptance Criteria

1. WHEN a user message is received THEN the D.I.O.T Agent SHALL create a system message with agent identity and instructions
2. WHEN invoking the agent THEN the D.I.O.T Agent SHALL bind all tools to the LLM
3. WHEN the model node executes THEN the D.I.O.T Agent SHALL invoke the LLM with the current message state
4. WHEN the workflow completes THEN the D.I.O.T Agent SHALL extract the final response from tool or model messages
5. WHEN tool messages contain JSON THEN the D.I.O.T Agent SHALL parse and return the structured data
6. WHEN tool messages contain plain text THEN the D.I.O.T Agent SHALL return the text as-is
7. WHEN no tool messages exist THEN the D.I.O.T Agent SHALL return the last model message content

### Requirement 9: Health Monitoring

**User Story:** As a system administrator, I want automated health checks, so that I can ensure the API and agent are functioning correctly.

#### Acceptance Criteria

1. WHEN the application starts THEN the D.I.O.T Agent SHALL perform an initial health check
2. WHEN performing a health check THEN the D.I.O.T Agent SHALL make a request to the configured PERMANENT_URL
3. WHEN the health check request is made THEN the D.I.O.T Agent SHALL include the X-API-Key header
4. WHEN the API health check succeeds THEN the D.I.O.T Agent SHALL test the LLM with a simple message
5. WHEN the LLM responds successfully THEN the D.I.O.T Agent SHALL log the agent health result
6. WHEN the application is running THEN the D.I.O.T Agent SHALL perform health checks every 30 minutes

### Requirement 10: Error Handling and Logging

**User Story:** As a developer, I want comprehensive error handling and logging, so that I can debug issues and monitor system behavior.

#### Acceptance Criteria

1. WHEN an error occurs during sensor data fetching THEN the D.I.O.T Agent SHALL catch the error and log it to the console
2. WHEN an error occurs during sensor data fetching THEN the D.I.O.T Agent SHALL return null or an error response
3. WHEN tool execution fails THEN the D.I.O.T Agent SHALL return a JSON response with error status and message
4. WHEN unauthorized access is attempted THEN the D.I.O.T Agent SHALL log a warning message
5. WHEN a request is received THEN the D.I.O.T Agent SHALL log the message and context
6. WHEN tool calls are evaluated THEN the D.I.O.T Agent SHALL log the tool call details
7. WHEN the fallback tool is invoked THEN the D.I.O.T Agent SHALL log the invocation
