# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Dewy Resort Hotel is a sample application demonstrating **enterprise MCP (Model Context Protocol) architecture patterns** for integrating LLMs with backend systems. It showcases a two-tier tool design with orchestrators (high-level workflows) and atomic skills (composable building blocks).

**Tech Stack:** Next.js 14, React 18, TypeScript, Tailwind CSS, SQLite (local), with Workato as the integration hub connecting to Salesforce, Stripe, and Twilio backends.

## Common Commands

```bash
# Development
pnpm install          # Install dependencies
pnpm dev              # Start dev server (runs env check first)
pnpm build            # Production build
pnpm lint             # ESLint

# Database (local SQLite for app-specific data)
pnpm db:init          # Initialize database
pnpm db:seed          # Seed with sample data
pnpm db:migrate       # Run migrations

# Testing
pnpm test             # Run Jest tests (pattern: **/__tests__/**/*.test.ts)
pnpm test:salesforce  # Automated Salesforce endpoint tests

# Mock mode verification
pnpm verify:mock      # Check mock mode configuration

# CLI tools (Workato and Salesforce)
make setup                    # Install all CLIs
make setup tool=workato       # Install Workato CLI only
make setup tool=salesforce    # Install Salesforce CLI only
make workato-init             # Deploy Workato recipes
make start-recipes            # Start all Workato recipes
make sf-deploy org=<alias>    # Deploy Salesforce metadata
```

## Architecture

### Data Flow
All backend integrations flow through Workato as the central hub:
- **Hotel App** → **Workato** → **Salesforce/Stripe/Twilio**
- **LLM Agent** → **MCP Server** → **Workato** → **Backends**

No direct system integrations - all data movement is orchestrated through Workato recipes.

### Key Directories

- `app/` - Next.js App Router pages and API routes
  - `app/guest/` - Guest portal (dashboard, billing, room controls, services, chat)
  - `app/manager/` - Manager portal (dashboard, rooms, maintenance, billing, chat)
  - `app/api/` - API routes calling Workato endpoints
- `components/` - React components organized by role
  - `components/guest/` - Guest-specific components
  - `components/manager/` - Manager-specific components
  - `components/shared/` - Shared components (chat interfaces, error boundaries)
  - `components/ui/` - Base UI components (shadcn/ui style)
- `lib/` - Core libraries
  - `lib/workato/` - Workato client with mock mode support
  - `lib/auth/` - Auth providers (mock, Cognito, Okta)
  - `lib/bedrock/` - AWS Bedrock chat integration
  - `lib/db/` - SQLite database client
- `workato/` - MCP server implementation (Workato recipes)
  - `orchestrator-recipes/` - High-level workflow recipes (12 orchestrators)
  - `atomic-salesforce-recipes/` - Salesforce atomic skills (15 recipes)
  - `atomic-stripe-recipes/` - Payment atomic skills (6 recipes)
- `salesforce/` - Salesforce metadata and seed data

### Workato Client Architecture

The `lib/workato/client.ts` implements:
- Authenticated requests to Workato API Collection endpoints
- **Mock mode** - When `WORKATO_MOCK_MODE=true`, returns simulated responses
- Response caching for GET requests
- Retry logic with exponential backoff
- Correlation ID tracking for debugging

### Authentication Providers

Configured via `AUTH_PROVIDER` env var:
- `mock` - Local email/password auth (development)
- `cognito` - Amazon Cognito OAuth 2.0
- `okta` - Okta OAuth 2.0

### MCP Tool Design Pattern

**Orchestrators** - Optimized for common scenarios:
- Execute multi-step workflows in < 3 seconds
- Encode business rules and prerequisites
- Handle state transitions in correct dependency order

**Atomic Skills** - Building blocks for flexibility:
- Single-responsibility operations
- Composable by AI agents at runtime
- Enable handling of unexpected scenarios

## Quick Start (Mock Mode)

```bash
cp .env.example .env.local    # Then set AUTH_PROVIDER=mock, WORKATO_MOCK_MODE=true
pnpm install                  # Native modules (better-sqlite3, bcrypt) build automatically
pnpm db:init && pnpm db:seed  # Initialize and seed SQLite database
pnpm dev                      # Start at http://localhost:3000
```

**Test credentials (from seed):**
- Manager: `manager1@hotel.com` / `password123`
- Guest: `guest1@hotel.com` / `password123` (Room 100)

## Development Notes

- Path alias `@/*` maps to project root
- Mock mode (`WORKATO_MOCK_MODE=true`) simulates API responses for frontend development
- Restart dev server after changing env vars like `WORKATO_MOCK_MODE`
- Jest tests are in `**/__tests__/**/*.test.ts` files within `lib/`
- Native module builds are pre-approved in `package.json` under `pnpm.onlyBuiltDependencies`
