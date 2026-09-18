# storyShelf

## Local Development

**Prerequisites:** Docker Desktop running, Node.js installed.

### 1. Start Postgres

From the repo root:

```bash
docker compose up -d
```

This starts a Postgres 17 container on `localhost:5433`, matching the
`DATABASE_URL` in `backend/.env.example`. Data persists between restarts
in a named Docker volume — just run `docker compose up -d` again for
subsequent sessions (it's a no-op if the container is already running).

### 2. Start the backend

```bash
cd backend
cp .env.example .env   # fill in real values first time
npm install
npm run dev
```

Runs the Express API on `http://localhost:3000`. Verify with:

```bash
curl localhost:3000/health
```

Expected output:

```json
{"status":"ok"}
```

### 3. Start the mobile app

```bash
cd mobile
npm install
npm run ios     # or: npm run android / npm run web
```
