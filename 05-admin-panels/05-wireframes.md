# 05 — Annotated ASCII Wireframes (Key Admin Screens)

> **EXTRAPOLATION.** These are illustrative low-fidelity layouts for the most important screens in `01-reference-admin-spec.md`. All data is fictitious. Numbered callouts `[n]` are explained under each screen. Width is 108 columns or less. Date: 23 Sep 2026.

| # | Screen | Module | Primary users |
|---|---|---|---|
| W1 | Command Center dashboard | M01 | CCO, CFO, heads of ops |
| W2 | Investor 360 profile | M06 | SUPP, KYCA, IRCM, MLRO |
| W3 | KYC review queue item | M07 | KYCA, MLRO |
| W4 | Offering builder (multi-step) | M04 | CAPM, CCO, LEGL |
| W5 | Offering live raise monitor | M04/M05 | CAPM, CFO |
| W6 | Token Operations console with maker-checker panel | M09 | TOKO, CCO, CFO, X-CUS |
| W7 | Cap table and 3-way reconciliation | M11 | REGO, CCO, CFO, X-TA |
| W8 | Distribution run review | M12 | FINA, CFO |
| W9 | Secondary market surveillance | M14 | MKTO, CCO |
| W10 | Property operations and compliance | M22 | PROP, ASTM, LEGL |
| W11 | Approvals inbox and audit log | M25 | All approvers, X-AUD |


---

## W1 — Command Center dashboard (M01)

```text
+----------------------------------------------------------------------------------------------------------+
| RealX Admin  [Tenant: RealX Direct v] [Entity: All v]  PROD   ETH ok  POLY ok  XRPL lag 2m   Q Search... |
|                                               [Approvals 7]  [Alerts 3]  [Break-glass]  [CFO-1 v]        |
+-------------------+--------------------------------------------------------------------------------------+
| > Command Center  | COMMAND CENTER   Range [MTD v]  Region [All v]  Role view [CFO v]  [PDF]             |
+-------------------+-------------+-------------+-------------+-------------+-------------+----------------+
|   Pipeline        | AUM         | Raised MTD  | Pledged/    | Paid MTD    | Recon       | Approvals      |
|   Investors       | $412.6M     | $18.2M      | funded $M   | $2.91M      | breaks      | pending        |
|   On-chain        | +3.1% MoM   | 112% plan   | 24.1/19.8   | 99.6% ok    | 2 (Sev2)    | 7 (2 < 24h)    |
+-------------------+------------------------------------------+-------------------------------------------+
|   Money           | [1] OPERATIONS QUEUES                    | [2] COMPLIANCE CLOCKS                     |
|   Markets         | KYC review       41  oldest 19h SLA 24h  | SAR draft   case 8812    due 4d  MLRO     |
|   Asset life      | Accreditation     9  oldest 2d           | Form D      SPV-114      due 6d  LEGL     |
|   Risk & Compl.   | Bank matches     12  oldest 51h BREACH   | DDO report  TMD-07       due 3bd CCO      |
|   Engagement      | Withdrawals       6  over $25k: 2        | Cap meter   Reg CF       91%     CAPM     |
|   Platform        | Whitelist reqs   33                      | 12(g)       SPV-021      1,640 / 2,000    |
|                   | DLD sync pending  3  oldest 40m          | Licence     VARA cond.   renew in 28d     |
+-------------------+------------------------------------------+-------------------------------------------+
|                   | [3] CHAIN & CUSTODY                      | [4] TREASURY                              |
|                   | Stuck tx 1 (mint batch 221, 34m) [open]  | Client money bank   12,440,210.55 AED     |
|                   | Pending signatures 2 (oldest 12m)        | Client ledger       12,440,210.55  D=0    |
|                   | Paused tokens 0  |  Oracle age 3d ok     | USDC float 1.92M (9.4 days of payouts)    |
|                   | Paymaster runway 11 days                 | Next 7d payouts 1.41M | Escrow 3.2M       |
+-------------------+------------------------------------------+-------------------------------------------+
|                   | [5] TOP OFFERINGS   %fund days soft cap  | [6] ALERTS   Sev1 0 | Sev2 2 | Sev3 5     |
|                   | Marina 2BR AE-DLD    98%  0.3  met       | S2 recon break TOK-SVQ6-B qty 12          |
|                   | Austin SFR14 506c    61%  9    at risk   | S2 bank match > 48h (x3)                  |
|                   | Madrid Flip9 ECSP    44%  12   --        | S3 KYC provider latency p95 40s           |
+-------------------+------------------------------------------+-------------------------------------------+
```

**Annotations.**

- **Global header:** tenant and entity switchers, environment badge, per-chain health pills, global search (investor / wallet / tx hash / TokenID), Approvals and Alerts badges, and the break-glass entry point (M25.7, visible only to authorised roles).
- **KPI tiles:** *Pledged vs funded* comes from the Securitize Control Panel raise KPIs [`WLD` §3]. The tiles are role-filtered ("Role view").
- **[1] Operations queues:** every queue shows count, oldest item and SLA. The 24h KYC SLA comes from Blocksquare operator practice [`eu/02-blocksquare` §2.5]. DLD sync pending applies to A3 only [`mena/01-prypco` §11 #5].
- **[2] Compliance clocks:** combined feed of M19 filings, the M20 SAR clock, M04 cap meters and M10 holder monitors (12(g) 2,000 threshold, cf. RealT's per-property cap [`us/04-realt` §2.3]).
- **[3] Chain & custody:** stuck-transaction and pending-signature views mirror Brickken's prepared/sent/mined monitor [`eu/04-brickken` §11 A] and the Fireblocks pending-approvals dashboard [`WLD` §12]. Paymaster runway follows Reental [`eu/01-reental` §11 #15].
- **[4] Treasury:** client-money reconciliation must show D=0 (PRYPCO Zand ↔ ledger pattern [`mena/01-prypco` §11 #6]). The float is shown as days of upcoming payouts (Lofty alert [`us/01-lofty` §11.4]).
- **[5]/[6]:** "soft cap at risk" follows Brickken's alert [`eu/04-brickken` §11.4]. Alert severities follow `01` §0.5.

---

## W2 — Investor 360 profile (M06)

```text
+----------------------------------------------------------------------------------------------------------+
| Investors > INV-004821  Amina K. (individual)   [KYC valid to 2027-03-01]  [Class: AE Qualified (VARA)]  |
| Residency AE | Nationality IN | Risk: Medium | PEP: No | Tier: Pre-sale | Channel: Distributor D-07      |
| [Suspend] [Request docs] [Open case] [Message] [Start recovery]                PII masked  [Unmask*]     |
+----------------------------------------------------------------------------------------------------------+
| Profile | Limits | Wallets | Holdings | Orders | Payouts | Tax | Docs | Cases | Comms | Votes | Audit    |
+----------------------------------------------------+-----------------------------------------------------+
| [1] LIMITS & CAPS                                  | [2] WALLETS                                         |
| DFSA per property AED 120,000/183,500  65%         | 0x9f..21c passkey-AA proof:sig KYT 12  IR ok        |
| DFSA annual       AED 240,000/367,000  65%         | 0x3a..e07 external  proof:none KYT 71  (!)          |
| Holding cap Marina 2BR   14.2% / 20%   71%         |   -> ownership proof required before use            |
| Suitability: passed 2026-02-10 (v7)                | Omnibus sub-account CA-OMNI-3 / sub 55812           |
+----------------------------------------------------------------------------------------------------------+
| [3] HOLDINGS  as of [2026-09-23 v]                                                                       |
| Token          Units    %prop  Frozen  Lock-up end  Legal record             Value (NAV)                 |
| MARINA-2BR     18,460   14.2%  0       2026-11-02   DLD cert TKN-77120       AED 36,920                  |
| AUSTIN-SFR14      120    6.0%  0       2027-08-01   TA entry A-2231          USD 12,000                  |
+----------------------------------------+-------------------------------+---------------------------------+
| PAYOUTS (last 3)                       | [4] TAX                       | [5] CASES & FLAGS               |
| 09-04 MARINA  AED 211.40  paid         | W-8BEN valid 2028-01          | AML: none open                  |
| 08-05 MARINA  AED 208.10  paid         | Treaty rate 15% (US div.)     | Complaint C-311 resolved        |
| 07-31 AUSTIN  USD 61.20   WHT 15%      | CRS: AE reportable            | Support: 2 tickets              |
| Reinvest OFF | Payout asset: AED       |                               | Votes: 3 cast / 3 eligible      |
+----------------------------------------+-------------------------------+---------------------------------+
```

**Annotations.**

- **Header chips:** KYC validity, classification per jurisdiction, risk, PEP, tier and acquisition channel. PII is masked until unmasked with a reason code (audited as `pii.view.unmasked`).
- **[1] Limits & caps:** DFSA crowdfunding retail caps (AED 183.5k per property / 367k per year) [`mena/03-stake` §2.6]; per-property holding cap (PRYPCO 20%) [`mena/01-prypco` §2.6]; suitability test version.
- **[2] Wallets:** type, ownership proof and KYT score per address. External self-hosted wallets need proof (EU TFR > €1,000; UAE unhosted EDD) [`REG-EU` §4; `REG-ME` §4]. Omnibus sub-accounts are shown for A3.
- **[3] Holdings:** the as-of date picker gives historical look-back [`WLD` §2]. The **legal record** column links each position to the DLD certificate, TA entry or unit register.
- **[4] Tax:** form validity and treaty rate feed the withholding engine (M13). Lofty's 30%/15% rates are the pattern [`us/01-lofty` §2.7].
- **[5] Cases & flags:** AML detail is **not** shown to support roles (tipping-off guard). Only the existence of a restriction is visible.

---

## W3 — KYC review queue item (M07)

```text
+----------------------------------------------------------------------------------------------------------+
| Onboarding > KYC queue > KYC-55190   Provider: Sumsub  Decision: REFER   Risk 62 (medium-high)   SLA 5h  |
+--------------------------------------------+-------------------------------------------------------------+
| [1] DOCUMENTS                              | [2] EXTRACTED vs ENTERED                                    |
| +----------------+ +----------------+      | Field       Document       Applicant      Match             |
| | PASSPORT       | | SELFIE /       |      | Name        K. AMINA       Amina K.       ~ order           |
| | (image)        | | LIVENESS 0.97  |      | DOB         1988-04-11     1988-04-11     ok                |
| +----------------+ +----------------+      | Doc expiry  2031-02-02     --             ok                |
| Proof of address: utility bill 2026-08     | Address     Dubai Marina   Dubai Marina   ok                |
+----------------------------------------------------+-----------------------------------------------------+
| [3] SCREENING HITS                                 | [4] RISK FACTORS                                    |
| PEP list 'Amina Khan' score 0.71, DOB mismatch     | Country risk          low           +0              |
|   [False positive] [True match] [Need info]        | Product risk          ARVA retail   +10             |
| Adverse media 0 | Sanctions 0 | Wallet KYT 71 (!)  | Wallet exposure       mixer 2 hops  +30             |
|                                                    | Source of funds       pending       +22             |
+----------------------------------------------------------------------------------------------------------+
| [5] SOURCE OF FUNDS / WEALTH (VARA)   Declared: salary + property sale   Evidence: 1 of 2 uploaded       |
+----------------------------------------------------------------------------------------------------------+
| [6] DECISION   ( ) Approve   ( ) Reject   (x) Request info   ( ) Escalate to EDD (MLRO)                  |
| Reason code [SOF_MISSING v]   Note [Need bank statement showing sale proceeds ...............]           |
| RFI template [EN/AR v] -> investor; expiry timer 30 days            [Submit]   Checker required: no      |
| On approve -> eligibility job: registerIdentity(0x9f..21c, ONCHAINID, 784) + KYC claim (topic 1)         |
+----------------------------------------------------------------------------------------------------------+
```

**Annotations.**

- **Header:** provider and provider decision (e.g., Sumsub REFER; Brickken's six-state model [`eu/04-brickken` §2.5]), risk score, SLA.
- **[1]/[2]:** document and liveness evidence side by side with extracted vs entered data. The reviewer adjudicates mismatches.
- **[3] Screening hits:** each hit is adjudicated. *True match* immediately restricts the account (`SGL+R`) and opens an M20 case. *False positive* needs `MLRO` as checker (`02` §3 row 4).
- **[4] Risk factors:** explainable scoring. Wallet exposure comes from KYT (Chainalysis/TRM/Elliptic) [`VND` §3].
- **[5] SoF/SoW:** mandatory under VARA classification rules [`REG-ME` §3] and PRYPCO CDD gating [`mena/01-prypco` §2.5].
- **[6] Decision:** reason codes are mandatory. Approval triggers the **eligibility grant** job: ERC-3643 `registerIdentity(wallet, ONCHAINID, country=784)` plus KYC claim [`TSD` §4]. Overrides of provider rejects need `MLRO`.

---

## W4 — Offering builder, step 2 of 9 (M04)

```text
+----------------------------------------------------------------------------------------------------------+
| Offerings > New offering 'Austin SFR 14'                          Draft v3   [Save]  [Simulate]  [Close] |
| 1 Asset > [2 REGIME] > 3 Economics > 4 Raise > 5 Eligibility > 6 Token > 7 Docs > 8 Pages > 9 Review     |
+--------------------------------------------------------+-------------------------------------------------+
| [1] STEP 2 - REGIME / EXEMPTION                        | [2] CONSTRAINTS LOADED (pack US-506c v4)        |
| Jurisdiction [United States v]                         | Investors: accredited only, verified            |
| Regime  ( ) Reg D 506(b)   (x) Reg D 506(c)            | General solicitation: allowed                   |
|         ( ) Reg A+ Tier 2  ( ) Reg CF                  | Raise cap: none; Form D <= 15d after sale       |
| Parallel tranche [x] Reg S (pack US-RegS v2)           | Resale: Rule 144 1-year lock (token lock)       |
| Issuer [SPV-114 Austin SFR 14 LLC v]                   | Docs: PPM, subscription agmt, Form D            |
| Broker-dealer [Partner BD v]  TA [External TA v]       | Tax: K-1, or 1099-DIV if REIT election          |
| Escrow [Bank escrow v]                                 | Holder monitor: 12(g) 2,000 / 500 non-accr.     |
+----------------------------------------------------------------------------------------------------------+
| [3] VALIDATION  x Reg S tranche needs offshore geo-block rule    x Form D task not created in calendar   |
|                 ! Allow min-investment verification path ($200k natural / $1M entity + reps)?  [Yes/No]  |
+----------------------------------------------------------------------------------------------------------+
| Preview step 4: soft cap $1.2M | hard cap $2.0M | min ticket $10k | per-investor cap 20% | window 30d    |
| Failure mode (x) revert & refund  ( ) finalise partial  | Oversubscription: pro-rata | Pre-sale: off     |
+----------------------------------------------------------------------------------------------------------+
|                                   [< Back]   [Next: Economics >]   [Submit for compliance review]        |
+----------------------------------------------------------------------------------------------------------+
```

**Annotations.**

- **Stepper:** asset → **regime** → economics → raise → eligibility → token → docs → pages → review. Drafts are versioned. *Simulate* runs rule and cap checks.
- **[1] Regime selector:** binds the offering to an exemption/regime pack that auto-loads constraints [`XCC` #2]. Parallel Reg D 506(c) + Reg S tranches follow RedSwan's dual track [`us/02-redswan` §2.3].
- **[2] Constraints loaded:** investor class, solicitation, caps, filings (Form D ≤ 15 days [`REG-US` §2]), resale lock (Rule 144) and holder monitors. They are read-only here and changed only through M10 pack versions.
- **[3] Validation:** blocking (x) and warning (!) items. The minimum-investment verification path ($200k/$1M plus reps) follows the 12 Mar 2025 NAL [`REG-US` §3].
- **Raise mechanics preview:** soft/hard cap, failure mode (Blocksquare finalise vs revert [`APE` M3]), oversubscription rule, pre-sale tier (Domoblock [`eu/05-domoblock` §1]).

---

## W5 — Offering live raise monitor (M04/M05)

```text
+----------------------------------------------------------------------------------------------------------+
| Offerings > OFF-2291 Marina 2BR (AE-DLD rails)   LIVE  closes in 07:42:10   [Pause] [Extend] [Close now] |
+----------------------------------------------------------------------------------------------------------+
| Target AED 2.40M | Pledged 2.356M (98.2%) | Funded 2.19M (91.3%) | Investors 221 | Waitlist 4,180        |
| Pledged [#################################################################################-]  98%        |
| Funded  [##########################################################################--------]  91%        |
+--------------------------------+------------------------------------+------------------------------------+
| [1] FUNNEL (24h)               | [2] TICKET SIZES                   | JURISDICTION / CLASS               |
| Viewed          9,812          | < 5k      ##########  131          | AE residents (EID) 100%            |
| KYC passed      1,044          | 5k-20k    #####        62          | Retail 87% | Qualified 13%         |
| Committed         221          | 20k-100k  ##           24          | 20% cap blocks today: 3            |
|   in cooling-off    38         | > 100k    #             4          | Nationalities: 44                  |
| Funded            192          |                                    |                                    |
+----------------------------------------------------------------------------------------------------------+
| COMMITMENTS                                                               [Export] [Remind unfunded]     |
| Investor     Amount AED  State             Cooling-off ends  Funds     Flags                             |
| INV-004821       18,460  reflection_period 2026-09-24 09:12  received  --                                |
| INV-010233       40,000  awaiting_funds    --                pending   bank match > 24h                  |
| INV-011870       96,000  eligibility_check --                --        20% cap -> capped at 48,000       |
+----------------------------------------------------------------------------------------------------------+
| [3] CLIENT MONEY: Zand client a/c 2,190,000 = funded commitments (D=0)    Next: allocation -> finalise   |
+----------------------------------------------------------------------------------------------------------+
```

**Annotations.**

- **Progress bars:** pledged vs funded (Securitize [`WLD` §3]). Waitlist size reflects PRYPCO-style demand (10,700+ waitlist) [`mena/01-prypco` header "Scale"].
- **[1] Funnel:** viewed → KYC → committed (including those in **cooling-off**, 24h at PRYPCO [`mena/01-prypco` §2.8]) → funded.
- **[2] Tickets and eligibility:** ticket histogram; jurisdiction/class mix; count of 20% cap blocks. Orders above the cap are capped, not silently accepted.
- **Commitments table:** order states follow `03` §5. Flags include bank match > 24h and cap breaches.
- **[3] Client money:** the escrow/client account must equal funded commitments (D=0) before allocation and finalisation (`02` §3 rows 12, 35).

---

## W6 — Token Operations console with maker-checker pending panel (M09)

```text
+----------------------------------------------------------------------------------------------------------+
| Token Ops > MADRID-FLIP9 (ERC-3643, Polygon) 0x71c..9ab   supply 30,000 / cap 30,000   ACTIVE   [PAUSE]  |
| Owner: Safe 3-of-5 | Agents: V2 token-agent MPC 2-of-3, V3 pauser, V4 identity agent | IR 0x44..10       |
+----------------------------------------------------------+-----------------------------------------------+
| [1] NEW ACTION                                           | [2] PENDING MAKER-CHECKER                     |
| Action [Forced transfer v]                               | A-7781 Recovery INV-009 -> 0x5e..c11          |
| From 0x3a..e07   To 0x5e..c11   Qty [ 120 ]              |   SUPP > KYCA ok | CCO ok | CFO pending       |
| Reason [COURT_ORDER v]                                   |   timelock ends 2026-09-25 10:00              |
| Legal basis [Madrid court order 44/2026]                 | A-7790 batchMint alloc OFF-2288 (x212)        |
| Evidence court_order.pdf sha256 9c1e..  [+ add]          |   TOKO | CFO ok | CCO ok | custody 1/2        |
| Timelock 48h  [ ] waive (needs CCO + CEO)                | A-7794 Unfreeze 0x88..f1                      |
|                                                          |   MLRO | CCO pending                          |
+----------------------------------------------------------+-----------------------------------------------+
| [3] PRECHECK                                             | [4] CUSTODY - V2 token-agent vault            |
| canTransfer(to) ok | to in IdentityRegistry ok (ES 724)  | Tx 221 batchMint  signers 1/2                 |
| token paused: no | from partially frozen: 120            |   waiting: X-CUS signer (12m)                 |
| pause guard: forced transfer allowed (not paused)        | Policy P1 approval-ID ok | P3 supply ok       |
| Register impact: INV-004 -120 / INV-771 +120             | P4 destination ok | P9 initiator != signer    |
|                           [Submit for approval]          |                                               |
+----------------------------------------------------------------------------------------------------------+
| HISTORY   time              action            qty     approval  tx          status                       |
|           2026-09-22 16:02  freezePartial     120     A-7760    0x9a..11    confirmed / reconciled       |
|           2026-09-20 09:15  batchMint         30000   A-7701    0x2b..7c    confirmed / reconciled       |
+----------------------------------------------------------------------------------------------------------+
```

**Annotations.**

- **Header:** standard, chain, supply vs cap, state, and a one-click **PAUSE** (`SGL+R`; the pauser key can only restrict). The owner and agent structure mirrors the vault design in `02` §6.1.
- **[1] New action:** action type; targets (single or CSV batch); quantity; **reason code, legal basis and evidence hash**; timelock (48h default for forced transfer/recovery; waiver needs `CCO` + CEO).
- **[2] Pending maker-checker:** every in-flight approval with the checkers still outstanding. Custody progress is shown so approvers see where the request is stuck.
- **[3] Precheck:** `canTransfer`, identity-registry membership, pause guard (forced transfers are blocked while paused, cf. T-REX audit issue [`VND` §12]) and register impact preview.
- **[4] Custody:** policy checks P1 (approval-ID binding), P3 (supply), P4 (destinations), P9 (initiator ≠ signer). This is the Fireblocks Policy Engine pattern [`WLD` §12].
- **History:** every action shows *confirmed / reconciled*. An action is not complete until M11 reconciliation passes (`03` §0).

---

## W7 — Cap table and 3-way reconciliation (M11)

```text
+----------------------------------------------------------------------------------------------------------+
| Register & Recon > 3-way reconciliation    Run 2026-09-23 02:00 UTC    Tokens 41    Breaks 1   [Re-run]  |
+----------------------------------------------------------------------------------------------------------+
| Token          Chain supply  Golden record  Custodian/omnibus  Registry (DLD/TA)   D qty  Status         |
| MARINA-2BR        1,300,000      1,300,000  1,300,000 (omni)   1,300,000 (cert)       0  OK              |
| SVQ-6                12,000         12,000  --                 --                     0  OK              |
| TOK-SVQ6-B           20,000         19,988  --                 --                    12  BREAK (Sev2)    |
| AUSTIN-SFR14          2,000          2,000  2,000 (TA)         2,000 (TA)             0  OK              |
| [1] Unknown holders (on chain, not in IR): 0      [2] Omnibus: sum(sub-accounts) = on-chain balance ok   |
+----------------------------------------------------------------------------------------------------------+
| BREAK BRK-1182  TOK-SVQ6-B  type: qty mismatch  age 5h  owner REGO-2                                     |
| Chain: 0x3a..e07 holds 132 | Register: INV-004 holds 120 | Diff +12                                      |
| Likely cause: P2P trade T-5510 settled on-chain 01:58 (tx 0x7f..2d); settlement webhook in DLQ (M26)     |
| Proposed fix  (x) Register adjustment +12 to INV-004 (evidence: tx 0x7f..2d, trade T-5510)               |
|               ( ) Corrective forced transfer via M09                                                     |
| Approvals: REGO (maker) -> CCO + CFO (+ X-TA)                     [Propose fix]  [Assign]  [Notes]       |
+----------------------------------------------------------------------------------------------------------+
| CAP TABLE  TOK-SVQ6-B  as of [2026-09-23 v]  holders 341  top-10 22.4%  pools/lending 0%   [Extract]     |
| Holder     Wallet       Balance   %      Frozen  Lock-up      Legal record    Jurisdiction               |
| INV-004    0x3a..e07        120   0.60%  120     --           reg R-9912      ES                         |
| INV-771    0x5e..c11        900   4.50%  0       2026-11-01   reg R-9920      DE                         |
+----------------------------------------------------------------------------------------------------------+
```

**Annotations.**

- **Recon grid:** chain supply vs golden record vs custodian/omnibus vs registry (DLD certificate / TA). Tolerance is 0 for quantities (`04` §7, R2/R3).
- **[1] Unknown holders:** any address holding tokens but absent from the identity registry is **Sev1**, because it means compliance enforcement has failed.
- **[2] Omnibus check:** Σ sub-accounts = on-chain omnibus balance (A3 PRYPCO/Ctrl Alt [`mena/01-prypco` §2.4]).
- **Break panel:** root cause in this example is a missed settlement webhook. Fix options are a register adjustment or a corrective forced transfer, both approved by `CCO` + `CFO` (+`X-TA`) (`02` §3 row 26). This follows the RedSwan and Propvest registrar-reconciliation patterns [`us/02-redswan` §11 #6; `eu/03-propvest` §11 #6].
- **Cap table:** as-of date, concentration, pool/lending share, legal-record reference, lock-ups and frozen amounts. Extracts are signed and archived (M11.7).

---

## W8 — Distribution run review (M12)

```text
+----------------------------------------------------------------------------------------------------------+
| Distributions > D-2026-09-MARINA   monthly   record date 2026-08-31   state: in_review   maker FINA-3    |
+------------------------------------+------------------------------------+--------------------------------+
| [1] CALCULATION              AED   | [2] VARIANCE                       | [3] FUNDING                    |
| Gross rent collected   13,850.00   | vs last period      -1.8%          | Required  11,584.38 AED        |
| - PM fee                 (692.50)  | vs forecast         -4.2%          | Wallet ledger ready  ok        |
| - Maintenance/insurance  (910.00)  | Arrears: 1 unit (0 days)           | Client a/c recon D=0 ok        |
| - Reserve top-up         (500.00)  | Snapshot SNP-8812 locked           |                                |
| - Mgmt fee 0.5%          (100.00)  |   point-in-time, DLD-ref           |                                |
| = Net distributable    11,647.50   |                                    |                                |
+----------------------------------------------------------------------------------------------------------+
| [4] EXCEPTIONS                                                       held 2 | carried 1 | ok 218         |
| INV-003311  frozen (AML)          52.10  -> HOLD in escrow; no investor notice (tipping-off guard)       |
| INV-008870  account closed        11.02  -> HOLD; contact via M21                                        |
| INV-010442  below minimum          0.004 -> carry forward (min AED 0.01)                                 |
+----------------------------------------------------------------------------------------------------------+
| LINES (221)  Holder      Units    Gross    WHT   Net      Asset  Destination       Reinvest  Status      |
|              INV-004821  18,460   165.36   0%    165.36   AED    Mint wallet       no        ok          |
|              INV-002119   9,000    80.62   0%     80.62   PAXG   gold reinvest     yes       ok          |
+----------------------------------------------------------------------------------------------------------+
| File hash 6f2e..a1   Approval: FINA (maker) -> CFO    [Approve]  [Reject - reason]  [Export file]        |
+----------------------------------------------------------------------------------------------------------+
```

**Annotations.**

- **[1] Calculation:** income waterfall from the locked income period (M22 → M12.1). Deduction lines match PRYPCO's enumerated deductions [`mena/01-prypco` §5.3]. The management fee is illustrative.
- **[2] Variance:** checks against the last period and forecast, plus a snapshot lock. Point-in-time vs average-balance method is set per product (RealT average balance including DeFi deposits [`us/04-realt` §5.3]).
- **[3] Funding:** required amount = net minus held lines minus carried dust. The wallet ledger and client-money reconciliation must be green before approval.
- **[4] Exceptions:** AML-frozen holders are held silently (tipping-off guard); closed accounts are held; dust below AED 0.01 is carried forward [`mena/01-prypco` §5.3].
- **Lines:** payout asset and reinvest option per holder (PRYPCO gold reinvest [`mena/01-prypco` §5.3]; Reental auto-reinvest [`eu/01-reental` §5.3]). The file hash binds approval to custody (`02` §6.2 P4).

---

## W9 — Secondary market surveillance (M14)

```text
+----------------------------------------------------------------------------------------------------------+
| Markets > Surveillance     Venue modes: AE listing board (DLD band)  |  AU order book (80% floor)        |
+----------------------------------------------------------------------------------------------------------+
| [1] INSTRUMENT  State   Ref price (date)       Band/floor  Last    D vs NAV  24h vol  Lock-in            |
| MARINA-2BR      OPEN    AED 2.00 (DLD 09-01)   +/-15%      2.21    +10.5%    41k      elapsed            |
| BONDI-3         HALTED  AUD 101.3 (val 06-30)  >= 80%      84.0    -17.1%    9k       meeting            |
| SVQ-6           OPEN    EUR 100 (NAV 08-31)    +/-20%      78.5    -21.5% !  2k       --                 |
+----------------------------------------------------------------------------------------------------------+
| [2] ALERTS                                                                        [Open case (M20)]      |
| S2 Wash trade       SVQ-6    INV-5521 <-> INV-5523, same device, 6 round trips / 48h    score 0.91       |
| S2 Marking close    BONDI-3  3 sells at floor within 10 min of valuation cut-off        score 0.74       |
| S3 Band rejects     MARINA   27 rejects above +15% in 1h (demand exceeds band)          info             |
| S3 Insider window   MARINA   sale-vote notice drafted; 2 staff-linked accounts traded   review           |
+--------------------------------------------+-------------------------------------------------------------+
| [3] BOOK MARINA-2BR (listing board)        | [4] HALTS & CIRCUIT BREAKERS                                |
| Ask 2.30  12,000  INV-7720                 | BONDI-3 halted 14:00 (meeting); resume at close (4E)        |
| Ask 2.28   4,400  INV-1123                 | Auto rule: |move| > 10% in 1h -> halt 30m, MKTO ratify      |
| Bids: n/a (board: buyers take asks)        | [Halt all AE]  [Halt all AU]   (SGL+R, CCO <= 2h)           |
+--------------------------------------------+-------------------------------------------------------------+
```

**Annotations.**

- **Venue modes** are shown per jurisdiction and are licence-bound (M02/M27).
- **[1] Instruments:** reference price and date; band or floor (PRYPCO ±15% vs DLD valuation [`mena/01-prypco` §5.4]; BrickX ≥ 80% of Brick Valuation [`australia/01-brickx` §5.4]); deviation vs NAV (Propvest-style discounts [`eu/03-propvest` §9]); lock-in status.
- **[2] Alerts:** wash trades, marking the close near valuation cut-offs, band-reject spikes and insider windows around sale votes. Cases hand off to M20. Market-abuse surveillance is an obligation, not a vendor feature [`XCC` #31].
- **[3] Book:** a listing board shows seller asks with buyers taking them (PRYPCO [`mena/01-prypco` §5.4]). An order book shows both sides (BrickX).
- **[4] Halts & breakers:** meeting halts (BrickX [`australia/01-brickx` §5.4]); auto-halt rules (not observed in vendors, built by RealX [`APE` M9]); global halt buttons (`SGL+R`, ratify ≤ 2h).

---

## W10 — Property operations and compliance (M22)

```text
+----------------------------------------------------------------------------------------------------------+
| Property Ops > DETROIT-4412 (US-MI)   Occupancy 1/1   Compliance score 41/100 (RED)   PM: Acme PM LLC    |
+----------------------------------------------------------------------------------------------------------+
| [1] MUNICIPAL & REGULATORY                                                                               |
| Item                       Authority         Notice      Due/Hearing  Amount   Status      Owner         |
| Rental registration / CoC  City              2026-06-02  2026-10-01   --       MISSING     PROP-2        |
| Blight ticket BT-99127     City              2026-08-19  2026-09-30   $500     OPEN        PROP-2        |
| Property tax 2025 winter   County Treasurer  --          overdue 44d  $1,912   DELINQUENT  [Pay ->]      |
|   [2] foreclosure countdown: 2027-03-31 (T-189 days)                                                     |
| Water bill                 Utility           2026-09-01  2026-09-25   $214     DUE         X-PM          |
| Tenant safety: no heat     Tenant complaint  2026-09-20  48h SLA      --       SEV2        ASTM-1        |
+------------------------------------+------------------------------------+--------------------------------+
| [3] TITLE & ENCUMBRANCES           | RENT ROLL (AppFolio sync 06:00)    | RESERVES                       |
| Owner of record: SPV-021 LLC ok    | Rent $1,250 | Sep paid: yes        | Operating 2.1 / 3.0 mo (!)     |
| Deed-to-token: 1 deed/2,000 ok     | Arrears $0 | Lease to 2027-05      | Capex $4,100 / $6,000          |
| Liens: tax-lien risk (above)       | PM remittance received 09-04       | Insurance exp. 2026-12-01      |
| Title insurance to 2027-02         |                                    |                                |
+----------------------------------------------------------------------------------------------------------+
| [Record notice] [Pay bill (-> CFO)] [Escalate distressed (-> CCO)] [Replace PM (-> vote)] [Report]       |
| Escalation effect: asset -> distressed; M14 halt review; M15 collateral factor 0; holder disclosure      |
+----------------------------------------------------------------------------------------------------------+
```

**Annotations.**

- **Header:** compliance score (RAG) feeds M03 and investor property reports.
- **[1] Municipal & regulatory:** certificate of compliance / rental registration, blight tickets, taxes, utilities, tenant-safety items. These are the controls whose absence sank RealT: Detroit's suit over 408 properties, unpaid taxes and blight fines, 300+ properties facing tax foreclosure [`us/04-realt` §2.8, §10–11 ⚑].
- **[2] Foreclosure countdown:** tax delinquency is tracked to the statutory foreclosure date. It is a Sev2 alert at 30 days delinquent.
- **[3] Title & encumbrances:** owner of record = SPV; deed-to-token reconciliation; liens; title insurance. RealT faced "never owned" allegations [`us/04-realt` §2.8].
- **Rent roll / reserves:** PM-system sync (AppFolio/Yardi/Buildium [`VND` §9]); remittance timeliness (mogul by the 5th [`us/05-mogul` §11.4]); reserve months vs target (BrickX 3 months [`australia/01-brickx` §2.8]).
- **Escalation:** a distressed flag cascades to an M14 halt review, M15 collateral factor 0 and investor disclosure (M21).

---

## W11 — Approvals inbox and audit log (M25)

```text
+----------------------------------------------------------------------------------------------------------+
| Approvals inbox (CFO-1)     Pending 7   Expiring < 24h: 2   Agents cannot approve   [Filter: Money v]    |
+----------------------------------------------------------------------------------------------------------+
| ID      Action               Object        Maker    Impact         Other approvers    TTL   [ ]          |
| A-7790  Mint batch (212)     OFF-2288      TOKO-1   30,000 units   CCO ok             21h   [ ]          |
| A-7795  Distribution run     D-09-MARINA   FINA-3   AED 11,584     --                 70h   [ ]          |
| A-7781  Recovery (forced)    INV-009       SUPP-4   120 units      CCO ok, TL 48h     5d    [ ]          |
| A-7799  Withdrawal           INV-010233    TRSY-2   USD 61,000     --                 23h   [ ]          |
+----------------------------------------------------------------------------------------------------------+
| [1] DETAIL A-7795   Diff vs last run -1.8% | Simulation ok | File hash 6f2e..a1 | Evidence: 3 files      |
| SoD: you are not the maker ok | Custody: distribution vault V5 auto-signs on file-hash match             |
| [Approve with FIDO key]    [Reject - reason required]    [Comment]                                       |
+----------------------------------------------------------------------------------------------------------+
| [2] AUDIT LOG  filter object=D-09-MARINA                      hash chain VERIFIED to 2026-09-23 06:00    |
| 06:12  FINA-3            distribution.run.calculated    D-09-MARINA  after 6f2e..  prev 1a0c..           |
| 06:15  AGNT-7 (FINA-3)   distribution.exception.held    INV-003311   reason AML_HOLD                     |
| 06:20  FINA-3            approval.requested             A-7795       payload 6f2e..                      |
| 08:02  CFO-1             pii.view.unmasked              INV-008870   reason CONTACT_CLOSED               |
+----------------------------------------------------------------------------------------------------------+
```

**Annotations.**

- **Inbox:** my pending approvals with impact, other approvers and TTL. **Agents cannot approve** [`WLD` §8, §17]. Bulk approval is allowed only for low-risk types (`01` M25.5).
- **[1] Detail:** diff vs previous, simulation result, file hash, evidence, SoD check ("you are not the maker") and the custody route. Approval uses a FIDO hardware key (`02` §0.7).
- **[2] Audit log:** hash-chain verification status; each event shows actor, **on-behalf-of** for agents (Vertalo "delegation on every write" [`WLD` §8]), approval ID and before/after hashes. PII unmasking is itself an audited event.
