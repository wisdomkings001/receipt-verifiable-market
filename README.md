# 🩺 Pulse — Autonomous Fan Signal & Strategy Agent

Built for the Superteam Earn **World Cup Hackathon** (Consumer and Fan Experiences track), powered by **TxODDS / TxLINE**.

## The idea

An agent that watches a World Cup match's odds and scores, detects three named, explainable signals, and reacts by adjusting a simulated paper position — unattended by default, with a manual override for controlled demonstration. The track explicitly calls for autonomous agents that "ingest live odds and scores, detect signals, run strategies, and execute decisions without manual input." Pulse does exactly that, plus a deliberate, disclosed supervisory override.

The design is built around the name: a vitals-monitor aesthetic — an ECG strip that spikes every time a signal fires, color-coded by signal type — treating the market like a patient whose pulse the agent is reading in real time.

## Live vs. simulated data (please read)

- **Default: simulation.** All odds/score data flowing through the agent matches TxLINE's real API schema where applicable (the settlement proof call matches the confirmed `/api/scores/stat-validation` schema exactly).
- **Confirmed working live, zero gas:** `POST /auth/guest/start` (guest JWT).
- **Confirmed real and reachable, correctly rejects bad auth:** `GET /api/scores/stat-validation` (returns 401 to an invalid `X-Api-Token` — confirms the URL and header structure are correct).
- **Not confirmed:** exact paths/schemas for live Fixtures/Odds/Scores-snapshot endpoints. These exist per the API reference sidebar but were never verified live. `LiveProvider` does not guess a path for these — it throws explicitly, and the app falls back to simulation with a toast notice.
- The mode badge in the header always reflects the **actual** source of the last successful fetch, not a static intention — including automatic fallback if a live call fails.
- The demo video runs in simulation and discloses this on screen, same as the badge does live.

## Why the manual toggle doesn't undercut "autonomous"

Autonomous mode is the default and the primary mode being demonstrated — it runs unattended, reacting to a live timer-driven simulation with no clicks required. Manual mode is a deliberate, disclosed supervisory override: real trading/signal systems always have a kill switch or manual trigger, and including one is a sign of responsible design, not reduced autonomy. The three manual "Inject" buttons don't take a shortcut around the agent's reasoning — they mutate real fixture state (odds history, score, goal events) to deterministically satisfy the same thresholds the autonomous engine checks, then run through the exact same `detectSignals()` and `applyStrategy()` functions. Same pipeline, different trigger source — see code comments in `index.html` for exactly where this is enforced.

## Known limitations (disclosed honestly)

- **State is in-memory only.** Bankroll, position, and feed history are not persisted — reloading the page resets everything. Don't reload mid-demo.
- **One open position across the whole book at a time.** If you switch the active fixture while holding a position, the agent will not open a second position elsewhere — it will note that it's already committed and decline to act further until the held position settles. This is intentional (avoids overtrading) but worth knowing before a demo recording.
- **Live odds/scores integration is not wired** — only the settlement proof call has confirmed-real request logic. See "Live vs. simulated data" above.

## Run locally

Open `index.html` in a browser. Requires `https://` or `localhost` (not a bare `file://` path in some browsers) because it uses the Web Crypto API.

```bash
npx serve .
```

## Deploy

### GitHub Pages
1. Push `index.html` to the repo root.
2. **Settings → Pages → Source: Deploy from a branch → main → / (root) → Save.**
3. Visit the generated `https://<username>.github.io/<repo>/` URL after 1–2 minutes.

### Railway (alternative)
1. New Railway project → "Deploy from GitHub repo."
2. Use a minimal static site template pointing to `index.html`.
3. Deploy and grab the public URL.

## Configure (top of `index.html`)

```js
const CONFIG = {
  USE_LIVE_DATA: false,   // false = simulation (default, recommended for demo)
  TXLINE_BASE_URL: "https://txline.txodds.com",
  TXLINE_API_TOKEN: ""    // long-lived token from on-chain activation (gas-blocked, not yet available)
};
```

## TxLINE endpoints used

- `POST /auth/guest/start` — guest JWT (confirmed working live, zero gas).
- `GET /api/scores/stat-validation` — three-stage Merkle proof, used for the Settlement Receipt feature (confirmed real URL and auth-header structure; not yet exercised with a real activated token).

## Credits

TechCraft & Coding By Wisdom — GitHub [@wisdomkings001](https://github.com/wisdomkings001), Instagram [@wisdomtech_creations](https://instagram.com/wisdomtech_creations).
