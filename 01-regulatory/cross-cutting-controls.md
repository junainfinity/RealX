# Cross-Cutting Compliance Controls → Admin-Panel Modules

> **Research, not legal advice.** A synthesis of `us.md`, `eu.md`, `mena.md` and `australia.md` (snapshot 23 Sep 2026). It maps regulatory obligations to the platform controls and admin-panel features RealX would need.

**Evidence labels.** Evidence is inherited from the region files. `US-C#8` means us.md source 8 (confirmed); `EU-`, `ME-` and `AU-` prefixes work the same way. [I] = inferred mapping (the obligation is sourced; the admin feature design is RealX's inference). [S] = speculative.

**Design principle.** Every jurisdiction ends up needing the same ~15 control families. They differ mainly in **parameters**: thresholds, test types, retention periods and report formats [I]. So build one **jurisdiction-parameterised rules engine** plus shared queues. Avoid building four separate compliance stacks.

---

## A. Master table: obligation → control → admin feature

### A1. Entity, licence and product set-up

| # | Obligation (region: legal basis) | Required platform control | Admin-panel feature / module | Evidence |
|---|---|---|---|---|
| 1 | Operate only within licensed activities: US BD/ATS/TA/portal; EU ECSP/MiFID; UAE VARA broker-dealer/Cat 1 issuance; AU AFSL/RE/market licence | Activity-permission matrix per legal entity and jurisdiction; feature flags that hard-block unlicensed activities (e.g., order matching) | **Licence & Permissions Registry**: licences, conditions, expiry, authorised reps / CARs / tied agents; feature-flag console | US-C#23, EU-C#10, ME-C#6/#7, AU-C#1/#6 |
| 2 | Choose and document the offering exemption or regime per asset: US Reg D/S/A/CF; EU prospectus exemption / ECSP / DLT regime; UAE ARVA Cat 1; AU registered vs wholesale MIS | Offering template bound to an exemption, with its caps, investor-type rules and resale rules | **Offering Set-up Wizard**: exemption selector that auto-loads constraints; legal-opinion upload; classification record (security / MIS / ARVA / TS) | US-C#1/#24, EU-C#3, ME-C#5, AU-C#1 |
| 3 | Raise caps: Reg CF $5M rolling 12 mo; Reg A $20M/$75M; EU prospectus €12M (MS option €5M); ECSP €5M per project owner; AU CSF $5M | Rolling-window cap meters per issuer (and per Member State for EU) that block over-subscription | **Raise-Cap Meter** on each offering and issuer; alerts at 80/95/100% | US-C#25/#26/#27, EU-C#9/#10 |
| 4 | Product governance and target market: AU DDO TMD; EU MiFID TM; VARA product gating | Target-market fields per product; distribution conditions; outside-TM detection | **Product Governance**: TMD builder, distribution conditions, outside-TMD flag queue (AU 10-business-day report clock) | AU [I], EU [I], ME-C#3 |
| 5 | Issuer/ARVA capital: VARA ARVA capital = higher of AED 1.5M or 2% of reserves; AU NTA ($10M custody); US BD net capital | Capital computation from live reserve and valuation data | **Prudential Dashboard**: capital/NTA calculators, breach alerts, board-report export | ME-C#5, AU-C#1, US [I] |

### A2. Investor onboarding and classification

| # | Obligation | Required platform control | Admin-panel feature / module | Evidence |
|---|---|---|---|---|
| 6 | KYC / CIP / CDD (all regions: US BSA/CIP; EU AMLD→AMLR 2027; UAE FDL 10/2025; AU AML/CTF Act as amended 2024) | Identity verification, document and liveness checks, address, DOB; risk scoring; periodic refresh | **KYC Review Queue**: provider results, manual review, risk rating, refresh scheduler, re-KYC triggers | US [I], EU-C#11, ME-C#12, AU-C#3 |
| 7 | KYB and UBO: US CDD 25% + control prong (CDD rule unchanged although domestic BOI filing ended Aug 2026); EU UBO ≥25%; UAE beneficial-ownership transparency | Entity onboarding, ownership tree capture, UBO ≥25% + control persons | **KYB & UBO Graph**: ownership tree editor, UBO verification tasks, register cross-checks | US-C#22, EU [I], ME-C#12 |
| 8 | PEP and adverse media | Screening at onboarding and ongoing | **Screening Hub** (shared with sanctions; see #20) | [I] |
| 9 | Accredited-investor verification, US 506(c): reasonable steps, or $200k/$1M minimum-investment path with reps | Evidence upload, third-party letters, min-ticket path with signed reps and "no third-party financing" attestation; expiry | **Accreditation Review Queue**: doc upload, CPA/attorney/RIA/BD letter intake, min-investment reps flow, 90-day/annual expiry tracking, reviewer sign-off | US-C#8/#24 |
| 10 | Non-accredited limits: US Reg CF 12-month cross-offering limits (calendar-year income); Reg A Tier 2 10% cap | Self-certified income/NW; cumulative 12-month investment ledger across offerings | **Investment Limit Tracker** per investor; hard stop at limit | US-C#25/#27 |
| 11 | EU client categorisation (retail / professional / ECP) + ECSPR sophisticated vs non-sophisticated | Categorisation workflow with opt-up evidence (2 of 3 tests) | **Client Categorisation** module; opt-up evidence; annual review | EU-C#10, EU [I] |
| 12 | Appropriateness / knowledge tests: ECSPR entry knowledge test + loss simulation; MiFID appropriateness; VARA suitability with 1-week cooling-off after fail | Test engine with versioned questionnaires, scores, retest timers | **Knowledge & Suitability Tests**: question bank, scoring, retest lockout (VARA ≥1 week), results history | EU-C#10, ME-C#3 |
| 13 | VARA investor classification: Qualified = AED 3.5M net assets or AED 700k income, plus knowledge; SoW/SoF; 8-yr records | Evidence capture and verification of wealth and income | **Investor Classification Queue (UAE)**: SoW/SoF upload, AED thresholds, periodic review, product-access mapping | ME-C#3 |
| 14 | AU wholesale / sophisticated tests: $500k ticket; $2.5M net assets / $250k income accountant's certificate (2-yr validity) | Certificate intake with expiry; ticket-size check | **Wholesale Certificate Manager** with 2-year expiry | AU [I] |
| 15 | Nationality / residency eligibility: KSA non-Saudi ownership rules (holy cities: Saudi companies and Muslim individuals only); DLD EID-holder access; FIRB established-dwelling ban to 30 Jun 2029; US state foreign-adversary laws; EU non-EU buyer rules | Investor nationality/residency attributes vs asset-location rules | **Eligibility Rules Engine** (nationality × residency × asset country/zone × property type) with override audit | ME-C#8/#18, AU-C#9, US [I], EU [I] |

### A3. Orders, payments, escrow, client money

| # | Obligation | Required platform control | Admin-panel feature / module | Evidence |
|---|---|---|---|---|
| 16 | Warnings and consent: ECSPR explicit consent above €1,000 / 5% NW; 4-day reflection period | Ticket vs declared-NW check; reflection-period order state with revoke | **Order Workflow** with states: reflection → funds → allocation; revoke button; consent log | EU-C#10 |
| 17 | Cooling-off: AU CSF 5 business days [I]; AU PDS cooling-off for retail MIS (14 days) [I] | Timers per regime | Same Order Workflow (regime-specific timers) | AU [I] |
| 18 | Escrow / client money: US Reg CF qualified third-party escrow & Rule 15c2-4; EU PSD2 / qualified custodian; AU trust account; UAE segregated client accounts / escrow law | Segregated accounts; min-target logic; automatic refunds | **Escrow & Client Money Ledger**: per-offering escrow status, min/max logic, refund automation, daily bank reconciliation | US [I], EU-C#10, AU [I], ME [I] |
| 19 | Accepted settlement assets: US GENIUS Act (permitted payment stablecoins, ~Jan 2027); UAE CBUAE PTSR (licensed AED payment tokens; VARA NoC circular); EU MiCA EMT; AU Instrument 2025/867 | Allow-list of stablecoins/payment tokens per jurisdiction | **Payment Asset Allow-list**: issuer status monitoring, per-jurisdiction toggles | US-C#6, ME-C#1, EU-C#2, AU-C#2 |

### A4. AML monitoring, sanctions, travel rule

| # | Obligation | Required platform control | Admin-panel feature / module | Evidence |
|---|---|---|---|---|
| 20 | Sanctions screening (OFAC; EU restrictive measures; UAE EOCN/UN TFS; DFAT) of persons **and wallets** | Real-time and batch screening, list-update re-screening, wallet blocklists | **Screening Hub**: name + wallet screening, hit adjudication, freeze/unfreeze with dual control, EOCN/OFAC reporting templates | US-C#7, ME-C#1, EU [I], AU [I] |
| 21 | Travel rule: US ≥$3,000 (31 CFR 1010.410(f)); EU TFR no de minimis + self-hosted >€1,000 ownership proof; UAE ≥AED 3,500 beneficiary verification + unhosted-wallet EDD; AU reformed VA travel rule | Originator/beneficiary data exchange; wallet-ownership proofs; aggregation windows | **Travel Rule Console**: VASP counterparty directory, message status, missing-data holds, self-hosted wallet proof (signature / Satoshi test), return/decline flow | US [I], EU-C#13, ME-C#11, AU [I] |
| 22 | Transaction monitoring and suspicious reporting (SAR / STR / SMR; goAML in UAE; AUSTRAC SMR) | Rules plus scenario engine; case management; filing clocks | **AML Case Management**: alerts → cases → filing, deadline clocks (e.g., US 30-day SAR), tipping-off controls | [I] |
| 23 | Business-wide risk assessment (VARA BWRA guidance Jun 2026; UAE FDL 10/2025 gap assessment; AMLR 2027) | Annual BWRA with evidence | **BWRA Module**: inherent/residual risk matrix, board approval | ME-C#1/#12 |
| 24 | Real-estate-specific reporting: US RRE rule (vacated 19 Mar 2026; watch GTOs); AU tranche 2 real-estate designated services (1 Jul 2026); UAE DNFBP real-estate reports | Flag property acquisitions and disposals by SPVs; capture buyer/seller KYC | **Property Transaction AML** tab on each asset | US-C#15, AU-C#3, ME [I] |

### A5. Token, register, custody

| # | Obligation | Required platform control | Admin-panel feature / module | Evidence |
|---|---|---|---|---|
| 25 | Authoritative holder record: US TA master securityholder file (TA proposal Sep 2026: DLT allowed, TA exclusive control; names/addresses still required); EU DLT registers (Ley 6/2023; eWpG); AU register of members s169; DLD title sync | Off-chain golden record reconciled to chain; TA/RE override powers | **Registry / Cap Table**: golden record, daily on-chain reconciliation, break alerts, forced transfer / freeze / re-issue with maker-checker, 6-yr retention (US TA) | US-C#11, EU-C#15, AU [I], ME-C#8 |
| 26 | Transfer restrictions: Rule 144/Reg S/Reg CF 1-yr; ECSP bulletin-board-only; VARA product gating; AU wholesale-only schemes; DLD controlled market | On-chain compliance rules (allow-list, lock-up dates, jurisdiction flags, max holders) | **Token Compliance Rules Engine**: per-token rule sets, lock-up calendar, allow-list management, simulation before rule change | US-C#25, EU-C#10, ME-C#9, AU [I] |
| 27 | Holder-count and concentration thresholds: US §12(g) 2,000 / 500 non-accredited; EU RETT share-deal (e.g., DE 90%); AU landholder duty and foreign-surcharge thresholds; FIRB foreign caps | Real-time holder and concentration analytics that block threshold-crossing transfers | **Threshold Monitor** per SPV/trust: holder counts, % by holder/associate group, foreign %, pre-trade block | US [I], EU [I], AU [I] |
| 28 | Custody: US BD 15c3-3 five conditions (Dec 2025); ECSPR Art 10 qualified custodian; VARA licensed custodian / ARVA reserve segregation; AU RG 133 + omnibus relief (2025/871) | Custodian integration; key-management policy; DLT assessment; omnibus reconciliation | **Custody Console**: wallet inventory (hot/warm/cold), key ceremonies log, DLT/chain risk assessments, custodian reconciliation, BCP playbooks (forks, 51% attack) | US-C#19, EU-C#10, ME-C#5, AU-C#1/#8 |
| 29 | ARVA reserve and redemption: VARA Annex 2 (segregation, no rehypothecation, 6-monthly audit, fee-free redemption) | Reserve ledger tied to property valuations; redemption SLA | **Reserve & Redemption** module: valuation schedule, auditor portal, redemption queue | ME-C#5 |

### A6. Secondary trading

| # | Obligation | Required platform control | Admin-panel feature / module | Evidence |
|---|---|---|---|---|
| 30 | Multilateral matching only with a licence: US ATS (BD); EU MTF/OTF or DLT MTF (ECSP = bulletin board only); UAE VARA exchange/broker-dealer + DLD controlled market; AU market licence; QFC token exchange TSP | Venue-mode switch per jurisdiction: bulletin board / RFQ / partner ATS / own venue | **Marketplace Control**: venue mode, eligible-instrument list, eligible-investor filters, halts, circuit breakers, partner-venue integration status | US [I], EU-C#10, ME-C#9, AU [I], ME-C#22 |
| 31 | Market surveillance and fair access (US Reg ATS; EU MAR; VARA market conduct) | Surveillance alerts (wash trades, spoofing, insider lists) | **Trade Surveillance** queue; insider list; STOR/SAR hand-off | [I] |
| 32 | Post-trade: TA/registry update, confirmations, best-execution records | Atomic settlement → registry update; trade confirmations | **Post-Trade Ops**: settlement status, fails, confirmations archive | [I] |

### A7. Disclosure, reporting, marketing

| # | Obligation | Required platform control | Admin-panel feature / module | Evidence |
|---|---|---|---|---|
| 33 | Offering documents: US PPM / Form C / 1-A; EU prospectus / KIIS / KID; UAE whitepaper (Sched. 1) + ARVA disclosures; AU PDS / TMD | Versioned document library; delivery and acknowledgement logs | **Disclosure Library**: templates per regime, versioning, regulator submission tracking, investor acknowledgement log | US-C#24, EU-C#10, ME-C#5, AU [I] |
| 34 | Regulatory filings calendar: Form D (15 days), C-AR, 1-K/1-SA/1-U; FINRA; ECSP annual NCA report; VARA returns; AU compliance-plan audit, ASIC lodgements | Task calendar with owners and evidence | **Regulatory Calendar & Filings Tracker** | US-C#24, EU [I], AU [I] |
| 35 | Ongoing investor reporting: distributions, NAV/valuations, property ops; ARVA 6-monthly audits; AU periodic statements | Scheduled report generation | **Investor Reporting** module (statements, distribution notices, valuation reports) | ME-C#5, AU [I] |
| 36 | Marketing rules: US 506(b) ban / 506(c) allowed / Reg CF tombstone / FINRA 2210 / no misleading "backed by RE" claims (Texas SSB 2026); EU MiFID fair-clear-not-misleading + ECSPR KIIS consistency; VARA Marketing Regulations + RERA Trakheesi permits; AU RG 234 (Jun 2026) | Pre-approval of all promotions; jurisdiction gating of pages; claims library; finfluencer register | **Marketing Compliance**: submission → review → approval, permit numbers (Trakheesi), disclaimer injection by geo-IP/residency, archive | US-C#18, EU [I], ME-C#24, AU-C#11 |

### A8. Tax

| # | Obligation | Required platform control | Admin-panel feature / module | Evidence |
|---|---|---|---|---|
| 37 | Self-certification and TIN collection (FATCA W-9/W-8; CRS; DAC8/CARF) | Tax-form capture with expiry and validation | **Tax Profile** per investor: forms, TINs, residency, expiry reminders | US [I], EU-C#12, ME-C#1 |
| 38 | Broker reporting: US 1099-DA (tokenized securities, unless limited-access regulated network → 1099-B); basis from 2026 | Cost-basis lot tracking; network-status flag | **Tax Reporting Engine**: 1099-DA/1099-B/1042-S, K-1 (LLC), AMMA (AU AMIT), CRS/DAC8 XML | US-C#20/#21, EU-C#12, AU [I] |
| 39 | Withholding: US FIRPTA/§1446(f)/§1441; AU MIT withholding 15/30%, TFN 47%; EU national WHT with treaty relief | Withholding calculator by investor residency and payment type | **Withholding Engine**, integrated with distributions | US [I], AU [I], EU [I] |
| 40 | Transfer and real-estate taxes: DLD 4% fee; KSA RETT 5%; EU RETT share-deal; AU landholder / foreign surcharge duty | Transaction-tax calculator; threshold monitor (see #27) | **Transaction Tax & Fees** module | ME [I], EU [I], AU [I] |

### A9. Property, title, Shariah, records

| # | Obligation | Required platform control | Admin-panel feature / module | Evidence |
|---|---|---|---|---|
| 41 | Title-registry linkage: DLD tokenised title sync; elsewhere the SPV/RE holds title | Registry-sync monitor or SPV title-document vault | **Asset Registry**: title documents, registry IDs, sync status (DLD), encumbrances, insurance | ME-C#8, US/EU/AU [I] |
| 42 | Property operations compliance (taxes, code violations, reserves). Lesson from RealT/Detroit | Property-ops SLAs and reserves | **Property Operations** tracker: tax bills, violations, maintenance, reserve balances, manager KPIs | US-C#16/#17 |
| 43 | Shariah compliance for Islamic offerings (AAOIFI; Shariah board) | Screening of tenants and leverage; fatwa storage; purification | **Shariah Compliance** tab per asset | ME [I] |
| 44 | Record-keeping: US TA 6 yrs; EU AML 5 yrs; VARA classification 8 yrs; AU 7 yrs [I] | Immutable audit trail; retention policies per record type and jurisdiction | **Audit Log & Retention Manager**; regulator read-only access portal | US-C#11, ME-C#3, EU/AU [I] |
| 45 | Regulator access and data export (TA proposal: independent regulator access; ESMA JSON order-book records; VARA reporting) | Export in required formats | **Regulator Data Room**: scheduled and ad-hoc exports (JSON / ISO 20022, CSV), access logs | US-C#11, EU-C#16 |

---

## B. Region-by-region comparison matrix

| Dimension | United States | EU / EEA | MENA — UAE (Dubai onshore) | MENA — Saudi Arabia | Australia |
|---|---|---|---|---|---|
| **Classification of a property-fraction token** | Security (LLC/REIT interest or investment contract; Howey). "Format doesn't matter" (SEC staff, 28 Jan 2026) [US-C#1] | Transferable security or AIF unit under MiFID II → **outside MiCA** (ESMA GL, 17 Dec 2024) [EU-C#2/#3] | **ARVA** (VARA) + DLD tokenised title-deed fraction [ME-C#5/#8] | Security / fund unit under CMA [I] | **MIS interest** (INFO 225 Example 9) or share [AU-C#1] |
| **Main pathway for a new entrant** | Reg D 506(c) + Reg S SPVs via partner BD + registered TA; later Reg A Tier 2 / Reg CF; partner ATS [US-C#24–26] | ECSP licence (≤€5M per project owner) or MiFID firm / tied agent; prospectus above €12M; DLT Pilot venues for trading [EU-C#9/#10/#5] | VARA broker-dealer as DLD-approved distributor (PRYPCO model), or own Cat 1 issuance + BD (Ctrl Alt / Tokinvest) [ME-C#7] | CMA FinTech ExPermit (real-estate fund distribution platform) + CMA-licensed fund manager [ME-C#20] | Partner RE ("RE-for-hire") or own AFSL + RE; wholesale unregistered MIS to start [AU-C#1/#6] |
| **Investor tests / limits** | Accredited: $200k/$300k income or $1M NW; 506(c) verification or $200k/$1M min-ticket NAL; Reg CF and Reg A T2 non-accredited caps [US-C#8/#25] | Retail / professional / ECP; ECSP non-sophisticated protections (knowledge test, €1,000 / 5% warning, 4-day reflection) [EU-C#10] | Retail / Qualified (AED 3.5M NA or AED 700k income) / Institutional; suitability test [ME-C#3]. DLD: EID holders, AED 2,000 min at launch (AED 1,000 shown on prypco.com/mint in Sep 2026; see `04-synthesis/conflicts-and-gaps.md` #19) [ME-C#8] | Retail vs qualified clients [I] | Retail vs wholesale ($500k; $2.5M NA / $250k income); DDO TMD [I] |
| **Secondary trading** | Restricted securities (Rule 144 / Reg CF 1-yr); ATS (BD) needed. Innovation exemption covers NMS stocks only [US-C#5] | MTF/OTF licence; ECSP bulletin board only; DLT Pilot → proposed €100bn, permanent [EU-C#7/#10] | DLD Phase Two controlled secondary market (20 Feb 2026) [ME-C#9]; VARA exchange licence for multilateral venue [I] | Fund redemptions; no token secondary market [I] | Market licence (s791A) or partner; CS facility [I] |
| **Custody** | TA golden record; BD 15c3-3 possession conditions; qualified custodian for RIAs [US-C#11/#19] | Qualified custodian (ECSPR Art 10); MiFID segregation; direct holding on national DLT registers [EU-C#10/#15] | Licensed custodian; ARVA reserves segregated; Ripple Custody on XRPL in DLD project [ME-C#5/#9] | CMA custodian for funds [I] | RE or RG 133 custodian; $10M NTA; omnibus relief (2025/871) [AU-C#1/#8] |
| **AML / travel rule** | BSA/CIP/CDD; OFAC; travel rule ≥$3,000; BOI domestic exempt (Aug 2026); RRE rule vacated [US-C#15/#22] | AMLD → AMLR (10 Jul 2027); AMLA direct supervision 2028; TFR no de minimis, self-hosted >€1,000 [EU-C#11/#13] | FDL 10/2025 (eff. 14 Oct 2025); Cabinet Decision 134/2025; travel rule ≥AED 3,500; unhosted-wallet EDD [ME-C#11/#12] | AML Law + CMA rules [I] | AML/CTF Amendment Act 2024; reforms 31 Mar 2026; tranche 2 real estate 1 Jul 2026 [AU-C#3] |
| **Tax reporting** | 1099-DA (tokenized securities) unless limited-access regulated network; basis from 2026; K-1; FIRPTA / §1446(f); FATCA [US-C#20/#21] | DAC8 from 1 Jan 2026 (reports by 30 Sep 2027); CRS; national WHT; RETT share-deal rules [EU-C#12] | No PIT; CT 9%; VAT; DLD 4% fee; CRS; CARF consultation Oct 2025 [ME-C#1] | RETT 5%; VAT 15%; Zakat; CRS [I] | AMIT/AMMA; MIT withholding; TFN; CRS; landholder and foreign surcharge duty [I] |
| **Land title / foreign ownership** | County recorder; SPV holds deed; state foreign-adversary land laws; CFIUS [I] | National land registries; SPV holds title; some non-EU buyer restrictions [I] | Freehold zones; **DLD tokenised title linked to registry** [ME-C#8] | Non-Saudi ownership in force 22 Jan 2026; holy-cities limits [ME-C#18] | Torrens/PEXA; RE holds title; **FIRB established-dwelling ban to 30 Jun 2029** [AU-C#9] |
| **Key 2025–26 change** | SEC tokenization taxonomy; DTC NAL; TA modernisation proposal (Sep 2026); CLARITY failed cloture 15 Sep 2026 [US-C#1/#7/#11/#4] | Market integration package (4 Dec 2025); MiCA grandfathering ended 1 Jul 2026; DAC8 live [EU-C#7/#2/#12] | DLD tokenisation live (May 2025) → secondary market (Feb 2026); VARA v2.0; SCA → CMA; CMA VA framework (Apr 2026) [ME-C#8/#9/#14/#13] | Foreign ownership law live (22 Jan 2026); REGA tokenisation standard, first tokenised deed and sandbox track confirmed (SPA, 19 Nov 2025 and 13 Feb 2026; synthesis fetch F2); final tokenisation law / executive regulations and the CMA's role still unconfirmed [ME-C#18][S] | INFO 225 update (Apr 2026); **Digital Assets Framework Act 2026** enacted (No. 38 of 2026; Royal Assent 8 Apr 2026; commences 8 Apr 2027; 6-month transition) — confirmed on legislation.gov.au during synthesis (`04-synthesis/conflicts-and-gaps.md` #1), superseding the Sep 2025 exposure-draft status in australia.md; AML tranche 2 (Jul 2026); FIRB ban extended [AU-C#1/#4/#3/#9] |

---

## C. Jurisdiction parameter table (rules-engine seed values)

| Parameter | US | EU | UAE (VARA/DLD) | Australia | Evidence |
|---|---|---|---|---|---|
| Travel-rule trigger | ≥ USD 3,000 | No de minimis (crypto); self-hosted proof > €1,000 | Beneficiary verification ≥ AED 3,500 daily aggregate | Reform rules 2026 (TBC) | US [I]; EU-C#13; ME-C#11; AU [I] |
| Retail/individual "qualified" threshold | $1M NW (excl. home) or $200k/$300k income | Professional opt-up: 2 of 3 tests (€500k portfolio…) | AED 3.5M NA or AED 700k income | $2.5M NA or $250k income; or $500k ticket | US-C#10; EU [I]; ME-C#3; AU [I] |
| Minimum-investment verification shortcut | $200k natural / $1M entity (506(c) NAL) | — | — | $500k wholesale ticket | US-C#8; AU [I] |
| Retail raise cap (crowd regime) | Reg CF $5M / 12 mo rolling | ECSP €5M / 12 mo per project owner | — | CSF $5M / 12 mo | US-C#25/#27; EU-C#10; AU [I] |
| Prospectus-free public offer ceiling | Reg A T2 $75M (qualified offering circular) | €12M (MS option €5M) | — | — | US-C#26; EU-C#9 |
| Resale lock | Reg CF 1 yr; Rule 144 1 yr (non-reporting) | Contractual / exemption-based | DLD-controlled market | Scheme constitution | US-C#25; ME-C#9 |
| Holder-count trigger | 2,000 / 500 non-accredited (§12(g)) | — | — | MIS registration >20 members (s601ED) | US [I]; AU [I] |
| Record retention | 6 yrs (TA proposal) | 5 yrs (AML) | 8 yrs (VARA classification records) | 7 yrs | US-C#11; ME-C#3; EU/AU [I] |
| Cooling-off / reflection | — | 4 calendar days (ECSP non-sophisticated) | ≥1 week retest after failed suitability | 5 business days (CSF) | EU-C#10; ME-C#3; AU [I] |

---

## D. Suggested admin-panel module map (derived)

1. **Licences & Permissions** (#1)
2. **Offerings & Product Governance** (#2–#5, #33)
3. **Investor Onboarding**: KYC/KYB, UBO, Screening (#6–#8, #20)
4. **Investor Classification & Tests**: accreditation, categorisation, knowledge/suitability, wholesale certificates, VARA classification (#9–#14)
5. **Eligibility Rules Engine**: nationality, residency, foreign-ownership (#15)
6. **Orders, Escrow & Client Money** (#16–#19)
7. **AML Case Management, Travel Rule, BWRA** (#21–#24)
8. **Registry / Cap Table / Token Compliance Rules** (#25–#27)
9. **Custody & Reserves** (#28–#29)
10. **Marketplace Control & Surveillance** (#30–#32)
11. **Disclosure, Filings & Investor Reporting** (#33–#35)
12. **Marketing Compliance** (#36)
13. **Tax Profiles, Reporting & Withholding** (#37–#40)
14. **Asset Registry & Property Operations** (#41–#42)
15. **Shariah Compliance** (#43)
16. **Audit, Retention & Regulator Data Room** (#44–#45)

All module designs above are [I]: they are RealX-oriented inferences from the cited obligations, not observed features of any regulator-mandated system.
