# AI Agent Guidelines

## Agent Architecture
The DIOT agent uses LangGraph with a tool-based workflow:
1. User sends message → Model node
2. Model decides which tool to call (or END)
3. Tool executes and returns result
4. Result flows to end node → Response sent

## Available Tools

### fetchIoTSensorData
- Retrieves latest sensor reading by address
- Uses x402 payment ($0.001)
- Returns formatted sensor data summary
- Handles temperature, humidity, pressure, TVOC, eCO2, accelerometer

### fetchIoTSensorTop10Analysis
- Retrieves last 10 readings by address
- Uses x402 payment ($0.005)
- Analyzes accelerometer data for anomalies
- Alerts if any axis exceeds ±3 (abnormal vibration/movement)

### fallback
- Activates on simple greetings
- Provides friendly welcome message

## Tool Development Best Practices
- Use zod schemas for input validation
- Return JSON strings with `status` and `message` fields
- Handle errors gracefully with try-catch
- Add descriptive tool descriptions for model selection
- Format data for human readability in responses

## Model Configuration
- Model: Ollama llama3.1:8b
- Temperature: 0.1 (low for consistency)
- Context window: 25,600 tokens
- Keep alive: 24 hours
- System prompt: Present as "DIOT agent", friendly and knowledgeable

## Adding New Tools
1. Define tool function with async logic
2. Wrap with `tool()` from @langchain/core/tools
3. Add zod schema for parameters
4. Add to `all_api_tools` array
5. Update tool descriptions to guide model selection

## Memory & Context
- Uses MemorySaver for conversation history
- Each request gets unique thread_id via uuidv4
- Context data can be passed via `context` field in API request
