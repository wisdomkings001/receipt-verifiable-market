# Technical Documentation — Receipt

## Core idea
A soccer prediction market where settlement is verifiable. On full time, the app calls TxLINE's three-stage Merkle proof endpoint for the final score stat and renders a Receipt that recomputes and checks the proof client-side.

## Architecture highlights
- Single dual-mode data layer. LiveProvider and SimProvider share identical signatures and return identically-shaped objects. CONFIG.USE_LIVE_DATA chooses one. UI/receipt/prediction code never branches on mode; the LIVE/SIMULATED label is derived from the actual data source in use.
- Real proofs in simulation. A generic SHA-256 buildMerkle produces a self-consistent 3-stage tree (event stats -> event sub-tree -> batch). verifyProof recomputes Stages 1 & 2 and only shows "Verified" when they pass. Deterministic, inspectable resolution logic.
- Honest Stage 3. The confirmed stat-validation schema does not return the global batch root, so Stage 3 shows the proof nodes and labels the anchor "anchored" — the batch root would come from the dedicated batch-proof endpoint. This mirrors real API behavior rather than inventing a field.
- Lightweight persistence. Firebase Realtime DB via REST (no SDK) when configured, else localStorage. Prediction schema: predictions/{predictionId}: { fixtureId, sessionId, predictedOutcome, createdAt, status, ... }.

## TxLINE endpoints used
POST /auth/guest/start, POST /api/token/activate, GET /api/scores/stat-validation (core). Fixtures/odds/scores snapshot + SSE endpoints are stubbed in LiveProvider for future expansion.

## stat-validation response schema (confirmed field names)
ts (int64), eventStatRoot (hash), statToProve { key, value, period }, statProof (ProofNode[]), subTreeProof (ProofNode[]), mainTreeProof (ProofNode[]), summary { fixtureId, updateStats { updateCount, minTimestamp, maxTimestamp }, eventStatsSubTreeRoot }. Optional: statToProve2, statProof2 when statKey2 is supplied.