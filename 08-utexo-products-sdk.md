# Utexo product suite + SDK — interview study pack

**Primary sources (read these, don’t invent):**
- https://docs.utexo.com/getting-started/product-suite
- https://docs.utexo.com/getting-started/architecture
- https://docs.utexo.com/product-suite/sdk.md
- https://docs.utexo.com/sdk/wdk-overview
- https://docs.utexo.com/sdk/wdk-rgb-lightning (WDK LN reference)
- https://docs.utexo.com/getting-started/quickstart/overview
- Index: https://docs.utexo.com/llms.txt

**Study goal:** Narrate the product map + pick the right SDK package + walk one end-to-end flow (RGB receive/send or LN invoice/pay) with correct method names and pitfalls.

---

## 1. Product suite map (memorize)

From Product Suite docs — composable stablecoin-native primitives on Bitcoin:

| Product | One-liner (docs) | Interview angle |
|---|---|---|
| **SDK** | REST/client libs for native USDT transfers + RGB ops on Bitcoin | App integration surface; non-custodial client-side validation |
| **Cloud** | Managed RGB-enabled Lightning nodes (no self-host) | Control plane: lifecycle, health, backup/restore, VSS |
| **Mint** | Cross-chain USDT → native Bitcoin **RGB USDT** (EVM / non-EVM) | Liquidity gateway; not wrapped/synthetic per docs |
| **Swap** | Non-custodial BTC ↔ USDT with instant finality + LP | Intent/RFQ model (HotPot); resolvers; atomic settle |

**Line for interview:** “Each component is a layer — settlement/execution (SDK+LN+RGB), infra (Cloud/RLN), inbound liquidity (Mint), rebalancing/FX (Swap) — composable via one stack.”

---

## 2. Architecture stack (from Architecture page)

| Layer | Role |
|---|---|
| Bitcoin | Settlement anchor / finality / dispute |
| Lightning | Off-chain execution, near-instant payments |
| RGB | Asset issuance + **client-side validation** |
| **Utexo execution layer** | Routing, liquidity, fees, SDK/API abstraction |
| Mint | ETH/Tron/(+ docs: Solana via USDT0) ↔ RGB USDT |
| Swap | Cross-chain / BTC↔USDT via HotPot intents |

**Docs claims to handle carefully (ask, don’t over-sell):**
- Pooled liquidity / channel mgmt abstracted for API apps
- ~200 ms latency class claim
- Mint: TEE federation / Nitro enclaves / threshold signing described in architecture
- Swap: quote → intent → sign → resolver escrow → fulfill or revert

**Fallback story:** Off-chain fails → Bitcoin-backed guarantees / commitments.

---

## 3. SDK family — critical (easy to fail the interview here)

| Package | Platform | Status / use |
|---|---|---|
| `@utexo/wdk-rgb-lightning` | Node.js & Bare | **Current Node path** — channels, invoices, payments, LSP, VSS (pre-1.0 beta) |
| `@utexo/wdk-wallet-rgb` | Node.js & Bare | Stable — on-chain RGB issuance/inventory (separate `dataDir`) |
| `@utexo/rgb-sdk-web` | Browser | WASM RLN + RGB/LN |
| `@utexo/rgb-sdk-rn` | React Native | On-device RLN |
| `@utexo/rgb-sdk` | Node (archived Jul 28, 2026) | **Do not use for new integrations** |

**Trap:** Copying Web/RN `UTEXOWallet` (`init`/`unlock`/`onchainSend`) into WDK code — wrong.  
**Trap:** Using archived `@utexo/rgb-sdk` `initialize()`/`send()` for a new Node project.

### When WDK vs app SDKs
- **New Node server / wallet infra:** `@utexo/wdk-rgb-lightning` (+ `wdk-wallet-rgb` for issuance)
- **Web/RN app from scratch:** `rgb-sdk-web` / `rgb-sdk-rn`
- WDK: lower-level account/manager, external signer (mnemonic in host; VLS in-process for channel crypto)

---

## 4. Node path deep dive — `@utexo/wdk-rgb-lightning`

### Install (docs)
```bash
npm install @utexo/wdk-rgb-lightning
npm install @utexo/rgb-lightning-node-nodejs   # Node binding
```

### Lifecycle you must narrate
1. `new WalletManagerRgbLightning(mnemonic, { network, dataDir, ... })`
2. `account = await manager.getAccount(0)` — single account
3. `await account.unlock({ indexer_url OR bitcoind_rpc_*, proxy_endpoint, announce_addresses, announce_alias })`
4. Ops: peers → channels → invoices/payments / RGB
5. `manager.dispose()`

**Unlock rule:** exactly one chain backend — `indexer_url` **or** all `bitcoind_rpc_*` fields — not both.

### Networks
Docs SDK overview table:
| Env | id | Notes |
|---|---|---|
| Mainnet | `mainnet` | prod |
| Testnet | `testnet` | |
| Utexo signet | `utexo` | default for many app SDK examples |

WDK examples also show `regtest` / explicit indexer+proxy at unlock.

### Vanilla vs colored (SDK overview)
- **Vanilla:** normal BTC path — fees, funding (`getAddress()`)
- **Colored:** RGB allocations on UTXOs
- Before RGB issue/receive: fund vanilla → `createUtxos()` → refresh

### RGB invoices
- **Blinded** (common): receiver `createRgbInvoice({ witness: false, ... })`; sender `transfer({ recipient, amount, token })`
- **Witness:** `witness: true`; sender must pass `witnessData.amountSats`
- Don’t pass `witnessData` on blinded — rejected

### Lightning methods (WDK)
- `createInvoice` / `createLightningInvoice`
- `sendPayment`, `keysend`, `listPayments`, `getPayment`
- HODL: `createHodlInvoice`, `claimHodlInvoice`, `cancelHodlInvoice`
- Async/LSP: needs `lspBaseUrl` + `lspBearerToken`; inbound async operational; **outbound async in active development** (docs)

### Channels / peers
- `connectPeer('pubkey@host:port')`, `openChannel({ capacity_sat, ... })`, optional `asset_id`/`asset_amount` for RGB channels
- Virtual channels / APay: `enableVirtualChannelsV0` + `virtualPeerPubkeys`

### VSS
- Optional `vssUrl` — backup RLN KV state; **does not** replicate VLS signer DB → cross-device recovery with open channels limited (docs)

### Errors to name
`UnlockError`, `AccountLockedError`, `VssNotConfiguredError`, `ApayError`, LSP timeout errors — branch on `err.name` / `err.code`.

### Separate dataDir rule
`wdk-wallet-rgb` and `wdk-rgb-lightning` each need their **own** `dataDir` (exclusive `rgb-lib` lock; no shared asset records).

---

## 5. End-to-end flows to practice out loud

### Flow A — On-chain RGB transfer (conceptual)
1. Fund vanilla address with test BTC  
2. `createUtxos`  
3. Receiver creates blinded RGB invoice  
4. Sender transfers with assetId + base units (`10 ** precision`)  
5. Both refresh; receiver verifies balance  
6. Pending until consignment + Bitcoin confirmations

### Flow B — Lightning BTC payment (WDK)
1. Unlock node  
2. Connect peer + open channel (or use LSP path)  
3. Receiver `createInvoice({ amt_msat, expiry_sec })`  
4. Sender `sendPayment({ invoice })`  
5. Check `listPayments` / status  

### Flow C — Product-level story (no code)
PSP wants USDT on Bitcoin: Mint (bring USDT to RGB) → SDK/Cloud execute payments on LN+RGB → Swap to rebalance BTC↔USDT → Cloud if they won’t self-host RLN.

---

## 6. Cloud / RLN (enough for interview)

- Self-hosted RGB Lightning Node **or** Utexo Cloud managed RLN  
- Cloud: create/upgrade/destroy, connect via mTLS or API token, webhooks, backup/restore  
- Remote signer / VLS keeps keys off node (security docs)  
- **Interview ask:** “For this role, is day-one work Cloud API, SDK embed, or Swap/Mint?”

---

## 7. Mint & Swap (product talking points)

**Mint:** USDT from EVM/Tron/(USDT0 path) → RGB USDT on Bitcoin; architecture describes orchestrator, connectors, TEE federation, RGB multisig mint, BTC relay SPV in enclave.  
**Swap:** HotPot intents — quote → intent → sign → resolver escrow → fulfill/refund; non-custodial, atomic.

Don’t memorize every Swap REST path unless the role is Swap-focused; know the **lifecycle**.

---

## 8. Likely tech-expert questions (docs-grounded)

1. Walk the product suite and how SDK vs Cloud vs Mint vs Swap differ.  
2. Which Node package for a **new** integration in 2026 — and why not `@utexo/rgb-sdk`?  
3. Vanilla vs colored UTXOs — why `createUtxos` matters.  
4. Blinded vs witness RGB invoice.  
5. Unlock configuration pitfalls (indexer vs bitcoind).  
6. How async/HODL/LSP payments work and what’s still in development.  
7. Why separate `dataDir` for wallet-rgb vs rgb-lightning.  
8. What VSS does and does **not** backup.  
9. How Mint preserves “native RGB USDT” vs wrapped.  
10. Swap intent lifecycle in four steps.  
11. Where client-side validation sits vs Lightning execution.  
12. Week-1 plan: which quickstart + which network (`utexo` signet) + what you’d build.

---

## 9. Spoken model answers (short)

**Q: What is Utexo?**  
“A coordination/execution layer for Bitcoin-native USDT: RGB for assets, Lightning for speed, Bitcoin for finality — exposed as SDK, Cloud RLN, Mint, and Swap.”

**Q: Where do I start coding on Node?**  
“`@utexo/wdk-rgb-lightning` — archived `@utexo/rgb-sdk` is read-only since July 2026. Issuance via `@utexo/wdk-wallet-rgb` with a separate dataDir.”

**Q: How do RGB amounts work?**  
“Integer base units; one display unit is `10 ** precision`. Resolve `assetId` from `listAssets().nia`, not a hard-coded doc ID.”

---

## 10. Study plan (next 2–3 sessions)

**Session 1 (45m):** Product suite + architecture tables — redraw from memory.  
**Session 2 (60m):** SDK family + WDK lifecycle — narrate unlock → invoice → pay.  
**Session 3 (60m):** Read WDK reference sections you can’t recite (HODL, LSP, VSS) + answer §8 Qs timed.  
**Optional:** Skim Mint + Swap overview pages if interview leans payments FX.

**Practice rule:** Open docs only to verify — then close and speak.

---

## 11. Honest trial pitch tied to docs

“In week 1 I’d run the official quickstart for the package matching the stack (WDK vs Web/RN), hit signet/`utexo`, ship a thin vertical slice (RGB invoice or LN invoice + status), and write a failure/notes doc from real SDK errors — Unlock, locked account, pending RGB, no channel.”

---

*Sources: docs.utexo.com pages listed above · Interview Prep*
