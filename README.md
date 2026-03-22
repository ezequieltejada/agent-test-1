# Dev Practice with Agents

A webapp built with Angular (frontend), HonoJS (backend), and SQLite (dev) / PostgreSQL (production), developed inside a fully isolated dev container.

---

## Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Angular 21 |
| Backend | HonoJS on Bun |
| Database (dev) | SQLite |
| Database (prod) | PostgreSQL 16 |
| ORM | Drizzle ORM |
| Package manager | npm (frontend) / Bun (backend) |

---

## Project Structure

```
.
├── .devcontainer/
│   ├── Dockerfile.dev          # Dev container: Node 22 + Bun + Angular CLI + OpenCode
│   ├── docker-compose.dev.yml  # Dev container orchestration
│   └── devcontainer.json       # Editor integration (VS Code / OpenCode)
├── frontend/
│   ├── Dockerfile              # Prod: multi-stage Node build → Nginx
│   ├── nginx.conf              # SPA routing + /api/ proxy
│   └── ...                     # Angular 21 project
├── backend/
│   ├── Dockerfile              # Prod: multi-stage Bun build → minimal runtime
│   └── ...                     # HonoJS project
├── docker-compose.prod.yml     # Production: frontend + backend + PostgreSQL
├── .dockerignore
└── .gitignore
```

---

## Environments

### Development (Dev Container)

A single, fully isolated container with all tools pre-installed:

- Node.js 22
- Bun (backend runtime + package manager)
- Angular CLI (`ng`)
- OpenCode CLI (AI coding agent)
- SQLite

**Prerequisites:** Docker Desktop with WSL2 integration enabled.

#### Connecting with Zed (recommended)

Zed v0.218+ has native Dev Container support -- no SSH required. It reads `.devcontainer/devcontainer.json`, starts the container, and automatically installs `zed-remote-server` inside.

1. Open Zed
2. `Ctrl+Shift+P` → `dev containers: open folder in container`
3. Select this project folder
4. Zed builds the container on first run (takes a few minutes) and connects
5. Open a terminal inside Zed and run `opencode`, `ng serve`, `bun run dev`, etc.

#### Connecting via terminal

**Start the dev container:**

```bash
docker compose -f .devcontainer/docker-compose.dev.yml up -d
```

**Shell into it:**

```bash
docker compose -f .devcontainer/docker-compose.dev.yml exec devcontainer bash
```

**Ports forwarded to host:**

| Port | Service |
|------|---------|
| 4200 | Angular dev server |
| 3000 | HonoJS backend |

**Run the frontend (inside the container):**

```bash
cd /workspace/frontend
npm start
```

**Run the backend (inside the container):**

```bash
cd /workspace/backend
bun run dev
```

**Stop the dev container:**

```bash
docker compose -f .devcontainer/docker-compose.dev.yml down
```

---

### Production

Three containers orchestrated with Docker Compose:

- **frontend**: Angular build served by Nginx on port 80. Proxies `/api/` to the backend.
- **backend**: HonoJS running on Bun, port 3000.
- **postgres**: PostgreSQL 16 with a named volume for persistence.

All services run as non-root users and include health checks.

**Build and start:**

```bash
docker compose -f docker-compose.prod.yml up -d --build
```

**Stop:**

```bash
docker compose -f docker-compose.prod.yml down
```

**Stop and remove volumes (destructive):**

```bash
docker compose -f docker-compose.prod.yml down -v
```

---

## Database

The ORM is [Drizzle ORM](https://orm.drizzle.team), which supports both SQLite (dev) and PostgreSQL (prod) with minimal code changes.

The `DATABASE_URL` environment variable controls which database is used:

| Environment | Value |
|-------------|-------|
| Development | SQLite file (e.g. `./data/dev.db`) |
| Production | `postgresql://appuser:apppassword@postgres:5432/appdb` |

> The production database password should be changed and managed via a secrets manager or `.env` file that is **not** committed to git.

---

## Configuration

OpenCode is configured via `opencode.json` at the project root. The Perplexity MCP server is enabled and reads its API key from `.secrets/PERPLEXITY_API_KEY` (git-ignored).
