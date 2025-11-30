# D.I.O.T Protocol

<img src="./assets/logoS.png">

## Decentralized Internet of Things Protocol

> A machine-to-machine payments and data-exchange network powered by autonomous agents, x402 messaging, and on-chain verification. Where machines pay machines for truth.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Built with Kiro](https://img.shields.io/badge/Built%20with-Kiro-blue)](https://kiro.ai)

# FAST LINKS: 

🔗 [Live Demo](https://diot-client.expo.app) · 📁 [Kiro Config](./.kiro) · 🛠️ [Development Guide](#built-with-kiro-ai)

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

## Components

### diot-api — Data Marketplace Backend
> Node.js · Express · x402-express · LowDB · CDP Facilitator

The marketplace coordinator that verifies payments and facilitates data exchange.

```javascript
// x402 payment middleware - pay per endpoint
const x402RouteConfigs = {
  "GET /api/sensors/latest": { price: "$0.001", network: "base" },
  "GET /api/sensors/latestTop": { price: "$0.005", network: "base" }
};
app.use(paymentMiddleware(RECEIVING_ADDRESS, x402RouteConfigs, facilitator));
```

| Endpoint | Price | Description |
|----------|-------|-------------|
| `GET /api/sensors` | Free | List all sensors |
| `GET /api/sensors/latest?address=0x...` | $0.001 | Latest reading |
| `GET /api/sensors/latestTop?address=0x...` | $0.005 | Last 10 readings |

📄 [Full Code](./diot-api/index.js)

---

### diot-agent — Autonomous Economic Agent
> LangChain · LangGraph · Ollama · x402-fetch · viem · CDP Facilitator

An AI that operates its own wallet, autonomously deciding when to purchase data based on user queries.

**Workflow:** User asks → Agent selects tool → Pays via x402 → Analyzes data → Responds

```javascript
// Autonomous payment execution
const fetchWithPay = wrapFetchWithPayment(fetch, walletClient);
const response = await fetchWithPay(`${API_URL}/api/sensors/latest?address=${address}`);

// Built-in anomaly detection
if (Math.abs(x) > 3 || Math.abs(y) > 3 || Math.abs(z) > 3) {
  alert = "⚠️ ALERT: Abnormal vibration detected!";
}
```

📄 [Full Code](./diot-agent/index.js)

---

### diot-device — Data Provider App
> React Native · Expo · expo-sensors

Transforms smartphones into autonomous data providers that earn revenue.

```javascript
// Collect and publish sensor data every 10 seconds
Accelerometer.addListener((data) => {
  const filtered = applyLowPassFilter(data, gravity, 0.8);
});

setInterval(() => {
  fetch(API_URL, {
    method: "POST",
    body: JSON.stringify({ address: SENSOR_ADDRESS, data: { accel, gyro, magnet } })
  });
}, 10000);
```

📄 [Full Code](./diot-device/src/app/index.js)

---

### diot-client — Chat Interface
> React Native · Expo · React Native Paper

Natural language interface that abstracts blockchain complexity from end users.

```javascript
// Simple chat API
const response = await fetch("/api/chatWithAgent", {
  method: "POST",
  body: JSON.stringify({ message, context: { address } })
});
```

📄 [Full Code](./diot-client/src/app/index.js)

---

## Built with Kiro AI

> **2 weeks from concept to production** — 3,700+ lines of code, 85% AI-generated

### Development Approach

| Method | Use Case | Example |
|--------|----------|---------|
| **Spec-Driven** | Complex features | 50+ requirements with acceptance criteria |
| **Vibe Coding** | Rapid prototyping | "Build an AI agent that queries IoT sensors" → 400+ lines generated |

### Kiro Features Used

**🪝 Agent Hooks** — 6 automated quality checks (security scan, lint, error handling, performance, complexity, commit messages). *Impact: 20 min manual review → 30 seconds*

**📋 Steering Documents** — 5 always-included context docs eliminated repetitive explanations. *Result: 90% reduction in "how do I..." questions*

**🔌 MCP Integration** — Coinbase Developer Platform server for live blockchain documentation

```json
// .kiro/settings/mcp.json
{ "mcpServers": { "coinbase-mcp": { "url": "https://docs.cdp.coinbase.com/mcp" } } }
```

### Development Metrics

| Component | Lines | AI Generated | Time Saved |
|-----------|-------|--------------|------------|
| diot-agent | 600 | 95% | 12h |
| diot-api | 400 | 90% | 8h |
| diot-client | 800 | 80% | 10h |
| diot-device | 700 | 80% | 10h |
| diot-iot | 200 | 70% | 4h |
| **Total** | **3,700+** | **85%** | **55h** |

### Kiro Configuration

```
.kiro/
├── specs/          # 5 requirement documents (AI Agent, Client, IoT, Device, API)
├── steering/       # 5 context docs (project, workflow, agent, standards, x402)
├── hooks/          # 6 quality automation hooks
└── settings/       # MCP server configuration
```

### Key Learnings

| Insight | Impact |
|---------|--------|
| Write steering docs first | 1 hour saved 10+ hours of explanations |
| Start vibe, then spec | Creativity + structure combined |
| Trust AI output | 85% production-ready with good context |
| Build hooks early | Minutes saved per use compound to hours |
| Context is everything | Open files + steering = dramatically better suggestions |

---

## License

MIT License — see [LICENSE](LICENSE) for details.

## Acknowledgments

[Kiro AI](https://kiro.ai) · [Coinbase x402](https://github.com/coinbase/x402) · [Ollama](https://ollama.ai) · [LangChain](https://langchain.com)

---

<p align="center"><b>Built for the Kiro Hackathon 2025</b></p>
