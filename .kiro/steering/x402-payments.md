# x402 Micropayments Integration

## What is x402?
x402 is a protocol for HTTP-based micropayments on blockchain networks. It allows APIs to charge small amounts (fractions of a cent) per request.

## How It Works in DIOT

### API Side (diot-api)
```javascript
const { paymentMiddleware } = require("x402-express");

// Define pricing for each route
const x402RouteConfigs = {
  "GET /api/sensors/latest": {
    price: "$0.001",
    network: "base",
    config: { description: "..." }
  }
};

// Apply middleware globally
app.use(paymentMiddleware(RECEIVING_ADDRESS, x402RouteConfigs, facilitator));
```

### Agent Side (diot-agent)
```javascript
import { wrapFetchWithPayment } from "x402-fetch";

// Wrap fetch with payment capability
const fetchWithPay = wrapFetchWithPayment(fetch, walletClient);

// Make paid request
const response = await fetchWithPay(url, { method: "GET" });
```

## Wallet Setup
- Uses viem for wallet management
- Base network (Coinbase L2)
- Private keys stored in environment variables
- Each agent can have multiple wallets for different endpoints

## Pricing Strategy
- **Latest reading**: $0.001 (frequent, low-cost queries)
- **Top 10 analysis**: $0.005 (more data, more processing)
- Free endpoints: `/api/sensors` (list all sensors)

## Testing Payments
1. Fund test wallets with Base ETH
2. Set AGENT_PRIVATE_KEY1/2 in environment
3. Set RECEIVING_ADDRESS in diot-api
4. Monitor wallet balances to verify payments
5. Check API logs for payment confirmations

## Adding New Paid Endpoints
1. Add route config to `x402RouteConfigs` with price
2. Define input schema for validation
3. Implement route handler after middleware
4. Create corresponding tool in agent with `wrapFetchWithPayment`
