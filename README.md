# MQTT Football Scores

A real-time football (soccer) scores demo that uses **MQTT** (Eclipse Mosquitto) as the message backbone and **Server-Sent Events (SSE)** to push live updates to the browser.

## What it does

- **Admin** — Create matches, update scores, and add events (goals, cards, substitutions, etc.). All changes are published to MQTT and streamed to viewers.
- **Viewer** — See live scores and a live feed of match updates. Connects via SSE to the server, which forwards MQTT messages in real time.
- **API** — REST endpoints for matches and score/event updates; an SSE endpoint for real-time streaming.

## Tech stack

- **Node.js** (ES modules) + **Express** — HTTP API and static files
- **MQTT** — [mqtt](https://www.npmjs.com/package/mqtt) client; connects to a broker and publishes/subscribes to football topics
- **Eclipse Mosquitto** — MQTT broker (run via Docker)
- **SSE** — Server forwards MQTT messages to browser clients over `/api/events`

## Quick start

### 1. Start the MQTT broker

```bash
docker compose up -d
```

This runs Mosquitto on port **1883** using `mosquitto.conf` (anonymous access, stdout logging).

### 2. Start the app

```bash
npm install
npm start
```

Server runs at **http://localhost:3000** (override with `PORT`).

### 3. Open the UI

| Page | URL | Purpose |
|------|-----|--------|
| Home | http://localhost:3000 | Links to Viewer and Admin |
| Viewer | http://localhost:3000/viewer.html | Live scores and feed |
| Admin | http://localhost:3000/admin.html | Create matches, update scores, add events |

Use Admin to create a match and change scores/events; open Viewer in another tab to see updates in real time.

## Environment

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `3000` | HTTP server port |
| `MQTT_BROKER` | `mqtt://localhost:1883` | MQTT broker URL |

Example: `MQTT_BROKER=mqtt://your-broker:1883 PORT=4000 npm start`

## API

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/matches` | List all matches (in-memory). |
| `POST` | `/api/matches` | Create a match. Body: `homeTeam`, `awayTeam`, optional `league`, `venue`, `kickoff`. |
| `PATCH` | `/api/matches/:id/score` | Update score/state. Body: `homeScore`, `awayScore`, optional `minute`, `status`. |
| `POST` | `/api/matches/:id/events` | Add event. Body: `type` (goal, yellow_card, red_card, substitution, penalty), `team`, `player`, optional `minute`, `description`. |
| `GET` | `/api/events` | SSE stream of all MQTT messages under `sports/football/#`. |

Matches are kept in memory; restarting the server clears them. MQTT is used for real-time distribution; the server subscribes to `sports/football/#` and republishes received messages to SSE clients.

## MQTT topics

| Topic | Usage |
|-------|--------|
| `sports/football/match/<id>` | Full match object (create/update). |
| `sports/football/scores` | Score and status updates; payload includes `type` (e.g. `match_created`, `score_update`). |
| `sports/football/events` | Match events; payload includes `type: 'match_event'`, `matchId`, `event`. |

The server subscribes to `sports/football/#` (QoS 1) and publishes with QoS 1. You can add other MQTT clients (e.g. mobile or scripts) that subscribe to these topics to build more consumers.

## Project structure

```
mqtt-football-scores/
├── server/
│   ├── index.js      # Express app, MQTT connection, routes
│   ├── sse.js        # SSE: subscribe to MQTT, stream to clients
│   └── routes/
│       └── matches.js # Match CRUD + MQTT publish helpers
├── public/
│   ├── index.html    # Landing (links to Viewer / Admin)
│   ├── viewer.html   # Live scores + feed (SSE)
│   └── admin.html    # Create match, update score, add events
├── docker-compose.yml # Mosquitto service
├── mosquitto.conf    # Broker config (listener 1883, anonymous)
└── package.json
```

## License

ISC
