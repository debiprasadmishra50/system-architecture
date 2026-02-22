# WebSockets

## Table of Contents
1. [What are WebSockets](#what-are-websockets)
2. [How WebSockets Work](#how-websockets-work)
3. [WebSocket vs WebRTC](#websocket-vs-webrtc)
4. [WebSocket vs TURN](#websocket-vs-turn)
5. [WebSocket vs QUIC](#websocket-vs-quic)
6. [Protocol Comparison Table](#protocol-comparison-table)
7. [Applications of WebSockets](#applications-of-websockets)
8. [NodeJS Example with ws](#nodejs-example-with-ws)
9. [Architect's Perspective](#architects-perspective)

---

## What are WebSockets

- **Definition**: A communication protocol providing full-duplex (bidirectional) communication over a single TCP connection
- **Upgrade Mechanism**: Starts as HTTP request, upgrades to WebSocket protocol via handshake
- **Persistent Connection**: Connection remains open, allowing server and client to send data independently
- **Low Latency**: Minimal overhead compared to HTTP polling
- **Standardized**: RFC 6455 defines the WebSocket protocol
- **Port**: Typically uses port 80 (ws://) or 443 (wss://)

---

## How WebSockets Work

### WebSocket Handshake

```
Client                                 Server
  │                                      │
  ├─── HTTP GET /chat HTTP/1.1 ─────────→│
  │     Upgrade: websocket               │
  │     Connection: Upgrade              │
  │     Sec-WebSocket-Key: ...           │
  │     Sec-WebSocket-Version: 13        │
  │                                      │
  │←─── HTTP/1.1 101 Switching ──────────┤
  │     Protocols                        │
  │     Upgrade: websocket               │
  │     Connection: Upgrade              │
  │     Sec-WebSocket-Accept: ...        │
  │                                      │
  │ (Connection upgraded to WebSocket)   │
  │                                      │
  ├─────── Frame Data ──────────────────→│
  │←─────── Frame Data ──────────────────┤
  │                                      │
  │ (Bidirectional communication)        │
  │                                      │
```

### Key Steps

1. **Client Initiates**: Sends HTTP GET request with `Upgrade: websocket` header
2. **Server Accepts**: Responds with HTTP 101 Switching Protocols
3. **Protocol Upgrade**: TCP connection switches from HTTP to WebSocket
4. **Persistent Connection**: Connection remains open for bidirectional communication
5. **Frame Exchange**: Data sent as frames (text or binary)
6. **Connection Close**: Either party can close with close frame

### Frame Structure

- **Opcode**: Indicates frame type (text, binary, close, ping, pong)
- **Payload Length**: Size of data being sent
- **Masking**: Client-to-server frames are masked for security
- **Payload Data**: Actual message content

---

## WebSocket vs WebRTC

| Aspect | WebSocket | WebRTC |
|--------|-----------|--------|
| **Purpose** | General bidirectional communication | Peer-to-peer media (audio/video) |
| **Connection Type** | Client-Server | Peer-to-Peer |
| **Protocol** | TCP-based | UDP-based |
| **Latency** | Low (10-100ms) | Ultra-low (1-50ms) |
| **Data Type** | Text, binary messages | Audio, video, data streams |
| **Firewall Traversal** | Works through proxies | Requires STUN/TURN servers |
| **Bandwidth** | Moderate | High (media streams) |
| **Use Case** | Chat, notifications, live feeds | Video calls, screen sharing, gaming |
| **Complexity** | Simple | Complex (NAT traversal) |
| **Server Requirement** | Always required | Only for signaling |

### Key Differences

- **WebSocket**: Centralized server-client model, suitable for messaging and real-time data
- **WebRTC**: Decentralized peer-to-peer, optimized for media streaming with minimal latency

---

## WebSocket vs TURN

| Aspect | WebSocket | TURN |
|--------|-----------|------|
| **Purpose** | Bidirectional messaging | NAT/Firewall traversal for P2P |
| **Protocol** | TCP/WebSocket | UDP/TCP relay |
| **Connection Model** | Client-Server | Peer-to-Peer (with relay) |
| **Use Case** | Real-time chat, notifications | Media relay when direct P2P fails |
| **Latency** | Low | Higher (relay overhead) |
| **Server Role** | Primary endpoint | Fallback relay |
| **Bandwidth** | Moderate | High (relays all traffic) |
| **Complexity** | Simple | Complex (NAT traversal) |

### Key Differences

- **WebSocket**: Direct communication channel between client and server
- **TURN**: Relay server that forwards traffic when direct P2P connection impossible (used with WebRTC)

---

## WebSocket vs QUIC

| Aspect | WebSocket | QUIC |
|--------|-----------|------|
| **Protocol Layer** | Application (over TCP) | Transport (replaces TCP) |
| **Connection Type** | Single TCP connection | Multiple UDP streams |
| **Latency** | Low (TCP handshake overhead) | Ultra-low (0-RTT connection) |
| **Multiplexing** | Single stream | Multiple independent streams |
| **Head-of-Line Blocking** | Yes (TCP limitation) | No (independent streams) |
| **Connection Migration** | No | Yes (IP/port changes) |
| **Encryption** | Optional (TLS) | Built-in (mandatory) |
| **Use Case** | Real-time messaging | High-performance, mobile-friendly |
| **Adoption** | Widespread | Growing (HTTP/3) |
| **Complexity** | Simple | Complex |

### Key Differences

- **WebSocket**: Application-level protocol, relies on TCP
- **QUIC**: Transport-level protocol, UDP-based, designed for modern internet with better performance

---

## Protocol Comparison Table

| Feature | WebSocket | WebRTC | TURN | QUIC |
|---------|-----------|--------|------|------|
| **Bidirectional** | ✓ | ✓ | ✗ (relay) | ✓ |
| **Low Latency** | ✓ | ✓✓ | ✗ | ✓✓ |
| **P2P** | ✗ | ✓ | ✓ (with relay) | ✓ |
| **Media Streaming** | ✗ | ✓ | ✓ | ✓ |
| **Firewall Friendly** | ✓ | ✗ (needs TURN) | ✓ | ✓ |
| **Simple to Implement** | ✓ | ✗ | ✗ | ✗ |
| **Server Required** | ✓ | ✗ (signaling only) | ✓ (fallback) | ✗ |
| **Encryption** | Optional | ✓ | ✓ | ✓ |

---

## Applications of WebSockets

### Real-Time Communication
- **Live Chat**: Instant messaging between users
- **Notifications**: Push notifications, alerts, status updates
- **Collaborative Tools**: Real-time document editing (Google Docs), whiteboards, code editors

### Live Data Streaming
- **Stock Market**: Real-time price updates, trading data
- **Sports**: Live scores, game updates, commentary
- **Weather**: Real-time weather data, alerts
- **Cryptocurrency**: Live price feeds, order updates

### Gaming
- **Multiplayer Games**: Real-time player interactions, game state synchronization
- **Turn-Based Games**: Move notifications, game updates
- **Leaderboards**: Live ranking updates

### IoT and Monitoring
- **Device Monitoring**: Real-time sensor data, device status
- **Dashboard Updates**: Live metrics, system monitoring
- **Alerts**: Immediate notifications for critical events

### Media and Entertainment
- **Live Streaming**: Chat during live streams
- **Video Conferencing**: Signaling for WebRTC connections
- **Online Whiteboarding**: Real-time drawing and collaboration

---

## NodeJS Example with ws

### Installation

```bash
npm install ws
```

### Simple Echo Server

```javascript
const WebSocket = require('ws');

// Create WebSocket server on port 8080
const wss = new WebSocket.Server({ port: 8080 });

// Handle new client connections
wss.on('connection', (ws) => {
  console.log('Client connected');

  // Handle incoming messages
  ws.on('message', (message) => {
    console.log(`Received: ${message}`);
    
    // Echo message back to client
    ws.send(`Server received: ${message}`);
    
    // Broadcast to all connected clients
    wss.clients.forEach((client) => {
      if (client.readyState === WebSocket.OPEN) {
        client.send(`Broadcast: ${message}`);
      }
    });
  });

  // Handle client disconnect
  ws.on('close', () => {
    console.log('Client disconnected');
  });

  // Handle errors
  ws.on('error', (error) => {
    console.error(`WebSocket error: ${error}`);
  });

  // Send welcome message
  ws.send('Welcome to WebSocket server!');
});

console.log('WebSocket server running on ws://localhost:8080');
```

### Client-Side HTML

```html
<!DOCTYPE html>
<html>
<head>
  <title>WebSocket Client</title>
</head>
<body>
  <h1>WebSocket Chat</h1>
  <input type="text" id="messageInput" placeholder="Enter message">
  <button onclick="sendMessage()">Send</button>
  <div id="messages"></div>

  <script>
    const ws = new WebSocket('ws://localhost:8080');

    ws.onopen = () => {
      console.log('Connected to server');
    };

    ws.onmessage = (event) => {
      const messagesDiv = document.getElementById('messages');
      messagesDiv.innerHTML += `<p>${event.data}</p>`;
    };

    ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };

    ws.onclose = () => {
      console.log('Disconnected from server');
    };

    function sendMessage() {
      const input = document.getElementById('messageInput');
      ws.send(input.value);
      input.value = '';
    }
  </script>
</body>
</html>
```

### Running the Example

```bash
# Terminal 1: Start server
node server.js

# Terminal 2: Open browser and navigate to client HTML
# Or use wscat for testing:
npm install -g wscat
wscat -c ws://localhost:8080
```

---

## Architect's Perspective

### When to Use WebSockets

- **Real-time Requirements**: Applications needing instant data delivery
- **Bidirectional Communication**: When both client and server need to send data independently
- **Persistent Connections**: Long-lived connections with frequent updates
- **Low Latency Critical**: Chat, notifications, live feeds
- **Scalable Real-Time**: Better than polling for many concurrent users

### When NOT to Use WebSockets

- **Infrequent Updates**: Use polling or HTTP for occasional data
- **Simple Request-Response**: Standard REST API sufficient
- **Media Streaming**: Use WebRTC or QUIC instead
- **Firewall Constraints**: Some corporate firewalls block WebSocket
- **Stateless Requirement**: WebSocket requires server-side state management

### Implementation Considerations

- **Connection Management**: Handle reconnections, timeouts, graceful shutdown
- **Scaling**: Use load balancers with sticky sessions or message brokers (Redis, RabbitMQ)
- **Security**: Use WSS (WebSocket Secure) with TLS, validate origins, implement authentication
- **Monitoring**: Track connection count, message throughput, error rates
- **Fallback**: Implement fallback to polling or SSE for unsupported environments
- **Memory**: Persistent connections consume server memory; implement connection limits

### Technology Stack

- **Node.js**: `ws`, `socket.io`, `uWebSockets`
- **Python**: `websockets`, `Django Channels`
- **Java**: `Spring WebSocket`, `Tyrus`
- **Go**: `gorilla/websocket`
- **Scaling**: Redis Pub/Sub, RabbitMQ, Kafka for multi-server deployments

---

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
