# Mañana — cram para el screen (Boosty / Utexo)

**Ricardo · entrevista tech-expert → pasantía/trial (2–3 meses)**  
**Idioma del screen:** inglés. Estudia en español; **habla las respuestas en inglés**.  
**Regla de oro:** no inventes funding, stack ni “I built Utexo X”. Mapea *tu* trabajo real al producto público.

Deck clickeable (ensayar en voz alta): [`study-deck-manana.html`](./study-deck-manana.html)  
Decks extra: [live](https://ricardofv2.github.io/interview-prep-deck/) · `study-deck.html` · `study-deck-utexo.html`  
Referencia completa: [`ALL-IN-ONE.md`](./ALL-IN-ONE.md)

---

## 0. Plan de hoy (domingo) → mañana

No leas ALL-IN-ONE entero. Habla en voz alta. Escribir ≠ entrevista.

| Bloque | Tiempo | Qué hacer |
|---|---|---|
| **A. Producto Utexo** | 40 min | Suite + stack + trampas SDK. Cierra docs y recita. |
| **B. Lightning** | 30 min | 90s + liquidez + no-route + A1 en papel |
| **C. RGB + settlement** | 25 min | RGB 60s, vs ERC-20, payment vs settlement, custody |
| **D. STAR (crítico)** | 30 min | Escribe 2–3 stories *tuyas* en `04-star-stories.md` |
| **E. Pitch + preguntas** | 15 min | Trial 60s + 4 preguntas de cierre |
| **F. Mock 10 min** | 15 min | Timer. Script de la sección 8. En voz alta. |

**Mañana, 30 min antes:** solo sección 2 (scripts) + 4 productos + paquete Node. No metas temas nuevos.

**Barra de pase:** liquidez correcta · no inventar experiencia · un plan RGB honesto · una pregunta de producto nítida.

---

## 1. Qué es esta entrevista (no te despistes)

- Gate corto de **tech expert** → si pasas, **pasantía/trial**.
- CVs enviados en julio. Employer puede ser Boosty staff-aug a Utexo, Utexo directo, o Boosty Web3 general — **pregúntalo**.
- Boosty Labs: studio Web3; Utexo es case público; founder compartido Viktor Ihnatiuk.
- Utexo: USDT nativo en Bitcoin (Lightning + RGB). Seed público Mar 2026 (Tether / Big Brain / Portal, según PR). No recites el seed a menos que pregunten.

---

## 2. Scripts que debes poder decir sin pensar

Practica cada uno **en inglés, de pie, con timer**. Si tartamudeas, recórtalo; no lo alargues.

### 2.1 Lightning — 90 segundos

> Lightning is a Bitcoin layer-2. Two parties lock BTC in a payment channel — an on-chain 2-of-2 — then update balances off-chain with signed commitment transactions. You get near-instant payments; on-chain is for open, close, and disputes. Multi-hop uses HTLCs so the payment is atomic: the receiver reveals a preimage, or timeouts refund. Capacity is the size of the pipe; liquidity is the direction. A huge channel can still fail if the split is wrong.

### 2.2 Por qué “big channel” falla

> Capacity is total BTC locked. Liquidity is how it’s split. If my local balance is near zero I can’t send. If I have no inbound room I can’t receive. Routing fails when *some hop* lacks the right direction for that amount — not because “Lightning is down.”

### 2.3 Debug “no route” (semana 1)

> I’d reproduce with the payment hash. Check invoice expiry and amount first. Then: is this no-route, insufficient local outbound, or a peer timeout? Inspect local channels, then attempted paths and fee budget. Confirm whether we’re sender, forwarder, or receiver — the checklist changes. I’d rather classify the failure than restart the node.

### 2.4 Qué es Utexo (60s, solo hechos públicos)

> Publicly, Utexo is a coordination layer for Bitcoin-native USDT: RGB for assets, Lightning for speed, Bitcoin for finality. They ship it as SDK, Cloud-managed RGB Lightning Nodes, Mint to bring USDT onto RGB, and Swap as a HotPot intent/RFQ for BTC↔USDT. Targets are PSPs, exchanges, custodians, wallets, iGaming. Boosty is publicly connected as engineering support. I haven’t shipped their internals — I’d start from the public docs and a signet vertical slice.

### 2.5 RGB — 60s + honest gap

> RGB is a Bitcoin-centric asset system using client-side validation: you verify the history for the UTXOs you care about, instead of a global ERC-20-style contract state. Assets are anchored to Bitcoin UTXOs. That gives a different privacy and scaling profile — and a harder backup/UX story. Lightning moves value fast; RGB is how stablecoin state can live in a Bitcoin-native design.
>
> I haven’t shipped RGB to production. In week one I’d run the official WDK quickstart on the `utexo` signet, ship a thin invoice→status slice, and pair with whoever owns node and liquidity.

### 2.6 Trial pitch — 60s

> I’m interested in this because it’s a real payments problem: stablecoin UX with Bitcoin settlement topology, not another token wrapper. In the first two weeks I’d get the env and docs, ship one vertical slice — invoice to webhook to ledger or SDK status — and write a failure taxonomy so we stop saying “Lightning failed.” You’ll see daily written updates, early questions, and small shipped work. My gap is production RGB/LN ops; I’ll close it by pairing and logging real SDK errors — Unlock, locked account, pending RGB, no channel — not by bluffing.

### 2.7 Settlement vs payment

> Payment is the user-visible “sent.” Settlement is when the partner’s system of record treats it as final. UI can confirm in a second while finance waits for an idempotent `payment.settled` webhook keyed by payment hash. I’d ask what “final” means here — Lightning preimage, RGB state, or their ledger credit.

---

## 3. Producto Utexo (prioridad #1 esta noche)

### Suite — memoriza esta tabla

| Producto | Una línea | Ángulo en entrevista |
|---|---|---|
| **SDK** | Libs para USDT/RGB + Lightning en el cliente | Superficie de integración; keys locales (non-custodial) |
| **Cloud** | RLN gestionados (provision/upgrade/backup) | Control plane ≠ API de pagos del nodo |
| **Mint** | USDT cross-chain → **RGB USDT nativo** en Bitcoin | Gateway de liquidez; docs: no wrapped/synthetic |
| **Swap** | BTC ↔ USDT non-custodial | **HotPot intent/RFQ**, no un AMM. Quote → intent → sign → escrow → fulfill/refund |

**Frase:** “Each layer is composable — SDK+LN+RGB execute, Cloud runs infra, Mint brings USDT in, Swap rebalances.”

### Stack

Bitcoin (ancla / disputa) → Lightning (ejecución off-chain) → RGB (activos + client-side validation) → capa Utexo (routing, fees, SDK) → Mint + Swap.

**Claims que debes manejar con cuidado (pregunta, no vendas):**
- ~200 ms de latencia (copy de arquitectura)
- Mint: TEE federation / Nitro / threshold signing
- Fees “deterministic” — **schedule completo no publicado**. Mint sí: **0.03%** + gas + fee RGB
- RLN **mainnet ≈ RGB on-chain**; Lightning sigue **testnet/beta**
- Destino Lightning en Mint: **WIP**. Mint BTC mainnet: **N/A** en docs crawleadas

### APIs (no las mezcles)

| Superficie | Qué hace |
|---|---|
| Cloud `cloud-api.thunderstack.org` | CRUD de nodos, webhooks, logs. **No paga invoices.** |
| RLN REST (en el nodo) | invoices, channels, send, RGB |
| Mint `transfer.gateway.dev.utexo.com/api/v0` | estimate, bridge-in, verify, history |
| Swap (partner, `X-API-Key`) | quote, intents, approvals, swaps |
| “RGB Node API” en el overview | **404 / no publicado** — no lo menciones como si existiera |

Cloud webhooks: `X-Utexo-Signature`. Swap resolver: Ed25519 `X-Signature`.

---

## 4. SDK — donde más se falla la entrevista

| Package | Uso |
|---|---|
| `@utexo/wdk-rgb-lightning` | **Node nuevo 2026** — channels, invoices, LSP, VSS (beta) |
| `@utexo/wdk-wallet-rgb` | Emisión / inventario RGB on-chain (stable). **Otro `dataDir`** |
| `@utexo/rgb-sdk-web` / `-rn` | Browser / React Native (`UTEXOWallet`) |
| `@utexo/rgb-sdk` | **Archivado 28 Jul 2026** — no lo uses |

**Trampas:**
1. Copiar `UTEXOWallet` (`init`/`unlock`/`onchainSend`) a código WDK.
2. Unlock con `indexer_url` **y** `bitcoind_rpc_*` a la vez. Es **uno u otro**.
3. Compartir `dataDir` entre wallet-rgb y rgb-lightning (lock exclusivo de rgb-lib).
4. Hacer RGB sin `createUtxos()` (vanilla → colored).
5. Pasar `witnessData` en invoice **blinded**.
6. Creer que `init`/`unlock` = puedes pagar. Faltan peer/channel o LSP.
7. VSS **no** replica la DB del signer VLS → recovery cross-device con channels abiertos es limitado.
8. APay outbound: **en desarrollo**. Inbound async: operacional.

### Lifecycle WDK (recítalo)

1. `new WalletManagerRgbLightning(mnemonic, { network, dataDir })`
2. `account = await manager.getAccount(0)`
3. `unlock({ indexer_url XOR bitcoind_rpc_*, proxy_endpoint, announce_* })`
4. peers → channels → invoices / RGB
5. `manager.dispose()`

Red de estudio: `utexo` (signet). Mainnet: `mainnet`.

**RGB amounts:** enteros en base units = `10 ** precision`. `assetId` de `listAssets().nia`, no un ID de docs.

**Errores a nombrar:** `UnlockError`, `AccountLockedError`, `VssNotConfiguredError`, `ApayError` — branch on `err.name` / `err.code`.

### Flows E2E (narra, no codees de memoria)

**A. RGB on-chain:** fund vanilla → `createUtxos` → receiver blinded invoice → sender `transfer` → ambos `refresh` → pending hasta consignment + confirms.

**B. Lightning BTC:** unlock → peer + channel (o LSP) → `createInvoice({ amt_msat })` → `sendPayment` → `listPayments`.

**C. Producto PSP:** Mint (USDT → RGB) → SDK/Cloud paga en LN+RGB → Swap rebalancea BTC↔USDT → Cloud si no self-hostean RLN.

---

## 5. Lightning — lo que el expert va a pinchar

### Números (hazlo en papel ahora)

Alice↔Bob capacity **1,000,000** sats. Antes: Alice 700k / Bob 300k. Alice paga 50k.

**Después:** Alice **650k** · Bob **350k** · capacity **igual**.  
*“Capacity didn’t change — only the split did.”*

Merchant 5 BTC channel, local ≈ 0 después de muchos receives: **inbound saturado**. Recibe fallan hasta rebalance / loop-out / nuevo inbound. Pregunta **dirección** antes de pathfinding.

### Fallos (síntoma → causa → check)

| Síntoma | Causa | Check |
|---|---|---|
| No route / unpaid | Liquidez en un hop, grafo stale, amount grande | Balances, rutas, amount vs capacity |
| Pending eterno | HTLC in-flight, peer offline | Timeouts, liveness, ventana de force-close |
| Send OK, receive fail | Sin **inbound** | Rebalance, LSP inbound, dual-fund |
| Fees raros | Fee policy, grafo sparse | Fee floor, MPP, canal directo |
| Force-close sorpresa | Nodo offline, sin watchtower | Always-online, watchtower |

**Frase senior:** “I’d separate routing failure from liquidity failure from peer liveness — same symptom, different fixes.”

### Vocabulario que debes usar (no recitar de lista)

HTLC · preimage · commitment tx · revocation / justice · watchtower · dual-funding · JIT channel · MPP · gossip · CLTV expiry delta · channel jamming

**Revocation (si empujan):** cada update crea un nuevo par de commitment txs. El estado viejo se hace peligroso al compartir el revocation secret: si broadcastas un estado viejo, el peer te punea. Asumes que *tú o un watchtower* ves la chain a tiempo.

**HTLC mid-path offline:** fondos locked hasta timeout; UX = pending; a veces force-close; liquidez atrapada. Force-close es **incidente**, no control flow.

**Jamming:** atacante lockea HTLCs que no settle. Mitigaciones (reputación, upfront fees) — tema **abierto**, no “resuelto”.

### BOLT11 — qué mirar en un unpaid

`amount_msat` · `payment_hash` · `expiry` · description/order id · routing hints (canal privado).  
Clock skew / expiry · amount > hop · hint faltante · fee budget bajo.

### Idempotencia

Payment hash + id interno. Webhooks = at-least-once. Credit **una** vez. “User paid / merchant unpaid”: invoice → nodo HTLC → webhook log → ledger → ¿existe preimage?

---

## 6. RGB + settlement + custody

| | USDT tipo ERC-20 | RGB (framing Utexo) |
|---|---|---|
| Estado | Contrato global | Historia client-side de *tus* coins |
| Explorer | Balances públicos | Disclosure distinto / más privado |
| Fees | Gas de esa chain | Fee market de Bitcoin + LN |
| Speed | L1/L2 de ese ecosistema | Lightning |
| Tooling | Maduro | Más nuevo; SDK importa |

**Tres dominios de riesgo — no los mezcles:**
1. Issuance / bridge (lock-mint, TEE)
2. Lightning ops (channels, liquidez)
3. Partner custody (keys, freeze, compliance)

*“I’d never debug a mint failure with only LN pathfinding tools.”*

**Preguntas de custody (semana 1):**
1. Who holds end-user keys today?
2. Can anything freeze or reverse after preimage reveal?
3. What’s DR for a force-close?
4. System of record — your DB or chain proofs?

**PSP pitch (60s):** instant UX, fees configurables en USDT (vs spikes de BTC), custody boundary del partner, split API vs Cloud, honest RGB edge.

**Métricas de trial:** success rate **por clase de fallo**, p50/p95 invoice→settled, inbound headroom, webhook lag, force-close count.  
*“I’d ship a failure taxonomy in week 1.”*

---

## 7. STAR + fit (desde tu CV)

Scripts listos: [`11-casos-cv.md`](./11-casos-cv.md) · bullets: [`04-star-stories.md`](./04-star-stories.md).

**Memoriza hoy:** Caso 1 (transfers API) · Caso 2 (MPC) · Caso 4 (CUBO+ / LND honesto) · pitch.

LND en el CV = fellowship CUBO+, **no** producción en REALTOKN. No lo mezcles.

Slots que más importan:
1. Integración ambigua (API / partner / webhook)
2. Incidente / retries / idempotencia / reconciliación
3. Lo más cercano a payments / crypto / ledgers

Plantilla: Situation 1 frase · Task tuya · Action 2–4 pasos · Result con métrica o outcome + lesson. ~90s.

Puentes (solo si salen naturales):
- “…maps to Lightning **liquidity direction** because…”
- “…maps to **settlement reconciliation** because…”
- “…maps to **partner API + webhooks** because…”

### 4 preguntas (memoriza)

1. Is the seat Boosty staff-aug into Utexo, Utexo direct, or general Boosty Web3?
2. What does “strong after trial” look like in week 2 vs month 2?
3. Day-one stack: LN ops, RGB, backend API, or partner integrations?
4. Who is the tech expert and what product area do they own?

### Red flags (escucha, no debatas)

Sin criterios de keep · presión a fingir profundidad RGB/LN · unclear quién decide keep/fire.

---

## 8. Mock de 10 minutos (timer ON)

Usa esto **esta noche**. Respuestas en voz alta.

| Tiempo | Pregunta |
|---|---|
| 0:30–2:00 | What is Lightning, and why can a high-capacity node still fail send/receive? |
| 2:00–3:30 | BOLT11 fails no-route. Debug it in week one. |
| 3:30–5:00 | Why would a PSP care about Bitcoin-native USDT? Custody and fees? |
| 5:00–6:30 | What is RGB, and how does it fit Lightning in a USDT-on-Bitcoin story? |
| 6:30–8:00 | Time you shipped or stabilized a payments / API / reliability system. |
| 8:00–9:00 | First two weeks of a 2–3 month pasantía — how do you prove keep? |
| 9:00–10:00 | Tus 2–3 preguntas. |

**Si empujan más profundo:** revocation · failure codes (`invoice_expired`, `insufficient_outbound`, `no_route`, `peer_unavailable`, `htlc_timeout` + retryable) · jamming · webhook idempotente · cuándo force-close · RGB vs ERC-20 para un PSP institucional (3 criterios: trust model, tooling maturity, Lightning path) · SLOs de trial.

---

## 9. Never say / always say

**Never**
- “I built Utexo / I ran their mainnet LN.”
- “RGB is anonymous.” Di *different disclosure model*.
- “Swap is an AMM.” Di **HotPot intent/RFQ**.
- “Cloud API pays invoices.” Cloud **provisiona**; RLN **paga**.
- “I’ll just learn fast” sin plan.
- Inventar endpoints, OpenAPI, o el fee schedule completo.

**Always**
- Capacity ≠ liquidity.
- Classify failures.
- Ask what “final” means.
- Separate mint vs LN vs custody.
- State the RGB gap + week-1 verification plan.
- `@utexo/wdk-rgb-lightning` for new Node; archived `@utexo/rgb-sdk` is dead.

---

## 10. Checklist de esta noche (marca al hablarlo)

- [ ] Lightning ≤90s sin ensalada de jerga
- [ ] Dibujo Alice/Bob + pago 50k
- [ ] Inbound vs outbound con merchant
- [ ] No-route en 45s
- [ ] Utexo 60s (4 productos)
- [ ] Paquete Node correcto + por qué no `rgb-sdk`
- [ ] Vanilla vs colored + `createUtxos`
- [ ] Blinded vs witness
- [ ] Cloud API vs RLN API
- [ ] RGB 60s + gap honesto
- [ ] Payment vs settlement
- [ ] 3 preguntas de custody
- [ ] 2 STAR reales escritas y dichas
- [ ] Trial pitch 60s
- [ ] 4 preguntas de cierre
- [ ] Mock 10 min cronometrado

---

*Fuentes: docs.utexo.com (crawl 2026-09-17) · este repo · no afiliado a Boosty/Utexo.*
