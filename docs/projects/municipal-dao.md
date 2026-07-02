# Municipal DAO

Source: [`360magicians/municipal-dao`](https://github.com/360magicians/municipal-dao)

> On-chain governance DAO platform with real-time WebSocket voting and notifications.

---

## Overview

Municipal DAO is a decentralized governance platform that enables Deaf communities and municipal
stakeholders to participate in transparent, real-time on-chain voting.

---

## Real-Time WebSocket Integration

### Features

#### 🔄 Real-Time Voting
- Live vote counting with animated updates
- Quorum progress tracking with visual indicators
- Recent votes display (who voted and when)
- Live viewer count for active proposal watchers
- Instant vote casting with real-time feedback

#### 🔔 Real-Time Notifications
- Instant notifications for votes, comments, and system updates
- Priority-based alerts (low, medium, high)
- Browser notifications for high-priority items
- Connection status indicators showing WebSocket health
- Notification management (mark as read, clear, etc.)

#### 🌐 WebSocket Infrastructure
- Robust connection management with auto-reconnection
- Room-based messaging for proposal-specific updates
- Message type routing for different notification types
- Authentication integration with user sessions
- Error handling and recovery for network issues

---

## Technical Architecture

### Client-Side

| File | Purpose |
|------|---------|
| `lib/websocket.ts` | Singleton WebSocket connection manager with exponential backoff |
| `hooks/useRealTimeVoting.ts` | Live vote count updates, quorum tracking, vote casting |
| `hooks/useRealTimeNotifications.ts` | Notification state, browser notifications, read/unread status |
| `components/RealTimeVotingCard` | Animated voting interface |
| `components/RealTimeNotifications` | Notification dropdown with live updates |

### Server-Side

- **Node.js WebSocket server** — `server/websocket-server.js`
- Room management for proposal-specific messaging
- Message broadcasting to relevant users
- Authentication verification
- Graceful shutdown handling

---

## Getting Started

```bash
# Install dependencies
npm install

# Start the WebSocket server (development)
npm run ws-server

# Start the Next.js application
npm run dev
```

---

## Environment Variables

```env
# WebSocket server configuration
WS_PORT=8080
WS_HOST=localhost

# Production WebSocket URL
NEXT_PUBLIC_WS_URL=wss://your-domain.com/ws
```

---

## Message Types

### Vote Message
```json
{
  "type": "vote",
  "data": {
    "proposalId": "1",
    "userId": "alice.eth",
    "choice": "for",
    "votingPower": 12.5,
    "reason": "Strong proposal with clear benefits"
  },
  "timestamp": "2024-01-20T10:30:00Z"
}
```

### Notification Message
```json
{
  "type": "notification",
  "data": {
    "id": "notification-123",
    "type": "vote",
    "title": "New Vote Cast",
    "message": "alice.eth voted FOR on Marketing Budget proposal",
    "priority": "medium",
    "actionUrl": "/proposals/1"
  },
  "timestamp": "2024-01-20T10:30:00Z"
}
```

### System Message (quorum reached)
```json
{
  "type": "quorum_reached",
  "data": {
    "proposalId": "1",
    "quorum": 51.2,
    "proposalTitle": "Marketing Budget Allocation"
  },
  "timestamp": "2024-01-20T10:30:00Z"
}
```

---

## Security

1. **Authentication** — All WebSocket connections require valid user authentication
2. **Rate Limiting** — Prevent spam
3. **Input Validation** — All incoming messages are validated
4. **CORS Configuration** — Proper CORS setup for cross-origin requests
5. **SSL/TLS** — Use secure WebSocket connections (`wss://`) in production

---

## Ecosystem Architecture

The DAO sits within the **mbtquniverse.com** platform:
- `dao-governance`
- `web3-staking`
- `showcase-projects`
- `community-treasury`

See [Infrastructure Docs](../infrastructure.md) for full ecosystem context.

---

## Future Enhancements

- Horizontal scaling via Redis-based message broadcasting
- Message persistence for offline users
- User-customizable notification preferences
- Mobile push notification integration
- Real-time analytics dashboard
