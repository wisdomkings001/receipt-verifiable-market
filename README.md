# 🧾 Receipt — Verifiable Prediction Market with Cryptographic Settlement Proof

Built for the Superteam Earn World Cup Hackathon (Prediction Markets & Settlement track), powered by TxODDS / TxLINE.

## The idea
Most prediction markets ask you to *trust* an API result. Receipt shows you the proof. Every settled bet renders a visual Receipt: the final score plus the actual three-stage TxLINE Merkle proof that anchors the result on Solana — verified in your browser, not just asserted.

This directly implements the judges' optional "Verifiable Resolution UI" idea.

## Live vs. simulated data (please read)
> Live integration implemented and partially verified (guest auth confirmed working live; protected endpoints confirmed correctly reachable via expected 401 response). Full live data retrieval pending mainnet gas fee access for on-chain subscription activation. App defaults to simulation mode for reliable demonstration, per the track's explicit allowance of simulated data feeds.

- Default: simulation. Simulated /api/scores/stat-validation payloads match the real TxLINE schema field-for-field (same names, nesting, types).
- The simulated Stage 1 & Stage 2 Merkle proofs are real and self-consistent — recomputed and verified in-browser with SHA-256. They are not fake-shaped placeholders.
- The live code path is complete and correct per the confirmed docs; it is NOT claimed to have run end-to-end. What is confirmed: the guest JWT endpoint works live; the protected stat-validation endpoint is reachable and correctly returns 401 to a bad X-Api-Token.
- Every receipt carries a visible LIVE / SIMULATED badge. The demo video runs in simulation and says so.

## Known limitations (documented, not hidden)
- Live score progression is not wired: startMatch() always runs the local simulator. Only the settlement proof call (getStatValidation) has a real live code path. Flipping USE_LIVE_DATA=true would still simulate match play until the fixtures/scores endpoints are wired.
- Match progress (State.scores) is in-memory only. Reloading the page after full-time but before settling loses the live score, so Settle cannot re-activate for that prediction. Predictions themselves persist (Firebase/localStorage).
- seq is set from match minute as a stand-in; the real seq is the score-event sequence number.
- statKey + statKey2 in one call assumes the API returns two independent stat proofs; the docs example is a two-stat predicate. Verify empirically once live.

## Run locally
Open index.html in a browser. It MUST be served over https:// or localhost because it uses the Web Crypto API. e.g. `npx serve .`

## Deploy
- GitHub Pages: push index.html to the repo root, enable Pages on the main branch.
- Railway / any static host: serve the single file.

## Configure (top of index.html)
- USE_LIVE_DATA — false = simulation (default). Set true only with a real activated TXLINE_API_TOKEN.
- TXLINE_API_TOKEN — long-lived token from on-chain activation (gas-blocked, not yet available).
- FIREBASE_DB_URL — optional Firebase Realtime Database URL; empty falls back to localStorage.

## TxLINE endpoints used
- POST /auth/guest/start — guest JWT (confirmed working live).
- POST /api/token/activate — token activation after on-chain subscribe (implemented per docs; gas-blocked).
- GET /api/scores/stat-validation — core endpoint: three-stage Merkle proof for a score statistic.

## Credits
TechCraft & Coding By Wisdom — GitHub @wisdomkings001, IG @wisdomtech_creations.