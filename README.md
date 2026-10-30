# PayPost

PayPost is a full-stack publishing and digital learning platform built for the Kenyan market. It combines moderated article publishing, content monetization, M-Pesa deposits and withdrawals, course delivery, video engagement rewards, referral earnings, investment-style balances, and role-based administration in one Next.js application.

The repository demonstrates practical backend engineering: domain modeling, transactional wallet operations, payment-provider integration, authentication and authorization, webhook handling, audit trails, relational data access, and operational administration.

## Why PayPost

Creators need systems that connect content, audience activity, trust, and payouts. PayPost brings those concerns into one account model:

- Writers publish and manage articles in a moderation workflow.
- Readers consume public articles and academy content.
- View and video activity can generate earnings.
- Users deposit funds, transfer earnings, invest balances, and request withdrawals.
- Administrators review members, content, transactions, and withdrawal requests.

## Engineering Highlights

- Full-stack TypeScript application built with Next.js App Router and React.
- PostgreSQL data model managed through Prisma.
- NextAuth credential authentication with JWT sessions and role checks.
- REST-style API routes organized by domain.
- M-Pesa and PayHero payment provider integrations.
- Wallet, earnings, investment, transaction, and withdrawal domain models.
- Article moderation, publishing controls, and configurable monetization values.
- Course, lesson, enrollment, progress, review, video, and user-level models.
- Admin dashboards for operational management and audit visibility.
- Rich-text editing, uploads, public feeds, referrals, and password reset flows.

## System Architecture

```mermaid
flowchart LR
    Browser[Web client] --> Next[Next.js App Router]
    Next --> Auth[NextAuth]
    Next --> API[Domain API routes]
    API --> Prisma[Prisma ORM]
    Prisma --> Postgres[(PostgreSQL)]
    API --> Mpesa[Safaricom M-Pesa APIs]
    API --> PayHero[PayHero API]
    Mpesa --> Callback[M-Pesa callback route]
    PayHero --> Webhook[PayHero webhook route]
    Callback --> Prisma
    Webhook --> Prisma
    API --> Blob[Object storage or local uploads]
```

## Core Domains

### Identity and Access

- Credential registration and bcrypt password hashing.
- Unique email and username constraints.
- JWT-backed NextAuth sessions.
- Writer, administrator, and super-administrator roles.
- Middleware-level route protection and role-based navigation.

### Publishing and Monetization

- Draft and published article states.
- Pending, approved, and rejected moderation states.
- Featured images, rich content, categories, and article views.
- Dynamic article click values and boost configuration.
- Per-view earnings records and wallet crediting.
- Recent-article and public-feed API surfaces.

### Wallet and Payments

- KES balances split between wallet balance and earnings.
- Deposit and course-payment transaction flows.
- M-Pesa STK Push initiation through Safaricom or PayHero APIs.
- Provider callback and webhook handling.
- Pending, completed, and failed transaction states.
- Earnings-to-wallet transfers.
- Manual withdrawal approval and processing workflows.

### Learning and Engagement

- Categories, courses, sections, lessons, and enrollments.
- Article, video, PDF, and quiz lesson types.
- Enrollment progress and course reviews.
- Video reward rules, daily watch limits, and user levels.
- Academy and user dashboard views.

### Administration

- Member management and role visibility.
- Article moderation and boost controls.
- Course and category management.
- Transaction and withdrawal review.
- Platform settings and operational dashboards.
- Audit log model for user actions.

## Technology Stack

| Area | Technology |
|---|---|
| Frontend | React 18, Next.js 14 App Router, Tailwind CSS |
| Language | TypeScript |
| API | Next.js Route Handlers |
| Authentication | NextAuth, JWT, bcrypt |
| Database | PostgreSQL |
| ORM | Prisma 6 |
| Validation | Zod |
| Rich text | Tiptap / React Simple WYSIWYG |
| Payments | Safaricom M-Pesa, PayHero |
| File storage | Vercel Blob-compatible or local upload paths |
| UI | Radix Dialog, Lucide, Framer Motion, Tailwind |

## Repository Layout

```text
app/                    Next.js pages and API route handlers
components/             Reusable UI and workflow components
lib/                    Auth, database, wallet, payment, and utility services
prisma/schema.prisma    Relational domain model
prisma/migrations/      Versioned PostgreSQL migrations
scripts/                Seed and migration utilities
tests/                  Repository security audit helpers
public/                 Static assets and download resources
docs/                   Feature and implementation notes
```

## Local Setup

### Prerequisites

- Node.js 20 or later
- npm
- PostgreSQL
- M-Pesa and/or PayHero sandbox credentials for payment testing

### 1. Install dependencies

```bash
npm install
```

### 2. Configure environment variables

Create `.env.local`:

```bash
NEXTAUTH_URL="http://localhost:3000"
NEXTAUTH_SECRET="generate-a-long-random-secret"
NEXT_PUBLIC_APP_URL="http://localhost:3000"
DATABASE_URL="postgresql://USER:PASSWORD@HOST:5432/DATABASE"

MPESA_CONSUMER_KEY=""
MPESA_CONSUMER_SECRET=""
MPESA_PASS_KEY=""
MPESA_SHORT_CODE=""

PAYHERO_CHANNEL_ID=""
PAYHERO_TILL_NUMBER=""
PAYHERO_API_KEY=""
PAYHERO_WEBHOOK_SECRET=""

BLOB_READ_WRITE_TOKEN=""
VERCEL_BLOB_READ_WRITE_TOKEN=""
ENABLE_DEBUG_LOGS="false"
```

Do not commit credentials. Use separate sandbox credentials locally and production secrets in the deployment environment.

### 3. Prepare the database

```bash
npx prisma generate
npx prisma migrate deploy
npx prisma db seed
```

For schema prototyping, `npm run db:push` is also available. Use migrations for shared and production environments.

### 4. Start the application

```bash
npm run dev
```

Open `http://localhost:3000`.

## Common Commands

| Command | Purpose |
|---|---|
| `npm run dev` | Start the development server |
| `npm run build` | Create a production build |
| `npm start` | Serve the production build |
| `npm run db:push` | Push the Prisma schema to the configured database |
| `npm run db:reset` | Reset the database and reapply migrations |
| `npm run db:studio` | Open Prisma Studio |
| `npx prisma migrate dev` | Create and apply a development migration |

## Payment Integration Notes

- PayHero deposits create a pending transaction before provider confirmation.
- Provider callbacks reconcile provider outcomes with internal transaction state.
- Callback processing should correlate provider identifiers with internal transaction IDs.
- Withdrawal funds require controlled approval and processing states.
- Payment secrets, callback secrets, and signing credentials must remain server-only.

The repository includes diagnostics at `/api/check-env`, `/api/test-mpesa`, and `/api/admin/test-payhero` for development-time provider configuration checks. These routes should be protected or removed outside controlled environments.

## API Overview

```text
/api/auth/*             Registration, login support, password reset
/api/articles/*         Article creation and view workflows
/api/admin/*            Moderation, members, settings, transactions, courses
/api/courses/*          Public and enrolled course data
/api/dashboard/*        User dashboard aggregation
/api/views              Article view and earnings tracking
/api/videos/*           Video catalog and watch rewards
/api/wallet/*           Balances, deposits, transfers, investments, withdrawals
/api/webhooks/*         Provider callbacks and reconciliation events
/api/user/*             Profiles, referrals, and levels
```

Route names are implementation boundaries, not a public API contract. Productionization should include versioned schemas, typed error responses, rate limits, and consistent authorization middleware.

## Data Model Summary

- `User` owns a profile, wallet, articles, earnings, transactions, enrollments, withdrawals, audit logs, referrals, level, and video watches.
- `Article` belongs to an author and tracks views and earnings.
- `Wallet` holds balance, earnings, investment capital, and currency.
- `Transaction` records wallet movement with provider references and metadata.
- `Withdrawal` tracks manual or automated payout state and approval details.
- `Course` contains sections and lessons and relates to enrollments and reviews.
- `Investment` tracks principal, period, interest, maturity, and status.
- `VideoWatch` records engagement rewards and level context.

See [`prisma/schema.prisma`](prisma/schema.prisma) for the authoritative data contract.

## Security Considerations

The current implementation includes password hashing, JWT sessions, role checks, and server-side provider credentials. A production-ready version should additionally demonstrate:

- Defensive validation at every write boundary.
- Authorization checks inside API handlers, not only middleware.
- Signed and duplicate-safe provider webhooks.
- Idempotent financial state transitions.
- Rate limiting on authentication, payments, and content endpoints.
- Audit coverage for privileged actions.
- Secret scanning and environment separation.
- Removal or strict protection of diagnostics and debug endpoints.

## Documentation

- [`PROBLEM.md`](PROBLEM.md) - problem analysis and implementation approach
- [`SUGGESTION.md`](SUGGESTION.md) - engineering tasks for a production-grade repository
- [`docs/FEATURE_IMPLEMENTATION.md`](docs/FEATURE_IMPLEMENTATION.md) - recent notification and download feature notes

## License

Add an explicit license before public distribution if the repository is intended for reuse or external contribution.
