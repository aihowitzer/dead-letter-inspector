# dead-letter-inspector

> Visual dashboard for RabbitMQ dead letter queues. Inspect, replay, purge — without touching the management UI.

[![Node.js](https://img.shields.io/badge/Node.js-18+-339933?style=flat-square&logo=node.js&logoColor=white)](https://nodejs.org)
[![RabbitMQ](https://img.shields.io/badge/RabbitMQ-3.x+-FF6600?style=flat-square&logo=rabbitmq&logoColor=white)](https://rabbitmq.com)
[![React](https://img.shields.io/badge/React-18+-61DAFB?style=flat-square&logo=react&logoColor=black)](https://react.dev)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)
[![Author](https://img.shields.io/badge/by-aihowitzer.com-FF6A00?style=flat-square)](https://aihowitzer.com)

---

## The problem

Dead letter queues are where your messages go to die — silently.

The default RabbitMQ management UI shows you the count. That's it. You can't easily see *why* a message failed, *what* it contained, or *replay* it without writing custom scripts at 2am while your on-call phone is ringing.

**dead-letter-inspector** gives you a proper UI to deal with DLQs like a human.

---

## Features

```
📋  Inspect       — read message payload, headers, and death reason
🔁  Replay        — requeue single messages or entire DLQ to origin queue
🗑️  Purge         — clear DLQ with one click (with confirmation)
🔍  Filter        — search messages by payload content, routing key, error type
📊  Timeline      — see when messages started dying and at what rate
🚨  Alerts        — get notified when DLQ depth crosses your threshold
📤  Export        — download DLQ contents as JSON or CSV
⚡  Realtime      — live updates via SSE, no refresh needed
```

---

## Quickstart

```bash
# Clone
git clone https://github.com/aihowitzer/dead-letter-inspector
cd dead-letter-inspector

# Install
npm install

# Configure
cp .env.example .env
# edit .env with your RabbitMQ connection details

# Run
npm start

# Open
http://localhost:3001
```

---

## .env config

```env
RMQ_HOST=localhost
RMQ_PORT=5672
RMQ_MANAGEMENT_PORT=15672
RMQ_USERNAME=admin
RMQ_PASSWORD=yourpassword
RMQ_VHOST=/

# Alert threshold — notify when DLQ exceeds this count
DLQ_ALERT_THRESHOLD=100

# Optional — Slack webhook for alerts
SLACK_WEBHOOK_URL=https://hooks.slack.com/your-webhook
```

---

## Dashboard preview

```
┌─────────────────────────────────────────────────────────────┐
│  DEAD LETTER INSPECTOR          ● LIVE    [↻ 3s ago]        │
├──────────────────┬──────────────┬──────────────┬────────────┤
│  QUEUE           │  DEPTH       │  RATE        │  ACTIONS   │
├──────────────────┼──────────────┼──────────────┼────────────┤
│  orders.dlq      │  12,443  🔴  │  +142/min    │ [inspect]  │
│  webhooks.dlq    │     891  ⚠️  │   +12/min    │ [inspect]  │
│  notif.dlq       │      34  ✓   │    +1/min    │ [inspect]  │
│  payments.dlq    │       0  ✓   │       0      │ [inspect]  │
└──────────────────┴──────────────┴──────────────┴────────────┘

  INSPECTING: orders.dlq  (12,443 messages)
  ┌────────────────────────────────────────────────────────┐
  │ #1  order_id: ORD-9182  |  failed: 3x  |  reason: NACK│
  │ ─────────────────────────────────────────────────────  │
  │ payload: { "order_id": "ORD-9182",                     │
  │            "amount": 4299,                             │
  │            "customer": "rahul@example.com" }           │
  │                                                        │
  │ death_reason  : rejected                               │
  │ original_queue: orders.processing                      │
  │ first_death   : 2026-04-22T14:33:00Z                   │
  │ death_count   : 3                                      │
  │                                                        │
  │  [↩ REPLAY]  [🗑 DELETE]  [📤 EXPORT]                 │
  └────────────────────────────────────────────────────────┘
```

---

## API

The backend exposes a REST API if you want to integrate with your own tooling:

```bash
# List all DLQs with stats
GET /api/queues

# Get messages from a specific DLQ
GET /api/queues/:name/messages?limit=50&offset=0

# Replay a single message back to origin queue
POST /api/queues/:name/messages/:messageId/replay

# Replay entire DLQ
POST /api/queues/:name/replay-all

# Purge a DLQ
DELETE /api/queues/:name/purge

# Get DLQ depth timeline (last 24h)
GET /api/queues/:name/timeline
```

---

## Architecture

```
┌─────────────────┐     SSE      ┌──────────────────┐
│   React UI      │◄─────────────│   Node.js API    │
│   (port 3001)   │              │   (port 3000)    │
└─────────────────┘              └────────┬─────────┘
                                          │
                                  AMQP + HTTP
                                          │
                                 ┌────────▼─────────┐
                                 │    RabbitMQ      │
                                 │  Management API  │
                                 └──────────────────┘
```

---

## Roadmap

- [ ] Multi-vhost support
- [ ] Kafka dead letter topic support
- [ ] Azure Service Bus dead letter viewer
- [ ] Regex filter on message payload
- [ ] Bulk replay with rate limiting
- [ ] Webhook alerts (Slack, Teams, WhatsApp)

---

## DLQs overflowing in prod right now?

This tool helps you see what's in the queue. If you need someone to figure out **why messages are dying** and fix it permanently — that's what I do.

**[→ Book a free 30-min audit at aihowitzer.com](https://aihowitzer.com)**

Fixed-price. No hourly surprises. Starts with a free call.

---

## License

MIT © [AI Howitzer](https://aihowitzer.com)
