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
npx prisma migrate dev   # creates/updates the database tables
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
npx expo start
```

This starts the Metro bundler and opens an interactive menu in your
terminal. From there, press:

- **`i`** — opens the app in the iOS Simulator, running on your Mac.
  Requires Xcode to be fully installed (Xcode > Settings, or the App
  Store) — the CLI will prompt you to install it if it isn't.
- **`a`** — same, but for the Android Emulator (requires Android Studio).
- Scan the QR code shown in the terminal with your phone's camera
  (iOS) or the Expo Go app (Android) to run it on a physical device
  instead — no simulator/emulator install needed. Your phone must be
  on the same Wi-Fi network as your laptop.
- **`w`** — opens the app in a browser tab via React Native Web.

Alternatively, `npm run ios` / `npm run android` / `npm run web` skip
the menu and launch a specific target directly.
