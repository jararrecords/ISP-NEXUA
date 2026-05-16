# Ayyan Fiber Solutions (AFS) — ISP Management Platform

## Overview

Full-stack ISP management platform with admin/reseller panels, MikroTik integration, customer self-service portal, Telegram bot, AI chatbot, and billing system.

pnpm workspace monorepo using TypeScript. Each package manages its own dependencies.

## Stack

- **Monorepo tool**: pnpm workspaces
- **Node.js version**: 24
- **Package manager**: pnpm
- **TypeScript version**: 5.9
- **API framework**: Express 5
- **Database**: PostgreSQL + Drizzle ORM
- **Validation**: Zod, drizzle-zod
- **API codegen**: Orval (from OpenAPI spec)
- **Build**: esbuild
- **Frontend**: React + Vite + shadcn/ui + Tailwind
- **Auth**: JWT (admin: `afs_token`, customer: `afs_customer_token`)
- **AI**: OpenAI via Replit AI Integrations proxy
- **Telegram**: Long-polling Telegram Bot API (no extra package)

## Key Commands

- `pnpm run typecheck` — full typecheck across all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks/Zod schemas
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)

## Artifacts

| Artifact | Path | Description |
|----------|------|-------------|
| `afs-dashboard` | `/` | Admin + Reseller + Customer frontend |
| `api-server` | `/api` | Express REST API backend |

## MikroTik Integration

- **Router**: `mikrotik.ispledger.com:9001` (RouterOS REST API)
- **Credentials**: `MIKROTIK_USERNAME`, `MIKROTIK_PASSWORD` (env secrets)
- **Service**: `artifacts/api-server/src/services/mikrotik.ts`
- **Features**: Live sessions sync, user enable/disable, kick session, system resource info
- **Ping endpoint**: `POST /api/routers/:id/ping` — real MikroTik connectivity test
- **Sync endpoint**: `POST /api/sessions/sync` — pulls live PPPoE sessions from MikroTik

## Telegram Bot

- **Token**: `TELEGRAM_BOT_TOKEN` (env secret)
- **Service**: `artifacts/api-server/src/services/telegram.ts`
- **Polling**: Long-polling, starts on server boot
- **Multi-role**: Admin, Reseller, Customer (auth via `/login username password`)
- **Commands**:
  - All: `/login`, `/logout`, `/help`
  - Admin: `/stats`, `/users`, `/online`, `/expired`, `/resellers`, `/requests`, `/approve <id>`, `/reject <id>`
  - Reseller: `/stats`, `/users`, `/online`, `/expired`, `/balance`
  - Customer: `/status`, `/usage`, `/upgrade`
- **AI fallback**: Free-text → OpenAI GPT-4o-mini (English + Urdu/Roman Urdu)
- **Sessions stored in**: `telegram_sessions` DB table

## Database Schema (lib/db/src/schema/)

| Table | Description |
|-------|-------------|
| `admins` | Admin accounts |
| `resellers` | Reseller accounts with balance |
| `isp_users` | Customer PPPoE accounts |
| `packages` | Internet packages |
| `routers` | MikroTik router registry |
| `sessions` | Active PPPoE sessions (synced from MikroTik) |
| `transactions` | Billing transactions |
| `alerts` | System alerts |
| `package_requests` | Customer upgrade requests (approved/rejected by admin) |
| `telegram_sessions` | Telegram bot user sessions (role + auth state) |

## Frontend Routes (afs-dashboard)

| Route | Component | Auth |
|-------|-----------|------|
| `/login` | Admin/Reseller login | None |
| `/` | Dashboard | Admin/Reseller |
| `/users` | User management | Admin/Reseller |
| `/resellers` | Reseller management | Admin only |
| `/packages` | Package management | Admin/Reseller |
| `/routers` | Router management | Admin/Reseller |
| `/sessions` | Live sessions | Admin/Reseller |
| `/transactions` | Billing transactions | Admin/Reseller |
| `/alerts` | System alerts | Admin/Reseller |
| `/requests` | Package upgrade requests | Admin only |
| `/chat` | AI support chat | Admin/Reseller |
| `/customer` | Customer login | None |
| `/customer/dashboard` | Customer self-service | Customer JWT |

## API Routes

### Auth
- `POST /api/auth/login` — admin/reseller login
- `GET /api/auth/me` — get current user

### Customer Portal
- `POST /api/customer/login` — customer login (returns JWT)
- `GET /api/customer/me` — account info + active session
- `GET /api/customer/usage` — billing + usage stats
- `POST /api/customer/request-upgrade` — submit package upgrade request
- `GET /api/customer/requests` — customer's own requests

### Admin Package Requests
- `GET /api/admin/package-requests` — list all requests (filter by status)
- `POST /api/admin/package-requests/:id/review` — approve or reject

### Sessions
- `GET /api/sessions` — list active sessions
- `POST /api/sessions/sync` — sync from MikroTik (real data)
- `POST /api/sessions/:id/disconnect` — kick user from MikroTik + remove session

### Routers
- `POST /api/routers/:id/ping` — real MikroTik connectivity test
- `GET /api/routers/:id/info` — MikroTik system resource info

## Environment Secrets

| Secret | Purpose |
|--------|---------|
| `MIKROTIK_PASSWORD` | MikroTik router password |
| `MIKROTIK_USERNAME` | MikroTik username (env var, not secret) |
| `MIKROTIK_URL` | MikroTik base URL (env var, not secret) |
| `TELEGRAM_BOT_TOKEN` | Telegram Bot API token |
| `SESSION_SECRET` | JWT signing secret |
| `AI_INTEGRATIONS_OPENAI_BASE_URL` | Replit AI proxy base URL |
| `AI_INTEGRATIONS_OPENAI_API_KEY` | Replit AI proxy key |
