# D.I.O.T Client Chat Interface - Design Document

## Overview

The D.I.O.T Client Chat Interface is a cross-platform mobile application built with React Native and Expo that provides a conversational interface for users to interact with an AI agent. The application follows a modern mobile chat UI pattern with real-time message exchange, persistent storage, and responsive design across iOS, Android, and web platforms.

The design leverages React Native's component architecture with centralized state management through React Context, ensuring consistent data flow and UI updates. The application communicates with a backend AI agent service through RESTful API endpoints, handling authentication and error scenarios gracefully.

**Key Design Principles:**
- **Simplicity**: Minimal UI focused on conversation flow
- **Responsiveness**: Adaptive layouts for different screen sizes and orientations
- **Reliability**: Graceful error handling and fallback mechanisms
- **Security**: Encrypted storage for sensitive data
- **Performance**: Optimized rendering and state updates

## Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   D.I.O.T Client App                    │
│                                                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │           Smart Provider (Layout)                │  │
│  │  ┌────────────────────────────────────────────┐  │  │
│  │  │      Context Provider (State)              │  │  │
│  │  │  ┌──────────────────────────────────────┐  │  │  │
│  │  │  │     Chat Interface Component         │  │  │  │
│  │  │  │  - Message List                      │  │  │  │
│  │  │  │  - Input Field                       │  │  │  │
│  │  │  │  - Send Button                       │  │  │  │
│  │  │  └──────────────────────────────────────┘  │  │  │
│  │  └────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────┘  │
│                                                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │              Utility Layer                       │  │
│  │  - Storage (Encrypted/Async)                     │  │
│  │  - Time Formatting                               │  │
│  │  - API Communication                             │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                          │
                          │ HTTP/REST
                          ▼
┌─────────────────────────────────────────────────────────┐
│                  Backend API Service                    │
│  ┌──────────────────────────────────────────────────┐  │
│  │         /api/chatWithAgent Endpoint              │  │
│  └──────────────────────────────────────────────────┘  │
│                          │                              │
│                          ▼                              │
│  ┌──────────────────────────────────────────────────┐  │
│  │              AI Agent Service                    │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### Component Hierarchy

```
App Root (_layout.js)
└── Smart Provider (responsive layout wrapper)
    └── Context Provider (state management)
        └── Chat Interface (index.js)
            ├── Message List (ScrollView)
            │   └── Message Items (user/system)
            └── Input Container
                ├── Text Input (multiline)
                └── Send Button
```

### Data Flow

1. **User Input Flow:**
   - User types message → Input field updates local state
   - User presses send → Message added to context state
   - API call initiated → Loading state activated
   - Response received → System message added to context state
   - UI updates automatically via context subscription

2. **State Management Flow:**
   - Context Provider holds global state (chat history)
   - Components access state via useContext hook
   - State updates trigger re-renders in subscribed components
   - Async operations use setValueAsync for promise-based updates

3. **Storage Flow:**
   - Sensitive data → Encrypted Storage (expo-secure-store)
   - General data → Async Storage (AsyncStorage)
   - Fallback mechanism if encrypted storage unavailable
   - All storage operations return null on failure (no exceptions)

## Components and Interfaces

### 1. Context Provider (`contextModule.js`)

**Purpose:** Centralized state management for the entire application.

**Interface:**
```javascript
interface ContextState {
  chatHistory: Message[];
  // Additional state fields as needed
}

interface ContextValue {
  value: ContextState;
  setValue: (updates: Partial<ContextState>) => void;
  setValueAsync: (updates: Partial<ContextState>) => Promise<void>;
}

const AppContext = createContext<ContextValue>();
```

**Responsibilities:**
- Initialize state with welcome message
- Provide synchronous state updates (setValue)
- Provide asynchronous state updates (setValueAsync)
- Merge updates with existing state
- Expose state and update methods to child components

**Design Rationale:** React Context provides a clean way to share state across components without prop drilling. The dual update methods (sync/async) allow flexibility for different use cases - immediate UI updates vs. operations requiring promises.

### 2. Smart Provider (`smartProvider.js`)

**Purpose:** Responsive layout wrapper that adapts to screen size and orientation.

**Interface:**
```javascript
interface SmartProviderProps {
  children: React.ReactNode;
}

function SmartProvider({ children }: SmartProviderProps): JSX.Element
```

**Responsibilities:**
- Detect platform (iOS, Android, web)
- Detect screen orientation (landscape/portrait)
- Apply mobile frame wrapper for web landscape mode
- Render full-screen for mobile platforms
- Constrain dimensions for web landscape (mobile-like experience)

**Design Rationale:** Separating layout logic from business logic keeps components focused. The frame wrapper on web provides a consistent mobile-like experience during development and testing.

### 3. Chat Interface Component (`index.js`)

**Purpose:** Main UI component displaying conversation and handling user input.

**Interface:**
```javascript
interface Message {
  type: 'user' | 'system';
  content: string;
  timestamp: number;
  tool?: string; // Optional tool information
}

function ChatInterface(): JSX.Element
```

**State:**
- `inputText`: Current text in input field (local state)
- `isLoading`: Whether API request is in progress (local state)
- `chatHistory`: Array of messages (from context)

**Responsibilities:**
- Render message list with proper styling
- Handle user input and validation
- Send messages to API endpoint
- Display loading indicators
- Auto-scroll to latest message
- Format timestamps for display

**Design Rationale:** Keeping input state local (not in context) prevents unnecessary re-renders of the entire message list when user types. Only committed messages go into global state.

### 4. API Communication (`chatWithAgent+api.js`)

**Purpose:** Handle communication with backend AI agent service.

**Interface:**
```javascript
async function POST(request: Request): Promise<Response>

interface RequestBody {
  message: string;
  context?: any;
}

interface ResponseBody {
  response: string;
  tool?: string;
}
```

**Responsibilities:**
- Receive message from client
- Forward to AI agent service with authentication
- Handle timeout and network errors
- Return agent response or null on failure
- Apply response modifiers if needed

**Design Rationale:** Using Expo Router's API routes keeps the API layer within the same codebase, simplifying deployment. The timeout mechanism (30 seconds) prevents hanging requests while giving the AI agent sufficient time to respond.

### 5. Storage Utilities (`utils.js`)

**Purpose:** Provide secure and reliable data persistence.

**Interface:**
```javascript
async function setEncryptedItem(key: string, value: string): Promise<void>
async function getEncryptedItem(key: string): Promise<string | null>
async function setItem(key: string, value: string): Promise<void>
async function getItem(key: string): Promise<string | null>
```

**Responsibilities:**
- Store sensitive data with encryption (expo-secure-store)
- Store general data without encryption (AsyncStorage)
- Provide fallback mechanism for encrypted storage
- Handle storage failures gracefully (return null)
- Never throw exceptions to calling code

**Design Rationale:** The fallback mechanism ensures the app works even on platforms where secure storage is unavailable. Returning null instead of throwing exceptions simplifies error handling in components.

### 6. Time Formatting Utilities (`utils.js`)

**Purpose:** Convert timestamps to human-readable relative time.

**Interface:**
```javascript
function formatTimestamp(timestamp: number): string
```

**Responsibilities:**
- Calculate time difference from current time
- Return "just now" for < 1 minute
- Return "X minutes ago" for < 60 minutes
- Return "X hours ago" for < 24 hours
- Return "Yesterday" for exactly 1 day
- Return formatted date for > 1 day

**Design Rationale:** Relative timestamps provide better context for recent messages while absolute dates work better for older messages. This hybrid approach balances readability with precision.

## Data Models

### Message Object

```javascript
interface Message {
  type: 'user' | 'system';
  content: string;
  timestamp: number;
  tool?: string;
}
```

**Fields:**
- `type`: Identifies sender (user or AI agent)
- `content`: The actual message text
- `timestamp`: Unix timestamp in milliseconds
- `tool`: Optional field indicating which AI tool was used

**Validation Rules:**
- `type` must be either 'user' or 'system'
- `content` must be non-empty string for user messages
- `timestamp` must be valid Unix timestamp
- `tool` is optional and only relevant for system messages

### Context State

```javascript
interface ContextState {
  chatHistory: Message[];
}
```

**Fields:**
- `chatHistory`: Ordered array of all messages in conversation

**Initialization:**
```javascript
const initialState = {
  chatHistory: [
    {
      type: 'system',
      content: 'Hello! I\'m your D.I.O.T assistant. How can I help you today?',
      timestamp: Date.now()
    }
  ]
};
```

### API Request/Response Models

**Request to Backend:**
```javascript
interface ChatRequest {
  message: string;
  context?: {
    userId?: string;
    sessionId?: string;
    // Additional context fields
  };
}
```

**Response from Backend:**
```javascript
interface ChatResponse {
  response: string;
  tool?: string;
  error?: string;
}
```

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property 1: Message creation preserves user input
*For any* non-empty string entered by the user, when the send button is pressed, the created message object should contain that exact string as its content field.
**Validates: Requirements 1.1**

### Property 2: Message addition grows chat history
*For any* chat history state and any valid message object, adding the message should result in the chat history length increasing by exactly one.
**Validates: Requirements 1.2, 2.4**

### Property 3: Input field clearing after send
*For any* non-empty input field state, when a message is successfully sent, the input field should be empty.
**Validates: Requirements 1.3**

### Property 4: Send button disabled for empty input
*For any* input field state, the send button should be disabled if and only if the input field contains only whitespace or is empty.
**Validates: Requirements 1.5**

### Property 5: Message ordering preservation
*For any* sequence of messages added to chat history, the messages should appear in chronological order based on their timestamps.
**Validates: Requirements 3.1**

### Property 6: User message alignment
*For any* message with type 'user', the rendered message should be aligned to the right side of the screen.
**Validates: Requirements 3.3**

### Property 7: System message alignment
*For any* message with type 'system', the rendered message should be aligned to the left side of the screen.
**Validates: Requirements 3.4**

### Property 8: Timestamp formatting for recent messages
*For any* message with timestamp less than 60 seconds old, the formatted timestamp should be "just now".
**Validates: Requirements 4.1**

### Property 9: Timestamp formatting for minutes
*For any* message with timestamp between 60 seconds and 60 minutes old, the formatted timestamp should be "X minutes ago" where X is the number of minutes.
**Validates: Requirements 4.2**

### Property 10: Timestamp formatting for hours
*For any* message with timestamp between 60 minutes and 24 hours old, the formatted timestamp should be "X hours ago" where X is the number of hours.
**Validates: Requirements 4.3**

### Property 11: Storage fallback mechanism
*For any* storage operation where encrypted storage is unavailable, the system should successfully complete the operation using async storage as fallback.
**Validates: Requirements 5.2, 5.3**

### Property 12: Storage error handling
*For any* storage operation that fails, the system should return null without throwing an exception.
**Validates: Requirements 5.5**

### Property 13: Platform-specific rendering
*For any* platform (iOS, Android, web), the application should render with platform-appropriate styling without errors.
**Validates: Requirements 6.1, 6.2, 6.3, 6.4, 6.5**

### Property 14: Message styling consistency
*For any* message of a given type (user or system), all messages of that type should have consistent gradient background and rounded corners.
**Validates: Requirements 7.1, 7.2**

### Property 15: State update merging
*For any* context state and any partial state update, the resulting state should contain all fields from the original state plus the updated fields.
**Validates: Requirements 8.4**

### Property 16: API error resilience
*For any* API request that fails (timeout, network error, or null response), the application should continue functioning without crashing.
**Validates: Requirements 9.1, 9.2, 9.3**

### Property 17: Loading state management
*For any* message send operation, the send button should be disabled and loading indicator visible during the API request, and re-enabled after completion.
**Validates: Requirements 1.4, 9.5**

### Property 18: Multiline input support
*For any* text input containing line breaks, the input field should preserve and display the line breaks correctly.
**Validates: Requirements 10.1, 10.2**

## Error Handling

### Input Validation Errors

**Scenario:** User attempts to send empty or whitespace-only message
- **Prevention:** Send button disabled when input is empty/whitespace
- **UI Feedback:** Button appears grayed out and non-interactive
- **Recovery:** User must enter valid text to enable button

### Network Errors

**Scenario:** API request fails due to network connectivity
- **Detection:** Fetch timeout (30 seconds) or network exception
- **Handling:** Promise resolves with null value
- **UI Feedback:** Loading indicator disappears, send button re-enabled
- **Recovery:** User can retry sending message
- **Logging:** Error logged to console for debugging

**Scenario:** API request times out
- **Detection:** 30-second timeout in fetch request
- **Handling:** Promise resolves with null value
- **UI Feedback:** Same as network error
- **Recovery:** Same as network error

### Storage Errors

**Scenario:** Encrypted storage unavailable
- **Detection:** expo-secure-store throws exception
- **Handling:** Automatic fallback to AsyncStorage with backup key
- **UI Feedback:** None (transparent to user)
- **Recovery:** Automatic

**Scenario:** All storage mechanisms fail
- **Detection:** Both encrypted and async storage fail
- **Handling:** Return null from storage functions
- **UI Feedback:** None (app continues with in-memory state)
- **Recovery:** Data persists only for current session

### API Response Errors

**Scenario:** Backend returns error response
- **Detection:** Response contains error field
- **Handling:** Process through response modifier
- **UI Feedback:** Error message may be displayed (implementation-dependent)
- **Recovery:** User can retry or modify request

**Scenario:** Backend returns null response
- **Detection:** Response is null or undefined
- **Handling:** Skip adding system message to chat history
- **UI Feedback:** No new message appears
- **Recovery:** User can retry sending message

### Platform-Specific Errors

**Scenario:** Platform detection fails
- **Detection:** Platform.OS returns unexpected value
- **Handling:** Default to mobile rendering (no frame wrapper)
- **UI Feedback:** May appear slightly different than expected
- **Recovery:** Automatic (app still functional)

### State Management Errors

**Scenario:** Context provider not available
- **Detection:** useContext returns undefined
- **Handling:** Component-level error boundary (if implemented)
- **UI Feedback:** Error message or fallback UI
- **Recovery:** App restart required

**Scenario:** State update fails
- **Detection:** setState throws exception (rare)
- **Handling:** Try-catch in update functions
- **UI Feedback:** Previous state maintained
- **Recovery:** User can retry action

## Testing Strategy

### Unit Testing

The application will use **Jest** as the testing framework with **React Native Testing Library** for component testing. Unit tests will focus on:

**Component Tests:**
- Message rendering with correct styling based on type
- Input field validation and state management
- Send button enable/disable logic
- Timestamp formatting edge cases
- Platform-specific rendering logic

**Utility Function Tests:**
- Storage functions (encrypted and async)
- Time formatting with various timestamp values
- API request/response handling
- Error handling in storage operations

**Context Tests:**
- State initialization with welcome message
- setValue and setValueAsync functionality
- State merging behavior
- Context provider rendering

**Example Unit Tests:**
- Empty input disables send button
- "just now" appears for timestamps < 60 seconds
- User messages align right, system messages align left
- Storage fallback when encrypted storage unavailable
- API timeout returns null after 30 seconds

### Property-Based Testing

The application will use **fast-check** for property-based testing. Property tests will verify universal behaviors across many randomly generated inputs:

**Configuration:**
- Minimum 100 iterations per property test
- Each test tagged with format: `**Feature: diot-client-chat-interface, Property {number}: {property_text}**`
- Tests run as part of standard test suite

**Property Test Coverage:**

1. **Message Creation Properties:**
   - Property 1: Message creation preserves user input
   - Property 2: Message addition grows chat history
   - Property 3: Input field clearing after send

2. **UI Rendering Properties:**
   - Property 6: User message alignment
   - Property 7: System message alignment
   - Property 14: Message styling consistency

3. **Timestamp Formatting Properties:**
   - Property 8: Timestamp formatting for recent messages
   - Property 9: Timestamp formatting for minutes
   - Property 10: Timestamp formatting for hours

4. **Storage Properties:**
   - Property 11: Storage fallback mechanism
   - Property 12: Storage error handling

5. **State Management Properties:**
   - Property 15: State update merging
   - Property 5: Message ordering preservation

6. **Error Handling Properties:**
   - Property 16: API error resilience
   - Property 17: Loading state management

**Generator Strategies:**
- Random strings of varying lengths (including empty, whitespace, special characters)
- Random timestamps (past, present, edge cases)
- Random message arrays of varying sizes
- Random platform values (iOS, Android, web)
- Simulated storage failures and network errors

### Integration Testing

Integration tests will verify end-to-end flows:

**Chat Flow Integration:**
- User sends message → API call → Response received → UI updates
- Multiple messages in sequence maintain correct order
- Loading states transition correctly during API calls

**Storage Integration:**
- Data persists across app restarts
- Encrypted storage fallback works correctly
- Storage errors don't crash the app

**Platform Integration:**
- App renders correctly on iOS simulator
- App renders correctly on Android emulator
- Web version displays mobile frame in landscape

### Manual Testing Checklist

- [ ] Send various message types (short, long, multiline, special characters)
- [ ] Test on iOS device/simulator
- [ ] Test on Android device/emulator
- [ ] Test web version in different browsers
- [ ] Test landscape and portrait orientations
- [ ] Verify encrypted storage on physical devices
- [ ] Test with poor network connectivity
- [ ] Test with backend service offline
- [ ] Verify timestamps update correctly over time
- [ ] Test rapid message sending (stress test)

### Test Organization

```
diot-client/
├── src/
│   ├── app/
│   │   ├── index.test.js          # Chat interface component tests
│   │   └── api/
│   │       └── chatWithAgent.test.js  # API endpoint tests
│   ├── providers/
│   │   ├── contextModule.test.js   # Context provider tests
│   │   └── smartProvider.test.js   # Smart provider tests
│   ├── core/
│   │   └── utils.test.js           # Utility function tests
│   └── __tests__/
│       ├── properties/             # Property-based tests
│       │   ├── message.properties.test.js
│       │   ├── timestamp.properties.test.js
│       │   ├── storage.properties.test.js
│       │   └── state.properties.test.js
│       └── integration/            # Integration tests
│           └── chat-flow.integration.test.js
└── jest.config.js
```

### Testing Dependencies

```json
{
  "devDependencies": {
    "jest": "^29.7.0",
    "@testing-library/react-native": "^12.4.0",
    "@testing-library/jest-native": "^5.4.3",
    "fast-check": "^3.15.0",
    "jest-expo": "~51.0.0"
  }
}
```

## Security Considerations

### Data Storage Security

- **Sensitive Data:** User credentials, API keys, and personal information stored using expo-secure-store with hardware-backed encryption on supported devices
- **General Data:** Chat history and preferences stored in AsyncStorage (unencrypted but sandboxed per app)
- **Fallback Security:** When secure storage unavailable, backup key used with AsyncStorage (less secure but functional)

### API Communication Security

- **Authentication:** API requests include authentication headers (AI_URL_API_KEY)
- **HTTPS:** All production API calls should use HTTPS to prevent man-in-the-middle attacks
- **Timeout Protection:** 30-second timeout prevents hanging connections
- **Error Masking:** Detailed error messages not exposed to UI (logged for debugging only)

### Input Validation

- **Client-Side:** Input validated before sending (non-empty, reasonable length)
- **Server-Side:** Backend should validate all inputs (defense in depth)
- **Injection Prevention:** No direct HTML rendering of user input (React handles escaping)

### State Management Security

- **Immutability:** State updates create new objects (prevents accidental mutations)
- **Isolation:** Context state not directly accessible outside provider
- **No Sensitive Data in State:** Passwords and keys never stored in React state

## Performance Considerations

### Rendering Optimization

- **Message List:** Use FlatList instead of ScrollView for large chat histories (virtualization)
- **Memo Components:** Memoize message components to prevent unnecessary re-renders
- **Local State:** Keep input text in local state to avoid re-rendering entire message list on each keystroke

### State Management Optimization

- **Selective Updates:** Only update changed portions of state (merge pattern)
- **Async Operations:** Use setValueAsync for operations that don't need immediate UI updates
- **Context Splitting:** Consider splitting context if state grows large (separate chat history from settings)

### Network Optimization

- **Request Debouncing:** Prevent rapid-fire API calls (send button disabled during requests)
- **Timeout Management:** 30-second timeout balances responsiveness with AI processing time
- **Error Recovery:** Failed requests don't block subsequent attempts

### Storage Optimization

- **Lazy Loading:** Load chat history on demand rather than at app start
- **Pagination:** Consider paginating old messages if history grows very large
- **Cleanup:** Implement message retention policy (e.g., keep last 1000 messages)

## Deployment Considerations

### Platform-Specific Builds

- **iOS:** Build with EAS, configure app.json for iOS-specific settings
- **Android:** Build with EAS, configure app.json for Android-specific settings
- **Web:** Use `npm run webbuild` and deploy to EAS or static hosting

### Environment Configuration

- **Development:** Local API endpoint (http://localhost:8000)
- **Production:** Production API endpoint (HTTPS required)
- **API Keys:** Stored in environment variables, not committed to git

### Version Management

- **App Version:** Increment version in app.json for each release
- **API Versioning:** Consider API version headers for backward compatibility
- **Feature Flags:** Use environment variables to enable/disable features

### Monitoring and Analytics

- **Error Tracking:** Consider integrating Sentry or similar for crash reporting
- **Usage Analytics:** Track message counts, response times, error rates
- **Performance Monitoring:** Monitor app startup time, API latency

## Future Enhancements

### Potential Features

1. **Message Persistence:** Save chat history to backend for cross-device sync
2. **Rich Media:** Support images, files, and formatted text in messages
3. **Voice Input:** Add speech-to-text for message input
4. **Push Notifications:** Notify users of agent responses when app in background
5. **Message Search:** Search through chat history
6. **Message Editing:** Allow users to edit sent messages
7. **Conversation Management:** Multiple conversation threads
8. **Offline Mode:** Queue messages when offline, send when reconnected
9. **Typing Indicators:** Show when agent is processing
10. **Message Reactions:** Allow users to react to messages with emoji

### Technical Improvements

1. **Performance:** Implement FlatList virtualization for message list
2. **Testing:** Increase test coverage to 80%+
3. **Accessibility:** Add screen reader support and keyboard navigation
4. **Internationalization:** Support multiple languages
5. **Theming:** Dark mode and customizable color schemes
6. **Animation:** Smooth transitions for message appearance
7. **Caching:** Cache API responses for faster repeated queries
8. **Compression:** Compress large message payloads
9. **Rate Limiting:** Client-side rate limiting to prevent abuse
10. **Telemetry:** Detailed performance and usage metrics

