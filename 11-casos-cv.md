# Casos prácticos desde tu CV — Ricardo Fuentes

Solo hechos del CV. Si no lo viviste así, recorta. **No inventes números.**  
Inglés B1: frases cortas. ~90 segundos cada uno.

**Usa en el screen:** casos 1, 2, 4 y el pitch. El 3 y el 5 son backup.

---

## Mapa rápido — si preguntan X, cuenta Y

| Si preguntan… | Cuenta |
|---|---|
| Payments / API / high-frequency transfers | **Caso 1** — REALTOKN serverless |
| Custody / keys / who can freeze | **Caso 2** — MPC wallets |
| Bitcoin-native assets / RGB / Liquid / USDT | **Caso 3** — Liquid AMP |
| Lightning / LND / “have you run a node?” | **Caso 4** — CUBO+ (honesto) |
| Trial / first 30 days / prove fit | **Caso 5** — intern 3 meses + hackfest |
| Compliance / identity / transfer rules | **Caso 1** + 2 frases de ERC-3643 |
| Reliability / latency | **Caso 1** |

---

## Caso 1 — Transfers API (el más importante)

**Prompt típico:** “Tell me about a payments or API system you shipped.”

**Hechos CV:** REALTOKN, Blockchain Developer, Apr 2024–present. Serverless AWS (SAM, Lambda, API Gateway, DynamoDB) for high-frequency crypto + RWA transfers. Backend Go/Python. Throughput + latency.

### STAR (notas)

- **S:** REALTOKN needed a backend for frequent crypto and RWA transfers, not a toy demo.
- **T:** I owned the serverless path — API → Lambda → DynamoDB — so transfers stay consistent under load.
- **A:** Designed the SAM stack. Split work into Go/Python microservices. Used DynamoDB for transfer state. Watched latency and throughput for end users.
- **R:** We could take high-frequency transfer traffic without standing up always-on servers. Lesson: define transfer state clearly (created / pending / settled / failed) or you double-credit.

### Script (inglés, 90s)

> At REALTOKN I build the transfer backend for crypto and real-world asset moves. It’s serverless on AWS: API Gateway, Lambda, DynamoDB, packaged with SAM. My job is to keep transfers fast and consistent when volume goes up. I work in Go and Python. The hard part is not “call the chain” — it’s the state machine. A user can retry. A webhook can arrive twice. So I treat each transfer as a record with a clear status, not a fire-and-forget request. That’s the same discipline I would use on a Lightning payments API: payment hash as the id, credit once, classify failures instead of saying “it failed.”

### Follow-ups (ensaña 20s)

**Q: How do you avoid double credits?**  
> I key the transfer by a stable id. If the same request comes again, I return the existing record. I don’t create a second debit.

**Q: What did you measure?**  
> Latency for the user path, and whether transfers finish or stay pending. I care more about pending age than raw TPS.

**Q: DynamoDB vs a node?**  
> DynamoDB is our application ledger. The chain or wallet is another source of truth. Week one I always ask: which one is final for the partner?

### No digas

- Números de TPS/latencia que no tienes.
- “This is Lightning.” Di: *same discipline*, not same protocol.

---

## Caso 2 — MPC / custody (van a preguntar keys)

**Prompt típico:** “Who holds the keys? How do you think about custody?”

**Hechos CV:** Implemented MPC wallet protocols for security and decentralized custody.

### STAR

- **S:** REALTOKN holds digital assets. A single hot key is a single point of failure.
- **T:** Help implement MPC so signing is split — no one machine has the full key.
- **A:** Worked on MPC wallet flows so custody is shared across parties. Separated “app can request a transfer” from “who can actually sign.”
- **R:** Better security story than one seed on one server. Lesson: always ask who can sign, freeze, or recover.

### Script (75s)

> At REALTOKN I worked on MPC wallets. Instead of one private key on one server, the key is split. A transfer needs more than one share to sign. That reduces the blast radius if one machine is compromised. For Utexo I would ask the same questions in week one: who holds end-user keys today? Can anything freeze or reverse after a Lightning preimage? What’s the recovery path if a node force-closes? I won’t pretend MPC is the same as an LN node seed — different blast radius — but the habit is the same: draw the custody boundary before you write the API.

### Follow-ups

**Q: Is this non-custodial?**  
> MPC can still be custodial if the company holds enough shares. I don’t call it non-custodial unless the user controls the threshold.

**Q: vs VLS / remote signer on Utexo Cloud?**  
> I haven’t run their VLS setup. I understand the idea: keep signing keys off the node. I’d read their security docs in week one and pair with whoever owns the signer.

---

## Caso 3 — Liquid AMP (puente Bitcoin-assets)

**Prompt típico:** “Have you worked with Bitcoin-native assets? What is RGB?”

**Hechos CV:** Integrated Liquid AMP (Blockstream) for token issuance/management. Skills: Liquid Network, Bitcoin Core. RGB is **not** on the CV.

### STAR

- **S:** REALTOKN needed issuance and management of tokens in a Bitcoin-adjacent ecosystem, not only EVM.
- **T:** Integrate Liquid AMP so we can issue and manage those assets.
- **A:** Wired issuance/management into our stack. I had to learn Liquid’s model: a Bitcoin sidechain with its own federation and asset layer — different from ERC-20 on Ethereum.
- **R:** We could issue assets on Liquid, not only on account chains. Lesson: always separate issuance trust from transfer rails.

### Script (90s)

> Besides EVM work, I integrated Liquid AMP at REALTOKN — Blockstream’s issuance tools on the Liquid Network. Liquid is a Bitcoin sidechain. Assets live there with a federation trust model, not a global ERC-20 contract. I have not shipped RGB to production. What I *do* know is the question: where does issuance trust sit, and where do payments sit? On Utexo, RGB is client-side validation on Bitcoin UTXOs, and Lightning is the fast rail. Liquid is a different trust model — federation versus client-side. In week one I would run their WDK quickstart on the utexo signet and map those differences with whoever owns the asset layer.

### Follow-ups

**Q: Liquid USDT vs RGB USDT?**  
> I’d compare three things: trust model, tooling maturity, and the Lightning path. I won’t fanboy either. Utexo’s public story is RGB + Lightning.

**Q: Did you write RGB?**  
> No. CUBO+ and REALTOKN got me close to Bitcoin and Liquid. RGB is the gap I’ll close in the trial.

---

## Caso 4 — Lightning / CUBO+ (honesto, fuerte)

**Prompt típico:** “Have you run LND? What is Lightning?”

**Hechos CV:** Skills list Lightning Network (LND), Bitcoin Core. Cert: CUBO+ Devs Generation (Bitcoin & Lightning Fellowship), ONBTC, 2025. **No** dice que corriste LN en producción en REALTOKN.

### STAR

- **S:** I joined CUBO+, the Bitcoin and Lightning fellowship from El Salvador’s National Bitcoin Office.
- **T:** Learn Bitcoin Core and Lightning — channels, invoices, LND — well enough to use it as an engineer, not only as a reader.
- **A:** Practiced the mental model: capacity vs liquidity, BOLT11, HTLCs, why a big channel can still fail.
- **R:** I can explain Lightning clearly and I know LND as a tool. I have not been the on-call LN SRE for a PSP. In a trial I would pair on their node/Cloud path from day one.

### Script (80s)

> I listed LND because of CUBO+, a Bitcoin and Lightning fellowship with El Salvador’s National Bitcoin Office in 2025. That’s where I learned to think in channels, invoices, and liquidity — not from running Utexo’s mainnet. At work my production system is transfers, MPC, and Liquid, not forwarding HTLCs. So if a BOLT11 payment fails no-route in week one, I know the checklist: expiry, amount, local outbound, then the bottleneck hop. I would not restart the node first. I would classify the failure and ask the person who owns liquidity.

### Follow-ups

**Q: So you haven’t done production LN?**  
> Correct. Production for me is transfer APIs and custody. Lightning ops is the trial learning path — docs, signet, then their Cloud or self-hosted RLN.

**No digas:** “I operated LND in production at REALTOKN.”

---

## Caso 5 — Trial fit (intern + hackfest)

**Prompt típico:** “How would you prove you’re a keep in two weeks?”

**Hechos CV:** Conexión intern Sep–Nov 2023 (3 months). Cacao Track: top 10 / winner, Ticongle Hackfest, Dec 2023. Solidity supply-chain traceability.

### STAR (intern)

- **S:** Three-month blockchain intern at Conexión El Salvador.
- **T:** Ship real Solidity work, not only tutorials.
- **A:** Wrote contracts for product traceability in a supply chain. Asked early, shipped small, showed progress every week.
- **R:** Finished the intern with shipped contracts. Then Cacao Track won a top-10 place at Ticongle Hackfest — same idea: authenticity and traceability, under time pressure.

### Script (80s)

> I already did a short engagement. In 2023 I was a blockchain intern for three months at Conexión. I wrote Solidity for supply-chain traceability. I treated it like a trial: learn the domain, ship a thin slice, show it every week. After that, Cacao Track — a cacao authenticity project — was a top-10 winner at the Ticongle Hackfest. For Boosty I would do the same in two weeks: env and docs, one vertical slice — invoice to status — and a short failure notes doc from real SDK errors. You will see written updates, not silence.

---

## Extra — ERC-3643 / OID (30s, no story completa)

Úsalo **dentro** del caso 1 si preguntan compliance.

> On the EVM side I implemented ERC-3643 and ONCHAINID so transfers can be blocked if identity or compliance rules fail. That’s a permissioned token — the opposite of “anyone can send.” For Utexo I wouldn’t copy that onto RGB. I would ask where compliance sits: partner, mint, or protocol. Different layer, same question: who can freeze a transfer after it looks paid?

---

## Extra — proyectos (solo si sobra tiempo)

**DCA Calculator (dic 2025):** Bitcoin historical DCA / ROI. Muestra que vives en BTC, no que hayas hecho payments.

**Pupusa Index (nov 2025):** scrape + indicador de precios. Muestra data pipelines. No lo fuerces a Lightning.

---

## Pitch de trial (60s) — con TU CV

> I’m a blockchain developer at REALTOKN. I ship transfer APIs on AWS, MPC custody, and Liquid issuance. I also went through CUBO+, so I can talk Lightning without pretending I run your nodes. What I want in this trial is Bitcoin-native USDT rails — stablecoin UX with Lightning speed. Week one: official WDK or Cloud path on signet, one invoice-to-status slice, and a failure taxonomy. My gap is production RGB and LN ops. I’ll close it by pairing and logging real errors, not by bluffing.

---

## 4 preguntas (igual que el cram)

1. Boosty staff-aug into Utexo, Utexo direct, or general Boosty Web3?
2. What does “strong after trial” look like in week 2 vs month 2?
3. Day-one stack: LN ops, RGB, backend API, or partner integrations?
4. Who is the tech expert and what product area do they own?

---

## Checklist de práctica (esta noche)

- [ ] Caso 1 en voz alta, timer 90s
- [ ] Caso 2 en 75s
- [ ] Caso 4 (CUBO+) sin decir “production LND”
- [ ] Pitch 60s
- [ ] Un follow-up de double-credit + uno de custody
