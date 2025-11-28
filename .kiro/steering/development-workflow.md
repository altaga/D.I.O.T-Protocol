# Development Workflow

## Running the Project

### Mobile Apps (diot-client, diot-device)
```bash
cd diot-client  # or diot-device
npm start       # Start Expo dev server
npm run android # Run on Android
npm run ios     # Run on iOS
npm run web     # Run in browser
```

### Backend Services
```bash
# API Server
cd diot-api
node index.js   # Runs on port 3000

# AI Agent
cd diot-agent
node index.js   # Runs on port 8000 (or PORT env var)
```

### Arduino/IoT Device
- Open `diot-iot/code.ino` in Arduino IDE
- Configure WiFi credentials and API endpoint
- Upload to ESP32/Arduino board

## Environment Variables Required

### diot-agent
- `AGENT_PRIVATE_KEY1` - Wallet private key for x402 payments
- `AGENT_PRIVATE_KEY2` - Secondary wallet for different endpoints
- `FETCH_URL` - Base URL for diot-api (e.g., http://localhost:3000/)
- `AI_URL_API_KEY` - API key for agent authentication
- `PERMANENT_URL` - Public URL for health checks
- `PORT` - Server port (default: 8000)

### diot-api
- `RECEIVING_ADDRESS` - Wallet address to receive x402 payments

## Testing
- No automated tests currently configured
- Manual testing via Expo Go app for mobile
- Use Postman/curl for API endpoint testing
- Test x402 payments with funded Base wallets

## Deployment
- Mobile apps: Use EAS (Expo Application Services)
  - `npm run webbuild` - Build for web
  - `npm run webdeploy` - Deploy to EAS
- Backend: Deploy to Ubuntu server or cloud provider
- Keep db.json persistent for lowdb data
