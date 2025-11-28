# DIOT Project Overview

## Architecture
This is a decentralized IoT ecosystem with 4 main components:

1. **diot-iot** - Arduino/ESP32 hardware sensors (C++)
2. **diot-device** - React Native mobile app for IoT device management (Expo)
3. **diot-client** - React Native mobile app for end users (Expo)
4. **diot-agent** - AI agent using LangChain + Ollama (Node.js/Express)
5. **diot-api** - Backend API with x402 micropayments (Node.js/Express)

## Key Technologies
- **Frontend**: React Native 0.81.5, Expo ~54, React 19.1.0
- **Backend**: Express, LangChain, Ollama (llama3.1:8b)
- **Blockchain**: Base network (viem), x402 micropayments
- **Database**: lowdb (JSON file-based)
- **AI**: LangGraph for agent workflows, tool-based architecture

## Payment System
The project uses x402 protocol for micropayments:
- API endpoints are protected with payment middleware
- Agents use `wrapFetchWithPayment` to pay for API calls
- Pricing: $0.001 for latest reading, $0.005 for top 10 readings
- Receiving address: 0x5c2aec641952fb009aa879d411f13f274362e7ea
