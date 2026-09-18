# Utexo public docs — interview prep digest

Crawl date: 2026-09-17 (America/El_Salvador). Primary source: https://docs.utexo.com (index: https://docs.utexo.com/llms.txt). Related: https://docs.wdk.tether.io (UTEXO WDK community modules). **Nothing below invents endpoints/methods; unknowns are marked.**

---

## Product suite map

| Product | What it does | Who it’s for | How it relates |
| --- | --- | --- | --- |
| **SDK / client libraries** | Programmatic RGB asset ops + Lightning (platform-dependent): wallets, invoices, transfers, channels, LSP/APay, backup | App / wallet builders (Web, RN, Node/Bare) | Client-side / local RLN or WDK bindings; **non-custodial** keys stay local. Complements Cloud; does not replace Mint/Swap APIs. |
| **Cloud (RLN control plane)** | Managed provision/upgrade/backup/destroy of **RGB Lightning Nodes**; webhooks for node status | Operators who need a hosted RLN without self-hosting | Control plane API (`cloud-api.thunderstack.org`) ≠ node runtime API. App still calls **RLN REST** on the node. Powered by Thunderstack. |
| **RGB Lightning Node (RLN)** | One daemon for **on-chain RGB** + **RGB-over-Lightning**; REST JSON API | Self-hosters or Cloud users needing node-level control | Runtime under Cloud or self-hosted. Docs: mainnet = on-chain RGB only; Lightning = **testnet / beta**. |
| **Mint** | Cross-chain USDT ↔ Bitcoin RGB (via Arbitrum hub + USDT0/LayerZero) | Users / apps moving USDT onto/off Bitcoin RGB | Separate REST gateway; TEE federated signers. Not the same as Swap. |
| **Swap (HotPot)** | Resolver RFQ + signed intents; cross-chain settlement (EVM, Tron, Solana, Bitcoin HTLC) | Integration partners (wallets, exchanges); resolvers as LPs | Partner REST API (`X-API-Key`); Utexo coordinates, **does not custody**. Separate from Mint. |

**Relation summary (from Architecture + Product Suite):**  
Bitcoin (anchor) → Lightning (execution) → RGB (assets/CSV) → Utexo execution layer (SDK/API abstraction) → Mint (ingress/egress) + Swap (intent DEX).

**Naming note:** Product Suite page markets Swap as “non-custodial BTC ↔ USDT … LP functionality”; Architecture/Swap docs describe **HotPot intent protocol** and multi-chain RFQ. Prefer Architecture/Swap sections for interview accuracy.

---

## SDK deep dive

### Package matrix (current)

| Package | Platform | Role | Status (per docs) |
| --- | --- | --- | --- |
| `@utexo/wdk-rgb-lightning` | Node.js & Bare | **Current** Node path: RLN channels, invoices, payments, LSP, VSS | Pre-1.0 beta |
| `@utexo/wdk-wallet-rgb` | Node.js & Bare | On-chain RGB issuance/inventory (WDK) | Stable |
| `@utexo/rgb-sdk-web` | Browser (WASM RLN) | `UTEXOWallet` — RGB + Lightning in-browser | Beta |
| `@utexo/rgb-sdk-rn` | React Native (iOS/Android) | On-device RLN via TurboModule | Beta |
| `@utexo/rgb-sdk` | Node.js | **Archived** (2026-07-28); last `1.0.0-beta.9` | Do not use for new work |
| `@utexo/rgb-sdk-core` | Shared | Types / conformance for Web + RN | Shared by Web/RN |

**WDK rule:** `@utexo/wdk-wallet-rgb` and `@utexo/wdk-rgb-lightning` each need a **separate `dataDir`** — they do not share RGB asset records.

Also documented on **docs.wdk.tether.io** as community modules (e.g. `@utexo/wdk-rgb-lightning@0.1.0-beta.15` + peer `@utexo/rgb-lightning-node-nodejs`).

### Install (from docs)

```bash
# Current Node / Bare Lightning
npm install @utexo/wdk-rgb-lightning
npm install @utexo/rgb-lightning-node-nodejs   # or -bare

# On-chain WDK
npm install @utexo/wdk-wallet-rgb

# Web / RN
npm install @utexo/rgb-sdk-web
npm install @utexo/rgb-sdk-rn

# Legacy only (archived)
npm install @utexo/rgb-sdk
```

Sources: https://docs.utexo.com/product-suite/sdk.md , platform SDK pages, https://docs.wdk.tether.io/sdk/community-modules/wdk-rgb-lightning/guides/get-started/

### Auth / keys / custody

- **SDK:** BIP-39 mnemonic (or seed); **never transmitted** to remote servers (stated for Web/SDK overview).
- Lightning node often **external-signer / VLS**: mnemonic in host secret manager; channel crypto in-process VLS (WDK Lightning).
- Web: `password` encrypts local RLN state; optional `vssUrl` for encrypted cloud backup.
- RN: `NativeExternalRLNSigner` or `PasswordRLNSigner`.
- Cloud: Bearer **Cloud API token** for control plane; node access via **mTLS** or Bearer token (separate credentials).
- Swap: `X-API-Key` on protected partner endpoints (base URL obtained from Utexo team; includes `/v1`).
- Mint: mostly public GETs; some POSTs use signature over fixed message `Bridge Authentication Proof`.

### Networks / env (SDK overview)

| Identifier | RGB transport (example) | Indexer (example) |
| --- | --- | --- |
| `mainnet` | `rpcs://rgb-proxy-mainnet.utexo.com/json-rpc` | `ssl://electrum.iriswallet.com:50003` |
| `testnet` | `rpcs://rgb-proxy-testnet3.utexo.com/json-rpc` | `ssl://electrum.iriswallet.com:50013` |
| `utexo` (signet, default for dev) | `rpcs://rgb-proxy.utexo.com/json-rpc` | `https://esplora-api.utexo.com` |

Web defaults also list LN gateway `wss://ln-gateway-signet.utexo.com` for `utexo`. Faucet: Telegram `@Utexo_RLN_bot`.

### Core types / methods (evidenced)

**Web / RN — `UTEXOWallet`** (shared conformance; not identical to WDK):

- Lifecycle: Web `init()` → optional VSS restore → `unlock()` → `goOnline()`; RN `init()` → `unlock()` / `reinit()` / `destroy()`.
- RGB: `onchainReceive` / `onchainSend` (blinded vs witness); `issueAssetNia` / IFA / CFA (/ UDA on RN); `createUtxos`; `listAssets`; `refreshWallet` / `syncWallet`.
- Lightning: `createLightningInvoice`, `payLightningInvoice`, `getLightningSendStatus`, peers/channels, HODL, LSP/`createLsp`, APay.
- Backup: file backup + VSS (`backupNow`, `restoreFromVss` on Web).

**WDK Lightning — `WalletManagerRgbLightning` + `WalletAccountRgbLightning`:**

- Construct with mnemonic + `{ network, dataDir, ... }`; `getAccount(0)`; `unlock({ indexer_url | bitcoind_rpc_*, proxy_endpoint, announce_* })`.
- Invoices/payments: `createInvoice` / `createLightningInvoice`, `sendPayment`, HODL, `createRgbInvoice`, `transfer`, channels, LSP, VSS.
- Errors (exported): `RgbLightningError`, `UnlockError`, `AccountLockedError`, `VssError`, `VssNotConfiguredError`, `ApayError`, …

**WDK on-chain — `WalletManagerRgb` / `WalletAccountRgb`:** issuance, `blindReceive` / `witnessReceive`, `transferAsset`, UTXOs, backup.

**Archived Node `@utexo/rgb-sdk`:** `UTEXOWallet(mnemonic, opts)` + `initialize()`; `blindReceive` + `send()` — documented for historical pin only.

### Example flows (adapted from public docs — cite URLs)

**1) Web wallet init** — https://docs.utexo.com/sdk/web-sdk.md

```typescript
import { UTEXOWallet, generateKeys } from '@utexo/rgb-sdk-web';
const keys = await generateKeys('utexo');
const wallet = new UTEXOWallet({ mnemonic: keys.mnemonic, password: '…', network: 'utexo' });
await wallet.init();
await wallet.unlock();
```

**2) Blinded RGB receive + send (Web)** — same page

```typescript
const blind = await wallet.onchainReceive({ assetId, amount: 100, witness: false });
await sender.onchainSend({ invoice: blind.invoice, assetId, amount: 100, feeRate: 2 });
await wallet.refreshWallet();
```

**3) WDK Lightning unlock + invoice** — https://docs.utexo.com/sdk/wdk-rgb-lightning.md (content under that path in crawl)

```typescript
import WalletManagerRgbLightning from '@utexo/wdk-rgb-lightning';
const manager = new WalletManagerRgbLightning(mnemonic, { network: 'regtest', dataDir: '/path' });
const account = await manager.getAccount(0);
await account.unlock({
  indexer_url: 'tcp://localhost:50001',
  proxy_endpoint: 'rpc://localhost:3000/json-rpc',
  announce_addresses: [],
  announce_alias: 'my-node',
});
const invoice = await account.createInvoice({ amt_msat: 5_000, expiry_sec: 3600 });
await account.sendPayment({ invoice: bolt11 });
```

**4) Lightning payment (Web)** — web-sdk.md

```typescript
const { lnInvoice } = await receiver.createLightningInvoice({ expirySeconds: 900, asset: { assetId, amount: 10 } });
const { txid: paymentHash } = await sender.payLightningInvoice({ lnInvoice });
const status = await sender.getLightningSendStatus(paymentHash); // Pending|Claimable|…|Succeeded|Failed
```

### Error handling / config notes

- Branch on typed errors (`err.name` / `err.code`) for WDK Lightning.
- Amounts: integer **base units** (`10 ** precision`); do not assume `1` = one display unit.
- Vanilla vs colored derivation paths; fund vanilla → `createUtxos` before RGB.
- Lightning needs peers/channels (or LSP); init alone is not enough.
- Async payments (APay): inbound operational; outbound “in active development” (SDK overview).

---

## Key API / Cloud surfaces

Utexo has **no single global API** (API Reference Overview).

### 1) Utexo Cloud API  
Base: `https://cloud-api.thunderstack.org` — Bearer token.

| Group | Endpoints (evidenced) |
| --- | --- |
| Nodes | `GET/POST /api/nodes`, `GET /api/nodes/{id}`, `DELETE /api/nodes`, `POST …/start|stop|upgrade|settings`, `GET …/latest-rln-image` |
| Webhooks | `GET /api/webhook-public-key`; node `settings.webhookUrl` |
| Logs | `POST/GET /api/nodes/{id}/logs` |

### 2) RGB Lightning Node runtime API  
Cloud node URL or self-hosted `:daemon-listening-port` (default 3001). Auth: mTLS / Bearer / Biscuit (self-hosted).

Endpoint **groups** (all POST unless noted): wallet/balances (`/address`, `/btcbalance`, `/createutxos`, …), assets (`/issueassetnia`, `/sendrgb`, …), payments (`/lninvoice`, `/rgbinvoice`, `/sendpayment`, …), channels/peers, swaps (`/makerinit`, `/taker`, …), lifecycle (`/init`, `/unlock`, `/backup`, …). Full OpenAPI: https://utexo-protocol.github.io/rgb-lightning-node

### 3) Mint API  
Base (test/dev): `https://transfer.gateway.dev.utexo.com/api/v0` — interactive docs under `/docs/`. **Bitcoin mainnet not available yet.**

| Group | Endpoints |
| --- | --- |
| Networks | `GET /networks`, `GET /networks/{id}/supported-tokens`, `GET …/balance/…` |
| Transfers | `GET /transfers/estimate/…`, `POST /transfers/bridge-in-signature`, `POST /transfers/verify-bridge-in`, `POST /transfers/submit-transaction`, `GET /transfers/history/…`, `GET /transfers/invoice/…` |

RGB Lightning destinations (`networkId` 94/95): **API surface present; “not available for use yet.”** Fee rate stated on Mint product page: **0.03%**.

### 4) Swap (partner) API  
Base URL: **placeholder** — obtain from Utexo; includes `/v1`. Auth: `X-API-Key`.

| Group | Endpoints |
| --- | --- |
| Discovery | `GET /healthcheck`, `GET /networks`, `GET /tokens` |
| Quotes | `POST /quote` |
| Intents | `POST /intents`, `POST /intents/{id}/approvals`, `GET /intents/{id}/status` |
| Swaps | `GET /swaps`, `GET /swaps/{id}` |
| Affiliates | `POST/GET /affiliates` |

Approval mechanisms: `permit2` (EVM/Tron), `htlc`+PSBT (Bitcoin), `cosign` (Solana).  
Partner SDKs mentioned: TypeScript / Go / Rust; TS examples use legacy `@hot-pot/hotpot-sdk-ts` (confirm version with team).

### 5) Swap resolver Protocol API (separate)  
Resolver webhooks: `X-Signature` (Ed25519) + `X-Timestamp`; events `IntentAssigned`, `DepositConfirmed`, `WithdrawReady`, `SwapConfirmed`, `RefundConfirmed`; reporting via `POST /v1/intents/{id}/deposit|fulfill` etc. **Many gaps** documented on Validation Gaps page.

### 6) “RGB Node API”  
Listed in API overview; **no published page found** (404 on guessed path). Treat as unknown / unpublished.

---

## End-to-end flows an interviewee should narrate

1. **On-chain RGB USDT transfer (app SDK)**  
   Fund BTC → create colored UTXOs → receiver `onchainReceive` (blinded/witness) → sender `onchainSend` / WDK `transfer` → both `refreshWallet` → confirm balance. Client-side validation; Bitcoin UTXO anchors state; proxy delivers consignments.

2. **RGB over Lightning payment**  
   Connect peer → open (possibly RGB) channel → wait usable → create LN invoice (BTC or asset) → pay → poll send status. Requires liquidity/LSP on real deployments. **Mainnet Lightning still restricted per Node Overview.**

3. **Mint EVM → Bitcoin RGB**  
   Discover networks → estimate → `bridge-in-signature` → user `fundsIn` (or entrypoint) → `verify-bridge-in` → poll history to `FINISHED`. Hub: Arbitrum + USDT0/LayerZero. TEE threshold signers + in-enclave RGB/SPV checks.

4. **Mint RGB → EVM**  
   Pre-register → pay returned RGB invoice → verify → wait Bitcoin confirmations → USDT out on EVM.

5. **Swap (partner)**  
   healthcheck → networks/tokens → quote → intent (addresses) → chain-specific approval → monitor status → settlement or refund. Utexo coordinates; resolvers provide liquidity; per-swap escrow isolation.

6. **Cloud-hosted RLN**  
   Create node (API/dashboard) → wait `RUNNING` → connect mTLS or token → call RLN REST → optional webhooks on status (`RUNNING`/`FAILED`/…).

---

## Prerequisites the docs assume

- **Bitcoin UTXO model**; fees in sats; PSBT / Taproot awareness for advanced flows.
- **Lightning:** channels, BOLT11, HTLCs, peers, liquidity, LSP; LDK as RLN base.
- **RGB:** client-side validation; consignments; blinded vs witness invoices; single-use seals; NIA/IFA/CFA/UDA schemas; vanilla vs colored paths.
- **Cross-chain:** EVM contracts, Permit2, LayerZero/USDT0, Tron TIP-712, Solana versioned txs.
- **Ops:** BIP-39 mnemonics; encrypted backups vs VSS; TEE/Nitro concepts for Mint.

---

## Architecture / custody / fees / webhooks (extracted)

### Architecture (textual diagram)

```text
Application
  ├─ RGB Lightning Node API ──► RLN ──► Bitcoin + RGB indexer + RGB proxy + LN peers (testnet)
  └─ Cloud API ───────────────► Utexo Cloud control plane ──► managed RLN instances
```

Stack table: Bitcoin / Lightning / RGB / Utexo / Mint / Swap (Architecture page). Claims ~**200 ms** payment latency in marketing-style architecture copy.

### Custody claims (as stated)

| Area | Claim |
| --- | --- |
| SDK | Non-custodial; keys/mnemonics not sent to servers |
| Swap | Utexo does not custody user funds / not counterparty |
| Mint | “No custodian holds funds during mint” (product suite); signing via **federated TEE threshold** (orchestrator/connectors don’t hold keys) — still a multi-party TEE custody model for mint keys |
| Cloud | Managed infra; separate control-plane trust; RLN holds wallet/LN state |

### Fee models (evidenced)

- Positioning: **deterministic / protocol-level fees** (What Utexo Is) — exact schedule **not published** in crawled pages.
- Mint: **0.03%** commission + chain gas + BTC RGB fee; `CommissionManager` on-chain.
- Swap: RFQ pricing; `slippageBps`; affiliate `feeBps`; amounts in “lots”.
- SDK: on-chain sends take `feeRate` (sat/vB); Lightning routing fee caps appear on WDK `sendPayment`.

### Webhooks

| System | Mechanism |
| --- | --- |
| Cloud | `POST` to `webhookUrl`; `X-Utexo-Signature`; statuses RUNNING/STARTING/PAUSED/FAILED/IN_PROGRESS |
| Swap resolvers | Ed25519 `X-Signature` over `timestamp + raw body`; lifecycle events listed above |

---

## Likely interview questions (grounded in these docs)

1. How do Bitcoin, Lightning, and RGB divide responsibilities in Utexo’s stack?  
2. Vanilla vs colored addresses — why `createUtxos` before RGB receive?  
3. Blinded vs witness RGB invoices — when is `witnessData` required?  
4. Why is `@utexo/rgb-sdk` archived, and what replaces it on Node?  
5. Why must `wdk-wallet-rgb` and `wdk-rgb-lightning` use different `dataDir`s?  
6. Mainnet vs testnet: what can RLN do on each today?  
7. Cloud API vs RLN API — which provisions nodes vs which pays invoices?  
8. Mint security: Nitro enclaves, threshold signing, burnId replay protection, SPV in TEE.  
9. Mint flow: why Arbitrum/USDT0 as hub? What’s still WIP (Lightning dest, BTC mainnet)?  
10. Swap intent lifecycle statuses and approval mechanisms per chain.  
11. Difference between Cloud webhooks and Swap resolver webhooks.  
12. How does VSS backup interact with Lightning channel state / multi-device risk?  
13. APay / virtual channels — what’s production-ready vs in development?  
14. Where docs admit gaps (resolver Validation Gaps, unpublished OpenAPI for Mint/Cloud).

---

## Gaps / thin / blocked pages

| Issue | Detail |
| --- | --- |
| No 403 observed | Fetches of docs.utexo.com and docs.wdk.tether.io returned content (200). |
| `/sdk/utexo-sdk` | HTTP 308; crawl resolved to **wdk-rgb-lightning** content — legacy path / routing quirk. |
| “RGB Node API” | Mentioned in API overview; **page not found** (404). |
| Swap base URL / keys | Placeholders only; partner-gated. |
| Mint OpenAPI | API overview: “Not currently published”; HTML docs at gateway `/docs/`. |
| Cloud OpenAPI | “Not currently published”. |
| Product Suite vs Swap docs | Suite page still reads like AMM/LP BTC↔USDT; Swap section is HotPot RFQ — reconcile in interviews. |
| Mint Getting Started | UI-oriented; duplicated sections in fetch; route caveat (Arbitrum mainnet → Utexo **signet**). |
| Resolver Validation Gaps | Explicit incomplete: missing base URLs, schemas, webhook freshness window, typos (`tapscipt`). |
| Exact protocol fee schedule | Claimed deterministic; **numeric schedule not in crawled SDK pages** (Mint 0.03% is the clear figure). |
| Outbound APay | Documented as in active development. |

---

## Source URL list

**Index / intro**  
- https://docs.utexo.com/llms.txt  
- https://docs.utexo.com/  
- https://docs.utexo.com/what-utexo-is  
- https://docs.utexo.com/getting-started/product-suite  
- https://docs.utexo.com/getting-started/architecture  
- https://docs.utexo.com/getting-started/glossary  
- https://docs.utexo.com/getting-started/quickstart/overview  
- https://docs.utexo.com/getting-started/quickstart/node-js  

**SDK**  
- https://docs.utexo.com/product-suite/sdk  
- https://docs.utexo.com/sdk/web-sdk  
- https://docs.utexo.com/sdk/react-native-sdk  
- https://docs.utexo.com/sdk/wdk-overview  
- https://docs.utexo.com/sdk/wdk-wallet-rgb  
- https://docs.utexo.com/sdk/wdk-rgb-lightning (also served under legacy `/sdk/utexo-sdk`)  

**Node / Cloud / security**  
- https://docs.utexo.com/overview  
- https://docs.utexo.com/rgb-lightning-node/rgb-lightning-node-api  
- https://docs.utexo.com/product-suite/RLN-overview  
- https://docs.utexo.com/cloud/getting-started  
- https://docs.utexo.com/cloud/rln-node/connect-rln-node  
- https://docs.utexo.com/cloud/webhooks  
- https://docs.utexo.com/access-token-authorization/cloud-api  
- https://docs.utexo.com/security/rln-remote-signer  
- https://utexo-protocol.github.io/rgb-lightning-node  

**Mint**  
- https://docs.utexo.com/product-suite/mint  
- https://docs.utexo.com/product-suite/mint-api-reference  
- https://docs.utexo.com/mint/getting-started  
- https://transfer.gateway.dev.utexo.com/api/v0/docs/  

**Swap**  
- https://docs.utexo.com/product-suite/swap  
- https://docs.utexo.com/product-suite/swap/security-model  
- https://docs.utexo.com/product-suite/swap/integration/overview  
- https://docs.utexo.com/product-suite/swap/integration/authorization  
- https://docs.utexo.com/product-suite/swap/integration/user-flow  
- https://docs.utexo.com/product-suite/swap/api/networks-and-tokens  
- https://docs.utexo.com/product-suite/swap/api/quotes  
- https://docs.utexo.com/product-suite/swap/api/intents-and-approvals  
- https://docs.utexo.com/product-suite/swap/api/swaps  
- https://docs.utexo.com/product-suite/swap/api/affiliates  
- https://docs.utexo.com/product-suite/swap/resolver-integration/webhooks-and-lifecycle  
- https://docs.utexo.com/product-suite/swap/resolver-integration/validation-gaps  

**API hub**  
- https://docs.utexo.com/api-reference/overview  

**Related WDK (Tether)**  
- https://docs.wdk.tether.io/sdk/community-modules/wdk-rgb-lightning/  
- https://docs.wdk.tether.io/sdk/community-modules/wdk-wallet-rgb/  
- https://docs.wdk.tether.io/sdk/community-modules/wdk-rgb-lightning/api-reference/  
- https://docs.wdk.tether.io/sdk/community-modules/wdk-rgb-lightning/configuration/  
- https://docs.wdk.tether.io/sdk/community-modules/wdk-rgb-lightning/guides/get-started/  

**Marketing (secondary)**  
- https://utexo.com/api-product  

