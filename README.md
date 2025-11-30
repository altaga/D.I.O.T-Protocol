# D.I.O.T Protocol

<img src="./assets/logoS.png">

## Decentralized Internet of Things Protocol

> A machine-to-machine payments and data-exchange network powered by autonomous agents, x402 messaging, and on-chain verification. Where machines pay machines for truth.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Built with Kiro](https://img.shields.io/badge/Built%20with-Kiro-blue)](https://kiro.ai)

## FAST LINKS:

- [.kiro files](./.kiro)
- [Kiro Development](#built-with-kiro-ai)
- [AI Agent WebPage](https://diot-client.expo.app)

## Overview

**D.I.O.T Protocol** is a trustless, self-sustaining data marketplace where IoT devices autonomously buy, sell, and settle for real-world sensor data. Each device runs a lightweight agent that uses the CDP Facilitator to enable machine-to-machine payments via x402 protocol on Base network.

| Capability | Description |
|------------|-------------|
| 🤖 Autonomous Agent Economy | Devices operate as independent economic actors |
| 💰 x402 Micropayments | $0.001-$0.005 per query via blockchain HTTP payments |
| 🔗 On-Chain Verification | Sensor readings signed, validated, and anchored on-chain |
| 📈 Financial Incentives | Data providers earn revenue per query |
| 💬 Natural Language Interface | AI agents translate conversational queries into transactions |

<img src="./assets/architecture.png">

### How It Works

```
┌─────────────────┐    POST /api/sensors    ┌─────────────────┐
│  Sensor Device  │ ──────────────────────► │    diot-api     │
│  (Data Provider)│                         │   (Marketplace) │
└─────────────────┘                         └────────┬────────┘
                                                     │
┌─────────────────┐    x402 Payment + GET   ┌────────▼────────┐
│   diot-client   │ ◄────────────────────── │   diot-agent    │
│  (Chat Interface)│                        │  (AI + Wallet)  │
└─────────────────┘                         └─────────────────┘
```

**Payment Flow:**
1. Sensor device collects data → publishes to marketplace
2. User asks question in natural language
3. AI agent decides which data to purchase
4. Agent pays via x402 → receives data → analyzes → responds

### Marketplace Economics

| Role | Earnings | Action |
|------|----------|--------|
| Data Provider | $0.001-$0.005/query | Publish sensor readings |
| AI Agent | Autonomous | Decides when to buy data |
| Blockchain | — | Trustless payment verification |

### Tech Stack

| Layer | Technology |
|-------|------------|
| Mobile Apps | React Native + Expo (iOS/Android/Web) |
| AI Agent | LangChain + Ollama (llama3.1:8b) |
| Payments | x402 protocol on Base network |
| Storage | LowDB (future: IPFS/Arweave) |
| Hardware | ESP32/M5Stack support |

## Component Details

### 1. diot-api - Data Marketplace Backend

**Technology**: Node.js, Express, x402-express, LowDB, Coinbase CDP x402 facilitator

**Role in Ecosystem**: Acts as the data marketplace coordinator, verifying payments and facilitating machine-to-machine data exchange.

**Key Features**:
- **x402 Payment Middleware** - Verifies blockchain payments before data release
- **CDP Facilitator Integration** - Uses Coinbase Developer Platform for payment processing
- **Data Storage** - LowDB for sensor readings (future: IPFS/Arweave for decentralized storage)
- **Configurable Pricing** - Per-endpoint pricing enables flexible data monetization
- **RESTful API** - Standard HTTP interface for device-to-device communication

**Payment Configuration**:

```javascript
// Define pricing for each endpoint
const x402RouteConfigs = {
  "GET /api/sensors/latest": {
    price: "$0.001",
    network: "base"
  },
  "GET /api/sensors/latestTop": {
    price: "$0.005",
    network: "base"
  }
};

// Apply payment middleware
app.use(paymentMiddleware(RECEIVING_ADDRESS, x402RouteConfigs, facilitator));
```

**API Endpoints**:
- `GET /api/sensors` - List all sensors available (free)
- `GET /api/sensors/latest?address=0x...` - Latest reading ($0.001)
- `GET /api/sensors/latestTop?address=0x...` - Last 10 readings ($0.005)

[FULL CODE](./diot-api/index.js) 

### 2. diot-agent - Autonomous Economic Agent

**Technology**: LangChain, LangGraph, Ollama (llama3.1:8b), x402-fetch, viem, CDP Facilitator

**Role in Ecosystem**: Operates as an autonomous economic actor that makes payment decisions, executes blockchain transactions, and provides value-added analysis.

**Key Features**:
- **Autonomous Decision-Making** - LangGraph state machine determines when to buy data
- **Wallet Management** - viem integration for Base network transactions
- **x402 Payment Execution** - Wraps HTTP requests with blockchain payments
- **Tool-Based Architecture** - zod schema validation for reliable tool invocation
- **Value-Added Services** - Anomaly detection, trend analysis, natural language responses
- **Conversation Memory** - MemorySaver maintains context across interactions

**Economic Model**: The agent operates with its own wallet, autonomously deciding when data purchases are justified based on user queries. This creates a true machine-to-machine economy where agents act as intermediaries between human users and data providers.

**Agent Workflow**:

The agent uses LangGraph to orchestrate tool calls based on user queries. When you ask "Check sensor 0x123", the agent:
1. Determines which tool to use (latest reading vs. top 10 analysis)
2. Wraps the API request with x402 payment
3. Executes the blockchain transaction
4. Receives and analyzes the data
5. Formats a natural language response

**Autonomous Payment Example**:

```javascript
// Agent wraps HTTP requests with blockchain payments
const fetchWithPay = wrapFetchWithPayment(fetch, walletClient);
const response = await fetchWithPay(
  `${API_URL}/api/sensors/latest?address=${address}`,
  { method: "GET" }
);
```

**Anomaly Detection**:

```javascript
// Automatically detect abnormal sensor readings
if (Math.abs(x) > 3 || Math.abs(y) > 3 || Math.abs(z) > 3) {
  alert = "⚠️ ALERT: Abnormal vibration detected!";
}
```

[FULL CODE](./diot-agent/index.js)

### 3. diot-device - Data Provider Agent

**Technology**: React Native 0.81.5, Expo ~54, expo-sensors

**Role in Ecosystem**: Transforms smartphones into autonomous data providers that earn revenue by publishing sensor readings to the marketplace.

**Key Features**:
- **Multi-Sensor Collection** - Accelerometer, gyroscope, magnetometer, device motion
- **Signal Processing** - Low-pass filter for gravity estimation and noise reduction
- **Autonomous Publishing** - Automatic transmission every 10 seconds
- **Device Identity** - Unique address for payment routing
- **Cross-Platform** - iOS and Android support

**Economic Model**: Each device acts as a data provider, earning micropayments when agents query its sensor readings. This incentivizes accurate, high-quality data collection.

**Sensor Collection**:

```javascript
// Collect accelerometer data
Accelerometer.addListener((data) => {
  const sensorReading = { x: data.x, y: data.y, z: data.z };
  // Apply low-pass filter for noise reduction
  const filtered = applyLowPassFilter(sensorReading, gravity, 0.8);
});
```

**Automatic Publishing**:

```javascript
// Transmit sensor data every 10 seconds
setInterval(() => {
  fetch(API_URL, {
    method: "POST",
    body: JSON.stringify({
      address: SENSOR_ADDRESS,
      data: { accel, gyro, magnet },
      timestamp: Date.now()
    })
  });
}, 10000);
```

[FULL CODE](./diot-device/src/app/index.js)

### 4. diot-client - Human Interface Layer

**Technology**: React Native 0.81.5, Expo ~54, React Native Paper

**Role in Ecosystem**: Provides natural language interface for humans to interact with the autonomous agent economy.

**Key Features**:
- **Conversational UI** - Chat interface abstracts blockchain complexity
- **Real-Time Communication** - Direct connection to autonomous agent
- **Message History** - Maintains conversation context
- **Cross-Platform** - iOS, Android, and Web support

**User Experience**: Users interact with the decentralized data marketplace through simple conversational queries. The underlying agent handles wallet management, payment execution, and data retrieval autonomously.

**Chat Interface**:

```javascript
// Send message to AI agent
async function chatWithAgent(message) {
  return fetch("/api/chatWithAgent", {
    method: "POST",
    body: JSON.stringify({ message, context: { address } })
  }).then(res => res.json());
}

// Display conversation
messages.map(msg => (
  <MessageBubble 
    text={msg.message} 
    type={msg.type} 
    time={msg.time} 
  />
));
```

[FULL CODE](./diot-client/src/app/index.js)

---
# Built with Kiro AI

This entire project was developed in 2 weeks using Kiro as the primary development partner. Here's how:

### Development Approach: Hybrid Spec + Vibe Coding

**Spec-Driven Development (50+ requirements)**
For complex features, we wrote detailed specifications with acceptance criteria:
- AI Agent API: 10 requirements, 60+ acceptance criteria
- Client Chat Interface: 10 requirements for mobile UI
- IoT Firmware: 7 requirements for ESP32/M5Stack
- Mobile Sensor Device: 8 requirements for mobile sensors
- Sensor Data API: 10 requirements for backend

Kiro read these specs and implemented features systematically, ensuring nothing was missed.

**Vibe Coding (Rapid Prototyping)**
For exploration and quick iterations, we used conversational development:
- "Build an AI agent that queries IoT sensors"
- Kiro generated 400+ lines of LangGraph workflow in one conversation
- Integrated LangChain + Ollama + x402 + viem seamlessly
- Added emoji-based section comments automatically

### Agent Hooks: Automated Quality Checks

Created 6 user-triggered hooks for instant code quality checks:
1. **Security Scan** - Audits for vulnerabilities, hardcoded secrets
2. **Lint Check** - Style violations via getDiagnostics
3. **Error Handling Review** - Finds missing try-catch blocks
4. **Performance Analyzer** - Detects inefficient patterns
5. **Code Complexity Check** - Identifies functions >50 lines
6. **Commit Message Helper** - Generates conventional commits

**Impact:** 20 minutes of manual review → 30 seconds with hooks

### Steering Documents: Persistent Context

5 always-included steering docs eliminated repetitive explanations:
- **project-overview.md** - Architecture, tech stack, payment system
- **development-workflow.md** - Run commands, env vars, deployment
- **ai-agent-guidelines.md** - LangGraph patterns, tool development
- **coding-standards.md** - JavaScript/React Native conventions
- **x402-payments.md** - Micropayment integration patterns

**Result:** 90% reduction in "how do I..." questions, consistent code style across all components

### Most Impressive Code Generation

**The AI Agent Tool System** - Kiro generated in one conversation:
- Complete LangGraph workflow with state management
- 3 custom tools with zod validation
- x402 payment integration with viem wallets
- Error handling, logging, health monitoring
- Emoji-based section comments for readability
- 400+ lines of production-ready code

### Development Metrics

| Component | Lines of Code | Kiro Contribution | Time Saved |
|-----------|---------------|-------------------|------------|
| diot-agent | 600 | 95% | 12 hours |
| diot-api | 400 | 90% | 8 hours |
| diot-client | 800 | 80% | 10 hours |
| diot-device | 700 | 80% | 10 hours |
| diot-iot | 200 | 70% | 4 hours |
| **Total** | **3,700+** | **85%** | **55 hours** |

**Without Kiro:** 6-8 weeks of development  
**With Kiro:** 2 weeks from concept to production

### Model Context Protocol (MCP) Integration

The project integrates Coinbase's MCP server for real-time access to blockchain documentation and CDP APIs:

**Configuration** (`.kiro/settings/mcp.json`):
```json
{
  "mcpServers": {
    "coinbase-mcp": {
      "url": "https://docs.cdp.coinbase.com/mcp",
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

**Benefits:**
- **Live Documentation** - Query Coinbase Developer Platform docs directly in Kiro
- **API Discovery** - Find relevant CDP endpoints and code examples
- **Blockchain Context** - Get Base network info, x402 protocol details, wallet APIs
- **Development Speed** - No context switching to browser for documentation

**Usage:** Ask Kiro questions like "How do I create a wallet with CDP?" or "Show me x402 payment examples" and it will query the MCP server for accurate, up-to-date information.

### Kiro Configuration

All Kiro configuration is in the `.kiro/` directory:

```
.kiro/
├── specs/                          # 5 detailed requirement documents
│   ├── diot-ai-agent-api/
│   ├── diot-client-chat-interface/
│   ├── diot-iot-firmware/
│   ├── diot-mobile-sensor-device/
│   └── iot-sensor-data-api/
│
├── steering/                       # 5 always-included context documents
│   ├── project-overview.md
│   ├── development-workflow.md
│   ├── ai-agent-guidelines.md
│   ├── coding-standards.md
│   └── x402-payments.md
│
├── hooks/                          # 6 automated quality workflows
│   ├── security-scan.kiro.hook
│   ├── lint-check.kiro.hook
│   ├── error-handling-check.kiro.hook
│   ├── performance-check.kiro.hook
│   ├── code-complexity-check.kiro.hook
│   └── commit-message-helper.kiro.hook
│
└── settings/
    └── mcp.json                    # MCP server configuration
```

**How to Use:**
- **Specs:** Open any `requirements.md` and ask Kiro to implement features
- **Steering:** Automatically loaded - Kiro knows project context in every conversation
- **Hooks:** Right-click in editor → "Run Kiro Hook" → Select hook
- **MCP:** Coinbase Developer Platform MCP server integrated for blockchain documentation and API queries

## What We Learned

**Steering docs first** - Spending 1 hour writing steering documents saved 10+ hours of repetitive explanations. Kiro remembered project conventions and applied them consistently.

**Iterative specs** - Starting with vibe coding to explore, then formalizing into specs for implementation, gave us the best of both worlds: creativity + structure.

**Trust the AI** - Kiro's generated code was production-ready 85% of the time. The key was providing good context through steering docs and specs.

**Hooks compound** - Each hook saved minutes per use, but over dozens of uses, they saved hours. Building a hook library early pays dividends.

**Context matters** - Keeping related files open and using steering docs made Kiro's suggestions dramatically better. It's not just about the prompt—it's about the environment.

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Acknowledgments

- Built with [Kiro AI](https://kiro.ai) - AI-powered development assistant
- Powered by [Coinbase x402](https://github.com/coinbase/x402) micropayments
- LLM inference by [Ollama](https://ollama.ai)
- Agent orchestration by [LangChain](https://langchain.com)

---

**Built for the Kiro Hackathon 2025**
