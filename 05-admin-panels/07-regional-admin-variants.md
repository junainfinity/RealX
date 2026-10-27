# 07 — Regional Admin-Panel Variants (US · EU · MENA · Australia)

> **EXTRAPOLATION; not legal advice.** The reference panel (`01`) is jurisdiction-agnostic. Each region is a **jurisdiction pack** (M10/M27): parameters, extra pages, extra roles, calendars and report formats. The design principle comes from the regulatory pack: "every jurisdiction ends up needing the same ~15 control families; they differ mainly in parameters" [`XCC` design principle]. Obligations and dates are as of 23 Sep 2026 and are taken from `01-regulatory/`. Confidence grades follow `00-overview.md` §3.

**Legend for the "Δ" column:** **ADD** = new page, control or role · **PARAM** = parameter of an existing control · **GATE** = feature flag or licence gate · **FMT** = report or file format.

---

## 0. Cross-region parameter crosswalk (rule-pack seed values)

| Parameter | US | EU | MENA (UAE / KSA) | Australia | Module |
|---|---|---|---|---|---|
| Investor classes | Non-accredited / accredited / QP | Retail / professional / ECP; ECSP sophisticated vs non-sophisticated | UAE VARA Retail / Qualified / Institutional; DFSA Retail / Professional; KSA retail / qualified | Retail / wholesale / sophisticated / professional | M07 |
| "Qualified" individual threshold | $1M NW (excl. home) or $200k/$300k income | Opt-up: 2 of 3 tests (e.g., portfolio > €500k) | VARA: AED 3.5M net assets or AED 700k income, plus knowledge | $2.5M net assets or $250k income (accountant certificate); or $500k ticket | M07 |
| Shortcut verification | $200k natural / $1M entity minimum ticket + reps (NAL 12 Mar 2025) | — | — | $500k ticket | M07/M05 |
| Retail raise cap | Reg CF $5M / 12 months rolling (from each closing) | ECSP €5M per project owner / 12 months | DFSA crowdfunding caps per investor | CSF $5M / 12 months | M04 |
| Prospectus-free ceiling | Reg A T2 $75M (qualified circular) | €12M (MS option €5M) | — | — | M04 |
| Per-investor caps | Reg CF limits; Reg A T2 10% of income/NW | ECSP €1,000 / 5% NW warning and consent | DFSA AED 183.5k per property / 367k per year; platform caps (PRYPCO 20%) | CSF $10k per issuer; platform caps (BrickX 5%) | M05/M10 |
| Cooling-off / reflection | — (contractual) | **4 calendar days** (ECSP non-sophisticated) | PRYPCO 24h (contractual); VARA **≥ 1 week** retest after failed suitability | CSF 5 business days; retail PDS 14 days [I] | M05/M07 |
| Resale lock | Reg CF 1 year; Rule 144 1 year (non-reporting); Reg S compliance period | Contractual / exemption-based | DLD 3-month lock-in (PRYPCO); DFSA SPV 1 year (Stake/SmartCrowd) | Scheme constitution | M10/M14 |
| Holder-count triggers | §12(g) 2,000 / 500 non-accredited; 3(c)(1) 100 owners | — | — | MIS registration > 20 members (s601ED) | M10 |
| Travel-rule trigger | ≥ $3,000 | No de minimis (crypto); self-hosted > €1,000 proof | **≥ AED 3,500 daily aggregate**; unhosted EDD | Reformed VA travel rule (2026) | M20 |
| Suspicious-report clock | SAR 30 days | FIU per national law | goAML "without delay" | SMR 24h (terrorism) / 3 business days | M20 |
| Record retention | 6 years (TA proposal) | 5 years (AML) | 8 years (VARA classification) | 7 years | M18/M25 |
| Payout stablecoin allow-list | GENIUS-permitted issuers (effective ~18 Jan 2027) | MiCA EMTs (EURC, EURCV); non-MiCA USDT restricted | CBUAE-licensed AED payment tokens (e.g., AE Coin) | Instrument 2025/867 relief (e.g., AUDM) | M16 |
| Sovereign ID | — (SSN/TIN) | eIDAS (national eIDs) [I] | **UAE PASS / Emirates ID; Nafath / Absher / Yakeen** | DVS-based eIDV (Equifax ID Matrix) | M07 |

Sources: [`XCC` §C; `REG-US` §2–3; `REG-EU` §2–4; `REG-ME` §2–4; `REG-AU` §2–4; `mena/01-prypco` §2.8; `mena/03-stake` §2.6].

---

## 1. United States — what the admin panel must add or change

**Operating model assumed:**
- Reg D 506(c) + Reg S SPVs through a partner broker-dealer and a registered TA;
- then Reg A Tier 2 or Reg CF for retail;
- then a partner ATS after holding periods [`REG-US` §2 "practical sequence"].

**Default extra roles:** `X-BDP` (registered principal), `X-TA` (external transfer agent).

| Δ | Area | Module / page | Control and parameters | Obligation / evidence | Conf. |
|---|---|---|---|---|---|
| ADD | **Accreditation queue** | M07.3 | Methods: third-party letters (CPA/attorney/RIA/BD); tax/bank documents; **minimum-investment path $200k natural / $1M entity + written reps + no third-party financing**. Default validity 90 days [I]. Reviewer sign-off. Watch the INVEST Act (House-passed 11 Dec 2025; Senate pending) | Rule 506(c); NAL 12 Mar 2025 [`REG-US` §3]; RedSwan queue [`us/02-redswan` §11 #3] | H |
| PARAM | Exemption packs | M10.1 / M04.2 | **506(b)**: no general solicitation, ≤ 35 sophisticated non-accredited. **506(c)**: accredited-verified only. **Reg S**: offshore only, no directed selling, US-person screen, distribution-compliance lock. **Reg A T1/T2**: $20M / $75M, T2 non-accredited 10% limit unless listed. **Reg CF**: $5M rolling from each closing, investor limits (greater of $2,500 or 5% below $124k; else 10% capped at $124k), calendar-year income, 1-year resale bar, qualified third-party escrow | [`REG-US` §2–3, §6] | H |
| ADD | **Reg CF / Reg A investment-limit tracker** | M05.2 / M06.2 | Cross-offering 12-month ledger per investor (Reg CF); 10% income/NW check (Reg A T2); hard stop at limit | [`XCC` #10]; Arrived Reg A calculator [`us/03-arrived` §11 #3] | H |
| ADD | **Filing calendars** | M19.1 | **Form D ≤ 15 days after first sale**; **state blue-sky notice filings keyed to investor residence**; Form C / C-AR; 1-A / 253G2 / **1-K / 1-SA / 1-U**; stale Reg CF financials after 30 April; FOCUS (BD); Form TA-2 (if the TA proposal is adopted) | [`REG-US` §7, checklist]; WA DFI blue-sky consent order ($40k) [`us/03-arrived` §2.3] | H |
| ADD | **TA golden record** | M11 + `X-TA` | Master securityholder file controlled by the TA. **Sep 2026 proposal:** DLT may be "all or part" of the master file if the TA keeps **exclusive control**; names and addresses still required (wallet ≠ identity); **6-year retention**; audit trail; Form TA-2 tokenisation disclosures; comments due 3 Nov 2026. TA co-approves forced transfers and re-issues | [`REG-US` §5]; SEC staff statement 28 Jan 2026 (issuer-sponsored tokens) [`REG-US` §1] | H |
| ADD | **12(g) holder monitor** | M10.4 / M11.6 | Per SPV: assets ≥ $10M **and** 2,000 holders of record or 500 non-accredited → alert at 80%. **Each wallet may count as a holder of record**. Reg A T2 / Reg CF conditional exemptions; 3(c)(1) 100 beneficial owners; 3(c)(7) qualified purchasers | [`REG-US` §1]; RealT 2,000 cap [`us/04-realt` §2.3]; RedSwan 3(c)(1) alert [`us/02-redswan` §11.4] | M |
| ADD | **Tax forms and withholding** | M13 | **1099-DIV** (REIT / C-corp); **1099-B vs 1099-DA** (a "limited-access regulated network" flag decides); basis reporting from 2026; **K-1** for partnership SPVs (PTP §7704 risk if ATS-traded); **1042-S**, FDAP 30% or treaty rate; §1446(a) ECI; **§1446(f) 10%**; **FIRPTA 15%** on USRPHC dispositions (flag secondary trades by foreign holders); W-9/W-8 expiry; state non-resident withholding | [`REG-US` §8]; Lofty/Arrived/mogul/RealT tax patterns | M |
| ADD | **BD / ATS supervision console** | M21.4, M19, M14, M16 | **FINRA 2210** principal approval of communications; **Reg BI / Form CRS** delivery log; OSJ supervision; 17a-4 archives; **Rule 15c3-3 "five conditions"** for BD custody of crypto-asset securities; **Rule 15c2-4** escrow for contingency offerings; ATS window ops; fair access | [`REG-US` §5–7]; RedSwan BD console [`us/02-redswan` §11 #4] | M |
| PARAM | **OFAC** and AML | M20 | Strict-liability name **and wallet** screening (the DTC pilot screens every wallet); CIP/CDD with 25% owners + control prong (domestic BOI exempt since Aug 2026; CDD unchanged); **SAR 30-day clock**; travel rule ≥ $3,000; RRE rule vacated (19 Mar 2026), so watch GTOs | [`REG-US` §4] | H |
| GATE | Stablecoin allow-list | M16.6 | Only **GENIUS-permitted payment stablecoins** once effective (earlier of 18 Jan 2027 or 120 days after final rules) | [`REG-US` §4, §10] | M |
| ADD | Marketing claims library | M21.5 | 506(b) no general solicitation; 506(c) allowed; Reg CF tombstone off-portal; Reg S no directed selling; **banned claims "backed by real estate", "capital protected"** (Texas SSB order, Jun 2026) | [`REG-US` §7, §11] | M |
| PARAM | Foreign ownership | M10.3 | CFIUS near military installations; **state foreign-adversary land laws** (e.g., FL SB 264, TX 2025); AFIDA for agricultural land; nationality × property-state check before allocation | [`REG-US` §9] | L |
| ADD | **Municipal compliance** | M22.5 | Rental registration / certificate of compliance; code violations; blight tickets; water bills; **county tax-foreclosure countdown**. This is the RealT lesson (Detroit suit 2 Jul 2025; fiduciary Apr 2026; liquidation 2 Jul 2026) | [`REG-US` §9, §11]; [`us/04-realt` §2.8] ⚑ | M |
| GATE | Money movement | M16 | Route funds via a BD, a bank escrow or a qualified third party (e.g., Modern Treasury at Arrived) to avoid MSB / state money-transmitter exposure | [`REG-US` §4]; [`us/03-arrived` §2.4] | M |
| GATE | Secondary venue mode | M14.1 | Partner ATS only (Reg ATS via BD). The SEC innovation exemption (17 Sep 2026) covers **tokenized NMS stock only**, not real-estate tokens | [`REG-US` §6] | H |

---

## 2. European Union — additions and changes

**Operating model assumed:** one of the following:
- an **ECSP licence** (≤ €5M per project owner per 12 months; bulletin board only);
- a **MiFID** investment firm or tied agent with prospectus exemptions (≤ €12M) or a prospectus;
- a national DLT register (eWpG, Ley 6/2023, DEEP, LU Blockchain Laws, TVTG) for on-chain legal records;
- DLT Pilot venues for trading [`REG-EU` §2].

**Default extra roles:** placement ESI / MiFID principal (as `X-BDP` analogue), DLT register-keeper (as `X-TA` analogue).

| Δ | Area | Module / page | Control and parameters | Obligation / evidence | Conf. |
|---|---|---|---|---|---|
| GATE | Licence routes | M02.3 / M27.5 | Feature flags bound to ECSP / MiFID / DLT-register / DLT Pilot licences. **ECSP: bulletin-board mode only; multilateral matching disabled** unless an MTF/DLT MTF licence or partner is linked | [`REG-EU` §2, §6] | H |
| ADD | **ECSP KIIS builder** | M04.6 / M18.3 | One KIIS per offer; NCA language; version control; marketing must stay consistent with the KIIS | ECSPR Art 23 [`REG-EU` §7] | H |
| ADD | **Entry knowledge test + loss simulation** | M07.4 | Non-sophisticated investors; versioned question bank; re-test every 2 years | ECSPR Art 21 [`REG-EU` checklist] | H |
| ADD | **€1,000 / 5% warning** | M05.2 | Ticket vs declared net worth; explicit consent captured and logged | ECSPR Art 21(7) | H |
| ADD | **4-day reflection period** | M05.3 | Order state `reflection_period` (4 calendar days) with revoke button and timer | ECSPR Art 22 | H |
| ADD | **MiFID categorisation and appropriateness** | M07.5 / M07.4 | Retail / professional (2 of 3 opt-up tests, annual review) / ECP; **appropriateness** for complex products (unlisted SPV shares); suitability for advice/portfolio management (Propvest Sparplan); target market; PRIIPs KID if packaged | [`REG-EU` §3]; [`eu/03-propvest` §2.3] | M |
| ADD | MiFID cost reports and best execution | M19.5 | Ex-ante / ex-post cost reports per client; best-execution records | MiFID II Art 24–25 [`REG-EU` §7]; [`eu/03-propvest` §11 #12] | M |
| PARAM | Raise-cap meters | M04.5 | **Per issuer per Member State**: prospectus €12M (MS option €5M; ES €8M reference); ECSP €5M per project owner | [`REG-EU` §2]; Reental ESI route [`eu/01-reental` §2.3] | H |
| ADD | **DAC8 / CRS / WHT** | M13 | **DAC8** (from 1 Jan 2026; first RCASP reports **30 Sep 2027**) only if RealX deals in MiCA crypto-assets (e.g., EMT legs); **CRS/DAC2** for tokenised securities; national WHT (ES 19%, DE Abgeltungsteuer, FR 12.8%/25%) with treaty relief; FASTER (2030) | [`REG-EU` §8] | M |
| ADD | **Travel rule (TFR)** | M20.6 / M08.1 | No de minimis on crypto-asset transfers; **self-hosted wallet > €1,000 needs an ownership proof** (signature or Satoshi test); tokenised securities are not TFR crypto-assets, but EMT cash legs are. Best practice: apply to all wallet movements | TFR 2023/1113 [`REG-EU` §4] | M |
| GATE | MiCA stablecoins and utility tokens | M16.6 / M02 | Payout assets = **MiCA EMTs** (EURC, EURCV); non-MiCA USDT restricted for EU CASPs (Reental risk); any utility token (RNT/BST/BKN-type) needs a MiCA white paper; CASP grandfathering ended **1 Jul 2026** | [`REG-EU` §0, §10]; [`eu/01-reental` §2.8] | M |
| ADD | **GDPR erasure vs immutability** | M06 / M18.6 / `04` §8 | No PII on-chain; erasure = pseudonymise the off-chain record and seal the address link; AML retention (5 years) overrides erasure; DSAR export | GDPR; [`eu/02-blocksquare` §2.8] (PII stays with the CP) | M |
| PARAM | AML | M20 / M07 | AMLD → **AMLR from 10 Jul 2027**; **AMLA direct supervision 2028**; UBO ≥ 25%; EU restrictive measures incl. Russia/Belarus crypto and securities bans | [`REG-EU` §4] | H |
| ADD | **RETT share-deal monitor** | M10.4 | Holder-concentration thresholds per SPV (e.g., Germany 90% within 10 years) | [`REG-EU` §8] | L |
| PARAM | Non-EU buyer rules | M10.3 | Permit or approval rules (DK, MT, AT Länder, FI, CY, HR reciprocity) keyed to asset country | [`REG-EU` §9] | L |
| ADD | Placement intermediary | M24 / M04.3 | ESI / MiFID placement sign-off where offers must go through an authorised entity (Spanish model) | [`eu/01-reental` §2.3] | M |
| PARAM | Complaints | M23 | MiFID complaints handling; national ADR (e.g., Bundesbank Schlichtungsstelle); EdW/investor-compensation disclosures | [`eu/03-propvest` §2.4] | M |

---

## 3. MENA — additions and changes

### 3.1 UAE — Dubai onshore (VARA + DLD), DIFC (DFSA), federal (CMA)

**Operating model assumed:**
- **route (a):** a VARA broker-dealer that acts as a DLD-approved distributor on the DLD/Ctrl Alt rails (the PRYPCO model), or
- **route (b):** own Category 1 issuance + BD.

DIFC crowdfunding SPVs (the Stake/SmartCrowd model) remain the off-chain alternative [`REG-ME` §0, §2].

**Default extra roles:** `SHRO` (Shariah officer), DLD/issuer service accounts (`SVCA`), distributor tenants (`X-DST`).

| Δ | Area | Module / page | Control and parameters | Obligation / evidence | Conf. |
|---|---|---|---|---|---|
| GATE | VARA licences | M02.3 | BD vs **Category 1 VA Issuance (ARVA)**; IPA → full-licence tracker (Stake, SmartCrowd); **issuer licence revoked if no issuance within 6 months**; CMA-VASP registration numbers | [`REG-ME` §2]; [`mena/03-stake` §2.3] | H |
| ADD | **DLD registry sync & certificates** | M11.4 | Register owners and TokenIDs with DLD via the issuer; **Property Token Ownership Certificate** per investor; no on-chain move without a registry ack; nightly XRPL ↔ register ↔ DLD reconciliation | [`mena/01-prypco` §2.2, §11 #5]; [`mena/02-ctrl-alt` §11 #4] | H |
| PARAM | **DLD market rules** | M14 / M10 | **±15% band vs DLD smart valuation**; **3-month lock-in**; **20% holder cap**; minimum ticket AED 1,000 (AED 2,000 at launch; primary-offering applicability of the AED 1,000 unverified, `04-synthesis/conflicts-and-gaps.md` #19); approved-buyer visibility; controlled secondary market (Phase Two, 20 Feb 2026) | [`mena/01-prypco` §5.4, §9]; [`REG-ME` §6] | H |
| ADD | **Emirates ID / UAE PASS** | M07 | EID OCR + liveness; UAE PASS login and e-signature; EID expiry suspends the account; currently **EID holders only** for DLD tokens | [`mena/01-prypco` §2.5–2.6] | H |
| ADD | **VARA investor classification** | M07.5 | Retail / **Qualified (AED 3.5M net assets or AED 700k income + knowledge)** / Institutional; SoW/SoF; **suitability test with ≥ 1-week cooling-off after failure**; product gating; **8-year records** | VARA circular 8 Jan 2026 [`REG-ME` §3] | H |
| PARAM | **VARA travel rule** | M20.6 | **Beneficiary verification for daily aggregated transfers ≥ AED 3,500**; unhosted-wallet EDD (may decline or return); domestic limited-data allowance if the full set is available within 3 working days | VARA circular 24 Feb 2026 [`REG-ME` §4] | H |
| ADD | **VARA / ARVA reporting pack** | M19 | Periodic VARA returns; ARVA monthly disclosures; **6-monthly independent audit + senior-management attestation**; **capital = max(AED 1.5M, 2% of reserves)**; **fee-free redemption** SLA; BWRA (guidance Jun 2026); FDL 10/2025 gap assessment | [`REG-ME` §2, §4, checklist]; [`mena/02-ctrl-alt` §11 #7] | M |
| ADD | AML specifics | M20 | **EOCN / UN TFS** screening; **goAML** STR filing; DNFBP real-estate reports for cash- or VA-funded purchases | [`REG-ME` §4] | M |
| GATE | **CBUAE payment tokens** | M16.6 | Only **CBUAE-licensed AED payment tokens** (e.g., AE Coin) for onshore payments; VARA NoC registration record | VARA circular 24 Jul 2025 [`REG-ME` §4] | M |
| ADD | **RERA Trakheesi permits and VARA marketing** | M21.4 | **Trakheesi permit number per listing**; VARA Marketing Regulations disclaimers; endorsement rules; CMA financial-promotion rules | [`REG-ME` §7] | M |
| PARAM | Client money | M16 | Segregated client-money accounts (Zand, ENBD); Dubai escrow law for off-plan | [`mena/01-prypco` §2.4]; [`REG-ME` §5] | H |
| PARAM | Cooling-off and complaints | M05.3 / M23 | 24h cooling-off (contractual); complaints acknowledged ≤ 7 days, resolved ≤ 30 days (max 8 weeks); 8-year retention; DIFC Courts | [`mena/01-prypco` §2.8] | H |
| ADD | **Shariah board sign-off** | M03 / M04 (+ `SHRO`) | Fatwa repository; AAOIFI standards (No. 12 Sharikah, No. 17 Sukuk, Ijarah); tenant and leverage screening; purification calculator | [`REG-ME` §7]; PRYPCO Blocks Shariah-certified [`mena/01-prypco` §1] | L |
| PARAM | **Residency / nationality** | M10.3 | Freehold zones; EID-holder restriction for DLD tokens; excluded-nationality lists (Stake: e.g., Iran, Syria, North Korea…); non-residents allowed on DIFC SPV products (Stake) | [`mena/03-stake` §2.5–2.6] | M |
| PARAM | **DIFC (DFSA) crowdfunding** | M10 / M05 | Retail caps **AED 183,500 per property / AED 367,000 per year**; Professional up to 33%; Islamic window; **Business Cessation Plan**; DIFC public-register verification | [`mena/03-stake` §2.3–2.8]; [`mena/05-smartcrowd` §2.3] | H |
| PARAM | Tax | M13 | No PIT; VAT 5% on fees; **DLD 4% transfer fee** (PRYPCO markets 2%); CRS; CARF consultation (Oct 2025) | [`REG-ME` §8] | M |

### 3.2 Saudi Arabia (REGA / RER / CMA)

| Δ | Area | Module / page | Control and parameters | Obligation / evidence | Conf. |
|---|---|---|---|---|---|
| ADD | **Nafath KYC tiers** | M07 | Nafath MFA; Yakeen/Absher data; verifiable credential per investor; **tiers: citizen / resident / non-Saudi** | [`mena/04-saudi` §2.5] | M |
| ADD | **Transfer-time eligibility** | M10.3 | Evaluated on **every transfer**: designated zones; **Makkah/Madinah limited to Saudi companies and Muslim individuals**; non-Saudi ownership law in force 22 Jan 2026 | [`REG-ME` §9]; [`mena/04-saudi` §2.5] | M |
| ADD | **RER registry filing** | M11.4 | Aamal submissions; fractional deed per investor; 1:1 deed token ↔ fractional tokens; **registry update = legal settlement**; rejection handling | [`mena/04-saudi` §2.2, §11.1] | M |
| ADD | **Weekly REGA sandbox report** | M19.1 | Weekly KPI report (profitability, local hiring, complaints); audit evidence room; sandbox stage tracker (6–24 months) | [`mena/04-saudi` §2.3] | M |
| PARAM | Escrow and DvP | M16.4 | Dedicated acquisition account (mada/SADAD); release only after RER confirmation | [`mena/04-saudi` §5.2] | M |
| ADD | Transaction taxes | M13.5 | **RETT 5%**; non-Saudi surcharge up to 5%; Zakat statements | [`mena/04-saudi` §1, §2.7] | M |
| PARAM | Pricing controls | M03.4 / M14 | **REGA price controls** before launch | [`mena/04-saudi` §2.8] | L |
| PARAM | CMA fund distribution | M10 / M05 | **50% of each fund reserved for qualified investors, 50% retail** (Stake) | [`mena/03-stake` §2.6] | M |
| GATE | Data residency and cyber | M27.8 | **In-Kingdom hosting**; NCA cybersecurity standards | [`mena/04-saudi` §7] | M |
| ADD | Exceptions desk | M09.4 | Inheritance, court orders and corrections built into workflows | [`mena/04-saudi` §9] | M |

### 3.3 Other MENA (notes)

- **Egypt (FRA):** fund-issuance, promotion and underwriting licences; **instalment schedules** for off-plan fractions; advisor-led resale (Nawy Shares) [`mena/05-smartcrowd` §2.3, §5.8]. Adds an instalment/default engine to M05/M16.
- **Qatar (QFC):** Investment Token rules, five TSP activities (custody capital QAR 1.8M) [`REG-ME` §2].
- **Bahrain (CBB):** CRA module. Both markets need licence-bound flags only (L confidence).

---

## 4. Australia — additions and changes

**Operating model assumed:** one of:
- a **registered MIS** with a Responsible Entity (own AFSL or RE-for-hire), one sub-trust per property or an umbrella scheme with sub-funds (BrickX / Assetora);
- a wholesale unregistered MIS to start.

The **Corporations Amendment (Digital Assets Framework) Act 2026** (No. 38 of 2026) is enacted: Royal Assent 8 Apr 2026, the whole Act commences **8 Apr 2027**, with a 6-month transition for existing operators (to ~8 Oct 2027) and a low-value exemption ($10m per 12 months; $5k per client). This was confirmed against legislation.gov.au during synthesis [`04-synthesis/conflicts-and-gaps.md` #1, F1/F5], superseding the "unverified" status in [`REG-AU` §2]. **What still needs counsel** is whether fractional MIS-unit tokens (and custodial holding of them) fall within the TCP definition — s761GD reads as "a single digital token" per underlying asset — or remain plain MIS interests under INFO 225 [`conflicts-and-gaps.md` #2]. The AU pack therefore carries both a TCP/DAP licensing track and an MIS-only track until that is settled.

**Default extra roles:** `X-RE` (RE director/compliance committee), custodian (`X-CUS` read + sign).

| Δ | Area | Module / page | Control and parameters | Obligation / evidence | Conf. |
|---|---|---|---|---|---|
| ADD | **RE compliance plan** | M19 (new page "Compliance plan") | Compliance-plan obligations mapped to controls; task calendar; compliance-committee minutes; **annual compliance-plan audit**; breach register | Corporations Act s601HA [`REG-AU` checklist] | M |
| ADD | **TMD / DDO** | M04.6 | TMD per product (library searchable by property/ASF, cf. Assetora); distribution conditions; review triggers; **significant dealings outside the TMD reported within 10 business days**; complaints data to issuer | Pt 7.8A [`REG-AU` §3]; [`australia/05-assetora` §11 #2] | H |
| ADD | **Wholesale certificate manager** | M07.3 | $500k ticket; accountant certificate ($2.5M net assets / $250k income) with **2-year expiry**; sophisticated-investor (s761GA) assessment record; professional investor | [`REG-AU` §3]; [`XCC` #14] | H |
| PARAM | Retail disclosure | M18.3 / M05.3 | PDS / SPDS versioning and in-use notices; PDS acknowledgement at checkout; retail PDS cooling-off (14 days) [I]; CSF 5 business days and $10k per issuer cap (if CSF used) | [`REG-AU` §2–3, §7] | M |
| PARAM | **AML/CTF** | M20 | AML/CTF Amendment Act 2024; reformed obligations for existing reporting entities and VASPs from **31 Mar 2026**; **tranche 2 from 1 Jul 2026** (real-estate professionals); **SMR 24h (terrorism) / 3 business days**; TTR ($10k cash); IFTI; VA travel rule; DFAT sanctions | [`REG-AU` §4]; [`australia/01-brickx` §12] | H |
| ADD | **FIRB foreign-person caps** | M10.3 / M10.4 | Foreign-person flag; **established-dwelling ban for foreign persons 1 Apr 2025 – 30 Jun 2029** (exceptions: ≥ 20-dwelling redevelopment, BTR, student, aged care); indirect interests in land-rich trusts; per-asset foreign-holding cap and pre-trade block; FIRB approval evidence store | [`REG-AU` §9] | M |
| ADD | **Landholder-duty concentration monitor** | M10.4 | Per trust: holder % and associated-person aggregation vs "significant interest" (e.g., NSW private unit trust 20%, company 50%); foreign-trust % for surcharge duty (e.g., NSW ≥ 50%); blocks threshold-crossing transfers. BrickX's 5% cap is the pragmatic version | [`REG-AU` §8]; [`australia/01-brickx` §2.6] | M |
| ADD | **AMIT statements and tax** | M13.3 | **AMMA statements**; annual tax/distribution statement (~3 months after 30 June); **MIT withholding 15%/30%**; **TFN 47%**; CRS/FATCA; CGT cost base; foreign-resident CGT withholding (direct sales) | [`REG-AU` §8]; [`australia/01-brickx` §2.7] | M |
| PARAM | Custody and prudential | M08 / M19.3 | Scheme property held by the RE or an **RG 133** custodian; **NTA $10M** (custody) / $150k incidental; **omnibus relief 2025/871** (record-keeping and reconciliation); stablecoin distributor relief **2025/867** (e.g., AUDM) | [`REG-AU` §5] | H |
| PARAM | Register | M11 | **Register of members (s169)**; if the chain is designated as register, document it in the constitution. The RBA flags legal finality of on-chain records as unresolved | [`REG-AU` §5]; [`australia/02-redbelly` §2.3] | M |
| GATE | **Market licence** | M14.1 | Order book for the scheme's own interests needs a **market licence (s791A) or exemption** (BrickX's basis unconfirmed); CS facility licence; **AMM = likely a "financial market" (s767A)** | [`REG-AU` §6]; [`australia/03-tokeniser` §2.3] | M |
| PARAM | Advertising | M21.4 | **RG 234 (updated 9 Jun 2026)**: yield claims, "guaranteed" language, finfluencer register | [`REG-AU` §7] | M |
| PARAM | Complaints | M23 | IDR plus **AFCA** membership; DDO complaints data | [`australia/01-brickx` §11 #14] | M |
| PARAM | Continuous disclosure | M19.6 | Registered schemes with ≥ 100 members are disclosing entities; annual financial reports and audits; periodic statements | [`REG-AU` §7] | M |
| ADD | Occupancy evidence | M22.9 | FIRB **vacancy fee** (dwelling not occupied or available for rent 183 days/year) for foreign-held residential | [`REG-AU` §9] | L |
| ADD | Pilot / relief tracker | M02.3 / M19 | Project Acacia relief conditions (time-bound AFS, market and CS exemptions); DFMI sandbox watch | [`australia/04-canvas` §2.3] | M |
| GATE | **DAF Act TCP/DAP licensing track** | M02.3 / M19.1 / M10.1 | Licence-condition and calendar entries for the TCP/DAP regime: commencement **8 Apr 2027**; 6-month transition to ~8 Oct 2027; low-value exemption meter ($10m per 12 months; $5k per client) in M04.5; feature flag "custodial holding of tokens" bound to TCP/DAP licence or exemption status. Counsel decision recorded in M02 on whether the product is a TCP product or an MIS interest only | [`04-synthesis/conflicts-and-gaps.md` #1–2, F1/F5]; [`australia/02-redbelly` §2.3] | M |

---

## 5. Region-specific sidebar deltas (what a regional admin sees in addition)

| Region | Extra pages switched on by the pack |
|---|---|
| **US** | Accreditation queue (506(c)) · Blue-sky tracker · EDGAR filings (D / C / 1-A / 1-K / 1-SA / 1-U) · 12(g) & 3(c)(1) monitors · BD supervision (2210, Reg BI) · TA golden-record certification · 1099 / K-1 / 1042-S / FIRPTA centre · Municipal compliance (county tax-foreclosure clocks) |
| **EU** | KIIS builder · Knowledge test & loss simulation · Reflection-period queue · €1,000 / 5% consent log · MiFID categorisation & appropriateness · Cost reports · Raise-cap meters per Member State · DAC8 / CRS · TFR self-hosted proofs · GDPR DSAR / erasure desk |
| **MENA** | DLD registry sync & certificates · DLD band & lock-in controls · VARA classification & suitability lockouts · VARA / ARVA reporting pack · goAML STR · EOCN screening · Trakheesi permit registry · Shariah board desk · Nafath / RER console · Weekly REGA report · DFSA crowdfunding caps · Cessation plan run-book |
| **AU** | RE compliance plan & committee · TMD / DDO library & 10-business-day reports · Wholesale certificates (2-year) · AUSTRAC SMR / TTR / IFTI · FIRB foreign-person register · Landholder-duty & foreign-trust monitor · AMIT / AMMA centre · Custody / NTA prudential · AFCA complaints |
