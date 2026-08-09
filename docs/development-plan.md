# Development Plan

## Phase 0 — Foundation
- Establish the project structure.
- Document responsibilities of each layer.
- Keep the existing EMA Tracker implementation untouched on `main`.

## Phase 1 — Frontend foundation
- Set up React + TypeScript + Vite/TanStack Start.
- Add TanStack Router.
- Add Tailwind CSS v4 and shadcn/ui.
- Build the initial dashboard shell.

## Phase 2 — Data-table foundation
- Add TanStack Table.
- Create mock market data.
- Add sorting, filtering, timeframe selection, and trend-score columns.

## Phase 3 — Backend foundation
- Add Hono.
- Add health endpoint.
- Add typed environment configuration.
- Connect frontend to backend with TanStack Query.

## Phase 4 — Delta Exchange integration
- Implement instrument discovery.
- Implement ticker retrieval.
- Implement historical candles.
- Implement realtime WebSocket ingestion.
- Normalize Delta-specific payloads into exchange-independent domain types.

## Phase 5 — Indicator engine
Implement and test each indicator independently:

1. EMA
2. ATR
3. ADX
4. RSI
5. RVOL

No strategy code should calculate indicators directly. Indicators belong in the core indicator layer.

## Phase 6 — Trend engine
Build a deterministic trend classifier using:

- EMA alignment
- EMA slope
- ADX
- RVOL
- ATR expansion
- market structure
- multi-timeframe alignment

Output a normalized trend state and score.

## Phase 7 — Reversal engine
Detect:

- trend exhaustion
- swing failure
- structure break
- momentum deterioration
- EMA reversal
- volume confirmation

The output should be a **reversal candidate**, not an automatic trade instruction.

## Phase 8 — Strategy engine
Apply the user's workflow:

- 15m = primary trend context
- 5m = setup
- 3m = optional confirmation
- 9/15 EMA = entry trigger

## Phase 9 — Database
Persist:

- instruments
- candles where required
- trend snapshots
- scanner results
- user configuration

Use Prisma for migrations/table management, Kysely for typed queries, and Drizzle only where it provides a clearly justified application-layer benefit. Avoid unnecessary duplication between database tools.

## Phase 10 — Authentication
Add Better Auth and protect user-specific configuration and dashboard functionality.

## Phase 11 — Charts
Add:

- price candles
- EMA 9
- EMA 15
- EMA 200
- volume
- trend/reversal annotations

## Phase 12 — Cloudflare deployment
Introduce progressively:

- Workers
- KV
- R2
- Hyperdrive
- Alchemy IaC

Do not introduce Cloudflare services until their local responsibility is understood.

## Phase 13 — Validation
Before considering the scanner useful:

- unit-test indicator calculations
- test trend classification against known candle sequences
- test reversal detection against fixtures
- test API contracts
- replay historical candles
- measure false signals
- compare scanner rankings with manually reviewed charts

## Phase 14 — Production hardening
- rate-limit external API access
- handle WebSocket reconnects
- add structured logs
- add health/readiness checks
- handle stale market data
- protect secrets
- add CI
- add deployment checks
