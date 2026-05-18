---
name: websocket-devtools-extension
description: Chrome/Edge extension for debugging WebSocket connections with message simulation, traffic blocking, and real-time monitoring
triggers:
  - debug websocket connections in chrome
  - monitor websocket traffic in browser
  - simulate websocket messages
  - block websocket messages for testing
  - inspect websocket frames in devtools
  - capture websocket data in background
  - test websocket error handling
  - replay websocket messages
---

# WebSocket DevTools Extension

> Skill by [ara.so](https://ara.so) — Devtools Skills collection

WebSocket DevTools is a Chrome/Edge browser extension that provides comprehensive WebSocket debugging capabilities including real-time message monitoring, bidirectional message simulation, traffic blocking, and favorites management - all within the browser's native DevTools interface.

## Installation

### Chrome Web Store
```bash
# Visit and install from:
https://chromewebstore.google.com/detail/websocket-devtools/fmnaobbfmjaaaebelkacpmmmpaaefbod

# After installation:
# 1. Open any webpage with WebSocket connections
# 2. Press F12 to open DevTools
# 3. Navigate to "WebSocket DevTools" tab
```

### Microsoft Edge Add-ons
```bash
# Visit and install from:
https://microsoftedge.microsoft.com/addons/detail/websocket-devtools/idkoddoekbiekjkpfjeadehmknaoppol

# After installation, same steps as Chrome
```

### Developer Mode (Local Installation)
```bash
# Clone the repository
git clone https://github.com/law-chain-hot/websocket-devtools.git
cd websocket-devtools

# In Chrome/Edge:
# 1. Navigate to chrome://extensions/ (or edge://extensions/)
# 2. Enable "Developer mode"
# 3. Click "Load unpacked"
# 4. Select the extension directory
```

## Core Concepts

### Background Monitoring
The extension automatically captures all WebSocket connections and messages in the background, even when DevTools is closed. This means:
- No missed connections if you open DevTools after WebSocket establishment
- Persistent message history during page lifetime
- Zero configuration required

### Traffic Control
- **Message Blocking**: Intercept and block messages in either direction (client→server or server→client)
- **Simulation**: Send custom messages as if they came from client or server
- **Pattern Matching**: Block messages based on content, type, or URL patterns

### Favorites System
Save frequently used messages for quick replay and testing scenarios.

## Key Features & Usage

### 1. Monitoring WebSocket Connections

Once installed, the extension automatically captures all WebSocket activity:

```javascript
// Example WebSocket connection that will be automatically monitored
const ws = new WebSocket('wss://example.com/socket');

ws.onopen = () => {
  console.log('Connected');
  ws.send(JSON.stringify({ type: 'auth', token: 'user-token' }));
};

ws.onmessage = (event) => {
  console.log('Received:', event.data);
  // All messages appear in WebSocket DevTools panel
};

ws.onerror = (error) => {
  console.error('WebSocket error:', error);
};

// The extension captures:
// - Connection URL and protocol
// - All sent and received messages
// - Timestamps and message sizes
// - Connection state changes
```

### 2. Message Simulation

Send custom messages to test client/server behavior:

**Simulate Server → Client Message**
```javascript
// In DevTools "Simulate" tab, send this JSON:
{
  "type": "notification",
  "title": "Test Alert",
  "message": "This is a simulated server message",
  "timestamp": 1704067200000
}

// Your client-side handler receives this as if from the server:
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  if (data.type === 'notification') {
    showNotification(data.title, data.message);
  }
};
```

**Simulate Client → Server Message**
```javascript
// In DevTools "Simulate" tab, choose "Client→Server" direction
{
  "action": "subscribe",
  "channel": "trades",
  "symbol": "BTC/USD"
}

// Your server receives this as if the client sent it
// Useful for testing server-side handlers without modifying client code
```

### 3. Message Blocking

Block specific messages to test error handling and edge cases:

**Block Pattern Example**
```javascript
// Your application code
const ws = new WebSocket('wss://api.example.com/feed');

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  
  // Test what happens when 'price_update' messages are blocked
  if (data.type === 'price_update') {
    updatePrice(data.symbol, data.price);
  }
};

// In DevTools:
// 1. Go to "Block" tab
// 2. Add block rule: { "type": "price_update" }
// 3. Test your app's behavior when price updates stop arriving
// 4. Verify your timeout/fallback logic works correctly
```

**Bidirectional Blocking**
```javascript
// Block outgoing messages to test retry logic
const sendWithRetry = async (data, maxRetries = 3) => {
  for (let i = 0; i < maxRetries; i++) {
    try {
      ws.send(JSON.stringify(data));
      
      // If blocked in DevTools, this tests your retry mechanism
      await waitForAck(data.id);
      return;
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await sleep(1000 * Math.pow(2, i));
    }
  }
};
```

### 4. JSON Message Inspector

Automatically parses and formats JSON messages:

```javascript
// When your WebSocket sends/receives JSON:
const message = {
  event: 'order_placed',
  data: {
    orderId: '12345',
    symbol: 'BTC/USD',
    side: 'buy',
    quantity: 0.5,
    price: 45000,
    timestamp: new Date().toISOString()
  }
};

ws.send(JSON.stringify(message));

// In DevTools:
// - Messages automatically parsed and pretty-printed
// - Tree view for nested objects
// - Copy formatted JSON
// - Search within message content
```

### 5. Favorites Management

Save and organize frequently used messages:

```javascript
// Common test messages to save as favorites:

// 1. Authentication test
{
  "type": "auth",
  "username": "testuser",
  "token": "${TEST_AUTH_TOKEN}"
}

// 2. Market data subscription
{
  "action": "subscribe",
  "channels": ["ticker", "trades", "orderbook"],
  "symbols": ["BTC/USD", "ETH/USD"]
}

// 3. Error scenario test
{
  "type": "error",
  "code": 401,
  "message": "Unauthorized"
}

// Use favorites to:
// - Quickly replay common scenarios
// - Build test suites
// - Document API message formats
// - Share testing patterns with team
```

## Common Patterns

### Testing WebSocket Reconnection Logic

```javascript
// Your reconnection logic
class WebSocketClient {
  constructor(url) {
    this.url = url;
    this.reconnectAttempts = 0;
    this.maxReconnectAttempts = 5;
    this.connect();
  }

  connect() {
    this.ws = new WebSocket(this.url);
    
    this.ws.onopen = () => {
      console.log('Connected');
      this.reconnectAttempts = 0;
    };
    
    this.ws.onclose = () => {
      console.log('Disconnected');
      this.reconnect();
    };
  }

  reconnect() {
    if (this.reconnectAttempts >= this.maxReconnectAttempts) {
      console.error('Max reconnection attempts reached');
      return;
    }
    
    this.reconnectAttempts++;
    const delay = Math.min(1000 * Math.pow(2, this.reconnectAttempts), 30000);
    
    setTimeout(() => this.connect(), delay);
  }
}

// Testing with WebSocket DevTools:
// 1. Monitor the initial connection
// 2. Use DevTools to block all messages (simulates network issue)
// 3. Verify reconnection attempts in the connection list
// 4. Check exponential backoff timing
// 5. Remove block and verify successful reconnection
```

### Testing Message Order and Race Conditions

```javascript
// Your application code
class MessageQueue {
  constructor(ws) {
    this.ws = ws;
    this.queue = [];
    this.processing = false;
  }

  async send(message) {
    this.queue.push(message);
    if (!this.processing) {
      await this.processQueue();
    }
  }

  async processQueue() {
    this.processing = true;
    
    while (this.queue.length > 0) {
      const message = this.queue.shift();
      this.ws.send(JSON.stringify(message));
      await this.waitForAck(message.id);
    }
    
    this.processing = false;
  }
}

// Testing with WebSocket DevTools:
// 1. Send multiple messages rapidly using Simulate tab
// 2. Observe order in message list
// 3. Block acknowledgment messages to test queue behavior
// 4. Verify no race conditions or message loss
```

### Testing Binary Message Handling

```javascript
// Your binary WebSocket handler
const ws = new WebSocket('wss://example.com/binary');
ws.binaryType = 'arraybuffer';

ws.onmessage = (event) => {
  if (event.data instanceof ArrayBuffer) {
    const view = new DataView(event.data);
    const messageType = view.getUint8(0);
    const messageId = view.getUint32(1, true);
    
    console.log('Binary message:', messageType, messageId);
    
    // Process binary data
    processBinaryMessage(event.data);
  }
};

// In WebSocket DevTools:
// - Binary messages shown with size and hex preview
// - Can inspect binary content
// - Monitor binary message frequency
// - Block binary messages to test fallback logic
```

### Socket.IO Integration

```javascript
// Socket.IO uses WebSocket under the hood
const socket = io('https://example.com', {
  transports: ['websocket'],
  auth: {
    token: process.env.SOCKET_IO_TOKEN
  }
});

socket.on('connect', () => {
  console.log('Socket.IO connected');
  
  // All these events are visible in WebSocket DevTools
  socket.emit('join_room', { room: 'trading' });
});

socket.on('price_update', (data) => {
  console.log('Price:', data);
});

// In WebSocket DevTools:
// - See Socket.IO protocol messages (42["event",data])
// - Monitor heartbeat (ping/pong) messages
// - Simulate Socket.IO events by sending proper format
// - Block specific event types for testing
```

## Configuration

The extension works with zero configuration, but you can customize behavior:

### Extension Settings (in DevTools panel)

```javascript
// Settings accessible in WebSocket DevTools panel:

// 1. Message Filter
// - Filter by direction (sent/received)
// - Filter by content (text search)
// - Filter by connection URL

// 2. Display Options
// - Auto-scroll to latest message
// - Show/hide timestamps
// - Message format (raw/formatted JSON)
// - Max messages to display (performance)

// 3. Block Rules
// - Enable/disable all blocking
// - Export/import block rules
// - Match type: exact, contains, regex

// 4. Favorites
// - Organize by tags
// - Export/import favorites
// - Quick access shortcuts
```

## Troubleshooting

### DevTools Panel Not Showing

```javascript
// Issue: "WebSocket DevTools" tab not visible in DevTools

// Solutions:
// 1. Verify extension is enabled in chrome://extensions/
// 2. Refresh the page after enabling extension
// 3. Check if extension has permissions for the current site
// 4. Try opening DevTools before navigating to the page
// 5. Check browser console for extension errors
```

### Messages Not Being Captured

```javascript
// Issue: WebSocket connections not appearing

// Check your WebSocket creation:
const ws = new WebSocket('wss://example.com/socket');

// Ensure:
// 1. Using standard WebSocket API (not custom implementations)
// 2. Connection established after extension loaded
// 3. No Content Security Policy blocking
// 4. Not using WebWorkers (limited support)

// For iframes:
// Extension supports iframe WebSockets
// Ensure iframe has same origin or CORS configured
```

### Simulation Not Working

```javascript
// Issue: Simulated messages not received by application

// Verify message format matches server expectations:
// Wrong:
{
  type: "message",
  data: "test"  // Missing fields
}

// Correct:
{
  type: "message",
  id: "msg-123",
  timestamp: 1704067200000,
  data: "test",
  checksum: "abc123"  // Include all required fields
}

// Check:
// 1. JSON is valid (use built-in validator)
// 2. Message structure matches protocol
// 3. Direction is correct (Client→Server vs Server→Client)
// 4. WebSocket is connected (not closed)
```

### Block Rules Not Applied

```javascript
// Issue: Messages not being blocked despite rules

// Check block rule format:
// Exact match (case-sensitive):
{
  "type": "heartbeat"
}

// Contains match:
{
  "*message*": "*error*"
}

// Regex match:
{
  "type": "/^(ping|pong)$/"
}

// Common issues:
// 1. Rule disabled in Block tab
// 2. Wrong match type selected
// 3. Typo in field name
// 4. Message format different than expected (binary vs text)
```

### Performance Issues with High Message Volume

```javascript
// Issue: DevTools slow with many messages

// Optimization:
// 1. Reduce max messages displayed (Settings → Max Messages: 1000)
// 2. Use filters to show only relevant messages
// 3. Clear old connections periodically
// 4. Disable auto-scroll for high-frequency messages

// For high-frequency trading apps:
const ws = new WebSocket('wss://hft.example.com/feed');

// Consider:
// - Using message sampling in DevTools filters
// - Only monitoring during specific test scenarios
// - Clearing message history when not actively debugging
```

## Advanced Usage

### Testing WebSocket Security

```javascript
// Test authentication flows
const ws = new WebSocket('wss://secure.example.com/socket');

ws.onopen = () => {
  // Test invalid token
  ws.send(JSON.stringify({
    type: 'auth',
    token: 'invalid-token'
  }));
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  
  if (data.type === 'auth_failed') {
    // Verify proper error handling
    console.error('Authentication failed:', data.reason);
  }
};

// In WebSocket DevTools:
// 1. Save valid auth message as favorite
// 2. Create variations with invalid tokens
// 3. Test different error scenarios
// 4. Verify server responses and client handling
```

### Multi-Connection Testing

```javascript
// Test multiple simultaneous connections
const connections = [];

for (let i = 0; i < 5; i++) {
  const ws = new WebSocket(`wss://example.com/socket?client=${i}`);
  
  ws.onopen = () => {
    console.log(`Client ${i} connected`);
  };
  
  connections.push(ws);
}

// In WebSocket DevTools:
// - See all 5 connections listed separately
// - Monitor messages for each connection
// - Simulate messages to specific connections
// - Test connection-specific logic
// - Verify server handles multiple clients correctly
```

### Integration with Automated Testing

```javascript
// While extension is manual, you can prepare test scenarios

// 1. Document test cases as favorites
const testCases = [
  { name: 'Valid Login', message: { type: 'auth', token: 'valid' } },
  { name: 'Invalid Login', message: { type: 'auth', token: 'invalid' } },
  { name: 'Subscribe', message: { type: 'subscribe', channel: 'prices' } }
];

// 2. Export favorites for team sharing
// 3. Create block rule sets for different test scenarios
// 4. Document expected behaviors in favorite descriptions

// This helps manual QA and developers test consistently
```

## Best Practices

1. **Always enable background monitoring** before loading pages with WebSockets
2. **Save complex test messages as favorites** for repeatability
3. **Use message filtering** to focus on specific message types during debugging
4. **Export favorites and block rules** to share testing scenarios with team
5. **Clear old connections** periodically to maintain performance
6. **Test both directions** - client→server and server→client simulation
7. **Document your WebSocket protocol** using favorites as examples
8. **Use blocking to test error handling** and recovery logic
9. **Monitor message frequency** to identify performance issues
10. **Check iframe WebSocket support** if working with embedded content

## Resources

- Official Website: https://websocket-devtools.com
- GitHub Repository: https://github.com/law-chain-hot/websocket-devtools
- Chrome Web Store: https://chromewebstore.google.com/detail/websocket-devtools/fmnaobbfmjaaaebelkacpmmmpaaefbod
- Video Demo: https://www.youtube.com/watch?v=L64x__1xORQ
- Documentation Wiki: https://github.com/law-chain-hot/websocket-devtools/wiki
