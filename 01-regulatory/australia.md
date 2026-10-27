# Australia — Regulatory Landscape for Tokenised Fractional Real Estate

> **Research, not legal advice.** Snapshot as of 23 Sep 2026; confirm with Australian financial-services, tax and property counsel.

**Evidence labels:** [C#n] confirmed by numbered source n · [I] inferred (reasoning, or well-established law not re-fetched this session) · [S] speculative.

**Research constraint.** The shared WebSearch quota was exhausted before this region, so research used WebFetch of ASIC, Treasury, AUSTRAC, FIRB, APH and legislation.gov.au pages. Several were 404, blocked or content-less:
- blocked: AUSTRAC sub-pages (robots.txt), APH bill search (403);
- 404: RBA Project Acacia landing page (the Final Report PDF itself is reachable, see source 14), ASIC CSF/DDO pages.

Statutory mechanics (s601ED, s708, s761G, CSF caps, duty rules) are therefore [I]. **The DAP/TCP legislation is now verified (QA pass, 23 Sep 2026):** the **Corporations Amendment (Digital Assets Framework) Act 2026 (No. 38, 2026)** received Royal Assent on **8 Apr 2026** and commences **8 Apr 2027** ("the day after the end of the period of 12 months beginning on the day this Act receives the Royal Assent"), with a **6-month transition** for existing operators (s1732) and a **low-value exemption** (≤$10m of transactions per 12 months across all relevant platforms and ≤$5,000 entry value per client, s911A(2)(ja)) [C#14][C#15].

**Bottom line.**
- ASIC's own worked example (INFO 225, **Example 9 "83Smith"**, a token representing a part interest in an apartment building) concludes the token is **"likely to be an interest in a managed investment scheme"** [C#1].
- A retail offering therefore normally needs:
  - a **registered MIS** run by a **Responsible Entity** holding an AFSL (public company, compliance plan, PDS, TMD under DDO); or
  - a wholesale-only unregistered scheme whose trustee holds an AFSL [C#1][C#6][I].
- Secondary trading on a multilateral venue needs an **Australian market licence** (or an exemption) and possibly a CS facility licence [I].
- **AML/CTF tranche 2 (1 Jul 2026)** brings real-estate professionals into AUSTRAC regulation [C#3].
- **FIRB's ban on foreign purchases of established dwellings now runs 1 Apr 2025 – 30 Jun 2029** [C#9]. This constrains foreign token holders if the structure is treated as foreign acquisition of residential land [I].
- **Digital Assets Framework Act 2026** (DAP and **Tokenised Custody Platform** licensing) is enacted and **commences 8 Apr 2027** [C#14][C#15]. The statutory TCP definition (s761GD) is built around an operator that "creates a single digital token" per underlying asset, held in trust or as bailee, with a right to redeem or direct delivery of that asset [C#15]. **A fractional property token (many units per property) may therefore sit outside the TCP definition and remain a plain MIS interest under INFO 225; whether custodial holding of such tokens is a DAP is a separate question. Needs counsel** [I].

---

## 1. Classification of a fractional real-estate token

| Structure | Likely classification | Test / basis |
|---|---|---|
| Token = unit/interest in a trust or pooled arrangement owning property, managed by the operator (leasing, maintenance) | **MIS interest** → financial product | Corporations Act s9 MIS definition: contributions pooled or used in a common enterprise to produce financial benefits; members lack day-to-day control [I]. **ASIC INFO 225 Example 9** ("Company H… fractionalised real estate investment… tokens known as '83Smith' representing a part interest in an apartment building") → "likely to be an interest in a managed investment scheme" [C#1] |
| Token = share in a company owning the property | **Security (share)** | INFO 225: digital assets may be shares or debentures if they grant ownership, voting or profit rights [C#1] |
| Token = debt claim secured on property | **Debenture / note** (RG 69 retail disclosure) [C#1][I] | Ch 2L / s9 [I] |
| Token referencing property value without ownership | **Derivative** [C#1] | s761D [I] |
| Direct co-ownership (tenants in common on title) with no management pooling | Possibly not an MIS. Practically unworkable for many holders on Torrens title [I] | — |
| Token-based "facility for financial investment" | Financial product (s763B) [C#1] | — |

- **ASIC stance.** INFO 225 (created 2017; major rewrite announced in **25-250MR on 29 Oct 2025**, which also granted a sector-wide **no-action position until 30 Jun 2026**, now lapsed; page **last modified 30 Apr 2026**, the current version) sets out the classification framework: MIS, securities, derivatives, facilities for financial investment, non-cash payment facilities [C#1][C#16]. The two dates cited across this pack (29 Oct 2025 in the platform files, 30 Apr 2026 here) are both correct: they are successive versions.
- **AFSL authorisations.** ASIC applies the ordinary categories (dealing, arranging, advice, custody, RE) according to the product, with no bespoke "crypto" authorisation [C#1][C#5].

## 2. Licence / exemption pathways for a new platform

### Licence pathway options for a new entrant

| Pathway | Scope | Limits / conditions | Partners required | Cost / time (indicative) |
|---|---|---|---|---|
| **Registered MIS + Responsible Entity (own AFSL)** | Retail offering of property-trust tokens | RE must be a public company with an AFSL authorising it to operate the scheme [C#1]. Compliance plan and (if board not majority external) compliance committee; constitution; PDS; TMD; financial requirements (RG 166). Custody of scheme property by RE or RG 133 custodian; **$10M NTA** if holding custody, reduced for incidental custody [C#1][I]. MIS registration needed if >20 members, promoted by a professional promoter, or retail PDS offered (s601ED) [I] | Custodian, auditor (scheme and compliance-plan audits), registry | AFSL 6–12+ months [S]; annual running cost high [S] |
| **Hire an RE ("RE-for-hire")** | Platform acts as investment manager / promoter; third-party RE issues the tokens | RE's policies apply | Professional RE + custodian | Faster (months) [I] |
| **Unregistered (wholesale) MIS** | Wholesale / sophisticated investors only | Trustee of an unregistered MIS needs an AFSL (INFO 251) [C#6]. No PDS; s708-style exemptions apply to disclosure [I] | Custodian optional (depends on AFSL conditions) | Lighter; still AFSL (or authorised-rep of a licensee) [I] |
| **Corporate Authorised Representative (CAR) of an AFSL holder** | Distribute or arrange under a partner licence | Partner's authorisations | Sponsoring AFSL | Weeks [I] |
| **Crowd-sourced funding (CSF)** | Offer shares in an eligible company via a CSF intermediary | $5M per 12 months; retail investor cap $10,000 per company per 12 months; 5-business-day cooling-off; eligible company assets/turnover < $25M [I]. **Companies whose principal business is investing in securities or interests in other entities/schemes are excluded**, so a property-holding investment company is likely ineligible [I] | AFSL holder with CSF intermediary authorisation (RG 262) [C#5] | [S] |
| **s708 / s1012D exemptions** | Sophisticated (≥$500k investment or accountant's certificate), professional, small-scale personal offers (20 investors / $2M per 12 months) | No retail PDS; still AFSL for dealing/arranging if in business [I] | Accountant certificate process | Low [I] |
| **Australian market licence** (s795B) or exemption | Secondary trading venue | Market-integrity obligations; ASIC oversight [I] | Possibly CS facility licence for settlement | 12+ months [S] |
| **Digital Asset Platform / Tokenised Custody Platform regime (enacted, not yet commenced)** | New financial products: DAP and **TCP** (Corporations Act Pt 7.1 Div 3, Sch 1 of the Act) | Exposure draft released **25 Sep 2025**, consultation closed **24 Oct 2025**, submissions published **2 Feb 2026** [C#4]. **Corporations Amendment (Digital Assets Framework) Act 2026, No. 38, 2026: Royal Assent 8 Apr 2026; whole Act commences 8 Apr 2027; 6-month transition for existing operators (s1732, i.e. to 8 Oct 2027); low-value exemption ≤$10m transactions/12 months and ≤$5,000 entry value per client (s911A(2)(ja))** [C#14][C#15]. It no longer appears in "bills before Parliament" because it is an Act [C#10][I]. **TCP = single token per asset held on trust (s761GD); fit for fractional property tokens is doubtful** [C#15][I] | AFSL with DAP/TCP authorisations (from 8 Apr 2027) [C#15][I] | Apply in the transition window Apr–Oct 2027 [I] |
| **ASIC enhanced regulatory sandbox** | Test some services without AFSL (up to 24 months) | Excludes many products; MIS operation generally not eligible [I] | — | — |
| **AUSTRAC registration** | VASP (ex-DCE) registration if exchanging or transferring virtual assets (e.g., stablecoins) [I] | Reformed VASP categories under the AML/CTF Amendment Act 2024 [I] | — | Months [I] |

## 3. Investor eligibility, suitability, limits

- **Retail vs wholesale** (s761G/761GA) [I]:
  - wholesale if investing ≥**$500,000**;
  - or accountant-certified **net assets ≥$2.5M** or **gross income ≥$250,000** for each of the last two years;
  - or professional investor;
  - or the "sophisticated investor" test (s761GA) with licensee satisfaction.
- **s708 sophisticated/professional exemptions** for disclosure; accountant's certificates valid 2 years [I].
- **Wholesale-test reform.** Treasury and parliamentary reviews (2023–25) considered raising the thresholds; the PJC's May 2025 report recommended **no immediate increase** to the $500k product-value and $2.5m/$250k wealth tests and an objective sophisticated-investor test [C#18]. **No enacted change confirmed** as of Sep 2026 [I][S].
- **DDO (Pt 7.8A).** The issuer must make a **Target Market Determination**. Distributors take reasonable steps and report **significant dealings outside the TMD within 10 business days**, with review triggers [I]. DDO applies to retail MIS interests and securities [I].
- **Personal advice** triggers the best-interests duty (s961B). General advice needs a warning [I].
- **CSF.** $10k retail cap per issuer per 12 months [I].

## 4. KYC / AML / CFT, sanctions, travel rule, beneficial ownership

| Topic | Rule | Status |
|---|---|---|
| AML/CTF reform | **AML/CTF Amendment Act 2024** [I]. **Tranche 2** (incl. **real-estate professionals**, lawyers, accountants, TCSPs) regulated **from 1 Jul 2026** [C#3] | Real-estate designated services include brokering the sale, purchase or transfer of real estate [I]. A platform acting as agent in property acquisitions or disposals may be caught [I] |
| Existing designated services | Issuing, dealing in or arranging securities and MIS interests are designated services; AFSL holders need an AML/CTF program, KYC (ACIP), SMRs, TTRs ($10k cash), IFTIs [I] | Reformed obligations for existing reporting entities and VASPs commence **31 Mar 2026** [I] (AUSTRAC page last updated 15 Dec 2025 [C#3]) |
| Virtual assets / travel rule | Reforms extend travel-rule obligations to virtual-asset transfers and replace "DCE" with broader VASP services [I] | Applies to stablecoin legs [I] |
| Sanctions | DFAT Consolidated List (Autonomous Sanctions Act; Charter of UN Act) — strict liability for some offences [I] | Screen investors and wallets [I] |
| Beneficial ownership | CDD must identify beneficial owners (≥25% / control) [I]. Public BO register for unlisted companies proposed, not enacted [S] | — |

## 5. Custody and client money

- **Scheme property** of a registered MIS must be held by the RE or an RG 133-compliant custodian [C#7][I].
- **NTA.** ASIC's NTA expectations for custody are **$10M** standard, **$150k** where custody is incidental, and nil for incidental non-physical custody [C#1].
- **Omnibus holding** of digital-asset financial products is permitted. ASIC extended the omnibus-account relief (**ASIC Corporations (Amendment) Instrument 2025/871**, which amends ASIC Instrument 2024/17; CS 32 outcome **9 Dec 2025**) subject to record-keeping and reconciliation [C#8].
- **Stablecoins.** Distributors of eligible stablecoins and wrapped tokens get class relief from AFSL, market and CS-facility licence requirements under **ASIC Instrument 2025/867** (in force; registered 8 Dec 2025) [C#1][C#2][C#8].
- **Client money.** Hold in a trust account with an ADI (Div 2 Pt 7.8) [I].
- **Self-custody.** Legally possible for tokens evidencing MIS interests. The RE must still keep the **register of members** (s169), and the on-chain record must meet register requirements. Transfer restrictions are enforced via allow-lists [I].

## 6. Secondary trading

- Operating a facility where offers to buy and sell financial products are regularly made or accepted is **operating a financial market**. That needs an **Australian market licence** (or exemption) under s791A [I].
- Bilateral matched deals by an AFSL holder or a bulletin board may fall outside, depending on design [I].
- **Settlement** may require a **CS facility licence** [I]. Instrument 2025/867 shows ASIC is willing to grant class relief for some digital assets, but not for MIS tokens [C#1][I].
- **Liquidity (registered MIS).** Withdrawal rights apply only if the scheme is "liquid" (Pt 5C.6). Illiquid property schemes use periodic withdrawal offers, so a token secondary market is the main liquidity path [I].
- **DAP regime (enacted; commences 8 Apr 2027)** regulates platforms that hold digital assets for clients (trustee, bailee or instruction-based holding) [C#15]. Its treatment of trading venues for tokenised real-world assets, and whether a token-holding secondary venue is a DAP in addition to (or instead of) a financial market, is not settled [I]. The Act also carves out "public digital token infrastructure" (permissionless protocols), which is the hook Tokeniser relies on for its "exempt public protocol" AMM theory (see `02-platforms/australia/03-tokeniser-liquidise.md` §2.3) [C#15][I].

## 7. Disclosure, ongoing reporting, marketing

- **PDS** (Pt 7.9) for retail MIS interests; prospectus (Ch 6D) for securities; RG 228 [C#1][I].
- **Disclosing-entity continuous disclosure** for registered schemes with ≥100 members [I]. Annual financial reports and audits; compliance-plan audit [I]. Periodic statements [I].
- **DDO.** TMD, distribution conditions, review triggers, complaints data reporting [I].
- **Advertising.** RG 234 (**updated 9 Jun 2026**) addresses false or misleading statements [C#11]. Yield claims, "guaranteed" language and finfluencer promotion attract enforcement [I]. INFO 269 covers discussing financial products online [C#1].
- **Real-estate advertising law** (state fair trading) also applies to property descriptions [I].

## 8. Tax reporting

| Item | Rule | Platform action |
|---|---|---|
| **AMIT** (attribution managed investment trust) | A property trust electing AMIT issues **AMMA statements**; attribution to members [I] | AMMA generation per holder per year |
| **MIT withholding** | Fund payments to foreign residents: **15%** (information-exchange countries) or **30%** [I] | Residency capture, withholding engine |
| **TFN withholding** | 47% if no TFN/ABN quoted [I] | TFN capture and validation |
| **CRS / FATCA** | Australia participates; the RE/custodian is a Reporting FI [I] | Self-cert, annual ATO report |
| **CARF** | Australia committed to CARF; implementing legislation status unverified [S] | — |
| **CGT** | Token disposals are CGT events for holders; foreign residents taxed on taxable Australian property (≥10% indirect interest in land-rich entity) [I] | Cost-base records; **15% foreign resident capital gains withholding** with no value threshold for contracts from 1 Jan 2025 (rate was 12.5% above $750k before that date) applies to direct property sales [I] |

> **QA note (Fable):** the FRCGW figure was changed from 12.5% to 15%: the Treasury Laws Amendment (2024 Tax and Other Measures No. 1) Act 2024 raised the rate to 15% and removed the $750,000 threshold for contracts from 1 Jan 2025. The ATO page could not be fetched this session (404), so re-verify with counsel.
| **Stamp / landholder duty** | State **landholder duty** when acquiring interests above thresholds in land-rich trusts or companies (e.g., 50% for private companies; **20% for private unit trusts in NSW**; aggregation of associated persons) [I]. **Foreign purchaser surcharge duty and land-tax surcharges** apply when a trust is "foreign" (e.g., ≥50% foreign-held in NSW) [I] | **Holder-concentration and foreign-holding % monitor per trust**; block transfers that cross thresholds [I] |
| **GST** | Residential rent input-taxed; commercial rent taxable [I] | — |

## 9. Land registry, title linkage, foreign ownership

- **Torrens title** state registries; e-conveyancing via **PEXA** (and competitors under the ARNECC model). The **RE/custodian** is registered proprietor; token holders are beneficial interest holders in the trust, not on title [I]. No Australian registry records token holders [I].
- **FIRB.**
  - Foreign persons must notify **residential land acquisitions regardless of value** [C#9].
  - **Temporary ban on buying established dwellings: 1 Apr 2025 – 30 Jun 2029.** Exceptions: redevelopment adding ≥20 dwellings, commercial-scale housing (BTR, student, aged care), PALM employers [C#9]. The ban was originally announced (Feb 2025) as running to 31 Mar 2027 [I: the FIRB news item of 17 Feb 2025 gives the start date only; the 31 Mar 2027 end date is from the Treasurer's announcement, not re-fetched] and **has been extended to 30 Jun 2029** (GN6 v5, 1 Jul 2026; residential-land page updated 1 Jul 2026) [C#9].
  - Register of Foreign Ownership; **vacancy fee** if not occupied or available for rent 183 days/yr [C#9].
  - Indirect interests via land-rich trusts or companies can require FIRB approval (thresholds per FATA) [I]. GN6 v5 does not address fractional or tokenised interests [C#9].
  - **Admin control:** foreign-person flag and cap per residential asset.
- **Enforcement posture.** Federal Court imposed **$14M** penalties (30 Jan 2026) for failing to comply with a Treasurer disposal order (critical minerals, not real estate) [C#13]. FIRB fee payments by card ended 19 Sep 2026 [C#12].

## 10. 2025–2026 developments (status on 23 Sep 2026)

| Date | Development | Status |
|---|---|---|
| 1 Apr 2025 | FIRB established-dwelling ban starts [C#12] | Now extended to **30 Jun 2029** [C#9] |
| 25 Sep – 24 Oct 2025 | **Treasury DAP/TCP exposure draft** consultation [C#4] | Submissions published 2 Feb 2026 [C#4]; enacted as the Act below |
| 29 Oct 2025 | **ASIC 25-250MR: INFO 225 rewritten** (tokenised securities, stablecoins, wrapped tokens, wallets can be financial products); sector-wide **no-action position to 30 Jun 2026** [C#16]; ASIC CS 32 (stablecoin/wrapped-token relief; omnibus custody extension) [C#8] | INFO 225 later revised again (30 Apr 2026); **no-action position has lapsed** |
| 8–9 Dec 2025 | **ASIC Instruments 2025/867 and 2025/871** made [C#2][C#8] | In force |
| 31 Mar 2026 | AML/CTF reforms commence for existing reporting entities and VASPs [I] | In force (verify) |
| **8 Apr 2026** | **Corporations Amendment (Digital Assets Framework) Act 2026 (No. 38) — Royal Assent** [C#14][C#15] | Enacted; **commences 8 Apr 2027**; 6-month transition (to 8 Oct 2027); low-value exemption $10m / $5k |
| 30 Apr 2026 | **INFO 225 updated** (incl. Example 9 tokenised real estate → MIS) [C#1] | Current |
| May 2026 | **Project Acacia Final Report** (RBA/DFCRC): 20 use cases (12 pilots, 8 PoCs), A$4.4m pilot wCBDC issued; ASIC opt-in exemptions from AFS, market and CS-facility licensing and AUSTRAC AML/CTF exemptions for participants; open issues named: enforceable on-chain ownership records and legal settlement finality; 11 follow-on initiatives incl. a **DFMI sandbox** and a tokenised government bond initiative [C#17] | Published (report released 19 May 2026) |
| 9 Jun 2026 | RG 234 (advertising) updated [C#11] | Current |
| **1 Jul 2026** | **AML/CTF tranche 2** (real estate, legal, accounting) [C#3]; FIRB GN6 v5 and residential-land page updated with the ban extension to 30 Jun 2029 [C#9] | In force |
| Sep–Oct 2026 | Treasury consultations open incl. CSLR and **post-market product regulation** [C#10b] | Open |
| — | Wholesale investor test reform | No enacted change found [S]; PJC (May 2025) recommended keeping the thresholds [C#18] |

## 11. Enforcement / litigation relevant to fractional or tokenised property

- No ASIC enforcement specific to tokenised real estate was confirmed this session [I].
- **Relevant patterns** [I]:
  - ASIC action against **unregistered MIS** property syndicates, a long-standing theme;
  - **DDO** actions against digital-asset firms (e.g., Bit Trade/Kraken margin product, 2024 penalty);
  - unlicensed financial-product cases against crypto firms (e.g., BPS Financial/Qoin; Block Earner litigation).
  - Details not re-verified [I].
- **Lesson for design:** treat the product as an MIS from day one (AFSL/RE or partner RE, PDS or wholesale gating, TMD) rather than argue it is a "digital asset" [C#1][I].

---

## Compliance obligations checklist

| Obligation | Legal basis | Who's responsible | Platform control needed |
|---|---|---|---|
| Classify product (MIS / security / debenture) and document | Corporations Act s9, s763A–764A; INFO 225 | Issuer / RE | Product classification record with legal opinion |
| AFSL with correct authorisations (RE, dealing, arranging, custody, advice) or CAR appointment | s911A; RG 1; INFO 251 | Platform / RE | Licence and authorisation matrix; CAR register; feature gating by authorisation |
| MIS registration, constitution, compliance plan | s601ED, 601GA, 601HA | RE | Scheme register, compliance-plan task calendar, compliance-committee minutes |
| PDS / prospectus and in-use notices | Pt 7.9; Ch 6D | Issuer | Document library with versioning; PDS acknowledgement at checkout |
| Target Market Determination and distribution conditions | Pt 7.8A (DDO) | Issuer + distributor | TMD builder; investor-eligibility questionnaire mapped to TMD; outside-TMD flag and 10-business-day ASIC report queue; complaints data capture |
| Retail vs wholesale / sophisticated classification | s761G, 761GA, 708 | Licensee | Wholesale-certificate upload with 2-year expiry, $500k-ticket check, sophisticated-investor assessment record |
| CSF caps (if using CSF) | Pt 6D.3A | CSF intermediary | $10k retail cap tracker per issuer; 5-day cooling-off timer |
| AML/CTF program, KYC, SMR, TTR, IFTI | AML/CTF Act 2006 as amended 2024 | Reporting entity (licensee; tranche 2 real-estate agent) | KYC/KYB, BO capture, risk rating, SMR case queue (24h/3-day clocks), TTR/IFTI generation |
| Travel rule for VA transfers | AML/CTF reforms (2026) | VASP | Travel-rule messaging for stablecoin/token transfers |
| Sanctions screening | Autonomous Sanctions Act 2011 | All | DFAT list screening of investors and wallets |
| Custody of scheme property and tokens; NTA | RG 133; RG 166; Instrument 2025/871 | RE / custodian | Custodian integration, omnibus reconciliation, NTA monitor |
| Register of members / on-chain register | s169; s601HA | RE / registry | Golden-record register synced to chain; transfer approval workflow |
| Secondary market licensing | s791A (market licence); s820A (CS facility) | Venue operator | Venue mode switch (bulletin board vs licensed market partner); trade surveillance |
| Advertising compliance | s1041H; RG 234 (Jun 2026) | Issuer / licensee | Marketing approval queue; yield-claim checker; finfluencer register |
| AMIT / MIT withholding / TFN / CRS / FATCA | ITAA 1997 Div 276; TAA Sch 1 Subdiv 12-H; CRS | RE (trustee) | AMMA statements; residency and withholding engine; TFN capture; CRS/FATCA reporting |
| Landholder duty and foreign surcharge thresholds | State duties and land-tax acts | RE / trustee | Per-trust holder-concentration and foreign-holding % monitor; transfer blocks at thresholds |
| FIRB approval / established-dwelling ban | FATA 1975; GN6 v5 | Foreign investor / RE | Foreign-person flag; residential-asset foreign-cap; FIRB approval evidence store |
| Register of Foreign Ownership / vacancy fee | FATA; Register of Foreign Ownership of Australian Assets Act | Foreign owner | Occupancy/leasing evidence tracking (183 days) |
| DAP / TCP classification and, if caught, AFSL authorisation (from 8 Apr 2027; apply within the 6-month transition) | Corporations Amendment (Digital Assets Framework) Act 2026, Sch 1 (s761GC–GD; s911A(2)(ja) low-value exemption) [C#14][C#15] | Platform / custodian | Legal opinion on whether the token structure is a TCP (single token per asset) or the custody model is a DAP; transaction-value and per-client entry-value monitors against the $10m / $5k exemption thresholds; ASIC notice if relying on the exemption |

---

## Sources (accessed 23 Sep 2026)

1. ASIC, INFO 225 "Digital assets: Financial products and services" (last modified 30 Apr 2026) — https://asic.gov.au/regulatory-resources/digital-transformation/digital-assets-financial-products-and-services/
2. Federal Register of Legislation, ASIC Corporations (Stablecoin and Wrapped Token Relief) Instrument 2025/867 (F2025L01512) — https://www.legislation.gov.au/F2025L01512/latest/text
3. AUSTRAC, "AML/CTF reform" (last updated 15 Dec 2025) — https://www.austrac.gov.au/amlctf-reform
4. Treasury, "Regulating digital asset platforms" consultation (c2025-701519) — https://consult.treasury.gov.au/c2025-701519
5. ASIC, RG 262 Crowd-sourced funding: Guide for intermediaries (landing page) — https://www.asic.gov.au/regulatory-resources/find-a-document/regulatory-guides/rg-262-crowd-sourced-funding-guide-for-intermediaries
6. ASIC, INFO 251 "AFS licensing requirement for trustees of unregistered managed investment schemes" (URL via INFO 225) — https://www.asic.gov.au/for-finance-professionals/afs-licensees/applying-for-and-managing-an-afs-licence/licensing-certain-service-providers/afs-licensing-requirement-for-trustees-of-unregistered-managed-investment-schemes
7. ASIC, "Managed investment schemes" — https://www.asic.gov.au/regulatory-resources/managed-funds/managed-investment-schemes
8. ASIC, CS 32 "Proposed relief for certain stablecoins and wrapped tokens and extension of omnibus accounts for digital asset custody" (29 Oct 2025; outcome 9 Dec 2025) — https://www.asic.gov.au/regulatory-resources/find-a-document/consultations/cs-32-proposed-relief-for-certain-stablecoins-and-wrapped-tokens-and-extension-of-omnibus-accounts-for-digital-asset-custody
9. Foreign Investment Australia, "Residential land" guidance and Guidance Note 6 v5 (2026) — https://foreigninvestment.gov.au/guidance/types-investments/residential-land ; https://foreigninvestment.gov.au/sites/foreigninvestment.gov.au/files/2026-06/guidance-note-6-residential-land-v5.pdf
10. Parliament of Australia, "Bills before Parliament" (snapshot 23 Sep 2026) — https://www.aph.gov.au/Parliamentary_Business/Bills_Legislation/Bills_before_Parliament ; 10b. Treasury, Consultations list — https://treasury.gov.au/consultation
11. ASIC, RG 234 Advertising financial products and services (updated 9 Jun 2026) — https://www.asic.gov.au/regulatory-resources/find-a-document/regulatory-guides/rg-234-advertising-financial-products-and-services-including-credit
12. Foreign Investment Australia, News (incl. "Changes to foreign purchases of established dwellings", 17 Feb 2025; fee payment changes 11 Sep 2026) — https://foreigninvestment.gov.au/news-and-reports/news
13. Foreign Investment Australia, "Penalties ordered for foreign investors' non-compliance", 6 Feb 2026 — https://foreigninvestment.gov.au/news-and-reports/news/penalties-ordered-foreign-investors-non-compliance
14. Federal Register of Legislation, Corporations Amendment (Digital Assets Framework) Act 2026 (No. 38, 2026; C2026A00038), landing page showing Royal Assent 8 Apr 2026 and Schedule 1 "Digital asset platforms and tokenised custody platforms" — https://www.legislation.gov.au/C2026A00038/latest/text *(added in QA, 23 Sep 2026)*
15. Federal Register of Legislation, same Act, as-made text (s2 commencement table; s761GC–GD DAP/TCP definitions; s911A(2)(ja) low-value exemption; s1732 transition) — https://www.legislation.gov.au/C2026A00038/asmade/2026-04-08/text/original/epub/OEBPS/document_1/document_1.html *(added in QA)*
16. ASIC, 25-250MR "Updated ASIC guidance supports digital asset innovation and boosts investor protection" (29 Oct 2025) — https://www.asic.gov.au/about-asic/news-centre/find-a-media-release/2025-releases/25-250mr-updated-asic-guidance-supports-digital-asset-innovation-and-boosts-investor-protection *(added in QA)*
17. RBA/DFCRC, "Project Acacia – Exploring the role of digital money in wholesale tokenised asset markets: Final Report" (May 2026) — https://www.rba.gov.au/payments-and-infrastructure/tokenised-money/pdf/project-acacia-final-report.pdf *(added in QA; the landing page 404'd but the PDF is reachable)*
18. Gilbert + Tobin, "Wholesale client tests remain the same" (22 May 2025) — https://www.gtlaw.com.au/insights/wholesale-client-tests-remain-the-same *(added in QA)*
