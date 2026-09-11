# PageToScreen — Build Roadmap

A sequence of milestones, not calendar weeks — move to the next one once the current one demos cleanly. Checkboxes are meant to actually be checked off as you go.

---

## Milestone 0 — Repo, tooling, and local environment

Goal: an empty-but-real project you can `git clone` and run, before any feature code exists.

### 0.1 Create the repo

In GitHub Desktop: **File → New Repository**.
- Name: `pagetoscreen`
- Local path: wherever you keep projects (Desktop creates the `pagetoscreen/` folder for you)
- Git ignore: **Node**
- Leave "Initialize with a README" checked if you like — you'll overwrite it later

Hit **Create Repository**. Desktop initializes git, makes the first commit with the `.gitignore`/README, and opens the repo. Don't publish to GitHub yet — do that in Milestone 0.5, once there's a real scaffold to push.

One monorepo (`backend/` + `mobile/` + docs at the root) is the right call for a solo project — one README, one commit history, one link to send a recruiter. Don't split into two repos unless you have a specific reason to.

### 0.2 Root-level files

Create `README.md` and put `DESIGN.md` and this file at the repo root too.

GitHub Desktop already generated a `.gitignore` from the Node template. Open it in your editor and make sure it covers:
```
node_modules/
.env
.env.*.local
dist/
build/
.expo/
*.log
.DS_Store
```
Add any of these lines that the template is missing — `.env` and `.expo/` especially.

### 0.3 Backend scaffold

```bash
mkdir backend && cd backend
npm init -y
npm install express cors zod jsonwebtoken google-auth-library @prisma/client
npm install -D typescript ts-node-dev @types/express @types/node @types/cors @types/jsonwebtoken prisma
npx tsc --init
npx prisma init
```

Then drop in the files you already have:
- `prisma/schema.prisma` ← your existing schema
- `src/app.ts`, `src/lib/`, `src/middleware/`, `src/modules/` ← your existing Express code

Add scripts to `package.json`:
```json
"scripts": {
  "dev": "ts-node-dev --respawn src/main.ts",
  "build": "tsc",
  "start": "node dist/main.js"
}
```

You'll need a small `src/main.ts` that isn't written yet — first real code to add:
```typescript
import app from './app';

const port = process.env.PORT ?? 3000;
app.listen(port, () => console.log(`API listening on port ${port}`));
```

### 0.4 Local Postgres via Docker Compose

Don't install Postgres directly on your machine — one `docker-compose.yml` at the repo root means anyone (including future-you on a new laptop) can spin up the exact same DB:

```yaml
services:
  postgres:
    image: postgres:16
    restart: unless-stopped
    environment:
      POSTGRES_USER: pagetoscreen
      POSTGRES_PASSWORD: devpassword
      POSTGRES_DB: pagetoscreen
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

```bash
docker compose up -d
```

### 0.5 Environment variables

`backend/.env` (never committed):
```
DATABASE_URL="postgresql://pagetoscreen:devpassword@localhost:5432/pagetoscreen"
JWT_ACCESS_SECRET="dev-only-change-me"
JWT_REFRESH_SECRET="dev-only-change-me-too"
GOOGLE_CLIENT_ID="get-this-from-google-cloud-console"
```

Also commit a `backend/.env.example` with the same keys and no real values — it's the file that tells the next developer (or you, in six months) what's needed without leaking secrets. Small detail, but it quietly signals you know what you're doing.

### 0.6 First migration

```bash
npx prisma migrate dev --name init
```

Creates every table from the schema in one shot — a real, working database.

### 0.7 Mobile scaffold

```bash
cd ..
npx create-expo-app mobile -t expo-template-blank-typescript
cd mobile
npm install axios expo-secure-store @react-navigation/native @react-navigation/native-stack
```

Drop your existing `src/api/client.ts` and `src/screens/*` files in.

---

## Milestone 0.5 — the initial commit

This is the point to commit — a working scaffold, not a finished feature. First commits should prove the project runs, nothing more.

In GitHub Desktop:
1. Open the **Changes** tab and scan the file list — sanity check: no `node_modules/`, no `.env`, nothing huge. If any of those show up, fix `.gitignore` before committing.
2. Summary: `Initial commit: project scaffold (Express + Prisma backend, Expo mobile app)`
3. Click **Commit to main**.

Then push it up:
- Click **Publish repository** in the top bar.
- Name `pagetoscreen`, uncheck "Keep this code private" if you want it visible to recruiters, and publish.
- After that, new commits go up with the **Push origin** button.

From here, commit in small, real chunks tied to one milestone/feature each — use summaries like `feat: add Google auth routes`, `feat: adaptation accuracy voting + auto-approval`, etc. A readable commit history is one of the few parts of a portfolio project a recruiter might actually scroll through.

---

## Milestone 1 — Walking skeleton (thinnest possible end-to-end slice)

Goal: prove the whole system works together before building any real feature. This is a legitimate engineering practice worth naming in an interview — "I built a thin vertical slice end-to-end before adding breadth."

- [ ] `GET /health` on the backend returns `200 OK`
- [ ] Mobile app makes one successful request to that endpoint and renders the result
- [ ] `POST /auth/google` works against a real Google account (needs Google Cloud Console setup first — see 1.1)
- [ ] The access token from that response successfully authenticates a `GET /logs` call

Once that loop works end to end, every other feature is "more of the same shape," not a new architecture.

### 1.1 Google Cloud Console setup (needed before auth can work at all)
- [ ] Create a project in Google Cloud Console
- [ ] Configure the OAuth consent screen (internal/testing scope is fine for now)
- [ ] Create an OAuth Client ID (you'll likely need one config for the Expo app's sign-in flow, verified server-side against the same client ID)
- [ ] Drop the client ID into `GOOGLE_CLIENT_ID` in `.env`

---

## Milestone 2 — Core logging loop

- [ ] `POST /logs`, `GET /logs`, `DELETE /logs/:id` working against real data (already written — wire up + test)
- [ ] A minimal "log a movie" screen on mobile hitting these endpoints
- [ ] Seed a handful of books/movies by hand (`prisma/seed.ts`) so there's something real to log against before search exists

---

## Milestone 3 — The differentiator: adaptations + accuracy voting

- [ ] Seed a few real `Adaptation` rows (book ↔ movie pairs) by hand
- [ ] `GET /adaptations/:id` and `POST /adaptations/:id/votes` working (already written)
- [ ] Auto-approval verified — vote 100 times (a quick seed script simulating 100 different users is faster than doing it by hand) and confirm `approved` flips
- [ ] `AdaptationDetailScreen` on mobile, showing the pending/verified badge

This is the point where you have a genuinely demoable core product — a reasonable place to record a first demo GIF for the README.

---

## Milestone 4 — Mobile navigation & polish

- [ ] Tab navigation: Feed / Search / Log / Profile
- [ ] Real book/movie search (TMDB/Open Library, cached locally as described in the design doc)
- [ ] Scrollable feed (even a simple "recent logs across all users" query is enough to start)

---

## Milestone 5 — Stand-out features (pick based on what you want to talk about most)

- [ ] Ratings on books/movies (separate from accuracy voting)
- [ ] Adaptation leaderboard
- [ ] Yearly "Wrapped" stats page (design doc §7.6)
- [ ] Notifications for logged books (design doc §7.7)
- [ ] Spotify soundtrack embed (design doc §7.4 — mind the Development Mode constraints)
- [ ] Content-based recommendations (Phase 1 version, design doc §7.5)

---

## Milestone 6 — Engineering polish

- [ ] Tests: unit tests on services (especially the auto-approval logic — the one piece of business logic worth being paranoid about), a few integration tests on key endpoints
- [ ] GitHub Actions CI: lint + test on every push
- [ ] Deploy backend (Railway/Render) against a real hosted Postgres
- [ ] EAS Build for a real installable mobile build
- [ ] Basic error tracking (Sentry free tier)

---

## Milestone 7 (stretch) — Admin dashboard

- [ ] Small Next.js app for reviewing pending adaptations, flagging spam submissions
- [ ] Reuses the same backend API — no new auth system needed

---

**Day to day:** check boxes off as you go. Milestones 4–7 can be reordered freely based on what you find most interesting to build next — the only hard ordering constraint is 0 → 1 → 2 → 3.
