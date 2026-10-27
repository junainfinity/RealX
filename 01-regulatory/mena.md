# MENA — Regulatory Landscape for Tokenised Fractional Real Estate
### UAE (Dubai VARA/DLD/RERA, DIFC-DFSA, ADGM-FSRA, federal CMA [ex-SCA], CBUAE) · Saudi Arabia · Bahrain · Qatar · brief notes: Egypt, Oman, Kuwait, Morocco

> **Research, not legal advice.** Snapshot as of 23 Sep 2026; confirm with licensed UAE/KSA/Bahrain/Qatar counsel.

**Evidence labels:** [C#n] confirmed by numbered source n · [I] inferred (reasoning, or well-established law not re-fetched this session) · [S] speculative.

**Research constraint.** The session's shared WebSearch quota was exhausted before this region, so it was researched by WebFetch of regulator sites only. Several regulator sites returned no content or blocked fetching: CBB (403), CBB rulebook (404), QFCRA rulebook (robots.txt), and the DLD, CBUAE and ADGM news pages (no article content). Bahrain, ADGM, CBUAE and the "brief-note" countries therefore rely mainly on [I].

**Bottom line.**
- **Dubai onshore is the most developed real-estate tokenisation regime in the world.** DLD's REES tokenisation project went live on **24 May 2025**. Title-deed tokens are minted on the XRP Ledger in sync with DLD's registry, and Ctrl Alt, the **VARA Category 1 issuer**, issues them. They are distributed by **PRYPCO Mint**, a **VARA broker-dealer**. Access was initially **Emirates-ID holders only**, from **AED 2,000** [C#7][C#8].
- **Secondary trading** started with Phase Two on **20 Feb 2026** (~7.8M tokens eligible, 10 properties, ~AED 18.5M) [C#9].
- VARA classifies such tokens as **Asset-Referenced Virtual Assets (ARVAs)**. Issuing them requires a **Category 1 VA Issuance licence** and must meet Annex 2 requirements: segregated reserve assets, 6-monthly audits, redemption rights, and capital of the higher of AED 1.5M or 2% of reserves [C#5].
- A new entrant's realistic Dubai routes:
  - (a) become a DLD-approved distributor under a VARA **broker-dealer** licence, using the DLD/Ctrl Alt rails; or
  - (b) obtain its own **Category 1 issuance** + broker-dealer licences (as Tokinvest did) [C#7].
- Alternatives:
  - **DIFC (DFSA)**: tokens are "Investment Tokens" (securities), plus a Property Investment Crowdfunding regime [C#15][C#17];
  - **ADGM (FSRA)**: digital securities / MTF [I];
  - **Saudi Arabia**: CMA fund-distribution FinTech permits (fractional exposure via real-estate funds) and the new non-Saudi ownership law in force **22 Jan 2026** [C#18][C#20]. For tokenisation proper, the route is **REGA's PropTech Regulatory Sandbox**: REGA completed the first title-deed tokenisation (NHC ↔ investors) on **19 Nov 2025** and announced "the world's first official tokenization standards", with technical specifications due early 2026 [C#25]; the sandbox's 2nd edition (**13 Feb 2026**) carries a fractional-ownership-in-tokenisation track and nine first-batch platforms are "operating officially" [C#26]. The final tokenisation law / executive regulations and the CMA's role remain **unconfirmed** (neither SPA release mentions the CMA) — see `02-platforms/mena/04-saudi-rer-national-tokenization.md`.

---

## 1. Classification of a fractional real-estate token

| Jurisdiction | Classification of a property-fraction token | Test / basis |
|---|---|---|
| **Dubai onshore (VARA)** | **ARVA** — a VA that "represents, or purports to represent, direct, indirect, current, future or contingent ownership of any RWA". RWAs include physical/tangible assets [C#5]. Issuance is **Category 1 VA Issuance**, needing a VARA licence (Rule I.D.1) [C#6] | VARA Issuance Rulebook v2.0 (effective 19 Jun 2025), Schedule 2 definitions, Annex 2 ARVA rules [C#5][C#2]. The Rulebook contains **no real-estate-specific provisions**; DLD supplies the property-law layer [C#5] |
| **Dubai onshore (DLD)** | A **tokenised title-deed fraction** recorded against DLD's registry. "Ctrl Alt integrated directly with DLD to synchronize digital and traditional ledgers" [C#8]. Phase Two adds **ARVA management tokens** alongside ownership tokens [C#9] | DLD tokenisation framework under the **Real Estate Evolution Space (REES)** initiative [C#8]; Law 7/2006 (real property registration) [I] |
| **DIFC (DFSA)** | **Investment Token**, i.e. a security or derivative in token form, regulated under the general securities regime, not the crypto regime [C#15]. A property SPV share token is an Investment Token [I] | DFSA token taxonomy (GEN 3A) [C#15] |
| **ADGM (FSRA)** | **Digital Security** (a Specified Investment in digital form) under the FSRA digital-securities guidance [I] | FSMR "Specified Investment" tests [I] |
| **UAE federal (CMA, ex-SCA)** | Security tokens fall under the CMA; crypto-assets under the CMA **Virtual Assets Framework** (13 Apr 2026: five modules, eight regulated activities) [C#13]. Onshore UAE outside Dubai follows CMA rules [I] | New CMA law (FDL 32 & 33 of 2025, effective 1 Jan 2026) [C#14] |
| **Saudi Arabia (CMA)** | A token giving profit or ownership rights = **security** under the Capital Market Law. Fractional retail exposure today comes via **real-estate investment fund units** distributed by CMA-permitted platforms [C#20][I]. "Real-estate contributions" (مساهمات عقارية) are a separate CMA-registered product [C#21] | CMA Investment Funds Regulations / Real Estate Investment Funds Regulations [I] |
| **Saudi Arabia (REGA / Real Estate Registry)** | **Registry-linked fractional title deed**: "machine-readable digital tokens linked to official real estate records", issued through the Real Estate Registry to sandbox PropTechs, with fractions registered in investors' names (first NHC deed tokenised 19 Nov 2025; nine sandbox platforms operating by Feb 2026) [C#25][C#26]. Whether the CMA additionally treats yield-bearing property tokens as securities is **unresolved** — the SPA releases do not mention the CMA [I] | REGA "official tokenization standards" (Nov 2025; technical specs early 2026) + PropTech Regulatory Sandbox 2nd edition (13 Feb 2026) [C#25][C#26]; Real Estate Registration Law (register has probative force) [I] |
| **Bahrain (CBB)** | Security-like tokens → CBB capital-markets rules (Vol 6). Crypto-assets → CBB **Crypto-Asset Module (CRA)** [I] | CBB Rulebook Vol 6 [I] (site blocked) |
| **Qatar (QFC)** | **Investment Token** — a permitted token representing a specified product, e.g. shares, debt, CIS units [C#23]. The framework recognises "tokens themselves and the property rights attached to their underlying assets" [C#22]. Cryptocurrencies are **excluded** [C#23] | QFC Digital Assets Regulations 2024 + QFCRA Investment Token Rules 2024 [C#23] |

## 2. Licence / exemption pathways

### Licence pathway options for a new entrant

| Pathway | Regulator | What it allows | Limits / conditions | Partners | Cost / time (indicative) |
|---|---|---|---|---|---|
| **DLD-approved distributor + VARA Broker-Dealer licence** | VARA + DLD | Distribute DLD-tokenised properties issued by Ctrl Alt | PRYPCO Mint model: EID holders only, min AED 2,000 at launch [C#8] (AED 1,000 on the product page as of Sep 2026 [C#27]). VARA broker-dealer licence (PRYPCO FZE VL/25/05/001, issued 24 May 2025) [C#7] | DLD, Ctrl Alt (issuer), Ripple Custody / XRPL [C#9][C#10], UAE bank for AED | VARA licensing typically 6–12 months [S]; capital per the Company Rulebook [I] |
| **VARA Category 1 VA Issuance (ARVA) + Broker-Dealer** | VARA | Issue your own ARVAs over real-world assets | Annex 2: reserve assets legally segregated, not rehypothecated (III.C.3); licensed custodian (III.C.2.a); independent audit every 6 months + senior-management attestation (III.D); fee-free redemption for equal AED value (III.E); cannot be called a "stablecoin" (III.F.1); capital = higher of **AED 1.5M or 2%** of average reserve value (III.G.1); licence revoked if no issuance within 6 months (I.D.7) [C#5][C#6] | Licensed VA custodian, auditor, DLD (for title linkage) [I] | Examples: Ctrl Alt (VL/25/05/002), Tokinvest (VL/2024/12/004) [C#7] |
| **VARA Category 2 issuance** | VARA | Issue without own licence **via a Licensed Distributor**, which validates issuer compliance (Rule I.E.1) [C#5] | Category 1 captures ARVAs, so a real-estate token is likely Category 1 [I] | Licensed distributor | Lower [I] |
| **DIFC: Operating a Property Investment Crowdfunding Platform** | DFSA | Investors pool into an SPV "that holds title to the property" [C#17] | Regime from 2019 (CP124). Detailed investor limits not confirmed in this session [C#17] | Custodian / client-money bank; SPV administrator | [S] |
| **DIFC: Investment Token activities** (dealing, arranging, MTF, custody) | DFSA | Securities licences apply to tokenised securities [C#15] | Innovation Testing Licence (12-month restricted test) available [C#16] | — | [S] |
| **ADGM: FSRA digital securities / MTF / custody** | FSRA | Primary issuance and MTF trading of digital securities; RWA tokenisation hub [I] | Professional-client focus common [I] | — | [S] |
| **UAE federal CMA** | CMA (ex-SCA) | Onshore securities / virtual-asset activities outside Dubai VARA and the free zones [C#13][C#14]. VARA licensees also carry **CMA-VASP registration numbers** [C#7] | New framework from 13 Apr 2026 [C#13] | — | [S] |
| **KSA: CMA FinTech ExPermit — "Investment and Real Estate Funds Distribution Platform"** | CMA | Distribute units of CMA-regulated real-estate funds digitally (15 permits, e.g., Stake Financial Technology 13/06/2023; Taadod 14/05/2025) [C#20] | Fund must be CMA-regulated; tokenisation not yet a permitted feature [I] | Licensed fund manager (CMA Capital Market Institution), custodian | ExPermit then full licence [I] |
| **KSA: debt crowdfunding ExPermit** | CMA | Offering and investment in debt instruments (~10–12 entities; count approximate, see QA note below) [C#20] | — | — | — |
| **KSA: REGA PropTech Regulatory Sandbox — fractional ownership / tokenisation track** | REGA (+ Real Estate Registry, RER) | Issue registry-linked fractional deeds via RER's national tokenisation infrastructure and distribute to retail (SAR 1,000 tickets observed at operators) [C#25][C#26] | Four stages (application → readiness → testing 6–24 months → exit); 2nd-edition applications closed 30 Apr 2026; operators report weekly to REGA (per operator interviews); final tokenisation law / executive regulations **not confirmed**; CMA role unresolved [C#26][I] | RER (SettleMint infrastructure), Nafath/Absher identity, mada/SADAD, escrow | Sandbox admission then 6–24-month test [C#26]; see `02-platforms/mena/04-saudi-rer-national-tokenization.md` |
| **Bahrain: CBB crypto-asset licence (Cat 1–4) / regulatory sandbox** | CBB | Crypto-asset services; security tokens via capital-markets rules [I] | Accredited-investor restrictions for some products [I] | — | [I] |
| **Qatar: QFC Token Service Provider** | QFCA + QFCRA | Five TSP activities: validation, token generation, custody, transfer, token exchange [C#22] | Investment tokens need a QFCA commercial licence + **QFCRA authorisation (3–6 months)**; e.g., custody min capital **QAR 1.8M** [C#23] | — | 3–6 months [C#23] |

> **QA note (Fable):** two re-fetches of the CMA ExPermit list on 23 Sep 2026 counted 11 and 12 entities in the debt-instruments category (automated extraction is unreliable for that table); the 15-entity funds-distribution count and the "Stake Financial Technology Company, 13/06/2023" and "Taadod, 14/05/2025" entries were confirmed. Treat the debt-crowdfunding count as approximate.

## 3. Investor eligibility, suitability, limits

- **VARA** (circular of 8 Jan 2026) [C#3]:
  - Three client types: **Retail, Qualified, Institutional**.
  - **Qualified (individual):** net assets ≥ **AED 3.5M** (excluding primary residence) **or** annual income ≥ **AED 700,000**, plus relevant knowledge.
  - **Qualified (entity):** net assets ≥ AED 3.5M, plus directors with VA knowledge.
  - Verify source of wealth and funds.
  - Mandatory **suitability assessment**; a failed test triggers a **≥1-week cooling-off** before retest.
  - Records kept **8 years**.
  - Retail cannot access Qualified-only or high-risk products.
- **DLD/PRYPCO model.** EID holders (UAE residents/nationals) from AED 2,000 at launch [C#8]. As of Sep 2026 PRYPCO's product page states "the minimum investment amount is AED 1,000" (without distinguishing primary from secondary), while its T&C still says AED 2,000 and 18+ and its help centre says AED 2,000 and 21+ [C#27][C#28] — see `02-platforms/mena/01-prypco-mint.md`. International access for real-estate tokens had not been confirmed in the sources fetched; PRYPCO's separate tokenised-gold product (Jun 2026) is open to international investors [I].
- **DFSA / FSRA.** Retail, Professional (assessed or deemed; typically ~USD 1M net assets for individuals) and Market Counterparty categories. Crowdfunding caps apply to retail [I].
- **KSA CMA.** Retail vs "Qualified Client" categories; private placements and funds are restricted to qualified or sophisticated investors [I].
- **QFC.** Investment tokens are subject to the QFCRA client-classification rules. The user guide does not address retail vs professional access [C#23].
- **Bahrain.** "Accredited investor" defined in the CBB rulebook (thresholds unverified) [I].

## 4. KYC / AML / CFT, sanctions, travel rule, beneficial ownership

| Topic | Rule | Detail |
|---|---|---|
| UAE AML law | **Federal Decree-Law No. 10 of 2025** (issued 30 Sep 2025, effective **14 Oct 2025**); implementing **Cabinet Decision No. 134 of 2025** [C#12][C#11] | Explicitly covers VASPs; beneficial-ownership transparency; stronger penalties. VARA required a **gap assessment within 60 days**, with remediation milestones at 90 and 120 days [C#12] |
| UAE travel rule | VARA circular of **24 Feb 2026** [C#11] | Applies to all UAE VASPs, including free zones. Originator and beneficiary data must accompany transfers. **Beneficiary verification for daily aggregated transfers ≥ AED 3,500**. EDD for **unhosted wallets**, which may be declined or returned. Domestic transfers may send limited data if the full set can be provided within 3 working days [C#11] |
| Other VARA AML circulars | AML/CFT/CPF implementation (4 Mar 2026); business risk assessment guidance (12 Jun 2026); PF NRA 2026 (1 Jun 2026); FATF high-risk jurisdictions (22 Jan 2026); EOCN targeted financial sanctions guidance (21 Nov 2025) [C#1] | Build: business-wide risk assessment, TFS screening against UAE Local Terrorist List + UN lists via EOCN [C#1][I] |
| Real-estate AML | Real-estate brokers are **DNFBPs**. Cash/VA-funded property purchases trigger goAML real-estate activity reports (thresholds unverified) [I] | Tokenised property purchases funded in VA should be flagged [I] |
| CBUAE payment tokens | VARA circular of 24 Jul 2025: **"Compliance with CBUAE PTSR – [NoC] Non-Objection Registration Requirement"** [C#1] | Using AED or other payment tokens as settlement brings in the CBUAE Payment Token Services Regulation (2024) [I]. **Only CBUAE-licensed AED payment tokens may be used for payments onshore** [I] |
| KSA | AML Law (Royal Decree M/20, 2017) + CMA AML rules; Yaqeen/Nafath e-KYC [I] | — |
| Bahrain / Qatar | CBB FC module; QFCRA AML/CFT rules; travel rule per FATF R.16 [I] | — |

## 5. Custody and client money

- **VARA.**
  - ARVA reserve assets go to **licensed custodians**, segregated from VASP funds [C#5].
  - Custody of client VAs requires a **VARA Custody Services licence** (or use of a licensed custodian) [I].
  - In the DLD project, **Ripple Custody** secures XRPL tokens (Ctrl Alt partnership, 16 Jul 2025) [C#9][C#10].
- **DLD tokens.** The title-deed record is synced with DLD's registry, so the on-chain token is not the sole source of truth [C#8]. Self-custody by investors is not offered: per the PRYPCO Mint T&C, tokens are "held in a warm multi party computation omnibus account, with Ctrl Alt Solutions DMCC as the custodian", and investor cash sits in a Zand Bank client-money account [C#28].
- **DFSA.** Since 12 Jan 2026, strengthened custody, governance and disclosure requirements for crypto tokens [C#15]. Investment tokens follow CMB/COB client-asset rules [I].
- **QFC.** "Token custody services" is a licensed TSP activity (QAR 1.8M capital) [C#23].
- **Client money.** Fiat must sit in segregated client accounts at UAE banks. Off-plan property payments remain subject to Dubai **escrow law** (Law 8/2007) [I].

## 6. Secondary trading

- **Dubai DLD Phase Two (20 Feb 2026).** "Controlled secondary market environment" on distribution platforms (PRYPCO and others), with ~**7.8M tokens eligible for resale** [C#9].
  - A new distributor would need DLD integration to offer resale [I].
  - Operating a multilateral venue for VAs requires a **VARA Exchange licence**; bilateral broker-dealer matching is narrower [I].
- **DFSA / FSRA.** Secondary trading of investment tokens / digital securities requires an **MTF** or ATS-equivalent licence. The DFSA property-crowdfunding regime allows a limited secondary facility (details unverified) [I][C#17].
- **QFC.** "Token exchange operations" (bringing together multiple buying and selling interests) is a TSP activity; basic order-routing is excluded [C#22].
- **KSA.** CMA fund units are redeemed through the fund [I]. For REGA-sandbox fractional deeds, a national tokenised marketplace was still being built as of Apr 2026 (operator-level resale exists, e.g. Jozo); the legally effective transfer is the Real Estate Registry entry, not the token movement [I — see `02-platforms/mena/04-saudi-rer-national-tokenization.md` §5.4, §9].
- **P2P transfers.** In Dubai, P2P transfers between whitelisted wallets outside the DLD rails would break title-registry sync. The DLD model keeps transfers inside approved platforms [I].

## 7. Disclosure, ongoing reporting, marketing

- **VARA.**
  - Whitepaper requirements (Issuance Rulebook Schedule 1); ARVA ongoing disclosures (Annex 2 Part II) [C#5].
  - The **Market Conduct Rulebook** has Part I "Marketing, Advertising and Promotions" (incl. Marketing Regulations), Part II Client Agreements, and Part V Public Disclosures incl. risk disclosure statements (v19 May 2025) [C#24].
  - VARA **Marketing Regulations** (2024) prohibit unlicensed VA marketing in Dubai and require disclaimers [I].
  - VARA publishes a "notice regarding endorsements" [C#4].
- **RERA / DLD.** Property ads in Dubai require a **Trakheesi advertising permit** [I]. Tokenised-property listings are likely subject to the same regime [I].
- **UAE CMA (ex-SCA).** "CMA Drives Responsible Financial Communication" (20 Apr 2026). Financial promotions rules apply federally [C#13][I].
- **DFSA.** Enhanced crypto-token disclosure from 12 Jan 2026 [C#15]; prospectus rules for investment tokens [I].
- **Shariah.** Islamic-structured offerings are expected to carry a Shariah board fatwa and follow **AAOIFI** Shari'ah Standards, notably:
  - No. 17 (Investment Sukuk);
  - No. 12 (Sharikah/partnership);
  - Ijarah standards.
  - Implications: rental-income tokens should avoid interest-bearing leverage and non-compliant tenants, and trading of tokens representing mainly tangible assets is permissible at market price [I]. The QFC framework explicitly contemplates tokens over "Islamic financial contracts" [C#23].

## 8. Tax reporting

| Country | Items |
|---|---|
| **UAE** | No personal income tax. Corporate tax 9% (from 2023) at SPV level; free-zone regimes apply [I]. **VAT 5%**: commercial property standard-rated; first supply of residential is zero-rated and subsequent residential sales/leases exempt [I]. **DLD transfer fee 4%** of property value [I] — not clear whether token transfers trigger it (the DLD model appears to embed fees) [S]. **CRS** participant [I]. **CARF:** UAE public consultation on CARF implementation (10 Oct 2025) [C#1] |
| **KSA** | **Real Estate Transaction Tax 5%** on property disposals [I]; VAT 15% (with RETT exemptions) [I]; Zakat/CIT at entity level; CRS/FATCA participant [I] |
| **Bahrain** | No income tax (except oil); VAT 10% [I]; CRS participant [I] |
| **Qatar** | QFC 10% corporate tax; CRS participant [I] |
| **Foreign investors** | FATCA/CRS self-certification for non-residents; home-country tax on distributions [I] |

## 9. Land registry / title linkage / foreign ownership

- **Dubai.**
  - Foreigners may own freehold in **designated freehold areas** (Law 7/2006; Regulation 3/2006) [I].
  - The DLD tokenisation model is the **only government-registry-linked tokenised title system** in scope. The deed is tokenised on XRPL and synced with DLD records [C#8].
  - The project targets **AED 60bn of tokenised property by 2033 (~7% of transactions)** [C#8].
  - Phase Two added management (ARVA) tokens and resale [C#9].
- **Abu Dhabi.** Foreign freehold in investment zones (Law 3/2005 as amended 2019) [I]. ADGM is a common-law jurisdiction; SPVs there hold onshore title via registration with ADREC/DMT [I].
- **Saudi Arabia.**
  - **Non-Saudi Real Estate Ownership System in force 22 Jan 2026** [C#18].
  - Eligible: non-Saudi residents (by Iqama), non-residents (via embassy digital ID), and foreign companies (after Ministry of Investment registration) [C#18].
  - Ownership regulated in Riyadh, Jeddah, Makkah and Madinah. **In the two holy cities, ownership is limited to Saudi companies and Muslim individuals** [C#18].
  - Zones set by a Geographical Zones Document due Q1 2026 [C#18]. Applications go through the **Saudi Properties** platform, integrated with the Real Estate Registration System [C#18][C#19].
  - Indirect ownership via funds or tokens is not addressed [C#18].
  - **Tokenised fractional deeds** are being issued to retail through the Real Estate Registry under REGA's sandbox (fractions registered in investors' names; first NHC deed 19 Nov 2025) [C#25][C#26] — see `02-platforms/mena/04-saudi-rer-national-tokenization.md`.
- **Bahrain.** Foreign freehold in designated areas (Resolution 43/2003 as expanded) [I].
- **Qatar.** Non-Qatari ownership in designated areas (Law 16/2018) [I].
- **Oman.** Integrated Tourism Complexes, plus 2023/2024 liberalisation for foreign residents [I].
- **Egypt.** Foreigners may own up to two residences (Law 230/1996) [I].
- **Admin control:** nationality × location × Muslim-only rule engine (KSA holy cities).

## 10. 2025–2026 developments

| Date | Development | Status |
|---|---|---|
| 19 May 2025 | **VARA Rulebooks v2.0** (all activity rulebooks + Issuance); compliance by 19 Jun 2025 [C#2] | In force |
| 24 May 2025 | **DLD tokenisation live** (REES; Ctrl Alt issuer; PRYPCO Mint; XRPL; EID holders; AED 2,000 min); Ctrl Alt and PRYPCO licensed the same day [C#7][C#8] | Live |
| 16 Jul 2025 | Ripple partners with Ctrl Alt (custody) [C#10] | — |
| 24 Jul 2025 | VARA circular: CBUAE PTSR non-objection registration [C#1] | In force |
| 30 Sep / 14 Oct 2025 | **FDL 10/2025 AML/CFT/CPF** issued / effective; Cabinet Decision 134/2025 [C#12][C#11] | In force |
| 10 Oct 2025 | UAE CARF public consultation [C#1] | Implementation pending [I] |
| 1 Jan 2026 | **SCA renamed Capital Market Authority** (FDL 32 & 33 of 2025) [C#14] | In force |
| 8 Jan 2026 | VARA investor classification circular (Qualified: AED 3.5M / AED 700k) [C#3] | In force |
| 12 Jan 2026 | **DFSA** removes Recognised Crypto Token list (firm-level suitability); lifts fund crypto limits [C#15] | In force |
| 22 Jan 2026 | **KSA Non-Saudi Real Estate Ownership System** in force [C#18] | In force |
| 20 Feb 2026 | **DLD Phase Two: secondary market** + ARVA management tokens [C#9] | Live |
| 24 Feb 2026 | **VARA travel-rule circular** (AED 3,500) [C#11] | In force |
| 13 Apr 2026 | **UAE CMA Virtual Assets Framework** (5 modules, 8 activities) [C#13] | Issued |
| 22 Jun 2026 | VARA licenses Tribe Tokenisation FZE (broker-dealer) [C#7] | Sign of more tokenisation licensees |
| 2025–26 | **CBUAE new law** (Federal Decree-Law No. 6 of 2025) extends CBUAE licensing to certain VA and payment-token activities with a transition period, reportedly to ~Sep 2026 [I]. **Not verified this session** — CBUAE site returned no content | Verify |
| 19 Nov 2025 | **KSA REGA: first title-deed tokenisation** (NHC ↔ investors, under government oversight) and "the world's first official tokenization standards"; technical specifications promised for early 2026 [C#25] | Confirmed (pilot + standards). Final tokenisation law / executive regulations and the CMA's role **not confirmed** (corrected at QA — an earlier draft of this row said "not confirmed by any fetched source") |
| 13 Feb 2026 | **KSA REGA PropTech Regulatory Sandbox, 2nd edition**, with a fractional-ownership-in-tokenisation track; nine first-batch platforms "operating officially"; applications to 30 Apr 2026 [C#26] | Live (sandbox) |

## 11. Enforcement / litigation

- No MENA enforcement action specific to real-estate tokenisation was confirmed this session. VARA's 2025–26 news list shows circulars, not published fines, in the items reviewed [C#1].
- VARA has historically fined unlicensed marketing and operations, and publishes enforcement notices on its site [I].
- **Structural risk:** mis-sync between on-chain tokens and DLD title records. The DLD model mitigates it by keeping issuance with one licensed issuer and resale inside approved platforms [I].

## Brief notes — other MENA markets

| Country | Status (all [I] unless marked) |
|---|---|
| **Bahrain** | CBB Vol 6 **Crypto-Asset Module (CRA)**: licence categories 1–4, custody, and travel-rule rules; CBB regulatory sandbox; **Stablecoin Issuance & Offering module (2025)**; Bahrain FinTech Bay. Security tokens fall under CBB capital-markets rules. Freehold for foreigners in designated areas. (CBB site 403 / rulebook 404 this session) |
| **Qatar** | QFC Digital Assets Regulations 2024 and Investment Token Rules 2024; Digital Assets Lab; five TSP activities; cryptocurrencies excluded [C#22][C#23]. QCB bans crypto services onshore, but runs tokenisation / CBDC work. Onshore real estate: foreign ownership in designated areas (Law 16/2018) |
| **Egypt** | FRA regulates fintech (Law 5/2022); real-estate fund / fractional ownership rules under FRA discussion; CBE Law 194/2020 prohibits crypto issuance and trading without CBE licence [S/I] |
| **Oman** | CMA (formerly FSA) virtual-asset regulatory framework under development; foreign ownership liberalised in designated zones [S/I] |
| **Kuwait** | CMA ban on crypto-asset activities (2023); CMA crowdfunding rules exist; no tokenisation regime [I] |
| **Morocco** | Draft crypto-asset law (AMMC/BAM) published 2025; crypto previously banned (2017 notice); no real-estate token regime [S/I] |

---

## Compliance obligations checklist

| Obligation | Legal basis | Who's responsible | Platform control needed |
|---|---|---|---|
| Hold the correct VARA licences (broker-dealer; Cat 1 issuance if issuing ARVAs) | VARA Regulations 2023; Issuance Rulebook I.D.1 | Platform / issuer | Licence register and activity-permission matrix; feature flags per licence |
| ARVA reserve segregation and licensed custody | Annex 2 III.C.2–3 | Issuer | Reserve-asset ledger, custodian reconciliation, no-rehypothecation attestations |
| 6-monthly independent audit + management attestation | Annex 2 III.D | Issuer | Audit calendar, evidence vault, attestation workflow |
| Fee-free redemption | Annex 2 III.E | Issuer | Redemption request queue with SLA timer, AED valuation |
| Capital: higher of AED 1.5M or 2% of reserves | Annex 2 III.G.1 | Issuer | Capital-adequacy dashboard fed by reserve valuations |
| Whitepaper and ongoing disclosures | Issuance Rulebook Sched. 1; Annex 2 Part II | Issuer | Disclosure CMS with version history and VARA submission log |
| Investor classification (Retail / Qualified / Institutional) | VARA circular 8 Jan 2026 | VASP | Classification workflow: AED 3.5M NW / AED 700k income evidence, SoW/SoF upload, suitability test, 1-week cooling-off, periodic review, 8-yr retention |
| Product gating (retail vs qualified-only) | VARA circular 8 Jan 2026 | VASP | Product eligibility rules tied to client class |
| KYC / CDD / UBO / PEP / TFS screening | FDL 10/2025; Cabinet Decision 134/2025; EOCN guidance | VASP / DNFBP | KYC/KYB with Emirates ID/UAE Pass, UBO capture, EOCN/UN list screening, goAML reporting |
| Travel rule (≥ AED 3,500 beneficiary verification; unhosted-wallet EDD) | VARA circular 24 Feb 2026 | VASP | Travel-rule messaging, daily aggregation per beneficiary, unhosted-wallet EDD queue, decline/return flow |
| Business-wide AML risk assessment | VARA guidance 12 Jun 2026 | VASP | Annual BWRA module |
| Payment-token settlement compliance | CBUAE PTSR; VARA NoC circular 24 Jul 2025 | VASP | Allowed-payment-token list (CBUAE-licensed AED tokens), NoC registration record |
| Marketing permissions and disclaimers | VARA Market Conduct Rulebook Part I; RERA Trakheesi | VASP / broker | Marketing approval queue; Trakheesi permit number per listing; risk-disclosure templates |
| Title-registry sync (DLD) | DLD tokenisation framework; Law 7/2006 | Issuer / DLD | Registry-sync monitor: on-chain vs DLD record reconciliation, break alerts |
| Foreign-ownership eligibility | Dubai freehold rules; KSA Non-Saudi Ownership System | Platform / issuer | Nationality × location rule engine; KSA holy-cities Muslim/Saudi-company check |
| Shariah compliance (if marketed as Islamic) | AAOIFI standards; Shariah board | Issuer | Shariah screening of tenants and leverage; fatwa repository; purification calc |
| CRS / FATCA / CARF | UAE CRS; CARF (consultation 2025) | Reporting FI / RCASP | Self-cert + TIN collection; annual XML reporting |
| VAT / DLD fees / KSA RETT | UAE VAT law; DLD fees; KSA RETT | SPV / platform | Fee and tax engine per transaction type |
| DFSA crypto-token suitability (if DIFC) | GEN 3A.2.1 (from 12 Jan 2026) | DIFC firm | Token suitability assessment record per asset |

---

## Sources (accessed 23 Sep 2026)

1. VARA, News & circulars list (2025–2026) — https://www.vara.ae/en/news/
2. VARA, "VARA Issues Updated Activity Rulebooks to Strengthen Market Integrity and Risk Oversight", 19 May 2025 — https://www.vara.ae/en/news/vara-issues-updated-activity-rulebooks-to-strengthen-market-integrity-and-risk-oversight/
3. VARA circular, "Onboarding and Classification of Qualified Investors" (Guidance on investor classifications v3), 8 Jan 2026 — https://media.umbraco.io/dwtc/nllk32pi/vasp-circular-guidance-on-investor-classifications-v3.pdf
4. VARA Rulebooks portal & rulebook index — https://rulebooks.vara.ae/ ; https://rulebooks.vara.ae/rulebook/rulebooks
5. VARA, Virtual Asset Issuance Rulebook v2.0 (PDF, 19 May 2025) — https://rulebooks.vara.ae/sites/default/files/en_net_file_store/VARA_EN_293_VER20250519.pdf
6. VARA, Issuance Rulebook Part D "Category 1 VA Issuance" — https://rulebooks.vara.ae/rulebook/d-category-1-va-issuance
7. VARA, Public Register (Prypco FZE VL/25/05/001; Ctrl Alt Solutions DMCC VL/25/05/002; Tokinvest DMCC VL/2024/12/004; Tribe Tokenisation FZE VL/26/06/002) — https://www.vara.ae/en/licenses-and-register/public-register/
8. Ctrl Alt, "Ctrl Alt and Dubai Land Department go live with tokenized real estate", 24 May 2025 — https://www.ctrl-alt.co/press-releases/ctrl-alt-dld
9. Ctrl Alt, "…Launch Phase Two of Real Estate Tokenization Project, Enabling Secondary Market Trading", 20 Feb 2026 — https://www.ctrl-alt.co/press-releases/ctrl-alt-dld-phase-two
10. Ctrl Alt, Press releases index (incl. Ripple partnership 16 Jul 2025) — https://www.ctrl-alt.co/press-releases
11. VARA circular, "Implementation of the UAE Virtual Assets Travel Rule Requirements", 24 Feb 2026 — https://media.umbraco.io/dwtc/hhebemw5/circular-to-vasp-travel-rule-ardocx.pdf
12. VARA circular, "Publication of UAE Federal Decree-Law on AML/CFT/CPF (2025) — Mandatory GAP Assessment for VASPs", 24 Nov 2025 — https://media.umbraco.io/dwtc/bmpneboh/vasp-circular-aml-federal-law-2025-v1.pdf
13. UAE Capital Market Authority (ex-SCA), homepage news incl. "The CMA Issues the Virtual Assets Framework…" (13 Apr 2026) — https://www.uaecma.gov.ae/en/home.aspx (article URL returned 404)
14. UAE Capital Market Authority, "New CMA Law" (FDL 32 & 33 of 2025; effective 1 Jan 2026) — https://www.uaecma.gov.ae/en/new-cma-law
15. DFSA, "Crypto" regime page (12 Jan 2026 changes) — https://www.dfsa.ae/crypto
16. DFSA, "Innovation" (Innovation Testing Licence) — https://www.dfsa.ae/innovation
17. DFSA, "Regulatory Regimes for Crowdfunding" (PDF) — https://365343652932-web-server-storage.s3.eu-west-2.amazonaws.com/files/1516/5780/2018/DFSA_Regulatory_Regimes_for_Crowdfunding.pdf
18. REGA, "Non-Saudi property ownership system enters into force as of today", 22 Jan 2026 — https://rega.gov.sa/en/media-center/news-announcements/rega-non-saudi-property-ownership-system-enters-into-force-as-of-today/
19. REGA, "Non-Saudi Real Estate Ownership in Saudi Arabia" platform page — https://rega.gov.sa/en/rega-services/platforms/non-saudi-real-estate-ownership-in-saudi-arabia/
20. Saudi CMA, FinTech ExPermit companies list — https://cma.gov.sa/en/Market/fintech/Pages/ExpFinTechs.aspx
21. Saudi CMA, Real Estate Contributions registry — https://cma.gov.sa/en/Market/RealestateContributions/Pages/default.aspx
22. QFC, Digital Assets Lab — https://www.qfc.qa/en/digital-ai-and-technology/digital-assets-lab/
23. QFC, Digital Assets Framework User Guide (25 Aug 2024) — https://www.qfc.qa/media/gromto2z/qfc_digital_assets_user_guide_25-aug-2024.pdf
24. VARA, Market Conduct Rulebook (table of contents, v19 May 2025) — https://rulebooks.vara.ae/rulebook/market-conduct-rulebook
25. Saudi Press Agency, "REGA Announces Completion of First Real Estate Tokenization, Establishes First Global Regulatory Standard…", 19 Nov 2025 — https://www.spa.gov.sa/en/N2448380 (added at QA, 23 Sep 2026)
26. Saudi Press Agency, "REGA Launches Regulatory Sandbox Featuring Real Estate Tokenization", 13 Feb 2026 — https://www.spa.gov.sa/en/N2511536 (added at QA, 23 Sep 2026)
27. PRYPCO, Mint product page (FAQ: "The minimum investment amount is AED 1,000") — https://prypco.com/mint (added at QA, 23 Sep 2026)
28. PRYPCO, Mint Terms & Conditions (§3.1 custody; §4.4 AED 2,000 minimum; §14.1 age 18) — https://prypco.com/mint/terms-and-conditions (added at QA, 23 Sep 2026)
