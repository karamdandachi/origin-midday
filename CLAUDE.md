# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Context

Origin is an AI-native supply chain platform that unifies global trade operations - connecting orders, shipments, documentation, and financial workflows in one intelligent interface. Built for importers and exporters, Origin automates the fragmented world of global trade, delivering total visibility, accuracy, and control across every shipment and trade.

This codebase is forked from Midday and will be progressively modified to support supply chain and global trade operations.

## Development Commands

### Package Manager
This project uses **Bun** as the package manager. Do not use npm or yarn.

### Essential Commands
```bash
# Install dependencies
bun install

# Development - run specific apps
bun run dev:dashboard    # Dashboard on http://localhost:3001
bun run dev:api          # API on http://localhost:3003
bun run dev:engine       # Engine on http://localhost:3002
bun run dev:website      # Marketing website
bun run dev              # Run all apps in parallel

# Build
bun run build            # Build all apps
bun run build:dashboard  # Build dashboard only

# Testing
bun test                 # Run all tests
bun test --watch        # Run tests in watch mode
cd apps/dashboard && bun test  # Run tests for specific app

# Code Quality
bun run lint            # Check code with Biome + manypkg
bun run format          # Format code with Biome
bun run typecheck       # TypeScript type checking
```

## Architecture

### Monorepo Structure
- **Turbo** monorepo with Bun workspaces
- `/apps/*` - Applications (dashboard, api, engine, website, desktop, docs)
- `/packages/*` - Shared packages and libraries
- `/packages/email/*` - Email templates

### Core Applications

#### Dashboard (`/apps/dashboard`)
- **Stack**: Next.js 15.5.2 with App Router, React 19, TypeScript
- **Styling**: TailwindCSS, Shadcn UI components
- **State**: Zustand, React Query (TanStack Query)
- **Auth**: Supabase Auth with MFA support
- **Routing**: Internationalized with `[locale]` parameter
- **Key patterns**:
  - Server Components by default
  - tRPC for type-safe API calls
  - Server Actions via next-safe-action
  - Parallel data fetching with `batchPrefetch`

#### API (`/apps/api`)
- **Stack**: Hono framework running on Bun runtime
- **Port**: 3003
- **Features**: tRPC endpoints, OpenAPI documentation, rate limiting
- **Key patterns**:
  - tRPC routers in `/src/trpc/routers`
  - Zod validation for all endpoints
  - Supabase integration for auth/database

#### Engine (`/apps/engine`)
- **Port**: 3002
- Document processing and AI operations
- Handles OCR, document intelligence, embeddings

### Key Packages

#### Database & Auth (`/packages/supabase`)
- Supabase client configuration
- Database queries and mutations
- Row-level security (RLS) policies
- Real-time subscriptions

#### UI Components (`/packages/ui`)
- Shared React components
- Built on Radix UI primitives
- Consistent design system

#### Background Jobs (`/packages/jobs`)
- Trigger.dev integration
- Async task processing
- Email sending, data sync

### Data Flow Architecture

1. **Client → tRPC → API**: Type-safe API calls from dashboard to API server
2. **API → Supabase**: Database operations via Supabase client
3. **API → Engine**: Document processing and AI operations
4. **Background Jobs**: Async operations via Trigger.dev
5. **Real-time**: Supabase real-time subscriptions for live updates

### Authentication Flow
- Supabase Auth handles user authentication
- Session management via cookies
- MFA support (TOTP)
- Team-based access control
- Row-level security in Supabase

### Environment Configuration

Required environment variables are organized by service:
- **Supabase**: Database and auth credentials
- **API URLs**: Service endpoints for local development
- **Third-party services**: Optional integrations (Resend, OpenAI, etc.)

Copy `.env` to each app directory that needs it (`/apps/dashboard/.env`, `/apps/api/.env`).

### Database Schema Patterns

The application uses Supabase (PostgreSQL) with:
- Teams-based multi-tenancy (`team_id` foreign keys)
- Soft deletes pattern (`deleted_at` timestamps)
- Audit fields (`created_at`, `updated_at`, `created_by`)
- JSON columns for flexible data storage
- Full-text search indexes

### Testing Strategy

- Unit tests with Bun's built-in test runner
- Component testing for React components
- API endpoint testing with tRPC
- Test files colocated with source files (`*.test.ts`, `*.test.tsx`)

### Deployment Targets

- **Dashboard & Website**: Vercel (Next.js)
- **API**: Fly.io (Bun/Hono)
- **Database**: Supabase (managed PostgreSQL)
- **Background Jobs**: Trigger.dev cloud

### Important Patterns

1. **Internationalization**: All user-facing text uses `next-international` with locale files in `/apps/dashboard/src/locales`

2. **Error Handling**: Consistent error boundaries and Sentry integration for monitoring

3. **Data Fetching**: Prefer server-side fetching with RSC, use React Query for client-side

4. **Type Safety**: End-to-end type safety with tRPC, Zod validation at boundaries

5. **Performance**:
   - Parallel data loading with `Promise.all`
   - Optimistic UI updates
   - Turbopack for fast development builds

6. **Security**:
   - All API endpoints require authentication
   - Row-level security in Supabase
   - Input validation with Zod
   - CORS configuration for API access