# Shift and Connect

A fast-paced, cyberpunk-themed multiplayer Connect-5 game built with Rust and Vue.js. Unlike traditional Connect-4, players can place pieces anywhere on the grid and use side-shifters to slide entire rows or columns, adding a strategic layer to the classic "N-in-a-row" formula.

## 🚀 Features

- **Dynamic Gameplay**: Connect 5 pieces horizontally, vertically, or diagonally.
- **Shifting Mechanics**: Use arrow buttons to shift rows and columns, moving your pieces and your opponent's.
- **Real-time Multiplayer**: Powered by WebSockets for instantaneous move updates.
- **Lobby System**: Create public games or secure private sessions with a 4-character room code.
- **Stateless Authentication**: Uses local storage to assign player roles (Player 1/Player 2) without needing accounts.
- **Cyberpunk UI**: A high-contrast, neon-glow aesthetic built with Vuetify.

## 🛠️ Tech Stack

- **Backend**: [Rust](https://www.rust-lang.org/) (Axum, SQLx, Tokio)
- **Frontend**: [Vue 3](https://vuejs.org/) (Vite, TypeScript, Vuetify)
- **Database**: [PostgreSQL](https://www.postgresql.org/)
- **Infrastructure**: [Docker](https://www.docker.com/), Docker Compose, Nginx

---

## 📋 Prerequisites

Before you begin, ensure you have the following installed:

- **Docker** (Desktop or Engine)
- **Docker Compose**
- **Git**

---

## 📦 Installation & Setup

### 1. Clone the repository
```bash
git clone https://github.com/JBrownSchue/shiftandconnect.git
cd shiftandconnect
```

### 2. Environment Configuration
The backend requires a `DATABASE_URL`. In a Docker environment, this is handled via `docker-compose.yml`. For local development, create a `.env` file in the `backend/` directory:
```bash
DATABASE_URL=postgres://shift_user:super_secret_password@127.0.0.1:5432/shift_connect
```

### 3. Running with Docker (Recommended)
This command will build the frontend and backend images, set up the PostgreSQL database, and run all migrations automatically.
```bash
docker compose up --build -d
```
Once finished:
- The **Frontend** will be accessible at `http://localhost:6010` (or the port defined in your compose file).
- The **Backend** API will run on `http://localhost:3000`.

---

## 🎮 How to Play

1. **Connect**: Be the first to get 5 of your colored pieces in a line.
2. **Shift**: Use the d-pad arrows on the edges to move a row or column. Shifting "loops" the pieces — the piece pushed out on one side reappears on the other.
3. **Wait for Opponent**: A move can only be made when both players are connected to the room.
