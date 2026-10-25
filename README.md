# Toronto x New Jersey: The Property Empire

A property-trading board game set across Toronto/the GTA and New Jersey, played in Empire Credits (EC).

> **Status: engine milestone (Phases 1-2 plus the first slice of Phase 4).**
> The rules engine, economy, AI bot, save/load and a test suite exist and pass.
> **There is no user interface, no server and no online multiplayer yet, so the game cannot be played in a browser yet.**
> This README describes only what is really in the repository.

## What exists

| Area | State |
|---|---|
| 40-space board, 8 property groups + transport + utilities | Done (`shared/constants/board.ts`) |
| Central economy config (prices, rent ladder, mortgages, taxes, fares) | Done (`shared/constants/economy.ts`) |
| 48 original event cards, each with an implemented effect | Done (`shared/constants/cards.ts`) |
| Engine: dice, movement, Start bonus, purchase, rent, development, mortgages, auctions, trades, doubles, Holding Cell, taxes, inspection, cross-border exchange, cards | Done (`shared/engine/engine.ts`) |
| Debt resolution, bankruptcy (to player or bank), resign, last-solvent and round-limit victory | Done |
| Ledger for every money movement, with an audit function | Done |
| Save/load with schema version, validation and tamper detection | Done (`shared/engine/persistence.ts`), in-memory string only |
| Rule-abiding AI (4 difficulty levels) | Basic (`shared/ai/bot.ts`) |
| Seeded, deterministic RNG held in game state | Done |

## What does NOT exist yet

- Any browser UI (menus, board, dialogs, animations, sound), responsive layout, accessibility work
- IndexedDB storage (the save format exists; nothing writes it to disk or the browser)
- Server, Socket.IO, rooms, reconnects, accounts, database, rate limiting
- AI personalities; AI-initiated trades (the bot only responds to trade offers)
- Statistics screen and achievements (per-player counters are tracked, nothing awards achievements)
- Card balancing weights, Cross-Border Exchange variety beyond three outcomes
- ESLint (not configured), end-to-end/UI tests, deployment guide (nothing to deploy yet)

## Known issues and design decisions

- **Balance is unverified.** In 12 simulated 4-player bot games, only 3 ended by bankruptcy; 9 hit the 150-round cap. Unlimited bot-vs-bot games can stalemate. Rents, start bonus and bot spending need tuning. Use `maxRounds` for guaranteed-length matches.
- **Performance:** every action deep-clones the state (this is what makes failed actions harmless). A full bot game takes about 0.4-7 seconds. Fine for human play; should be revisited before large-scale simulation.
- Trades and development management are only allowed to the **current player**, before or after rolling.
- Transferred mortgaged properties stay mortgaged and pay no extra fee on transfer. Interest applies when repaid.
- Dice are rolled by the engine's seeded RNG. The `dice` option on `applyAction` is for tests and server code only; client payloads cannot set it. `rngState` is a secret and must be stripped before state is sent to any online client.
- Groups differ slightly from the spec's suggestion: "Urban Core" is used instead of a separate group list, and transport/utility are their own groups.
- The engine lives in `shared/` (not `server/`) so the offline browser game and the future server run identical code.

## Setup

Requires Node.js 20+ (developed on 22) and npm.

```bash
npm install
npm run typecheck      # TypeScript, strict
npm test               # 127 tests, ~25 seconds
npm run simulate -- 12 4 150   # games, players, round limit: bot-only balance statistics
```

## Verified results (last run)

- `npm run typecheck`: 0 errors
- `npm test`: 7 files, 127 tests passed
- Mutation spot-checks: deliberately breaking the start bonus, mortgage rent rule and even-development rule each made tests fail.

## Using the engine

```ts
import { createGame, applyAction } from './shared';

let game = createGame({
  players: [
    { name: 'Ana', token: 'streetcar', color: '#2196f3' },
    { name: 'Raj', token: 'pizza', color: '#e53935' },
  ],
  rules: { startingBalance: 1500, maxRounds: 60 },
});

const result = applyAction(game, { type: 'roll', playerId: 'p1' });
if (result.ok) game = result.state; else console.log(result.error); // plain-language reason
```

`applyAction` never mutates its input and either returns a new valid state or an error. Every action carries the acting `playerId`; the future server must derive it from the authenticated socket, never from client data.

## Project layout

```
shared/constants   board, economy, cards (single source of truth)
shared/engine      engine, persistence/validation, rng
shared/schemas     zod action schemas (the only inputs the engine accepts)
shared/ai          bot + full-game runner
tests/unit         127 tests
scripts/simulate   balance simulation
```

## Roadmap (next phases)

1. Phase 3: React + Vite UI for the offline game (board, dialogs, setup, save/load to IndexedDB).
2. Balance pass using `npm run simulate`.
3. Phase 4: AI personalities and AI trade proposals.
4. Phase 5-6: Express + Socket.IO authoritative server, rooms, reconnects, PostgreSQL persistence.
5. Phase 7-8: polish, accessibility, deployment guide, end-to-end tests.
