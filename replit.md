# Mess Manager

Smart hostel mess (canteen) food & attendance tracking app. Students log daily lunch/dinner attendance, track their monthly meal bill, and get reminders. Admins manage the mess and view all user records.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080)
- `pnpm --filter @workspace/mess-tracker run dev` — run the frontend (port from $PORT)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: Vite + React 19, Tailwind CSS v4, wouter, shadcn/ui
- API: Express 5, pino logging
- Auth & Data: Supabase (auth + Postgres via supabase-js)
- DB (chatbot): PostgreSQL + Drizzle ORM (conversations/messages tables)
- AI Chatbot: Groq SDK (MessBot)
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle for api-server)

## Where things live

- `artifacts/mess-tracker/` — Vite + React frontend
- `artifacts/api-server/` — Express 5 backend
- `lib/api-spec/openapi.yaml` — OpenAPI spec (source of truth for API)
- `lib/api-zod/src/generated/` — generated Zod schemas (server-side)
- `lib/api-client-react/src/generated/` — generated React Query hooks (client-side)
- `lib/db/src/schema/` — Drizzle schema (mess_entries, user_settings, conversations, messages)
- `artifacts/mess-tracker/src/lib/supabase-client.ts` — Supabase client (frontend)
- `artifacts/api-server/src/lib/supabase.ts` — Supabase admin client (backend)

## Architecture decisions

- Auth is fully Supabase-based (not Clerk). Frontend uses supabase-js directly; backend verifies JWT via supabase.auth.getUser().
- Most data (mess_entries, user_settings) lives in Supabase Postgres, accessed via supabase-js (not Drizzle). Drizzle is used only for chatbot conversations/messages tables in the Replit DB.
- Admin access is determined by ADMIN_USER_IDS env var (comma-separated emails or user IDs).
- Groq (grok-2) powers MessBot — falls back gracefully if GROQ_API_KEY is absent.
- UI is in Marathi (Devanagari script) with English fallbacks.

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

- Running `pnpm dev` at workspace root won't work — use artifact-specific commands or restart workflows.
- After any OpenAPI spec change, always re-run codegen before using updated types.
- The Supabase DB (mess_entries, user_settings) is separate from the Replit DB (conversations, messages). Don't mix them.
- Admin check uses ADMIN_USER_IDS env var — add emails or Supabase user IDs there.

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
