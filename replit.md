# Mess Manager

Smart hostel mess (canteen) food & attendance tracking app. Students log daily lunch/dinner attendance, track their monthly meal bill, and get reminders. Admins manage the mess and view all user records.

## Run & Operate

```bash
# Install dependencies
npm install --prefix artifacts/mess-tracker
npm install --prefix artifacts/api-server

# Development
npm run dev --prefix artifacts/mess-tracker    # frontend (port from $PORT)
npm run dev --prefix artifacts/api-server      # API server (port 8080)

# Production build
npm run build --prefix artifacts/mess-tracker  # → artifacts/mess-tracker/dist/
npm run build --prefix artifacts/api-server    # → artifacts/api-server/dist/

# Vercel (frontend only)
# installCommand: npm install --prefix artifacts/mess-tracker
# buildCommand:   npm run build --prefix artifacts/mess-tracker
# outputDirectory: artifacts/mess-tracker/dist
```

## Stack

- npm (standalone packages, no workspace protocol)
- Node.js 24, TypeScript 5.9
- Frontend: Vite + React 19, Tailwind CSS v4, wouter, shadcn/ui
- API: Express 5, pino logging
- Auth & Data: Supabase (auth + Postgres via supabase-js)
- DB (chatbot): PostgreSQL + Drizzle ORM (conversations/messages tables)
- AI Chatbot: Groq SDK (MessBot)
- Build: esbuild (bundled ESM for api-server)

## Where things live

```
artifacts/
  mess-tracker/          ← React/Vite frontend (standalone npm package)
    src/
      lib/api-client-react/  ← inlined from lib/api-client-react (generated hooks)
      pages/, components/, hooks/, lib/
    package.json           ← all real versions (no catalog:, no workspace:*)
    vite.config.ts
    tsconfig.json
  api-server/            ← Express backend (standalone npm package)
    src/
      lib/api-zod/           ← inlined from lib/api-zod (generated Zod schemas)
      lib/db/                ← inlined from lib/db (Drizzle ORM + schema)
      routes/, middlewares/, lib/
    package.json
    build.mjs              ← esbuild bundler (with @workspace/* alias)
    tsconfig.json          ← paths for @workspace/api-zod, @workspace/db
lib/
  api-spec/openapi.yaml  ← OpenAPI spec (source of truth)
  api-client-react/      ← source (inlined into mess-tracker)
  api-zod/               ← source (inlined into api-server)
  db/                    ← source (inlined into api-server)
vercel.json              ← Vercel deployment config (frontend)
pnpm-workspace.yaml      ← minimal shim (needed for Replit workflow commands)
```

## Architecture decisions

- Auth is fully Supabase-based (not Clerk). Frontend uses supabase-js directly; backend verifies JWT via supabase.auth.getUser().
- Most data (mess_entries, user_settings) lives in Supabase Postgres, accessed via supabase-js (not Drizzle). Drizzle is used only for chatbot conversations/messages tables.
- Admin access is determined by ADMIN_USER_IDS env var (comma-separated emails or user IDs).
- Groq (groq-sdk) powers MessBot — falls back gracefully if GROQ_API_KEY is absent.
- UI is in Marathi (Devanagari script) with English fallbacks.
- **Lib inlining**: `@workspace/api-client-react`, `@workspace/api-zod`, `@workspace/db` are copied into their consuming artifact's `src/lib/` and resolved via path aliases in tsconfig + vite config + esbuild. Original imports (`@workspace/*`) still work unchanged.
- **pnpm-workspace.yaml shim**: Replit's artifact-managed workflows run `pnpm --filter @workspace/* run dev`. A minimal pnpm-workspace.yaml pointing to `artifacts/*` is kept so pnpm can locate packages. The scripts themselves invoke npm.

## Product

- **Landing page** — marketing page with sign up / log in CTAs
- **Dashboard** — mark today's lunch/dinner, view monthly summary card
- **Analytics** — charts for attendance rate, meal cost trends
- **Range Tracker** — view and edit entries across a custom date range
- **History** — paginated log of all past entries
- **Settings** — set meal price per meal, configure reminder times
- **Admin panel** — admin-only: manage all users and entries
- **MessBot chatbot** — AI assistant powered by Groq, answers mess-related questions in Marathi/English

## Required env vars / secrets

- `SUPABASE_URL` / `VITE_SUPABASE_URL` — Supabase project URL
- `VITE_SUPABASE_ANON_KEY` — Supabase anon/public key (frontend)
- `SUPABASE_SERVICE_KEY` — Supabase service_role key (backend)
- `SUPABASE_DB_PASSWORD` — used by lib/db for direct Supabase Postgres connection
- `GROQ_API_KEY` — Groq API key for MessBot chatbot
- `ADMIN_USER_IDS` — comma-separated admin emails/user IDs
- `VITE_PINGRAM_CLIENT_ID` / `PINGRAM_API_KEY` / `PINGRAM_CLIENT_SECRET` — push notifications

## User preferences

_None yet._

## Gotchas

- `pnpm-workspace.yaml` is intentionally minimal (only `artifacts/*`) — it's a compatibility shim for Replit's artifact-managed workflow runner which uses pnpm filter commands. Do NOT add `lib/*` back; the lib sources are now inlined into each artifact.
- The `@workspace/api-client-react` import in frontend and `@workspace/api-zod` / `@workspace/db` imports in the backend resolve via path aliases (vite.config alias + tsconfig paths + esbuild alias). Do not remove these aliases.
- After any OpenAPI spec change, re-run codegen and copy the generated files into both artifact `src/lib/` directories:
  - `lib/api-client-react/src/generated/` → `artifacts/mess-tracker/src/lib/api-client-react/generated/`
  - `lib/api-zod/src/generated/` → `artifacts/api-server/src/lib/api-zod/generated/`
- Admin check uses ADMIN_USER_IDS env var — add emails or Supabase user IDs there.
- For Vercel deploy: only the frontend is deployed via vercel.json. The api-server must be hosted separately (e.g., Railway, Render, or another Vercel project).
