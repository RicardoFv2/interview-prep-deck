# STAR stories + trial fit (tech-expert screen)

**Filled from Ricardo Fuentes CV.** Full spoken scripts: [`11-casos-cv.md`](./11-casos-cv.md).  
**Rule:** no invented metrics. LND = CUBO+ fellowship, not REALTOKN production.

---

## STAR template (strict)

- **S**ituation — 1 sentence context  
- **T**ask — your responsibility  
- **A**ction — 2–4 concrete steps *you* did  
- **R**esult — metric or clear outcome + lesson  

---

## Five slots (from CV)

### Story 1 — Ambiguous integration / transfers API  → Caso 1

- **S:** REALTOKN needed a backend for high-frequency crypto + RWA transfers.
- **T:** Own the serverless path (API Gateway, Lambda, DynamoDB, SAM).
- **A:** Designed the SAM stack; Go/Python microservices; transfer records with clear status so retries don’t double-credit.
- **R:** System can take frequent transfer traffic without always-on servers. Lesson: state machine > “just call the chain.”
- **Bridge:** Same discipline as Lightning payment hash + webhook settlement.

### Story 2 — Incident / reliability / custody  → Caso 2

- **S:** Single hot key would be a single point of failure for digital assets.
- **T:** Implement MPC wallet protocols for split signing / decentralized custody.
- **A:** Built MPC flows so a transfer needs more than one share; separated “app requests” from “who can sign.”
- **R:** Smaller blast radius than one seed on one box. Lesson: always draw the custody boundary first.
- **Bridge:** Week-1 questions: keys, freeze after preimage, force-close recovery.

### Story 3 — Explained hard tech / Bitcoin assets  → Caso 3

- **S:** Needed token issuance beyond EVM — Bitcoin-adjacent stack.
- **T:** Integrate Liquid AMP (Blockstream) for issuance and management.
- **A:** Learned Liquid’s federation + asset model vs ERC-20; wired it into our stack.
- **R:** Issuance on Liquid, not only account chains. Lesson: issuance trust ≠ payment rails.
- **Bridge:** RGB is a different trust model (client-side). Honest gap: haven’t shipped RGB.

### Story 4 — Short engagement / proving fit  → Caso 5

- **S:** 3-month intern at Conexión (Sep–Nov 2023).
- **T:** Ship Solidity, not tutorials.
- **A:** Supply-chain traceability contracts; weekly visible progress.
- **R:** Intern completed with shipped contracts. Cacao Track = top-10 Ticongle Hackfest (Dec 2023).
- **Bridge:** Same keep-signal for a 2–3 month pasantía: slice + written updates.

### Story 5 — Closest to Lightning / Bitcoin  → Caso 4

- **S:** CUBO+ Devs Generation — Bitcoin & Lightning Fellowship, ONBTC, 2025.
- **T:** Learn Bitcoin Core + LND well enough to think like an engineer.
- **A:** Channels, BOLT11, HTLCs, capacity vs liquidity.
- **R:** Can explain Lightning and debug with a checklist. Have **not** been LN SRE in production.
- **Bridge:** Week-1: classify no-route before restarting the node; pair on Cloud/RLN.

---

## Trial / pasantía pitch (60 seconds)

> I’m a blockchain developer at REALTOKN. I ship transfer APIs on AWS, MPC custody, and Liquid issuance. I also went through CUBO+, so I can talk Lightning without pretending I run your nodes. What I want in this trial is Bitcoin-native USDT rails. Week one: official docs/SDK on signet, one invoice-to-status slice, and a failure taxonomy. My gap is production RGB and LN ops. I’ll close it by pairing and logging real errors, not by bluffing.

---

## Questions you ask them (memorize 4)

1. Is the seat Boosty staff-aug into Utexo, Utexo direct, or general Boosty Web3?  
2. What does “strong after trial” look like in week 2 vs month 2?  
3. Day-one stack: LN ops, RGB, backend API, or partner integrations?  
4. Who is the tech expert and what product area do they own?

---

## Practice drill

- Tell Stories 1, 2, 5 out loud (90s each)  
- Deliver trial pitch once  
- Ask your 4 questions as if closing the screen  

Full scripts + follow-ups: `11-casos-cv.md`.
