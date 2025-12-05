# D.I.O.T AI Agent API - Design Document

## Overview

The D.I.O.T AI Agent API is a conversational AI service built with LangChain/LangGraph that enables natural language interaction with IoT sensor data. The system uses Ollama for local LLM inference (llama3.1:8b model) and integrates x402 micropayments for accessing sensor data endpoints. The agent provides intelligent analysis, anomaly detection, and user-friendly data presentation through a REST API.

**Key Design Decisions:**
- **Local LLM with Ollama**: Chosen for privacy, cost-effectiveness, and low latency compared to cloud-based LLMs
- **LangGraph for orchestration**: Provides stateful workflow management with clear tool routing and conversation memory
- **x402 micropayments**: Enables pay-per-use model for IoT data access, aligning costs with actual usage
- **Tool-based architecture**: Separates concerns between model reasoning and data fetching, making the system extensible

## Architecture

### High-Level Architecture

```
┌─────────────┐
│   Client    │
│ Application │
└──────┬──────┘
       │ HTTP POST /api/chat
       │ (message + context)
       ▼
┌─────────────────────────────────────────┐
│         Express API Server              │
│  ┌───────────────────────────────────┐  │
│  │   Authentication Middleware       │  │
│  │   (X-API-Key validation)          │  │
│  └───────────────┬───────────────────┘  │
│                  ▼                       │
│  ┌───────────────────────────────────┐  │
│  │      Agent Orchestrator           │  │
│  │  ┌─────────────────────────────┐  │  │
│  │  │   LangGraph StateGraph      │  │  │
│  │  │                             │  │  │
│  │  │  ┌──────────┐  ┌─────────┐ │  │  │
│  │  │  │  Model   │→ │  Tool   │ │  │  │
│  │  │  │   Node   │  │  Node   │ │  │  │
│  │  │  └──────────┘  └─────────┘ │  │  │
│  │  │       ↓             ↓       │  │  │
│  │  │  ┌─────────────────────┐   │  │  │
│  │  │  │     End Node        │   │  │  │
│  │  │  └─────────────────────┘   │  │  │
│  │  └─────────────────────────────┘  │  │
│  └───────────────┬───────────────────┘  │
│                  │                       │
│  ┌───────────────▼───────────────────┐  │
│  │      Ollama LLM (llama3.1:8b)    │  │
│  └───────────────────────────────────┘  │
│                                          │
│  ┌───────────────────────────────────┐  │
│  │         Tool Registry             │  │
│  │  • fetchIoTSensorData             │  │
│  │  • fetchIoTSensorTop10Analysis    │  │
│  │  • fallback                       │  │
│  └───────────────┬───────────────────┘  │
└────────────────────┬─────────────────────┘
                     │
                     │ x402 Payments
                     ▼
          ┌──────────────────────┐
          │   diot-api Server    │
          │  (Sensor Data API)   │
          └──────────────────────┘
```

### Component Interaction Flow

1. **Request Reception**: Express server receives POST request with message and optional context
2. **Authentication**: Middleware validates X-API-Key header
3. **State Initialization**: Create unique thread_id and initialize conversation state
4. **Model Invocation**: LangGraph routes to model node with system prompt and user message
5. **Tool Selection**: LLM decides which tool to invoke (or END if no tool needed)
6. **Tool Execution**: Selected tool executes with x402 payment for API calls
7. **Response Assembly**: Final response extracted from tool/model messages
8. **Client Response**: JSON response returned to client

## Components and Interfaces

### 1. Express API Server

**Responsibilities:**
- HTTP request handling and routing
- Authentication and authorization
- Request/response formatting
- Health monitoring

**Key Endpoints:**
- `GET /` - Health check endpoint
- `POST /api/chat` - Main conversational interface

**Interface:**
```javascript
// Request
POST /api/chat
Headers: {
  "X-API-Key": "<api_key>",
  "Content-Type": "application/json"
}
Body: {
  "message": "What's the latest reading from sensor 0x123...?",
  "context": "optional context string"
}

// Response
{
  "response": "The latest reading shows temperature at 22.5°C...",
  "thread_id": "uuid-v4-string"
}
```

**Design Rationale:**
- Simple REST API for easy integration with any client
- API key authentication provides basic security without complex OAuth flows
- JSON format for structured data exchange

### 2. LLM Configuration Module

**Responsibilities:**
- Initialize and configure Ollama LLM
- Manage model parameters for optimal performance
- Handle model lifecycle (keep-alive, retries)

**Configuration:**
```javascript
{
  model: "llama3.1:8b",
  temperature: 0.1,        // Low for consistent, deterministic responses
  maxRetries: 2,           // Fault tolerance for transient failures
  keepAlive: "24h",        // Keep model in memory for fast responses
  numCtx: 25600            // Large context window for conversation history
}
```

**Design Rationale:**
- **Low temperature (0.1)**: Ensures consistent, factual responses for sensor data queries
- **Large context window (25,600 tokens)**: Supports multi-turn conversations with full history
- **24-hour keep-alive**: Balances memory usage with response latency
- **llama3.1:8b**: Good balance of capability and performance for local inference

### 3. Wallet Client Manager

**Responsibilities:**
- Initialize viem wallet clients for x402 payments
- Manage multiple wallet accounts
- Provide payment-enabled fetch wrapper

**Interface:**
```javascript
// Wallet Client Creation
const walletClient = createWalletClient({
  account: privateKeyToAccount(AGENT_PRIVATE_KEY),
  chain: base,
  transport: http()
});

// Payment-Enabled Fetch
const fetchWithPay = wrapFetchWithPayment(fetch, walletClient);
```

**Design Rationale:**
- **Multiple wallets**: Allows load balancing and separation of concerns for different endpoints
- **Base network**: Low transaction fees suitable for micropayments
- **viem library**: Modern, type-safe Ethereum library with excellent developer experience

### 4. IoT Sensor Tools

#### 4.1 fetchIoTSensorData Tool

**Purpose**: Retrieve and format the latest sensor reading for a given address

**Input Schema:**
```javascript
{
  address: z.string().describe("The blockchain address of the IoT sensor")
}
```

**Output Format:**
```json
{
  "status": "success",
  "message": "Latest reading: Temperature: 22.5°C, Humidity: 45%, Pressure: 1013 hPa, TVOC: 120 ppb, eCO2: 400 ppm, Accelerometer: x=0.1, y=-0.2, z=9.8",
  "data": { /* raw sensor data */ }
}
```

**Processing Logic:**
1. Validate address parameter
2. Make x402-paid request to `/api/sensors/latest?address={address}`
3. Parse response and extract sensor readings
4. Format values with appropriate units
5. Return structured summary

**Design Rationale:**
- **Unit formatting**: Makes data immediately understandable to users
- **Structured response**: Allows both human-readable message and machine-parseable data
- **Error handling**: Graceful degradation with descriptive error messages

#### 4.2 fetchIoTSensorTop10Analysis Tool

**Purpose**: Analyze last 10 readings for anomaly detection

**Input Schema:**
```javascript
{
  address: z.string().describe("The blockchain address of the IoT sensor")
}
```

**Output Format:**
```json
{
  "status": "alert" | "normal" | "error",
  "message": "Analysis of 10 readings: Max values x=2.1, y=4.5, z=10.2. ALERT: Abnormal readings detected!",
  "maxValues": { "x": 2.1, "y": 4.5, "z": 10.2 },
  "abnormalDetected": true,
  "readingCount": 10
}
```

**Processing Logic:**
1. Fetch last 10 readings via x402-paid request
2. Parse accelerometer values (handle string/int/float types)
3. Calculate maximum absolute values for x, y, z axes
4. Check if any value exceeds ±3 threshold
5. Generate alert or confirmation message
6. Return analysis with structured data

**Design Rationale:**
- **Threshold of ±3**: Reasonable default for detecting excessive vibration/movement
- **Maximum value tracking**: Identifies worst-case scenarios across the dataset
- **Type flexibility**: Handles various data formats from different sensor implementations
- **Clear alerting**: Binary normal/abnormal classification for easy interpretation

#### 4.3 Fallback Tool

**Purpose**: Provide friendly greeting and usage guidance

**Input Schema:**
```javascript
{} // No parameters required
```

**Output:**
```json
{
  "status": "success",
  "message": "Hello! I'm the DIOT agent. I can help you check sensor readings and analyze IoT data. Just ask me about a sensor address!"
}
```

**Design Rationale:**
- **User onboarding**: Helps new users understand capabilities
- **Graceful handling**: Prevents awkward responses to simple greetings
- **Inviting tone**: Encourages further interaction

### 5. LangGraph State Management

**State Structure:**
```javascript
// Uses MessagesAnnotation for automatic message history management
{
  messages: [
    { role: "system", content: "You are the DIOT agent..." },
    { role: "user", content: "What's the latest reading?" },
    { role: "assistant", content: "...", tool_calls: [...] },
    { role: "tool", content: "{...}", tool_call_id: "..." }
  ]
}
```

**Graph Structure:**
```
START → model_node → [tool_node | END]
                ↓
         tool_node → end_node → END
```

**Routing Logic:**
- **After model_node**: If tool_calls exist → route to tool_node, else → END
- **After tool_node**: Always route to end_node
- **After end_node**: Always route to END

**Design Rationale:**
- **MessagesAnnotation**: Simplifies state management with built-in message handling
- **Linear flow with conditional branching**: Easy to understand and debug
- **MemorySaver checkpointer**: Enables conversation continuity across requests
- **Unique thread_id per conversation**: Isolates user sessions

### 6. Agent Orchestrator

**Responsibilities:**
- Coordinate workflow execution
- Bind tools to LLM
- Extract and format final responses
- Manage conversation context

**Workflow:**
1. Generate unique thread_id (UUID v4)
2. Create system message with agent identity
3. Append user message to state
4. Invoke StateGraph with configurable thread_id
5. Extract response from final state
6. Parse JSON tool responses or return model text

**Response Extraction Logic:**
```javascript
// Priority order:
1. Tool messages with JSON content → parse and return
2. Tool messages with text content → return as-is
3. Model messages → return last message content
```

**Design Rationale:**
- **System message per request**: Ensures consistent agent behavior
- **Tool binding**: Makes all tools available to LLM for selection
- **Flexible response handling**: Supports both structured and unstructured responses
- **Thread-based isolation**: Prevents cross-contamination between conversations

## Data Models

### Request Model
```typescript
interface ChatRequest {
  message: string;      // User's natural language query
  context?: string;     // Optional additional context
}
```

### Response Model
```typescript
interface ChatResponse {
  response: string;     // Agent's response (text or JSON string)
  thread_id: string;    // Conversation identifier
}
```

### Sensor Data Model
```typescript
interface SensorReading {
  address: string;
  timestamp: number;
  temperature?: number;  // °C
  humidity?: number;     // %
  pressure?: number;     // hPa
  tvoc?: number;         // ppb (Total Volatile Organic Compounds)
  eco2?: number;         // ppm (equivalent CO2)
  accelerometer?: {
    x: number | string;
    y: number | string;
    z: number | string;
  };
}
```

### Tool Response Model
```typescript
interface ToolResponse {
  status: "success" | "error" | "alert" | "normal";
  message: string;
  data?: any;           // Optional structured data
  maxValues?: {         // For analysis tool
    x: number;
    y: number;
    z: number;
  };
  abnormalDetected?: boolean;
  readingCount?: number;
}
```

### LangGraph State Model
```typescript
interface AgentState {
  messages: Array<{
    role: "system" | "user" | "assistant" | "tool";
    content: string;
    tool_calls?: Array<{
      id: string;
      name: string;
      args: Record<string, any>;
    }>;
    tool_call_id?: string;
  }>;
}
```

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property 1: Authentication enforcement
*For any* request to protected endpoints, if the X-API-Key header is missing or invalid, the system should reject the request with status code 401 and not process the message.
**Validates: Requirements 1.4**

### Property 2: LLM configuration consistency
*For any* application startup, the Ollama LLM should be initialized with the exact configuration parameters (model: llama3.1:8b, temperature: 0.1, maxRetries: 2, keepAlive: 24h, numCtx: 25600).
**Validates: Requirements 2.1, 2.2, 2.3, 2.4, 2.5**

### Property 3: Wallet client initialization
*For any* application startup, exactly two wallet clients should be created with the configured private keys and Base chain configuration.
**Validates: Requirements 3.1, 3.2, 3.3, 3.4**

### Property 4: Sensor data unit formatting
*For any* successful sensor data retrieval, the formatted response should include the correct unit suffix for each sensor type (°C for temperature, % for humidity, hPa for pressure, ppb for TVOC, ppm for eCO2).
**Validates: Requirements 4.3, 4.4, 4.5, 4.6, 4.7**

### Property 5: Anomaly detection threshold
*For any* set of accelerometer readings, if any axis value exceeds ±3, the analysis should flag the reading as abnormal and return an alert status.
**Validates: Requirements 5.4, 5.5**

### Property 6: Maximum value calculation
*For any* set of accelerometer readings, the calculated maximum values for x, y, and z axes should be the absolute maximum across all readings in the dataset.
**Validates: Requirements 5.2**

### Property 7: Tool routing correctness
*For any* model response containing tool_calls, the LangGraph workflow should route to the tool_node; for any model response without tool_calls, it should route to END.
**Validates: Requirements 7.4, 7.5**

### Property 8: Conversation state persistence
*For any* conversation with a given thread_id, subsequent messages should have access to the complete message history from previous interactions.
**Validates: Requirements 7.7**

### Property 9: Response extraction priority
*For any* completed workflow, the response extraction should prioritize tool messages over model messages, and parse JSON content when present.
**Validates: Requirements 8.5, 8.6, 8.7**

### Property 10: Error handling graceful degradation
*For any* tool execution failure, the system should catch the error, log it, and return a structured error response without crashing the application.
**Validates: Requirements 10.1, 10.2, 10.3**

### Property 11: Health check completeness
*For any* health check execution, the system should test both the API endpoint availability and the LLM responsiveness before reporting success.
**Validates: Requirements 9.4, 9.5**

### Property 12: Thread ID uniqueness
*For any* new conversation, the generated thread_id should be a valid UUID v4 and unique across all concurrent conversations.
**Validates: Requirements 7.1**

## Error Handling

### Error Categories

1. **Authentication Errors**
   - Missing X-API-Key header
   - Invalid API key
   - Response: 401 Unauthorized with warning log

2. **Validation Errors**
   - Invalid sensor address format
   - Missing required parameters
   - Response: 400 Bad Request with descriptive message

3. **External API Errors**
   - x402 payment failure
   - Sensor data API unavailable
   - Network timeout
   - Response: Tool returns error status with message, conversation continues

4. **LLM Errors**
   - Model inference failure
   - Context window exceeded
   - Response: Retry up to maxRetries (2), then return error message

5. **Data Parsing Errors**
   - Invalid JSON in tool response
   - Missing expected fields
   - Type conversion failures
   - Response: Return partial data with error note

### Error Handling Strategy

**Principle**: Fail gracefully and provide actionable feedback

```javascript
// Tool-level error handling
try {
  const response = await fetchWithPay(url);
  const data = await response.json();
  return formatSuccess(data);
} catch (error) {
  console.error(`Tool error: ${error.message}`);
  return JSON.stringify({
    status: "error",
    message: `Failed to fetch sensor data: ${error.message}`
  });
}
```

**Logging Strategy:**
- All errors logged to console with context
- Request/response pairs logged for debugging
- Tool invocations logged with parameters
- Health check results logged periodically

**Recovery Mechanisms:**
- LLM retries (maxRetries: 2)
- Graceful degradation (return partial data)
- User-friendly error messages
- Conversation continues despite tool failures

## Testing Strategy

### Unit Testing

**Framework**: Jest or Mocha with Chai

**Test Coverage:**

1. **Tool Function Tests**
   - Test sensor data formatting with mock API responses
   - Test anomaly detection logic with various accelerometer values
   - Test error handling with failed API calls
   - Test fallback tool response format

2. **Wallet Client Tests**
   - Test wallet initialization with valid private keys
   - Test payment wrapper integration
   - Mock blockchain interactions

3. **Response Extraction Tests**
   - Test JSON parsing from tool messages
   - Test text extraction from model messages
   - Test priority ordering (tool > model)

4. **Authentication Tests**
   - Test valid API key acceptance
   - Test invalid API key rejection
   - Test missing API key handling

### Property-Based Testing

**Framework**: fast-check (JavaScript property-based testing library)

**Configuration**: Each property test should run a minimum of 100 iterations to ensure statistical confidence in correctness.

**Test Tagging**: Each property-based test must include a comment with the format:
```javascript
// Feature: diot-ai-agent-api, Property X: [property description]
```

**Property Tests:**

1. **Property 1: Authentication enforcement**
   ```javascript
   // Feature: diot-ai-agent-api, Property 1: Authentication enforcement
   fc.assert(
     fc.property(
       fc.record({ message: fc.string(), apiKey: fc.option(fc.string()) }),
       (request) => {
         const response = makeRequest(request);
         if (!request.apiKey || !isValidKey(request.apiKey)) {
           return response.status === 401;
         }
         return response.status !== 401;
       }
     ),
     { numRuns: 100 }
   );
   ```

2. **Property 4: Sensor data unit formatting**
   ```javascript
   // Feature: diot-ai-agent-api, Property 4: Sensor data unit formatting
   fc.assert(
     fc.property(
       fc.record({
         temperature: fc.option(fc.float()),
         humidity: fc.option(fc.float()),
         pressure: fc.option(fc.float()),
         tvoc: fc.option(fc.integer()),
         eco2: fc.option(fc.integer())
       }),
       (sensorData) => {
         const formatted = formatSensorData(sensorData);
         if (sensorData.temperature !== undefined) {
           assert(formatted.includes('°C'));
         }
         if (sensorData.humidity !== undefined) {
           assert(formatted.includes('%'));
         }
         if (sensorData.pressure !== undefined) {
           assert(formatted.includes('hPa'));
         }
         if (sensorData.tvoc !== undefined) {
           assert(formatted.includes('ppb'));
         }
         if (sensorData.eco2 !== undefined) {
           assert(formatted.includes('ppm'));
         }
         return true;
       }
     ),
     { numRuns: 100 }
   );
   ```

3. **Property 5: Anomaly detection threshold**
   ```javascript
   // Feature: diot-ai-agent-api, Property 5: Anomaly detection threshold
   fc.assert(
     fc.property(
       fc.array(
         fc.record({
           x: fc.float({ min: -10, max: 10 }),
           y: fc.float({ min: -10, max: 10 }),
           z: fc.float({ min: -10, max: 10 })
         }),
         { minLength: 1, maxLength: 10 }
       ),
       (readings) => {
         const analysis = analyzeAccelerometer(readings);
         const hasAbnormal = readings.some(
           r => Math.abs(r.x) > 3 || Math.abs(r.y) > 3 || Math.abs(r.z) > 3
         );
         return analysis.abnormalDetected === hasAbnormal;
       }
     ),
     { numRuns: 100 }
   );
   ```

4. **Property 7: Tool routing correctness**
   ```javascript
   // Feature: diot-ai-agent-api, Property 7: Tool routing correctness
   fc.assert(
     fc.property(
       fc.record({
         content: fc.string(),
         tool_calls: fc.option(fc.array(fc.object()))
       }),
       (modelResponse) => {
         const nextNode = routeAfterModel(modelResponse);
         if (modelResponse.tool_calls && modelResponse.tool_calls.length > 0) {
           return nextNode === 'tool_node';
         } else {
           return nextNode === 'END';
         }
       }
     ),
     { numRuns: 100 }
   );
   ```

5. **Property 12: Thread ID uniqueness**
   ```javascript
   // Feature: diot-ai-agent-api, Property 12: Thread ID uniqueness
   fc.assert(
     fc.property(
       fc.integer({ min: 100, max: 1000 }),
       (numThreads) => {
         const threadIds = new Set();
         for (let i = 0; i < numThreads; i++) {
           const id = generateThreadId();
           assert(isValidUUIDv4(id));
           threadIds.add(id);
         }
         return threadIds.size === numThreads; // All unique
       }
     ),
     { numRuns: 100 }
   );
   ```

### Integration Testing

1. **End-to-End Workflow Tests**
   - Test complete request flow from API to response
   - Test tool invocation with mocked external APIs
   - Test conversation continuity across multiple messages

2. **x402 Payment Integration**
   - Test payment wrapper with test wallets
   - Verify payment amounts match configured prices
   - Test payment failure handling

3. **Health Check Tests**
   - Test periodic health check execution
   - Test API and LLM availability checks
   - Test health check logging

### Manual Testing Scenarios

1. **Sensor Data Queries**
   - "What's the latest reading from sensor 0x123...?"
   - "Analyze the last 10 readings for sensor 0xabc..."
   - "Show me temperature data for 0x456..."

2. **Conversation Flow**
   - Multi-turn conversations with context
   - Tool selection accuracy
   - Response quality and formatting

3. **Error Scenarios**
   - Invalid sensor addresses
   - Network failures
   - Payment failures
   - Invalid API keys

4. **Performance**
   - Response latency under load
   - Memory usage with long conversations
   - Model keep-alive effectiveness

## Deployment Considerations

### Environment Variables
```
AGENT_PRIVATE_KEY1=<wallet_private_key>
AGENT_PRIVATE_KEY2=<wallet_private_key>
FETCH_URL=<diot_api_base_url>
AI_URL_API_KEY=<api_key>
PERMANENT_URL=<public_health_check_url>
PORT=8000
```

### Dependencies
- Node.js 18+ (for native fetch support)
- Ollama installed and running locally
- llama3.1:8b model pulled (`ollama pull llama3.1:8b`)
- Funded Base network wallets for x402 payments

### Scaling Considerations
- **Horizontal scaling**: Stateless design allows multiple instances behind load balancer
- **Memory management**: 24-hour keep-alive balances performance and resource usage
- **Rate limiting**: Consider adding rate limits per API key
- **Conversation cleanup**: Implement TTL for old thread_ids in MemorySaver

### Monitoring
- Health check endpoint for uptime monitoring
- Log aggregation for error tracking
- Wallet balance monitoring for payment capability
- LLM response time tracking

### Security
- API keys stored in environment variables
- Private keys never logged or exposed
- Input validation on all user-provided data
- HTTPS recommended for production deployment
