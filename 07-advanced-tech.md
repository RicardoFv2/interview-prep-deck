# Advanced layer — tech-expert depth

**Why this exists:** Days 1–3 get you fluent. This file is what a tech expert probes next: protocol mechanics, adversarial thinking, production systems, and design tradeoffs.

**Rules**
- Prefer precise vocabulary + tradeoffs over buzzwords.
- If you don’t know, say how you’d verify (spec, logs, experiment) — don’t bluff RGB internals.
- Map answers to *payments systems* (idempotency, finality, liquidity, custody), not coin price talk.

**Timebox:** 2–3 focused sessions. Speak answers out loud.

---

## 1. Lightning — protocol depth

### 1.1 Commitment transactions & revocation (must sound real)
- Each channel update creates a new **commitment transaction** pair (yours + theirs), pre-signed, not broadcast.
- Old states are made dangerous via **revocation**: sharing a revocation secret for the previous commitment so the peer can punish (justice tx) if you broadcast an outdated state.
- **Interview point:** Lightning’s security assumption is *you (or a watchtower) notice a revoked state on-chain in time*.

**Practice answer (60s):**  
“Channel balances aren’t a database row — they’re encoded in the latest mutual commitment txs. When we update, we revoke the prior state. If I broadcast an old fat state, my peer can claim a penalty using the revocation secret. That’s why offline risk and watchtowers matter.”

### 1.2 HTLC lifecycle (more than “hash + time”)
Know the stages: add → fulfill (preimage) / fail → remove.  
Timeouts are **directional** along the path (upstream usually has more time than downstream).  
**Cltv expiry delta** mismanagement → stuck funds or unsafe routing.

**Hard question:** “What happens if an intermediate hop goes offline while HTLCs are in-flight?”  
**Good:** Funds locked until timeout; sender/receiver experience pending; force-close may be needed to resolve on-chain; operational cost and liquidity lockup.

### 1.3 Fee market & pathfinding
- Nodes advertise base fee + proportional fee + CLTV delta + HTLC min/max.
- Pathfinding is probabilistic under **liquidity uncertainty** (you don’t know remote balances).
- **MPP** (multi-path): split large payments; partial failure complexity.
- Cheap route ≠ reliable route.

### 1.4 Channel jamming (adversarial)
Attacker locks liquidity with HTLCs that never settle (or slow-fail), reducing useful capacity.  
Mitigations discussed in ecosystem: reputation, upfront fees, liquidity ads, better failure attribution — know it’s an open ops/security topic, not “solved.”

### 1.5 Dual-funding, splicing, JIT (product relevance)
- **Dual-funding:** both sides fund open — better initial balance symmetry.
- **Splicing:** adjust channel capacity without full close/reopen (implementation maturity varies).
- **JIT channels / LSPs:** open liquidity when an invoice arrives — UX win, trust/dependency on LSP.

### 1.6 Force-close economics
Cooperative close is cheap; force-close burns fees + timelocks + degraded UX.  
**Interview line:** “I’d treat force-close as an incident class with runbooks, not a normal control flow.”

---

## 2. Production engineering for Lightning payments

### 2.1 Failure taxonomy (build this in week 1 of a trial)
| Class | Examples | Owner intuition |
|---|---|---|
| Invoice | expired, amount mismatch, bad hints | API validation |
| Local liquidity | insufficient outbound | Treasury / rebalance |
| Path / remote liquidity | no-route, temporary channel fail | Graph + peers |
| Peer liveness | disconnect mid-HTLC | Ops / SRE |
| Policy | fee insufficient, HTLC size limits | Config |
| Chain | force-close, sticky unresolved | Incident |

**Metric set:** success rate by class, p50/p95 invoice→settled, pending age histogram, inbound headroom, webhook lag, force-close count.

### 2.2 Idempotency & reconciliation
- **Payment hash** (and your internal payment id) as idempotency keys.
- Webhooks can duplicate — at-least-once delivery assumed.
- Ledger rule: credit once per settled payment; store preimage/proof; reconcile nightly vs node.

**Hard question:** “User says paid; merchant says unpaid.”  
Walk: invoice state → node HTLC/payment state → webhook delivery log → partner ledger entry → whether preimage exists.

### 2.3 Custody & key management boundaries
Separate: hot node keys (LN) vs cold/treasury vs partner custody.  
Compromise of node seed ≠ same blast radius as partner exchange cold wallet — say so explicitly.

---

## 3. System design prompts (practice whiteboarding)

### Prompt A — PSP USDT payout API
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

### Prompt B — Liquidity ops for Cloud/LSP-like product
How do you keep inbound liquidity for thousands of wallets/merchants?

Talk: pool management, rebalancing (circular / submarine swaps if relevant), pricing liquidity, alerting on headroom, per-tenant isolation vs shared pools, jamming resistance.

### Prompt C — Bridge + LN (careful)
If assets move from another system onto Bitcoin-native representation, separate trust domains:
1. Issuer/bridge (lock/mint, operators, TEEs if claimed)
2. LN ops
3. Partner custody / compliance

Never debug mint failures with only pathfinding tools.

---

## 4. RGB — interview-depth (still honest)

### 4.1 What to own intellectually
- **Client-side validation:** verify consignments for UTXOs you care about; no single global token VM state like ERC-20.
- **UTXO anchoring:** asset allocations move with Bitcoin coin selection constraints.
- **Privacy/disclosure tradeoff:** different from public explorers; UX and backup story harder.
- **RGB + Lightning:** asset-aware invoices / LN transport for assets — newer stack; beta tooling exists in ecosystem (e.g. public WDK modules) — treat maturity as a question, not a boast.

### 4.2 Hard questions + good shapes
**Q:** “How do you backup an RGB wallet differently from an LN-only wallet?”  
**Shape:** seeds + channel backups + RGB state/consignment availability; losing client-side data can be worse than LN alone.

**Q:** “What breaks if two systems disagree on asset state?”  
**Shape:** consignment completeness, reorg assumptions, bridge finality vs LN pending — define source of truth per layer.

**Q:** “Compare RGB USDT to Liquid USDT / other Bitcoin-adjacent stables.”  
**Shape:** trust model (federation vs client-side), tooling maturity, Lightning integration path, compliance optics — pick criteria, don’t fanboy.

### 4.3 Honest competence script (keep)
State gap → show mental model → week-1 verification plan on their docs/SDK → who you’ll pair with.

---

## 5. Security & privacy questions

- Invoice metadata leakage (description, amount correlation)
- Payment probing to discover balances
- Watchtower privacy vs security
- Supply-chain risk on node software / LSP
- “Private” claims for USDT-on-Bitcoin — ask *what* is hidden from whom

**Line:** “I’d ask for a threat model doc before echoing marketing privacy language.”

---

## 6. Hard mock questions (answer ≤90s each)

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

### Model sketches (short)

**(2) Failure codes:** `invoice_expired`, `amount_invalid`, `insufficient_outbound`, `no_route`, `peer_unavailable`, `fee_insufficient`, `htlc_timeout`, `policy_rejected`, `internal` — plus retryable boolean.

**(5) Webhook:** partner sends `Idempotency-Key` or you key by `payment_hash`; handler is transactional insert-if-not-exists; return 200 on duplicates; signed payloads; outbox for retries.

**(9) Trial SLOs:** payment success rate by class, p95 settle time, zero silent double-credits, runbook for pending>N minutes, weekly liquidity headroom report.

---

## 7. What “too simple” vs “tech-expert” sounds like

| Topic | Too simple | Tech-expert |
|---|---|---|
| Lightning | “It’s fast Bitcoin” | Commitments, revocation, liquidity direction, HTLC timeouts |
| Failure | “No route, try again” | Classified failures + metrics + playbooks |
| Product | “USDT on Bitcoin” | Finality policy, custody boundary, fee quote vs settle |
| RGB | “It’s private tokens” | Client-side validation + backup/ops tradeoffs + honesty on maturity |
| Trial | “I’ll learn fast” | Week-1 taxonomy + vertical slice + keep signals |

---

## 8. Drill plan (replace shallow review)

**Session A (60–90m)** — Protocol: §§1.1–1.3 out loud + Q1, Q4, Q6  
**Session B (60–90m)** — Production: §2 + design Prompt A outline on paper  
**Session C (45m)** — RGB/security §§4–5 + Q8, Q10  
**Session D (30m)** — Hard mock §6 timed

Score Clarity / Depth / Rigor (not vibes).

---

## 9. Fill with *your* depth

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
