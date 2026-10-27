# 01 — Reference Admin-Panel Specification (RealX)

> **EXTRAPOLATION.** This is a synthesis of 20 platform extrapolations, observed vendor admin surfaces and regulatory obligations. Aliases and the confidence scale are defined in `00-overview.md` §2–3; role codes are listed in §5 of that file (full catalogue in `02-rbac-and-approvals.md`); state names are defined in `03-workflows-and-state-machines.md`. Date: 23 Sep 2026.

**How to read each module.** Each module has:
- **Purpose**, **primary users**, **evidence**, **confidence**;
- a **pages** table: widgets/KPIs, main table columns, filters, row actions, bulk actions, and page confidence;
- an **action catalogue**: roles, maker-checker, on-chain call, state transition, audit events, notifications;
- **alerts & automations**.

---

## 0. Shared conventions

### 0.1 Privileged-action lifecycle (applies to every action marked M-C ≠ `—`)

`draft → submitted (maker) → prechecked (simulation / canTransfer / rule dry-run) → approved (n-of-m checkers) → queued_for_custody → signed (custody quorum) → broadcast → confirmed (≥ N blocks / registry ack) → reconciled → closed`

Side exits: `rejected`, `expired` (approval TTL, default 72h), `cancelled` (maker, before approval), `failed` (on-chain revert or registry reject, which opens a break in M11).

- The **ApprovalRequest ID** travels into the custody transaction note (e.g., Fireblocks external tx ID). The policy engine can then refuse any on-chain call that lacks an approved request [`APE` M15, `WLD` §12; I].
- **Prechecks are mandatory** for on-chain actions: ERC-3643 `canTransfer`/compliance simulation, ERC-7943 `canTransfer`, balance/allowance preflight (as in Brickken's `dividendDistribution` preflight [`eu/04-brickken` §5.3]), and a rule-change impact simulation (M10).

### 0.2 Maker-checker notation (column "M-C")

| Code | Meaning |
|---|---|
| `4E` | Four-eyes: one maker plus one different checker with the required role |
| `4E+Q` | Four-eyes in the app **plus** a custody quorum (MPC/multisig) for signing |
| `6E` | Maker plus two checkers from different functions (e.g., Compliance + Finance) |
| `SGL+R` | Single authorised actor, immediate effect, **mandatory post-hoc ratification** within the SLA (emergency pauses, halts) |
| `AUTO` | System executes on a rule, with sampling QA and an override path |
| `—` | No approval (read, draft, or low-risk edit; still audited) |

Thresholds are parameters (M27 → Approval policies). Defaults are listed in `02-rbac-and-approvals.md` §3.

### 0.3 On-chain notation (column "On-chain")

- **RealX default:** ERC-3643 (T-REX) with ERC-7943 views on an EVM chain [`TSD` §0.1–0.2].
- **Alternates** are shown where a studied platform used them:

| Code | Standard / platform |
|---|---|
| `ASA` | Algorand ASA (Lofty) |
| `XRPL` | XRPL issued tokens/MPT (DLD / Ctrl Alt) |
| `HTS` | Hedera Token Service (RedSwan) |
| `XLM` | Stellar (RedSwan) |
| `T22` | Solana Token-2022 |
| `1400` | ERC-1400/1644 (Hedera ATS) |
| `BSQ` | Blocksquare contracts |
| `BKN` | Brickken Dapp API |
| `7540` | Async vault |

Function mapping follows `TSD` §4. `—` means the action is off-chain only.

### 0.4 Audit events

- **Naming:** `<domain>.<object>.<verb>`, e.g., `token.mint.requested` → `.approved` → `.signed` → `.confirmed` → `.reconciled`.
- **Payload:**
  - actor and on-behalf-of (for `AGNT`, the "delegation on every write" pattern [`WLD` §8]);
  - approval ID, tenant, jurisdiction pack version;
  - before/after hash, tx hash or registry ack;
  - reason code, evidence attachment hashes;
  - IP/device.
- **Storage:** append-only and hash-chained. Retention follows the jurisdiction pack: US TA 6 years, EU AML 5 years, VARA classification 8 years, AU 7 years [`XCC` #44].
- **Exports and PII views** are themselves audited events (`data.export.created`, `pii.view.unmasked`).

### 0.5 Notifications and severity

- **Channels:** in-app inbox, email, chat (Slack/Teams) and pager (Sev1) for staff. Email, push, SMS and in-app feed for investors. Templates are versioned in M21.
- **Severity:**
  - **Sev1:** safety of assets or market integrity; page on-call within 15 min.
  - **Sev2:** regulatory clock or money at risk; 1h.
  - **Sev3:** operational; 1 business day.
  - **Sev4:** informational.

### 0.6 Common table behaviour

- Saved views and a column chooser.
- **As-of date (time-travel)** on any register-derived table, following Tokeny's "historical look-back" [`WLD` §2].
- PII is masked by default; unmasking is permissioned and audited.
- Export is a separate permission (`X` in the RBAC matrix).
- **Tenant/jurisdiction scoping** filter chips are always visible.
- Every row carries a deep link and an activity drawer showing that object's audit events.

---

## 1. Information architecture

### 1.1 Global chrome

- **Header:**
  - tenant switcher (A5) and legal-entity switcher;
  - environment badge (`PROD` / `SANDBOX`, cf. Brickken sandbox [`APE` M6]);
  - network status pills (per chain: RPC, indexer lag, paused contracts);
  - global search (investor, wallet address, tx hash, property, offering, certificate/TokenID, case #);
  - **Approvals** badge (M25);
  - **Alerts** bell (M01);
  - "Break-glass" button, visible only to `CCO`/`SECO`/on-call and opening the M09 emergency runbook.
- **Right drawer:** activity feed and linked approvals for the object in view.
- **Command palette** (keyboard) for power users; every command respects RBAC.

### 1.2 Sidebar navigation tree

```text
RealX Admin
├── Command Center (M01)
│   ├── Executive overview · Operations queues · Compliance posture
│   └── Chain & custody health · Treasury snapshot · Alerts centre
├── PIPELINE & PRODUCTS
│   ├── Issuers, SPVs & Entities (M02)   Entities · Licences & permissions · Sponsor KYB · Formation
│   ├── Properties & Assets (M03)        Pipeline · Asset register · Valuations · Supply calculator
│   └── Offerings & Deals (M04)          Offerings · Builder · Compliance review · Raise monitor
│                                         · Cap meters · Product governance (TMD/KIIS) · Waitlists
├── INVESTORS
│   ├── Investors / CRM (M06)            Directory · Investor 360 · Segments · Households/entities
│   ├── Onboarding Review (M07)          KYC/KYB queue · Accreditation & wholesale · Tests & suitability
│   │                                     · Classification (VARA/MiFID) · Periodic review
│   └── Subscriptions & Orders (M05)     Order blotter · Cooling-off · Allocation · Settlement
│                                         · Refunds · Redemptions · Recurring plans
├── ON-CHAIN & REGISTRY
│   ├── Wallets & Custody (M08)          Wallet registry · Whitelist requests · Custody vaults
│   │                                     · Signer policies · Gas sponsorship
│   ├── Token Operations (M09)           Token registry · Actions console · Transfer requests
│   │                                     & exceptions · Agents & roles · Emergency
│   ├── Compliance Rules (M10)           Rule packs · Token rule sets · Eligibility matrix
│   │                                     · Threshold monitors · Simulator
│   └── Register & Reconciliation (M11)  Cap table · Snapshots · 3-way recon · Registry sync · Breaks
├── MONEY
│   ├── Treasury & Client Money (M16)    Accounts & wallets · Pay-ins · Pay-outs · Escrow
│   │                                     · Client-money recon · Reserves & funds · Stablecoin ops
│   ├── Distributions & Corp. Actions (M12)  Income ledger · Runs · Waterfalls · Claims · Exits
│   └── Tax (M13)                        Tax profiles · Withholding · Forms · Transaction taxes
├── MARKETS
│   ├── Secondary Market (M14)           Venue control · Instruments · Order book / listings
│   │                                     · Windows · Trade blotter · Surveillance
│   └── Lending & Liquidity (M15)        Pools · Lending markets · Oracles · Liquidations · Repo
├── ASSET LIFE
│   ├── Property Operations (M22)        Rent roll · Managers · Work orders · Municipal & title
│   │                                     compliance · Insurance & tax calendar · Reserves
│   ├── Governance & Voting (M17)        Proposals · Meetings · Ballots · Execution tracker
│   └── Documents & E-sign (M18)         Templates · Envelopes · Disclosure library · Token docs
├── RISK & COMPLIANCE
│   ├── AML Cases (M20)                  Screening hub · Alerts · Cases · SAR/STR · Travel rule · BWRA
│   ├── Reporting & Filings (M19)        Calendar · Filings · Prudential · Regulator data room
│   │                                     · Investor/board reports
│   └── Support & Complaints (M23)       Tickets · Complaints register · External escalations
├── ENGAGEMENT
│   ├── Communications & IR (M21)        Announcements · Campaigns · Inbox · Marketing approvals
│   └── Partners & Referrals (M24)       Sponsors · Distributors · Advisers/clubs · Rewards ledger
└── PLATFORM
    ├── Access, Approvals & Audit (M25)  Users · Roles · SoD · Approval policies · Approvals inbox
    │                                     · Audit log · Break-glass
    ├── Integrations & API (M26)         Connectors · API keys · Webhooks · Agents (MCP) · Jobs
    └── Settings & Tenancy (M27)         Tenants · Jurisdiction packs · Fees · Feature flags
                                          · Branding & domains · Billing
```

### 1.3 Object URL scheme (deep links used in alerts, audit and notifications)

```text
/t/{tenant}/assets/{assetId}            /t/{tenant}/offerings/{offeringId}
/t/{tenant}/investors/{investorId}      /t/{tenant}/tokens/{chain}/{contract}
/t/{tenant}/approvals/{approvalId}      /t/{tenant}/cases/{caseId}
/t/{tenant}/distributions/{runId}       /t/{tenant}/recon/{breakId}
```

---

## 2. Module specifications

---

### M01 — Home / Command Center · Confidence **H**

**Purpose.** A single view of operational load, money at risk, compliance clocks and chain/custody health, with one-click drill-down into queues.

**Primary users:** `CCO`, `CFO`, heads of operations; everyone sees a role-filtered version.

**Evidence:**
- Securitize Control Panel: raise KPIs, "pledged vs funded", KYC progress, registrations chart, activity log [`APE` M3/M15 C#11; `WLD` §3].
- DigiShares top-level corporate dashboard over per-STO dashboards [`WLD` §1].
- Zoniqx "audit-ready operational dashboards" [`APE` M13].
- Platform §11s: Lofty ops dashboard (TVL, daily rent, PMM depth, failed payouts) [`us/01-lofty` §11 #1]; PRYPCO (funds raised today, waitlist, pending withdrawals, complaints SLA) [`mena/01-prypco` §11 #1]; Reental (KYC funnel 2,300 → 700, paymaster balance) [`eu/01-reental` §11 #1]; mogul growth dashboard [`us/05-mogul` §11 #1].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 1.1 | Executive overview | Tokenised value / AUM; capital raised (day/MTD/QTD); **pledged vs funded**; investors (registered → KYC-passed → funded); live offerings % funded; distributions paid MTD vs scheduled; realised vs projected yield; secondary volume and avg discount/premium to NAV; open recon breaks; approvals pending; compliance SLA % | Top offerings: offering, asset, % funded, days left, soft-cap status, pledged, funded | Tenant, entity, jurisdiction, date range, archetype product line | Open offering; open asset | Export board pack (PDF) | H |
| 1.2 | Operations queues | One tile per queue with count, oldest age and SLA breach: KYC, accreditation, whitelist requests, bank matches, withdrawals, transfer/recovery requests, DLD/RER sync pending, complaints, governance deadlines | Unified queue: item, queue, age, SLA due, owner, priority, linked object | Queue, owner, SLA state, priority | Assign, claim, open, snooze (reason) | Re-assign, bulk-claim | H |
| 1.3 | Compliance posture | Alerts by severity; sanctions hits pending; open cases and **SAR/STR clocks**; raise-cap meters ≥80%; holder-count monitors (12(g), 3(c)(1), s601ED); next 30 days of filings; licence-condition expiries; outside-TMD dealings | Clock list: obligation, object, due, owner, status | Region, regime, severity | Open, acknowledge | Export regulator pack | M |
| 1.4 | Chain & custody health | Per chain: RPC and indexer lag, pending/stuck tx, gas price; **paymaster balances** (days of runway); custody pending signatures and oldest age; paused contracts; oracle/valuation freshness; registry-sync backlog | Tx monitor: approval ID, action, chain, status (prepared/signed/broadcast/mined/failed), age, tx hash (cf. Brickken prepare/send/status [`eu/04-brickken` §11 A]) | Chain, status, action type | Retry (re-queue), cancel, open explorer | Re-broadcast batch (4E) | H |
| 1.5 | Treasury snapshot | Client money: bank vs ledger (should be 0 difference); stablecoin float; **payout funding requirement for the next 7 days** (Lofty alert pattern [`us/01-lofty` §11.4]); escrow by offering; pending withdrawals | Accounts: account, rail, currency, balance, ledger, diff, last recon | Currency, entity | Open recon | — | H |
| 1.6 | Alerts centre | Feed with severity, rule, object and owner; MTTA/MTTR | Alerts: ID, sev, rule, object, created, owner, state | Sev, module, state, owner | Ack, assign, escalate, snooze (reason), create incident | Bulk ack (Sev3–4 only) | H |

**Action catalogue**

| Action | Roles (maker → checker) | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Acknowledge / assign / snooze alert | Any owner-eligible role | — | — | Alert `new → acknowledged/assigned/snoozed` | `alert.ack`, `alert.assign`, `alert.snooze` | Assignee in-app |
| Declare incident | `CCO`, `SECO`, on-call `ENGO` | — | — | Incident `open` | `incident.declared` | Pager (Sev1), exec channel |
| Launch emergency pause (deep link) | `CCO`/`SECO`/pauser | `SGL+R` | ERC-3643 `pause` (see M09) | Token `active → paused` | `token.pause.executed` | Pager, regulators per runbook, investor banner |
| Export board pack | `CFO`, `CCO` | — | — | — | `data.export.created` | — |

**Alerts & automations.**
- Queue SLA breach: KYC 24h target [`eu/02-blocksquare` §11.4]; complaint acknowledgement at day 5 of 7 [`mena/01-prypco` §11.4].
- Treasury float below the next 7 days of payouts.
- Paymaster runway below 3 days.
- Stuck tx older than 30 min.
- Indexer lag above 5 min.
- Daily 07:00 digest per role; weekly exec pack.

---

### M02 — Issuers, SPVs & Entity Registry · Confidence **H**

**Purpose.** The legal backbone. It records every entity in the structure and what it is **licensed or authorised** to do:
- platform opco, BD, VARA broker-dealer, RE, ECSP, custodian affiliates;
- issuers and sponsors;
- SPVs: series LLCs, DAO LLCs, DIFC SPVs, unit sub-trusts, KSA funds.

It then gates product features on those licences.

**Primary users:** `LEGL`, `CCO`, `INVM`, `CFO`; `X-ISA` (own entities only).

**Evidence:**
- DigiShares multi-project/SPV dashboards and "SPV creation + bank account" step [`APE` M1 C#2–4].
- Securitize Issuer Management Panel [`APE` M1 C#11].
- Blocksquare Certified Partner vs Legal Issuer and corporate resolution [`APE` M1 C#16–20; `eu/02-blocksquare` §2.1].
- ERC-3643 `setOnchainID` [`TSD` §1.4].
- Licence & Permissions Registry [`XCC` #1–2].
- Platforms: Lofty DAO-LLC registry [`us/01-lofty` §11 #4]; mogul OA "chain is register" clause [`us/05-mogul` §2.2]; Stake SPV corporate secretarial [`mena/03-stake` §11 #9]; RealT's 165 entities (sprawl lesson) [`us/04-realt` §2.1].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 2.1 | Entities | Count by type; entities without bank account or officers; filings due in 30 days | Entity, type (opco/BD/RE/ECSP/issuer/SPV/series/sub-trust/DAO LLC/fund), jurisdiction, reg. no., parent, roles, licences, tax classification (C-corp/REIT/partnership/AMIT), linked assets, tokens, holders, status | Type, jurisdiction, status, tenant | Open, edit, add officer, link bank, start dissolution | Export register of entities | H |
| 2.2 | Entity detail | Tabs: Overview · Officers & registered agent (flag commercial nominee agents, cf. St. Louis case [`us/01-lofty` §2.1]) · UBO tree · Constitutional docs (OA/constitution/**corporate resolution + IPFS hash**) · Bank accounts · Tax IDs & elections (REIT/C-corp/AMIT) · On-chain identity (ONCHAINID) · Filings · Related-party register | Officers: name, role, KYC status, appointment date, expiry | — | Upload doc, set on-chain ID, record resolution | — | H |
| 2.3 | Licences & permissions | Licences by regulator; conditions and expiry; **activity → feature-flag map** (e.g., "order matching" is disabled unless an ATS/MTF/market licence or partner is linked [`XCC` #30]); CAR / authorised-rep / tied-agent register; IPA status (VARA IPA → full) | Licence, entity, regulator, number, activities, conditions, issued, expiry, linked flags, status | Regulator, status, expiring | Edit conditions, link feature flag, upload renewal | — | M |
| 2.4 | Sponsor / issuer KYB | Pipeline: applied → KYB → background check → fee agreement → approved | Sponsor, type, KYB status, background check, retained equity (Lofty seller ≥10% [`us/01-lofty` §1]), fee terms, deals, status | Status, country | Approve, request docs, reject | — | H |
| 2.5 | Formation workflow | Kanban per new vehicle: name reserved → formed → foreign registration → EIN/TIN → bank → OA signed → registry (DIFC register / ARSN sub-trust) → ready | Task list with owner and due date | Vehicle type, jurisdiction | Complete task, attach evidence | — | M |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Create entity / start formation | `LEGL`, `X-ISA` | — | — | Entity `draft` | `entity.created` | — |
| Activate entity (formation complete) | `LEGL` → `CCO` | `4E` | — | `forming → active` | `entity.activated` | Finance, TokOps |
| Link / change bank account | `TRSY` → `CFO` | `4E` | — | BankAccount `pending → verified` | `entity.bank.linked` | CFO; penny-test job |
| Set issuer on-chain identity | `TOKO` → `CCO` | `4E+Q` | ERC-3643 `setOnchainID` (token), ONCHAINID claim add; `XRPL` DID set; `HTS` token memo | Token config updated | `entity.onchainid.set` | TokOps |
| Record corporate resolution / OA version | `LEGL` → `CCO` | `4E` | Optional: hash anchored (IPFS CID in registry, cf. Blocksquare [`eu/02-blocksquare` §2.1]; ERC-1643 `setDocument`) | Doc `draft → effective` | `entity.resolution.recorded` | IR (if investor-facing) |
| Add / modify licence or condition | `CCO` → `LEGL` | `4E` | — | Licence `active/conditioned/expired` | `licence.updated` | Feature-flag owners |
| Map licence → feature flag | `SUPER` → `CCO` | `4E` | — | FeatureFlag `off → on` | `flag.licence_bound` | Product |
| Approve sponsor KYB | `KYCA` → `CCO` | `4E` | — | Sponsor `review → approved` | `sponsor.approved` | Sponsor email |
| Register related-party transaction | `LEGL` → `CCO` | `4E` | — | RPT `logged → approved` | `rpt.approved` | Board pack |
| Dissolve entity | `LEGL` → `CFO` + `X-RE` | `6E` | Burn residual tokens (M09) | `active → winding_down → dissolved` | `entity.dissolved` | Holders, regulators if needed |

**Alerts & automations.**
- Licence or condition expiry at T-90/-30/-7.
- **VARA issuer licence revocation risk if there is no issuance within 6 months** [`REG-ME` §2].
- Sandbox exit deadline (REGA 6–24 months testing) [`mena/04-saudi` §2.3].
- Entity missing a bank account, officers or tax election.
- Officer KYC expiry.
- More than N entities per sponsor (sprawl signal, RealT ⚑).
- Annual report/franchise tax calendar per state or country.
- Automation: licence change → recompute feature flags → notify affected modules.

---

### M03 — Properties & Assets · Confidence **H** (listing/valuation) / **M** (title, Shariah, municipal flags)

**Purpose.** Asset master data and the **asset lifecycle** from sourcing to wind-down, including diligence, **title verification**, valuations/NAV and eligibility flags on the asset (zone, foreign ownership, Shariah).

**Primary users:** `INVM`, `ASTM`, `VALO`, `LEGL`, `SHRO`; `X-VAL` (upload only); `X-PM` (read own assets).

**Evidence:**
- Vendors: Blocksquare property listing, asset valuation, buyback [`APE` M2 C#16–20]; DigiShares property presentation [`APE` M2 C#4]; ERC-7518 partitions per floor [`TSD` §1.6]; oracle/NAV valuation module [`VND` §10].
- Obligations: Asset Registry and Shariah [`XCC` #41, #43].
- Platforms: RealT ⚑ title & asset verification and deed-to-token reconciliation [`us/04-realt` §11 #12]; BrickX Brick Valuation formula [`australia/01-brickx` §11 #9]; Arrived buy-box and NAV committee [`us/03-arrived` §11 #4, #6]; PRYPCO DLD smart valuation and 10,000 tokens/sqm [`mena/01-prypco` §1, §5.7]; Saudi multi-stage vetting and REGA price controls [`mena/04-saudi` §11.1].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 3.1 | Pipeline (kanban) | Stage counts and conversion; avg days per stage; approval rate (mogul "<1%" [`us/05-mogul` §1]) | Card: address, type, ask price, target yield/IRR, sponsor, stage, stage age, blockers | Stage, market, sponsor, strategy (hold/flip/BMV) | Advance stage (gated), assign, reject (reason) | Re-assign | H |
| 3.2 | Asset register | AUM by market/type; occupancy; avg yield; assets with open compliance items | Asset ID, address, country/zone, type, SPV, token(s), acquisition price/date, latest valuation (date), NAV/token, LVR, occupancy, net yield, reserve months, PM, **title status**, insurance expiry, compliance score, lifecycle state | Market, state, PM, compliance score, zone flags | Open, publish valuation, flag legal hold | Export | H |
| 3.3 | Asset detail | Tabs: Overview · **Title & encumbrances** (registry ref: county/DLD/RER/Torrens; deed recorded to issuer ✓; title insurance; lis pendens) · Diligence (inspection, appraisal, environmental, rent-roll audit) · Valuations · Financials (links M22) · Tokens & supply · Eligibility flags (freehold zone, holy-city, FIRB residential, US state foreign-adversary) · Shariah screen · Property-transaction AML · Exit plan | Diligence checklist: item, owner, status, evidence, reviewer | — | Upload, request, approve item | — | H/M |
| 3.4 | Valuations workbench | Due and overdue list; deviation vs purchase / last / market trades; valuer independence check | Asset, method, valuer, value, date, NAV/token, Δ%, status (draft/committee/published), next due | Due window, Δ% | Compute NAV, submit to committee, publish | Schedule re-valuations | H |
| 3.5 | Supply & tokenomics calculator | Presets: 10,000 tokens per m² (DLD) · 100,000 units = 100% (Blocksquare BSPT) · 10,000 units per trust (BrickX) · €100 per token (Reental/Domoblock) · 1,000,000 shares (SmartCrowd) · custom | Tranches/partitions (floor, share class, Reg D vs Reg S) | — | Save as offering template | — | M |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Create asset / intake | `INVM`, `X-ISA` | — | — | Asset `sourced` | `asset.created` | — |
| Advance stage (gate checklist must be complete) | `INVM` | — | — | `sourced → screening → due_diligence` | `asset.stage.advanced` | Stage owner |
| Investment-committee approval | `INVM` → IC (2 of `CFO`/`CCO`/CEO) | `6E` | — | `due_diligence → ic_review → ic_approved` (then `→ title_verification`) | `asset.ic.approved` | Deal team |
| Record title verification | `LEGL` → `CCO` | `4E` | Optional hash anchor (ERC-1643 `setDocument`) | `title_verification → title_verified` | `asset.title.verified` | — |
| Link to SPV / registry ref | `LEGL` | — | — | `title_verified → spv_formed` | `asset.spv.linked` | — |
| Publish valuation / NAV | `VALO` → Valuation Committee (`CFO` + independent) | `6E` | Optional oracle push (e.g., Chainlink NAV feed); updates M14 band reference and M15 collateral price | Valuation `draft → published` | `valuation.published` | Market ops, risk, IR (announcement) |
| Shariah approval | `SHRO` (+ board fatwa upload) | `4E` | — | Screen `pending → compliant/non-compliant` | `asset.shariah.approved` | Offering owner |
| Flag distressed / legal hold | `ASTM` or `LEGL` → `CCO` | `4E` | May trigger M14 halt / M09 pause | `operating → distressed/legal_hold` | `asset.flagged` | IR, market ops |
| Propose exit / sale | `ASTM` | — | — | `operating → exit_proposed` (→ M17 vote) | `asset.exit.proposed` | Holders (via M21) |
| Record sale completion | `LEGL` → `CFO` | `4E` | Burn/redeem (M09) after waterfall (M12) | `under_sale → sold → wound_down` | `asset.sold` | Holders, tax |

**Alerts & automations.**
- Valuation older than 6 months [`australia/01-brickx` §11.4].
- Valuation Δ more than X% vs purchase [`mena/03-stake` §11.4].
- **Deed-to-token mismatch** ⚑ [`us/04-realt` §11.4].
- Title-insurance expiry.
- New lis pendens or court order (feed or manual).
- LVR above 45% (early warning ahead of a 50% cap) [`australia/01-brickx` §11.4].
- Affiliate/related-party acquisition premium requires independent reviewer [`us/03-arrived` §11].
- Zone-rule change (KSA designated zones) re-evaluates eligibility flags.
- Automation: publishing a valuation recomputes NAV/token, price-band references and collateral prices, and posts an investor announcement draft.

---

### M04 — Offerings & Deals · Confidence **H**

**Purpose.** Build, review, launch, monitor and close offerings. Each offering is **bound to a regime or exemption** that auto-loads its constraints: caps, investor classes, solicitation rules, resale rules and required documents [`XCC` #2].

**Primary users:** `CAPM`, `CCO`, `LEGL`, `X-BDP` (US), `X-RE` (AU), `SHRO`, `X-ISA`.

**Evidence:**
- Vendors: Securitize offering config (currencies, minimum, start date, price, subscription agreements) and fundraise-site CMS [`APE` M3 C#11–12]; Stobox caps and eligibility [`APE` M3 C#23]; Brickken public/private issuance and `newSto` soft/hard cap [`eu/04-brickken` §4]; Blocksquare finalise vs revert [`APE` M3 C#18]; InvestNext/Juniper deal rooms [`APE` M3].
- Obligations: [`XCC` #2–4, #33].
- Platforms: Arrived Tier 2 cap meter [`us/03-arrived` §11 #1]; PRYPCO offering builder with sqm calculator and Trakheesi permit [`mena/01-prypco` §11 #3]; Domoblock pre-sale tier [`eu/05-domoblock` §11 #5]; Assetora TMD library / SPDS [`australia/05-assetora` §11 #1–2].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 4.1 | Offerings | Live / scheduled / closed counts; capital raised; % funded distribution | Offering, asset, issuer/SPV, **regime** (506(b)/506(c)/Reg S/Reg A T1–2/Reg CF/ECSP/prospectus-exempt/MiFID/VARA ARVA/DFSA crowdfunding/AU registered MIS/s708 wholesale/REGA sandbox/CMA fund), jurisdictions, investor classes, target, min/max ticket, soft/hard cap, pledged, funded, investors, status, window | Regime, status, jurisdiction, channel | Open, clone, pause, extend, close | Export | H |
| 4.2 | Offering builder (9-step wizard) | Validation panel showing unmet constraints | Steps: (1) Asset & issuer; (2) **Regime selector** loads constraints; (3) Economics (price, supply, fees, hurdle/waterfall); (4) Raise mechanics (window, soft/hard cap, all-or-nothing vs finalise/revert, oversubscription rule, pre-sale tiers, waitlist, **per-investor cap** 20% PRYPCO / 15% Lofty / 5% BrickX); (5) Eligibility & distribution (jurisdiction × class matrix, target market/TMD, nationality/residency, distributors); (6) Token & rules (class/partition, chain, M10 rule set, lock-ups); (7) Documents (PPM/Form C/1-A/253G2/KIIS/PDS/SPDS/TMD/whitepaper, subscription template, risk disclosures, Trakheesi permit no., fatwa); (8) Page CMS by jurisdiction × domicile; (9) Review & submit | — | Save draft, simulate, submit | — | H |
| 4.3 | Compliance review queue | Pending reviews, age, reviewer load | Offering, regime, checklist %, legal opinion ✓, cap-meter ✓, BD/ESI/RE sign-off ✓, Shariah ✓, tenant guardrails ✓ | Reviewer, regime | Approve, request changes, reject | — | H |
| 4.4 | Live raise monitor | **Pledged vs funded**; registrations/day; funnel (visited → KYC → pledged → funded); soft-cap progress and ETA; ticket histogram; jurisdiction mix; waitlist; cap-breach blocks; escrow vs commitments | Commitments: investor, amount, status, reflection timer, funds, allocation | Status, jurisdiction | Open order, nudge investor | Send reminder (M21) | H |
| 4.5 | Raise-cap meters | Rolling 12-month meters per issuer and per Member State: Reg CF $5M · Reg A T2 $75M / T1 $20M · ECSP €5M per project owner · EU prospectus €12M (MS option €5M; ES €8M cited) · AU CSF $5M | Issuer, regime, window start, used, remaining, % | Regime, issuer | Open issuer | — | M |
| 4.6 | Product governance | TMD register (AU DDO) with review triggers; MiFID target market; KIIS versions; **outside-target dealings log** with a 10-business-day ASIC report clock; VARA retail vs qualified-only gating | Product, TMD version, review due, distribution conditions, dealings outside TMD, reports due | Region, due | Review TMD, log dealing, file report | — | M |
| 4.7 | Waitlists & tiers | Waitlist size (PRYPCO 10,700+ [`mena/01-prypco` header "Scale"]); tier eligibility (Domoblock ≥€50k or ≥5 projects) | Investor, tier, priority, invited | Tier | Invite, remove | Bulk invite | M |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Create / edit draft | `CAPM`, `X-ISO` | — | — | Offering `draft` | `offering.draft.saved` | — |
| Submit for compliance review | `CAPM`, `X-ISA` | — | — | `draft → compliance_review` | `offering.submitted` | Reviewers |
| Approve offering | `CCO` + `LEGL` (+ `X-BDP` US BD; `X-RE` AU; `SHRO` Islamic; platform `CCO` for tenant offerings) | `6E` | — | `compliance_review → approved` | `offering.approved` | Maker, TokOps |
| Schedule / go live | `CAPM` → `CCO` | `4E+Q` | Deploy/configure sale or escrow contract: `BKN` `newSto`; `BSQ` `OfferingsFixedPrice`; ERC-3643 compliance modules for offering phase; or off-chain only | `approved → scheduled → live` | `offering.live` | Waitlist/investors, IR |
| Pause offering | `CAPM` or `CCO` | `SGL+R` | Optional escrow pause | `live → suspended` | `offering.paused` | Investors with open orders |
| Extend window / material change | `CAPM` → `CCO` | `4E` | Update end date on escrow if on-chain | `live → live` (v+1). Triggers re-confirmation within 5 business days [`mena/01-prypco` §2.8] | `offering.amended` | Committed investors |
| Close offering | `AUTO` at end date / hard cap, or `CAPM` | `AUTO`/`4E` | `BKN` `closeOffer` | `live → closed` | `offering.closed` | Investors |
| Finalise & mint | `CAPM` → `CFO` + `CCO` | `6E`+Q | ERC-3643 `batchMint`; `ASA` transfer after opt-in; `XRPL` payment from issuer / MPT; `HTS` `TokenMint` + transfer; `BSQ` offering finalise | `closed → finalising → minted` | `offering.finalised`, `token.mint.*` | Investors (allocation notice), registry |
| Revert & refund (failed raise) | `CAPM` → `CFO` | `4E+Q` | Escrow refund (`BKN` claim/refund); off-chain refund batch | `closed → failed → refunding → refunded` | `offering.failed`, `refund.batch.*` | Investors, IR |
| Publish supplementary doc (SPDS/253G2/KIIS v+1) | `LEGL` → `CCO` | `4E` | Optional ERC-1643 `setDocument` | Doc `effective` | `offering.doc.published` | Investors (acknowledgement) |

**Alerts & automations.**
- Cap meter at 80/95/100% (hard block at 100%) [`XCC` #3].
- Soft cap at risk at T-3 days [`eu/04-brickken` §11.4].
- Hard cap reached.
- Stale financials for Reg CF offerings open past 30 April [`REG-US` §7].
- KIIS/PDS/TMD review due.
- Distribution outside TMD (ASIC report within 10 business days) [`REG-AU` §3].
- Trakheesi permit expiry [`REG-ME` §7].
- Automation: close at end date → allocation (M05) → finalise/revert proposal generated for approval.

---

### M05 — Subscriptions, Orders, Allocations & Redemptions · Confidence **H**

**Purpose.** Run primary orders from indication to settled holding:
- eligibility, investor limits and consent;
- cooling-off / reflection;
- funds matching and escrow;
- allocation (pro-rata / first-come / tiers);
- token delivery and refunds.

It also runs **redemption queues** for fund-like products and **recurring investment plans**.

**Primary users:** `CAPM`, `FINA`, `CFO`, `KYCA`, `SUPP`.

**Evidence:**
- Vendors: Brickken escrow with soft/hard cap, `newInvest`/`claimTokens`, refund on failure [`eu/04-brickken` §5.2]; Blocksquare revert-and-refund [`APE` M10 C#18]; ERC-7540 async request → claimable [`TSD` §1.5].
- Obligations: [`XCC` #10, #16–18].
- Platforms: PRYPCO 24h cooling-off and allocation [`mena/01-prypco` §11 #4]; Stake limits [`mena/03-stake` §11 #3]; BrickX pre-order and Smart Invest ≤50% [`australia/01-brickx` §5.2]; Reental proof-of-transfer reconciliation [`eu/01-reental` §11 #7]; Arrived fund redemptions [`us/03-arrived` §11 #10]; Propvest Sparplan [`eu/03-propvest` §11 #4].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 5.1 | Order blotter (primary) | Orders today; funded %; average time-to-fund; revocations | Order ID, investor, offering, qty/amount, payment asset/method, **eligibility result**, reflection/cooling-off timer, funds status, escrow ref, allocation, docs signed ✓, channel/distributor, flags (limit/cap/KYC stale) | Offering, status, flag, channel | Open, cancel, resend docs, mark funds received (→ M16 match) | Export | H |
| 5.2 | Order detail | Timeline; eligibility checks (KYC valid, classification, **investor-limit ledger** (Reg CF / Reg A 10% / DFSA AED 183.5k per property and 367k per year / KSA tranche), holding cap, ECSP €1,000 / 5% warning + consent captured, knowledge test valid); docs; payment match; escrow; allocation; delivery tx | — | — | Override (not allowed except documented exemption → `CCO` + `LEGL`) | — | H |
| 5.3 | Cooling-off & reflection | Timers expiring today; revocation rate | Order, regime timer (EU ECSP 4 days, PRYPCO 24h, AU CSF 5 business days, AU PDS 14 days), started, expires, state | Regime, expiring | Record revocation (investor instruction) | — | H |
| 5.4 | Allocation workbench | Oversubscription ratio; preview vs caps | Investor, requested, rule (pro-rata/FCFS/tier priority/waitlist), allocated, refund due, cap check | Offering | Adjust (documented), lock file | Generate allocation file | H |
| 5.5 | Settlement & delivery | Allocations awaiting mint; registry registrations pending (DLD/RER); certificates issued | Allocation, investor wallet/sub-account, qty, mint batch, tx hash / registry ack, certificate ID | Status | Retry, open tx | Create mint batch (→ M09) | H |
| 5.6 | Refunds | Refunds pending / paid / unclaimed | Order, reason (failed raise / revoked / over-allocation), amount, rail, status, age | Reason, status | Pay, retry, mark unclaimed | Refund batch (4E) | H |
| 5.7 | Redemptions | Queue vs gate; NAV strike date; pending → claimable | Request ID, investor, product, units, NAV, requested, state (`requested/pending/claimable/claimed`), gate % | Product, state | Approve, gate, reject | Process batch (strike NAV) | M |
| 5.8 | Recurring plans | Active plans; next run; failure rate | Plan, investor, amount, frequency, strategy (Smart Invest / Sparplan tier / AutoInvest theme), next run, last status | Status | Pause plan, retry | Run now (AUTO) | M |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Place assisted order | `SUPP` (on documented instruction) | `4E` (`KYCA`) | — | Order `submitted` | `order.created.assisted` | Investor confirmation |
| Eligibility check | System | `AUTO` | Precheck: ERC-3643 `canTransfer` for delivery; `isVerified` on identity registry | `submitted → eligibility_check → reflection_period/awaiting_funds/rejected` | `order.eligibility.evaluated` | Investor if rejected |
| Record revocation (reflection/cooling-off) | Investor, or `SUPP` on instruction | — | — | `reflection_period → revoked → refund_pending` | `order.revoked` | Investor, finance |
| Allocate | `CAPM` → `CFO` | `4E` | — | `funded → allocated` | `allocation.file.locked` | Investors (allocation notice) |
| Deliver tokens | `TOKO` → quorum | `4E+Q` | ERC-3643 `mint`/`batchMint` or `transfer` from issuer treasury; `ASA` transfer after opt-in (atomic group); `XRPL` payment into MPC omnibus; `HTS` grant KYC + transfer; `BKN` investor `claimTokens` | `allocated → settled` | `token.mint.*`, `order.settled` | Investor (holding, certificate) |
| Issue refund | `FINA` → `CFO` | `4E` (`4E+Q` if stablecoin) | Stablecoin transfer or escrow refund | `refund_pending → refunded` | `refund.paid` | Investor |
| Process redemption batch | `FINA` → `CFO` | `4E+Q` | `7540` fulfil (`requestRedeem` → claimable); ERC-3643 `burn` on claim | `pending → claimable → claimed` | `redemption.batch.*` | Investors |
| Gate redemptions | `CFO` → `CCO` | `4E` | — | Product `gated` | `redemption.gated` | Investors (notice) |

**Alerts & automations.**
- Funds not received within X days, then auto-cancel.
- Reflection or cooling-off expiring, then auto-advance to `awaiting_funds`.
- Allocation file differs from escrow.
- Refund unclaimed after N days [`eu/04-brickken` §11.4].
- Redemption queue exceeds gate.
- Recurring-plan payment failures [`mena/03-stake` §11.4].
- Order from a blocked jurisdiction or IP.
- Automations: investor-limit ledger update on settlement; auto-reinvest orders created from distributions (M12); ARVA redemption SLA timer (fee-free redemption) [`REG-ME` §2].

---

### M06 — Investors (CRM / Investor 360) · Confidence **H**

**Purpose.** One authoritative investor record. It joins identity, classification and limits, wallets, holdings, orders, payouts, tax, documents, cases and communications, and supports tokenised and non-tokenised holders on one register [`APE` M4 C#2].

**Primary users:** `SUPP`, `KYCA`, `IRCM`, `CCO`, `MLRO`; read-only views for most roles (PII masked).

**Evidence:**
- Off-chain comparators: InvestNext, Agora and Juniper CRM [`APE` M4 C#29–31].
- Securitize DS Registry Service (accreditation, qualification, confirmed wallets, hashed PII) [`APE` M4 C#13]; Blocksquare up to 10 wallets per account [`APE` M4 C#16].
- Platform §11s:
  - Reental Investor 360 (tiers, multi-wallet, loans) [`eu/01-reental` §11 #2];
  - Propvest Client 360 (MiFID profile, Mangopay, Baader, Tangany) [`eu/03-propvest` §11 #2];
  - PRYPCO (Emirates ID expiry, 20% cap usage, suspension flag) [`mena/01-prypco` §11 #2];
  - Lofty (California flag, W-8BEN, 15% cap utilisation) [`us/01-lofty` §11 #2].
- Life events ("death, divorce, lost digital asset wallet keys") are handled by the transfer agent [`WLD` §3].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 6.1 | Directory | Investors by state (registered/KYC'd/funded/dormant); by classification; by country | Investor ID, name/entity, type (individual/joint/SMSF/trust/company/IRA), residency, nationality, **classification** (retail/accredited/professional/qualified/wholesale), KYC status and expiry, risk rating, wallets, holdings value, positions, lifetime invested, last activity, tier/club/channel, flags (PEP, sanctions, suspended, estate, legal hold) | All columns; saved segments | Open 360, message, suspend | Add to segment, export (masked) | H |
| 6.2 | Investor 360 | Status chips (KYC, class, limits, tax form, flags). Tabs: Profile · Classification & **limit meters** (Reg CF 12-month, Reg A 10%, DFSA AED 183.5k/367k, holding caps per property) · **Wallets** (type: passkey/AA, external, omnibus sub-account; guardian status; KYT score) · Holdings (per token: balance, locked/frozen, lock-up end, % vs cap, as-of) · Orders · Payouts (history, failures, reinvest setting, payout asset) · Tax (forms, TIN, withholding rate) · Documents & signatures · Cases (AML/complaints/tickets) · Comms & consents (e-delivery, marketing, ECSP warnings) · Votes · Audit | Per-tab tables | Date range | Edit, request docs, open case | — | H |
| 6.3 | Segments & lists | Segment size; overlap | Segment, rule, size, owner, last used | — | Edit, clone | Use in campaign / waitlist | H |
| 6.4 | Entities & households | UBO completeness; joint and SMSF links | Entity, UBOs (≥25% + control), authorised signatories, linked personal records | — | Link, unlink | — | M |
| 6.5 | Life events & estates | Open estate/divorce/incapacity matters | Investor, event, evidence, legal status, linked transfer request (M09) | Status | Open request | — | M |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Edit contact / preferences | `SUPP` | — | — | — | `investor.profile.updated` | Investor confirmation |
| Edit identity fields (name, DOB, ID) | `SUPP` → `KYCA` | `4E` | — | May reopen KYC | `investor.identity.updated` | Investor |
| Change residency / nationality | `KYCA` → `CCO` | `4E` | ERC-3643 IdentityRegistry `updateCountry` | Re-evaluate eligibility (M10); may trigger `restricted` | `investor.residency.changed` | Investor, rules engine |
| Suspend account | `MLRO` / `CCO` | `SGL+R` | Optional ERC-3643 `setAddressFrozen` on all wallets (M09) | Investor `active → suspended` | `investor.suspended` | Investor (if no tipping-off risk), ops |
| Unsuspend | `MLRO` → `CCO` | `4E` (+Q if unfreeze) | `setAddressFrozen(false)` | `suspended → active` | `investor.unsuspended` | Investor |
| Merge duplicate records | `SUPP` → `KYCA` | `4E` | — | — | `investor.merged` | — |
| Record death / estate | `LEGL` → `CCO` | `4E` | Opens M09 transfer request | `active → estate` | `investor.estate.opened` | Executor |
| Offboard / close | `SUPP` → `CCO` | `4E` | `deleteIdentity` once balance is 0 | `active → offboarded` | `investor.offboarded` | Investor |
| Data-subject access export | `SUPP` → `CCO` | `4E` | — | — | `privacy.dsar.fulfilled` | Investor |
| Erasure / pseudonymisation | `LEGL` → `CCO` | `4E` | Addresses stay on chain; off-chain PII pseudonymised and the address ↔ person link sealed (GDPR vs immutability, see `07` EU) | Investor `erasure_restricted` (retention holds apply) | `privacy.erasure.processed` | Investor |

**Alerts & automations.**
- ID document expiry at T-30 (Emirates ID, passport) [`mena/01-prypco` §11.4].
- Holding approaching per-property cap [`us/01-lofty` §11.4].
- Dormant funded investors with unclaimed payouts.
- Duplicate-identity detection (same ID, different accounts).
- Tier recalculation on staking or activity change [`eu/01-reental` §11.4].

---

### M07 — Onboarding Review (KYC / KYB / accreditation / suitability / wholesale / classification) · Confidence **H**

**Purpose.** Decide who may invest, in what, and up to how much. The module turns vendor results and evidence into a **decision with a validity period**, and then into an **on-chain eligibility grant**: identity registry, claim or credential.

**Primary users:** `KYCA`, `CLSR`, `MLRO`, `CCO`.

**Evidence:**
- Vendors:
  - Securitize "KYC/AML progress tracking with tools to fix common issues" [`APE` M4 C#11];
  - Blocksquare auto-whitelist on KYC **or** admin confirmation [`APE` M4 C#17];
  - Tokeny "Identities" write domain, required claims and trusted claim issuers [`WLD` §2];
  - DS "Exchange" role registering investors [`APE` M4 C#13];
  - InvestNext accreditation letters [`APE` M4 C#30];
  - vendor surfaces for Sumsub, Entrust, IDWise, Equifax, iCapital, Verify Investor [`VND` §1–2].
- Obligations: [`XCC` #6–15].
- Platforms:
  - Brickken 6-state KYC (`NOT_REQUIRED/NOT_STARTED/PENDING/APPROVED/REJECTED/ON_HOLD`) [`eu/04-brickken` §2.5];
  - RedSwan accreditation queue with 90-day expiry [`us/02-redswan` §11 #3];
  - BrickX Equifax pass/refer/fail [`australia/01-brickx` §11 #1];
  - Saudi Nafath/Yakeen eligibility tiers [`mena/04-saudi` §11.1];
  - Stake Professional Client evidence [`mena/03-stake` §11 #2].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 7.1 | KYC/KYB queue | Queue depth; SLA (target 24h [`eu/02-blocksquare` §2.5]); auto-approve rate; sign-up → KYC conversion (Reental 2,300 → 700 [`eu/01-reental` §2.5]) | Case ID, applicant, individual/entity, jurisdiction, **provider** (Sumsub/Entrust/Persona/IDWise/Equifax/Nafath/UAE PASS), provider decision, risk score, hits (PEP/sanctions/adverse media), liveness, age, SLA due, assignee, state | Provider decision, risk, hits, SLA, jurisdiction | Claim, open, assign | Assign, bulk-approve **only** provider-approved low-risk cases with no hits (AUTO + QA sample) | H |
| 7.2 | Case review | Side-by-side document images vs extracted data vs entered data; liveness; address proof; **screening hits with adjudication**; risk-factor breakdown; **SoF/SoW** (VARA); EDD triggers; KYB **UBO tree (≥25% + control)** with registry cross-check; RFI composer; decision panel with reason codes | Hits: list, score, matched fields, decision | — | Approve, reject, RFI, escalate to EDD | — | H |
| 7.3 | Accreditation & wholesale | Expiring in 30 days; verification method mix | Investor, **regime/test** (US 506(c) accredited; EU professional opt-up 2 of 3; AU wholesale s761G / sophisticated s761GA; VARA Qualified AED 3.5M NA or AED 700k income; DFSA Professional; KSA qualified), **method** (3rd-party letter CPA/attorney/RIA/BD; income/net-worth docs; **min-investment path $200k/$1M + written reps + no third-party financing** [`REG-US` §3]; accountant certificate), evidence, reviewer, valid_from, **valid_until** (US letter 90d default [I]; AU certificate 2y; EU annual review; VARA periodic), state | Regime, method, expiring | Approve, reject, request renewal | Send renewal reminders | H |
| 7.4 | Tests & suitability | Pass rates; retest lockouts active | Investor, test (ECSP entry test + loss simulation; MiFID appropriateness; VARA suitability; Reg BI profile), version, score, result, **retest-lockout until** (VARA ≥1 week), taken at | Test, result | View answers, reset (documented) | — | M |
| 7.5 | Classification registry | Investors by class × jurisdiction; product-access map | Investor, jurisdiction, class, basis, valid_until, products accessible | Class | Reclassify | — | M |
| 7.6 | Periodic review | Due by risk tier (high 1y / medium 2y / low 3y [I]); credential expiries (VCs) | Investor, last review, due, trigger (expiry, risk change, list update), state | Due window | Start review | Bulk-schedule | M |
| 7.7 | Eligibility grant handoff | Approved but not yet registered on-chain; failures | Investor, wallet(s), tokens requested, grant type (identity registration / claim / credential / trustline / whitelist), tx status | Status | Retry | Batch register (AUTO) | H |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Approve KYC (standard risk) | `KYCA` | `AUTO` or `—` (QA sample 5–10% [I]) | — | KYCCase `manual_review → approved` | `kyc.case.approved` | Investor |
| Approve KYC (high risk / override provider reject) | `KYCA` → `MLRO` | `4E` | — | `manual_review → approved` | `kyc.case.override` | — |
| Reject / request info | `KYCA` | — | — | `→ rejected` / `→ rfi_pending` | `kyc.case.rejected` / `.rfi` | Investor |
| Escalate to EDD; approve PEP/high-risk | `KYCA` → `MLRO` + senior manager | `6E` | — | `→ edd_review → approved/rejected` | `kyc.edd.decided` | — |
| Clear screening hit as false positive | `KYCA` → `MLRO` | `4E` | — | Hit `open → cleared` | `screening.hit.cleared` | — |
| Confirm true sanctions match | `MLRO` | `SGL+R` | Freeze any wallets (M09) | Investor `→ restricted`; case opened (M20) | `screening.hit.confirmed` | **No tipping-off**; compliance only |
| Approve accreditation / wholesale / qualified status | `CLSR` (→ `CCO` for min-investment path or opt-up) | `—` / `4E` | Issue claim: ONCHAINID claim (accreditation topic) by trusted issuer; `XRPL` `CredentialCreate`; Redbelly/Averer VC | AccreditationRecord `under_review → verified` | `accreditation.verified` | Investor (validity date) |
| Expire / revoke accreditation | System / `CLSR` | `AUTO` | Revoke claim/credential | `verified → expired/revoked` | `accreditation.expired` | Investor (T-30 reminder earlier) |
| Record test result / start lockout | System | `AUTO` | — | Test `passed/failed(lockout)` | `suitability.test.recorded` | Investor |
| **Grant on-chain eligibility** | `SVCA` after approval | `AUTO` (policy-limited identity-agent key) | ERC-3643 `registerIdentity(wallet, onchainID, country)` + KYC claim; `ASA` opt-in + unfreeze; `HTS` `TokenGrantKyc`; `XLM` `SetTrustLineFlags` authorise; `XRPL` `MPTokenAuthorize` / credential; `BSQ` `addWalletList`; `BKN` `whitelist` | Case `approved → whitelisted` | `eligibility.granted` | Investor ("ready to invest") |
| Revoke on-chain eligibility | `MLRO` / `CCO` | `4E` | `deleteIdentity` / claim removal / `TokenRevokeKyc` / deauthorise / `removeWalletList` | `whitelisted → revoked` | `eligibility.revoked` | Investor (unless tipping-off) |

**Alerts & automations.**
- KYC backlog past SLA.
- Provider outage (queue spike of `PENDING`).
- Accreditation or certificate expiring at T-30 (US 90-day letters; AU 2-year certificates).
- Sanctions list update triggers re-screening of the whole base.
- VARA suitability fail starts a 7-day lockout.
- Emirates ID/passport/Iqama expiry.
- Credential revoked for a holder [`australia/02-redbelly` §11.4].
- Tenant configured with KYC disabled in production [`eu/04-brickken` §11 A] → Sev2 to platform `CCO`.
- Automation: `approved` → eligibility-grant job. The whitelist mode is configurable per tenant (auto vs admin confirmation, Blocksquare pattern).

---

### M08 — Wallets & Custody · Confidence **H**

**Purpose.**
- Map every address to an owner and purpose.
- Prove ownership of self-hosted wallets.
- Screen wallets (KYT).
- Manage custody vaults and signer policies (the **maker-checker substrate**).
- Operate omnibus sub-accounts (A3) and gas sponsorship (A1).

**Primary users:** `SECO`, `TRSY`, `TOKO`, `MLRO`, `ENGO`, `X-CUS`.

**Evidence:**
- Vendors:
  - Fireblocks policy engine (amount/destination/asset/time rules, quorum, admin quorum, **pending-approvals dashboard**, authorisation history) [`APE` M10/M15 C#26; `WLD` §12];
  - Taurus function-level governance [`WLD` §13];
  - custody integrations: Fireblocks, Dfns, AWS KMS [`APE` M16 C#25];
  - KYT vendors and Notabene self-hosted-wallet proofs [`VND` §3];
  - paymasters and EIP-7702 [`VND` §4; `TSD` §1.8].
- Obligations: [`XCC` #21, #28].
- Platforms:
  - Reental wallet registry and paymaster/gas treasury [`eu/01-reental` §11 #4, #15];
  - RealT whitelist manager and AA-wallet Guardians [`us/04-realt` §11 #2, §5.1];
  - Ctrl Alt MPC policy, signer quorum and key ceremonies [`mena/02-ctrl-alt` §11 #8];
  - mogul Fireblocks-policy minting [`us/05-mogul` §11 #4];
  - PRYPCO warm MPC omnibus [`mena/01-prypco` §2.4].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 8.1 | Wallet registry | Wallets by type; high-KYT-risk count; unverified self-hosted | Address, chain, **type** (passkey/AA embedded, external EOA, omnibus sub-account, custodian vault, pool/contract, treasury), owner, **ownership proof** (signature / Satoshi test; required above €1,000 under EU TFR; UAE unhosted EDD), identity-registry status, KYT score/provider, frozen, first/last seen, holdings | Type, risk, proof, chain | Open, request proof, block, open in explorer | Re-screen (KYT) | H |
| 8.2 | Whitelist requests | Requests today; per-issuer holder counter vs cap | Request, investor, wallet, token/property, requested at, holder count after, state | Token, state | Approve, reject | Batch approve (≤ cap) | M |
| 8.3 | Custody vaults | Balances by vault; pending signatures; policy coverage | Vault, **purpose** (owner/admin, token-agent, pauser, identity-agent, distribution, treasury, paymaster, escrow), custodian (Fireblocks/BitGo/Ripple Custody/Zodia/Tangany/Coinbase/Dfns), chain, balance, policy set, quorum, destination allow-list, last activity | Custodian, purpose | Open, propose policy change | — | H |
| 8.4 | Signer policies | **Pending approvals** (policy-engine mirror); signer availability; policy drift vs approved baseline | Rule, scope (vault/asset/function), amount band, destinations, time window, approvers/quorum, admin-quorum-required, version | Vault | Propose change, view history | — | H |
| 8.5 | Omnibus sub-ledger (A3) | Omnibus on-chain balance vs sum of sub-accounts (should be 0 difference) | Sub-account, investor, token, balance, last movement, attestation hash | Token | Generate investor attestation | — | M |
| 8.6 | Gas sponsorship | Paymaster balance and **runway days**; sponsored tx/day; cost per active user; per-user caps hit | Policy, allowed contracts/functions, per-user daily cap, budget, spend, status | Chain | Top up, pause sponsorship | — | M |
| 8.7 | Smart-wallet recovery | Open recovery requests; guardian status | Request, investor, wallet, method (guardian/passkey re-bind), re-verification ✓, state | State | Approve, reject | — | M |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Request / verify wallet ownership | Investor; `SUPP` | — | Signed message or micro-transfer (Satoshi test) | Wallet `unverified → verified` | `wallet.ownership.verified` | Investor |
| Block wallet (KYT high risk / sanctions) | `MLRO` | `SGL+R` | Freeze via M09 (`setAddressFrozen`) and deny-list in policy engine | Wallet `→ blocked` | `wallet.blocked` | Compliance |
| Approve whitelist batch over holder threshold | `KYCA` → `CCO` | `4E` | As M07 grant | Request `→ approved` | `whitelist.batch.approved` | Investors |
| Create vault / assign purpose | `SECO` → admin quorum (`CFO`, CEO) | `6E` | Custodian vault creation | Vault `active` | `custody.vault.created` | — |
| Change signer policy | `SECO` → **admin quorum 3-of-5** | `6E+Q` | Policy-engine update. Chain analogues: Safe `changeThreshold`; `XRPL` `SignerListSet`; `HTS` KeyList `TokenUpdate`; `XLM` `SetOptions` weights | Policy `v → v+1` | `custody.policy.changed` | All signers, `CCO` |
| Add / remove signer or device | `SECO` → admin quorum | `6E+Q` | As above | — | `custody.signer.changed` | Signers |
| Allow-list a withdrawal destination | `TRSY` → `CFO` + `SECO` | `6E` | Policy-engine address book | Destination `pending → allowed` | `custody.destination.allowed` | — |
| Record key ceremony | `SECO` + `X-CUS` witness | `4E` | — | — | `custody.ceremony.recorded` | — |
| Paymaster top-up | `AUTO` within cap; `TRSY` → `CFO` above cap | `AUTO`/`4E+Q` | ERC-4337 paymaster deposit [I] | — | `gas.topup.executed` | Treasury |
| Change gas-sponsorship policy | `ENGO` → `CFO` | `4E` | Paymaster policy config | — | `gas.policy.changed` | — |
| Approve smart-wallet recovery | `SUPP` → `KYCA` (re-verification) → `CCO` | `6E` | Guardian recovery / passkey re-bind. If tokens must move, open M09 recovery | Request `→ executed` | `wallet.recovery.approved` | Investor |

**Alerts & automations.**
- Pending signature older than X min [`mena/02-ctrl-alt` §11.4].
- Signer unavailable (quorum at risk).
- Policy drift detected.
- KYT high-risk inbound transfer to a platform address.
- Paymaster runway below 3 days [`eu/01-reental` §11.4].
- Omnibus ≠ Σ sub-accounts.
- Self-hosted wallet above €1,000 without ownership proof (EU TFR).
- Unhosted-wallet EDD required (UAE AED 3,500 aggregate) [`REG-ME` §4].
- Automation: nightly KYT re-screen of all whitelisted wallets; a sanctions-list update triggers wallet re-screening.

---

### M09 — Token Operations · Confidence **H**

**Purpose.** Every privileged state change on the token:
- deploy/configure;
- mint/burn;
- freeze (wallet/partial) and pause;
- forced transfer;
- lost-wallet recovery;
- agent/role management, documents, upgrades and migrations;
- an **exceptions desk** for court orders, estates, divorces and error corrections.

**Primary users:** `TOKO`, `CCO`, `CFO`, `LEGL`, `SECO`, `X-TA`, `X-CUS`.

**Evidence:**
- Tokeny token actions (Mint, Burn, Freeze/Block, Unfreeze, Pause, Force Transfer, **Recovery**, batch; agent signs with own wallet) [`WLD` §2; `APE` M6 C#7–9].
- DS Protocol roles (Master/Issuer/Exchange/Transfer Agent, `seize`, locks, pause) [`WLD` §3].
- Hedera ATS roles (Admin/Minter/Controller/Pauser/KYC Manager) and actions [`WLD` §11].
- Brickken "Confiscation / edits", `mintToken`/`burnToken`/`transferFrom` [`eu/04-brickken` §4].
- Stobox emergency controls [`APE` M6 C#22].
- Securitize "destroy and reissue" for lost keys [`WLD` §3].
- ERC-3643 function set [`TSD` §1.4]; cross-chain mapping [`TSD` §4].
- Audit issue "enforce the pause on forced transfers" [`VND` §12].
- Platforms: [`XCC` #25]; RedSwan Token Studio (HTS keys, Rule 144 lock dates) [`us/02-redswan` §11 #5]; Lofty ASA control [`us/01-lofty` §11 #5]; Ctrl Alt issuance console [`mena/02-ctrl-alt` §11 #3]; RealT bridge and REG upgrade roles [`us/04-realt` §11 #3, #9]; Saudi exceptions desk [`mena/04-saudi` §11.1 #10].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 9.1 | Token registry | Tokens by state; paused count; supply vs cap; tokens with pending actions | Token/class, asset, SPV, chain, standard, address(es) (same-address multi-chain via CREATE2 [`WLD` §2]), supply/cap, holders, **paused**, compliance and identity-registry addresses, **owner** (multisig), agents, upgradeability (non-proxy/UUPS/1967), audit report, state | Chain, standard, state | Open, pause, new action | — | H |
| 9.2 | Token detail | Supply history; holders (→ M11); rules (→ M10); documents (ERC-1643); roles; pending actions | Action history: action, amount, target, approval ID, tx, status | Action type | Open action | — | H |
| 9.3 | Actions console | **Pending maker-checker panel**; simulation/precheck result; custody signing status; estimated gas | Action draft: type, token, targets (single/batch CSV), amounts, reason code, evidence, legal basis | — | Submit, cancel, approve (if checker) | Upload batch CSV | H |
| 9.4 | Transfer requests & exceptions desk | Open requests by type and age; timelocks running | Request, **type** (lost wallet recovery / forced transfer: court order, estate, divorce, error correction, sanctions / **transfer consent** for non-transferable positions (mogul, Domoblock assignment) / **conditional transfer or DvP approval** (Tokeny)), parties, qty, evidence ✓, legal review ✓, identity re-verification ✓, timelock ends, state | Type, state | Open, request evidence, approve (role-gated) | — | H |
| 9.5 | Agents & roles | On-chain role holders and scopes (Tokeny "Identities" vs "Assets" write domains) | Role (Owner/Agent/Minter/Controller/Pauser/KYC Manager/Transfer Controller/Allowance Controller), holder (multisig/MPC/service), scope, added by/at, approval ID | Token, role | Propose add/remove | — | H |
| 9.6 | Token studio (deploy) | Template library: ERC-3643, ERC-7518, CMTAT, ATS 1400+3643, HTS, ASA, XRPL MPT, Token-2022 (cf. Taurus template library [`WLD` §13]) | Wizard fields (Tokeny): name, symbol, decimals, network, logo, instrument type, base currency, owner wallet; identity eligibility (required claims, trusted issuers); compliance (jurisdictions, supply cap, balance limit, transfer type simple/conditional/whitelisting, limits per period, fees); **sandbox deploy first** | — | Save, deploy to sandbox, submit prod deploy | — | H |
| 9.7 | Upgrades & migrations | Proxy admin, timelock queue | Proposal, contract, new implementation, audit ✓, timelock ETA, state | — | Approve, cancel | — | M |
| 9.8 | Emergency | Break-glass state; runbooks; paused tokens; last drills | Runbook, trigger, owner, last tested | — | Execute runbook step | Global pause (all tokens) | M |

**Action catalogue** (default ERC-3643; alternates in brackets)

| Action | Roles (maker → checker) | M-C | On-chain call | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Deploy token | `TOKO` → `CCO` + `SECO`; owner quorum | `6E+Q` | T-REX factory deploy + `setIdentityRegistry`, `setCompliance`, `setOnchainID` [`ASA` `AssetConfig` with manager/reserve/freeze/clawback and default-frozen · `XRPL` `MPTokenIssuanceCreate` (RequireAuth, CanLock, CanClawback) · `HTS` `TokenCreate` with KYC/freeze/wipe/pause/supply keys · `T22` mint + extensions · `BKN` `newTokenization` · `BSQ` `createNewPropTokenForCPAndLI`] | Token `designed → deployed → configured` | `token.deploy.*` | TokOps, `CCO` |
| Mint / batch mint | `TOKO` → `CFO` (+`CCO` if not from a locked allocation file) | `4E+Q` | `mint` / `batchMint` [`ASA` transfer from reserve · `XRPL` Payment from issuer · `HTS` `TokenMint` · `BKN` `mintToken`] | Supply ↑; allocation `→ settled` | `token.mint.*` | Investors (M05) |
| Burn / redeem | `TOKO` → `CFO` | `4E+Q` | `burn` / `batchBurn` [`1400` `redeem`/`controllerRedeem` · `ASA` clawback to reserve · `XRPL` `Clawback` · `HTS` `TokenBurn`/`TokenWipeAccount` · `BKN` `burnToken`] | Supply ↓ | `token.burn.*` | Holder |
| Freeze wallet | `MLRO`/`CCO` (sanctions: `SGL+R`); otherwise `CCO` → `LEGL` | `SGL+R` / `4E+Q` | `setAddressFrozen(addr,true)` [`7943` `setFrozenTokens` · `ASA` `AssetFreeze` · `XRPL` MPT holder lock / TrustSet freeze · `HTS` `TokenFreezeAccount` · `XLM` deauthorise · `T22` `freeze_account`] | Holding `active → frozen` | `token.freeze.*` | Compliance (holder notice per tipping-off rules) |
| Partial freeze / lock | `CCO` → `LEGL` (lock-ups: `REGO` → `CCO`) | `4E+Q` | `freezePartialTokens` [`7518` `lockTokens` · ATS locks · `7943` `setFrozenTokens(amount)`] | Holding `partially_frozen` | `token.freeze_partial.*` | Holder |
| Unfreeze | `MLRO` → `CCO` | `4E+Q` | `setAddressFrozen(false)` / `unfreezePartialTokens` | `frozen → active` | `token.unfreeze.*` | Holder |
| **Pause token** | `CCO` / `SECO` / pauser | `SGL+R` (ratify ≤ 2h) | `pause` [`T22` pausable · `HTS` `TokenPause` · `XRPL` global lock · DigiShares `isTradingAllowed=false`] | Token `active → paused` | `token.pause.executed` | Pager, M14 halt, investor banner, regulator per runbook |
| Unpause | `CCO` → `SECO` + `CFO` | `6E+Q` | `unpause` | `paused → active` | `token.unpause.*` | As above |
| **Forced transfer** (court order / estate / divorce / error / sanctions directive) | `LEGL` → `CCO` + `CFO` (+ `X-TA` where an external TA is appointed) | `6E+Q`; **48h timelock** unless the court order requires immediacy | `forcedTransfer` / `batchForcedTransfer` [`1400` `controllerTransfer` · `7943` `forcedTransfer` · `7518` `forceTransfer` · DS `seize` · `ASA` clawback · `XRPL` `Clawback` + re-issue · `HTS` `TokenWipeAccount` + transfer · `XLM` `Clawback` · `T22` permanent delegate `transfer_checked`] | TransferRequest `approved → executed`; register updated (M11) | `token.forced_transfer.*` (with evidence hashes) | Both holders; `X-TA`; regulator if required |
| **Recover lost wallet** | `SUPP` intake → `KYCA` re-verification → `CCO` → quorum | `6E+Q`; 48h timelock plus holder notice to old contact channels | `recoveryAddress(lost, new, onchainID)` (freezes old, re-issues to new) [others: forced transfer / "destroy and reissue"] | `requested → … → executed` | `token.recovery.*` | Investor (all channels) |
| Add / remove agent or role | `SECO` → owner quorum (3-of-5) | `6E+Q` + 48h timelock | `addAgent` / `removeAgent` (ERC-3643 `IAgentRole`, called by the ERC-173 owner) [`HTS` key update · `XRPL` `SignerListSet` · `ASA` rekey or role change (**irreversible if set empty**)] | Role set `v+1` | `token.role.*` | All signers, `CCO` |
| Attach / update token document | `LEGL` → `CCO` | `4E+Q` | ERC-1643 `setDocument` [ATS documents · `XRPL` MPT metadata (mutable only once the DynamicMPT amendment is enabled; not enabled on mainnet as of 23 Sep 2026)] | Doc `effective` | `token.document.set` | Holders (if material) |
| Change compliance / identity-registry pointer | `TOKO` → `CCO` + `SECO` | `6E+Q` | `setCompliance`, `setIdentityRegistry` | Token `reconfigured` | `token.config.changed` | `CCO` |
| Approve conditional transfer / DvP | `REGO` or `MKTO` | `4E` below threshold: single approver + rule | Conditional-transfer approval (Tokeny) [`XLM` SEP-8 approval server] | TransferRequest `pending → approved` | `transfer.conditional.approved` | Parties |
| Approve transfer consent (non-transferable positions) | `IRCM` → `LEGL` | `4E` | Off-chain assignment. Mirror mint/burn on the "fan-token" ledger (Domoblock) | `requested → approved` | `transfer.consent.approved` | Parties |
| Contract upgrade | `ENGO` → `SECO` + `CCO`; owner quorum | `6E+Q` + **72h timelock** | Proxy `upgradeTo` (UUPS/1967) | Implementation `v+1` | `token.upgrade.*` | Public notice (material) |
| Bridge / multi-chain supply move | `TOKO` → `CFO` + `SECO` | `6E+Q` | Lock/burn-and-mint or CREATE2 same-address deploy | Per-chain supply ledger updated | `token.bridge.*` | — |
| Renounce powers (irreversible) | Board resolution + `CCO` + `SECO` + CEO | `6E+Q` | Renounce ownership / `ASA` empty role / `XLM` AUTH_IMMUTABLE | Irreversible | `token.powers.renounced` | Public notice |

**Guardrails.**
- Forced transfers are blocked while a token is paused, unless the override is explicitly approved. This mitigates the known T-REX issue "enforce the pause on forced transfers" [`VND` §12].
- Mints must reference a locked allocation file (M05) or an approved corporate action (M12).
- Every recovery or forced transfer needs an evidence bundle (court order, grant of probate, death certificate, identity re-verification, police report for theft) and a legal-basis field.

**Alerts & automations.**
- On-chain supply ≠ register (M11) → Sev1.
- Mint without an allocation reference attempted.
- Token paused more than X hours without ratification.
- Owner/agent change executed.
- Failed or reverted tx.
- Timelock ending in 1h.
- Unusual volume of forced transfers.
- Automation: a daily drill checks that the pauser key can sign on testnet and that runbooks were tested in the last 90 days.

---

### M10 — Compliance Rules Engine · Confidence **H**

**Purpose.** One **jurisdiction-parameterised** rules engine [`XCC` design principle]. It covers:
- regime packs with versioned parameters;
- per-token rule sets compiled to on-chain compliance modules;
- off-chain eligibility rules (nationality × residency × class × asset zone);
- threshold monitors (holder counts, concentration, foreign %);
- a **simulator** that must run before any rule change.

**Primary users:** `CCO`, `LEGL`, `TOKO`, `REGO`.

**Evidence:**
- Tokeny compliance rules: jurisdictions, supply cap, investor balance limit, transfer types (simple/conditional/whitelisting, mutually exclusive), limits per period, transfer fees [`WLD` §2].
- DS Compliance Service parameters: max US accredited, max retail per EU country, lock-ups per investor type, vesting, Reg S block, affiliates, `preTransferCheck` codes [`WLD` §3].
- DigiShares ERC-1404 per-address buy/sell restriction dates and `isTradingAllowed` [`WLD` §1].
- ERC-7518/7943 [`TSD` §1.6–1.7]; Taurus function-level governance [`APE` M5 C#28]; ATS "Customize" layer [`APE` M5 C#25].
- Obligations: [`XCC` #15, #26–27, §C seed values].
- Platforms: RealT 2,000-holder 12(g) cap [`us/04-realt` §2.3]; BrickX 5% cap (landholder duty/FIRB) [`australia/01-brickx` §2.6]; Saudi transfer-time eligibility and holy-city rule [`mena/04-saudi` §2.5]; Ctrl Alt transfer rules engine and rule-change log [`mena/02-ctrl-alt` §11 #6].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 10.1 | Jurisdiction rule packs | Packs in force; pending versions; law-change watchlist | Pack (e.g., `US-506c`, `US-RegS`, `US-RegA-T2`, `US-RegCF`, `EU-ECSP`, `ES-Ley6/2023`, `DE-eWpG`, `AE-VARA-ARVA`, `AE-DLD`, `AE-DIFC-CF`, `SA-REGA-sandbox`, `SA-CMA-fund`, `AU-MIS-retail`, `AU-wholesale`, `AU-CSF`), version, effective date, key parameters (travel-rule trigger, qualified thresholds, retail caps, resale lock, holder trigger, retention, cooling-off), approver | Region, status | Open, propose version | — | H |
| 10.2 | Token rule sets | Tokens per pack; rule-change frequency | Token, pack, modules enabled (country allow/deny; max holders; max % / balance limit; supply cap; lock-ups by investor type; vesting; transfer limit per period; transfer fee; **transfer mode** simple/conditional/whitelisting; Reg S → US block; affiliate restrictions; buy/sell restriction dates; trading switch; required claims; trusted issuers), on-chain module addresses, version | Token, module | Edit (→ simulator), view on-chain | — | H |
| 10.3 | Eligibility matrix | Coverage heat-map; overrides count | Rule: nationality × residency × investor class × asset country/zone × property type → allow/deny/condition. Examples: **KSA holy cities → Saudi companies and Muslim individuals only**; **FIRB established-dwelling ban for foreign persons (1 Apr 2025 – 30 Jun 2029)**; US state foreign-adversary laws; DLD Emirates-ID holders; Stake excluded-nationality list; EU non-EU buyer permits | Region, rule type | Edit, add override (investor-specific) | Import list | M |
| 10.4 | Threshold monitors | Per SPV/trust: holders of record vs **12(g) 2,000 / 500 non-accredited**; **3(c)(1) 100 beneficial owners**; AU **s601ED > 20 members**; **RETT share-deal** (e.g., DE 90% within 10 years); AU **landholder-duty significant interest** (e.g., NSW 20% private unit trust) and foreign-trust % (e.g., NSW ≥ 50%); holding caps | Vehicle, metric, value, threshold, % used, projected (after open orders), block active | Metric, % used | Set threshold, enable pre-trade block | — | M |
| 10.5 | Simulator | Impacted holders; would-be violations; blocked open orders; required investor notices | Scenario, change, affected holders, blocked transfers, recommendation | — | Run, attach to change request | — | M |
| 10.6 | Restriction-code dictionary | Codes in use | Code (ERC-1404 / EIP-1066 / T-REX reason), message (investor-facing), internal meaning | — | Edit message (IR review) | — | L |
| 10.7 | Decision log | Decisions/day; deny rate by reason | Time, from, to, token, qty, decision, reason codes, pack version | Reason, token | Open related order | Export sample | M |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Propose pack version | `CCO` | — | — | Pack `draft` | `rulepack.drafted` | `LEGL` |
| Publish pack version | `CCO` + `LEGL` | `6E` | — | Pack `approved → effective(date)` | `rulepack.published` | Tenants/modules using the pack |
| Add / remove compliance module on token | `TOKO` → `CCO` | `4E+Q` | ModularCompliance `addModule` / `removeModule` (T-REX reference implementation `IModularCompliance`; the EIP itself specifies only the `Compliance` interface) [`7518` `restrictTransfer` · `XRPL` `PermissionedDomainSet` · `T22` transfer-hook update · ATS customize] | Rule set `v+1` | `rules.module.changed` | Holders if material |
| Change rule parameter (cap, country list, limit) | `CCO` → `LEGL`; **simulation attached** | `4E+Q` | Module-specific setter via compliance contract [I] | Rule set `v+1` | `rules.param.changed` | Affected holders |
| Set lock-up / restriction dates | `REGO` → `CCO` | `4E+Q` | Partial freeze with release date / `7518` `lockTokens` / ERC-1404 restriction dates | Holding `locked(until)` | `rules.lockup.set` | Holder |
| Add / remove trusted claim issuer or claim topic | `CCO` → `SECO` | `4E+Q` | `addTrustedIssuer` / `removeTrustedIssuer`; `addClaimTopic` | — | `rules.trust.changed` | — |
| Investor-specific eligibility override | `CCO` → `LEGL` (legal basis required) | `4E` | — | Override `active(until)` | `rules.override.granted` | — |
| Enable pre-trade threshold block | `CCO` | `4E` | Off-chain pre-trade check (+ on-chain max-holders module where available) | Monitor `blocking` | `rules.threshold.block_on` | Market ops |

**Alerts & automations.**
- Holder count ≥80% of trigger (RealT 2,000 cap pattern).
- 3(c)(1) approaching 100 [`us/02-redswan` §11.4].
- Foreign % or landholder threshold within 5 points.
- Spike in deny decisions for one reason.
- Watchlist dates that should prompt a pack review:
  - GENIUS Act effective ~18 Jan 2027 (stablecoin allow-list);
  - AU Digital Assets Framework Act commences 8 Apr 2027 (TCP/DAP licensing; 6-month transition to ~8 Oct 2027);
  - DAC8 first reports due 30 Sep 2027;
  - AMLR applies 10 Jul 2027;
  - FIRB ban ends 30 Jun 2029;
  - TA proposal comment deadline 3 Nov 2026.
- Automation: a published pack version creates change tickets for every token whose rule set derives from it.

---

### M11 — Cap Table, Register & Reconciliation · Confidence **H**

**Purpose.** Maintain the **golden record** of who owns what. Keep it reconciled to the chain, custodians and banks, and (A3) to the **sovereign registry**. Produce snapshots for distributions, votes and tax, and official register extracts.

**Primary users:** `REGO`, `X-TA`, `CFO`, `CCO`, `X-AUD`, `X-REG`.

**Evidence:**
- Vendors: Tokeny real-time cap table with historical look-back and position reports [`WLD` §2]; Securitize token-holder registry and **RFE API** reconciliation of venue holders [`WLD` §3]; Vertalo "ledger is the register" [`WLD` §8]; ATS/Polymesh snapshots and checkpoints [`TSD` §2.5–2.6]; DigiShares reporting to TAs/CSDs [`APE` M7].
- Obligations: [`XCC` #25, #32]. The SEC TA proposal (Sep 2026) requires a DLT master file under **TA exclusive control**, names and addresses still required, and 6-year retention [`REG-US` §5]. The Saudi register has "absolute probative force" [`mena/04-saudi` §2.2].
- Platforms: RedSwan registrar reconciliation [`us/02-redswan` §11 #6]; Propvest chain vs Tangany vs ledger [`eu/03-propvest` §11 #6]; PRYPCO daily 3-way recon [`mena/01-prypco` §11 #6]; Ctrl Alt nightly XRPL ↔ holder register ↔ DLD [`mena/02-ctrl-alt` §11.4]; Canvas CDSR [`australia/04-canvas` §11 #3]; RealT average-balance snapshots incl. RMM aTokens [`us/04-realt` §5.3].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 11.1 | Cap table / register | Holders; top-10 concentration; % in pools/lending; tokenised vs non-tokenised holders; **as-of date picker** | Holder (investor/entity/pool/lending market/omnibus), wallet/sub-account, balance, % supply, frozen/locked, lock-up end, acquisition lots (date, cost basis), jurisdiction, class/partition, **legal-record ref** (TA entry / DLD certificate / unit register / RER deed share) | Token, class, jurisdiction, holder type | Open holder, open lots, issue statement | Export register extract | H |
| 11.2 | Snapshots & record dates | Scheduled snapshots and purposes | Snapshot, token, purpose (distribution/vote/tax/audit), **method** (point-in-time / average-balance incl. pool & lending deposits), block/time, file hash, status | Purpose | Recompute (before lock), lock | — | H |
| 11.3 | **3-way reconciliation** | Breaks by type; zero-break streak; last run per token | Token, chain supply/balances, golden record, custodian/omnibus sub-ledger, (bank for cash legs), Δ qty, Δ holders, status | Token, status | Open breaks, re-run | Run all (manual) | H |
| 11.4 | Registry sync (DLD / RER / external TA) | Pending instructions; ack latency; rejects | Instruction, type (register owners / transfer / certificate), property/TokenID, investors, sent, ack, status, retries | Registry, status | Retry, escalate | — | M |
| 11.5 | Breaks queue | Open breaks by age and severity | Break ID, type (qty mismatch / unknown holder / missing registration / pending tx / bridge drift / custodian mismatch), token, qty/value, age, owner, root cause, proposed fix | Type, age | Propose fix, assign | — | H |
| 11.6 | Holder analytics | Holders of record per SPV (12(g)); concentration; foreign %; geo map | Vehicle, holders, non-accredited holders, foreign %, top holder % | Vehicle | Open threshold monitor (M10) | — | M |
| 11.7 | Register extracts & position reports | Requests from auditors, courts and regulators | Request, requester, scope, as-of, generated, signed by | — | Generate, sign | — | H |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Run reconciliation | `AUTO` nightly / `REGO` | `AUTO` | Read: indexer, custodian API, registry API, bank feed | ReconRun `completed(breaks=n)` | `recon.run.completed` | Owners if breaks > 0 |
| Resolve break by register adjustment | `REGO` → `CCO` + `CFO` (+ `X-TA`) | `6E` | None, or corrective mint/burn/forced transfer via M09 | Break `open → resolved` | `recon.break.resolved`, `register.adjusted` | Affected holders |
| Force re-sync with registry | `REGO` → head of ops | `4E` | Registry API re-submit | Instruction `failed → resent` | `registry.sync.retried` | — |
| Take / lock snapshot | `AUTO` / `FINA` → `CFO` (lock) | `4E` | ATS/Polymesh checkpoint where native; else indexer read at block N | Snapshot `computed → locked` | `snapshot.locked` | Distribution owner |
| Import venue holders (RFE-style) | `SVCA` / `REGO` | `AUTO` | — | — | `register.venue.imported` | — |
| Certify register (monthly) | `REGO` → `X-TA` or `X-RE` | `4E` | Optional hash anchor | Register `certified(period)` | `register.certified` | Auditors |
| Issue register extract / position report | `REGO` | — | — | — | `register.extract.issued` | Requester |

**Alerts & automations.**
- Any break at T+1 → Sev2.
- **Unknown holder** (tokens held by an address not in the identity registry) → Sev1, because it is a compliance-control failure.
- DLD/RER sync failure [`mena/01-prypco` §11.4].
- Bridge supply drift.
- Snapshot missing 24h before a distribution record date.
- Register vs custodian mismatch.
- Automations: nightly 3-way recon; a monthly certified register extract is archived to M19 retention.

---

### M12 — Distributions & Corporate Actions · Confidence **H**

**Purpose.** Turn property income into correct, reconciled payouts at the product's cadence (daily, weekly, monthly, quarterly, or a bullet at exit). Run exit waterfalls, reinvestment, claims and other corporate actions.

**Primary users:** `FINA` (maker), `CFO` (checker), `TRSY`, `TOKO`, `TAXO`, `ASTM`.

**Evidence:**
- Vendors:
  - DigiShares dividends [`APE` M8 C#1];
  - Blocksquare `addRevenue` (admin or authorised partner only), `pushRevenueToUser`, claims, `getAverageMonthlyPayout` [`WLD` §5];
  - Hedera ATS dividends, coupons, balance adjustments, **Mass Payout** [`WLD` §11];
  - ERC-7518 `payout`/`batchPayout` [`TSD` §1.6];
  - Brickken `dividendDistribution` with preflight [`eu/04-brickken` §5.3];
  - InvestNext point-and-click **waterfall builder** and ACH; Agora waterfall automation [`WLD` §17];
  - Polymesh checkpoints [`TSD` §2.6].
- Obligation: [`XCC` #35].
- Platforms:
  - Lofty daily at midnight UTC [`us/01-lofty` §5.3];
  - RealT weekly average balance incl. RMM, merkle claims on Ethereum [`us/04-realt` §5.3];
  - Reental monthly USDT with 19% withholding [`eu/01-reental` §5.3];
  - PRYPCO monthly AED with enumerated deductions, AED 0.01 floor [`mena/01-prypco` §5.3];
  - BrickX smoothing and reserve [`australia/01-brickx` §5.3];
  - Propvest zero-distribution messaging [`eu/03-propvest` §11 #8];
  - Domoblock bullet exits with delay interest and Guarantee Fund [`eu/05-domoblock` §5.3].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 12.1 | Income ledger | Net distributable by asset; missing periods; rent posted vs scheduled | Asset/SPV, period, gross rent, opex lines (PM fee, maintenance, insurance, service charges, rates/tax, interest), reserve top-up, platform fees (e.g., RealT 2% income fee, SmartCrowd 0.5% admin quarterly), **smoothing adj.**, net distributable, source (PM import / manual), status (draft/confirmed/locked) | Asset, period, status | Confirm, adjust (reason), lock | Import from PM (M22) | H |
| 12.2 | Distribution runs | Upcoming runs (calendar); success rate; failed lines; unclaimed | Run ID, token(s), period, **cadence**, record date + **snapshot method**, gross, withholding, net, **payout asset** (USDC/USDT/EURC/AED/AUD/AUDM/wCBDC/fiat), **rail** (on-chain push / claim root / bank / in-app wallet credit), holders, reinvest %, state, approvals | Cadence, state, asset | Open, clone | — | H |
| 12.3 | Run review | Calc waterfall (income → fees → reserves → net → per token); variance vs last period and vs forecast; funding requirement vs treasury; **exceptions** (frozen/sanctioned holders → hold; missing tax form → default rate; dust below minimum; closed accounts; invalid addresses); file hash | Per-holder lines: holder, units, gross, WHT rate & basis, net, asset, destination, reinvest, exception | Exception type | Hold line, change destination (4E) | Approve run, export file | H |
| 12.4 | Waterfalls & exits | Scenario tester; hurdles hit | Presets: Reental 60/40 of sale gain (min €2,500); Propvest 80/20; PRYPCO 1% exit + ≤15% above 30% ROI; SmartCrowd Flip 60/40 above 10% hurdle; Domoblock 40/40/20 (Guarantee Fund) + **delay interest 0.75%/month capped at 4.5%**; Arrived 6–7% disposition; Stake 2.5% exit + 7% performance; **liquidation waterfall** (fiduciary costs → repairs → back taxes → legal obligations → holders, RealT) | Product | Run scenario, attach to exit | — | H |
| 12.5 | Claims (pull model) | Claimable vs claimed; unclaimed ageing | Claim period, token, root/contract, total, claimed, unclaimed, expiry, escheat status | Age | Publish root, extend, escheat | — | H |
| 12.6 | Reinvestment | Reinvest uptake; orders generated | Investor, product, setting (auto-reinvest same token / new project tokens / Smart Invest / gold), last amount, next | Setting | Pause | — | M |
| 12.7 | Corporate-actions calendar | Record/ex/payment dates; votes linked | Action (dividend, capital return, balance adjustment/split, redemption at exit, burn), token, dates, state | Type | Open | — | H |
| 12.8 | Failed payouts | Failures by reason | Line, holder, reason (blocked/frozen address, bank reject, ACH return, invalid wallet), attempts, next retry | Reason | Retry, manual payout, hold | Retry batch | H |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Confirm / lock income period | `FINA` → `ASTM` | `4E` | — | Income `draft → locked` | `income.period.locked` | — |
| Calculate run | `FINA` (AUTO) | `AUTO` | Snapshot read (M11) | Run `draft → calculating → calculated` | `distribution.run.calculated` | Reviewer |
| Approve run | `FINA` → `CFO` (above threshold + CEO) | `4E` / `6E` | — | `in_review → approved` | `distribution.run.approved` | Treasury |
| Declare zero / reduced distribution | `ASTM` → `CFO` + management board | `6E` | — | Run `approved(zero)` | `distribution.zero.declared` | Holders: explanatory notice (M21) |
| Fund run | `TRSY` → `CFO` | `4E+Q` | Stablecoin mint/convert and transfer into the distribution vault | `approved → funding → funded` | `distribution.run.funded` | — |
| Execute push payout | `TOKO`/`TRSY` via distribution vault | `4E+Q` (policy: destinations = approved file hash only) | Batch ERC-20 stablecoin transfers; `BSQ` `addRevenue` + `pushRevenueToUser`; ATS Mass Payout; `7518` `batchPayout`; `BKN` `dividendDistribution` | `funded → executing → executed` | `distribution.run.executed` | Holders (payout received) |
| Publish claim root | `TOKO` → `CFO` | `4E+Q` | Merkle root to claim contract (RealT pattern [I]); `BSQ` `addRevenue` (claim mode) | `→ executed(claimable)` | `distribution.claim.published` | Holders ("claim available") |
| Credit in-app wallets (off-chain) | `FINA` → `CFO` | `4E` | — | `→ executed` | `distribution.wallet.credited` | Holders |
| Retry failed lines | `FINA` (≤ original approved amount) | `AUTO`/`4E` | Re-send | Line `failed → paid` | `distribution.line.retried` | Holder |
| Manual payout | `FINA` → `CFO` | `4E` | Off-chain or single transfer | Line `failed → paid_manual` | `distribution.line.manual` | Holder |
| Compute and approve exit waterfall | `FINA` → `CFO` + `LEGL` (+ `X-RE`/SPV director) | `6E` | Burn/redeem after payout (M09) | Exit `computed → approved → paid` | `exit.waterfall.approved` | Holders |
| Draw Guarantee Fund / reserve | `ASTM` → `CFO` + board | `6E` | — | Fund balance ↓ | `fund.guarantee.drawn` | Holders of affected project |
| Escheat / expire unclaimed | `FINA` → `LEGL` | `4E` | Sweep from claim contract [I] | Claim `unclaimed → escheated` | `distribution.unclaimed.escheated` | Holders (prior notices) |

**Alerts & automations.**
- Batch failure → Sev2 [`us/01-lofty` §11.4].
- No rent posted for X days on an asset [`us/01-lofty` §11.4].
- Rent collected below X% of scheduled [`us/04-realt` §11.4].
- Distribution yield below target or below the REIT 90% test path [`us/03-arrived` §11.4].
- Treasury below the funding requirement.
- More than N% of holders missing tax forms.
- Snapshot not locked at T-24h.
- Unclaimed older than N days [`eu/02-blocksquare` §11.4].
- Delay interest starts automatically when a sale is more than 2 months late (Domoblock) [`eu/05-domoblock` §11.4].
- Automations: cadence scheduler (e.g., daily 00:00 UTC; PRYPCO within the first 5 working days; BrickX within 10 business days of month end); reinvest orders to M05; Monday payout e-mail report (RealT pattern [`us/04-realt` §11.4]).

---

### M13 — Tax · Confidence **M**

**Purpose.**
- Capture and validate tax status per investor.
- Compute withholding on distributions and dispositions.
- Track cost basis.
- Generate jurisdictional forms and statements.
- Compute transaction and transfer taxes.

**Primary users:** `TAXO`, `CFO`, `TRSY`, `SUPP` (read).

**Evidence:**
- Vendors: only off-chain comparators show tax tooling (InvestNext K-1 and Global Tax Center; Agora K-1 delivery) [`WLD` §17]. A **withholding engine for on-chain distributions is not observed** in any vendor [`APE` gaps].
- Obligations: [`XCC` #37–40; `REG-US` §8; `REG-EU` §8; `REG-ME` §8; `REG-AU` §8].
- Platforms:
  - Lofty 1099-DIV/1099-B and 30%/15% non-US withholding [`us/01-lofty` §2.7, §11 #12];
  - Arrived per-series REIT 1099-DIV [`us/03-arrived` §2.7];
  - mogul K-1 burden [`us/05-mogul` §2.7];
  - RealT annual distribution report and FIRPTA gap [`us/04-realt` §2.7];
  - Reental/Domoblock 19% ES withholding [`eu/01-reental` §2.7; `eu/05-domoblock` §2.7];
  - BrickX TFN and annual statement by ~30 Sep [`australia/01-brickx` §2.7];
  - Stake KSA WHT/Zakat [`mena/03-stake` §2.7].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 13.1 | Tax profiles | Missing/expiring forms; CRS indicia conflicts | Investor, residency(ies), TIN/TFN/NIF, form (W-9/W-8BEN/W-8BEN-E/W-8IMY; CRS self-cert; TFN/ABN), valid until (W-8: 3 years [I]), treaty rate, FATCA/CRS status, US-person flag, backup-withholding flag | Form status, residency | Request form, validate TIN | Send reminders | M |
| 13.2 | Withholding engine | Rules in force; withholding MTD; remittances due | Rule: payer jurisdiction × payment type (dividend/interest/return of capital/disposition) × investor residency × form → rate. Examples: US FDAP 30% or treaty (Lofty 15%); §1446(a) ECI (partnerships); **§1446(f) 10%** on partnership-interest transfers; **FIRPTA 15%** on USRPHC dispositions; AU MIT 15%/30%; **TFN 47%**; ES 19%; KSA WHT for non-residents | Jurisdiction | Edit rule (→ 4E), simulate | — | M |
| 13.3 | Forms & statements | Generation status by form and year; corrections | Form (1099-DIV / 1099-B / **1099-DA** with network-status flag / K-1 / 1042-S / 8805; AU AMMA & annual tax statement; ES Modelo 193/123 certificates; CRS/DAC2 XML; **DAC8** (if RCASP); Zakat statement; annual distribution report splitting dividend vs return of capital), year, entity, count, generated, delivered, filed | Form, year | Regenerate, correct | Generate batch | M |
| 13.4 | Cost basis & lots | Lots per holder; unknown-basis transfers | Holder, token, lot, acquired, cost, method (FIFO/specific), disposed, gain | Token | Adjust (4E) | — | M |
| 13.5 | Transaction taxes & fees | Taxes per transaction; threshold proximity | Transaction, tax (DLD 4% / KSA RETT 5% + non-Saudi surcharge up to 5% / EU RETT share-deal / AU landholder & foreign-purchaser surcharge duty / VAT on fees), base, rate, amount, payer, status | Tax type | Recalculate | — | M |
| 13.6 | Filing & remittance calendar | Next 60 days | Obligation, authority, due, owner, status | Authority | Mark filed (evidence) | — | M |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Validate form / TIN match | `AUTO` / `TAXO` | `AUTO` | — | Profile `pending → valid/invalid` | `tax.profile.validated` | Investor if invalid |
| Apply default / backup rate | `AUTO` | `AUTO` | — | Profile `default_rate` | `tax.default_rate.applied` | Investor (request form) |
| Override rate (treaty/exemption) | `TAXO` → `CFO` | `4E` | — | — | `tax.rate.overridden` | — |
| Edit withholding rule | `TAXO` → `CCO` + `CFO` | `6E` | — | Rule `v+1` | `tax.rule.changed` | Finance |
| Generate forms batch | `TAXO` → `CFO` | `4E` | — | Batch `generated → delivered → filed` | `tax.forms.generated` | Investors |
| Correct / reissue form | `TAXO` → `CFO` | `4E` | — | `corrected` | `tax.form.corrected` | Investor |
| Remit withholding | `TRSY` → `CFO` | `4E` | — | Remittance `paid` | `tax.remittance.paid` | — |
| File CRS / DAC8 / FATCA report | `TAXO` → `CCO` | `4E` | — | Filing `submitted` | `tax.report.filed` | — |

**Alerts & automations.**
- W-8 expiry at T-60.
- Missing TIN on funded accounts.
- CRS indicia changed (address/phone).
- **Foreign holder disposes of a USRPHC interest on the secondary market → FIRPTA flag** (RealT gap) [`us/04-realt` §2.7].
- US form deadlines: 1099 (Jan/Feb), K-1 (Mar).
- AU annual statement by ~30 Sep.
- **DAC8 first report due 30 Sep 2027** [`REG-EU` §8].
- 1099-DA basis reporting from 2026 [`REG-US` §8].
- Landholder-duty threshold approaching (linked to M10).

---

### M14 — Secondary Market Ops & Surveillance · Confidence **M** (controls observed in platforms; price bands and circuit breakers not in vendors)

**Purpose.** Run secondary liquidity **within the licence perimeter**. Venue mode is a licence-gated feature flag: disabled → bulletin board → listing board → periodic windows → continuous book → partner ATS/MTF/DLT-MTF → pool. The module enforces eligibility, lock-ins, price bands/floors and holding caps, operates halts and windows, settles DvP, and runs surveillance.

**Primary users:** `MKTO`, `CCO`, `VALO`, `X-BDP` (US ATS), `TOKO`.

**Evidence:**
- Vendors:
  - DigiShares internal exchange ("invite & whitelist investors") and **SwapContract** (whitelist security/stable tokens, fees, fee recipients, expiry) [`WLD` §1];
  - Tokeny conditional transfers/DvP and DINO distribution network [`WLD` §2];
  - Securitize Markets ATS activation [`WLD` §3];
  - InvestaX AMM [`WLD` §9];
  - global halt (`isTradingAllowed`, ERC-3643 `pause`) [`APE` M9].
  - **Circuit breakers and price bands: not observed** [`APE` M9].
- Obligations: [`XCC` #30–32].
- Platforms:
  - PRYPCO ±15% band vs DLD valuation, 3-month lock-in, 20% cap, approved-buyer visibility [`mena/01-prypco` §5.4];
  - BrickX 80% floor and meeting halts [`australia/01-brickx` §5.4];
  - Stake May/Nov windows at valuation price [`mena/03-stake` §5.4];
  - SmartCrowd Mar/Sep STF ±15% with cost reimbursement [`mena/05-smartcrowd` §5.4];
  - Arrived monthly windows on PPEX ATS [`us/03-arrived` §5.4];
  - Lofty book + PMM with fees [`us/01-lofty` §5.4];
  - Propvest Handelsplatz 0.5% seller fee, <24h settlement [`eu/03-propvest` §5.4];
  - RealT YAM token whitelist [`us/04-realt` §5.4];
  - Assetora global suspend [`australia/05-assetora` §11 #6].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 14.1 | Venue control | Venue mode per jurisdiction × instrument with **licence binding** (M02); trading hours (24/7 "subject to compliance"); fee schedule | Jurisdiction, instrument class, mode, licence/partner, hours, fees (buyer/seller/market surcharge/in-kind split), eligibility filter, status | Jurisdiction | Change mode (→ approvals) | — | M |
| 14.2 | Instruments | Instruments by state; avg premium/discount to NAV; depth | Instrument, state (pre_open/open/halted/window_closed/closed), **reference price** (valuation/NAV + date), **band/floor** (±15% / ≥80%), **lock-in end** (3 months / 1 year / 6 months / Rule 144 / Reg CF 1 year), min order (e.g., AED 1,000), last price, Δ vs NAV, depth, 24h volume, holders | State, Δ vs NAV | Halt, resume, edit band, list/delist | Halt all (per asset class) | M |
| 14.3 | Order book / listings | Live orders; rejects by reason | Order, side, investor, qty, price, % vs reference, validation (band ✓, lock-in ✓, cap ✓, eligibility ✓), time, state (open/partially_filled/filled/cancelled/expired/suspended while the instrument is halted) | Instrument, reason | Cancel (notice), open investor | — | M |
| 14.4 | Windows scheduler | Next windows; imbalance in current window | Window, instruments, open/close, **price rule** (Stake: valuation − fixed − amortised costs; SmartCrowd market price ±15% + cost reimbursement; Arrived limit orders), buy vs sell, allocation rule (pro-rata/FCFS), settlement date | Window | Open/close, set price, run matching | — | M |
| 14.5 | Trade blotter & settlement | Settlement SLA (T+0 atomic / <24h / T+1–2); fails | Trade, instrument, buyer, seller, qty, price, fees, DvP status, settle time, registry update (DLD/TA), confirmation ID | Status | Bust (4E), retry settle | Export confirmations | M |
| 14.6 | Surveillance | Alerts by type; cases opened; STOR/SAR hand-offs | Alert: wash trade/self-match, spoofing/layering, marking the close near valuation, rapid relisting, related-account trading, insider trading before ad-hoc news (sale, zero distribution), price vs NAV deviation, velocity, pool manipulation; score, accounts, state | Type, score | Open case (M20), dismiss (reason) | — | M |
| 14.7 | Halts & circuit breakers | Active halts; auto-halt rules | Instrument, trigger (manual / meeting / price move > X% / valuation event / ad-hoc disclosure / token pause), started, planned end, approver | Active | Resume (4E) | — | M |
| 14.8 | Venue connectivity | Partner venues (ATS/MTF/DLT-MTF/DEX/DINO) status; holder sync | Venue, instruments, eligible-investor feed, last sync, holders not directly registered (RFE) | Venue | Resync | — | L |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Change venue mode | `CCO` → `LEGL` (requires licence flag) | `4E` | — | Venue `mode v+1` | `market.venue.mode_changed` | Investors (terms update) |
| List instrument | `MKTO` → `CCO` (gates: fully funded, valuation done, holding period elapsed [`us/03-arrived` §5.4]) | `4E` | Whitelist token in P2P/swap contract (DigiShares SwapContract; RealT YAM) | Instrument `pre_open → open` | `market.instrument.listed` | Holders |
| Set / modify band or floor | `MKTO` → `CCO` + `CFO` | `6E` | — (off-chain rule) | Band `v+1` | `market.band.changed` | Holders (rules page) |
| **Halt instrument** | `MKTO` (auto on trigger) | `SGL+R` | Optional token `pause` (M09) or swap-contract toggle | `open → halted` | `market.halt.started` | Holders with open orders; market banner |
| Resume | `MKTO` → `CCO` | `4E` | Reverse | `halted → open` | `market.halt.ended` | As above |
| Open / close window; set window price | `AUTO`; price: `VALO` → `CFO` (US ATS: `X-BDP`) | `4E` | — | Window `scheduled → open → closed → matching → settled` | `market.window.*` | All holders |
| Cancel order (ops) | `MKTO` | — | Cancel on-chain offer where applicable | Order `→ cancelled` | `market.order.cancelled_by_ops` | Investor |
| Bust trade | `MKTO` → `CCO` | `4E(+Q)` | Reversing transfer / forced transfer (M09) | Trade `settled → busted` | `market.trade.busted` | Both parties |
| Whitelist venue/pool/escrow address | `MKTO` → `CCO` + `TOKO` | `4E+Q` | Register pool contract identity (ERC-3643); swap-contract whitelist | — | `market.address.whitelisted` | — |
| Set fees | `CFO` → CEO (notice period, e.g., 14 business days [`mena/01-prypco` §2.8]) | `4E` | Swap-contract `fee` setters [I] | Fee schedule `v+1` (effective date) | `market.fees.changed` | Investors (notice) |
| Open surveillance case | `MKTO` | — | — | → M20 case | `surveillance.case.opened` | `MLRO` |

**Alerts & automations.**
- Order outside band or below floor rejected; spike in rejects.
- Halt older than X hours.
- Window imbalance above X% [`mena/03-stake` §11.4].
- Discount to NAV above 20%, a pattern seen at Propvest (75–80% of par) and in Reental's P2P price below 80% of NAV alert [`eu/03-propvest` §9; `eu/01-reental` §11.4].
- Unfilled sell orders above X% [`us/03-arrived` §11.4].
- Wash-trade pattern; related-party trades.
- Settlement fail.
- Automations:
  - lock-in release (e.g., 3 months after allocation);
  - auto-halt on valuation publication, meeting start (BrickX) or token pause;
  - window open/close schedule;
  - "approved buyers only" listing visibility.

---

### M15 — Lending, Liquidity & DeFi Risk · Confidence **M** · Applicable to A1 / A5 / A6 only

**Purpose.** Run and risk-manage the liquidity layers around property tokens:
- AMM/PMM pools;
- lending markets with property tokens as collateral;
- oracles;
- liquidations;
- repo/collateral;
- protocol-token staking tiers.

This is the **highest reflexive-risk module**. RealT's issuer-set oracle and RMM kept running after cash flows stopped [`us/04-realt` §9].

**Primary users:** `RISK`, `MKTO`, `VALO`, `CCO`, `CFO`, risk committee.

**Evidence:**
- Platforms:
  - Lofty PMM (20% of circulating supply cap, imbalance penalty, LP fees) [`us/01-lofty` §5.4, §11 #7];
  - RealT RMM v3 Aave fork, whitelisted liquidators, oracle [`us/04-realt` §11 #7];
  - Reental RNTlend (≤75% LTV), staking tiers, buy-backs [`eu/01-reental` §11 #12–13];
  - Oceanpoint pools and POINT [`eu/02-blocksquare` §9];
  - Tokeniser/Liquidise AMM with pool-as-eligible-holder and leverage add-on [`australia/03-tokeniser` §5.4, §11 #9–10];
  - Canvas repo and collateral [`australia/04-canvas` §11 #6].
- Vendors: InvestaX AMM [`APE` M9]; Chainlink and RedStone NAV oracles [`VND` §10].
- Regulatory caveat: an AMM for ERC-3643 securities may be a "financial market" (AU s767A) [`australia/03-tokeniser` §2.3].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 15.1 | Liquidity pools | TVL; pool price vs NAV; imbalance heatmap; **staked % of circulating supply vs cap** | Pool, token, reserves, price, Δ vs NAV, depth (±2%), fee, imbalance, LP count, cap used, eligibility status of pool contract, state | Δ vs NAV, imbalance | Pause, edit params | — | M |
| 15.2 | Lending markets | Total collateral/borrow; utilisation; bad debt | Market, collateral token, **LTV / liquidation threshold**, reserve factor, utilisation, supply/borrow APY, oracle, frozen/active | Utilisation | Freeze reserve, edit params | — | M |
| 15.3 | Oracles & collateral pricing | Staleness; deviation vs last trade; **independence flag** | Feed, token, source (independent valuation NAV / TWAP / last trade), value, updated, deviation, approver | Stale, deviation | Push update (4E), switch source | — | M |
| 15.4 | Positions & health | Borrowers near liquidation; concentration | Account, collateral, debt, health factor, LTV, eligibility status | HF < 1.2 | Notify borrower | Bulk notify | M |
| 15.5 | Liquidations | Queue; proceeds; penalties | Liquidation, account, collateral seized, debt repaid, liquidator (whitelisted and eligible), tx | State | Review | — | M |
| 15.6 | Repo & collateral (A6) | Pledged collateral; haircuts; margin calls due | Agreement, collateral token, cash leg (wCBDC/stablecoin), haircut, maturity, margin status | Maturity | Margin call, release | — | L |
| 15.7 | Protocol tokens & tiers | Staking tiers; buy-back schedule; insider movements | Token (e.g., RNT/BST/BKN/LIQ), tier rules, stakers, buy-backs, **conflict flags** (utility-token gate on securities access) | — | Edit tier (board) | — | L |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Create / enable pool | `MKTO` → `CCO` + `RISK` | `6E+Q` | Deploy/configure pool; **register pool contract as eligible identity** (ERC-3643) | Pool `proposed → active` | `pool.enabled` | Holders |
| Change pool params (fee, cap, curve) | `RISK` → risk committee | `4E+Q` + 24h timelock | Pool setters [I] | Params `v+1` | `pool.params.changed` | LPs |
| Pause pool / lending market | `RISK` | `SGL+R` | Pool pause; lending `setReserveFreeze` [I] | `active → paused/frozen` | `defi.pause.*` | LPs/borrowers, market ops |
| Set LTV / liquidation threshold | `RISK` → risk committee + `CCO` | `6E+Q` + 24h timelock | PoolConfigurator `configureReserveAsCollateral` [I] | Market params `v+1` | `lending.params.changed` | Borrowers |
| Update oracle price / source | Automation or `VALO` → `RISK` | `4E+Q` | Oracle push / `setAssetSources` [I] | Feed updated | `oracle.updated` | Risk |
| Add / remove liquidator | `RISK` → `CCO` | `4E+Q` | Liquidator allow-list (+ identity registration) | — | `lending.liquidator.changed` | — |
| Margin call / release collateral (repo) | `RISK` → CIO / custodian | `4E` | Collateral contract release | Agreement `margin_called/released` | `repo.*` | Counterparties |

**Alerts & automations.**
- Pool imbalance above threshold; staked % at the cap [`us/01-lofty` §11.4].
- Health-factor cluster below 1.1; utilisation above 90%.
- **Oracle stale > 24h, or deviation vs last P2P/YAM trade > X%** [`us/04-realt` §11.4].
- Bad debt > 0.
- Pool price vs NAV > X% [`australia/03-tokeniser` §11.4].
- Unusual insider token movements [`eu/01-reental` §11.4].
- **Kill-switch automation:** asset flagged `distressed` or `legal_hold` (M03) → collateral factor 0 and pool paused (a RealT lesson).

---

### M16 — Treasury, Payments & Client Money · Confidence **H**

**Purpose.** Move and safeguard money across fiat and stablecoin rails:
- pay-ins and matching;
- withdrawals;
- escrow per offering;
- daily **client-money reconciliation**;
- stablecoin mint/redeem and the allow-list of payment assets;
- reserves and guarantee funds;
- the fee/revenue ledger.

**Primary users:** `TRSY`, `FINA`, `CFO`, `CCO`, `X-CUS`.

**Evidence:**
- Vendors:
  - Fireblocks transaction policies (amount, destination, asset, time window) and **pending-approvals dashboard** [`APE` M10 C#26];
  - DigiShares on-chain or traditional payments [`APE` M10 C#1];
  - Blocksquare investor EUR bank accounts and escrow refund [`APE` M10 C#16–18];
  - Stobox treasury operations [`APE` M10 C#22];
  - InvestNext/Agora ACH and cross-border payouts [`APE` M10];
  - vendor surfaces for Stripe, Plaid, Dwolla, Modern Treasury, Circle Mint, Zand, Monoova/Zepto [`VND` §5–6].
- Obligations: [`XCC` #16–19, #28–29].
- Platforms:
  - PRYPCO Zand ↔ ledger ↔ omnibus 3-way recon [`mena/01-prypco` §11 #6];
  - Propvest Mangopay safeguarding [`eu/03-propvest` §11 #7];
  - Reental proof-of-transfer matching [`eu/01-reental` §11 #7];
  - Domoblock €10k DAI per-transaction cap and Guarantee Fund [`eu/05-domoblock` §11 #3, #12];
  - BrickX PayID and A$20k advances [`australia/01-brickx` §11 #3];
  - mogul refunds (card 45d, ACH 60d) and reserve-interest conflict [`us/05-mogul` §11 #6–7].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 16.1 | Accounts & wallets | Balances by entity, currency and purpose | Account/wallet, entity, **purpose** (client money/segregated, escrow, operating, SPV operating, trust, e-money, stablecoin treasury, distribution vault, paymaster), institution/custodian, currency, balance, ledger, Δ, signatories, last statement | Entity, purpose | Open, import statement | — | H |
| 16.2 | Pay-ins | Deposits today by rail; **unmatched** count and age; returns/chargebacks | Receipt, rail (ACH/SEPA/wire/card/Apple Pay/PayID-NPP/BPAY/Lean pay-by-bank/mada/SADAD/PayPal/Venmo/USDC/USDT/DAI/AE Coin/AUDM), amount, reference/virtual IBAN, payer, matched investor/order, KYT (crypto), state | Rail, state | Match, return, flag | Auto-match run | H |
| 16.3 | Pay-outs & withdrawals | Queue; SLA (24–72h); over-threshold items | Request, investor, amount, rail, destination (allow-listed?), travel-rule data ✓ (crypto), state | Threshold, state | Approve, reject, hold | Release batch | H |
| 16.4 | Escrow | Escrow by offering: commitments vs received vs released | Offering, escrow type (bank / qualified third party / smart contract), agent, min/max, received, releasable, refund due, state | State | Release (→ approvals), refund | — | H |
| 16.5 | Client-money reconciliation | **Bank vs ledger Δ** (target 0); safeguarding report status | Date, account, bank balance, Σ investor balances, Δ, explained items, status | Date | Explain item, escalate | — | H |
| 16.6 | Stablecoin operations | Float by coin/chain; depeg monitor; **payment-asset allow-list** per jurisdiction | Asset (USDC/EURC/USDT/AE Coin/AUDD/AUDM/RLUSD), issuer, regulatory status (GENIUS permitted from ~Jan 2027; MiCA EMT; CBUAE PTSR NoC; ASIC 2025/867), jurisdictions allowed, peg, float | Jurisdiction | Mint/redeem, toggle allow | — | M |
| 16.7 | Reserves & funds | Reserve months per asset; Guarantee Fund; ARVA reserve; reserve-interest attribution | Fund/reserve, asset, balance, target, Δ, inflows/draws, interest to (investors/platform, disclosed) | Below target | Draw (→ approvals), top-up | — | M |
| 16.8 | Fees & revenue ledger | Fee income by type and rail | Fee (acquisition/management/exit/performance/trading/listing/structuring/KYC pass-through), source, accrued, collected, invoice | Type | Adjust (4E) | — | H |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Match deposit | `AUTO` exact; `TRSY` manual (above €10k/$10k → `CFO`) | `AUTO`/`4E` | — | Receipt `unmatched → matched`; Order `awaiting_funds → funded` | `payin.matched` | Investor |
| Return unmatched funds | `TRSY` → `CFO` | `4E` | Stablecoin return (policy) | `unmatched → returned` | `payin.returned` | Payer |
| Approve withdrawal | `AUTO` ≤ limit; `TRSY` → `CFO` (> $25k); `CFO` + CEO (> $250k) | `AUTO`/`4E`/`6E` (+Q crypto) | Stablecoin transfer from treasury vault (allow-listed destination; travel-rule payload) | `requested → approved → paid` | `payout.withdrawal.*` | Investor |
| Release escrow | `FINA` → `CFO` + `CCO` (after finalise) | `6E(+Q)` | Escrow contract release (`BKN` post `closeOffer`) | Escrow `holding → released` | `escrow.released` | Issuer |
| Internal transfer between accounts | `TRSY` → `CFO` (dual bank authorisation) | `4E` | — | — | `treasury.transfer.*` | — |
| Mint / redeem stablecoin | `TRSY` → `CFO` | `4E+Q` | Issuer mint/redeem (e.g., Circle Mint, Macropod AUDM) | Float ↑/↓ | `stablecoin.mint.*` | — |
| Change payment-asset allow-list | `CCO` → `CFO` | `4E` | — | Allow-list `v+1` | `payment_asset.allowlist.changed` | Investors (if payout asset changes) |
| Draw reserve / Guarantee Fund | `ASTM` → `CFO` + board | `6E` | — | Reserve ↓ | `reserve.drawn` | IR |
| Grant deposit advance (BrickX pattern) | `AUTO` by rule / `TRSY` | `AUTO` | — | Advance `open → cleared` | `advance.*` | — |

**Alerts & automations.**
- **Client-money Δ ≠ 0 → Sev1** [`mena/01-prypco` §11.4].
- Unmatched deposit older than 48h [`eu/01-reental` §11.4].
- ACH return or chargeback.
- Withdrawal to a non-allow-listed destination attempted.
- Stablecoin depeg above 0.5%.
- **Payment-asset regulatory status change**, e.g., non-MiCA USDT for EU payouts [`eu/01-reental` §2.8].
- Float below 7 days of payouts.
- Escrow refund deadline.
- Safeguarding report due.
- Reserve below target (BrickX 3 months; mogul 8 months) [`australia/01-brickx` §11.4; `us/05-mogul` §11.4].
- DAI per-transaction cap exceeded [`eu/05-domoblock` §11.4].
- Automations: bank-statement import every hour; auto-match; daily recon; ledger freeze of the affected rail if the recon break exceeds tolerance.

---

### M17 — Governance & Voting · Confidence **H**

**Purpose.** Run holder decisions and statutory meetings: sale/exit, term extension, PM replacement, major capex, rent changes, wind-up, and protocol parameters. The module handles notices, timers, record-date snapshots, weighting, quorum/threshold rules, trading halts, certification, and **execution tracking** of what was decided.

**Primary users:** `LEGL` (company secretary), `ASTM`, `MKTO`, `X-RE`/SPV directors, `IRCM`.

**Evidence:**
- Vendors: DigiShares meetings with agendas, per-item votes, **vote weighting by share rights**, external invitees [`WLD` §1]; Brickken shareholder voting [`WLD` §4]; Hedera ATS voting rights as a corporate action [`WLD` §11]; Polymesh ballots [`TSD` §2.6].
- Platforms:
  - Lofty Robert's Rules: motion → second → 48h discussion (24h urgent) → 72h vote (24h) → 90-day cooldown [`us/01-lofty` §5.6];
  - PRYPCO sale vote 50%+1 over 14 days, **auto-triggered at 30% ROI** [`mena/01-prypco` §5.5];
  - BrickX 75% anytime / 50% at the 5-year anniversary, trading halted during meetings [`australia/01-brickx` §5.5];
  - mogul PM proposals with two quoted options and **non-responses defaulting to the manager** (flagged) [`us/05-mogul` §5.6];
  - Assetora 75%/50% term votes [`australia/05-assetora` §5.5];
  - protocol DAOs: Reental Aragon, Blocksquare Snapshot [`eu/01-reental` §5.6; `eu/02-blocksquare` §5.6];
  - Propvest SchVG/KapMUG bondholder logistics [`eu/03-propvest` §11 #15].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 17.1 | Proposals | Open votes; average turnout; failed-exit list; cooldowns | Proposal, asset/token, **type**, **origin** (holder motion / PM proposal / auto-trigger / statutory meeting / fiduciary), state, discussion end, vote end, quorum, **threshold** (50%+1 / 75% / super-majority), turnout, result, cooldown until | Type, state, asset | Open, withdraw | — | H |
| 17.2 | Proposal builder | Rule preview for the vehicle (OA/constitution/SchVG/Corporations Act) | Fields: type; options (quotes attached); record date; **weighting** (pro-rata / 1 + units (mogul) / share-rights weighting); eligibility; notice period; urgent flag; **default-on-no-response** (disallowed by default; warning, cf. mogul) | — | Save, simulate outcome thresholds, submit | — | H |
| 17.3 | Meetings | Upcoming meetings; notices sent; proxies | Meeting, vehicle, agenda items, notice date, proxies received, external invitees, **linked trading halt**, scrutineer | Date | Send notice, record minutes | — | H |
| 17.4 | Ballot monitor | Turnout curve; participation by channel; disputes | Voter, weight, choice (sealed until close if configured), channel, time | Proposal | Send nudge (M21) | Nudge non-voters | M |
| 17.5 | Execution tracker | Passed resolutions awaiting execution; overdue | Resolution, required actions (list for sale / replace PM / approve capex / change rent), owner, due, evidence, status | Status | Update, attach evidence | — | M |
| 17.6 | Protocol governance (A1/A5) | DAO proposals; parameter changes queued in timelock | Proposal (Snapshot/Aragon), parameters affected, result, execution tx, timelock ETA | State | Queue execution (→ approvals) | — | M |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Create proposal / intake holder motion | `ASTM`, `LEGL`; holder via portal | — | — | Proposal `draft → filed` | `gov.proposal.filed` | Holders of the asset |
| Second a motion (Lofty model) | Holder | — | — | `filed → discussion` | `gov.proposal.seconded` | Holders |
| Publish notice / open vote | `LEGL` → `CCO` | `4E` | Record-date snapshot (M11); optional ATS voting-rights action / Polymesh checkpoint | `discussion → voting` | `gov.vote.opened` | All eligible holders |
| Auto-create sale vote on trigger (e.g., ROI ≥ 30%) | System | `AUTO` | — | `→ filed(auto)` | `gov.proposal.auto_triggered` | Holders, `ASTM` |
| Halt trading for meeting | `MKTO` (AUTO by link) | `AUTO` | See M14 | Instrument `open → halted` | `market.halt.started` | Holders |
| Close and tally | System | `AUTO` | Optional result-hash anchor | `voting → tallying → passed/failed/quorum_failed` | `gov.vote.closed` | Holders |
| Certify result | `LEGL` → independent scrutineer / `X-RE` director | `4E` | — | `passed → certified` | `gov.result.certified` | Holders; public notice |
| Execute outcome | `ASTM` → `CFO` / `LEGL` | `4E` | Downstream (M03 exit, M22 PM change, M12 waterfall) | `certified → executing → executed` | `gov.execution.*` | Holders (progress) |
| Apply cooldown (failed proposal) | System | `AUTO` | — | `failed → cooldown(90d)` | `gov.cooldown.applied` | Proposer |

**Alerts & automations.**
- Vote closing within 48h with turnout below quorum.
- **Default-to-manager rule triggered** → compliance flag (Howey risk noted for mogul) [`us/05-mogul` §10].
- 5-year statutory meeting due in 90 days [`australia/01-brickx` §11.4].
- ROI trigger crossed [`mena/01-prypco` §11.4].
- Passed resolution not executed within SLA.
- Dispute raised.
- Automations: timers (48/72h, 24h urgent, 14-day window); trading halt at meeting start and resume at close (BrickX: resumed 3 pm the same day [`australia/01-brickx` §5.5]).

---

### M18 — Documents & E-sign · Confidence **H**

**Purpose.**
- Templates with variables, and e-signature envelopes.
- A versioned **disclosure library** with delivery/acknowledgement logs.
- Deal and data rooms.
- Token-attached documents (ERC-1643 / IPFS hashes).
- Retention and legal holds.

**Primary users:** `LEGL`, `CAPM`, `IRCM`, `CCO`.

**Evidence:**
- Vendors:
  - DigiShares contracts module: variable fields, distribution to a **selected subset of investors**, investor review/suggestions, DocuSign/Dropbox Sign [`WLD` §1];
  - Securitize subscription agreements [`WLD` §3];
  - ATS document management and ERC-1643 `setDocument` [`TSD` §1.2];
  - InvestNext/Agora templating and segmented sharing; Juniper data rooms [`WLD` §17];
  - Taurus legal docs with audit trail [`WLD` §13].
- Obligations: [`XCC` #33, #44].
- Platforms:
  - Blocksquare IPFS-hashed corporate resolutions [`eu/02-blocksquare` §2.8];
  - PRYPCO DLD Property Token Ownership Certificate [`mena/01-prypco` §2.2];
  - Reental transparency portal (a Google Drive folder, the anti-pattern) [`eu/01-reental` §11 #16];
  - Domoblock loan contracts and assignment deeds [`eu/05-domoblock` §11 #6];
  - Stake/SmartCrowd DIFC register-verifiable share certificates [`mena/03-stake` §2.8].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 18.1 | Template library | Templates by type and jurisdiction; pending approvals | Template (subscription agreement; OA/constitution; participative-loan contract; assignment deed; PPM/Form C/1-A/KIIS/PDS/SPDS/TMD/whitepaper; ToS/privacy with tenant placeholders; notices; certificates), variables, version, jurisdiction, language, state | Type, jurisdiction | Edit, clone, submit | — | H |
| 18.2 | Envelopes | Sent/completed/overdue | Envelope, investor, offering, template version, provider (DocuSign / Dropbox Sign / UAE PASS signature), sent, viewed, signed, **signed-doc hash** | Status | Remind, void, resend | Bulk remind | H |
| 18.3 | Disclosure library | Documents in use; re-acknowledgements outstanding | Document, product, version, effective, regulator submission (ref/date), in-use ✓, acknowledgements (count/%) | Product | Publish v+1, request re-ack | — | H |
| 18.4 | Data rooms | Rooms; views; watermarking | Room, offering/asset, docs, access segment, views, downloads | Offering | Share, revoke | — | M |
| 18.5 | Token-attached documents | Docs per token; sync status | Token, doc name, URI/IPFS CID, hash, on-chain ✓, updated | Token | Set / update (→ M09) | — | H |
| 18.6 | Retention & legal holds | Items due for destruction; holds | Record type, jurisdiction retention (US TA 6y, EU AML 5y, VARA 8y, AU 7y), hold (case/litigation), destroy after | Due | Place/release hold | Approve destruction batch | M |
| 18.7 | Investor document vault | Statements, certificates and tax forms delivered | Investor, doc, period, delivered, opened | Doc type | Resend | Publish batch | H |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Create / edit template | `LEGL` | — | — | Template `draft` | `doc.template.edited` | — |
| Approve template version | `LEGL` → `CCO` | `4E` | — | `draft → approved` | `doc.template.approved` | Users of template |
| Generate and send envelopes | `CAPM` / `AUTO` | — | — | Envelope `sent` | `doc.envelope.sent` | Investor |
| Void envelope | `LEGL` | — | — | `sent → voided` | `doc.envelope.voided` | Investor |
| Publish disclosure / supplement | `LEGL` → `CCO` | `4E` | Optional ERC-1643 `setDocument`; IPFS anchor | Doc `effective` | `doc.disclosure.published` | Holders (acknowledge) |
| Place / release legal hold | `LEGL` → `CCO` | `4E` | — | Record `hold/active` | `doc.hold.*` | — |
| Approve destruction batch | `LEGL` → `CCO` | `4E` | — | Record `destroyed` | `doc.destroyed` | — |

**Alerts & automations.**
- Envelope unsigned after X days, which blocks allocation.
- Disclosure change requires re-acknowledgement before the next order.
- Stale financials in an offering doc.
- Destruction due, but the record is under hold.
- Automation: documents are generated from the offering/investor variables, and signed hashes are stored with the order (M05).

---

### M19 — Reporting & Regulatory Filings · Confidence **M**

**Purpose.**
- A regulatory calendar and filing workspace per jurisdiction and entity.
- Prudential/capital monitors.
- A **regulator data room / observer portal**.
- Investor, board and public reporting built on governed metric definitions.

**Primary users:** `CCO`, `LEGL`, `CFO`, `IRCM`, `X-REG`, `X-AUD`.

**Evidence:**
- Vendors: Zoniqx audit-ready operational dashboards; Brickken reporting; Juniper investor reporting; Fireblocks policy docs and authorisation history for regulators; DigiShares reporting to TAs/CSDs [`APE` M13].
- Obligations: [`XCC` #5, #34–35, #44–45]; regional checklists in [`REG-US`], [`REG-EU`], [`REG-ME`], [`REG-AU`].
- Platforms:
  - REGA weekly sandbox reports [`mena/04-saudi` §2.3, §11.1 #8];
  - VARA returns and ARVA 6-monthly audit pack [`mena/02-ctrl-alt` §11 #7, #11];
  - Arrived 1-K/1-SA/1-U [`us/03-arrived` §11 #11];
  - Propvest BaFin/Bundesbank and MiFID cost reports [`eu/03-propvest` §11 #12, #16];
  - Assetora ASX continuous disclosure [`australia/05-assetora` §11 #13];
  - Canvas relief-conditions tracker [`australia/04-canvas` §11 #11];
  - metric-inflation lessons: TTV vs raised, "US$73.8bn" pipeline [`eu/04-brickken` §10; `australia/02-redbelly` §10].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 19.1 | Regulatory calendar | Due in 7/30/90 days; overdue | Obligation, e.g.: **US** Form D (≤15 days after first sale), blue-sky notices, Form C/C-AR, 1-A/253G2/1-K/1-SA/1-U, FOCUS (BD), Form TA-2 (proposed); **EU** ECSP annual NCA report, prospectus supplements, BaFin/Bundesbank, AIFM Annex IV, DAC8; **MENA** VARA returns, ARVA monthly disclosures + 6-monthly audit + attestation, BWRA, goAML, DFSA crowdfunding returns, CMA reports, **REGA weekly**; **AU** ASIC lodgements, compliance-plan audit, AUSTRAC compliance report, DDO significant dealings (10 business days), AFCA IDR data, TCP/DAP licensing and reporting (DAF Act commences 8 Apr 2027; 6-month transition to ~8 Oct 2027) — plus entity, owner, due, status, evidence | Region, entity, status | Open workspace | — | M |
| 19.2 | Filing workspace | Draft, review and submission state | Section, data source, reviewer, comments | — | Pull data, submit for review, record receipt | — | M |
| 19.3 | Prudential dashboard | Capital/NTA vs requirement and buffer: **VARA ARVA max(AED 1.5M, 2% of reserves)**; AU NTA ($10M custody / $150k incidental); BD net capital | Entity, requirement, actual, buffer %, trend | Entity | Export board pack | — | M |
| 19.4 | Regulator data room / observer portal | Active regulator sessions; exports | Grant, regulator, scope (entities/tokens/date range), expires, exports (JSON / ISO 20022 / CSV), access log | Regulator | Grant, revoke, schedule export | — | M |
| 19.5 | Investor reports | Statements issued; open rates | Report (monthly/quarterly/annual statement, NAV/valuation report, property report, MiFID ex-ante/ex-post costs, track record planned vs actual), period, audience, published | Type | Generate, publish | Publish batch | H |
| 19.6 | Board & management reports | KPI pack; risk register; compliance MI; audit-committee pack; listed-parent disclosures | Pack, period, owner, approved | — | Generate, approve | — | M |
| 19.7 | Metric definitions | **Governed KPI dictionary**, e.g., "capital raised" ≠ "tokenised asset value" ≠ "on-chain TVL" | Metric, definition, source, owner, version | — | Edit (4E) | — | L |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Prepare filing | Obligation owner | — | — | Filing `draft` | `filing.drafted` | Reviewer |
| Approve filing | Owner → `CCO` / `LEGL` | `4E` | — | `draft → approved` | `filing.approved` | — |
| Submit and record receipt | `LEGL` / `CCO` | — | — | `approved → submitted → acknowledged` | `filing.submitted` | Exec channel |
| Grant regulator/auditor access | `CCO` → `SECO` | `4E` | — | Grant `active(until)` | `access.regulator.granted` | Regulator contact |
| Publish investor report | `IRCM` → `CFO` / `CCO` | `4E` | Optional hash anchor | Report `published` | `report.investor.published` | Investors |
| Publish public KPIs / track record | `IRCM` → `CFO` + `CCO` (claims review) | `6E` | — | `published` | `report.public.published` | — |
| Change metric definition | `CFO` → `CCO` | `4E` | — | Metric `v+1` | `metric.definition.changed` | Report owners |

**Alerts & automations.**
- Filing due at T-14 and T-3; missed filing → Sev2.
- Capital buffer below 20%.
- Regulator access expiring.
- Export failure.
- **Public metric inconsistent with governed definition** (a lesson from Stake, SmartCrowd and RedSwan metric conflicts) [`mena/03-stake` §10; `mena/05-smartcrowd` §10; `us/02-redswan` §10].
- Automation: weekly REGA report auto-assembled from KPIs (profitability, local hiring, complaints) [`mena/04-saudi` §11.5].

---

### M20 — AML Case Management · Confidence **M** (screening H; SAR workflow not observed in any vendor)

**Purpose.**
- Screen names and wallets continuously.
- Monitor transactions with property-specific scenarios.
- Manage alerts → cases → SAR/STR/SMR under regulatory clocks with **tipping-off controls**.
- Operate the Travel Rule console.
- Maintain the business-wide risk assessment (BWRA).

**Primary users:** `MLRO`, `KYCA` (L1 analyst), `CCO`, `TRSY`.

**Evidence:**
- Vendors:
  - **"SAR/STR filing workflow: not observed in any vendor"** [`APE` M13, gaps];
  - Sumsub AML monitoring and Travel Rule [`VND` §1];
  - Chainalysis/TRM/Elliptic KYT alert queues and Notabene Travel Rule inbox [`VND` §3];
  - Fireblocks automated screening in transaction workflows [`VND` §3].
- Obligations: [`XCC` #20–24]; US 30-day SAR clock [`REG-US` checklist]; UAE FDL 10/2025, EOCN, goAML, AED 3,500 travel rule [`REG-ME` §4]; EU TFR [`REG-EU` §4]; AU tranche 2 [`REG-AU` §4].
- Platforms: BrickX AML/CTF case management and order-book round-tripping [`australia/01-brickx` §11 #2]; Domoblock DAI source of funds and €10k cap [`eu/05-domoblock` §11 #3]; mogul PayPal/Venmo/USDT inflows [`us/05-mogul` §11 #14]; PRYPCO STR log [`mena/01-prypco` §11 #12].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 20.1 | Screening hub | Hits pending; list versions (OFAC/EU/UN/EOCN/DFAT); re-screen jobs | Hit, subject (person/entity/wallet), list, score, matched fields, first seen, state | List, state | Adjudicate (→ M07 rules) | Re-screen all | H |
| 20.2 | Monitoring scenarios | Alert volume and precision per scenario | Scenario (rapid in-out; structuring; **round-tripping via order book**; third-party funding; high-risk wallet inflow; dormant reactivation; VA- or cash-funded property purchase; per-transaction crypto caps; P2P rails inflow; related-account trading), thresholds per jurisdiction, lookback, owner, version | Scenario | Tune (→ 4E, backtest attached) | — | M |
| 20.3 | Alerts queue | Ageing; per-analyst load | Alert, scenario, score, customer, amount, created, assignee, state (new/triage/closed_fp/escalated) | State, score | Triage, close as FP (reason), escalate | Assign | M |
| 20.4 | Cases | Open cases by stage; **filing clocks** | Case, subjects (party graph), linked alerts, stage (open/investigating/mlro_review/sar_decision), clock due (US SAR 30 days; AU SMR 24h terrorism / 3 business days other; UAE "without delay"), restrictions, need-to-know list | Stage, clock | Open, add evidence, restrict | — | M |
| 20.5 | SAR/STR/SMR filings | Filed per period; post-filing monitoring | Filing, jurisdiction/format (FinCEN SAR, **goAML** XML, AUSTRAC SMR, SEPBLAC, other FIU), case, submitted, reference, follow-up | Jurisdiction | Draft, submit | — | M |
| 20.6 | Travel Rule console | Messages in/out; missing-data holds; self-hosted proofs | Transfer, direction, counterparty VASP, originator/beneficiary data ✓, threshold rule (US ≥$3,000; EU no de minimis + >€1,000 self-hosted proof; **UAE ≥AED 3,500 daily aggregate** + unhosted EDD; AU reform), state (pending/complete/held/declined/returned) | State | Request data, decline, return | — | M |
| 20.7 | BWRA & risk model | Inherent/residual matrix; board approval status | Risk factor, inherent, controls, residual, owner, review date | — | Update, submit for approval | — | M |
| 20.8 | Property-transaction AML | SPV acquisitions/disposals under review | Transaction, counterparty KYC, funding source (cash/VA flag), report required (UAE DNFBP; AU tranche 2 from 1 Jul 2026; US GTOs watch) | State | Review, file report | — | M |
| 20.9 | Regulatory requests & orders | Production orders, freezing orders, information requests | Request, authority, deadline, scope, legal hold, response | Deadline | Respond (`LEGL` + `MLRO`) | — | M |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Triage / close false positive | `KYCA` (L1) | `—` (QA sample by `MLRO`) | — | Alert `triage → closed_fp` | `aml.alert.closed` | — |
| Escalate to case | `KYCA` | — | — | `triage → escalated`; Case `open` | `aml.case.opened` | `MLRO` |
| Restrict account / freeze holdings | `MLRO` | `SGL+R` | M09 `setAddressFrozen` / partial freeze | Investor `restricted` | `aml.restriction.applied` | Internal only (**tipping-off guard hides reasons from support**) |
| SAR/STR decision | `MLRO` (sole authority; documented rationale) | — | — | `sar_decision → file / no_file` | `aml.sar.decided` | — |
| File SAR/STR/SMR | `MLRO` | — | — | `→ sar_filed → post_filing_monitoring` | `aml.sar.filed` (content sealed) | — |
| Exit relationship | `MLRO` → `CCO` | `4E` | Redemption/forced sale per terms (M09/M14) | Investor `offboarding` | `aml.exit.approved` | Investor (neutral wording) |
| Decline / return travel-rule transfer | `MLRO` / `TRSY` | — | Return transfer | TR message `declined/returned` | `travelrule.declined` | Counterparty VASP |
| Tune scenario | `MLRO` → `CCO` | `4E` | — | Scenario `v+1` | `aml.scenario.tuned` | — |
| Approve BWRA | `MLRO` → board | `4E` | — | BWRA `approved(year)` | `aml.bwra.approved` | — |

**Alerts & automations.**
- SAR clock at T-5 days.
- Sanctions hit on an existing holder after a list update → Sev1 (freeze within minutes).
- Travel-rule data missing beyond 3 working days (UAE domestic allowance) [`REG-ME` §4].
- Unhosted-wallet EDD pending.
- Case ageing beyond 30 days.
- BWRA annual review due.
- goAML/AUSTRAC submission failure.
- Automations: list update → re-screen investors and wallets; new high-risk wallet → auto-hold of the deposit (M16).

---

### M21 — Communications & Investor Relations · Confidence **H**

**Purpose.**
- Announcements and property feeds; mass and targeted campaigns; the admin inbox.
- **Marketing compliance**: pre-approval, claims library, permit numbers, jurisdiction-gated disclaimers.
- Notification templates; investor-portal CMS; T&C versioning.

**Primary users:** `IRCM`, `MKTG`, `CCO`, `X-BDP` (registered principal, US), `LEGL`.

**Evidence:**
- Vendors: DigiShares mass email, **administrator inbox**, company news, CMS by jurisdiction × domicile [`WLD` §1]; Securitize shareholder communications and fundraise-site CMS [`WLD` §3]; InvestNext/Agora portal updates [`APE` M14].
- Obligations: marketing compliance [`XCC` #36]:
  - Texas SSB order against "backed by real estate" / "capital protected" claims [`REG-US` §7];
  - FINRA 2210 principal approval [`us/02-redswan` §11 maker-checker];
  - VARA marketing regulations and RERA **Trakheesi** permits [`REG-ME` §7];
  - ASIC RG 234 (Jun 2026) [`REG-AU` §7];
  - ECSPR KIIS consistency [`REG-EU` §7].
- Platforms:
  - PRYPCO T&C changes (30-day notice; 14 business days for fees; 5-day re-confirmation) [`mena/01-prypco` §2.8];
  - Reental phishing broadcasts [`eu/01-reental` §11 #17];
  - Assetora feed and smart notifications [`australia/05-assetora` §11 #12];
  - Lofty per-property community channels [`us/01-lofty` §3];
  - RedSwan "24/7 liquidity" vs disclaimer conflict [`us/02-redswan` §10].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 21.1 | Announcements & feed | Posts by asset; engagement | Post, asset/token, type (property update / valuation / distribution / zero-distribution explanation / meeting notice / ad-hoc disclosure / security alert), audience, state, published | Type, asset | Edit, publish (→ approval if disclosure) | — | H |
| 21.2 | Campaigns | Sends; open/click; unsubscribes | Campaign, segment, template, channel (email/push/SMS/in-app), **jurisdiction gating**, consent basis, schedule, state | State | Send test, schedule, cancel | — | H |
| 21.3 | Admin inbox | Unassigned; SLA | Thread, investor, topic, assigned team, age, SLA | Team, SLA | Reply, route (support/compliance), convert to complaint | — | H |
| 21.4 | Marketing compliance | Submissions pending; permits expiring | Item (ad / landing page / social / finfluencer / listing), channel, jurisdiction(s), claims used, reviewer, **permit no.** (Trakheesi), disclaimers injected, approved until, archive link | State, jurisdiction | Approve, reject, request edits | — | M |
| 21.5 | Claims library | Approved statements and banned phrases | Claim (e.g., yield definition; "backed by real estate"; "capital protected"; "24/7 liquidity"), status (approved/conditional/banned), jurisdictions, evidence | Status | Edit (4E) | — | M |
| 21.6 | Templates | Transactional templates by language (EN/AR/DE/ES) | Template, event, channel, language, version | Event | Edit, test | — | H |
| 21.7 | Portal CMS & T&C | Pages by jurisdiction × domicile; T&C versions and effective dates | Page/T&C, version, jurisdictions, effective date, notice sent | — | Publish (→ approval) | — | H |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Publish routine property update | `IRCM` (or `X-PM` draft → `IRCM`) | `—`/`4E` | — | Post `published` | `comms.post.published` | Holders |
| Publish disclosure-relevant announcement | `IRCM` → `CCO` (material: + `LEGL`) | `4E`/`6E` | — | `published` | `comms.disclosure.published` | Holders; public |
| Approve marketing item | `MKTG` → `CCO` (US BD: `X-BDP` registered principal) | `4E` | — | Item `approved(until)` | `marketing.item.approved` | Maker |
| Send campaign | `IRCM`/`MKTG` (marketing content must be approved) | `—` | — | Campaign `sent` | `comms.campaign.sent` | Recipients |
| Publish T&C version | `LEGL` → `CCO` | `4E` | — | T&C `scheduled(effective = +30 days)` | `terms.version.published` | All users (notice) |
| Security / incident broadcast | `CCO` / `SECO` | `SGL+R` | — | — | `comms.incident.broadcast` | All users |

**Alerts & automations.**
- Promotion or permit expiring.
- **Unapproved claim detected live** (site/social crawler) [L].
- Complaint tagged "marketing".
- Inbox SLA breach.
- Bounce spike.
- Automations: disclaimers injected by residency/geo-IP; T&C change triggers re-acceptance at next login after the effective date.

---

### M22 — Property Operations · Confidence **M** (no token vendor ships it; driven by platform failures and `XCC` #42)

**Purpose.** Manage property substance:
- rent roll and PM oversight;
- maintenance/capex approvals;
- **municipal, tax, insurance and title compliance**;
- reserves and vacancy.

It feeds the income ledger (M12) and flags assets for halts or holds. This module is where RealT failed ⚑.

**Primary users:** `PROP`, `ASTM`, `X-PM` (scoped), `LEGL`, `TRSY`.

**Evidence:**
- **Vendor gap:** "Rent roll / NOI / property accounting — not observed in token vendors", and no PM-system sync in any token vendor [`APE` M2, gaps]. PM-system integration surfaces exist for AppFolio, Buildium, Yardi and RentRedi [`VND` §9].
- Obligation: [`XCC` #42 (US-C#16/17)].
- Platforms:
  - RealT ⚑ municipal compliance (certificates of compliance, code violations, blight tickets, water bills, tax delinquency, foreclosure countdown), title verification, litigation hold [`us/04-realt` §11 #11–12, #14];
  - Lofty St. Louis squatters and "80+ days without heat" complaint [`us/01-lofty` §10, §11 #11];
  - Assetora NDIS vacancies → freeze [`australia/05-assetora` §2.8];
  - mogul PM under-$1,000 authority and remittance by the 5th [`us/05-mogul` §5.6, §11.4];
  - PRYPCO Ejari and rent arrears [`mena/01-prypco` §11 #10];
  - BrickX rates, insurance and strata calendar [`australia/01-brickx` §11 #10].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 22.1 | Property ops dashboard | Occupancy; arrears; open work orders; **compliance score**; next 30 days of tax/insurance deadlines; vacancy days; PM scorecards; reserve months | Property list with RAG status per dimension | Market, PM, RAG | Open property | — | M |
| 22.2 | Rent roll | Scheduled vs collected; arrears ageing | Property/unit, tenant (masked), lease ref (**Ejari/Ejar**), rent, due, paid, arrears, deposit, lease end, **source** (AppFolio/Buildium/Yardi/RentRedi sync or CSV), bank match ✓, ledger ✓ | Arrears, source | Chase, annotate | Import / sync | M |
| 22.3 | Property managers | PM fees and SLA performance | PM, properties, fee % (e.g., 8–10% / 2.5% / ~5%), remittance timeliness, arrears rate, response time, complaints, score | Score | Review, start replacement (→ M17) | — | M |
| 22.4 | Work orders & capex | Open tickets; spend vs budget | Ticket, property, type (repair/capex/safety), quotes (two options), cost, **approval tier** (PM alone below $1,000 / `ASTM` / holder vote), vendor, status, evidence | Tier, status | Approve, reject, request quote | — | M |
| 22.5 | **Municipal & regulatory compliance ⚑** | Violations open; **tax-delinquency foreclosure countdown**; certificates missing | Property, obligation (rental registration/certificate of compliance, code violation, blight ticket, water/utility bill, property tax, safety items (heat, smoke detectors), specialist-housing registration), authority, notice date, hearing date, amount, status, owner | Status, authority | Record notice, assign, attach evidence | — | M |
| 22.6 | **Title & encumbrance monitor ⚑** | Deed-to-token reconciliation; new liens | Property, registry ref, owner of record = SPV ✓, liens, lis pendens, title insurance, last check | Mismatch | Re-check, escalate (`LEGL`) | — | M |
| 22.7 | Insurance & tax calendar | Expiring policies; bills due | Item, property, provider/authority, amount, due/expiry, paid evidence | Due | Mark paid, renew | — | M |
| 22.8 | Reserves & budgets | Reserve vs target; budget vs actual | Property, operating reserve (months), capex reserve, target, Δ, budget variance | Below target | Top-up request (→ M16) | — | M |
| 22.9 | Vacancy & leasing | Vacant units; days vacant; leasing pipeline | Unit, vacant since, asking rent, applications, status | Days vacant | Update | — | M |
| 22.10 | Property P&L & transparency | Per-property ledger for publication (Lofty/RealT "differentiate" advice) | Property, period, income, expenses, reserves, published ✓ | Period | Publish report (→ M21) | — | L |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Import / sync rent roll | `AUTO` / `PROP` | `AUTO` | — | Period `imported` | `propops.rentroll.imported` | `FINA` |
| Approve work order above PM authority | `PROP` → `ASTM` (above capex threshold → M17 vote) | `4E` | — | Ticket `approved` | `propops.workorder.approved` | PM |
| Record municipal notice / violation | `PROP` / `X-PM` | — | — | Obligation `open` | `propops.notice.recorded` | `ASTM`, `LEGL` |
| Pay property tax / municipal bill | `PROP` → `TRSY` / `CFO` | `4E` | — | Bill `paid` | `propops.bill.paid` | — |
| Escalate distressed property | `ASTM` → `CCO` | `4E` | Triggers M03 `distressed`, possible M14 halt and M15 collateral 0 | Asset `→ distressed` | `propops.escalated` | IR, market ops |
| Replace property manager | `ASTM` → `LEGL` (or vote outcome) | `4E` | — | PM assignment `v+1` | `propops.pm.replaced` | Holders |
| Request reserve top-up | `ASTM` → `CFO` | `4E` | — | Reserve `topped_up` | `reserve.topup` | — |
| Record court order (e.g., rent into escrow) | `LEGL` → `CCO` | `4E` | May pause distributions (M12) | Asset `legal_hold` | `propops.court_order.recorded` | Holders (disclosure) |

**Alerts & automations.**
- **Property-tax delinquency beyond 30 days ⚑** [`us/04-realt` §11.4].
- **New code violation or blight ticket ⚑**.
- Certificate of compliance missing or expired ⚑.
- Vacancy beyond 60 days [`us/04-realt` §11.4]; beyond N days for specialist housing [`australia/05-assetora` §11.4].
- Arrears beyond 30 days [`mena/01-prypco` §11.4].
- Rent collected below X% of scheduled.
- PM remittance missing by the 5th [`us/05-mogul` §11.4].
- Insurance expiry at T-30.
- **Tenant-safety complaint** (e.g., no heat) → Sev2.
- Reserve below target.
- **Court order received → Sev1** (RealT TRO redirected rent into escrow) [`us/04-realt` §2.8].
- Automations: nightly PM sync; the monthly compliance scorecard feeds the M03 compliance score and the investor property report.

---

### M23 — Support & Complaints · Confidence **M**

**Purpose.**
- Operational support tickets, with a guaranteed human escalation path.
- A **regulatory complaints register** with statutory clocks, redress, root-cause MI and external escalation to ombudsman/ADR bodies.

**Primary users:** `SUPP`, `CMPH`, `CCO`, `CFO` (redress).

**Evidence:**
- Platforms:
  - PRYPCO complaints policy: acknowledge ≤7 days, resolve ≤30 days (max 8 weeks), 8-year retention, DIFC Courts; Trustpilot complaints about AI-only support and a refund outstanding from Aug 2025 to Jul 2026 [`mena/01-prypco` §2.8, §10, §11 #11];
  - Propvest Salesforce tickets, 5-business-day SLA, Bundesbank arbitration [`eu/03-propvest` §11 #13];
  - BrickX IDR → AFCA [`australia/01-brickx` §11 #14];
  - Arrived BBB/FINRA complaint tracker [`us/03-arrived` §11 #13].
- Vendor: DigiShares admin inbox [`WLD` §1].
- Obligation: AU DDO complaints data [`REG-AU` §3].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 23.1 | Ticket queue | Open by category; first-response time; bot deflection vs **human escalation rate** | Ticket, investor, category (onboarding/payments/withdrawals/wallet recovery/tax forms/trading/distributions), channel, priority, SLA, assignee, state | Category, SLA | Reply, escalate, **convert to complaint** | Assign | M |
| 23.2 | Complaints register | Open complaints; ack and resolution clocks; redress paid | Complaint, investor, product, jurisdiction, category, received, **ack due**, **final response due** (parameterised: e.g., PRYPCO 7/30 days, max 8 weeks), state, root cause, redress, external body | Clock, category | Open | Export register | M |
| 23.3 | Complaint detail | Timeline; correspondence; investigation; redress calculator; approvals | — | — | Acknowledge, investigate, propose redress, final response | — | M |
| 23.4 | External escalations | Cases at AFCA / Bundesbank Schlichtungsstelle / FINRA / DIFC Courts | Case, body, deadline, status, outcome | Body | Record submission/outcome | — | M |
| 23.5 | Root cause & MI | Complaints by category/product/partner; DDO complaint data | Period, category, count, upheld %, redress | Period | Export (board, ASIC) | — | M |
| 23.6 | Knowledge base | Macros; articles; stale content | Article, topic, updated, views | — | Edit | — | L |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Convert ticket to complaint (mandatory when dissatisfaction is expressed) | `SUPP` | — | — | Complaint `received` | `complaint.received` | Investor ack (template) |
| Acknowledge | `AUTO` / `CMPH` | `AUTO` | — | `received → acknowledged` | `complaint.acknowledged` | Investor |
| Propose redress | `CMPH` → `CCO` (+ `CFO` above $X) | `4E`/`6E` | Refund/transfer via M16 | `investigating → redress_proposed` | `complaint.redress.proposed` | — |
| Issue final response | `CMPH` → `CCO` | `4E` | — | `→ response_issued → resolved` | `complaint.final_response` | Investor (incl. external-escalation rights) |
| Record external escalation | `CMPH` | — | — | `→ escalated_external` | `complaint.escalated` | `CCO`, `LEGL` |
| Reopen | `CMPH` | — | — | `resolved → reopened` | `complaint.reopened` | Investor |

**Alerts & automations.**
- Acknowledgement clock at day 5 of 7.
- Resolution clock at day 25 of 30 [`mena/01-prypco` §11.4].
- 8-week maximum breach → Sev2.
- Repeat complainant.
- Keywords "fraud", "misrepresent", "never owned" → `CCO` (cf. RealT "never owned" allegations [`us/04-realt` §2.8]).
- Complaint about marketing → M21.
- Refund outstanding more than 30 days.
- Automation: every chatbot conversation offers a human hand-off and logs a ticket.

---

### M24 — Partners, Distributors & Referrals · Confidence **M**

**Purpose.** Onboard and supervise everyone who brings supply or demand:
- sponsors/sellers/developers;
- distributors on the rails (the Ctrl Alt → PRYPCO "and others" model);
- advisers and placement agents;
- community clubs;
- referral/reward schemes, with a mandatory **broker-compensation review**;
- liquidity providers.

**Primary users:** `PART`, `CCO`, `CFO`, `LEGL`.

**Evidence:**
- Platforms:
  - Blocksquare Certified Partners and ambassadors [`eu/02-blocksquare` §3, §11];
  - Brickken partner marketplace (law firms, placement agents, custodians) [`eu/04-brickken` §11 A];
  - Ctrl Alt distributor management (VARA licence numbers, API scopes, SLAs) [`mena/02-ctrl-alt` §11 #1];
  - Reental distribution partner Akka (€300 minimum) [`eu/01-reental` §11 #18];
  - Assetora accredited advisers [`australia/05-assetora` §11 #4];
  - mogul Clubs with a 2% reward pool and the transaction-based compensation risk [`us/05-mogul` §2.3, §11 #10];
  - Lofty sellers retaining ≥10% and LPs [`us/01-lofty` §3];
  - Stake cashback/referrals [`mena/03-stake` §11 #11];
  - Domoblock referral bonuses taxed as income [`eu/05-domoblock` §11 #15].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 24.1 | Sponsors, sellers & developers | Deals by sponsor; realised vs underwritten | Sponsor, type, KYB, retained equity, fee agreement, deals, performance (yield vs pro forma), status | Type | Open, review | — | M |
| 24.2 | Distributors on rails | Volume by distributor; SLA; API health | Distributor, licence (e.g., VARA BD no.), jurisdictions, API scopes, due-diligence status, revenue share, volume, incidents | Status | Suspend, rotate keys (→ M26) | — | M |
| 24.3 | Advisers & placement agents | Authorisation status; clients | Firm/adviser, licence/CAR/tied-agent status, clients, campaigns, commissions | Licence status | Approve, suspend | — | M |
| 24.4 | Clubs & communities | Clubs; members; reward pool | Club, leader, members, invested via club, pool balance, moderation flags | Flags | Approve club, moderate | — | L |
| 24.5 | Referral & rewards ledger | Rewards accrued/paid; fraud flags; **broker-comp review status** | Reward, referrer, referee, trigger (sign-up/investment), amount/credit, tax treatment, review ✓, state | State, flag | Approve, reject, claw back | Pay batch | M |
| 24.6 | LPs & market makers | LP positions; MM agreements | Party, instruments, obligations, performance | — | Open | — | L |
| 24.7 | Commissions & partner billing | Accrued vs paid | Partner, period, basis, amount, invoice, status | Period | Approve payment | — | M |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Onboard partner (KYB + licence check) | `PART` → `CCO` | `4E` | Optional: register distributor role/API scope | Partner `review → active` | `partner.onboarded` | Partner |
| Set commission / revenue share | `PART` → `CFO` + `CCO` (**legal review if transaction-based**) | `6E` | — | Agreement `effective` | `partner.commission.set` | Partner |
| Approve reward payout | `PART` → `CFO` + `CCO` | `6E` | Credit (off-chain) | Reward `approved → paid` | `reward.paid` | Referrer |
| Suspend partner / distributor | `CCO` | `SGL+R` | Revoke scopes / API keys | `active → suspended` | `partner.suspended` | Partner, dependent teams |

**Alerts & automations.**
- Reward spike [`us/05-mogul` §11.4].
- Self-referral or ring patterns.
- Partner licence expiry.
- Distributor webhook/API down [`mena/02-ctrl-alt` §11.4].
- Commission configured on securities sales without a licence binding → block.

---

### M25 — Access Control, Approvals & Audit Log · Confidence **H**

**Purpose.** Govern who can do what:
- RBAC with tenant/entity/asset/jurisdiction scopes;
- **SoD** enforcement;
- **approval policies** (thresholds, quorums, TTLs) linked to custody policies;
- the approvals inbox;
- an **immutable audit log**;
- break-glass;
- access recertification;
- AI-agent principals.

**Primary users:** `SECO`, `SUPER`, `CCO`, all approvers, `X-AUD`, `X-REG`.

**Evidence:**
- Tokeny permission domains (Identities vs Assets write) [`WLD` §2].
- DS roles (Master/Issuer/Exchange/Transfer Agent) [`WLD` §3]; ATS roles (Admin/Minter/Controller/Pauser/KYC Manager) [`WLD` §11]; ERC-3643 Owner/Agent (an agent can be a multisig) [`TSD` §1.4].
- **Fireblocks quorum approvals, approval-group thresholds, admin quorum for policy changes, pending-approvals dashboard, authorisation history** [`WLD` §12]; **Taurus function-level governance** [`WLD` §13].
- Securitize activity log [`WLD` §3]; DigiShares 2FA [`WLD` §1].
- **Vertalo "delegation on every write"; Juniper "same permissions, same audit trail"** [`WLD` §8, §17].
- Retention: [`XCC` #44].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 25.1 | Users | Users by type; MFA coverage (target 100%); dormant privileged accounts | User, type (staff/external/service/agent), roles, scopes (tenant/entity/asset/jurisdiction), MFA/SSO, last login, device, status | Role, type, MFA | Edit roles, suspend, reset MFA | Access review | H |
| 25.2 | Roles & permissions | Role catalogue; permission diff | Role, module permissions (V/C/E/A/O/X), scopes, external-only constraints | — | Edit (→ approval) | — | H |
| 25.3 | SoD rules | Violations; active exceptions | Rule (conflicting role pair or action pair), severity, users in violation, exception expiry | — | Grant exception (time-boxed) | — | H |
| 25.4 | Approval policies | Policies by module; last changed; custody link | Action, maker roles, checker roles (n-of-m), thresholds, TTL, `SGL+R` ratification SLA, custody policy ID, version | Module | Propose change (admin quorum) | — | H |
| 25.5 | **Approvals inbox** | My pending; oldest; expiring | Approval, action, object, maker, amount/impact, **diff / simulation result / evidence**, required approvers, TTL | Module, urgency | Approve, reject (reason), comment | Bulk approve (low-risk types only) | H |
| 25.6 | Audit log | Events/day; **hash-chain verification status** | Time, actor, on-behalf-of (agent), action, object, tenant, approval ID, tx hash/registry ack, IP/device, before/after hash | Actor, object, action, time | View payload, open object | Export (audited) | H |
| 25.7 | Break-glass | Sealed accounts; active sessions; last drill | Account, holders (split credentials), activated by, reason, start/end, actions taken, post-review status | — | Activate (2 of N), end session | — | M |
| 25.8 | Agents (AI principals) | Active agents; actions/day; denials | Agent, delegating human, scopes (read / maker-write), rate limit, allowed modules, **cannot approve**, kill switch | — | Pause, edit scopes | — | M |
| 25.9 | Access reviews | Campaign completion | Reviewer, users, certified, revoked, due | — | Certify, revoke | — | M |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Invite user / grant role | `SUPER` → `SECO` (**no self-grant**; SoD check) | `4E` | Mirror on-chain role where the role has signing rights (→ M09 agents) | User `invited → active` | `iam.role.granted` | User, manager |
| Revoke role | `SUPER` / `SECO` | — | Remove signer/agent (M08/M09) | — | `iam.role.revoked` | User |
| Just-in-time elevation | User request → `SECO` | `4E` (time-boxed) | — | Elevation `active(until)` | `iam.jit.granted` | — |
| Change approval policy | `SECO` → admin quorum (`CCO` + `CFO` + CEO) | `6E` | Mirror in custody policy engine | Policy `v+1` | `approval.policy.changed` | All approvers |
| Approve / reject request | Checker | — | Releases custody signing | ApprovalRequest `submitted → approved/rejected` | `approval.decided` | Maker |
| **Activate break-glass** | 2 of (`CCO`, `SECO`, CEO) | `4E` | Enables emergency signer set | Break-glass `active (≤4h)` | `iam.breakglass.activated` | Board chair, `X-AUD` (post) |
| Register AI agent | `SECO` → `CCO` | `4E` | — | Agent `active` | `iam.agent.registered` | — |
| Export audit log | `CCO`, `X-AUD`, `X-REG` (scoped) | — | — | — | `audit.export.created` | — |

**Alerts & automations.**
- SoD violation → Sev2.
- Privileged login from a new country or device.
- MFA disabled.
- Approval-policy change.
- Break-glass active → Sev1 notification to the board chair.
- **Audit hash-chain verification failure → Sev1.**
- Dormant privileged account older than 30 days.
- Agent exceeding scope or rate.
- Automations: quarterly access review; approval TTL expiry; role auto-expiry for contractors.

---

### M26 — Integrations, API & Webhooks · Confidence **M** (APIs observed; webhooks not observed in fetched vendor pages)

**Purpose.** Manage every external dependency:
- connectors (KYC, KYT, custody, payments, registries, venues, oracles, PM systems, e-sign, comms);
- API keys and scopes for tenants/partners;
- webhooks;
- MCP/agent tooling;
- schedulers;
- chain and node configuration;
- sandboxes.

**Primary users:** `ENGO`, `SECO`, `PART`.

**Evidence:**
- Vendors: DigiShares API (1M calls/month) [`APE` M16 C#1]; Tokeny API v1/v2 [`APE` M16 C#7]; **Securitize RFE API** [`WLD` §3]; Hedera ATS SDK and custody integrations [`WLD` §11]; Vertalo/Juniper/Agora MCP [`WLD` §8, §17]; Brickken REST/SDK/CLI/MCP, x402 metering, sandbox [`eu/04-brickken` §6–7].
- **"Webhooks — not explicitly observed"** [`APE` M16].
- Vendor surfaces [`VND` all].
- Platform: Ctrl Alt distributor API keys and webhook health [`mena/02-ctrl-alt` §11 #1].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 26.1 | Connectors | Health per category; error rates | Connector (KYC: Sumsub/Entrust/Persona/IDWise/Equifax/Nafath/UAE PASS/Averer · accreditation: iCapital/Verify Investor · KYT: Chainalysis/TRM/Elliptic · Travel Rule: Notabene · custody: Fireblocks/BitGo/Ripple Custody/Dfns/Zodia/Tangany · payments: Stripe/Plaid/Dwolla/Modern Treasury/Mangopay/Checkout/Lean/Monoova/Zepto/banks · stablecoin: Circle Mint/AUDM/AE Coin · e-sign · PM: AppFolio/Buildium/Yardi/RentRedi · registries: DLD via issuer, RER Aamal, TA APIs · venues/ATS/DINO/RFE · oracles · indexers · RPC · comms/CRM), env, version, credential expiry, status, last sync | Category, status | Configure, test, rotate credential | — | M |
| 26.2 | API keys & scopes | Keys by tenant/partner; usage vs quota | Key, owner, env (sandbox/prod), scopes (module × read/write), IP allow-list, created, last used, expires, usage | Owner, scope | Rotate, revoke | — | H |
| 26.3 | Webhooks | Deliveries; failures; DLQ size | Subscription, event types (e.g., `offering.*`, `order.*`, `distribution.*`, `token.*`, `kyc.*`, `recon.break.*`), endpoint, signing secret age, success % | Status | Replay, pause, rotate secret | Replay DLQ | L |
| 26.4 | Agents & MCP | Tools exposed; calls/day; denials | Tool, module, mode (read / maker-write), mandate (scope, limits, expiry), delegating user | Mode | Enable/disable, edit mandate | — | M |
| 26.5 | Jobs & schedulers | Failed jobs; next runs | Job (snapshots, payouts, recon, screening, PM sync, reports, KYT re-screen), schedule, last run, duration, status | Status | Run now, pause | — | M |
| 26.6 | Chains & nodes | RPC latency; failovers; indexer lag; **contract address book** | Network, RPC primary/fallback, indexer (The Graph/Envio/in-house), lag, gas oracle, contracts | Network | Switch RPC (→ approval) | — | M |
| 26.7 | Sandbox | Tenant sandboxes; faucet balances | Sandbox, tenant, chain (testnet), data seed, expires | — | Reset, extend | — | M |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Add / modify connector (prod) | `ENGO` → `SECO` | `4E` | — | Connector `active` | `integration.connector.changed` | Module owners |
| Create API key with write scope | `ENGO` / `PART` → `SECO` | `4E` | — | Key `active` | `api.key.created` | Owner |
| Revoke API key | `SECO` | — | — | `revoked` | `api.key.revoked` | Owner |
| Add webhook endpoint | `ENGO` → `SECO` | `4E` | — | Subscription `active` | `webhook.subscription.created` | — |
| Enable MCP write (maker) tool | `SECO` → `CCO` | `4E` | — | Tool `enabled` | `agent.tool.enabled` | — |
| Switch RPC / network config | `ENGO` → `SECO` | `4E` | — | Config `v+1` | `chain.config.changed` | TokOps |

**Alerts & automations.**
- KYC provider degraded → queue banner in M07.
- Credential expiring in 14 days.
- Webhook DLQ growth.
- API key used from a new IP.
- RPC failover.
- Indexer lag above 5 min.
- Snapshot or payout job failure → Sev2.
- Agent anomaly → auto-pause.

---

### M27 — Platform Settings, Tenancy & White-label · Confidence **H**

**Purpose.**
- Multi-tenant administration: tenants, plans, capacity limits, guardrails, billing.
- Binding jurisdiction packs.
- Fee schedules.
- **Licence-bound feature flags and kill switches.**
- Branding/domains.
- Global settings: residency, retention, business calendars.

**Primary users:** `SUPER`, `CCO`, `CFO`, `PART`, `X-ISA` (own branding).

**Evidence:**
- Vendors:
  - DigiShares white-label (logo, colours, custom URL), provider configuration, capacity tiers (200/2,000/10,000), multi-project pricing [`WLD` §1];
  - Blocksquare custom domain, "new marketplace", payments & invoicing, pricing (US$5,990/tokenisation + US$69/property/month), sBST discount tiers [`WLD` §5];
  - Securitize fundraise-site configuration [`WLD` §3];
  - Brickken plans and **compliance guardrails** [`eu/04-brickken` §11 A];
  - Tokeniser per-investor pricing [`australia/03-tokeniser` §1];
  - two-tier pattern [`WLD` §18.1].
- Platforms: Ctrl Alt distributor tenants [`mena/02-ctrl-alt` §11 #1]; Assetora "stop new business" freeze switch [`australia/05-assetora` §11 #11]; Saudi in-Kingdom hosting [`mena/04-saudi` §7].

| # | Page | Widgets / KPIs | Main table — columns | Filters | Row actions | Bulk actions | Conf. |
|---|---|---|---|---|---|---|---|
| 27.1 | Tenants | Tenants by plan; guardrail violations; capacity usage | Tenant, type (issuer/sponsor, marketplace operator/CP, distributor), plan/tier, licences, KYB, users/investor cap, projects, discount tier, status, violations | Plan, status | Open, suspend | — | H |
| 27.2 | Tenant detail | Config: jurisdictions allowed, rule packs, **whitelisting mode** (auto on KYC vs admin confirm), **KYC required (locked on in prod)**, rails, chains, branding, users, billing, guardrails | — | — | Edit (→ approval where marked) | — | H |
| 27.3 | Jurisdiction bindings | Packs per tenant/entity; restricted-jurisdiction lists | Scope, pack, version, effective, restricted countries | Pack | Bind, unbind | — | M |
| 27.4 | Fee schedules | Fees in force; upcoming changes and notice periods | Fee, product, rate/formula, split (e.g., 0.5/0.5/0.5), effective, notice sent, disclosure ref | Product | Propose change | — | H |
| 27.5 | Feature flags & kill switches | Flags per tenant/jurisdiction; licence binding ✓ | Flag (secondary venue mode, lending, AMM, stablecoin payouts, crypto deposits, gold, AutoInvest, clubs, **global inflow freeze**, **marketplace suspend**), scope, state, licence ref | Scope | Toggle (→ approval) | — | H |
| 27.6 | Branding & domains | Domains; certificate expiry | Tenant, logo, colours, domain/path, email sender, app listing, languages | — | Edit | — | H |
| 27.7 | Billing | MRR; overdue; usage | Invoice, tenant, basis (setup, per tokenisation, per property/month, per investor/month, API calls), amount, due, status | Status | Issue, credit note | Run billing | M |
| 27.8 | Global settings | Data residency; retention; **business-day calendars** per jurisdiction (used by every SLA clock); currencies; locales | Setting, value, scope, changed by | — | Edit (→ approval) | — | M |

**Action catalogue**

| Action | Roles | M-C | On-chain | State transition | Audit events | Notifications |
|---|---|---|---|---|---|---|
| Create tenant | `PART` / `SUPER` → `CCO` (KYB) | `4E` | Optional role grant in protocol registry (e.g., Blocksquare `CertifiedPartners`) | Tenant `onboarding → active` | `tenant.created` | Tenant admins |
| Set tenant guardrails | `CCO` | `4E` (`LEGL`) | — | Guardrails `v+1` | `tenant.guardrails.changed` | Tenant |
| Suspend tenant | `CCO` | `SGL+R` | Revoke tenant roles/agents | `active → suspended` | `tenant.suspended` | Tenant, investors per plan |
| Change fee schedule | `CFO` → CEO (notice period) | `4E` | Fee setters where on-chain | Fee `scheduled(effective)` | `fees.changed` | Investors/tenants (notice) |
| Toggle feature flag | `SUPER` → `CCO` (**must be licence-bound**) | `4E` | — | Flag `on/off` | `flag.toggled` | Module owners |
| **Global inflow freeze / marketplace suspend** | `CCO` or RE board delegate | `SGL+R` | Optional token pause (M09) | Platform `freeze_active` | `platform.freeze.activated` | All investors; regulator per runbook |
| Change data residency / retention | `SECO` → `CCO` + `LEGL` | `6E` | — | Setting `v+1` | `settings.residency.changed` | — |

**Alerts & automations.**
- **Tenant guardrail violation** (KYC off in prod, offering above exemption threshold, restricted jurisdiction) [`eu/04-brickken` §11 A].
- Tenant investor count crossing a plan tier [`australia/03-tokeniser` §11.4].
- Licence instalment overdue; stake below discount tier [`eu/02-blocksquare` §11.4].
- Feature flag on without licence binding → auto-revert.
- Domain certificate expiry.

---

## 3. Appendix A — On-chain action crosswalk (quick reference)

Condensed from `TSD` §4 and platform evidence. `✘` means there is no native primitive; implement it off-chain or in a custom contract.

| Admin action (module) | ERC-3643 (RealX default) | ERC-7943 view/iface | Algorand ASA | XRPL (MPT / trust-line) | Hedera HTS / ATS | Stellar | Solana Token-2022 | Blocksquare / Brickken |
|---|---|---|---|---|---|---|---|---|
| Grant eligibility (M07) | `registerIdentity` + claims | `canSend/canReceive` true | opt-in + unfreeze | `MPTokenAuthorize` / Credentials | `TokenGrantKyc` / ATS control list | `SetTrustLineFlags` authorise | thaw / transfer-hook allow-list | `addWalletList` / `whitelist` |
| Mint (M09) | `mint`/`batchMint` | n/a | transfer from reserve | Payment from issuer | `TokenMint` | Payment from issuer | `mintTo` | `mint` / `mintToken` |
| Burn / redeem (M09) | `burn`/`batchBurn` | n/a | clawback to reserve | `Clawback` / redeem | `TokenBurn`/`TokenWipeAccount` | payment back / `Clawback` | burn / permanent delegate | `burn` / `burnToken` |
| Freeze wallet (M09) | `setAddressFrozen` | `setFrozenTokens` | `AssetFreeze` | lock holder / TrustSet freeze | `TokenFreezeAccount` | deauthorise | `freeze_account` | `freezeToken` (per property) / ✘ |
| Partial freeze / lock (M09/M10) | `freezePartialTokens` | `setFrozenTokens(amount)` | ✘ | escrow | ATS locks | ✘ | ✘ | ✘ |
| Pause (M09) | `pause`/`unpause` | ✘ | ✘ (no global) | global lock | `TokenPause` | ✘ | Pausable ext. | ✘ |
| Forced transfer (M09) | `forcedTransfer` | `forcedTransfer` | clawback | `Clawback` + re-issue | controller / wipe + transfer | `Clawback` | permanent delegate | ✘ / `transferFrom` |
| Recover lost wallet (M09) | `recoveryAddress` | via `forcedTransfer` | clawback + re-send | clawback + re-issue | controller | clawback + re-issue | permanent delegate | ✘ / re-issue |
| Compliance rule (M10) | `addModule` / module setters | `canTransfer` | ARC-20 | Permissioned Domains | ATS "Customize" | SEP-8 approval server | transfer hook | ✘ / offering params |
| Documents (M18) | off-chain + claims/metadata | ✘ | metadata URL | MPT metadata URI | ATS documents | stellar.toml | metadata ext. | IPFS hash in registry |
| Distribution (M12) | stablecoin batch transfer | ✘ | ✘ (off-chain) | ✘ | ATS dividends / Mass Payout | ✘ | ✘ / batch | `addRevenue` + push/claim / `dividendDistribution` |
| Agents / roles (M09/M25) | owner-appointed agents (ERC-173) | ✘ | rekey roles (**irreversible if emptied**) | `SignerListSet` | key updates / ATS roles | signer weights | authorities | Roles contract / tenant roles |

---

## 4. Appendix B — Items needing validation before build (lowest-confidence areas)

| Item | Why it is low confidence | Where it matters |
|---|---|---|
| Webhook catalogue and event contracts | Not observed on any fetched vendor page [`APE` M16] | M26, partner integrations |
| SAR/STR case workflow | "Not observed in any vendor" [`APE` M13] | M20 (design from regulation) |
| Withholding engine for on-chain distributions | Not observed in token vendors [`APE` gaps] | M12/M13 |
| Rent-roll / PM-system sync | Not observed in token vendors [`APE` gaps] | M22 |
| Order-book circuit breakers and price bands | Not observed in vendors; platform-specific [`APE` M9] | M14 |
| DLD distributor API choreography | Exact API sequence not published [`mena/01-prypco` §5.2 note] | M11 registry sync, M05 settlement |
| RER Aamal API and sandbox reporting format | No public OpenAPI [`mena/04-saudi` §7] | M11, M19 |
| ARVA reserve rules mapped onto title-referenced tokens | "Not published" [`mena/02-ctrl-alt` §2.2] | M16 reserves, M19 prudential |
| AU DAF Act: fit of fractional MIS-unit tokens with the TCP definition | The Act itself is **confirmed** (Corporations Amendment (Digital Assets Framework) Act 2026, No. 38 of 2026; Royal Assent 8 Apr 2026; commences 8 Apr 2027; 6-month transition; $10m/$5k low-value exemption) by the synthesis re-check [`04-synthesis/conflicts-and-gaps.md` #1, F1/F5]. What is unverified is whether fractional unit tokens are TCP products at all: s761GD reads as one token per underlying asset [`conflicts-and-gaps.md` #2] | M02 licences, M19 calendar, M10 AU pack |
| ERC-3643 agent-management function names | Resolved in QA: `addAgent` / `removeAgent` / `isAgent` are declared in the EIP's `IAgentRole` interface (eips.ethereum.org/EIPS/eip-3643, checked 23 Sep 2026). `addModule` / `removeModule` come from the T-REX reference implementation (`IModularCompliance`), not the EIP text | M09/M10 implementation |
| Aave-fork configurator function names | [I] | M15 implementation |
