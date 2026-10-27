# 06 — Per-Platform Admin Panels (20 platforms), Coverage Matrix and Most Distinctive Admin Features

> **EXTRAPOLATION.** Each condensed panel below comes from the platform file's Sections 3–5 and 11, normalised to the RealX module IDs (M01–M27, see `00-overview.md` §4). Archetypes are defined in `00-overview.md` §6. Role names are the platform §11.3 lists, lightly condensed. Date: 23 Sep 2026.

**How to read.**
- Each platform has: archetype and one-line shape; admin roles; the modules where the platform has a **specific twist** (generic behaviour is in `01`); unique admin features; evidence pointer.
- The coverage matrix (§5) grades every module for every platform.
- §6 ranks the most distinctive admin features across all 20.

---

## 1. United States

### LOF — Lofty · **A1 DeFi-native marketplace** · Algorand ASA per property, Wyoming DAO LLCs
**Shape:** a 24/7 marketplace-ops console. It pays rent every night in USDC from indexer snapshots, runs per-property PMM liquidity pools, and runs Robert's-Rules governance over DAO LLCs.
**Admin roles:** Super-admin (break-glass) · Compliance lead · KYC analyst · Listings analyst · Property ops · Distribution ops · Treasury · Market ops · Governance ops · Tax ops · Support · Auditor (RO) · Eng on-call (signing with approval).

| Module | Lofty-specific twist |
|---|---|
| M02 | DAO LLC registry: Wyoming DAO LLC per property, foreign registrations, registered agent (flag anonymised commercial agents after the St. Louis case), EIN and tax election (C-corp inferred from 1099-DIV) |
| M03 | Seller listing kanban: applied → background check → inspection → appraisal → LLC formed → title recorded → ASA minted → live. Check that the seller retains ≥ 10% |
| M09 | ASA control: mint, freeze/unfreeze, clawback, re-issue to a recovered wallet (clawback/freeze addresses inferred [I]) |
| M10 | 15% per-investor cap per property; California block (DFPI 2022 order) |
| M12 | **Daily rent at ~00:00 UTC** in USD or USDC; per-token daily rate = net rent / days / supply; reinvest option |
| M14 | Order book (2.5% buy / 3% sell) plus market-order surcharge (+2.5%), filled by the PMM; wash-trade and large-order surveillance |
| M15 | **PMM pool console: staked tokens capped at 20% of circulating supply**; imbalance penalty; LP fee parameters; USDC staking without KYC (an edge the rules engine must deliberately ring-fence) |
| M13 | 1099-DIV / 1099-B; 1042-S; 30% default or 15% with W-8BEN treaty rate |
| M17 | Motions: file → second → 48h (24h urgent) discussion → 72h (24h) vote → **90-day cooldown**; execution tracker (sell, change PM) |
| M22 | Municipal notices, squatters, "no heat" tenant complaints; property-tax calendar |

**Unique admin features:** PMM 20% supply cap; nightly payout batch with pool-aware snapshot; Robert's-Rules timers; a USDC treasury alert when the float covers less than the next 7 days of rent.
**Evidence:** `us/01-lofty` §2, §5.3–5.6, §11.

### RSW — RedSwan · **A2 regulated BD/TA back-office** · Hedera HTS / Stellar, Reg D 506(c) + Reg S
**Shape:** a broker-dealer supervision and issuance console for sponsor CRE deals. The registrar is authoritative and the chain is a mirror.
**Admin roles:** CCO · Registered principal · Registered rep · KYC/accreditation analyst · Issuance engineer · Fund administrator · Treasury/FINOP · Sponsor (external, own deals) · Investor relations · Auditor/regulator (RO).

| Module | RedSwan-specific twist |
|---|---|
| M04 | 506(c)/Reg S/506(b)-fund offering set-up; PPM versions; **Form D tracker and blue-sky notices**; tranche closings |
| M07 | **Accreditation verification queue** (third-party letters, 90-day expiry); Reg S attestation; wallet-address registry |
| M09 | **Token Studio:** choose chain; configure **HTS keys** (KYC, freeze, wipe, supply, pause, fee schedule) or Stellar flags (auth-required, revocable, clawback); **Rule 144 lock dates** |
| M11 | Registrar ↔ on-chain reconciliation; break reports; corporate actions |
| M14 | "Secondary readiness": holding-period tracker, resale-eligible list, future ATS connector (no active market) |
| M16 | Wire/hUSDC receipts matched to subscriptions; escrow-release approvals |
| M19 | Form D/blue-sky calendar; FOCUS/net-capital inputs; 17a-4 archives |
| M21 | Marketing publication approved by the registered principal (FINRA 2210) |
| M24 | Sponsor pipeline CRM; fee agreements |
| Regional funds | 3(c)(1)/(7) investor-count limits (alert at 100 beneficial owners) |

**Unique admin features:** HTS key configuration per token; FINRA 2210 comms approval; BD-to-registrar-to-chain issuance chain; alert on transfers to non-KYC-granted accounts (HTS rejection).
**Evidence:** `us/02-redswan` §2–§5, §11.

### ARR — Arrived · **A2 (off-chain benchmark)** · Reg A+ Tier 2 series LLCs, per-series REIT, PPEX ATS
**Shape:** a capital-markets and fund-admin console with no blockchain. It sets the **compliance baseline** a tokenised US retail offer must match.
**Admin roles:** Capital markets · Legal/compliance · KYC ops · Acquisitions · Asset management · Property ops · Valuation · Finance/treasury · Tax · Secondary-market ops · Support · Auditor (RO) · NCPS partner view (RO).

| Module | Arrived-specific twist |
|---|---|
| M02 | Master issuers (Arrived Homes, II…5, STR, funds) spawning series; launching a new master issuer resets the Reg A cap |
| M03 | **Buy-box scoring** (cap rate > 5%, 3+ bed/2+ bath, < 30 years, $200–400k, repairs < 20%); affiliate purchase/renovation tracker with **premium disclosure** and an independent reviewer |
| M04 | Series creation; 1-A/253G2 filing tracker; **$75M / 12-month Tier 2 cap meter per issuer** |
| M07 | **Reg A 10% investment-limit calculator**; state of residence; KYC via NCPS/NCIT |
| M12 | Dividend declaration per series; **REIT 90% distribution test tracker**; payment file to Modern Treasury/NCIT |
| M14 | **Monthly one-week trading windows**; eligible-series list (fully funded, first valuation done, 6-month hold); ATS/TA reconciliation with NCPS |
| M05 | Fund redemption queue (quarterly after 6 months; gating/pro-rata) |
| M19 | 1-K/1-SA/1-U drafting and auditor workflow; **blue-sky state notice-filing engine** (WA DFI consent order lesson) |
| M23 | BBB/FINRA complaint tracker, escalation to NCPS compliance |

**Unique admin features:** blue-sky engine; REIT test tracker; windowed ATS ops; valuation committee for quarterly NAV.
**Evidence:** `us/03-arrived` §1–§5, §10, §11.

### RLT — RealT · **A1 DeFi-native (in liquidation)** · Gnosis/Ethereum ERC-20, YAM, RMM
**Shape:** the most complete DeFi ops console, but with **missing property-substance modules** (⚑). The admin panel of the collapse.
**Admin roles:** Compliance lead · KYC/whitelist analyst · Issuance ops · Treasury/distribution ops · DeFi risk manager (RMM) · Marketplace admin (YAM) · Governance/contract admin (ADMIN/UPGRADER) · Property ops · ⚑ Municipal-compliance officer · ⚑ Title officer · Tax ops · Legal (litigation hold) · Support · Auditor.

| Module | RealT-specific twist |
|---|---|
| M08/M07 | **Per-property wallet whitelist manager**: ≤ 28 requests per user per day, 1–2 day turnaround; AA-wallet Guardian status |
| M10 | **Per-issuer holder counter vs the 2,000 cap** (12(g) management) |
| M09 | Token registry across Gnosis/Ethereum with **bridge supply**; REG contracts (ADMIN/UPGRADER roles) |
| M12 | **Weekly payouts on average balance over the week, including RMM-deposited tokens**; USDC/xDai; Ethereum claim roots; Monday report email |
| M15 | **RMM (Aave v3 fork) risk console**: collateral factors, RealToken price-oracle updates, **whitelisted liquidators**, utilisation, bad debt, pause |
| M14 | YAM admin: token whitelist for P2P offers; offer monitoring |
| M22 ⚑ | **Municipal compliance** (certificates of compliance, code violations, blight tickets, water bills, tax delinquency and foreclosure countdown) was **absent**, and its absence contributed to the collapse |
| M03 ⚑ | **Title & asset verification**: deed recorded to issuer, title insurance, token-to-deed reconciliation (absent; "never owned" allegations) |
| M12/M02 | **Liquidation/wind-down**: asset-sale pipeline; waterfall (fiduciary costs → repairs → back taxes → legal obligations → holders); final distribution and burn |
| M18 | Litigation and legal hold: court orders (rent into escrow), fiduciary instructions, lis pendens |

**Unique admin features:** average-balance snapshots across DeFi positions; RMM liquidation waterfall and liquidator whitelist; an issuer-set oracle (the anti-pattern that needs independent sign-off); a liquidation waterfall console.
**Evidence:** `us/04-realt` §2.3, §2.8, §5, §9–11.

### MOG — mogul · **A4 custodial chain-as-register retail** · Avalanche ERC-1155-style NFT via Fireblocks
**Shape:** a fiat-first LLC administration console. The chain is the contractual register, cash sits in an off-chain wallet ledger, and a community "Clubs" growth engine sits on top.
**Admin roles:** Founder/admin · Underwriting analyst · Asset manager · Finance/treasury · Compliance/KYC · Community manager (clubs) · Investor support · PM (external, own properties) · Chief Mogul (external, own club) · Auditor (RO).

| Module | mogul-specific twist |
|---|---|
| M02 | PropCo LLC formation; **operating agreement stating that "records of ownership are held on the Avalanche blockchain"**; lender information |
| M03 | Underwriting against a 12% hurdle; approval rate < 1% |
| M09 | Mint/burn through the **Fireblocks policy engine** (quorum); Snowtrace links |
| M16 | Plaid ACH ($3.99), card (~3.99%), wire, **PayPal/Venmo/USDT** matching; refund windows (card 45d / ACH 60d); **reserve accounts (8–12 months) with interest accruing to mogul** (a conflict to disclose) |
| M12 | Monthly distribution on the 2nd Tuesday to wallet balances |
| M17 | Proposal builder (PM quotes, two options); super-majority calculator; **default-to-manager tally** (a flagged anti-pattern) |
| M24 | **Clubs admin**: Chief Mogul approvals, 2% reward-pool ledger, payout splits, moderation; broker-compensation review |
| M09/M06 | Transfer-consent requests (Mogul + LLC consent) |
| M13 | K-1 per PropCo |

**Unique admin features:** club reward ledger with broker-compensation risk; default-to-manager tally; reserve-interest attribution.
**Evidence:** `us/05-mogul` §2–§5, §10, §11.

---

## 2. European Union

### REN — Reental · **A1 DeFi-native** · Polygon ERC-20 per property, HumanWallet passkeys, RNT protocol
**Shape:** a retail on-chain issuer console plus a **protocol console** (RNT P2P, RNTlend, staking tiers, DAO), with a **gas-sponsorship treasury**.
**Admin roles:** Super-admin · Compliance officer/MLRO · KYC analyst · Finance/treasury · CFO (approver) · Asset manager · Blockchain ops · Investor support · Partner manager · Read-only auditor (ESI/external).

| Module | Reental-specific twist |
|---|---|
| M01 | Monthly KPI pack (KYC funnel, e.g., 2,300 sign-ups → 700 KYC'd; RNTlend TVL; paymaster balance) |
| M04 | Project builder: SPV, loan-agreement template, token symbol (Reental-XXX-n), supply = price / €100, **100%-funded threshold**, **ESI sign-off** |
| M05/M16 | **Bank-transfer proof-upload reconciliation** ("My Transfers"), with 4-eyes above €10k; USDT path (min 100) vs bank (min 500) |
| M08 | **Paymaster / gas-treasury page** (POL balance, sponsored tx, cost per user, auto top-up); HumanWallet vs external flag |
| M12/M13 | Monthly USDT payouts with **19% Spanish withholding**; auto-reinvest into new project tokens; AEAT informative returns |
| M14 | RNT P2P surveillance (price vs NAV, alert below 80% of NAV); **listing eligibility gated by the promoter's RNT stake** |
| M15 | **RNTlend console** (AAVE fork, **LTV ≤ 75%**, health factors, liquidation queue, oracle/NAV updates under timelock) |
| M12 | Exit manager: 60/40 sale-gain split (min €2,500), investor consultation if the sale is below forecast |
| M24 | Distribution partner Akka (€300 minimum); RNT Studio promoter admin |

**Unique admin features:** gas-sponsorship treasury; utility-token staking gate on securities listing (flagged as a regulatory risk); RNTlend risk console; bank-proof reconciliation.
**Evidence:** `eu/01-reental` §1–§5, §11.

### BSQ — Blocksquare · **A5 two-tier protocol** · Ethereum BSPT (100,000 = 100%), Certified-Partner marketplaces
**Shape:** **two consoles**. (A) The protocol owner runs Certified Partners, the factory, fees, billing and Oceanpoint. (B) Each CP runs its investors, whitelist, offerings and revenue distribution.
**Admin roles:** Protocol owner · Protocol ops · Treasury/DAO liaison · CP admin (tenant owner) · End-user admin (on-chain whitelist role) · KYC/compliance officer · Asset manager · Finance · Support.

| Module | Blocksquare-specific twist |
|---|---|
| M27 (A) | **CP tenant registry**: licence type/instalments; **sBST-stake discount tier**; invoices by the 10th business day; **US$5,990 per tokenisation + US$69/property/month** billing |
| M09 (A) | Property-token factory `createNewPropTokenForCPAndLI` with IPFS resolution hash; registry/data-proxy owner functions; `freezeToken` emergency |
| M16 (A) | FeesWallet: **1.5% in-kind fee split 0.5% CP / 0.5% issuer / 0.5% Oceanpoint**; BST buy-back and burn |
| M15 (A) | Oceanpoint pool parameters, BSPT eligibility, discount/premium, stablecoin list, subject to Snapshot outcome |
| M07/M08 (B) | **Whitelist mode: auto on KYC vs admin confirm**; batch `addWalletList`/`removeWalletList`; ≤ 10 wallets per user; **one KYC across marketplaces** |
| M03 (B) | Owner application intake; corporate resolution upload to IPFS; **notary status** (Luxembourg framework) |
| M12 (B) | Revenue import → holding-period-weighted entitlement → **`addRevenue` batch → push (`pushRevenueToUser`) or claim**; unclaimed tracker |
| M18 (B) | Legal templates (ToS, privacy, AA disclaimer) with tenant placeholders |
| M27 (B) | Domain/path, logo, **restricted jurisdictions**, fee schedule |

**Unique admin features:** the canonical **two-tier protocol/operator console**; 100,000-unit percentage convention; push-or-claim revenue contract; protocol billing linked to staking discounts.
**Evidence:** `eu/02-blocksquare` §1–§5, §11; `WLD` §5.

### PVX — Propvest (Exporo) · **A2 regulated BD back-office** · token-based bonds on Ethereum, prospectus-approved
**Shape:** a MiFID securities-institution console over a **licensed-partner sandwich**: Mangopay cash, Baader depot, Tangany keys. The firm holds no client assets.
**Admin roles:** Management board · Compliance officer/MLRO · Portfolio manager · Fund/asset accountant (maker) · Finance approver (checker) · Asset manager (Sonar, external) · Ops/custody liaison · Customer service · Auditor (RO).

| Module | Propvest-specific twist |
|---|---|
| M06 | Client 360 showing the MiFID profile, Mangopay balance, Baader depot and Tangany address together |
| M05 | **Sparplan** discretionary mandate: 3 model portfolios, monthly pulls from €25, 0.5% p.a. fee accrual; investment-committee approval of model changes |
| M11 | **Token registry mirror: chain supply vs internal ledger vs Tangany balances** |
| M12 | Quarterly distribution run; **zero/reduced distribution approval with investor messaging** |
| M14 | Handelsplatz: seller-priced board, **0.5% seller fee**, < 24h settlement, price vs par, market-abuse alerts, compliance-only trading suspension |
| M12 | Exit waterfall: bank payoff then **80/20 appreciation split** |
| M19 | **MiFID ex-ante/ex-post cost reports**; BaFin/Bundesbank filings; EdW contribution basis |
| M17 | **Bondholder module**: holder lists, SchVG meetings, KapMUG collective-action tracking |
| M23 | Salesforce complaints, 5-business-day SLA, Bundesbank arbitration referrals |

**Unique admin features:** partner-sandwich 3-way reconciliation; discretionary savings-plan engine; bondholder/collective-action module.
**Evidence:** `eu/03-propvest` §1–§5, §8–11.

### BRK — Brickken · **A5 two-tier SaaS + API** · EVM ERC-20 + whitelist (ERC-7943 co-author), Sumsub
**Shape:** a self-serve **Issuer Studio** (tenant) plus an **operator console** with guardrails, API metering and AI-agent governance.
**Admin roles:** Brickken super-admin · Brickken compliance · Support/solutions · Issuer owner · Issuer compliance · Issuer finance · Issuer IR/marketing · API developer.

| Module | Brickken-specific twist |
|---|---|
| M09 (B) | Token wizard (`newTokenization`); mint/burn; issuer transfers; **"Confiscation/edits"**; documentation patch |
| M04/M05 (B) | STO manager (`newSto`: tokens, acceptedCoin USDC/USDT, dates, **soft/hard cap**, min/max ticket; one ongoing offering per token; no early close) → `closeOffer` → success or rollback; investor `claimTokens` gives tokens or a refund |
| M07 (B) | **Sumsub 6-state** KYC (`NOT_REQUIRED/NOT_STARTED/PENDING/APPROVED/REJECTED/ON_HOLD`); KYC-link generator; per-token whitelist tied to email + wallet |
| M12 (B) | Dividend wizard with **allowance/balance preflight** (`dividendDistribution`) |
| M27 (A) | Plans (Studio 1–5 issuances / White-label / API); **compliance guardrails flag `needKyc=false` in prod, offerings above exemption thresholds, restricted jurisdictions** |
| M26 (A) | API keys and metering (**x402** pay-per-call); network registry; **transaction monitor (prepared vs sent vs mined)** |
| M25/M26 (A) | **Agent governance**: registered agents, **mandates (grant/revoke/extend)**, agent enforcement freeze |
| M19 (A) | Metrics hygiene: TTV (asset value) vs on-chain TVL |

**Unique admin features:** tenant guardrails against KYC bypass; agent mandates; prepare/sign/send non-custodial transaction monitor; sandbox environment.
**Evidence:** `eu/04-brickken` §2–§5, §10–11.

### DMB — Domoblock · **A4 perimeter developer-operator** · participative loans, Polygon "fan tokens" as mirror
**Shape:** a developer's project-finance console. Loans sit in an off-chain ledger (the source of truth) with a non-rights token mirror, and the product is **outside the CNMV/ECSP perimeter** by design.
**Admin roles:** Management (CEO/COO) · Real-estate analyst · Project manager/site chief · Legal & accounting · Compliance/KYC · Customer success · Tech/product.

| Module | Domoblock-specific twist |
|---|---|
| M04 | **Pre-sale window (~50% of the project, +0.5% bonus; eligibility ≥ €50k invested or ≥ 5 projects of ≥ €2,500)** then general subscription; all-or-nothing at 100% |
| M07 | DNI + selfie, KYB, **NIE for non-EU**, suitability; **DAI source of funds and a €10,000 per-transaction cap** |
| M09/M11 | **Fan-token registry**: mint on funding close, burn on exit, reconcile to the positions ledger (the token carries no rights) |
| M09 | **Transfer consent**: loan-assignment deed with Domoblock approval (no secondary) |
| M22 | Works/project execution: site progress, contractor invoices, delay forecasts |
| M12 | Exit waterfall: target return, **excess split 40% investors / 40% Domoblock / 20% Guarantee Fund**; **delay interest 0.75%/month after 2 months, capped at 4.5%** |
| M16 | **Guarantee Fund** balance, inflows and draws |
| M19 | **Track-record publisher**: planned vs final duration/ROI/IRR per project |

**Unique admin features:** delay-interest automation; Guarantee Fund; pre-sale tier engine; fan-token mirror (flagged as "blockchain as theatre").
**Evidence:** `eu/05-domoblock` §1–§5, §11.

---

## 3. MENA

### PRY — PRYPCO Mint · **A3 registry-anchored distributor** · DLD title tokens on XRPL (issued by DLD, minted by Ctrl Alt)
**Shape:** a VARA broker-dealer console that is a custodial front end on a sovereign registry. It keeps the DLD registry, the Ctrl Alt omnibus and the Zand client-money bank in lock-step, and enforces the DLD market rules.
**Admin roles:** Super-admin (CTO, no money movement) · Investments/asset manager · Compliance officer/MLRO · KYC analyst · Finance/treasury · Head of Finance (checker) · Property manager · Customer support · Auditor/regulator view · Integration service accounts (Ctrl Alt, DLD, Zand, Paxos).

| Module | PRYPCO-specific twist |
|---|---|
| M03/M04 | Offering builder with **DLD valuation import** and **sqm → token supply calculator (10,000 tokens per m²)**; RERA/Trakheesi permit number; waitlist and allocation rules |
| M05 | **24h cooling-off** timers; oversubscription allocation; refunds; re-confirmation within 5 business days on material change |
| M07 | Emirates ID, **UAE PASS** login, proof of address, **source of funds** before any deposit; ID-expiry suspension |
| M10 | **20% per-investor cap per property**; minimum ticket AED 1,000 (AED 2,000 at the May 2025 launch; whether the AED 1,000 also applies to primary offerings is unverified — `04-synthesis/conflicts-and-gaps.md` #19) |
| M11 | **Registry & tokenisation console**: DLD registration status, **Property Token Ownership Certificates**, TokenID map, XRPL tx hashes, **daily 3-way reconciliation: Zand ↔ wallet ledger ↔ omnibus** |
| M12 | Monthly AED net rent within 5 working days; enumerated deductions; AED 0.01 floor; **rent → gold (PAXG) reinvest** |
| M14 | **Price band ±15% of the latest DLD smart valuation**; **3-month lock-in**; listings visible to approved buyers only; halt switch |
| M17 | Sale vote (50%+1, 14 days); **auto-triggered at 30% ROI**; up to 15% incentive fee above 30% ROI |
| M23 | Complaints: acknowledge ≤ 7 days, resolve ≤ 30 days (max 8 weeks), 8-year retention, DIFC Courts |
| M27/M02 | BCP and **Cessation Plan** run-book; T&C change notice periods (30 days; 14 business days for fees) |

**Unique admin features:** DLD-band validator; ROI-triggered sale vote; sqm-based supply; certificate issuance; gold reinvest.
**Evidence:** `mena/01-prypco-mint` §1–§5, §8–11.

### CTA — Ctrl Alt · **A3 issuer/TA infrastructure (two-tier toward distributors)** · XRPL, Ripple Custody MPC omnibus, ARVA
**Shape:** a VARA Category 1 issuer and digital transfer-agency console serving **distributor tenants** and a **government registry client**.
**Admin roles:** Super-admin (no signing rights) · Issuance officer · Transfer-agency officer · Custody signer (MPC key holder) · Compliance/MLRO · Distributor-relations manager · Regulatory-reporting analyst · Read-only regulator/auditor · Distributor service account (API) · DLD service account.

| Module | Ctrl Alt-specific twist |
|---|---|
| M27/M24 | **Distributor tenant management**: VARA licence numbers, API keys and scopes, webhook health, SLA, volume by distributor ("PRYPCO **and others**") |
| M09 | Issuance console: mint/burn requests, supply calculator, whitepaper link, approval chain, XRPL hashes; **MPC quorum as final control** |
| M11 | **DLD registry sync monitor**; holder register by distributor sub-account; **nightly XRPL ↔ holder register ↔ DLD reconciliation** |
| M10 | Transfer rules per asset (lock-in, ±15% band, 20% cap, residency) with a rule-change log and DLD notification |
| M09/M16 | **ARVA management-token module**: ARVA supply linked to ownership tokens, monthly disclosures, **6-monthly audit pack**, capital ≥ max(AED 1.5M, 2% of reserves) |
| M08 | MPC custody operations: policy, signer quorum, pending signatures, key ceremonies |
| M18/M19 | Whitepaper vault (8-year retention); VARA/FCA/CBI/GFSC returns per entity |

**Unique admin features:** dual-token (ownership + ARVA) transfer engine; registry-sync adapter; multi-regulator entity reporting.
**Evidence:** `mena/02-ctrl-alt` §1–§6, §11.

### STK — Stake · **A4 crowdfunding-SPV (moving to A3)** · DIFC SPVs (DFSA), KSA CMA fund distribution, VARA IPA
**Shape:** a multi-entity, multi-currency brokerage ledger with a **limits engine**, exit windows and heavy automation. A tokenisation sub-stack is being added.
**Admin roles:** Super-admin · Acquisitions · Asset/property manager · SPV company secretary · Finance/treasury (maker) · CFO (checker) · KYC analyst · MLRO/Compliance · KSA fund-distribution officer · Growth/rewards manager · Support · Auditor/regulator (RO).

| Module | Stake-specific twist |
|---|---|
| M10/M07 | **Investment-limit engine**: DFSA retail AED 183,500 per property and AED 367,000 per year; Professional Client up to 33% of a property; **KSA 50/50 qualified/retail tranche monitor**; excluded-nationality list |
| M07 | IDWise passport + selfie; Professional Client evidence; KSA National ID/Iqama |
| M02 | **SPV corporate secretarial**: DIFC register, share certificates, dissolution; RERA 4-owner limit forces the SPV wrapper |
| M14 | **Exit Window manager (May/Nov, 2 weeks)**: valuation upload; **share price = valuation − fixed costs − amortised costs**; order book and matching; 1-year lock-in |
| M12 | Weekly (target) dividends; RentReinvest routing |
| M05 | **AutoInvest** themes charged on schedule; card failures |
| M16 | AED/SAR wallets; Emirates NBD client money; Checkout.com settlements; AED ↔ SAR transfers |
| M13 | KSA withholding tax and **Zakat** handling |
| M24 | Rewards, cashback up to 3%, referrals, **StakePredict** rounds validated by Reidin data |
| M16 | **Explicit KYC/AML fee pass-through** (0.2% + 0.1% p.a.) |

**Unique admin features:** DFSA/CMA limits engine; valuation-priced exit windows; multi-currency wallets; AutoInvest.
**Evidence:** `mena/03-stake` §1–§5, §8, §11.

### KSA — Saudi RER / REGA national tokenisation + sandbox operators · **A3 registry-as-truth** · enhanced ERC-3643 on permissioned EVM (SettleMint)
**Shape:** two consoles. **(A) The operator back office** that RealX would build (Jozo-type). **(B) The registry/regulator console** it must interoperate with.
**Admin roles (A):** Super-admin · Investments · Registry liaison · Finance (maker/checker) · Compliance/MLRO · **Shariah officer** · Property manager · Support · REGA auditor (RO). **(B):** Policy admin · Sandbox supervisor · API admin · Auditor.

| Module | KSA-specific twist |
|---|---|
| M07 | **Nafath** MFA, Yakeen/Absher data, verifiable credential bound to investor; **eligibility tiers: citizen / resident / non-Saudi** |
| M10 | **Eligibility enforced at every transfer**: geo-fencing and zone rules; **Makkah/Madinah limited to Saudi companies and Muslim individuals**; non-Saudi designated zones (law in force 22 Jan 2026) |
| M11 | **RER registration console** (Aamal): fractional deed status per investor, deed-token (1:1) ↔ fractional tokens, rejections; **registry update = legal settlement** |
| M16 | Dedicated acquisition account (mada/SADAD); **escrow release (DvP) only after RER confirmation** |
| M19 | **Weekly REGA sandbox report builder** (profitability, local hiring, complaints) and an audit evidence room |
| M03/M04 | Multi-stage vetting; **REGA price controls** before launch; SAR 1,000 minimum |
| M13 | RETT 5% (plus up to 5% non-Saudi surcharge); Zakat statements |
| M09 | **Exceptions desk**: inheritance, court orders, corrections |
| M03 | Shariah certificates per product (Shariyah Review Bureau); purification |

**Unique admin features:** transfer-time eligibility with religion/zone rules; RER registration console; weekly sandbox reporting; in-Kingdom hosting.
**Evidence:** `mena/04-saudi-rer-national-tokenization` §2, §5–7, §11.

### SCN — SmartCrowd / Nawy Shares · **A4 crowdfunding-SPV (moving to A3)** · DIFC SPVs (1,000,000 shares), Egypt FRA funds, VARA IPA
**Shape:** a DFSA crowdfunding console with **Share Transfer Facility windows**, Hold/Flip products and an Egyptian instalment and advisor-resale arm.
**Admin roles:** Super-admin · Acquisitions · SPV secretary · Finance (maker) · CFO (checker) · Compliance/MLRO · Property manager · Support · Egypt ops · Auditor/regulator (RO).

| Module | SmartCrowd-specific twist |
|---|---|
| M04 | Offering builder by **strategy (Hold / Flip)**; 1,000,000-share allotment; fee preview; pass-through cost budget (DLD 4%, trustee fee, brokerage 2.1%) |
| M02 | "SC-" SPV incorporation, DIFC register, DLD title/trustee booking |
| M14 | **STF manager: windows in March and September (2 weeks); ±15% of current market price; property > 1 year; cost-reimbursement calculation; immediate wallet credit** |
| M12 | Monthly returns; **0.5% p.a. admin fee deducted quarterly**; **Flip profit split 60/40 above a 10% hurdle** |
| M17 | Majority vote to sell (~5 years or earlier for profit) |
| M16 | **Lean Payments pay-by-bank** reconciliation; One-Tap Invest coverage |
| M24/Egypt | Developer contracts, **instalment schedules** and default handling, FRA fund reporting, advisor resale pipeline |

**Unique admin features:** STF ±15% band with cost reimbursement; Flip hurdle calculator; Egyptian instalment engine.
**Evidence:** `mena/05-smartcrowd-nawy-shares` §1–§5, §8, §11.

---

## 4. Australia

### BRX — BrickX · **A4 registered-MIS fund admin** · off-chain unit register (10,000 Bricks per trust)
**Shape:** an RE/fund-administration console with an **order book guarded by a price floor**, meeting-linked halts and monthly smoothed distributions.
**Admin roles:** Super Admin (config only, cannot move money) · Compliance/MLRO · Member Services · Payments Ops · Finance/Fund Accounting · Investment/Property Manager · Market Ops/Surveillance · Company Secretary · Auditor/Custodian (RO) · Tech Support (break-glass).

| Module | BrickX-specific twist |
|---|---|
| M02/M03 | Trust registry per property: **5% top-holder cap check (500 Bricks)**, **LVR ≤ 50%**, **cash reserve ~3 months**, **5-year anniversary countdown** |
| M04/M05 | Pre-order manager: create trust, attach ADD, fixed initial price, **Smart Invest ≤ 50% allocation**, settlement trigger; A$20k advance exposure |
| M14 | **Order book: ask ≥ 80% of the latest Brick Valuation**, price-time priority, **halt/resume (e.g., during unitholder meetings)**, wash-trade/self-match detection |
| M03 | Valuations: recompute **Brick Valuation = (valuation + unamortised costs + cash − liabilities) ÷ 10,000**; publish announcement |
| M12 | Monthly run within 10 business days; **smoothing reserve**; month-end snapshot; Smart Invest auto-reinvest |
| M17 | Meetings: notices, **trading-halt scheduling**, thresholds **50% at the 5-year anniversary / 75% any time** (e.g., Mosman vote failed at 39.77%) |
| M07 | **Equifax ID Matrix** pass/refer/fail; AU residents only |
| M19/M04 | **DDO/TMD** register, significant-dealing log, IDR → AFCA; RG45/RG46 disclosure updates |

**Unique admin features:** 80% price floor with meeting halts; statutory 5-year liquidity vote; distribution smoothing.
**Evidence:** `australia/01-brickx` §1–§5, §9, §11.

### RBL — Redbelly Network · **A6 institutional network** · public-permissioned L1, VC + ZK eligibility, AUDM
**Shape:** (A) a network-operator console (governors, credential issuers, compliance modules) plus (B) the **issuer console** a property tokeniser would build on it.
**Admin roles (B):** Compliance/MLRO · Token operator · Fund accountant · Treasury signers (Fireblocks quorum) · Registry manager · Market ops · Auditor (RO, explorer view) · Super admin (no signing rights).

| Module | Redbelly-specific twist |
|---|---|
| M25/M27 (A) | **Governor and node registry** (33 governors / 40 nodes); onboarding of institutional nodes |
| M07 (A) | **Credential-issuer registry**: approved VC issuers, trust lists, revocation-registry health |
| M10 (A/B) | Compliance-module registry; **accepted credential types** (retail AU resident / wholesale s761G); per-holder caps; jurisdiction blocks |
| M11 (B) | Investor registry mirror: on-chain holders ↔ off-chain unit register; credential expiry |
| M12/M16 (B) | **AUDM** subscriptions and batch payouts; wCBDC pilot monitoring |
| M19 (A) | Network analytics that **reconcile headline claims** (US$73.8bn pipeline vs ~US$1bn issued) |
| M19 (B) | AUSTRAC, Travel Rule logs, DDO, and **TCP/DAP reports (from 2027)** |

**Unique admin features:** ZK credential eligibility instead of per-issuer whitelists; governor registry; credential revocation propagated to holders.
**Evidence:** `australia/02-redbelly-network` §2, §5, §10–11.

### TKL — Tokeniser + Liquidise · **A5 SaaS + A1 liquidity add-ons** · ERC-3643 + ERC-7575 on Redbelly, Averer identity
**Shape:** a fund-registry SaaS for wholesale unit trusts, SPVs and companies with **Protocol Finance** add-ons (AMM liquidity, leverage) and per-investor billing.
**Admin roles:** Tokeniser platform admin · Issuer owner · Issuer operator · Compliance/MLRO · Fund accountant · Administrator (IQ-EQ, API-scoped) · Trustee/RE sign-off · Market/liquidity manager · Read-only auditor · IR editor.

| Module | Tokeniser-specific twist |
|---|---|
| M02 | Fund/company set-up: unit classes, multiple closes, SPVs, co-investments (Enterprise) |
| M07 | **Averer** reusable KYC/KYB credentials; wholesale/sophisticated evidence; FIRB/residency flags as credentials |
| M11 | On-chain ERC-3643 register ↔ **IQ-EQ register** reconciliation |
| M05 | Subscriptions and redemptions: queues, NAV strike, gating, cut-offs |
| M15/M14 | **Protocol Finance**: enable AMM pools, depth, **price-vs-NAV circuit breaker**; leverage (collateral whitelist, LTVs, liquidation monitor). **The pool contract must itself be an eligible holder** |
| M12 | Distributions with **reinvestment mints** |
| M27 | **Per-investor billing** (A$5 → A$1 per investor per month for companies; A$11k/yr + A$2.5k set-up for funds); plan-tier alerts |
| Other | ESOP module: grants, vesting, exercise, leaver rules |

**Unique admin features:** pool-as-eligible-holder pattern; NAV-deviation circuit breaker; per-investor SaaS billing.
**Evidence:** `australia/03-tokeniser-liquidise` §1–§5, §9–11.

### CNV — Canvas Connect · **A6 DFMI pilot** · permissioned EVM L2, CDSR registry, DvP/repo in RBA pilot wCBDC
**Shape:** a wholesale network console for **participants, allowlists, DvP, collateral/repo and cash-leg monitoring**, with regulator-observer access.
**Admin roles:** Network super-admin · Participant admin (per institution) · Issuer operator · Registrar (CDSR) · Settlement/treasury · Risk/collateral · Compliance (MLRO) · Custodian operator · **Regulator observer (RO, selective disclosure)** · Auditor.

| Module | Canvas-specific twist |
|---|---|
| M07/M25 | **Participant & role admin**: institution directory; roles (issuer, investor, custodian, lender, settlement bank, regulator-observer); allowlist per asset |
| M09 | Asset issuance console: **structure type (direct beneficial ownership vs SPV claim)**; supply; restrictions |
| M11 | **CDSR legal register** ↔ on-chain balances; register extracts for auditors |
| M16 | **wCBDC/stablecoin cash leg**: ESA funding requests via Banking Circle; mint/redeem status |
| M14 | DvP trade blotter; failed-settlement queue |
| M15 | **Collateral & repo manager**: pledges, haircuts, margin calls, maturity, collateral release |
| M19 | **Relief/licence-conditions tracker** (Acacia relief, sandbox limits, AFSL application status); regulator data packs |
| M25 | Regulator observer with **selective-disclosure keys** (ZK privacy with auditability) |

**Unique admin features:** repo/collateral console for property-credit tokens; relief-condition tracking; selective-disclosure regulator view.
**Evidence:** `australia/04-canvas-connect` §2–§5, §11.

### AST — Assetora (ex-DomaCom) + Bricklet · **A4 umbrella MIS with sub-funds** · off-chain register, Bricklet DLT co-ownership module
**Shape:** a sub-fund factory under one ARSN, with a **TMD library**, a marketplace that can be frozen, adviser channels and a licensed shared-equity module.
**Admin roles:** Super Admin · RE Compliance/MLRO · Investment Committee member · Asset Manager (property/lending/PE) · Fund Accountant · Marketplace Ops · Member Services · Adviser (external, scoped) · Bricklet Ops · Custodian and auditor (RO) · Company Secretary (ASX and meetings).

| Module | Assetora-specific twist |
|---|---|
| M04 | **ASF campaign manager** by category (Real Estate / Lending / Income / PE); fixed price; bids vs target; **SPDS generator** when target reached |
| M04/M19 | **TMD/DDO library searchable by property address, ASF code or category**; review triggers; significant-dealing log |
| M27/M14 | **Global "stop new business" / inflow freeze and marketplace suspend switch** (used Jul 2024 – Aug 2025); per-ASF halt; tradeable-ASF whitelist (112+ of 130+) |
| M22 | Asset management: **vacancy alerting for specialist housing (NDIS lesson)**; rent roll; lending-ASF loan book (LVR, arrears) |
| M24 | **Accredited-adviser register**; campaigns per adviser |
| M22/M09 | **Bricklet homeowner module**: occupier applications, AMP pre-approval, equity co-investor allocation, **occupancy-fee billing**, **APCOR DLT ledger viewer**, buyout calculator, 50% revenue-share accrual (cap A$100m) |
| M19 | **ASX continuous-disclosure** drafting (listed parent); audit-committee pack; custodian (Perpetual) reporting |

**Unique admin features:** umbrella-MIS sub-fund factory; global freeze switch; shared-equity occupancy billing.
**Evidence:** `australia/05-assetora-domacom-bricklet` §1–§5, §10–11.

---

## 5. Platform × module coverage matrix

**Legend:**
- `●` core: dedicated pages, evidenced by user-facing features or obligations.
- `◐` partial or light.
- `○` absent, or evidence shows it lacking (⚑ where the absence contributed to failure).
- `n/a` not applicable to the model.

Coverage refers to the **extrapolated** owner/operator panel(s). For A5/A6 it includes both the protocol/network console and the tenant/issuer console. Codes are listed in `00-overview.md` §2.

| Module | LOF | RSW | ARR | RLT | MOG | REN | BSQ | PVX | BRK | DMB | PRY | CTA | STK | KSA | SCN | BRX | RBL | TKL | CNV | AST |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| M01 Command Center | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ◐ | ● |
| M02 Issuers/SPVs/Entities | ● | ● | ● | ◐ | ● | ● | ● | ● | ● | ◐ | ◐ | ● | ● | ◐ | ● | ● | ◐ | ● | ◐ | ● |
| M03 Properties & Assets | ● | ◐ | ● | ◐ | ● | ● | ● | ● | ◐ | ● | ● | ● | ● | ● | ● | ● | ◐ | ◐ | ○ | ● |
| M04 Offerings | ◐ | ● | ● | ◐ | ◐ | ● | ● | ● | ● | ● | ● | ◐ | ● | ● | ● | ● | ◐ | ● | ◐ | ● |
| M05 Orders/Alloc./Redemptions | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ◐ | ● | ● | ● | ● | ◐ | ● | ◐ | ● |
| M06 Investors CRM | ● | ● | ● | ● | ● | ● | ◐ | ● | ● | ● | ● | ◐ | ● | ● | ● | ● | ◐ | ● | ◐ | ● |
| M07 Onboarding Review | ◐ | ● | ● | ● | ◐ | ● | ● | ● | ● | ● | ● | ◐ | ● | ● | ● | ● | ● | ● | ● | ● |
| M08 Wallets & Custody | ● | ◐ | n/a | ● | ◐ | ● | ● | ◐ | ● | ◐ | ◐ | ● | ◐ | ◐ | ◐ | n/a | ● | ● | ● | n/a |
| M09 Token Operations | ● | ● | n/a | ● | ◐ | ● | ● | ◐ | ● | ◐ | ◐ | ● | ◐ | ◐ | ◐ | n/a | ● | ● | ● | n/a |
| M10 Compliance Rules | ◐ | ● | ◐ | ◐ | ◐ | ◐ | ◐ | ◐ | ◐ | ○ | ● | ● | ● | ● | ◐ | ◐ | ● | ● | ● | ◐ |
| M11 Register & Recon | ◐ | ● | ● | ◐ | ◐ | ◐ | ◐ | ● | ◐ | ◐ | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● |
| M12 Distributions | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ◐ | ● | ● | ● | ● | ● | ● | ● | ● |
| M13 Tax | ● | ◐ | ● | ◐ | ● | ● | ○ | ◐ | ○ | ● | ◐ | ○ | ● | ◐ | ○ | ● | ◐ | ◐ | ○ | ◐ |
| M14 Secondary Market | ● | ◐ | ● | ● | ○ | ● | ● | ● | ◐ | n/a | ● | ◐ | ● | ◐ | ● | ● | ◐ | ● | ● | ● |
| M15 Lending/DeFi Risk | ● | n/a | n/a | ● | n/a | ● | ◐ | n/a | n/a | n/a | n/a | n/a | n/a | n/a | n/a | n/a | ◐ | ● | ● | n/a |
| M16 Treasury & Client Money | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ◐ | ● | ● | ● | ● | ● | ◐ | ● | ● |
| M17 Governance & Voting | ● | ○ | ○ | ◐ | ● | ◐ | ◐ | ● | ◐ | ○ | ● | n/a | ○ | ○ | ● | ● | ◐ | ○ | n/a | ● |
| M18 Documents & E-sign | ◐ | ● | ● | ◐ | ◐ | ● | ● | ● | ◐ | ● | ● | ● | ● | ◐ | ◐ | ◐ | ◐ | ◐ | ◐ | ● |
| M19 Reporting & Filings | ◐ | ● | ● | ◐ | ○ | ◐ | ◐ | ● | ◐ | ○ | ● | ● | ● | ● | ● | ● | ● | ◐ | ● | ● |
| M20 AML Cases | ◐ | ● | ◐ | ◐ | ◐ | ◐ | ◐ | ● | ◐ | ◐ | ● | ● | ● | ◐ | ◐ | ● | ● | ◐ | ● | ◐ |
| M21 Comms & IR | ● | ● | ◐ | ● | ◐ | ● | ◐ | ● | ◐ | ◐ | ● | ◐ | ● | ◐ | ◐ | ● | ○ | ● | ○ | ● |
| M22 Property Ops | ◐ | ◐ | ● | ○ | ● | ◐ | ○ | ◐ | n/a | ● | ● | n/a | ● | ● | ◐ | ● | n/a | n/a | n/a | ● |
| M23 Support & Complaints | ◐ | ◐ | ● | ◐ | ◐ | ◐ | ◐ | ● | ◐ | ◐ | ● | ◐ | ● | ◐ | ◐ | ● | ○ | ◐ | ◐ | ● |
| M24 Partners/Distributors | ◐ | ● | ◐ | ◐ | ● | ● | ● | ◐ | ● | ◐ | ◐ | ● | ● | ◐ | ◐ | ◐ | ● | ◐ | ◐ | ● |
| M25 Access/Approvals/Audit | ◐ | ● | ● | ◐ | ◐ | ◐ | ● | ● | ● | ◐ | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● |
| M26 Integrations/API | ◐ | ◐ | ◐ | ◐ | ◐ | ◐ | ◐ | ◐ | ● | ◐ | ● | ● | ◐ | ● | ◐ | ◐ | ● | ● | ● | ◐ |
| M27 Settings/Tenancy/WL | ◐ | ◐ | ◐ | ◐ | ◐ | ◐ | ● | ◐ | ● | ◐ | ◐ | ● | ◐ | ◐ | ◐ | ◐ | ◐ | ● | ◐ | ● |
| **Core / partial count** | 14/13 | 17/8 | 17/6 | 11/15 | 11/13 | 17/10 | 15/10 | 18/8 | 14/10 | 11/11 | 20/6 | 14/9 | 21/4 | 14/11 | 14/11 | 19/5 | 13/11 | 17/8 | 13/9 | 20/4 |

**Reading the matrix.**
- **Near-universal core (≥ 17 of 20 platforms `●`):** M01 Command Center (19), M12 Distributions (19), M16 Treasury & Client Money (18), M05 Orders (17), M07 Onboarding (17). These, plus M06 CRM (16) and M25 Access/Audit (15), are the MVP spine.
- **Registry and rules discipline:** M10 Rules **and** M11 Register are both `●` only at RedSwan, PRYPCO, Ctrl Alt, Stake, Saudi RER, Redbelly, Tokeniser and Canvas. These are the regulated-intermediary, registry-anchored and institutional stacks: the A2/A3/A6 discipline RealX should copy (`00-overview.md` §9). The DeFi-native platforms (Lofty, RealT, Reental) enforce rules mostly in the app (`◐`).
- **The most frequent gaps (`○`)** are:
  - **M17 Governance**: 6 platforms (RedSwan, Arrived, Domoblock, Stake, Saudi RER, Tokeniser);
  - **M13 Tax**: 5 (Blocksquare, Brickken, Ctrl Alt, SmartCrowd, Canvas);
  - **M22 Property Ops**: RealT ⚑ and Blocksquare, plus 5 `n/a` infrastructure players.

  Tax and property substance are the systematically under-built modules, and RealT shows the cost.
- **M15 Lending/DeFi** is `●` at only 5 platforms (Lofty, RealT, Reental, Tokeniser, Canvas repo) and `n/a` for 13.
- **M27 Settings/Tenancy** is `●` only for the two-tier players (Blocksquare, Brickken, Ctrl Alt, Tokeniser) and Assetora's kill switches.

---

## 6. Most distinctive admin features across the 20 platforms

Ranked by how much they change admin-panel design (**★ = top 10**). "Admin implication" names the module and control RealX needs if it adopts the feature, or avoids it (anti-pattern ✗).

| # | Platform | Distinctive feature (observed behaviour → extrapolated admin control) | Admin implication for RealX | Evidence |
|---|---|---|---|---|
| ★1 | PRYPCO Mint | **±15% price band around the latest DLD smart valuation**, 3-month lock-in, 20% holder cap, approved-buyer-only listings | M14 band validator bound to the M03 official valuation feed; auto-halt on valuation publication | `mena/01-prypco` §5.4, §9 |
| ★2 | PRYPCO Mint | **Auto-triggered sale vote at 30% ROI** (50%+1, 14 days) plus up to 15% incentive fee above 30% | M17 auto-proposal trigger; M12 incentive-fee waterfall | `mena/01-prypco` §5.5, §8 |
| ★3 | Lofty | **PMM liquidity pools capped at 20% of circulating supply**, plus the nightly (00:00 UTC) payout run | M15 pool-cap parameter; M12 daily scheduler with pool-aware snapshots | `us/01-lofty` §5.3–5.4 |
| ★4 | RealT | **RMM lending (Aave v3 fork) with whitelisted liquidators and an issuer-set oracle**, plus a **liquidation waterfall** (fiduciary → repairs → back taxes → holders) | M15 independent-oracle rule and liquidator allow-list; M12 wind-down waterfall; ✗ issuer-set oracle | `us/04-realt` §5.4, §5.5, §11 |
| ★5 | RealT ⚑ | **Absent municipal-compliance and title-verification modules** (408 properties in Detroit suit; tax foreclosure; "never owned" allegations) | M22 municipal tracker with foreclosure countdown; M03 title verification gate; M11 deed-to-token reconciliation | `us/04-realt` §2.8, §10–11 |
| ★6 | Blocksquare | **Two-tier protocol owner vs Certified-Partner console**: CP registry, sBST discount tiers, 1.5% in-kind fee split 0.5/0.5/0.5, `addRevenue` with push-or-claim, auto vs admin-confirmed whitelisting | M27 tenancy and billing; M16 fee-split ledger; M12 push/claim modes; M07 whitelist-mode setting | `eu/02-blocksquare` §1, §5, §11 |
| ★7 | Stake | **Investment-limit engine** (DFSA AED 183.5k per property / AED 367k per year retail; 33% Professional; KSA 50/50 qualified/retail tranches) plus **valuation-priced exit windows** | M10/M05 limit ledger; M14 window scheduler with a price formula (valuation − fixed − amortised costs) | `mena/03-stake` §2.6, §5.4, §11 |
| ★8 | BrickX | **Order-book floor at 80% of Brick Valuation**, **meeting-linked trading halts**, statutory 5-year exit meeting (50%) vs any-time 75%, distribution smoothing | M14 floor and halt linkage to M17; M12 smoothing reserve | `australia/01-brickx` §5.4–5.5, §11 |
| ★9 | Saudi RER / REGA | **Nafath identity tiers and transfer-time eligibility** (holy-city Saudi/Muslim-only rule; non-Saudi zones), **registry-as-truth filing** (RER/Aamal), **weekly sandbox reports** | M07 sovereign-ID connector; M10 eligibility matrix evaluated per transfer; M11 registry console; M19 weekly report builder | `mena/04-saudi` §2, §5, §11 |
| ★10 | Reental | **Gas-sponsorship (paymaster) treasury** for passkey smart wallets, plus bank proof-of-transfer reconciliation and a utility-token staking gate for P2P listing | M08 paymaster runway; M16 virtual-IBAN matching; ✗ utility-token gate on securities access | `eu/01-reental` §5, §11 |
| 11 | Ctrl Alt | **Dual-token transfer engine** (DLD ownership token + ARVA management token) with nightly XRPL ↔ holder register ↔ DLD reconciliation, and **distributor tenants** | M09 dual-token actions; M11 registry sync; M27 distributor tenancy | `mena/02-ctrl-alt` §2.2, §5.4, §11 |
| 12 | Brickken | **Tenant compliance guardrails** (flag `needKyc=false` in production, offerings above exemption thresholds) and **AI-agent mandates** (grant/revoke, x402 metering) | M27 guardrails; M25/M26 agent principals and mandates | `eu/04-brickken` §2.5, §11 |
| 13 | Arrived | **Blue-sky state-filing engine** (after the WA DFI consent order), **Reg A $75M cap meter**, **REIT 90% test tracker**, monthly ATS windows | M19 blue-sky calendar; M04 cap meter; M12 REIT tracker; M14 windows | `us/03-arrived` §2.3, §11 |
| 14 | RedSwan | **HTS key configuration** (KYC/freeze/wipe/supply/pause/fee keys) and **FINRA 2210 comms approval** by a registered principal | M09 chain-native key console; M21 principal approval | `us/02-redswan` §11 |
| 15 | Propvest | **Licensed-partner sandwich reconciliation** (Mangopay ↔ Baader ↔ Tangany ↔ chain), **zero-distribution approval with messaging**, bondholder/KapMUG module | M11 multi-party reconciliation; M12 zero-run workflow; M17 bondholder meetings | `eu/03-propvest` §6, §11 |
| 16 | Domoblock | **Delay-interest automation** (0.75%/month after 2 months, cap 4.5%), **Guarantee Fund (20% of excess gains)**, pre-sale tier (+0.5%), **€10k DAI per-transaction cap** | M12 delay-interest rule; M16 guarantee-fund ledger; M04 tiers; M20 crypto caps | `eu/05-domoblock` §1, §5, §11 |
| 17 | SmartCrowd | **STF windows (Mar/Sep) with ±15% band and buyer cost reimbursement**; Flip 60/40 above a 10% hurdle; Egyptian instalment engine | M14 window pricing with reimbursement; M12 hurdle splits | `mena/05-smartcrowd` §5, §11 |
| 18 | mogul | **Clubs reward pool (2% of member investment)** and **default-to-manager vote tally** | ✗ both; if clubs are kept, M24 broker-compensation review; M17 disallows default-to-manager | `us/05-mogul` §2.3, §5.6 |
| 19 | Assetora | **Global inflow freeze and marketplace suspend switch** (13-month freeze); **TMD library by property/ASF**; NDIS vacancy alerting | M27 kill switches with board ratification; M04 TMD library; M22 vacancy alerts | `australia/05-assetora` §2.8, §11 |
| 20 | Tokeniser/Liquidise | **Pool contract registered as an eligible ERC-3643 holder**; price-vs-NAV circuit breaker; per-investor SaaS billing | M15 pool eligibility; M14/M15 breaker; M27 billing | `australia/03-tokeniser` §5.4, §11 |
| 21 | Canvas Connect | **Repo/collateral manager** for property-credit SPV tokens, **wCBDC cash-leg monitor**, **regulator observer with selective disclosure**, relief-conditions tracker | M15 repo; M16 cash leg; M25 regulator role; M19 relief tracker | `australia/04-canvas` §5, §11 |
| 22 | Redbelly | **Credential-issuer registry and ZK eligibility** replacing per-issuer whitelists; governor-node registry | M07/M10 credential-type rules; M25 network governance | `australia/02-redbelly` §2.2, §11 |
| 23 | PRYPCO Mint | **sqm-based token supply (10,000 per m²)** and **DLD Property Token Ownership Certificates** | M03 supply calculator; M11 certificate issuance | `mena/01-prypco` §1, §2.2 |
| 24 | Blocksquare / Tokeniser | **Legal anchoring by document hash**: IPFS corporate resolution bound to the property token; ERC-3643 + ERC-7575 vault structures | M18 token-attached documents; M09 token studio templates | `eu/02-blocksquare` §2.1; `australia/03-tokeniser` §2.2 |
| 25 | RealT | **Per-property holder cap at 2,000** (12(g) management) with a whitelist request quota (≤ 28 per user per day) | M10 threshold monitor with pre-trade block; M08 whitelist queue | `us/04-realt` §2.3, §2.5 |
