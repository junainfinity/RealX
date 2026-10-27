# 08 — Dashboards, KPI Catalogue, Report Catalogue, and Alert & Automation Catalogue

> **EXTRAPOLATION.** Consolidated from the alert/report lists in all 20 platform §11.4 sections, vendor dashboards (`APE` M13; `WLD` §3, §12, §17) and obligations (`XCC`). Default thresholds are starting points; production values belong in the jurisdiction packs (M27). Severity scale (`01` §0.5): **Sev1** assets or market integrity (page within 15 min) · **Sev2** regulatory clock or money at risk (1h) · **Sev3** operational (1 business day) · **Sev4** informational. Date: 23 Sep 2026.

---

## 1. Dashboards

| Dashboard | Audience | Refresh | Core widgets | Evidence |
|---|---|---|---|---|
| **Executive** (M01.1) | CEO, CFO, CCO, board | Real-time + daily snapshot | AUM / tokenised value; raised (day/MTD/QTD); pledged vs funded; funded investors; distributions vs schedule; realised vs projected yield; secondary volume and Δ vs NAV; approvals pending; open recon breaks; compliance SLA % | Securitize Control Panel [`WLD` §3]; Reental KPI posts [`eu/01-reental` §11 #1] |
| **Operations** (M01.2) | Heads of ops, team leads | Real-time | Queue tiles (KYC, accreditation, whitelist, bank matches, withdrawals, transfer requests, registry sync, complaints) with oldest age and SLA; stuck tx; pending signatures | PRYPCO ops dashboard [`mena/01-prypco` §11 #1] |
| **Compliance / MLRO** (M01.3, M20) | CCO, MLRO | Real-time | Hits pending; alert/case ageing; SAR clocks; cap meters; holder monitors; outside-TMD dealings; licence-condition expiries; filings due | [`XCC` #20–22, #27, #34] |
| **Finance & treasury** (M01.5, M16) | CFO, treasury | Hourly | Client money Δ (target 0); stablecoin float in payout-days; escrow by offering; withdrawals queue; fee revenue by type/rail; reserves vs target; capital/NTA buffer | Lofty USDC float [`us/01-lofty` §11.4]; Fireblocks pending approvals [`WLD` §12] |
| **Registry & reconciliation** (M11) | Registrar, TA, CFO | Nightly + event-driven | Breaks by type and age; unknown holders; registry-sync backlog; supply per chain; omnibus Σ check | Ctrl Alt nightly 3-way [`mena/02-ctrl-alt` §11.4] |
| **Markets** (M14/M15) | Market ops, risk | Real-time | Instruments by state; Δ vs NAV; band rejects; depth; window imbalance; surveillance alerts; pool imbalance; health factors; oracle freshness | Lofty PMM [`us/01-lofty` §11 #7]; RealT RMM [`us/04-realt` §11 #7] |
| **Property operations** (M22) | Asset/property managers | Daily | Occupancy; arrears ageing; compliance score by property; tax/insurance deadlines; work orders; PM scorecards; reserve months | RealT ⚑ [`us/04-realt` §11 #11]; BrickX property ops [`australia/01-brickx` §11 #10] |
| **Investor relations** (M21) | IR, marketing | Daily | Campaign performance; inbox SLA; announcement engagement; vote turnout; NPS | Propvest Zenloop/Braze [`eu/03-propvest` §7] |
| **Board & audit committee** (M19.6) | Board, audit committee | Monthly / quarterly | KPI pack; risk register; incidents; complaints MI; breaches; capital; reconciliation history; key-person/signer changes | Assetora audit-committee pack [`australia/05-assetora` §11 #11] |
| **Regulator observer** (M19.4) | `X-REG` | On grant | Scoped registers, filings, alerts summary, audit extracts, selective disclosure | Canvas regulator observer [`australia/04-canvas` §11 #10]; [`XCC` #45] |
| **Tenant (A5)** (M27) | Protocol owner, tenant admins | Daily | Tenants by plan; guardrail violations; usage vs capacity; billing; tenant-level KPIs | Blocksquare CP registry [`eu/02-blocksquare` §11 A]; Brickken metrics [`eu/04-brickken` §11 A] |

---

## 2. KPI catalogue

**Metric governance first.** Every KPI has one governed definition (M19.7).
- **Capital raised** is money received for issued tokens.
- **Tokenised asset value** is property valuation.
- **On-chain TVL** is market value of tokens on-chain.

The research shows these being conflated: Brickken TTV ~US$450M vs on-chain TVL ~US$41M [`eu/04-brickken` §10]; Redbelly "US$73.8bn" pipeline vs ~US$1bn [`australia/02-redbelly` §10]; RedSwan "$4B" vs "$100M+" [`us/02-redswan` §10].

### 2.1 Growth and funnel

| KPI | Definition / formula | Owner | Freq. | Default target / threshold | Source | Evidence |
|---|---|---|---|---|---|---|
| Sign-ups | New accounts | Growth | Daily | Trend | M06 | mogul growth dashboard [`us/05-mogul` §11 #1] |
| KYC conversion | KYC-approved ÷ sign-ups (cohort) | KYC lead | Weekly | ≥ 40% (benchmark: Reental ~30%) | M07 | [`eu/01-reental` §2.5] |
| KYC turnaround | Median time submission → decision | KYC lead | Daily | ≤ 24h | M07 | [`eu/02-blocksquare` §2.5] |
| Funded-investor rate | Investors with ≥ 1 settled order ÷ KYC-approved | Growth | Monthly | Trend | M05/M06 | Securitize funnel [`WLD` §3] |
| First-investment ticket | Median first ticket | Growth | Monthly | Trend | M05 | PRYPCO avg ticket AED 10,714 [`mena/01-prypco` header "Scale"] |
| Repeat-investor rate | Investors with ≥ 2 offerings ÷ funded investors | IR | Quarterly | Trend | M05 | — [L] |
| Recurring-plan AUM | Assets in Smart Invest / Sparplan / AutoInvest plans | Product | Monthly | Trend | M05.8 | BrickX, Propvest, Stake |
| Channel mix | % raised by direct / distributor / adviser / club | Partner mgr | Monthly | — | M24 | Reental Akka; mogul Clubs |

### 2.2 Raise (primary)

| KPI | Definition | Owner | Freq. | Threshold | Source | Evidence |
|---|---|---|---|---|---|---|
| Pledged vs funded | Σ commitments vs Σ funds received | CAPM | Real-time | Funded ≥ 90% of pledged at close | M04.4 | Securitize [`WLD` §3] |
| Time-to-fund | Open → 100% funded | CAPM | Per offering | Trend (PRYPCO fastest 1 min 58 s) | M04 | [`mena/01-prypco` header "Scale"] |
| Offering success rate | Finalised ÷ closed offerings | CAPM | Quarterly | ≥ 90% | M04 | Blocksquare finalise/revert [`APE` M3] |
| Cap-meter utilisation | Used ÷ cap (rolling 12 months) per issuer / Member State | CCO | Daily | Alert 80/95%; block 100% | M04.5 | [`XCC` #3] |
| Revocation rate | Revoked in reflection/cooling-off ÷ orders | CCO | Monthly | Trend (spikes signal mis-selling) | M05.3 | [`REG-EU` §3] |
| Refund SLA | Failed/revoked → refunded | FINA | Per event | ≤ 5 business days | M05.6 | PRYPCO refund complaint [`mena/01-prypco` §10] |

### 2.3 AUM, portfolio and valuation

| KPI | Definition | Owner | Freq. | Threshold | Source | Evidence |
|---|---|---|---|---|---|---|
| AUM (NAV basis) | Σ latest published NAV × units | CFO | Daily | — | M03/M11 | — |
| Valuation freshness | % assets with valuation ≤ 6 months old | VALO | Weekly | 100% | M03.4 | BrickX [`australia/01-brickx` §11.4] |
| Valuation Δ | Latest vs purchase and vs prior | VALO | Per valuation | Alert > ±10% [I] | M03 | Stake [`mena/03-stake` §11.4] |
| LVR / gearing | Debt ÷ asset value per SPV | ASTM | Monthly | Alert > 45% (cap 50%) | M03 | [`australia/01-brickx` §2.1] |
| Concentration | Top-city / top-sponsor share of AUM | CCO | Quarterly | Board-set limit | M03 | RealT Detroit concentration [`us/04-realt` §10] |

### 2.4 Yield and distributions

| KPI | Definition | Owner | Freq. | Threshold | Source | Evidence |
|---|---|---|---|---|---|---|
| Distribution success rate | Lines paid ÷ lines approved (first attempt) | FINA | Per run | ≥ 99.5% | M12 | Lofty failed-payout alert [`us/01-lofty` §11.4] |
| On-time distribution | Runs paid within the cadence SLA (e.g., ≤ 5 working days PRYPCO; ≤ 10 business days BrickX) | CFO | Per run | 100% | M12 | [`mena/01-prypco` §5.3]; [`australia/01-brickx` §5.3] |
| Realised vs projected yield | Annualised paid ÷ marketed | CFO | Quarterly | Alert if < 80% of projected | M12/M19 | Lofty 9.2% vs 11% marketed [`us/01-lofty` §10]; Propvest [`eu/03-propvest` §10] |
| Rent collection rate | Collected ÷ scheduled rent | ASTM | Monthly | ≥ 95% | M22 | RealT [`us/04-realt` §11.4] |
| Unclaimed ratio | Unclaimed ÷ distributed (claim model) | FINA | Monthly | < 2% | M12.5 | [`eu/02-blocksquare` §11.4] |
| Zero-distribution count | Runs declared zero or reduced | CFO | Quarterly | Board reporting | M12 | [`eu/03-propvest` §11.2] |
| Reinvest uptake | % payouts auto-reinvested | IR | Monthly | — | M12.6 | Reental, PRYPCO gold |

### 2.5 Liquidity and markets

| KPI | Definition | Owner | Freq. | Threshold | Source | Evidence |
|---|---|---|---|---|---|---|
| Secondary turnover | Traded value ÷ AUM (annualised) | MKTO | Monthly | Trend | M14 | Propvest early turnover [`eu/03-propvest` §9] |
| Premium/discount to NAV | Median (last trade − NAV) ÷ NAV | MKTO | Daily | Alert < −20% | M14 | Propvest 75–80% of par; Reental < 80% alert |
| Spread / depth | Best ask − best bid; depth within ±2% | MKTO | Real-time | Trend | M14/M15 | Lofty pool depth [`us/01-lofty` §11 #1] |
| Sell-order fill rate | Filled ÷ listed sell quantity (per window/month) | MKTO | Monthly | Alert < 50% | M14 | Arrived complaints [`us/03-arrived` §9] |
| Time to sell | Median listing → fill | MKTO | Monthly | Trend (SmartCrowd 1–3 weeks) | M14 | [`mena/05-smartcrowd` §9] |
| Band/floor reject rate | Rejected ÷ submitted orders | MKTO | Daily | Spike alert | M14 | PRYPCO ±15%; BrickX 80% |
| Halt minutes | Minutes halted per instrument | MKTO | Monthly | Trend | M14.7 | BrickX meeting halts |
| Pool utilisation | Staked ÷ cap (e.g., 20% of circulating supply) | RISK | Real-time | Alert ≥ 90% of cap | M15 | [`us/01-lofty` §5.4] |
| Lending health | % borrowers with HF < 1.2; utilisation; bad debt | RISK | Real-time | Bad debt = 0 | M15 | RealT RMM, Reental RNTlend |

### 2.6 Compliance, SLA and AML

| KPI | Definition | Owner | Freq. | Threshold | Source | Evidence |
|---|---|---|---|---|---|---|
| Queue SLA attainment | % items decided within SLA (per queue) | Ops leads | Daily | ≥ 95% | M01.2 | — |
| Screening latency | List update → base fully re-screened | MLRO | Per update | ≤ 24h (wallets ≤ 1h) | M20.1 | [`XCC` #20] |
| Alert-to-case rate | Cases ÷ alerts (scenario precision) | MLRO | Monthly | Tune if < 2% or > 40% [I] | M20 | — |
| SAR/STR timeliness | Filed within clock ÷ filed | MLRO | Monthly | 100% | M20.5 | US 30-day; AU SMR; goAML |
| Travel-rule completeness | Transfers with complete originator/beneficiary data | MLRO | Weekly | 100% above threshold | M20.6 | VARA AED 3,500 [`REG-ME` §4] |
| Accreditation/certificate currency | % investors with valid (unexpired) status | CLSR | Weekly | 100% for gated products | M07.3 | AU 2-year certificates; US 90-day letters |
| Filing timeliness | Filed on time ÷ due | CCO | Monthly | 100% | M19.1 | WA DFI blue-sky order [`us/03-arrived` §2.3] |
| Outside-TMD dealings | Count; % reported within 10 business days | CCO | Monthly | 100% reported | M04.6 | [`REG-AU` §3] |
| Holder-count headroom | Holders ÷ trigger per vehicle | CCO | Daily | Alert ≥ 80% | M10.4 | RealT 2,000 cap |
| Complaint SLA | % acknowledged ≤ 7 days and resolved ≤ 30 days (max 8 weeks) | CMPH | Weekly | 100% | M23 | [`mena/01-prypco` §2.8] |
| Marketing approvals | % live promotions with a valid approval / permit | CCO | Weekly | 100% | M21.4 | Trakheesi; FINRA 2210 |

### 2.7 Registry, reconciliation and custody

| KPI | Definition | Owner | Freq. | Threshold | Source | Evidence |
|---|---|---|---|---|---|---|
| Reconciliation breaks | Open breaks (count, value, age) | REGO | Daily | 0 at T+1 | M11.3 | PRYPCO/Ctrl Alt 3-way |
| Unknown holders | Addresses holding tokens that are not in the identity registry | REGO | Continuous | **0 (Sev1 if > 0)** | M11 | `04` §7 R3 |
| Registry-sync latency | Instruction → registry ack (DLD/RER/TA) | REGO | Daily | Within agreed SLA [L] | M11.4 | [`mena/01-prypco` §11.4] |
| Client-money Δ | Bank − Σ investor ledger | CFO | Daily | **0** | M16.5 | [`XCC` #18] |
| Signature latency | Approval → custody quorum reached | SECO | Daily | p95 ≤ 30 min | M08.4 | Ctrl Alt quorum alert [`mena/02-ctrl-alt` §11.4] |
| Paymaster runway | Gas budget ÷ daily spend | TRSY | Daily | ≥ 7 days | M08.6 | Reental [`eu/01-reental` §11.4] |
| Privileged-action volume | Forced transfers / recoveries / freezes per month | CCO | Monthly | Trend; spike alert | M09 | [`APE` headline 4] |

### 2.8 Property operations

| KPI | Definition | Owner | Freq. | Threshold | Source | Evidence |
|---|---|---|---|---|---|---|
| Occupancy | Occupied ÷ lettable units | ASTM | Monthly | ≥ 95% (Arrived reported 95.9%) | M22 | [`us/03-arrived` §11 #5] |
| Vacancy days | Days vacant per unit | PROP | Weekly | Alert > 60 | M22.9 | RealT; Assetora NDIS |
| Arrears | Arrears > 30 days (value, count) | PROP | Weekly | Alert on any | M22.2 | PRYPCO [`mena/01-prypco` §11.4] |
| **Municipal compliance score** ⚑ | % properties with no open violations, valid registration, taxes current | PROP | Weekly | 100% | M22.5 | RealT ⚑ [`us/04-realt` §11.4] |
| Tax-delinquency exposure ⚑ | Properties with any delinquent property tax | PROP | Weekly | 0 | M22.5 | Detroit foreclosures [`us/04-realt` §2.8] |
| Reserve coverage | Operating reserve months vs target | ASTM | Monthly | ≥ target (e.g., 3 months) | M22.8 | BrickX; mogul 8–12 months |
| PM remittance timeliness | % remitted by the due date (e.g., 5th) | PROP | Monthly | 100% | M22.3 | mogul [`us/05-mogul` §11.4] |
| Work-order SLA | Safety tickets resolved ≤ 48h | PROP | Weekly | 100% | M22.4 | Lofty "no heat" complaint [`us/01-lofty` §10] |

### 2.9 Platform, tenancy and technology

| KPI | Definition | Owner | Freq. | Threshold | Source | Evidence |
|---|---|---|---|---|---|---|
| Tx success rate | Confirmed ÷ broadcast (per chain) | ENGO | Daily | ≥ 99.5% | M26/M01.4 | Brickken tx monitor [`eu/04-brickken` §11 A] |
| Indexer lag | Chain head − indexed block | ENGO | Real-time | < 5 min | M26.6 | — |
| Tenant guardrail violations | Count of open violations | CCO | Daily | 0 | M27 | Brickken [`eu/04-brickken` §11 A] |
| Tenant capacity | Investors ÷ plan cap | PART | Monthly | Alert at 90% | M27.7 | DigiShares/Tokeniser tiers [`WLD` §1; `australia/03-tokeniser` §1] |
| Agent actions share | % maker actions drafted by AI agents; denial rate | SECO | Monthly | Trend | M25.8 | [`WLD` §8, §17] |

---

## 3. Report catalogue

### 3.1 Regulatory reports

| Report | Region | Frequency / deadline | Format | Owner | Source modules | Evidence |
|---|---|---|---|---|---|---|
| Form D | US | ≤ 15 days after first sale (per offering) | EDGAR | LEGL | M04, M19 | [`REG-US` §2] |
| Blue-sky notice filings | US | Per state, per offering (keyed to investor residence) | State portals | LEGL | M04, M06, M19 | Arrived WA DFI order [`us/03-arrived` §2.3] |
| Form C / C-AR | US | Per offering / annual | EDGAR | LEGL | M04, M19 | [`REG-US` §7] |
| 1-A / 253G2 / 1-K / 1-SA / 1-U | US | Qualification / annual / semi-annual / current events | EDGAR | LEGL, CFO | M04, M19 | [`us/03-arrived` §2.3] |
| FOCUS / net capital (if own BD) | US | Monthly / quarterly | FINRA | CFO | M19.3 | [`us/02-redswan` §11.4] |
| Form TA-2 tokenisation disclosures (proposed) | US | Annual (if adopted) | SEC | X-TA | M11, M19 | [`REG-US` §5] |
| SAR | US | ≤ 30 days from detection | FinCEN | MLRO | M20 | [`REG-US` checklist] |
| ECSP annual report to NCA; default-rate disclosure | EU | Annual | NCA format | CCO | M19 | [`REG-EU` §7] |
| Prospectus supplements / KIIS updates | EU | Event-driven | NCA | LEGL | M04, M18 | [`REG-EU` §7] |
| BaFin / Bundesbank returns; EdW contributions | EU (DE) | Periodic | National | CCO | M19 | [`eu/03-propvest` §11 #16] |
| AIFM Annex IV (if AIF) | EU | Periodic | ESMA XML | CCO | M19 | [`REG-EU` checklist] |
| Order-book / trading records (if MTF/DLT MTF) | EU | On request | ESMA JSON | MKTO | M14, M19.4 | [`XCC` #45] |
| VARA periodic returns | UAE | Per VARA schedule | VARA | CCO | M19 | [`mena/01-prypco` §11 #12] |
| ARVA monthly disclosures; 6-monthly audit + attestation | UAE | Monthly / 6-monthly | VARA | CCO, CFO | M16, M19 | [`REG-ME` §2] |
| BWRA | UAE | Annual | Board-approved | MLRO | M20.7 | VARA guidance Jun 2026 [`REG-ME` §4] |
| goAML STR / DNFBP real-estate reports | UAE | Event-driven | goAML XML | MLRO | M20 | [`REG-ME` §4] |
| DFSA crowdfunding returns | UAE (DIFC) | Periodic | DFSA | CCO | M19 | [`mena/03-stake` §11 #12] |
| REGA sandbox weekly report | KSA | **Weekly** | REGA | CCO | M19 | [`mena/04-saudi` §2.3] |
| CMA reports (fund distribution) | KSA | Periodic | CMA | CCO | M19 | [`mena/03-stake` §11 #12] |
| ASIC lodgements; compliance-plan audit | AU | Annual / event | ASIC | CCO, X-RE | M19 | [`REG-AU` checklist] |
| DDO significant-dealings report | AU | ≤ 10 business days | ASIC | CCO | M04.6 | [`REG-AU` §3] |
| AUSTRAC compliance report; SMR; TTR; IFTI | AU | Annual / event | AUSTRAC | MLRO | M20 | [`REG-AU` §4] |
| Continuous disclosure (≥ 100-member schemes); ASX announcements (listed parent) | AU | Event-driven | ASX / ASIC | LEGL | M19.6 | [`australia/05-assetora` §11 #13] |
| Regulator data-room exports | All | Ad hoc / scheduled | JSON / ISO 20022 / CSV | CCO | M19.4 | [`XCC` #45] |

### 3.2 Tax reports

| Report | Region | Deadline | Owner | Evidence |
|---|---|---|---|---|
| 1099-DIV / 1099-B / **1099-DA** (with basis from 2026) | US | Jan–Feb | TAXO | [`REG-US` §8] |
| Schedule K-1 | US | Mar (partnership SPVs) | TAXO | mogul [`us/05-mogul` §2.7] |
| 1042-S / 1042; 8805 (§1446) | US | Mar | TAXO | [`REG-US` §8] |
| FATCA / CRS XML | Global | Annual | TAXO | [`XCC` #37–38] |
| **DAC8** (if RCASP) | EU | First by 30 Sep 2027 | TAXO | [`REG-EU` §8] |
| ES Modelo 193 / 123 withholding certificates | ES | Annual | TAXO | [`eu/01-reental` §2.7] |
| AMMA / annual tax statement | AU | ~3 months after 30 Jun | TAXO | [`australia/01-brickx` §2.7] |
| MIT / TFN withholding remittance | AU | Periodic | TAXO | [`REG-AU` §8] |
| Zakat statements; RETT computations | KSA | Annual / per transfer | TAXO | [`mena/04-saudi` §11.5] |
| Annual distribution report (dividend vs return of capital) | All | Annual | TAXO | RealT [`us/04-realt` §2.7] |

### 3.3 Finance, investor, board and public reports

| Report | Audience | Frequency | Owner | Evidence |
|---|---|---|---|---|
| Per-property P&L and reserve report | Investors, board | Monthly / quarterly | ASTM | Lofty "differentiate" [`us/01-lofty` §11.4] |
| Investor statements and transaction confirmations | Investors | Monthly / annual | IRCM | BrickX [`australia/01-brickx` §2.7] |
| NAV / valuation report | Investors | Per valuation | VALO | Arrived NAV pack [`us/03-arrived` §11.4] |
| MiFID ex-ante / ex-post cost report | EU investors | Annual | CCO | [`eu/03-propvest` §11 #12] |
| Distribution reconciliation | CFO, auditors | Per run | FINA | RedSwan [`us/02-redswan` §11.4] |
| Fee revenue by type and rail | CFO | Monthly | FINA | Lofty [`us/01-lofty` §11.4] |
| Window / secondary liquidity report (orders, fills, discounts) | Board, investors | Per window / monthly | MKTO | Arrived [`us/03-arrived` §11.4] |
| Municipal compliance scorecard ⚑ | Board, investors | Monthly | PROP | RealT [`us/04-realt` §11.4] |
| Deed-to-token reconciliation ⚑ | Board, auditors | Monthly | REGO/LEGL | RealT [`us/04-realt` §11.4] |
| Public KPI / transparency page (governed definitions) | Public | Monthly | IRCM | Reental KPI posts; Domoblock track record [`eu/05-domoblock` §11 #14] |
| Track record: planned vs actual per project | Public | Per exit | IRCM | Domoblock [`eu/05-domoblock` §2.8] |
| Board risk and compliance pack | Board, audit committee | Monthly / quarterly | CCO, CFO | Assetora [`australia/05-assetora` §11 #11] |
| Liquidation / wind-down waterfall statement | Investors, court/fiduciary | Event | LEGL, CFO | RealT [`us/04-realt` §11.4] |
| Tenant usage and billing (A5) | Tenants, finance | Monthly | PART | Blocksquare invoices [`eu/02-blocksquare` §11 A] |

---

## 4. Alert and automation catalogue

**Columns:** trigger → severity → owner → automated action (if any) → human action → module.

### 4.1 Assets, registry and token

| # | Trigger | Sev | Owner | Automated action | Human action | Module |
|---|---|---|---|---|---|---|
| A1 | On-chain supply ≠ register supply | 1 | REGO | Freeze mint queue for that token | Investigate; resolve via M11 break | M11/M09 |
| A2 | **Unknown holder** (holds tokens, not in identity registry) | 1 | CCO | Partial-freeze proposal pre-drafted | Approve freeze; root-cause | M11 |
| A3 | Registry-sync failure / backlog > SLA (DLD/RER/TA) | 2 | REGO | Halt secondary for affected asset | Force re-sync (4E) | M11.4/M14 |
| A4 | Omnibus on-chain ≠ Σ sub-accounts | 1 | REGO | Block withdrawals from omnibus | Reconcile with custodian | M08.5 |
| A5 | Bridge supply drift across chains | 2 | TOKO | Pause bridge | Reconcile | M09 |
| A6 | Mint attempted without allocation-file reference | 2 | CCO | Block (policy P3) | Review | M09 |
| A7 | Owner/agent role change executed | 2 | SECO | Notify all signers | Confirm approval chain | M09.5 |
| A8 | Token paused > 2h without ratification | 1 | CCO | Escalate to CEO | Ratify or unpause | M09 |
| A9 | Forced-transfer / recovery timelock ends in 1h | 3 | LEGL | Notify holder channels again | Final check | M09.4 |
| A10 | Rule-parameter hash on-chain ≠ approved version | 1 | CCO | Pause token | Investigate unauthorised change | M10 |
| A11 | Deed-to-token mismatch ⚑ | 2 | LEGL | Asset → `legal_hold` proposal | Title review | M03/M22.6 |

### 4.2 Investors, onboarding and AML

| # | Trigger | Sev | Owner | Automated action | Human action | Module |
|---|---|---|---|---|---|---|
| B1 | Sanctions list update | 3 | MLRO | Re-screen all investors and wallets | Adjudicate hits | M20.1 |
| B2 | True sanctions match on an existing holder | 1 | MLRO | Freeze wallets (`SGL+R`); hold payouts | Report to authority; no tipping-off | M20/M09 |
| B3 | KYT high-risk inbound deposit | 2 | MLRO | Hold deposit | EDD / return | M16/M20 |
| B4 | KYC queue past SLA (24h) | 3 | KYC lead | Re-assign / overflow | Staffing | M07 |
| B5 | Accreditation / wholesale certificate expiring T-30 / expired | 3 / 2 | CLSR | Reminder; on expiry revoke claim | Review renewals | M07.3 |
| B6 | Emirates ID / passport / Iqama expiry | 3 | KYCA | Reminder; suspend at expiry | — | M07.6 |
| B7 | Failed suitability test (VARA) | 4 | System | Start ≥ 1-week lockout | — | M07.4 |
| B8 | Tenant has KYC disabled in production | 2 | CCO (platform) | Block tenant offerings | Guardrail review | M27 |
| B9 | SAR/STR clock T-5 days | 2 | MLRO | Escalate | File or document no-file | M20.4 |
| B10 | Travel-rule data missing > 3 working days (UAE) | 2 | MLRO | Hold / return transfer | Request data | M20.6 |
| B11 | Self-hosted wallet > €1,000 without ownership proof (EU) | 3 | MLRO | Block use | Request proof | M08 |
| B12 | Holding approaching per-property cap (e.g., 15% / 20% / 5%) | 4 | CCO | Warn investor at order entry | — | M10 |
| B13 | Holder count ≥ 80% of 12(g) / 3(c)(1) / scheme trigger | 2 | CCO | Enable pre-trade block at 95% | Structuring review | M10.4 |
| B14 | Foreign-holding or landholder threshold within 5 points | 2 | CCO | Pre-trade block on crossing | Legal review | M10.4 |
| B15 | Default-to-manager vote rule configured | 2 | CCO | Block publish | Reconfigure | M17 |

### 4.3 Offerings, orders and markets

| # | Trigger | Sev | Owner | Automated action | Human action | Module |
|---|---|---|---|---|---|---|
| C1 | Cap meter ≥ 80% / 95% / 100% | 3 / 2 / 1 | CAPM | Block new commitments at 100% | New tranche / issuer | M04.5 |
| C2 | Soft cap at risk T-3 days | 3 | CAPM | Investor reminder campaign draft | Extend / revert decision | M04.4 |
| C3 | Funds not received within X days | 3 | FINA | Auto-cancel order | — | M05 |
| C4 | Refund unclaimed > N days | 3 | FINA | Reminder | Escheat process | M05.6 |
| C5 | Distribution outside TMD | 2 | CCO | Start 10-business-day clock | File ASIC report | M04.6 |
| C6 | Trakheesi permit / promotion approval expiring | 3 | MKTG | Unpublish at expiry | Renew | M21.4 |
| C7 | Price outside band / below floor (spike) | 3 | MKTO | Reject orders | Review band vs valuation | M14 |
| C8 | Price move > 10% in 1h | 2 | MKTO | Auto-halt 30 min | Ratify / resume | M14.7 |
| C9 | Wash-trade / related-account pattern | 2 | MKTO | Open surveillance case | Investigate; STOR/SAR | M14.6 |
| C10 | Insider window (sale vote or zero distribution pending) | 3 | CCO | Insider list; pre-clearance required | Monitor | M14.6/M17 |
| C11 | Discount to NAV > 20% (instrument) | 3 | MKTO | Flag in IR pack | Liquidity measures | M14 |
| C12 | Window order imbalance > X% | 3 | MKTO | Pro-rata allocation preview | Communicate | M14.4 |
| C13 | Settlement fail | 2 | MKTO | Retry ×3 | Bust or fix | M14.5 |
| C14 | Pool imbalance > threshold / staked ≥ cap | 2 | RISK | Pause deposits to pool | Rebalance | M15 |
| C15 | Oracle stale > 24h or deviation > X% vs last trade | 1 | RISK | Collateral factor 0; halt pool | Revalue | M15 |
| C16 | Health-factor cluster < 1.1; bad debt > 0 | 1 | RISK | Pause borrowing | Liquidation oversight | M15 |

### 4.4 Money and distributions

| # | Trigger | Sev | Owner | Automated action | Human action | Module |
|---|---|---|---|---|---|---|
| D1 | **Client-money Δ ≠ 0** | 1 | CFO | Freeze affected rail's payouts | Reconcile | M16.5 |
| D2 | Unmatched deposit > 48h | 3 | TRSY | Chase payer | Match / return | M16.2 |
| D3 | ACH return / chargeback | 3 | TRSY | Reverse credit; restrict account | Recover | M16 |
| D4 | Withdrawal to non-allow-listed destination | 1 | TRSY | Block (policy P4) | Investigate | M16/M08 |
| D5 | Stablecoin depeg > 0.5% or issuer status change | 1 | CFO | Suspend payouts in that asset | Switch payout asset | M16.6 |
| D6 | Float < 7 days of upcoming payouts | 2 | TRSY | Top-up request drafted | Fund | M16 |
| D7 | Paymaster runway < 3 days | 2 | TRSY | Auto top-up within cap | Approve above cap | M08.6 |
| D8 | Distribution batch failure / partial failure | 2 | FINA | Auto-retry | Manual payout | M12.8 |
| D9 | Snapshot not locked 24h before record date | 2 | FINA | Lock default snapshot | Review | M11.2 |
| D10 | Variance vs last period > 25% | 3 | CFO | Require CEO co-approval | Explain | M12.3 |
| D11 | > N% holders missing tax forms | 3 | TAXO | Apply default rates; send reminders | — | M13 |
| D12 | Reserve below target | 3 | ASTM | Pause discretionary capex | Top-up | M16.7/M22.8 |
| D13 | Sale delayed > 2 months (bullet products) | 3 | ASTM | Start delay-interest accrual | Communicate | M12 |
| D14 | Escrow refund deadline approaching | 2 | FINA | Prepare refund batch | Approve | M16.4 |
| D15 | Capital / NTA buffer < 20% | 2 | CFO | Board notice | Capital plan | M19.3 |

### 4.5 Property operations

| # | Trigger | Sev | Owner | Automated action | Human action | Module |
|---|---|---|---|---|---|---|
| E1 | **Property-tax delinquency > 30 days** ⚑ | 2 | PROP | Payment request to CFO | Pay / dispute | M22.5 |
| E2 | **New code violation / blight ticket** ⚑ | 2 | PROP | Assign; add to scorecard | Remediate | M22.5 |
| E3 | Certificate of compliance / rental registration missing ⚑ | 2 | PROP | Block new offering on asset | Obtain | M22.5 |
| E4 | Tenant-safety complaint (e.g., no heat) | 2 | ASTM | 48h SLA clock | Dispatch | M22.4 |
| E5 | Vacancy > 60 days | 3 | PROP | Flag in distribution forecast | Leasing plan | M22.9 |
| E6 | Rent collected < X% of scheduled | 3 | ASTM | Forecast update | PM review | M22.2 |
| E7 | PM remittance missing by due date | 3 | PROP | Chase PM | Escalate / replace PM | M22.3 |
| E8 | Insurance expiry T-30 | 3 | PROP | Renewal task | — | M22.7 |
| E9 | **Court order received** (e.g., rent into escrow) | 1 | LEGL | Asset → `legal_hold`; pause distributions | Comply; disclose | M22/M12 |
| E10 | Asset → `distressed` | 2 | ASTM | M14 halt review; M15 collateral 0 | Remediation plan | M03 |

### 4.6 Platform, access and integrations

| # | Trigger | Sev | Owner | Automated action | Human action | Module |
|---|---|---|---|---|---|---|
| F1 | **Audit hash-chain verification failure** | 1 | SECO | Lock admin writes (read-only mode) | Forensics | M25.6 |
| F2 | Break-glass session active | 1 | SECO | Notify board chair; record session | Post-incident review | M25.7 |
| F3 | SoD violation detected | 2 | SECO | Suspend conflicting grant | Remediate | M25.3 |
| F4 | Privileged login from new country/device | 2 | SECO | Step-up auth; notify user | Verify | M25.1 |
| F5 | Custody policy drift vs approved baseline | 1 | SECO | Freeze vault | Reconcile policy | M08.4 |
| F6 | Pending signature > 30 min / quorum at risk | 3 | SECO | Page backup signers | — | M08.4 |
| F7 | Stuck tx > 30 min | 3 | ENGO | Re-price / re-broadcast (policy-bound) | — | M26/M01.4 |
| F8 | Indexer lag > 5 min / RPC failover | 2 | ENGO | Switch to fallback RPC | — | M26.6 |
| F9 | KYC / KYT provider degraded | 2 | ENGO | Banner in queues; failover provider | — | M26.1 |
| F10 | Webhook DLQ growth | 3 | ENGO | Retry with backoff | Replay | M26.3 |
| F11 | API key used from new IP / rate abuse | 2 | SECO | Throttle / revoke | Investigate | M26.2 |
| F12 | AI agent exceeds scope / rate | 2 | SECO | Auto-pause agent | Review mandate | M25.8 |
| F13 | Feature flag enabled without licence binding | 1 | CCO | Auto-revert flag | Review | M27.5 |
| F14 | Licence / condition expiry T-90 / -30 / -7 | 3 / 2 / 1 | CCO | Calendar task | Renew | M02.3 |

### 4.7 Scheduled automations (jobs)

| Job | Schedule (default) | Owner | On failure | Module |
|---|---|---|---|---|
| Nightly 3-way reconciliation | 02:00 UTC daily | REGO | Sev2 | M11 |
| KYT re-screen of whitelisted wallets | Nightly + on list update | MLRO | Sev2 | M20 |
| Accreditation / credential expiry sweep | Daily | CLSR | Sev3 | M07 |
| Distribution scheduler | Per product cadence (daily 00:00 UTC / weekly / monthly / quarterly) | FINA | Sev2 | M12 |
| Snapshot capture | Record date − 24h | FINA | Sev2 | M11 |
| PM rent-roll sync | 06:00 daily | PROP | Sev3 | M22 |
| Bank-statement import and auto-match | Hourly | TRSY | Sev3 | M16 |
| Client-money reconciliation | Daily | CFO | Sev1 | M16 |
| Lock-in release | Daily | MKTO | Sev3 | M14 |
| Exit-window open/close | Per calendar | MKTO | Sev2 | M14 |
| Governance timers (discussion/vote/cooldown) | Continuous | LEGL | Sev3 | M17 |
| Filing-calendar reminders | Daily | CCO | Sev2 | M19 |
| Weekly REGA sandbox report | Weekly | CCO | Sev2 | M19 |
| Paymaster auto top-up | Continuous within cap | TRSY | Sev2 | M08 |
| Access recertification campaign | Quarterly | SECO | Sev3 | M25 |
| Audit hash-chain verification (+ optional anchor) | Hourly / daily | SECO | Sev1 | M25 |
| Break-glass / pause drill on testnet | Quarterly | SECO, CCO | Sev3 | M09.8 |
