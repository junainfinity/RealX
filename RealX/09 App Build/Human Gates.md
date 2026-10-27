---
title: "Human Gates"
created: 2026-09-24
updated: 2026-09-24
status: complete
tags: [realx, app-build, human-gates]
aliases: ["What AI Agents Cannot Do", "Human Steps", "RealX Human Gates"]
---

# Human Gates

**The build agents can design, build, test and verify almost all of RealX. They cannot, and must not, do the twelve things below.** Under decision D19, Opus 5.5 (high) plans each node, Grok 4.7 (xhigh) builds it, Opus 5.5 (high) does the node QA at every gauntlet loop, and Fable 5.1 (high) runs the graph QA at each milestone close. Each of these steps needs a person or firm with legal standing, money or accountability. Agents prepare every one of them: the brief, the checklist and the evidence template. A human performs the step and records the evidence. No gate is ever closed on an agent's claim. The human-gate board in the operator console (A76) shows each one as open until a human records it, and A89 relies on that. This note is not legal advice: every legal point below is a question for counsel, not a conclusion.

Agents build and test locally, on a local Hardhat chain and on Base Sepolia testnet, with sandbox adapters and keys that carry no value. `RUN_MODE=local|test` refuses live adapters, production run modes and mainnet chain ids by test (A08, A23, A27, A86).

## Names used in the node specs

The node specs were written before this note and use some older gate names. They mean the same gates:

| This note | Name in the node specs | Notes |
|---|---|---|
| HG-LEGAL | HG-LEGAL | Counsel opinions. The counsel questions CQ-01 to CQ-10 are frozen in A00 (`counsel-questions.md`) |
| HG-LICENCE | Part of HG-LEGAL ("licences, CAR appointment, RE for hire" in A00) | Split out here because it runs on its own timeline and binds the P2 and P3 flags |
| HG-ACCOUNTS | HG-VENDOR, HG-VENDOR/ACCOUNTS, HG-ACCOUNTS | One gate: vendor contracts and accounts |
| HG-KEYS | HG-KEYS | |
| HG-MONEY | HG-CAPITAL | In the specs HG-CAPITAL means real client money and opening the trust accounts |
| HG-CHAIN | The mainnet part of HG-PROD | Mainnet deployment, after HG-AUDIT |
| HG-PROD | HG-PROD (the production-infrastructure part) | Production infrastructure and the named production operators |
| HG-AUDIT | HG-AUDIT | |
| HG-PENTEST | HG-PENTEST | |
| HG-DATA | Carried as HG-LEGAL questions (A04, A10, A102) and HG-PROD items (hosting, A84, A102) | New in this note |
| HG-DESIGN | HG-REVIEW, and HG-LAUNCH-P1-LOCAL in A89 | The founder's reviews and sign-offs during the build |
| HG-LAUNCH | HG-LAUNCH | Real go-live only |
| Standing human roles | HG-OPS | See [[#Standing human roles]] |

The next spec pass should rename these in the specs. Until then, both names are valid.

## Start now, in parallel with the build

Five human tracks have long lead times. They should start when A00 passes, which is about <!--g:finish.A00-->2-4<!--/g--> agent-hours into the build (figures in this note are generated from `Data\app-graph.json`, one milestone at a time on three streams). The lead times below are planning assumptions, not quotes.

| Track | Starts | Why now | Assumed lead time | What it unblocks |
|---|---|---|---|---|
| HG-LEGAL | When A00 passes. A00 writes the counsel brief with CQ-01 to CQ-10 | Counsel's answers change parameters in many nodes and decide whether P1 can go live | 4-12 weeks for the first opinion | Re-tests in the owning nodes; HG-LAUNCH |
| HG-LICENCE | Now | The CAR appointment decides whether RealX can arrange anything at all. Hiring an RE for P2 takes longer | Weeks for the CAR; months for the RE and scheme registration | HG-LAUNCH (P1); the P2 flags (A90-A94) |
| HG-ACCOUNTS | Choose vendors now. The bundler account is needed before A86 starts (about <!--g:start.A86-->528-798<!--/g--> agent-hours into the build); until then A86 is parked, not failed | Vendor sandboxes let the conformance suites run early. Contracts take weeks | Days for the testnet account; weeks for contracts | A86; selecting a live adapter at every swap point |
| HG-AUDIT (booking) | Now | Good audit firms book weeks ahead. The audit pack is ready at A86 | Booking 4-8 weeks; each audit 2-4 weeks | HG-CHAIN; HG-LAUNCH |
| HG-PENTEST (booking) | Now. The engagement starts after A83 and A88 | Testers book ahead | Engagement 3-6 weeks | HG-LAUNCH |

HG-DESIGN is not a lead-time track, but the user needs to be free for two reviews: extension A (A50), which now builds in AM2 so its review can be given while AM2 and AM3 run, and extension B (A51) in AM4. On the schedule the orchestration runs, A50 finishes at about <!--g:finish.A50-->178-254<!--/g--> agent-hours and A51 at about <!--g:finish.A51-->354-517<!--/g-->. A50 merges and holds its dependents until the user approves (`human_gate: blocks-dependents`); A51 does not pass until the user approves (`blocks-own-gate`).

## HG-LEGAL

External counsel answers the legal questions. Every one of them is already a parameter, flag or rule-pack row with a safe default, so no build node waits for an answer. An answer changes data, and the owning node's hidden tests re-run.

- **Digital Assets Framework Act.** Does RealX need a tokenised custody platform (TCP) or digital asset platform (DAP) authorisation, and when? Does the non-custodial passkey wallet with a RealX guardian, or the AgentGateway attestation design, change that answer (CQ-08; A15, A21, A26, A29)? What are the low-value exemption thresholds shown on ADM-04 (A21, A51, A69)? Timing: commencement on 8 Apr 2027 and transition to 8 Oct 2027.
- **s601ED.** How does the 20-member trigger apply to each sponsor's wholesale trust, and what must the holder cap and the ADM-16 meter enforce (CQ-07; A19, A36, A51)?
- **Retail cooling-off.** None or 14 days? How is it counted? Does it run before funding or after issue? Does it apply to an illiquid scheme (CQ-06; A35, A37, A90)?
- **FIRB and fractional units.** How do the established-dwelling ban and the notification duty apply to foreign persons buying fractional trust units, for residential and commercial property? Must onboarding capture citizenship and visa status, not only tax residency (G06; A19, A30, A56)? The build enforces the safe side until counsel answers.
- **T-REX under GPL-3.0.** Can RealX deploy and operate the vendored T-REX suite, ONCHAINID and the account-abstraction contracts? Are RealX modules that inherit from them derivative works? The clean-room implementation of the EIP interfaces is the fallback (CQ-05; A07, A24, A86).
- **Personal declarations versus agents (D04).** Which declarations must the person make themselves, not their agent: the investor declarations, that is wholesale-status truthfulness, target-market answers in P2 and the entity signatory declaration (CQ-01); the sponsor declarations, that is the application declaration and its commitments, the fee-agreement acceptance, fit-and-proper and submission declarations, the monthly operator attestation and the title-verification declaration (CQ-09); and the investor tax declarations, that is the FATCA/CRS self-certification, the residency and TIN declaration and the no-TFN acknowledgement (CQ-10)? May an agent acknowledge disclosures under the holder's standing consent (CQ-02)? Does offering agent tooling or accepting a mandate change RealX's licence perimeter (CQ-03)? Does a sponsor's agent count as one of the sponsor's two people (CQ-04)? May RealX staff use personal AI agents for regulated functions as a CAR (A77)? Until counsel answers, the rows marked `"counsel"` in A03's `delegable.ts` stay non-delegable (A03, A12, A32, A36, A60, A63, A100, A104). The build does not silently block these acts: they are recorded as counsel questions, and the answers are data.
- **Other questions recorded in the specs.** These include:
  - AML thresholds, PEP and beneficial-owner rules, and third-party deposits (A17, A30, A42);
  - SMR, TTR and IFTI reporting clocks under the AML/CTF Act 2006 as amended, reported to AUSTRAC (A13, A42);
  - complaint acknowledgement and final-response clocks under RG 271 (A13, A44);
  - retention periods (A10, A102);
  - whether the trustee must countersign each act (A98, A105);
  - whether the trust deed may name the token ledger as the s169 register (A18);
  - the forced-transfer and recovery order (A28);
  - float interest (A39);
  - withholding and AMMA content (A41);
  - whether Auto Invest counts as personal advice (A95);
  - minors (A97);
  - regulator notice duties in the runbooks (A85, A103).
- Counsel also approves the terms, privacy policy, PDS and TMD templates, disclosure wording and the developer terms for the SDK and MCP (A99).

*Agents prepare:* the counsel brief and `counsel-questions.md` (A00), draft documents, and the encoding of every answer as data. *Evidence:* a signed opinion per question, linked from the rule-pack row it sets. *Blocks:* no build node. It blocks HG-LAUNCH, and each P2 or P3 flag whose question is still open.

## HG-LICENCE

- An AFSL holder appoints RealX as a corporate authorised representative (CAR), covering arranging for wholesale clients. The authorised representative number and the licensee's conditions go into A21's licence records.
- The founder confirms each sponsor's own AFSL, or its licensed trustee, at admission (A33, A69).
- Before P1 go-live: AUSTRAC enrolment, the AML/CTF program and an appointed MLRO (A42, A75), and AFCA membership through the licensee or RealX (A44).
- For P2: engage the hired responsible entity, register the umbrella scheme with ASIC with its constitution and compliance plan, and obtain any ASIC relief or counsel opinion for the resale board and on-platform votes (A90-A94).
- Any TCP or DAP authorisation that counsel says is needed (HG-LEGAL).

*Agents prepare:* the licence-record schema and the flag bindings (A21). Only a licence or relief record can switch on `offers.retail`, `resale.board`, `governance.votes`, `autoinvest.enabled` or `payouts.stablecoin`. *Evidence:* the licence or relief document, recorded by a human in A21's register. *Blocks:* HG-LAUNCH (P1), and turning on the P2 and P3 flags (A90-A97).

## HG-ACCOUNTS

A human chooses, contracts and owns every vendor account. Agents get sandbox credentials only.

| Swap point | Assumed provider type | Adapter node | Other nodes that re-run on a swap |
|---|---|---|---|
| Identity, DVS, liveness, sanctions and PEP | Equifax, Sumsub or Averer type | A22 | A30, A56, A68 |
| Bank rail: NPP, PayID and PayTo into client-money trust accounts | Monoova or Zepto type | A17 | A16, A39, A74, A95 |
| Custody signing policy engine (TP-SIGN) | Fireblocks type | A15 | A25, A28, A72, A91 |
| Unit registry | IQ-EQ, Automic or Boardroom type, or the trustee's own register | A18 | A20, A73, A98, A105 |
| E-sign, email and SMS | Any | A35, A43 | |
| Bundler and RPC for Base Sepolia | Pimlico or Alchemy type | A29, A86 | |
| IP-to-country data | Licensed database | A101 | |
| P3: KYT, travel rule, stablecoin issuer, market venue | Chainalysis, TRM or Elliptic type; AUDM, AUDD or A$DC issuer | A96, A97 | |

- **Rule for every swap.** A vendor implementation may be selected only after the contract is signed, and only if it passes the adapter's conformance suite (A08) and the owning node's hidden tests unchanged. The rest of the system does not change: the register, ledger, reconciliation and rules engine never depend on vendor internals.
- **Needed during the build.** A free-tier bundler account and faucet funding for Base Sepolia, before A86 starts. This is the only vendor step on the build's path.

*Agents prepare:* the conformance suites, `SWAP-POINTS.md` and the vendor questionnaires. *Evidence:* the contract reference and a green conformance run against the vendor sandbox. *Blocks:* A86 (the testnet account); a live adapter at every swap point; HG-LAUNCH.

## HG-KEYS

- Key ceremonies for production keys, with named human holders on hardware or HSM or MPC custody:
  - **V1** owner, 3-of-5;
  - **V2** token agent, 2-of-3;
  - **V3** pauser, 1-of-N;
  - **V4** identity agent: an automated key with per-function limits, held in an HSM or the custody vendor's policy engine;
  - **V7** paymaster;
  - **V5 and V6** vaults, for P3.
- The staff hardware-key allow-list and the issue of staff keys (A53, A67). The break-glass credentials, split among the CCO, SECO and CEO (A67). The register signing key (A18). The backup escrow and erasure-seal keys (A102).
- Rotation ceremonies and recovery documentation.
- **No production key ever enters an agent environment.** Agents use the public Hardhat accounts and A15's no-value keystore. A86 uses throwaway testnet keys that hold only faucet test ETH.

*Agents prepare:* the ceremony scripts and checklists (A15, A25, A26, A85), and the rotation drills in the sandbox (A103). *Evidence:* a signed ceremony record with holders, device serials and witnesses. *Blocks:* HG-CHAIN, HG-MONEY and HG-LAUNCH. No build node is blocked.

## HG-MONEY

- Open the client-money trust accounts at an Australian ADI (operating, escrow and distributions), and set up the PayID domain.
- Accept the first real client money. Make the first real payout, refund, distribution and ATO withholding remittance.
- P2: open the trust accounts for each retail sub-fund.
- Every real money movement stays under maker-checker with two humans. A staff agent may prepare a money action as maker, but never approve it.

*Agents prepare:* the ledger, matching, reconciliation and treasury flows, all proven on sandbox money (A16, A17, A37, A39, A40, A41, A74, A90), and the first-money runbook. *Evidence:* the bank's account confirmations, and a clean three-way reconciliation on the first real day. *Blocks:* HG-LAUNCH. No build node is blocked.

## HG-CHAIN

- Choose the production chain: Base or Redbelly.
- Deploy the contracts to mainnet through the V1 owner multisig, only after HG-AUDIT has closed. The deployed bytecode must match the audited commit: A27's manifest and A86's preflight refuse the deploy while `audited_source_sha256` is empty or stale.
- Register the trusted issuers and claim topics on the public chain (A31). Fund the paymaster (V7).

*Agents prepare:* the deployment scripts and manifests (A24-A28), and the Base Sepolia rehearsal (A86). Agents have no mainnet RPC, chain id or manifest anywhere. *Evidence:* the deployment transaction hashes signed by V1, and the audited-commit match. *Blocks:* HG-LAUNCH; P3 contracts (A96).

## HG-PROD

- Name at least two production operators, so that every production action is paired and logged. They own the cloud root account and the KMS/HSM administration. They deploy only builds whose node gates and Fable 5.1 graph gate have passed, and they hold break-glass access under the incident commander.
- Production infrastructure:
  - hosting region, which ties to HG-DATA;
  - database capacity and replicas, informed by A84;
  - backups with off-site immutable copies, and the RPO and RTO targets (A102);
  - the telemetry backend (A85);
  - production origins and relying-party ids for `apps/web` and `apps/console` (A52, A53).
- Publishing `@realx/sdk` and `@realx/mcp` to the public npm registry, and hosting the API docs on a public domain, happen at go-live (A99, A104).

*Agents prepare:* the Compose stack, the images with pinned digests, the runbooks and the capacity report. *Evidence:* the operator roster, the deploy log and the restore drill on production. *Blocks:* HG-LAUNCH.

## HG-AUDIT

The founder contracts two independent smart-contract audit firms, booked early. Scope:
- the RealX compliance modules, claim-issuer override, document anchor and deployer (A24);
- the AgentGateway and owner multisig (A25);
- the passkey smart wallet, factory, guardian registry, RecoveryModule, SessionKeyModule and paymaster (A26);
- any change to the reference ClaimIssuer (A31);
- the resale-board settlement path (A91, P2);
- the V5 and V6 vaults (A96, P3).

Fixes land in the owning nodes, the auditors re-check them, and the reports are published. A bug bounty follows.

*Agents prepare:* the audit pack from A86 (source, spec, invariants, test report, gas, Slither output from A83, residual risks). A83's Slither run is hygiene, not an audit. *Evidence:* two final reports with every finding resolved or accepted, and the audited commit hash. *Blocks:* HG-CHAIN and HG-LAUNCH.

## HG-PENTEST

The founder contracts an independent penetration tester. The engagement starts after A83 (the internal security pass) and A88 (the regression), and uses `docs/security/pentest-brief.md`. Scope:
- the web app, operator console, partner portal, API, webhooks and infrastructure;
- the passkey, recovery and session surfaces;
- tenant isolation;
- the maker-checker paths.

**Agent-token abuse is named in the scope:**
- token theft, replay and forgery, including the verification memo;
- scope or mandate escalation past the owner;
- caps, limits and time windows evaded by splitting;
- agent calls reaching non-delegable acts;
- a staff agent reaching checker, approver or signer;
- session-key misuse beyond its contracts, functions, amounts or expiry;
- MCP transport and origin attacks, and prompt injection through tool outputs;
- webhook SSRF and signature replay;
- crossing from the sandbox to live data;
- evading anomaly holds and kill switches.

The relevant nodes are A12, A29, A46, A60, A77, A82, A83, A99, A100, A101 and A104.

*Agents prepare:* the brief, the threat model and fixes in the owning nodes; the tester re-tests. *Evidence:* the tester's report and re-test letter. *Blocks:* HG-LAUNCH, and each P2 or P3 switch-on.

## HG-DATA

- A privacy assessment under the Privacy Act 1988 and the Australian Privacy Principles, covering identity documents, tax file numbers, bank details and beneficial-owner data.
- Data residency: where personal data is hosted and backed up (HG-PROD), and any cross-border disclosure to vendors under APP 8 (identity, custody, e-sign, email and SMS providers).
- Record retention and erasure periods for each record class. The candidates are the AML/CTF records, the Corporations Act registers including former members, and the tax records. A102 builds them as parameters; counsel sets the values.
- The notifiable data breaches procedure in the runbooks (A85).
- Agent data egress (D04): what a person's own agent, webhook endpoint or third-party model provider may receive, the consent wording on ACC-05 and SPN-16, and PII masking in the MCP tool outputs.

*Agents prepare:* the PII inventory and column classes (A04), the masking and unmask audit (A30, A68), the retention and erasure machinery (A102), and a draft privacy impact assessment. *Evidence:* the approved assessment, the retention schedule and the hosting-region decision. *Blocks:* HG-PROD and HG-LAUNCH.

## HG-DESIGN

The founder's reviews during the build. The specs call this HG-REVIEW. Each node carries its mode as `human_gate` in its frontmatter and in `Data\app-graph.json` (`none`, `blocks-own-gate`, `blocks-dependents` or `milestone-close`), and the orchestration enforces it ([[Build Orchestration]] §6), so no agent has to read the mode from prose.

- **A00:** read the charter, the D04 non-delegable list, the partner surfaces (ENG-02) and the agent principal model (ENG-03). This does not block; the build continues on the recorded defaults.
- **A50:** review design extension A (investor and sponsor screens, including ACC-05, SPN-16 and the partner frame) in the browser. **This blocks its dependents** (A54-A60, A62-A64).
- **A51:** review design extension B (operator screens, including ADM-35 and ADM-36 to ADM-40). **This blocks A51's gate**, and with it A53 and A66-A77.
- **A49:** the money-path walk-through ("magic inputs"), a founder checkpoint after the gate.
- **A61:** walk the investor preview in Docker Desktop. This closes AM5, not the software gate. Feedback becomes fix loops.
- **A87:** the side-by-side fidelity walk, prototype against build, at 1280 and 375 px. **This blocks A87 and therefore A88.**
- **A89:** the P1 software sign-off in Docker Desktop (HG-LAUNCH-P1-LOCAL in the spec). This is not go-live.
- Later reviews: A92 (P2 screens, at A94), A95 and A97 (P3 screens), and A99 (the docs page).
- Specify-point confirmations recorded as data with defaults, for example the valuation committee's independent member (A03, A34).

*Agents prepare:* the extension files, the review checklists, the galleries and the preview guides. *Evidence:* `humanApproval` in the node's evidence manifest.

## HG-LAUNCH

Go-live with real investors and real money. It needs HG-LEGAL, HG-LICENCE, HG-ACCOUNTS, HG-KEYS, HG-MONEY, HG-CHAIN, HG-PROD, HG-AUDIT, HG-PENTEST and HG-DATA closed for P1, and the standing roles filled.
- The founder signs go or no-go.
- The founder approves every cap increase and every switch of a P2 or P3 flag, after its licence binding (A94 for P2; A95, A96 and A97 for P3).
- Publishing the SDK and MCP packages is part of go-live (A99, A104).

*Agents prepare:* the readiness report from A88 and A89 and the gate board (A76). *Evidence:* the signed go-live record. *Blocks:* nothing in the build, because every build node ends in local, testnet or sandbox evidence.

## Standing human roles

Some gates are permanent jobs, not one-off steps. The build models each role as a seeded persona, so no build node waits for a person. Each role must be filled by a named human before the event in the "Needed by" column.

| Role | Gate | Minimum headcount | Needed by | Modelled in |
|---|---|---|---|---|
| External counsel | HG-LEGAL | 1 firm | Now (the A00 brief) | A00 |
| AFSL licensee and responsible managers | HG-LICENCE | Per the licensee | P1 go-live | A21, A69 |
| Compliance officer (CCO) | HG-LICENCE, HG-LAUNCH | 1 | P1 go-live | A14, A48, A101 |
| MLRO | HG-LICENCE | 1, and it may not also be the CCO where the program forbids it | P1 go-live (the first real case) | A42, A75 |
| Complaints officer (G18) | HG-LICENCE | 1 | P1 go-live | A44 |
| Security officer (SECO): approves staff tokens, reviews held agent tokens within one business day, and ratifies kill-switch tiers with the CCO within 2 hours | HG-KEYS, HG-PROD | 1 plus a deputy | P1 go-live | A12, A77, A101 |
| V1 owner signers | HG-KEYS | 5 (3-of-5) | HG-CHAIN | A25 |
| V2 token-agent signers | HG-KEYS | 3 (2-of-3) | HG-CHAIN | A15, A28 |
| V3 pauser signers | HG-KEYS | 2 or more, so a pause never waits on one person | HG-CHAIN | A48 |
| Production operators | HG-PROD | 2 or more, so every action is paired | HG-PROD | A85, A102 |
| On-call rotation and incident commander | HG-PROD | 2 on call plus 1 commander | P1 go-live | A85, A103 |
| Trustee or registrar, per trust; the RE for P2 | HG-LICENCE | 1 per vehicle | The first real listing; P2 | A18, A98, A105 |
| Financial and client-money auditor | HG-MONEY | 1 firm | The first reporting period | A45, A105 |
| Tax adviser | HG-LEGAL | 1 | The first distribution | A41 |
| Valuer panel | HG-ACCOUNTS | Per property | The first listing | A34, A70 |

Agents never fill a standing role. Staff agents may act as maker within their human's role, but never as checker, approver or custody signer, and segregation of duties counts an agent as its human. One person may hold more than one seat only where A11's segregation-of-duties rules allow it. The founder funds these roles, the audits, the pentest, the vendors and counsel. None of that is an agent decision.

## Human steps by node

**Steps that block a build node or close a milestone:**

| Gate | Node | The step |
|---|---|---|
| HG-DESIGN | A50 | Blocks its dependents until the user approves extension A |
| HG-DESIGN | A51 | Blocks its own gate until the user approves extension B |
| HG-DESIGN | A87 | Blocks its own gate: the fidelity walk |
| HG-DESIGN | A61, A89 | Close AM5 and AM8 after their software gates pass |
| HG-DESIGN | A00, A49 | Non-blocking review and checkpoint |
| HG-ACCOUNTS | A86 | The Base Sepolia bundler account and faucet funding before its first session |

**The other gates block only go-live or a flag switch-on.** The node specs record them in their "Human gates touched" sections:
- HG-LEGAL: recorded in 100 specs;
- HG-ACCOUNTS: 62 (as HG-VENDOR or HG-ACCOUNTS);
- HG-PENTEST: 48;
- HG-PROD and HG-CHAIN: 44;
- HG-KEYS: 42;
- HG-AUDIT: 35;
- HG-MONEY: 20;
- HG-LAUNCH: 11;
- standing roles: 6.

Most of these entries say "not crossed", or "in scope later".

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]] · [[Decisions]] · App Plan QA Report
