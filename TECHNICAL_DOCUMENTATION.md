# Technical Documentation — Pulse

## Core idea

An autonomous agent that watches a single active World Cup fixture's odds and score data, detects three named signal types, and reacts by managing one simulated paper position across a 10,000-unit bankroll — running unattended by default, with a manual override for deterministic demonstration.

## Signal detection (`detectSignals(fixtureState)`)

Pure, deterministic function operating on fixture state and its odds/score history:

- **MOMENTUM** — any outcome's implied win probability shifts ≥8 percentage points within a rolling 5-minute window (compared against the nearest history entry at least 5 minutes old).
- **GOAL REACTION** — fires immediately whenever a score change occurs, independent of odds movement.
- **MARKET LAG** — a goal occurred but the scoring team's implied probability moved less than 3 percentage points within 2 minutes afterward — read as the market not having caught up yet.

## Strategy (`applyStrategy(signal, book, fixtureState)`)

- No open position → opens one at 5% of current bankroll in the signal's favored direction, recording the entry implied probability.
- Signal agrees with the existing position's direction → adds another 5% of bankroll to the stake (capped at 40% of bankroll total), recalculating a stake-weighted average entry probability.
- Signal disagrees with the existing position's direction → logged as noted but not acted on. The agent does not flip positions mid-match; this is a deliberate discipline choice, not a missing feature.

## Settlement (`settle(fixtureId)`)

On full time, compares the held direction to the actual result. Payout uses a fair-odds conversion from the entry implied probability (`1 / entryProb` as the decimal-odds-equivalent multiplier) — internally consistent with the odds data already being modeled, rather than an arbitrary flat payout. Win: `stake * (payoutMultiplier - 1)` added to bankroll. Loss: stake subtracted. Then calls `Provider.getStatValidation()` for the final score and renders a compact Settlement Receipt (see below) inline in the feed.

## Manual mode and pipeline parity

The three "Inject" buttons (Odds Swing / Goal / Market Lag) mutate real fixture state — odds history, score, goal-event records — in a way engineered to deterministically cross each signal's real threshold, then call the exact same `detectSignals()` and `applyStrategy()` functions the autonomous timer uses. This is not a separate, simplified demo path; it's the same logic with a different trigger source, which is the architectural claim this project rests on. See inline code comments at each injector function for the specific mechanism (e.g. `injectMarketLag()` backdates a goal event and advances the fixture clock by exactly the 2-minute lag window the detector checks for).

## Settlement Receipt (secondary feature, reused verification code)

Reuses the SHA-256 Merkle build/verify logic validated in an earlier project this hackathon: Stage 1 and Stage 2 are genuinely recomputed and verified client-side against the response from `getStatValidation()`; Stage 3 is shown as "anchored" rather than independently re-verified, since its target root lives in a separate batch-proof endpoint not returned by this call. Rendered compactly inline in the activity feed, not as a full-screen takeover — it supports the agent's credibility story without competing with the primary autonomous-reaction narrative for attention.

## Dual-mode data layer

`SimProvider` and `LiveProvider` share the same `getStatValidation()` signature and return identically-shaped objects. `CONFIG.USE_LIVE_DATA` selects one. The mode badge (`setSource()`) is updated from the actual result of the last fetch attempt, not the static config flag, including automatic fallback-with-toast if a live call throws — verified behavior, not just a stated intention.

## TxLINE endpoints used

| Endpoint | Status |
|---|---|
| `POST /auth/guest/start` | Confirmed working live, zero gas |
| `GET /api/scores/stat-validation` | Confirmed real URL and auth-header structure (live 401 on invalid token); not yet exercised with a real activated token |
| Fixtures / Odds / live Scores snapshot endpoints | Confirmed to exist per API reference sidebar; exact paths/schemas unconfirmed — not wired live, explicitly stubbed to throw and fall back to simulation |

## Business highlights

- Directly implements the track's own stated mechanic ("ingest live odds and scores, detect signals, run strategies, execute decisions without manual input") rather than a loosely-related interpretation of it.
- The manual override is framed and built as a disclosed supervisory control, not a workaround — consistent with how real autonomous trading/signal systems are built.
- Settlement payouts are computed from the agent's own modeled odds data (fair-odds conversion), not an arbitrary flat number — a small but real piece of internal consistency.
