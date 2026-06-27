# TxLINE API — Team Feedback (Pulse submission)

## What worked well

- **Guest auth is excellent** — instant, working JWT, zero gas, no signup friction.
- **The stat-validation endpoint's error behavior is genuinely useful for development** — a real 401 to an invalid `X-Api-Token` let us confirm our request shape, URL, and headers were correct before ever having a real activated token. Good signal-to-noise for debugging.
- **The three-stage Merkle proof structure is a strong primitive** for any product needing to prove a settlement rather than just assert it — we reused it directly for Pulse's Settlement Receipt feature.

## Where we hit friction

- **Free-tier activation still requires mainnet gas**, same finding as our previous submission this hackathon. This was the single largest blocker to building Pulse with real live odds/scores data — every endpoint beyond guest auth needs a real activated `apiToken`, which needs an on-chain mainnet transaction. We'd repeat our earlier suggestion: a devnet path, a sponsored-gas option for hackathon participants, or a short-lived sandbox token issued alongside guest auth.
- **Exact paths/schemas for the Fixtures, Odds, and live Scores-snapshot endpoints weren't fully confirmable** from the docs sidebar alone — we could see they exist (named in the API reference navigation) but didn't have time to confirm their full request/response shape via live calls the way we did for `stat-validation`. Clearer cross-linking from the sidebar list directly to each endpoint's full schema page would help future builders move faster.
- **No SSE example response shown** for the real-time streaming endpoints (odds/scores) in what we could review — a sample event payload would help builders plan client-side parsing before writing code against it blind.

## Net take

The verification layer is the most compelling part of TxLINE's design, and it's reusable across very different product ideas (we used it in two separate submissions this hackathon). The mainnet-gas requirement for free-tier activation remains the highest-leverage friction point to address for lowering the barrier to entry.
