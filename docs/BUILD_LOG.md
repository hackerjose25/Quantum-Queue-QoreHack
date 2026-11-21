# Build log

Only observed work and test results are recorded here.

## 2026-11-16: Mempool Guard built in `quest/`
- **Scaffold:** `create-qorechain-dapp@0.14.0 --template fullstack-web --network testnet` (pnpm), with `@qorechain/sdk` 0.14.0.
- **Build:** `pnpm build` (tsc + vite) passes. The main bundle is about 60 KB gzipped, with the SDK split out and loaded when the player connects a wallet.
- **Engine tests (15/15 pass):**
  - The deck is deterministic for a given seed.
  - Every legit card passes all rules.
  - Replay scoring equals live scoring.
  - Post-game, unfinished and malformed move strings are rejected.
  - The run memo is 59 bytes or less (the limit is 256).
- **Live Diana tests (through the dev proxy) pass:**
  - Latest and historic blocks are readable. The lowest retained height observed was 8,790,799.
  - 5/5 live hybrid transactions pass the hybrid checks.
  - An unknown hash returns "pending", not "verified".
  - A non-game `MsgSend` is rejected as a run.
  - The leaderboard query works (0 entries).
- **Run verification on a real seed block:**
  - An honest run verifies.
  - An inflated score, a different seed block and a signer mismatch are all rejected.
- **Encoder:** the `MsgSend` encoder is byte-identical to `cosmjs-types` 0.11.0.
- **Browser (headless Edge + Playwright):**
  - A full playthrough has no console errors.
  - A mocked QoreX returning an unrelated real transaction hash fails closed.
  - The play screen fits without scrolling at 1900×870, 1366×650 and 390×740.
- **Fixes during testing:**
  - The feedback step showed the next card instead of the judged one.
  - A `useEffect` returned the Promise from `window.scrollTo`, which crashed the app and left a black screen. Fixed, and an error boundary was added.
- **Real on-chain runs (submitted by the user with QoreX, read back from Diana REST):**
  - `2A65CCDD709784B2A7B75E3CF8E14C13D1E4F270277489BB16378A46C488DD48`: block 10,153,126, memo `mg1:10153067:RARARAARARARARARA:1300`, code 0, 4,627 B ML-DSA-87 signature, `pqc_hybrid_verify = verified`, fee 45,000 uqor, 167,420 gas used. It appears on the leaderboard with the replay check passing.
  - `A44D2B462BE9B3C887D4C3D35B2271201587AB5805909EA2902FC795595FD0BE`: block 10,152,719, memo `mg1:10152634:AAARAAAAAA:550`, code 0, 4,627 B signature, `verified`.
- **Deployment:** added a root `vercel.json` (it builds `quest/` and outputs `quest/dist`, with the `/lcd` proxy rewrite).
- **Live deployment verified:** https://mempool-guard.vercel.app/ returns HTTP 200 from Vercel.
  - The `/lcd` proxy returns 200, and the top bar shows the live `qorechain-diana` block height.
  - The live hybrid feed loads 7 real transactions.
  - The leaderboard shows the 1,300-point run with its replay check passing.
  - A shift starts from a live seed block, and the play screen fits a 1440×900 viewport.
  - No console errors (checked with headless Edge + Playwright).
- **More on-chain runs (2026-11-20, a second address `qor1lapj3hleq2nk699zh00d3vf6753p0vmw9m632t`, read back from Diana REST on 2026-11-21):** each has code 0, a 1 uqor `MsgSend` to the scoreboard, the `PQCHybridSignature` extension with a 4,627 B ML-DSA-87 signature, and `pqc_hybrid_verify = verified` (`dilithium5`, `hybrid_mode = required`). The leaderboard replay was not re-checked for these runs.
  - `409A6F1B5A8080F810F9DD41E14F5BD529CBBFD1DC2AF134A9E4785C346530AA`: block 10,442,358, memo `mg1:10442257:AAAARARARRARAARAAAAAAAAA:3650` (the run shown in the demo video).
  - `93748DC22AB9BEC97FF0C20F0C361903126B31B9EA6F8DCAB2376A7B4328AB26`: block 10,440,962, memo `mg1:10440625:AARARRARARRAARARAARA:2650`.
  - `09CBFF4B918CBC0397F4114AB92E41F4DD6CCC6D1452843D24B7C25E32C77D08`: block 10,441,802, memo `mg1:10441677:TRAAARRTRAAAARAAARA:2450`.
- **Submission check (2026-11-21):** the live demo, repo, YouTube video (https://youtu.be/ZPW_kj11_s8) and all five run transactions return HTTP 200. `pnpm install --frozen-lockfile && pnpm build` in `quest/` passes, with one Vite warning about a chunk over 500 kB.
- **Transaction size vs. Diana limits (2026-11-21, read-only queries to `api-testnet.qore.host` and `rpc-testnet.qore.host` at block 10,499,673):**
  - Run tx `2A65CCDD709784B2A7B75E3CF8E14C13D1E4F270277489BB16378A46C488DD48`, fetched raw from RPC `/tx`, is 5,022 bytes encoded. The 4,627 B ML-DSA-87 signature is about 92% of that. Gas used/wanted: 167,420 / 300,000, code 0.
  - Consensus params: block `max_bytes` = 22,020,096 (21 MiB), block `max_gas` = -1 (unlimited). One run tx is about 0.02% of the block byte limit.
  - The per-tx mempool limit (`max_tx_bytes`) is node config, not a consensus param, and the RPC doesn't expose it, so its value on Diana was not confirmed. No chain or node limits were changed for this project. All runs were accepted on stock testnet settings.
