# Interview Prep — Complete study document

**Boosty Labs / Utexo tech-expert screen → pasantía**

Single file with all markdown study materials. HTML decks stay separate (`study-deck*.html`).

**How to use:** Start **Part H** (Utexo products/SDK). Use A–G for Lightning depth and mocks. Part I is the full docs reference.

**Live decks:** https://ricardofv2.github.io/interview-prep-deck/

---

## Table of contents

1. [Part A — Lightning foundations](#part-a---lightning-foundations)
2. [Part B — BTC ↔ USDT settlement](#part-b---btc--usdt-settlement)
3. [Part C — RGB plain language](#part-c---rgb-plain-language)
4. [Part D — STAR stories + trial fit](#part-d---star-stories-+-trial-fit)
5. [Part E — 10-minute mock screen](#part-e---10-minute-mock-screen)
6. [Part F — Worked examples](#part-f---worked-examples)
7. [Part G — Advanced tech-expert depth](#part-g---advanced-tech-expert-depth)
8. [Part H — Utexo product suite + SDK (priority)](#part-h---utexo-product-suite-+-sdk-(priority))
9. [Part I — Full Utexo docs crawl digest](#part-i---full-utexo-docs-crawl-digest)


---

# Part A — Lightning foundations

*Source file: `01-lightning-day1.md`*

## Day 1 — Lightning Network (tech-expert screen)

**Goal:** Explain Lightning clearly in 60–90 seconds, then handle one follow-up on liquidity or failure modes.  
**Context:** Utexo’s Cloud product is framed as managed Lightning infra for Bitcoin-native USDT settlement. You don’t need to be an LN researcher — you need crisp production intuition.

**Timebox:** 45–60 min study + 15 min spoken practice.

---

### 1. One-sentence model

Lightning is a **layer-2 payment network** on Bitcoin: parties lock BTC in **payment channels** (on-chain 2-of-2 multisig), then send many **off-chain** payments by updating channel balances, settling to chain only when needed (open/close/dispute).

---

### 2. Core concepts (must-know)

#### Payment channel
- Two nodes fund a shared UTXO (channel capacity).
- Each side has a **local balance** (outbound liquidity) and sees the peer’s balance (inbound).
- Payments update signed commitment transactions **without** broadcasting every payment on-chain.

#### Capacity vs liquidity
- **Capacity** = total BTC locked in the channel.
- **Liquidity** = how that capacity is split (can I *send* vs can I *receive*?).
- Classic interview trap: “big channel” ≠ “can pay anyone.” Direction of funds matters.

#### Routing / multi-hop
- If A has no direct channel to C, payment can go A→B→C if intermediate nodes have the right liquidity.
- Relies on **HTLCs** (Hash Time-Locked Contracts): atomic forward-or-refund with a secret (preimage) and timeouts.
- Pathfinding uses public graph info + fees + probabilities; real networks also hit **liquidity uncertainty**.

#### Invoice (BOLT11)
- Receiver creates an invoice: amount (or open), payment hash, expiry, routing hints, memo.
- Payer pays the invoice; when receiver reveals the **preimage**, the payment settles along the path.
- **BOLT11** = standard invoice format most wallets/nodes speak.

#### Fees
- Routing nodes charge fees (base + proportional) for forwarding.
- Fee policy affects path selection; cheap ≠ reliable.

#### Force-close / justice
- Cooperative close: mutual agreement, efficient on-chain settle.
- Force-close: one side broadcasts latest commitment; timelocks + penalty (justice) txs protect against broadcasting an old state.
- Interview point: Lightning security assumes you **watch the chain** (or use a watchtower) so the peer can’t cheat with an outdated state.

#### LSP (Lightning Service Provider)
- Helps wallets that shouldn’t run a full always-online node: channel opens, inbound liquidity, sometimes invoice/JIT channels, backups.
- Utexo’s public Cloud framing (“managed Lightning infra”) maps to this class of problem: partners want payments without becoming LN ops experts.

---

### 3. Failure modes (what “senior” sounds like)

Practice naming **symptom → likely cause → what you’d check**:

| Symptom | Likely cause | What you’d check |
|---|---|---|
| Invoice unpaid / “no route” | Missing liquidity on path, graph stale, amount too large | Pathfinding logs, channel balances along candidate routes, amount vs capacity |
| Payment stuck pending | HTLC in-flight, peer offline, slow settle | HTLC timeouts, peer connectivity, force-close risk window |
| Can send but can’t receive | No **inbound** liquidity | Rebalance, loop-out/in, LSP inbound, dual-funded channels |
| High fees / weird routes | Fee policies, sparse graph, large amount | Fee floor, split payments (MPP if supported), direct channel |
| Force-close surprises | Offline node, ignored watchtower, old software | Always-online policy, watchtower, upgrade path |

**Phrase for interview:** “I’d separate routing failure from liquidity failure from peer liveness — same user symptom, different fixes.”

---

### 4. Map to Utexo / Boosty (use lightly)

Say only what public product claims support:
- Instant payments → Lightning speed vs on-chain Bitcoin.
- Managed Cloud → someone runs LN ops (channels, liquidity, uptime) for partners (PSP, exchange, wallet, iGaming).
- USDT on Bitcoin / RGB is **next topic** — today stay on Lightning rails, then bridge: “settlement needs LN liquidity + asset layer (RGB) + clear custody model.”

If asked “have you run LND/CLN?”: honest map — closest ops (nodes, payments APIs, reconciliations, on-call). Don’t fake channel ops.

---

### 5. Glossário rápido (EN terms you’ll hear)

- **HTLC** — conditional payment hop (hash + time lock)
- **Preimage** — secret that settles the HTLC chain
- **Commitment tx** — latest signed channel state
- **Watchtower** — third party that watches for cheating closes
- **Dual-funding** — both sides fund the channel open
- **JIT channel** — channel opened just-in-time when paying an invoice (LSP pattern)
- **MPP** — multi-path payments (split amount across routes)
- **Gossip** — public channel announcements for pathfinding

---

### 6. Study checklist (45–60 min)

- [ ] Explain Lightning in ≤90 seconds without jargon pile-up
- [ ] Draw A↔B channel: capacity, local, remote; show a $10 payment moving balances
- [ ] Explain why A→C needs routing + HTLCs
- [ ] Explain BOLT11 invoice fields you’d care about in an API integration
- [ ] Explain inbound vs outbound liquidity with a merchant example
- [ ] Name 3 payment-failure causes and what you’d log/metric
- [ ] One sentence on LSP value for a PSP/wallet partner
- [ ] One honest sentence on your real experience vs learning plan for trial week 1

---

### 7. Spoken answers (models — adapt to your voice)

#### Q: What is Lightning in plain English?
**Model:** “It’s a Bitcoin layer-2 where we lock funds in channels and move balances off-chain with cryptographic updates. You get near-instant payments and lower fees; on-chain is for open, close, and disputes. Multi-hop uses HTLCs so the payment is atomic across the path.”

#### Q: Why can a node with lots of capacity still fail payments?
**Model:** “Capacity is the size of the pipe; liquidity is the direction. If my local balance is near zero, I can’t send. If my remote balance is near zero, I can’t receive. Routing fails when *some hop* lacks the right direction for that amount.”

#### Q: What does an LSP do?
**Model:** “It productizes Lightning ops: helps open channels, provide inbound liquidity, keep connectivity reliable, sometimes handle invoice UX. A PSP or wallet can integrate payments without becoming a full LN SRE team on day one.”

#### Q: How would you debug a failed BOLT11 payment in week one of a trial?
**Model:** “Reproduce with the payment hash; check invoice expiry and amount; see if failure is no-route vs insufficient balance vs peer timeout. Inspect local channels first, then path attempts and fee constraints. Confirm whether we’re sender, forwarding hop, or receiver — the checklist changes.”

---

### 8. Practice drill (do out loud, phone timer)

**Round A — explain (3 min)**  
1. Lightning one-liner + why it exists  
2. Channel balances after one payment  
3. BOLT11: what the payer and payee each need  

**Round B — pressure (5 min)**  
Answer each in ≤45 seconds:
1. Inbound vs outbound liquidity  
2. Why HTLCs matter for multi-hop  
3. Three reasons “no route”  
4. What an LSP sells to a wallet  
5. What you’d measure on a Lightning payments API in production  

**Round C — fit for trial (2 min)**  
“In the first two weeks of a pasantía on a Lightning-related product, I would…” (learning plan + how you’d show early fit)

**Self-score:** Clarity / Depth / Fit — each 1–5. Fix the lowest score first.

---

### 9. Stretch (only if Round A–C are clean)

- Dual-funded channels vs single-funded  
- Watchtowers in one minute  
- Fee market and why “cheapest route” can be fragile  
- How Lightning + an asset layer (RGB/USDT) changes the story (teaser for Day 2)

---

### 10. Day 2 preview (don’t start yet)

BTC↔USDT settlement narrative, custody vs non-custodial partner model, then RGB in plain language — tied to Utexo’s public API/Cloud framing.

---

*Interview Prep · Boosty / Utexo screen · Lightning Day 1*

### Extra examples
See `06-worked-examples.md` sections **A1–A6** (balances, no-route, BOLT11, LSP).
Do A1 on paper, then answer A5 out loud.


---

# Part B — BTC ↔ USDT settlement

*Source file: `02-settlement-day2.md`*

## Day 2 — BTC ↔ USDT settlement (tech-expert screen)

**Goal:** Explain why a PSP/exchange/wallet wants Bitcoin-native USDT rails, and what “settlement” means in product terms.  
**Prereq:** Day 1 Lightning (channels, liquidity, invoices).  
**Timebox:** 45 min study + 15 min spoken practice.

---

### 1. One-sentence product model (public Utexo framing)

Utexo’s public pitch: **move USDT instantly and privately on Bitcoin** — API for settlement with configurable fees, and Cloud as managed Lightning infra for stablecoin payments / BTC↔USDT flows. Targets: PSPs, exchanges, custodians, wallets, iGaming.

---

### 2. Why this exists (business intuition)

On-chain Bitcoin alone: secure, slow/expensive for small payments.  
Classic USDT: often on other chains (Tron, Ethereum, etc.) — different trust/ops/compliance footprint.  
**Interview narrative:** partners want *stablecoin UX* (USD denomination, predictable fees) with *Bitcoin settlement topology* (Lightning speed + Bitcoin security model). Say it as product motivation, not as a claim you audited their code.

---

### 3. Core concepts (must-know)

#### Settlement vs payment
- **Payment:** user-visible “send / receive now.”
- **Settlement:** when obligations between parties are considered final in the system of record they care about (merchant, PSP, exchange ledger).
- Interview phrase: “I’d ask what ‘final’ means for this partner — Lightning preimage, RGB state, or their internal ledger credit.”

#### Fees in USDT (product claim)
- Public framing: configurable / predictable fees denominated in USDT (vs surprise BTC fee spikes only).
- What to discuss: fee transparency for PSPs; who pays (sender vs merchant); failure + refund fee behavior.

#### Custody models
- **Custodial:** partner holds user funds / keys → simpler UX, heavier compliance + trust.
- **Non-custodial / no custody rewrite:** partner keeps existing custody; integration is rails + API (Utexo public messaging leans this direction for partners — verify in interview, don’t assert internals).
- Always ask: who holds keys, who can freeze, what’s the recovery path.

#### ICP checklist (from their site categories)
For each, one sentence on *their* pain:
| Segment | Typical pain | What Lightning-style rails sell |
|---|---|---|
| PSP | Speed, fee predictability, payout UX | Instant settle, API webhooks |
| Exchange | Deposit/withdraw UX, liquidity | Fast BTC/stable rails |
| Custodian | Ops + security boundaries | Clear custody boundary |
| Wallet | UX without full LN SRE | LSP/Cloud-like managed layer |
| iGaming | Fast deposits/withdrawals | Instant + fee control |

#### API vs Cloud (public split)
- **API:** integrate settlement / payments programmatically (invoices, webhooks, fee config — exact surface is in their docs; don’t memorize fake endpoints).
- **Cloud:** managed Lightning (and related) so the partner doesn’t run the full node/liquidity stack alone.

---

### 4. Failure & risk talk (sounds senior)

| Topic | Soundbite |
|---|---|
| Liquidity | Stablecoin UX still rides Lightning liquidity constraints for BTC legs / channels |
| Finality | Off-chain speed ≠ same finality story as deep on-chain confirmations — be precise |
| Reconciliation | Idempotent webhooks, payment hashes, ledger double-entry |
| Compliance | Travel rule / AML may sit with partner; Boosty’s public AML partnerships are adjacent context only |
| Bridge risk | If funds move across systems (mint/lock), trust assumptions change — Day 3 |

---

### 5. Study checklist

- [ ] 60s: Why Bitcoin-native USDT rails for a PSP?
- [ ] Define settlement vs payment with one example
- [ ] Custody: 3 clarifying questions you’d ask in week 1
- [ ] API vs Cloud: who is each for?
- [ ] Name 2 production metrics for a payments API (success rate, p95 settle time, reconciliation lag, fee drift)
- [ ] Honest map: closest fintech/payments/crypto work you’ve done

---

### 6. Spoken models

#### Q: Explain Utexo in one minute (public facts only)
**Model:** “Publicly, Utexo focuses on USDT payments on Bitcoin using Lightning for speed and an RGB-related asset layer for Bitcoin-native stablecoins. They pitch API plus Cloud to PSPs, exchanges, custodians, wallets, and iGaming — configurable fees and a partner-friendly custody story. Boosty Labs is publicly connected as engineering support / case study, with a shared founder.”

#### Q: What would you clarify in the first trial week?
**Model:** “Where finality is recorded, who holds custody, how webhooks are idempotent, how fees are computed on success vs failure, and what SLOs the tech expert cares about for keep-after-trial.”

---

### 7. Practice drill (out loud)

1. 60s product narrative (no buzzword salad)  
2. 45s: settlement vs payment  
3. 45s: custody questions  
4. 45s: PSP vs wallet buyer  
5. 90s: “How my background maps” (honest)

Self-score Clarity / Depth / Fit.

### Extra examples
See `06-worked-examples.md` sections **B1–B5** (PSP fees, settlement vs payment, custody Qs, metrics).
Do B2 + B5 out loud.


---

# Part C — RGB plain language

*Source file: `03-rgb-day3.md`*

## Day 3 — RGB in plain language (tech-expert screen)

**Goal:** Explain RGB at interview depth: what problem it solves on Bitcoin, how it differs from account-based USDT chains, and how it meets Lightning — without pretending you’ve shipped RGB production.  
**Prereq:** Days 1–2.  
**Timebox:** 45 min study + 15 min drill.

---

### 1. One-sentence model

**RGB** is a smart-contract / asset system for Bitcoin that keeps most contract data **off** the public global ledger and uses **client-side validation**: parties verify history relevant to their coins, while Bitcoin UTXOs anchor ownership.

Interview use: “Bitcoin-native assets with a privacy and scalability profile different from ERC-20-style global state.”

---

### 2. Why Utexo talks about RGB (public)

Public narrative ties USDT-on-Bitcoin to **RGB + Lightning** (seed PR and product: Bitcoin-native USDT settlement; team history includes ThunderStack / RGB+Lightning; Tether WDK community module `@utexo/wdk-rgb-lightning` is public beta).

You need the *why*, not bytecode details.

---

### 3. Concepts to sound competent

#### Client-side validation
- You don’t re-execute the whole world’s state like a global VM.
- You validate the **consignment** / history for the UTXOs you care about.
- Tradeoff: different UX and infra (share proofs, sync, backups) vs “just read a block explorer balance.”

#### Assets on UTXOs
- Ownership attaches to Bitcoin UTXOs; spending Bitcoin can move/allocate asset state under RGB rules.
- Mental contrast: Ethereum account balance vs Bitcoin UTXO coin selection + asset allocation.

#### Privacy angle (product language)
- Less of a single public “everyone’s USDT ledger” view than typical L1 token contracts.
- Don’t overclaim anonymity; say “different disclosure model” unless you know their exact guarantees.

#### RGB + Lightning
- Lightning moves value fast off-chain; RGB aims to carry **asset** semantics in that world (invoices / RGB-over-Lightning patterns in public WDK module docs: BOLT11 + RGB invoices, LSP/VSS, etc.).
- Interview bridge: “Lightning solves speed for BTC payments; RGB is how stablecoin asset state can live in a Bitcoin-centric design.”

#### Bridge / mint (handle carefully)
- Public community materials discuss bridges (e.g. lock-and-mint / TEE claims). Treat as **architecture under discussion**, not your personal production truth.
- Smart answer: “I’d separate issuance trust, bridge operators, and Lightning ops — different failure domains.”

---

### 4. Compare table (use in interview)

| | Typical USDT on account chain | RGB-style Bitcoin asset framing |
|---|---|---|
| State model | Global contract state | Client-side validated history |
| Explorer UX | Easy public balances | Different / more private disclosure |
| Fee token | Gas on that chain | Bitcoin fee market (+ LN fees) |
| Speed path | L1/L2 of that ecosystem | Lightning for instant payments |
| Integration | Familiar EVM tooling | Newer stack; docs/SDK matter |

---

### 5. Honest competence script

If you’re new to RGB:
“I haven’t shipped RGB to production. I understand client-side validation, UTXO-anchored assets, and why a Lightning+RGB story fits Bitcoin-native USDT. In trial week 1 I’d run their public docs/SDK paths, ship a thin integration or internal tool, and pair with whoever owns node/liquidity.”

That’s stronger than faking.

---

### 6. Study checklist

- [ ] 60s RGB explanation with one clear tradeoff  
- [ ] Contrast with ERC-20 USDT without dumping on either  
- [ ] How Lightning and RGB complement in the Utexo pitch  
- [ ] 3 questions you’d ask the tech expert about their stack  
- [ ] One risk domain: bridge vs LN ops vs custody  

---

### 7. Spoken models

#### Q: What is RGB?
**Model:** “It’s a Bitcoin-centric asset and contract approach using client-side validation: you verify the history for your coins instead of a single global contract state. Assets are tied to Bitcoin UTXOs. That enables different privacy and scaling tradeoffs versus account-based tokens.”

#### Q: How does that relate to Lightning?
**Model:** “Lightning gives instant BTC payment rails. RGB is about representing assets in a Bitcoin-native way. Together, the product story is stablecoin settlement with Lightning speed — which is what Utexo markets publicly via API/Cloud and RGB-Lightning tooling.”

#### Q: What would you learn first on the job?
**Model:** “Their invoice/settlement API, how they expose RGB+Lightning to partners, where custody sits, and how they monitor failed payments and liquidity. I’d start from public docs and a sandbox path if they have one.”

---

### 8. Practice drill

1. 60s RGB  
2. 45s vs ERC-20 USDT  
3. 45s Lightning + RGB combo  
4. 45s bridge risk domains  
5. 90s honest learning plan for pasantía week 1–2  

Self-score Clarity / Depth / Fit.

### Extra examples
See `06-worked-examples.md` sections **C1–C4** (ERC-20 contrast, LN+RGB, bridge domains, honest gap).
Memorize C4 in your own words.


---

# Part D — STAR stories + trial fit

*Source file: `04-star-stories.md`*

## STAR stories + trial fit (tech-expert screen)

**Goal:** 3–5 stories you can tell in ~90 seconds each, plus a 60s “why keep me after trial” pitch.  
**Timebox:** 30–45 min to draft; practice until smooth.

---

### STAR template (strict)

- **S**ituation — 1 sentence context  
- **T**ask — your responsibility  
- **A**ction — 2–4 concrete steps *you* did  
- **R**esult — metric or clear outcome + lesson  

Cut company lore. Prefer production pain, payments, APIs, reliability, crypto/fintech adjacency.

---

### Five slots to fill (write your bullets)

#### Story 1 — Ambiguous integration
Prompt: Shipped a payment, wallet, API, or partner integration under unclear requirements.  
Your notes:
- S:
- T:
- A:
- R:

#### Story 2 — Incident / reliability
Prompt: Outage, stuck payments, retries, idempotency, reconciliation bug.  
Your notes:
- S:
- T:
- A:
- R:

#### Story 3 — Explained hard tech
Prompt: Made a complex system understandable (or killed a bad design).  
Your notes:
- S:
- T:
- A:
- R:

#### Story 4 — Short engagement / proving fit fast
Prompt: Contract, trial, sprint zero, or first 30 days where trust was earned early.  
Your notes:
- S:
- T:
- A:
- R:

#### Story 5 — Closest to Lightning / Bitcoin / stablecoins / fintech rails
Prompt: Even adjacent (ledgers, webhooks, custody, exchanges, on-chain txs).  
Your notes:
- S:
- T:
- A:
- R:

---

### Mapping lines (practice these bridges)

After each story, add one bridge if natural:
- “That maps to Lightning **liquidity/direction** thinking because…”
- “That maps to **settlement reconciliation** because…”
- “That maps to **partner API + webhooks** because…”

Don’t force it.

---

### Trial / pasantía pitch (60 seconds)

Structure:
1. Why this product problem interests you (Bitcoin-native USDT rails — specific).  
2. What you can contribute in weeks 1–2 (learning plan + shipping small).  
3. How you’ll show “keep” signal (reliability, communication, ownership).  
4. Honest gap + how you’ll close it.

Draft:
> …

---

### Questions you ask them (memorize 4)

1. Is the seat Boosty staff-aug into Utexo, Utexo direct, or general Boosty Web3?  
2. What does “strong after trial” look like in week 2 vs month 2?  
3. Day-one stack: LN ops, RGB, backend API, or partner integrations?  
4. Who is the tech expert and what product area do they own?

---

### Red flags (listen, don’t debate)

- No success criteria for the pasantía  
- Pressure to claim RGB/LN depth you don’t have  
- Unclear employer vs reviewer of keep/fire  

---

### Practice drill

- Tell Stories 1, 2, 5 out loud (90s each)  
- Deliver trial pitch once  
- Ask your 4 questions as if closing the screen  

Record yourself once if you can. Fix filler words and rambling Situations.

### Extra examples
See `06-worked-examples.md` section **D** for *fictional* STAR samples — adapt structure only.
Then run mocks **E1–E3**.


---

# Part E — 10-minute mock screen

*Source file: `05-mock-screen.md`*

## 10-minute mock tech-expert screen

**Use after Days 1–4.** Timer on. Answers out loud. Then paste answers here for scoring (Clarity / Depth / Fit).

---

### Script (interviewer side)

**0:00–0:30** Intro  
“Thanks for joining. We’re doing a short technical screen for a trial seat related to our Bitcoin / Lightning payments work.”

**0:30–2:00** Q1 — Lightning  
“In plain English, what is the Lightning Network, and why can a node with high channel capacity still fail to send or receive?”

**2:00–3:30** Q2 — Failure  
“A BOLT11 payment fails with no route. Walk me through how you’d debug it in your first week.”

**3:30–5:00** Q3 — Product  
“Why would a PSP care about Bitcoin-native USDT settlement? What would you clarify about custody and fees?”

**5:00–6:30** Q4 — RGB  
“What is RGB at a high level, and how does it fit with Lightning in a USDT-on-Bitcoin story?”

**6:30–8:00** Q5 — STAR  
“Tell me about a time you shipped or stabilized a payments / API / reliability-sensitive system.”

**8:00–9:00** Q6 — Trial fit  
“If you join for a 2–3 month pasantía, what would you do in the first two weeks to prove you’re a keep?”

**9:00–10:00** Your questions  
Ask 2–3 from your list.

---

### Scoring rubric (1–5 each)

| Q | Clarity | Depth | Fit | One fix |
|---|---|---|---|---|
| 1 Lightning | | | | |
| 2 Debug | | | | |
| 3 PSP/custody | | | | |
| 4 RGB | | | | |
| 5 STAR | | | | |
| 6 Trial plan | | | | |

**Pass bar for this screen:** no invented experience; Lightning liquidity correct; one crisp product question; honest RGB learning plan.

### Extra examples
Warm up with `06-worked-examples.md` mocks **E1–E3** (2 min each) before the full 10-min script.


---

# Part F — Worked examples

*Source file: `06-worked-examples.md`*

## Worked examples — tech-expert screen practice

**How to use:** Cover the answer first. Say your version out loud (45–90s). Then compare to the model. These are **practice scenarios**, not claims about your résumé — rewrite STAR ones with *your* real facts only.

---

### A. Lightning — numeric / picture examples

#### Example A1 — Channel balances after a payment
**Setup:** Alice ↔ Bob channel. Capacity = 1,000,000 sats.  
Before: Alice local 700,000 · Bob local 300,000.

Alice pays Bob **50,000 sats** (direct channel).

**After:**
- Alice local **650,000**
- Bob local **350,000**
- Capacity still **1,000,000**

**Interview line:** “Capacity didn’t change — only the split (liquidity direction) did.”

---

#### Example A2 — Why “big channel” still fails
**Setup:** Merchant M has a 5 BTC channel with LSP.  
M’s local balance ≈ 0 (customers paid M; funds sit on M’s side as **inbound for customers**, **outbound for M** wait — careful):

Clarify:
- To **receive** customer payments, M needs **inbound** liquidity (remote side has room / M’s local can grow).
- After many receives, M is “full” on the receive side relative to that channel → further receives fail until rebalance / new inbound / loop-out.

**Interview line:** “I’d ask which direction failed — send or receive — before touching pathfinding.”

---

#### Example A3 — Multi-hop with HTLC (storyboard)
Payee Carol creates BOLT11 invoice (payment hash H of preimage R).  
Alice pays via Bob:

1. Alice → Bob: HTLC locked with hash H + timeout  
2. Bob → Carol: HTLC with same hash H + tighter timeout  
3. Carol reveals R; Bob learns R; Alice learns R  
4. Balances update; if anyone stops, timeouts refund

**Interview line:** “Atomic across hops — either the whole path settles with the preimage, or it times out and unwinds.”

---

#### Example A4 — BOLT11 fields you’d care about in an API
Imagine partner payload:
- `amount_msat`: 250000000 (0.0025 BTC)  
- `payment_hash`: `ab12…`  
- `expiry`: 600 seconds  
- `description`: `order-98421`  
- routing hints: private channel to LSP  

**Debug checklist if unpaid:**
1. Clock skew / expiry already passed?  
2. Amount > any hop liquidity?  
3. Private channel / missing hint?  
4. Fee budget too low for path?

---

#### Example A5 — “No route” postmortem (mini case)
**Symptom:** 2.1M sat payment fails no-route.  
**Facts:** Sender has 3M outbound on one channel; graph shows path but middle hop only has 1.5M in the needed direction.

**Root cause:** Liquidity, not “Lightning is down.”  
**Fix options:** Smaller amount / MPP if supported / different peer / wait for rebalance / open channel closer to destination.

**Model answer (45s):**  
“I’d confirm amount vs local outbound, then inspect attempted routes and the bottleneck hop’s capacity in the send direction. No-route often means liquidity topology, not a bug in the invoice.”

---

#### Example A6 — LSP pitch (wallet partner)
**Scenario:** Mobile wallet doesn’t want users running always-online nodes.  
**LSP offers:** channel open UX, inbound liquidity, maybe JIT channels, connectivity.  
**Tradeoff:** trust/dependency on LSP policy and uptime; fees for liquidity.

**Map to Utexo (public framing only):** Cloud ≈ “don’t make every PSP become LN SRE.”

---

### B. Settlement / product examples (Utexo-shaped)

#### Example B1 — PSP wants predictable fees
**Partner:** LatAm PSP paying merchants in USDT-equivalent.  
**Pain:** BTC fee spikes make quote→settle messy; slow L1 UX kills conversion.  
**Pitch (public product language):** instant rails + fees configurable in USDT terms + API/webhooks for their ledger.

**Question you’d ask them:**  
“On success vs failure, who is charged, and what event is idempotent for credit?”

---

#### Example B2 — Settlement vs payment (concrete)
**Payment:** Player in iGaming sees “deposit confirmed” in app UI in ~1s.  
**Settlement:** PSP’s ledger marks merchant payable after webhook `payment_settled` with payment hash / id; finance reconciles nightly.

**Interview line:** “UI speed and ledger finality can differ — I’d align definitions in week 1.”

---

#### Example B3 — Custody clarifying questions (role-play)
Interviewer: “We integrate without rewriting custody.”  
You ask:
1. Who holds end-user keys today?  
2. Can your system freeze or reverse after preimage reveal?  
3. What’s the disaster recovery for channel force-close?  
4. Where is the system of record — your DB or chain proofs?

---

#### Example B4 — API vs Cloud buyer
| Buyer | Likely product | Why |
|---|---|---|
| Exchange with LN team | API-heavy | They already run infra |
| Wallet startup, 3 eng | Cloud | Need managed liquidity/uptime |
| PSP compliance-heavy | API + clear custody boundary | Audit story matters |

---

#### Example B5 — Metrics you’d put on a dashboard
- Payment success rate (by failure class: no-route, expired, peer, reject)  
- p50 / p95 time invoice → settled  
- Inbound liquidity headroom (sats / %)  
- Webhook delivery success + retry lag  
- Fee revenue vs routing cost (if you operate nodes)

**Trial signal:** “I’d ship a failure taxonomy in week 1 so we stop saying ‘Lightning failed.’”

---

### C. RGB examples (plain language)

#### Example C1 — ERC-20 USDT vs RGB-style framing
**ERC-20 mental model:** one contract, global balances, explorers show everyone’s token balance.  
**RGB mental model:** asset history validated client-side for *your* coins; anchored to Bitcoin UTXOs; less “world-readable token ledger.”

**One tradeoff each:**
- ERC-20: familiar tooling; public state  
- RGB: Bitcoin-centric + different privacy/scaling; newer ops/UX

---

#### Example C2 — Lightning + RGB combo sentence
“Lightning moves value quickly off-chain; RGB is how a stablecoin asset can be represented in a Bitcoin-native design. The product story is USDT UX with Bitcoin/Lightning settlement topology.”

---

#### Example C3 — Bridge risk domains (don’t mix them)
**Scenario:** User brings USDT from another system onto Bitcoin-native representation.  
Separate:
1. **Issuance / bridge trust** (who locks/mints, TEE claims if any)  
2. **Lightning ops** (channels, liquidity, invoices)  
3. **Partner custody** (keys, freezes, compliance)

**Interview line:** “I’d never debug a bridge mint failure with only LN pathfinding tools.”

---

#### Example C4 — Honest gap answer (use this pattern)
“I haven’t shipped RGB to production. I can explain client-side validation and why it pairs with Lightning for USDT-on-Bitcoin. In the first two weeks I’d complete your public docs/SDK path, write a thin integration or internal checklist, and pair with whoever owns node and liquidity.”

---

### D. Sample STAR answers (FICTIONAL — replace with your life)

> **Label every sample as practice.** Do not present these as your experience in the real interview.

#### Sample D1 — Ambiguous integration (fictional)
**S:** Fintech startup needed a partner payout webhook with unclear “paid” definition.  
**T:** Own the API contract + ledger write.  
**A:** Wrote sequence diagram; defined idempotency key; staged with replay tests; added dead-letter queue.  
**R:** Cut duplicate credits to near-zero; partner go-live in 2 weeks.  
**Bridge:** Same discipline as Lightning payment hashes + webhook settlement.

#### Sample D2 — Incident (fictional)
**S:** Spike of “pending” payments for 40 minutes.  
**T:** Incident lead.  
**A:** Separated provider timeout vs our DB lock; drained stuck jobs; added metric on age of pending.  
**R:** Restored p95 under SLO; postmortem → timeout budget change.  
**Bridge:** Like stuck HTLCs — classify failure before restarting everything.

#### Sample D3 — Short trial proving fit (fictional)
**S:** 8-week contract on payments API.  
**T:** Be keep-worthy by week 2.  
**A:** Shipped one visible dashboard + fixed top failure class; daily written update to tech lead.  
**R:** Converted to longer contract.  
**Bridge:** Matches pasantía “early fit signal” process.

---

### E. Mini mock scripts (2 minutes each)

#### Mock E1 — Liquidity trap
**Q:** “Our merchant channel is 10M sats. Receives started failing. Why?”  
**Good:** Talk inbound saturation / need to spend or rebalance / LSP inbound — not “channel too small.”  
**Bad:** “Bitcoin is congested” with no liquidity mention.

#### Mock E2 — PSP screen
**Q:** “Sell me Bitcoin-native USDT in 60 seconds.”  
**Good:** Instant UX, fee predictability, partner custody boundary, API/Cloud split, honest RGB learning edge.  
**Bad:** Token price talk, hype, fake logos.

#### Mock E3 — Trial plan
**Q:** “First two weeks?”  
**Good structure:**  
Day 1–2: env, docs, who owns LN vs API  
Day 3–5: one vertical slice (invoice → webhook → ledger)  
Week 2: failure taxonomy + one reliability fix  
**Show keep signal:** written notes, ask early, ship small.

---

### F. Generate-your-own (fill blanks)

1. Draw a channel: capacity ____ ; local ____ ; after paying ____ → new local ____  
2. Partner type: ____ ; their pain: ____ ; question I’d ask: ____  
3. Failure class I’ll own in trial: ____ ; metric: ____  
4. My real STAR for payments/reliability (not sample):  
   S: ____ T: ____ A: ____ R: ____  

---

*Interview Prep · worked examples · Boosty / Utexo screen*


---

# Part G — Advanced tech-expert depth

*Source file: `07-advanced-tech.md`*

## Advanced layer — tech-expert depth

**Why this exists:** Days 1–3 get you fluent. This file is what a tech expert probes next: protocol mechanics, adversarial thinking, production systems, and design tradeoffs.

**Rules**
- Prefer precise vocabulary + tradeoffs over buzzwords.
- If you don’t know, say how you’d verify (spec, logs, experiment) — don’t bluff RGB internals.
- Map answers to *payments systems* (idempotency, finality, liquidity, custody), not coin price talk.

**Timebox:** 2–3 focused sessions. Speak answers out loud.

---

### 1. Lightning — protocol depth

#### 1.1 Commitment transactions & revocation (must sound real)
- Each channel update creates a new **commitment transaction** pair (yours + theirs), pre-signed, not broadcast.
- Old states are made dangerous via **revocation**: sharing a revocation secret for the previous commitment so the peer can punish (justice tx) if you broadcast an outdated state.
- **Interview point:** Lightning’s security assumption is *you (or a watchtower) notice a revoked state on-chain in time*.

**Practice answer (60s):**  
“Channel balances aren’t a database row — they’re encoded in the latest mutual commitment txs. When we update, we revoke the prior state. If I broadcast an old fat state, my peer can claim a penalty using the revocation secret. That’s why offline risk and watchtowers matter.”

#### 1.2 HTLC lifecycle (more than “hash + time”)
Know the stages: add → fulfill (preimage) / fail → remove.  
Timeouts are **directional** along the path (upstream usually has more time than downstream).  
**Cltv expiry delta** mismanagement → stuck funds or unsafe routing.

**Hard question:** “What happens if an intermediate hop goes offline while HTLCs are in-flight?”  
**Good:** Funds locked until timeout; sender/receiver experience pending; force-close may be needed to resolve on-chain; operational cost and liquidity lockup.

#### 1.3 Fee market & pathfinding
- Nodes advertise base fee + proportional fee + CLTV delta + HTLC min/max.
- Pathfinding is probabilistic under **liquidity uncertainty** (you don’t know remote balances).
- **MPP** (multi-path): split large payments; partial failure complexity.
- Cheap route ≠ reliable route.

#### 1.4 Channel jamming (adversarial)
Attacker locks liquidity with HTLCs that never settle (or slow-fail), reducing useful capacity.  
Mitigations discussed in ecosystem: reputation, upfront fees, liquidity ads, better failure attribution — know it’s an open ops/security topic, not “solved.”

#### 1.5 Dual-funding, splicing, JIT (product relevance)
- **Dual-funding:** both sides fund open — better initial balance symmetry.
- **Splicing:** adjust channel capacity without full close/reopen (implementation maturity varies).
- **JIT channels / LSPs:** open liquidity when an invoice arrives — UX win, trust/dependency on LSP.

#### 1.6 Force-close economics
Cooperative close is cheap; force-close burns fees + timelocks + degraded UX.  
**Interview line:** “I’d treat force-close as an incident class with runbooks, not a normal control flow.”

---

### 2. Production engineering for Lightning payments

#### 2.1 Failure taxonomy (build this in week 1 of a trial)
| Class | Examples | Owner intuition |
|---|---|---|
| Invoice | expired, amount mismatch, bad hints | API validation |
| Local liquidity | insufficient outbound | Treasury / rebalance |
| Path / remote liquidity | no-route, temporary channel fail | Graph + peers |
| Peer liveness | disconnect mid-HTLC | Ops / SRE |
| Policy | fee insufficient, HTLC size limits | Config |
| Chain | force-close, sticky unresolved | Incident |

**Metric set:** success rate by class, p50/p95 invoice→settled, pending age histogram, inbound headroom, webhook lag, force-close count.

#### 2.2 Idempotency & reconciliation
- **Payment hash** (and your internal payment id) as idempotency keys.
- Webhooks can duplicate — at-least-once delivery assumed.
- Ledger rule: credit once per settled payment; store preimage/proof; reconcile nightly vs node.

**Hard question:** “User says paid; merchant says unpaid.”  
Walk: invoice state → node HTLC/payment state → webhook delivery log → partner ledger entry → whether preimage exists.

#### 2.3 Custody & key management boundaries
Separate: hot node keys (LN) vs cold/treasury vs partner custody.  
Compromise of node seed ≠ same blast radius as partner exchange cold wallet — say so explicitly.

---

### 3. System design prompts (practice whiteboarding)

#### Prompt A — PSP USDT payout API
Design an API for a PSP to pay merchants in USDT-on-Bitcoin rails.

Cover:
1. Auth (mTLS / API keys / HMAC)
2. CreatePayment / CreateInvoice resources
3. States: `created → pending → settled | failed | expired`
4. Webhooks + idempotency keys
5. Fee quote vs final fee
6. Liquidity reservation?
7. Observability + replay tooling
8. Abuse: amount limits, velocity, allowlists

**Strong close:** “Finality for the PSP ledger is an explicit policy object — not implied by UI speed.”

#### Prompt B — Liquidity ops for Cloud/LSP-like product
How do you keep inbound liquidity for thousands of wallets/merchants?

Talk: pool management, rebalancing (circular / submarine swaps if relevant), pricing liquidity, alerting on headroom, per-tenant isolation vs shared pools, jamming resistance.

#### Prompt C — Bridge + LN (careful)
If assets move from another system onto Bitcoin-native representation, separate trust domains:
1. Issuer/bridge (lock/mint, operators, TEEs if claimed)
2. LN ops
3. Partner custody / compliance

Never debug mint failures with only pathfinding tools.

---

### 4. RGB — interview-depth (still honest)

#### 4.1 What to own intellectually
- **Client-side validation:** verify consignments for UTXOs you care about; no single global token VM state like ERC-20.
- **UTXO anchoring:** asset allocations move with Bitcoin coin selection constraints.
- **Privacy/disclosure tradeoff:** different from public explorers; UX and backup story harder.
- **RGB + Lightning:** asset-aware invoices / LN transport for assets — newer stack; beta tooling exists in ecosystem (e.g. public WDK modules) — treat maturity as a question, not a boast.

#### 4.2 Hard questions + good shapes
**Q:** “How do you backup an RGB wallet differently from an LN-only wallet?”  
**Shape:** seeds + channel backups + RGB state/consignment availability; losing client-side data can be worse than LN alone.

**Q:** “What breaks if two systems disagree on asset state?”  
**Shape:** consignment completeness, reorg assumptions, bridge finality vs LN pending — define source of truth per layer.

**Q:** “Compare RGB USDT to Liquid USDT / other Bitcoin-adjacent stables.”  
**Shape:** trust model (federation vs client-side), tooling maturity, Lightning integration path, compliance optics — pick criteria, don’t fanboy.

#### 4.3 Honest competence script (keep)
State gap → show mental model → week-1 verification plan on their docs/SDK → who you’ll pair with.

---

### 5. Security & privacy questions

- Invoice metadata leakage (description, amount correlation)
- Payment probing to discover balances
- Watchtower privacy vs security
- Supply-chain risk on node software / LSP
- “Private” claims for USDT-on-Bitcoin — ask *what* is hidden from whom

**Line:** “I’d ask for a threat model doc before echoing marketing privacy language.”

---

### 6. Hard mock questions (answer ≤90s each)

1. Explain revocation without hand-waving.  
2. Design failure codes for a Lightning payments API.  
3. How would you detect channel jamming?  
4. Merchant receive failures with large capacity — diagnose.  
5. Idempotent webhook for `payment.settled`.  
6. When is force-close acceptable?  
7. Fee bump / stuck commitment interaction at a high level.  
8. RGB vs ERC-20 for institutional PSP — 3 decision criteria.  
9. What SLOs would you propose for a trial keep-decision?  
10. Walk a disputed payment reconciliation end-to-end.

#### Model sketches (short)

**(2) Failure codes:** `invoice_expired`, `amount_invalid`, `insufficient_outbound`, `no_route`, `peer_unavailable`, `fee_insufficient`, `htlc_timeout`, `policy_rejected`, `internal` — plus retryable boolean.

**(5) Webhook:** partner sends `Idempotency-Key` or you key by `payment_hash`; handler is transactional insert-if-not-exists; return 200 on duplicates; signed payloads; outbox for retries.

**(9) Trial SLOs:** payment success rate by class, p95 settle time, zero silent double-credits, runbook for pending>N minutes, weekly liquidity headroom report.

---

### 7. What “too simple” vs “tech-expert” sounds like

| Topic | Too simple | Tech-expert |
|---|---|---|
| Lightning | “It’s fast Bitcoin” | Commitments, revocation, liquidity direction, HTLC timeouts |
| Failure | “No route, try again” | Classified failures + metrics + playbooks |
| Product | “USDT on Bitcoin” | Finality policy, custody boundary, fee quote vs settle |
| RGB | “It’s private tokens” | Client-side validation + backup/ops tradeoffs + honesty on maturity |
| Trial | “I’ll learn fast” | Week-1 taxonomy + vertical slice + keep signals |

---

### 8. Drill plan (replace shallow review)

**Session A (60–90m)** — Protocol: §§1.1–1.3 out loud + Q1, Q4, Q6  
**Session B (60–90m)** — Production: §2 + design Prompt A outline on paper  
**Session C (45m)** — RGB/security §§4–5 + Q8, Q10  
**Session D (30m)** — Hard mock §6 timed

Score Clarity / Depth / Rigor (not vibes).

---

### 9. Fill with *your* depth

Closest systems you’ve touched (exchanges, webhooks, ledgers, nodes, L2s):  
_________________________________________________________________

Incidents you’ve owned:  
_________________________________________________________________

Gaps you’ll admit in interview:  
_________________________________________________________________

Week-1 learning plan you’ll actually do:  
_________________________________________________________________

---

*Interview Prep · advanced layer · Boosty / Utexo-oriented*


---

# Part H — Utexo product suite + SDK (priority)

*Source file: `08-utexo-products-sdk.md`*

## Utexo product suite + SDK — interview study pack

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

### 1. Product suite map (memorize)

From Product Suite docs — composable stablecoin-native primitives on Bitcoin:

| Product | One-liner (docs) | Interview angle |
|---|---|---|
| **SDK** | REST/client libs for native USDT transfers + RGB ops on Bitcoin | App integration surface; non-custodial client-side validation |
| **Cloud** | Managed RGB-enabled Lightning nodes (no self-host) | Control plane: lifecycle, health, backup/restore, VSS |
| **Mint** | Cross-chain USDT → native Bitcoin **RGB USDT** (EVM / non-EVM) | Liquidity gateway; not wrapped/synthetic per docs |
| **Swap** | Non-custodial BTC ↔ USDT with instant finality + LP | Intent/RFQ model (HotPot); resolvers; atomic settle |

**Line for interview:** “Each component is a layer — settlement/execution (SDK+LN+RGB), infra (Cloud/RLN), inbound liquidity (Mint), rebalancing/FX (Swap) — composable via one stack.”

---

### 2. Architecture stack (from Architecture page)

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

### 3. SDK family — critical (easy to fail the interview here)

| Package | Platform | Status / use |
|---|---|---|
| `@utexo/wdk-rgb-lightning` | Node.js & Bare | **Current Node path** — channels, invoices, payments, LSP, VSS (pre-1.0 beta) |
| `@utexo/wdk-wallet-rgb` | Node.js & Bare | Stable — on-chain RGB issuance/inventory (separate `dataDir`) |
| `@utexo/rgb-sdk-web` | Browser | WASM RLN + RGB/LN |
| `@utexo/rgb-sdk-rn` | React Native | On-device RLN |
| `@utexo/rgb-sdk` | Node (archived Jul 28, 2026) | **Do not use for new integrations** |

**Trap:** Copying Web/RN `UTEXOWallet` (`init`/`unlock`/`onchainSend`) into WDK code — wrong.  
**Trap:** Using archived `@utexo/rgb-sdk` `initialize()`/`send()` for a new Node project.

#### When WDK vs app SDKs
- **New Node server / wallet infra:** `@utexo/wdk-rgb-lightning` (+ `wdk-wallet-rgb` for issuance)
- **Web/RN app from scratch:** `rgb-sdk-web` / `rgb-sdk-rn`
- WDK: lower-level account/manager, external signer (mnemonic in host; VLS in-process for channel crypto)

---

### 4. Node path deep dive — `@utexo/wdk-rgb-lightning`

#### Install (docs)
```bash
npm install @utexo/wdk-rgb-lightning
npm install @utexo/rgb-lightning-node-nodejs   # Node binding
```

#### Lifecycle you must narrate
1. `new WalletManagerRgbLightning(mnemonic, { network, dataDir, ... })`
2. `account = await manager.getAccount(0)` — single account
3. `await account.unlock({ indexer_url OR bitcoind_rpc_*, proxy_endpoint, announce_addresses, announce_alias })`
4. Ops: peers → channels → invoices/payments / RGB
5. `manager.dispose()`

**Unlock rule:** exactly one chain backend — `indexer_url` **or** all `bitcoind_rpc_*` fields — not both.

#### Networks
Docs SDK overview table:
| Env | id | Notes |
|---|---|---|
| Mainnet | `mainnet` | prod |
| Testnet | `testnet` | |
| Utexo signet | `utexo` | default for many app SDK examples |

WDK examples also show `regtest` / explicit indexer+proxy at unlock.

#### Vanilla vs colored (SDK overview)
- **Vanilla:** normal BTC path — fees, funding (`getAddress()`)
- **Colored:** RGB allocations on UTXOs
- Before RGB issue/receive: fund vanilla → `createUtxos()` → refresh

#### RGB invoices
- **Blinded** (common): receiver `createRgbInvoice({ witness: false, ... })`; sender `transfer({ recipient, amount, token })`
- **Witness:** `witness: true`; sender must pass `witnessData.amountSats`
- Don’t pass `witnessData` on blinded — rejected

#### Lightning methods (WDK)
- `createInvoice` / `createLightningInvoice`
- `sendPayment`, `keysend`, `listPayments`, `getPayment`
- HODL: `createHodlInvoice`, `claimHodlInvoice`, `cancelHodlInvoice`
- Async/LSP: needs `lspBaseUrl` + `lspBearerToken`; inbound async operational; **outbound async in active development** (docs)

#### Channels / peers
- `connectPeer('pubkey@host:port')`, `openChannel({ capacity_sat, ... })`, optional `asset_id`/`asset_amount` for RGB channels
- Virtual channels / APay: `enableVirtualChannelsV0` + `virtualPeerPubkeys`

#### VSS
- Optional `vssUrl` — backup RLN KV state; **does not** replicate VLS signer DB → cross-device recovery with open channels limited (docs)

#### Errors to name
`UnlockError`, `AccountLockedError`, `VssNotConfiguredError`, `ApayError`, LSP timeout errors — branch on `err.name` / `err.code`.

#### Separate dataDir rule
`wdk-wallet-rgb` and `wdk-rgb-lightning` each need their **own** `dataDir` (exclusive `rgb-lib` lock; no shared asset records).

---

### 5. End-to-end flows to practice out loud

#### Flow A — On-chain RGB transfer (conceptual)
1. Fund vanilla address with test BTC  
2. `createUtxos`  
3. Receiver creates blinded RGB invoice  
4. Sender transfers with assetId + base units (`10 ** precision`)  
5. Both refresh; receiver verifies balance  
6. Pending until consignment + Bitcoin confirmations

#### Flow B — Lightning BTC payment (WDK)
1. Unlock node  
2. Connect peer + open channel (or use LSP path)  
3. Receiver `createInvoice({ amt_msat, expiry_sec })`  
4. Sender `sendPayment({ invoice })`  
5. Check `listPayments` / status  

#### Flow C — Product-level story (no code)
PSP wants USDT on Bitcoin: Mint (bring USDT to RGB) → SDK/Cloud execute payments on LN+RGB → Swap to rebalance BTC↔USDT → Cloud if they won’t self-host RLN.

---

### 6. Cloud / RLN (enough for interview)

- Self-hosted RGB Lightning Node **or** Utexo Cloud managed RLN  
- Cloud: create/upgrade/destroy, connect via mTLS or API token, webhooks, backup/restore  
- Remote signer / VLS keeps keys off node (security docs)  
- **Interview ask:** “For this role, is day-one work Cloud API, SDK embed, or Swap/Mint?”

---

### 7. Mint & Swap (product talking points)

**Mint:** USDT from EVM/Tron/(USDT0 path) → RGB USDT on Bitcoin; architecture describes orchestrator, connectors, TEE federation, RGB multisig mint, BTC relay SPV in enclave.  
**Swap:** HotPot intents — quote → intent → sign → resolver escrow → fulfill/refund; non-custodial, atomic.

Don’t memorize every Swap REST path unless the role is Swap-focused; know the **lifecycle**.

---

### 8. Likely tech-expert questions (docs-grounded)

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

### 9. Spoken model answers (short)

**Q: What is Utexo?**  
“A coordination/execution layer for Bitcoin-native USDT: RGB for assets, Lightning for speed, Bitcoin for finality — exposed as SDK, Cloud RLN, Mint, and Swap.”

**Q: Where do I start coding on Node?**  
“`@utexo/wdk-rgb-lightning` — archived `@utexo/rgb-sdk` is read-only since July 2026. Issuance via `@utexo/wdk-wallet-rgb` with a separate dataDir.”

**Q: How do RGB amounts work?**  
“Integer base units; one display unit is `10 ** precision`. Resolve `assetId` from `listAssets().nia`, not a hard-coded doc ID.”

---

### 10. Study plan (next 2–3 sessions)

**Session 1 (45m):** Product suite + architecture tables — redraw from memory.  
**Session 2 (60m):** SDK family + WDK lifecycle — narrate unlock → invoice → pay.  
**Session 3 (60m):** Read WDK reference sections you can’t recite (HODL, LSP, VSS) + answer §8 Qs timed.  
**Optional:** Skim Mint + Swap overview pages if interview leans payments FX.

**Practice rule:** Open docs only to verify — then close and speak.

---

### 11. Honest trial pitch tied to docs

“In week 1 I’d run the official quickstart for the package matching the stack (WDK vs Web/RN), hit signet/`utexo`, ship a thin vertical slice (RGB invoice or LN invoice + status), and write a failure/notes doc from real SDK errors — Unlock, locked account, pending RGB, no channel.”

---

*Sources: docs.utexo.com pages listed above · Interview Prep*

---

### 12. Docs-crawl deltas (extra interview ammo)

Pulled from full `docs.utexo.com` index crawl — use these; don’t invent more.

#### API surfaces (groups)
| Surface | Base / note |
|---|---|
| **Cloud control plane** | `https://cloud-api.thunderstack.org` — node CRUD/lifecycle, webhooks, logs (≠ RLN pay API) |
| **RLN REST** | On the node — issue/send/invoices/channels/peers (self-host or Cloud-connected) |
| **Mint gateway** | `https://transfer.gateway.dev.utexo.com/api/v0` — networks, estimate, bridge-in-signature, verify, history |
| **Swap** | Partner-gated base + `X-API-Key` — quote, intents, approvals, swaps, affiliates |
| **“RGB Node API”** in overview | **404 / unpublished** in crawl — don’t claim it |

#### Production caveats (high value in interview)
- **RLN:** docs say mainnet ≈ **on-chain RGB**; Lightning still **testnet/beta**
- **Mint:** stated commission **0.03%** (+ chain gas + BTC RGB fee); **BTC mainnet N/A** in crawled Mint notes; LN destination **WIP**
- **Swap marketing vs docs:** Suite page sounds LP/AMM-ish; Architecture/Swap = **HotPot intent/RFQ** — prefer the latter
- **Full fee schedule:** “deterministic fees” claimed; **complete schedule not published**
- **Cloud webhooks:** `X-Utexo-Signature`; Swap resolver: Ed25519 `X-Signature`

#### Narrate these E2E flows
1. RGB on-chain transfer (fund → createUtxos → receive invoice → send → refresh)  
2. RGB-over-Lightning (peer/channel → invoice → pay) — note mainnet LN limits  
3. Mint EVM→RGB and RGB→EVM  
4. Swap: health → quote → intent → approval → settle/refund  
5. Cloud: create RLN → RUNNING → connect → call node REST  

#### Extra hard Qs
- Cloud API vs RLN API — who provisions vs who pays?  
- Mint TEE threshold vs “no custodian” marketing — precise language  
- What’s WIP (LN dest, BTC mainnet mint, unpublished OpenAPI)?  

Full tables + cited snippets: `09-utexo-docs-digest.md`


---

# Part I — Full Utexo docs crawl digest

*Source file: `09-utexo-docs-digest.md`*

## Utexo public docs — interview prep digest

Crawl date: 2026-09-17 (America/El_Salvador). Primary source: https://docs.utexo.com (index: https://docs.utexo.com/llms.txt). Related: https://docs.wdk.tether.io (UTEXO WDK community modules). **Nothing below invents endpoints/methods; unknowns are marked.**

---

### Product suite map

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

### SDK deep dive

#### Package matrix (current)

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

#### Install (from docs)

```bash
## Current Node / Bare Lightning
npm install @utexo/wdk-rgb-lightning
npm install @utexo/rgb-lightning-node-nodejs   # or -bare

## On-chain WDK
npm install @utexo/wdk-wallet-rgb

## Web / RN
npm install @utexo/rgb-sdk-web
npm install @utexo/rgb-sdk-rn

## Legacy only (archived)
npm install @utexo/rgb-sdk
```

Sources: https://docs.utexo.com/product-suite/sdk.md , platform SDK pages, https://docs.wdk.tether.io/sdk/community-modules/wdk-rgb-lightning/guides/get-started/

#### Auth / keys / custody

- **SDK:** BIP-39 mnemonic (or seed); **never transmitted** to remote servers (stated for Web/SDK overview).
- Lightning node often **external-signer / VLS**: mnemonic in host secret manager; channel crypto in-process VLS (WDK Lightning).
- Web: `password` encrypts local RLN state; optional `vssUrl` for encrypted cloud backup.
- RN: `NativeExternalRLNSigner` or `PasswordRLNSigner`.
- Cloud: Bearer **Cloud API token** for control plane; node access via **mTLS** or Bearer token (separate credentials).
- Swap: `X-API-Key` on protected partner endpoints (base URL obtained from Utexo team; includes `/v1`).
- Mint: mostly public GETs; some POSTs use signature over fixed message `Bridge Authentication Proof`.

#### Networks / env (SDK overview)

| Identifier | RGB transport (example) | Indexer (example) |
| --- | --- | --- |
| `mainnet` | `rpcs://rgb-proxy-mainnet.utexo.com/json-rpc` | `ssl://electrum.iriswallet.com:50003` |
| `testnet` | `rpcs://rgb-proxy-testnet3.utexo.com/json-rpc` | `ssl://electrum.iriswallet.com:50013` |
| `utexo` (signet, default for dev) | `rpcs://rgb-proxy.utexo.com/json-rpc` | `https://esplora-api.utexo.com` |

Web defaults also list LN gateway `wss://ln-gateway-signet.utexo.com` for `utexo`. Faucet: Telegram `@Utexo_RLN_bot`.

#### Core types / methods (evidenced)

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

#### Example flows (adapted from public docs — cite URLs)

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

#### Error handling / config notes

- Branch on typed errors (`err.name` / `err.code`) for WDK Lightning.
- Amounts: integer **base units** (`10 ** precision`); do not assume `1` = one display unit.
- Vanilla vs colored derivation paths; fund vanilla → `createUtxos` before RGB.
- Lightning needs peers/channels (or LSP); init alone is not enough.
- Async payments (APay): inbound operational; outbound “in active development” (SDK overview).

---

### Key API / Cloud surfaces

Utexo has **no single global API** (API Reference Overview).

#### 1) Utexo Cloud API  
Base: `https://cloud-api.thunderstack.org` — Bearer token.

| Group | Endpoints (evidenced) |
| --- | --- |
| Nodes | `GET/POST /api/nodes`, `GET /api/nodes/{id}`, `DELETE /api/nodes`, `POST …/start|stop|upgrade|settings`, `GET …/latest-rln-image` |
| Webhooks | `GET /api/webhook-public-key`; node `settings.webhookUrl` |
| Logs | `POST/GET /api/nodes/{id}/logs` |

#### 2) RGB Lightning Node runtime API  
Cloud node URL or self-hosted `:daemon-listening-port` (default 3001). Auth: mTLS / Bearer / Biscuit (self-hosted).

Endpoint **groups** (all POST unless noted): wallet/balances (`/address`, `/btcbalance`, `/createutxos`, …), assets (`/issueassetnia`, `/sendrgb`, …), payments (`/lninvoice`, `/rgbinvoice`, `/sendpayment`, …), channels/peers, swaps (`/makerinit`, `/taker`, …), lifecycle (`/init`, `/unlock`, `/backup`, …). Full OpenAPI: https://utexo-protocol.github.io/rgb-lightning-node

#### 3) Mint API  
Base (test/dev): `https://transfer.gateway.dev.utexo.com/api/v0` — interactive docs under `/docs/`. **Bitcoin mainnet not available yet.**

| Group | Endpoints |
| --- | --- |
| Networks | `GET /networks`, `GET /networks/{id}/supported-tokens`, `GET …/balance/…` |
| Transfers | `GET /transfers/estimate/…`, `POST /transfers/bridge-in-signature`, `POST /transfers/verify-bridge-in`, `POST /transfers/submit-transaction`, `GET /transfers/history/…`, `GET /transfers/invoice/…` |

RGB Lightning destinations (`networkId` 94/95): **API surface present; “not available for use yet.”** Fee rate stated on Mint product page: **0.03%**.

#### 4) Swap (partner) API  
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

#### 5) Swap resolver Protocol API (separate)  
Resolver webhooks: `X-Signature` (Ed25519) + `X-Timestamp`; events `IntentAssigned`, `DepositConfirmed`, `WithdrawReady`, `SwapConfirmed`, `RefundConfirmed`; reporting via `POST /v1/intents/{id}/deposit|fulfill` etc. **Many gaps** documented on Validation Gaps page.

#### 6) “RGB Node API”  
Listed in API overview; **no published page found** (404 on guessed path). Treat as unknown / unpublished.

---

### End-to-end flows an interviewee should narrate

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

### Prerequisites the docs assume

- **Bitcoin UTXO model**; fees in sats; PSBT / Taproot awareness for advanced flows.
- **Lightning:** channels, BOLT11, HTLCs, peers, liquidity, LSP; LDK as RLN base.
- **RGB:** client-side validation; consignments; blinded vs witness invoices; single-use seals; NIA/IFA/CFA/UDA schemas; vanilla vs colored paths.
- **Cross-chain:** EVM contracts, Permit2, LayerZero/USDT0, Tron TIP-712, Solana versioned txs.
- **Ops:** BIP-39 mnemonics; encrypted backups vs VSS; TEE/Nitro concepts for Mint.

---

### Architecture / custody / fees / webhooks (extracted)

#### Architecture (textual diagram)

```text
Application
  ├─ RGB Lightning Node API ──► RLN ──► Bitcoin + RGB indexer + RGB proxy + LN peers (testnet)
  └─ Cloud API ───────────────► Utexo Cloud control plane ──► managed RLN instances
```

Stack table: Bitcoin / Lightning / RGB / Utexo / Mint / Swap (Architecture page). Claims ~**200 ms** payment latency in marketing-style architecture copy.

#### Custody claims (as stated)

| Area | Claim |
| --- | --- |
| SDK | Non-custodial; keys/mnemonics not sent to servers |
| Swap | Utexo does not custody user funds / not counterparty |
| Mint | “No custodian holds funds during mint” (product suite); signing via **federated TEE threshold** (orchestrator/connectors don’t hold keys) — still a multi-party TEE custody model for mint keys |
| Cloud | Managed infra; separate control-plane trust; RLN holds wallet/LN state |

#### Fee models (evidenced)

- Positioning: **deterministic / protocol-level fees** (What Utexo Is) — exact schedule **not published** in crawled pages.
- Mint: **0.03%** commission + chain gas + BTC RGB fee; `CommissionManager` on-chain.
- Swap: RFQ pricing; `slippageBps`; affiliate `feeBps`; amounts in “lots”.
- SDK: on-chain sends take `feeRate` (sat/vB); Lightning routing fee caps appear on WDK `sendPayment`.

#### Webhooks

| System | Mechanism |
| --- | --- |
| Cloud | `POST` to `webhookUrl`; `X-Utexo-Signature`; statuses RUNNING/STARTING/PAUSED/FAILED/IN_PROGRESS |
| Swap resolvers | Ed25519 `X-Signature` over `timestamp + raw body`; lifecycle events listed above |

---

### Likely interview questions (grounded in these docs)

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

### Gaps / thin / blocked pages

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

### Source URL list

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



---

*Compiled for Ricardo · Interview Prep · not affiliated with Boosty Labs or Utexo.*
