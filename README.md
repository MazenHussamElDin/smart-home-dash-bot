# Smart Home Dashboard

Ramadan, Mazen, 22405171

**Repository:** https://mygit.th-deg.de/mr06171/smart-home-dash-bot  
**Wiki:** https://mygit.th-deg.de/mr06171/smart-home-dash-bot/-/wikis/home

---

## Project Description

The Smart Home Dashboard is a full-stack web application that provides a centralized, at-a-glance overview of smart home sensor data and daily information. It aggregates data from real system sensors (CPU, memory, disk, network via `systeminformation`), simulated IoT sensors (temperature, humidity, CO2), and external APIs (weather via OpenWeatherMap, Mensa menu via OpenMensa) into a single responsive interface.

Users can monitor real-time and historical sensor data through interactive charts, manage sensor settings, and register/login with JWT-based authentication.

**Stack:** Vue 3 (Composition API) · Express.js · MongoDB · TypeScript · Docker

---

## Project Planning

Full planning documentation is in the [Wiki](https://mygit.th-deg.de/mr06171/smart-home-dash-bot/-/wikis/home):

| Page | Content |
|---|---|
| [Personas](https://mygit.th-deg.de/mr06171/smart-home-dash-bot/-/wikis/Personas) | Prof. Dr. Markus Weber · Lisa Meier |
| [Use Cases](https://mygit.th-deg.de/mr06171/smart-home-dash-bot/-/wikis/Use-Cases) | 4 use cases, Cockburn Casual, diagrams, error cases |
| [Architecture](https://mygit.th-deg.de/mr06171/smart-home-dash-bot/-/wikis/Architecture) | Three-tier architecture, data flow, API overview |
| [Frontend Layout](https://mygit.th-deg.de/mr06171/smart-home-dash-bot/-/wikis/Frontend-Layout-Mockup) | Mockup, responsive breakpoints |

---

## Architecture

```
┌─────────────────┐     HTTP/JSON     ┌──────────────────┐     Mongoose    ┌─────────────┐
│  Vue 3 Frontend │ ────────────────► │ Express.js API   │ ──────────────► │  MongoDB 7  │
│  (Vite / TS)    │ ◄──────────────── │ (Node.js / TS)   │ ◄────────────── │  (Docker)   │
└─────────────────┘                   └──────────────────┘                 └─────────────┘
                                              │
                                   ┌──────────┴──────────┐
                                   │  External APIs       │
                                   │  OpenWeatherMap      │
                                   │  OpenMensa           │
                                   │  systeminformation   │
                                   └─────────────────────┘
```

**Frontend** — Vue 3, Composition API, Vue Router, Pinia, Chart.js, Bootstrap 5  
**Backend** — Express.js, Mongoose, JWT (jsonwebtoken + bcryptjs), Pino, express-rate-limit  
**Database** — MongoDB 7 via Docker, persistent volume

---

## Prerequisites

- Node.js v22 LTS (managed via nvm)
- Docker + Docker Compose
- Git

---

## Quick Start (Development)

```bash
# 1. Clone
git clone https://mygit.th-deg.de/mr06171/smart-home-dash-bot.git
cd smart-home-dash-bot

# 2. Start MongoDB
docker-compose up mongodb -d

# 3. Backend
cd Backend
cp .env.example .env        # edit JWT_SECRET and OPENWEATHER_API_KEY if needed
npm install
npm run seed                 # populate DB with 48h demo data
npm run dev                  # http://localhost:3000

# 4. Frontend (new terminal)
cd frontend
npm install
npm run dev                  # http://localhost:5173
```

Open **http://localhost:5173**, register an account, and explore the dashboard.

---

## Docker (Full Stack)

Run the entire application with one command:

```bash
# Build and start all services (MongoDB + Backend + Frontend)
docker-compose up --build

# Run in background
docker-compose up --build -d

# Stop
docker-compose down

# Stop and remove volumes (clears database)
docker-compose down -v
```

The frontend is served at **http://localhost:5173** via nginx, which proxies `/api` to the backend.

---

## Environment Variables

Create `Backend/.env` from `Backend/.env.example`:

| Variable | Default | Description |
|---|---|---|
| `PORT` | `3000` | Express server port |
| `MONGODB_URI` | `mongodb://localhost:27017/smarthome` | MongoDB connection string |
| `JWT_SECRET` | *(see .env.example)* | Secret for JWT signing — **change in production** |
| `FRONTEND_URL` | `http://localhost:5173` | Allowed CORS origin |
| `OPENWEATHER_API_KEY` | *(empty)* | OpenWeatherMap API key — leave empty for mock data |

---

## Project Structure

```
smart-home-dash-bot/
├── Backend/
│   ├── src/
│   │   ├── config/         # SettingsManager singleton (settings.json)
│   │   ├── middleware/      # JWT auth middleware
│   │   ├── models/         # Mongoose schemas (AuthUser, SensorReading, User)
│   │   ├── routes/         # Express Router per resource
│   │   ├── services/       # SensorCollector background service
│   │   └── utils/          # Pino logger
│   ├── .env.example
│   ├── Dockerfile
│   ├── package.json
│   └── settings.json       # Auto-generated sensor config
├── frontend/
│   ├── src/
│   │   ├── api/            # Fetch wrappers per resource
│   │   ├── components/     # NavBar
│   │   ├── router/         # Vue Router + auth guards
│   │   ├── stores/         # Pinia auth store
│   │   └── views/          # Dashboard, Sensors, History, Settings, Users, Login, Register
│   ├── Dockerfile
│   ├── nginx.conf
│   └── package.json
├── docker-compose.yml
└── README.md
```

---

## API Overview

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/auth/register` | Register new account |
| POST | `/api/auth/login` | Login, receive JWT |
| GET | `/api/auth/me` | Current user (protected) |
| GET | `/api/sensors` | List readings (filter by sensorId, type, date) |
| GET | `/api/sensors/latest` | Latest reading per sensor |
| GET | `/api/sensors/stats` | Min/max/avg per sensor |
| POST | `/api/sensors` | Store new reading |
| GET | `/api/users` | List all users |
| POST | `/api/users` | Create user (form submission) |
| PUT | `/api/users/:id` | Update user |
| DELETE | `/api/users/:id` | Delete user |
| GET | `/api/settings` | Get full settings |
| PUT | `/api/settings` | Update settings (persists to settings.json) |
| POST | `/api/settings/sensors/test` | Reachability test for sensor endpoint |
| GET | `/api/external/weather` | Weather data (OpenWeatherMap proxy) |
| GET | `/api/external/mensa` | Mensa menu (OpenMensa proxy) |
| GET | `/api/system/all` | All real system metrics at once |

---

## Implementation of Course Requirements

| Requirement | Implementation |
|---|---|
| Vue 3 Composition API | All views use `<script setup>` with `ref`, `reactive`, `computed`, `onMounted` |
| At least 1 form saving to DB | Users view (POST /api/users → MongoDB) + Settings view (PUT /api/settings) |
| JWT authentication | bcryptjs password hashing, jsonwebtoken, authMiddleware, Pinia auth store |
| MongoDB / Mongoose | 3 schemas: AuthUser, SensorReading, User — compound indexes, timestamps |
| Express Router objects | Separate Router per resource in `routes/` |
| `.env` configuration | `dotenv`, `.env.example` provided, all secrets via env vars |
| Dynamic chart | Live Chart.js line chart in SensorsView polling `/api/system/all` every 1.5s |
| Static chart | Historical Chart.js chart in HistoryView with sensor/time-range selector |
| API configurability via JSON | `settings.json` read at startup by SettingsManager singleton |
| Reachability test | `POST /api/settings/sensors/test` with 3s timeout, used in Settings UI |
| Personas | Wiki: Prof. Dr. Markus Weber + Lisa Meier with descriptions |
| Use Cases | Wiki: 4 UCs in Cockburn Casual format, diagram, error cases |
| Bootstrap 5 responsive | Mobile navbar collapse, responsive grid breakpoints (sm/md/lg) |
| CORS | `cors` middleware configured for frontend origin |
| Rate limiting | `express-rate-limit`: 200 req/15min general, 20 req/15min auth routes |
| Input validation | Email regex, required fields, numeric range checks in all routes and forms |
| Docker | MongoDB + Backend + Frontend containers, nginx proxy, persistent volume |
| Seed data | `npm run seed` — 48h sensor history, 3 demo users |
