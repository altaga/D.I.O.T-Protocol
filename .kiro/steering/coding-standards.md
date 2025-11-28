# Coding Standards

## JavaScript/Node.js
- Use ES6+ syntax (async/await, arrow functions, destructuring)
- Use descriptive variable names (camelCase)
- Add section comments with emoji markers (💠) for major code blocks
- Keep functions focused and single-purpose
- Handle errors with try-catch and return meaningful error messages

## React Native
- Use functional components with hooks
- Follow Expo Router conventions for file-based routing
- Keep components in `src/app/` for routes, `src/providers/` for context
- Use expo-router for navigation, not react-navigation directly
- Leverage Expo SDK modules (expo-sensors, expo-haptics, etc.)

## API Design
- RESTful endpoints with clear naming: `/api/resource/action`
- Use query parameters for filters (e.g., `?address=0x...`)
- Return consistent JSON responses with `status` and `message` fields
- Protect sensitive endpoints with API keys or x402 payments
- Log all incoming requests for debugging

## AI Agent Patterns
- Define tools with clear descriptions and zod schemas
- Use LangGraph StateGraph for multi-step workflows
- Keep system prompts concise and role-focused
- Return structured JSON from tools for easy parsing
- Use MemorySaver for conversation context

## Security
- Never commit private keys or API keys to git
- Use environment variables for all secrets
- Validate all user inputs before processing
- Use API key middleware for protected endpoints
- Keep wallet private keys in secure environment variables
