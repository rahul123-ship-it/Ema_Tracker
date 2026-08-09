# Delta Trend Scanner

A full-stack cryptocurrency market-analysis application built from scratch around **Delta Exchange market data** and a multi-timeframe trend-scanning workflow.

The primary goal is not to blindly generate trades. The system first identifies **which markets are actually trending**, determines the trend direction and strength, detects possible trend exhaustion/reversal, and only then evaluates the user's **9/15 EMA crossover** as an entry trigger.

## Project Goals

1. Discover the strongest trending Delta Exchange markets.
2. Rank markets using objective trend metrics instead of manual chart hunting.
3. Support 15m trend context, 5m setup detection, and optional 3m entry confirmation.
4. Detect trend continuation and potential reversals using price structure and indicators.
5. Keep market-data ingestion, indicator calculations, strategy logic, persistence, API, and UI separated.
6. Build the application end-to-end by hand so every technology in the stack is understood.
7. Keep the architecture compatible with Cloudflare Workers and the chosen database stack.

> **Important:** This project is an analytical/trading-tool project, not financial advice. A scanner score is not a guarantee of profitable trades.

## Technology Stack

### Frontend

- React
- TypeScript
- TanStack Start
- TanStack Router
- TanStack Query
- TanStack Table
- Jotai
- Tailwind CSS v4
- shadcn/ui
- Charting library

### Backend / Platform

- Hono
- Cloudflare Workers
- Cloudflare KV
- Cloudflare R2
- Cloudflare Hyperdrive
- Vite
- Alchemy IaC

### Data / Database

- PostgreSQL (Supabase as the hosted database option)
- Prisma — migrations/schema/table management only
- Drizzle — application ORM where appropriate
- Kysely — typed SQL/querying

### Authentication

- Better Auth

## Architecture

```text
Delta Exchange
      |
      | REST / WebSocket market data
      v
+---------------------------+
| Market Data Layer         |
| instruments / ticker      |
| candles / realtime stream |
+-------------+-------------+
              |
              v
+---------------------------+
| Indicator Engine          |
| EMA / ADX / ATR / RVOL    |
| RSI / structure metrics   |
+-------------+-------------+
              |
              v
+---------------------------+
| Trend Engine              |
| direction / strength      |
| continuation / exhaustion |
| reversal candidates       |
+-------------+-------------+
              |
              v
+---------------------------+
| Strategy Engine           |
| 15m context               |
| 5m setup                  |
| 3m confirmation           |
| 9/15 EMA trigger          |
+-------------+-------------+
              |
              +-------------------+
              |                   |
              v                   v
        PostgreSQL             Hono API
              |                   |
              +---------+---------+
                        |
                        v
              TanStack Query
                        |
                        v
              React Dashboard
```

## Core Trading Logic

The project intentionally separates **trend selection** from **entry timing**.

### Trend selection

The scanner should answer:

> Which Delta Exchange markets are trending right now?

Candidate inputs include:

- ADX / trend strength
- price relative to EMA 200
- EMA slope and alignment
- relative volume (RVOL)
- ATR / volatility expansion
- recent percentage movement
- higher-high / higher-low or lower-high / lower-low structure
- multi-timeframe alignment

### Entry workflow

```text
15m: identify trend
        |
        v
5m: identify setup
        |
        v
3m: optional entry confirmation
        |
        v
9 EMA / 15 EMA crossover
```

### Reversal workflow

The scanner should not assume that one red/green candle means a reversal.

```text
Strong trend
   -> momentum weakens
   -> structure breaks
   -> EMA relationship changes
   -> volume/volatility confirmation
   -> reversal candidate
```

## Planned Folder Structure

```text
Ema_Tracker/
├── README.md
├── package.json
├── tsconfig.json
├── vite.config.ts
├── wrangler.jsonc
├── alchemy.run.ts
├── .env.example
├── .gitignore
│
├── app/                              # TanStack Start application entry
│   ├── client.tsx                    # Client-side application bootstrap
│   ├── router.tsx                    # TanStack Router configuration
│   └── routes/
│       ├── __root.tsx                # Root application layout/providers
│       ├── index.tsx                 # Main trend-scanner page
│       ├── scanner.tsx               # Full scanner table and filters
│       ├── markets.tsx               # Market/instrument browser
│       └── markets.$symbol.tsx       # Individual market analysis page
│
├── components/                       # Reusable React UI components
│   ├── scanner/
│   │   ├── TrendTable.tsx             # Ranked trending-market table
│   │   ├── TrendScoreBadge.tsx        # Visual trend-score indicator
│   │   ├── TrendFilters.tsx           # Scanner timeframe/filter controls
│   │   └── MarketStatus.tsx            # Current market state display
│   ├── charts/
│   │   ├── PriceChart.tsx             # Price + EMA chart
│   │   ├── VolumeChart.tsx            # Volume/RVOL visualization
│   │   └── IndicatorChart.tsx         # Indicator visualization
│   └── ui/                             # shadcn/ui components
│
├── features/                          # Business-oriented frontend modules
│   ├── scanner/
│   │   ├── api.ts                     # Scanner API calls
│   │   ├── queries.ts                 # TanStack Query definitions
│   │   ├── atoms.ts                   # Jotai scanner UI state
│   │   └── types.ts                   # Scanner frontend types
│   └── market/
│       ├── api.ts                     # Market detail API calls
│       ├── queries.ts                 # Market query definitions
│       └── types.ts                   # Market frontend types
│
├── server/                            # Hono/backend application
│   ├── index.ts                       # Worker/Hono application entry
│   ├── env.ts                         # Typed environment bindings/config
│   ├── middleware/
│   │   ├── auth.ts                    # Better Auth/session middleware
│   │   ├── errors.ts                  # API error handling
│   │   └── logging.ts                 # Request logging
│   ├── routes/
│   │   ├── health.ts                  # Health/readiness endpoints
│   │   ├── markets.ts                 # Market/instrument endpoints
│   │   ├── scanner.ts                 # Trend scanner endpoints
│   │   └── auth.ts                    # Authentication-related endpoints
│   └── services/
│       ├── delta/
│       │   ├── client.ts              # Delta Exchange HTTP client
│       │   ├── instruments.ts         # Instrument metadata retrieval
│       │   ├── ticker.ts              # Ticker retrieval
│       │   ├── candles.ts             # Historical candle retrieval
│       │   └── websocket.ts           # Realtime market-data connection
│       ├── cache/
│       │   ├── kv.ts                  # Cloudflare KV access
│       │   └── market-cache.ts        # Market-data caching rules
│       └── auth/
│           └── better-auth.ts         # Better Auth server configuration
│
├── core/                              # Exchange-independent trading logic
│   ├── indicators/
│   │   ├── ema.ts                     # Exponential moving average
│   │   ├── adx.ts                     # Average Directional Index
│   │   ├── atr.ts                     # Average True Range
│   │   ├── rsi.ts                     # Relative Strength Index
│   │   └── rvol.ts                    # Relative volume calculation
│   ├── structure/
│   │   ├── swings.ts                  # Swing high/low detection
│   │   ├── trend.ts                   # Higher-high/lower-low analysis
│   │   └── reversal.ts                # Reversal/exhaustion detection
│   ├── trend/
│   │   ├── trend-engine.ts            # Combines metrics into trend state
│   │   ├── trend-score.ts             # Calculates normalized trend score
│   │   └── timeframe-alignment.ts     # Multi-timeframe trend alignment
│   └── strategy/
│       ├── ema-crossover.ts           # 9/15 EMA crossover detection
│       ├── setup.ts                   # Trade-setup qualification
│       └── rules.ts                   # Strategy rule definitions
│
├── db/                                # Database schema and query layer
│   ├── schema/                        # Database domain schemas/types
│   ├── migrations/                    # Prisma-generated migrations
│   ├── prisma/                        # Prisma schema/config if needed
│   ├── drizzle/                       # Drizzle schema/config if needed
│   └── queries/                       # Kysely query implementations
│       ├── markets.ts                 # Market persistence queries
│       ├── candles.ts                 # Candle persistence queries
│       └── trend-snapshots.ts         # Trend snapshot queries
│
├── lib/                               # Shared application utilities
│   ├── logger.ts                      # Structured logging utility
│   ├── errors.ts                      # Shared application errors
│   ├── time.ts                        # Timeframe/date helpers
│   └── validation.ts                  # Shared validation helpers
│
├── types/                             # Shared TypeScript contracts
│   ├── market.ts                      # Market/instrument types
│   ├── candle.ts                      # OHLCV candle types
│   ├── indicators.ts                  # Indicator result types
│   ├── trend.ts                       # Trend-analysis types
│   └── api.ts                          # API request/response contracts
│
├── tests/                             # Automated tests
│   ├── unit/
│   │   ├── indicators/                # Indicator correctness tests
│   │   ├── structure/                 # Market-structure tests
│   │   └── strategy/                  # Strategy-rule tests
│   ├── integration/                   # API/database integration tests
│   └── fixtures/                      # Deterministic candle/market data
│
├── scripts/                           # Developer/maintenance scripts
│   ├── seed.ts                        # Development database seed
│   └── backfill.ts                    # Historical market-data backfill
│
├── infra/                             # Cloudflare/Alchemy infrastructure
│   ├── worker.ts                      # Worker infrastructure definition
│   ├── kv.ts                           # KV resource definition
│   ├── r2.ts                           # R2 bucket definition
│   └── hyperdrive.ts                   # Hyperdrive configuration
│
└── docs/                              # Project learning/design documentation
    ├── architecture.md                # Detailed system architecture
    ├── data-model.md                  # Database model documentation
    ├── trend-engine.md                # Trend scoring methodology
    ├── strategy.md                    # Trading strategy rules
    └── development-plan.md            # Step-by-step implementation plan
```

## Development Philosophy

Build this project incrementally. Do not start by connecting every service at once.

```text
1. React + TypeScript + Vite
2. TanStack Router
3. Tailwind + shadcn
4. TanStack Table
5. TanStack Query
6. Hono
7. Delta REST API
8. Delta WebSocket
9. Indicator engine
10. Trend engine
11. Database
12. Authentication
13. Realtime scanner
14. Charts
15. Cloudflare deployment
16. Alchemy infrastructure
```

Every layer should be understood and tested before the next layer is introduced.

## Initial Non-Goals

- Automatic order execution
- Leveraged trading automation
- Guaranteed-profit predictions
- Black-box ML before the deterministic strategy is validated
- Mixing exchange-specific code into indicator/strategy calculations

## Git Workflow

Development happens on feature branches. The initial architecture is being established separately from the existing EMA Tracker implementation.

```bash
git checkout -b crypto-trend-scanner-foundation
```

Future work should use small commits such as:

```text
feat: add delta market client
feat: add ema indicator
feat: add trend scoring engine
feat: add scanner endpoint
feat: add scanner table
```

## Current Status

**Phase 0 — Architecture / repository foundation**

The repository structure and documentation are being established first. Implementation will follow incrementally, with tests added alongside each core module.
