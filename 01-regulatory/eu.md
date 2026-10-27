# EU / EEA — Regulatory Landscape for Tokenised Fractional Real Estate

> **Research, not legal advice.** Snapshot as of 23 Sep 2026; confirm with counsel in the target Member State(s).

**Evidence labels:** [C#n] confirmed by numbered source n · [I] inferred (reasoning, or well-established law not re-fetched this session) · [S] speculative.

**Research constraint.** Partway through this region the session's shared WebSearch quota (200) ran out. EU findings rely on WebFetch of primary sources (EUR-Lex, ESMA, European Commission, BOE). National-regime details for Germany, France, Luxembourg and Liechtenstein could not be re-verified online and are marked [I].

**Bottom line.** A token giving profit or ownership rights in a property-holding company is, in almost every Member State, a **transferable security** (or a unit in an AIF) under MiFID II. It is therefore **excluded from MiCA** [C#2]. A new platform chooses between:
- **ECSPR crowdfunding licence**: €5M per project owner per 12 months, passportable, bulletin board only [C#10];
- **MiFID II investment-firm licence** (reception and transmission of orders, placement, MTF) plus a Prospectus-Regulation exemption, or a prospectus above **€12M** (harmonised threshold applicable from 5 Jun 2026; Member State option €5M) [C#9];
- national DLT-securities regimes (Germany eWpG, Spain Ley 6/2023, Luxembourg, France DEEP, Liechtenstein TVTG);
- for secondary settlement at scale, the **DLT Pilot Regime**. The Commission proposed in Dec 2025 to make it permanent and lift the cap to €100bn [C#7].

MiCA's CASP grandfathering ended on **1 July 2026** at the latest [C#2]. That matters only if RealX also deals in stablecoins or other MiCA crypto-assets.

---

## 1. Classification of a fractional real-estate token

| Token design | Likely EU classification | Test |
|---|---|---|
| Share / profit-participating note / bond of a property SPV, freely negotiable | **Transferable security** (MiFID II Art 4(1)(44)) → financial instrument → **outside MiCA** (MiCA Art 2(4)) [C#2] | ESMA guidelines (17 Dec 2024), three cumulative tests: (1) not a payment instrument; (2) belongs to a class (same issuer, same rights); (3) negotiable on the capital market, where "the abstract possibility of being transferred or traded should be deemed sufficient" [C#3]. Technology-neutral: "the technological format… should not be considered a determining factor" [C#3] |
| Token over a pooled multi-property vehicle with a defined investment policy | **Unit in a collective investment undertaking** → AIF under AIFMD (and a financial instrument) | ESMA guidelines: pooling + defined investment policy + return for investors [C#3]; AIFMD "defined investment policy" [I] |
| Token whose transfer is contractually blocked (non-negotiable) | May fall outside "transferable security". Could be a national "investment product" (e.g., German *Vermögensanlage*) or, if not a financial instrument, a **MiCA "other crypto-asset"** [I] | MiCA substance-over-form: "the features of the crypto-asset… determine the classification and not its designation by the issuer" [C#2] |
| Unique NFT of a whole property/deed | Potentially **outside MiCA** as "unique and not fungible" [C#2]; fractionalised NFTs are not unique [I] | MiCA Art 2(3) and recitals [C#2] |
| Shares of a private limited company offered via crowdfunding | **"Admitted instrument for crowdfunding purposes"** under ECSPR, if freely transferable [C#10] | ECSPR Art 2(1)(n) [C#10] |

- **Hybrid tokens.** Financial-instrument features "take precedence" [C#3].
- **National overlays.**
  - Spain: Ley 6/2023 expressly treats instruments "issued, registered, transferred or stored using distributed ledger technology" as financial instruments [C#15].
  - Germany: the eWpG treats electronic/crypto securities as securities (bearer bonds, and since 2023 certain shares; fund units via the KAG regulation) [I].
  - Liechtenstein: the TVTG "Token Container Model" allows a token to represent any right, including rights in real property [I][C#19 title only].

## 2. Licence / exemption pathways for a new platform

### Licence pathway options for a new entrant

| Pathway | Scope / what it allows | Caps and limits | Investor access | Secondary trading | Partners typically needed | Cost / time (indicative) |
|---|---|---|---|---|---|---|
| **ECSP licence (ECSPR 2020/1503)** | Placement without firm commitment and RTO for transferable securities and "admitted instruments" of project owners; **EU passport** [C#10] | **€5,000,000 per project owner per 12 months** [C#10] | Retail allowed, with non-sophisticated protections (knowledge test, loss simulation, warnings above €1,000 / 5% of net worth, 4-day reflection) [C#10] | **Bulletin board only**. Multilateral matching needs a separate MiFID II licence [C#10] | Qualified custodian for securities (MiFID/CRD-authorised); PSD2/EMD payment provider [C#10] | NCA authorisation ~6–12 months [S]; capital €25k or ¼ fixed overheads [I] |
| **MiFID II investment firm** (RTO, placing, advice; optionally MTF/OTF operation) | Full passport; can run a (DLT) MTF [I] | No raise cap on the firm; each issuance needs a prospectus or exemption | Retail with MiFID suitability/appropriateness | MTF/OTF permitted with the relevant licence | Custodian (or own safekeeping permission); CSD or DLT-Pilot infrastructure; payment partner | 9–18 months [S]; initial capital €75k–€750k depending on services [I] |
| **Tied agent of a MiFID firm ("licence-as-a-service")** | Market under a partner's licence | Partner's scope | As partner | Partner's venue | Partner firm | Weeks–months [I] |
| **Prospectus exemption / Listing Act** | Offer to the public without a prospectus | **Below €12M per issuer over 12 months** (Member State option €5M), applicable from **5 June 2026** [C#9]. Before that date national thresholds applied (e.g., Spain €8M under Ley 6/2023 Art 35; Germany €8M) [I]. Which option each Member State applies from June 2026 was not verified this session [I]. Also: qualified investors only; <150 persons per Member State; min. denomination/ticket ≥€100k [I] | Depends on exemption | — | — | National mini-docs may apply below the threshold (e.g., German WIB) [I] |
| **Full prospectus / EU Growth issuance doc / EU follow-on** | Public offer above the threshold; passportable | None | Retail | — | NCA approval | €100k+ and 2–4 months [S] |
| **AIFM (full or sub-threshold registered)** | If the vehicle is an AIF | Sub-threshold: <€100M (leveraged) / <€500M (unleveraged, 5-yr lock) [I] | Retail marketing of AIFs is subject to national rules or ELTIF [I] | Via MiFID venue | Depositary (full AIFM) | AIFMD II (Dir. 2024/927, in force 15 Apr 2024 [C#14]); transposition deadline 16 Apr 2026 [I] |
| **ELTIF 2.0** | Retail-accessible closed-ended fund investing in real assets | — | Retail with suitability | Matching allowed under ELTIF 2.0 [I] | AIFM + depositary | Heavy [I] |
| **DLT Pilot Regime** (DLT MTF / DLT SS / DLT TSS) | On-chain trading and settlement with exemptions from MiFID/CSDR [C#4] | Current: shares issuer <€500M market cap; bonds <€1bn; UCITS <€500M; **€6bn aggregate** [C#4] | Via members | Yes | Operator must be an investment firm, market operator or CSD (proposal adds MiCA trading-platform CASPs) [C#7] | Six infrastructures authorised by Jan 2026 (CZ, DE×2, LT, FR, ES) [C#5] |
| **National DLT-securities regimes** | Register issuance on-chain with legal effect (see below) | National | National | National | Registered register-keeper / DLT registrar | Varies |
| **MiCA CASP** (only if dealing in MiCA crypto-assets, e.g., EMT stablecoins) | Custody, exchange, transfer of crypto-assets | — | — | — | — | Grandfathering ended ≤1 Jul 2026 [C#2] |

**National DLT-securities regimes (summary).**

| Country | Instrument | Key points |
|---|---|---|
| **Germany – eWpG (2021)**, extended by the Zukunftsfinanzierungsgesetz (2023) | Crypto securities in a *Kryptowertpapierregister* | Registrar needs a BaFin licence (crypto securities register keeping, KWG). Covers bearer bonds; registered shares added in 2023; crypto fund units via KAGB regulation. Popular for real-estate crypto bonds under a prospectus exemption plus WIB [I]. BaFin pages could not be fetched (404) |
| **France** | DEEP (*dispositif d'enregistrement électronique partagé*) for unlisted securities (Ordonnance 2017-1674; Décret 2018-1226) | Also a DLT Pilot TSS (LISE, ACPR permission 13 Oct 2025) [C#5]. SCPI/SCI shares commonly tokenised via DEEP [I] |
| **Luxembourg Blockchain Laws I–IV** (2019, 2021, 2023, 2024) | Securities accounts and issuance on DLT; collateral on DLT | Blockchain IV introduced the "control agent" for DLT-issued dematerialised securities [I] |
| **Spain – Ley 6/2023** | Securities "represented by DLT" as a third form of representation alongside book entries and physical titles (Art 6.1) | The issuer appoints one or more entities "responsible for administration of the inscription and registry" (Art 8.4), authorised under Art 126(a). DLT must guarantee "integrity and immutability", identify holders and give holders access (Art 6.5); CNMV supervises (Art 8.5) [C#15]. CNMV sandbox under Ley 7/2020 (page 403) [I]. Securitize Europe DLT TSS authorised by CNMV 26 Nov 2025 [C#5] |
| **Liechtenstein – TVTG (2019)** (EEA) | Token Container Model | Registration with FMA for TT service providers (token issuer, token generator, TT depositary, etc.). Token issuers above ~CHF 5M must register and publish "basic information" [I]. Remains for non-MiCA tokens after MiCA [I] |

## 3. Investor eligibility, suitability and limits

- **MiFID II client categories.**
  - Retail.
  - Professional per se: Annex II.I.
  - Elective professional (Annex II.II): two of three tests — (a) 10 significant transactions per quarter over the last 4 quarters; (b) portfolio >€500k; (c) ≥1 year of relevant professional experience.
  - Eligible counterparty [I].
- **Suitability (Art 25(2))** for advice or portfolio management; **appropriateness (Art 25(3))** for non-advised complex products. Unlisted SPV shares are generally complex, so the appropriateness test is needed [I].
- **ECSPR.** Sophisticated vs non-sophisticated investors (Annex II criteria, linked to MiFID professional criteria) [C#10]. For non-sophisticated investors:
  - **entry knowledge test** (Art 21);
  - simulation of ability to bear loss;
  - **explicit acknowledgment for investments above €1,000 or 5% of net worth**;
  - **4-calendar-day pre-contractual reflection period** (Art 22) [C#10].
- **Product governance.** MiFID target market; PRIIPs **KID** required for retail if the product is "packaged" (e.g., notes or fund units; plain shares are exempt) [I].
- **Retail Investment Strategy.** Status in Sep 2026 not verified this session [S].
- **Prospectus thresholds.** Retail offers above €12M need a prospectus [C#9].

## 4. KYC / AML / CFT, sanctions, travel rule, beneficial ownership

- **Current law.** AMLD4/5 as transposed. Investment firms, ECSPs (which are investment-service-like; Member States generally treat them as obliged entities) [I] and CASPs are obliged entities: CDD, UBO ≥25%, PEP screening, record keeping for 5 years [I].
- **New single rulebook.**
  - AMLA gained legal existence on 26 Jun 2024 and started operations in summer 2025 [C#11].
  - AMLA will select **40 obliged entities for direct supervision in 2027**, and direct supervision begins in **2028** [C#11].
  - The **AMLR** (Reg. 2024/1624) applies from **10 Jul 2027**: harmonised CDD, a €10,000 cap on cash payments, a ban on anonymous crypto accounts [I].
- **Travel rule (TFR, Reg. 2023/1113).**
  - Applies with MiCA from **30 Dec 2024**.
  - Every crypto-asset transfer must carry originator and beneficiary name, DLT address, account number, and address/ID/DOB and LEI where available (Art 14).
  - **No de minimis threshold** (recital 30: crypto-asset transfers are subject to the same requirements "regardless of their amount") [C#13].
  - For **self-hosted addresses above €1,000**, the CASP must "take adequate measures to assess whether that address is owned or controlled by" the client — Art 14(5) (originator side) and Art 16(2) (beneficiary side) [C#13].
  - The TFR covers MiCA "crypto-assets", so **tokenised securities are not "crypto-assets"**. Their on-chain transfers are not strictly TFR crypto transfers, but the fiat or EMT legs are [C#13][I]. **Best practice:** apply travel-rule data to all wallet movements anyway [I].
- **Sanctions.** EU restrictive measures (Russia/Belarus packages restrict crypto and securities services to Russian nationals and residents), plus national lists [I]. Screen wallets as well as persons [I].
- **UBO registers.** National central registers. After *WM and Sovim* (CJEU, 22 Nov 2022), access is limited to legitimate interest; the AMLD6 framework carries this forward [I].

## 5. Custody and client money

- **ECSPR Art 10.** Transferable securities and admitted instruments must be safekept by a **qualified custodian** (CRD/MiFID-authorised). Payment services require a PSD2/EMD licence or partner [C#10].
- **MiFID II.** Client financial instruments and funds must be segregated (Art 16(8)–(10), MiFID Delegated Directive 2017/593). "Safekeeping and administration" is an ancillary service requiring authorisation [I].
- **National DLT registers.** Under the eWpG and Ley 6/2023, investors can hold directly on the register, and the register-keeper/registrar maintains the authoritative record [C#15][I]. This enables **self-custody** of DLT-native securities where national law permits [I].
- **AIFs.** A depositary is required for full AIFMs [I].
- **MiCA CASP custody** (for any stablecoins held): Art 75 segregation and liability [I].
- **DLT Pilot participants.** The DLT SS/TSS operator can admit natural persons as participants directly [I].

## 6. Secondary trading

- **Multilateral matching** of financial instruments = MTF/OTF/RM, which needs a MiFID licence. Operating without one is a breach [I].
- **ECSP bulletin boards.** Allowed for instruments originally offered on the platform; the ECSP may not match orders multilaterally without MiFID authorisation [C#10]. Bulletin-board users must be informed and get price-reference info [I].
- **DLT Pilot Regime.** A DLT MTF/TSS can trade tokenised shares or bonds with exemptions (e.g., book-entry and intermediary requirements) [C#4].
  - **Proposed reform (COM(2025) 943, 4 Dec 2025)** [C#7]:
    - aggregate cap raised to **€100bn**;
    - product-specific thresholds removed;
    - time limits on permissions removed, making it effectively permanent;
    - simplified regime for operators under €10bn;
    - MiCA trading-platform CASPs become eligible operators;
    - new "DLT notary" and "DLT central maintenance" service authorisations.
  - It is still in co-decision as of Sep 2026 [I].
- **P2P transfers** between whitelisted wallets are permissible where the register or law allows. Transfer restrictions (KYC'd holders, jurisdiction blocks) are implemented in the token or register logic [I].
- **Lock-ups.** No EU-wide statutory lock-ups for private placements. Selling restrictions come from prospectus exemptions (e.g., qualified-investor-only resale) and from contract [I].

## 7. Disclosure, ongoing reporting and marketing

- **Offering documents.**
  - Above €12M: prospectus or EU Growth issuance doc [C#9].
  - Below the threshold: national mini-docs (e.g., German WIB) [I].
  - ECSPR: **Key Investment Information Sheet** per offer (Art 23) [C#10].
  - PRIIPs KID where packaged [I].
- **Listing Act.** Reg. 2024/2809 (adopted 23 Oct 2024, published OJ 14 Nov 2024, in force 4 Dec 2024) sets the principal €12M threshold; the amended exemption threshold applies from **5 June 2026** (18 months after publication) [C#9]. Other Listing Act provisions and the new prospectus formats have their own application dates [I]. ESMA consulted on further Prospectus Regulation disclosure changes on 9 Sep 2026 [C#18].
- **Ongoing reporting.**
  - Issuers: annual accounts via corporate law.
  - MiFID firms: ex-post cost reports and quarterly statements (Art 24/25).
  - ECSP: annual reporting to the NCA and default-rate disclosure [I].
  - MiCA white papers use inline XBRL from 23 Dec 2025 [C#16] (only relevant for MiCA crypto-assets).
- **Marketing.**
  - MiFID "fair, clear and not misleading" (Art 24(3)).
  - ECSPR marketing communications must be consistent with the KIIS.
  - National rules on real-estate and investment advertising.
  - MiCA marketing rules for crypto-assets.
  - Cross-border marketing of funds follows the Cross-Border Distribution Directive (pre-marketing notification) [I].

## 8. Tax reporting

| Regime | Status | Platform relevance |
|---|---|---|
| **DAC8** (crypto) | Applies from **1 Jan 2026**. First RCASP reports due **30 Sep 2027**. Based on OECD **CARF**; covers crypto-assets incl. EMTs and some NFTs [C#12] | Applies if RealX is an RCASP (e.g., exchanges EMTs). Tokenised securities are generally reported under CRS/DAC2 rather than CARF, to avoid double reporting [I] |
| **CRS / DAC2** (financial accounts) | Ongoing; amended CRS aligned with DAC8 from 2026 [I] | A custodian or investment firm holding tokenised securities is a Reporting FI: self-certification, TIN, annual report [I] |
| **FATCA** | IGAs | W-8/W-9 collection for US persons [I] |
| **Withholding tax** | National dividend/interest WHT (e.g., DE 26.375%, FR 12.8%/25%, ES 19%); treaty relief. FASTER Directive (Council adoption Dec 2024; application 2030) [I] | WHT engine and relief-at-source forms [I] |
| **Real-estate transfer taxes on share deals** | Germany RETT at 90% share transfer within 10 yrs; France 5% on real-estate-rich company shares; Spain anti-abuse on real-estate-company shares [I] | **Token transfers may trigger RETT** → holder-concentration monitoring [I] |
| **VAT** | Transactions in securities are VAT-exempt (VAT Dir. Art 135(1)(f)). Letting of immovable property is exempt with option to tax [I] | Platform fees may be VAT-able unless they qualify as exempt negotiation of securities [I] |

## 9. Land registry / title linkage / foreign ownership

- National land registries (e.g., German *Grundbuch*, Spanish *Registro de la Propiedad*, French *publicité foncière*) record the **SPV** as owner. No Member State registers token holders as land owners [I].
- A tokenised transferable security represents company shares or notes, not title [I]. Liechtenstein's TVTG allows tokens to represent rights in real property, but conveyance of land still follows land-register rules [I].
- **Foreign ownership.** Free movement of capital (TFEU Art 63) bars most restrictions on EU/EEA buyers. Restrictions remain on non-EU buyers or on secondary homes in several states [I]:
  - Denmark: permission requirement;
  - Malta: AIP permit;
  - Austria: Länder land-transfer laws;
  - Finland: approval for non-EU/EEA buyers since 2020;
  - Cyprus: permit for non-EU buyers;
  - Croatia: reciprocity for non-EU buyers.
  - Spain proposed punitive taxes on non-EU buyers in 2025 (status unverified) [S].
  - **Admin control:** investor-nationality vs asset-country rule table.

## 10. 2025–2026 developments (status on 23 Sep 2026)

| Date | Development | Status |
|---|---|---|
| 30 Dec 2024 | MiCA fully applicable; TFR crypto travel rule live | In force [C#2][C#13] |
| 17 Dec 2024 | ESMA guidelines on qualifying crypto-assets as financial instruments | In force (comply-or-explain) [C#3] |
| 2025 | DLT Pilot authorisations: 360X (29 Apr 2025), Axiology (9 Jul 2025), LISE (13 Oct 2025), Securitize Europe (26 Nov 2025). Earlier: CSD Prague (Oct 2024), 21X (Dec 2024) | 6 infrastructures [C#5] |
| Summer 2025 | AMLA operational | Direct supervision 2028 [C#11] |
| 28 Nov 2025 | ESMA statement on MiCA data standards (white-paper iXBRL from 23 Dec 2025; JSON order book) | In force [C#16] |
| **4 Dec 2025** | **Market integration and supervision package** (SIU): Master Regulation COM(2025) 943, Master Directive, Settlement Finality Regulation. **DLT Pilot → €100bn cap, permanent**; **all CASP supervision to ESMA** (except credit institutions); CSDR DLT modernisation incl. EMT settlement [C#6][C#7][C#8] | Proposal in co-decision [I] |
| 1 Jan 2026 | **DAC8** applies | Reports due 30 Sep 2027 [C#12] |
| **5 Jun 2026** | **Listing Act** amended prospectus-exemption threshold applies (€12M per issuer/12 months; Member State option €5M) | In force [C#9] |
| 16 Apr 2026 | AIFMD II transposition deadline | [I] |
| **1 Jul 2026** | **End of MiCA CASP grandfathering** (latest date; some Member States ended earlier) | Unauthorised CASPs must have ceased [C#2] |
| 9 Sep 2026 | ESMA consults on Prospectus Regulation disclosure and updates guidelines and Q&As | Open [C#18] |
| 10 Jul 2027 | AMLR applies | Upcoming [I] |

## 11. Enforcement / litigation relevant to real-estate tokenisation

- **Coverage gap.** No EU-level enforcement against a real-estate tokenisation platform was verified this session (search quota exhausted) [I].
- **Cross-border exposure.** About 400 French investors are pursuing claims arising from the US RealT collapse, which shows EU retail exposure to non-EU tokenised offerings [C#US-17, see us.md].
- **Typical national risks** [I]:
  - AMF, CONSOB and CNMV warning and blacklist entries for unauthorised offers of "real-estate tokens";
  - BaFin orders against unauthorised offers of investment products without a prospectus or WIB;
  - NCAs treating secondary "marketplaces" as unauthorised MTFs.
- **Admin control:** jurisdiction-based offer gating plus a marketing approval log.

---

## Compliance obligations checklist

| Obligation | Legal basis | Who's responsible | Platform control needed |
|---|---|---|---|
| Classify each token (TS / AIF unit / MiCA / other) | MiFID II Art 4(1)(44); MiCA Art 2(4)–(5); ESMA GL 2024 | Issuer + platform | Token classification record with legal-opinion upload per asset |
| Prospectus or exemption tracking (€12M / €5M) | Prospectus Reg. as amended by 2024/2809 | Issuer / offeror | Per-issuer rolling 12-month raise meter per Member State |
| ECSP €5M cap per project owner | ECSPR Art 1(2)(c) | ECSP | Project-owner raise-cap meter (incl. lending) |
| KIIS per offer | ECSPR Art 23 | Project owner + ECSP | KIIS template builder, version control, NCA language |
| Entry knowledge test and loss simulation | ECSPR Art 21 | ECSP | Test engine, score storage, 2-yearly re-test |
| Warning / consent above €1,000 or 5% of net worth | ECSPR Art 21(7) | ECSP | Ticket-size check vs declared NW, explicit-consent capture |
| 4-day reflection period | ECSPR Art 22 | ECSP | Order state "reflection" with revoke button and timer |
| Client categorisation, appropriateness/suitability | MiFID II Arts 24–25; Annex II | Investment firm | Categorisation workflow, opt-up evidence, appropriateness questionnaire |
| Target market and product governance | MiFID II Art 16(3), 24(2) | Manufacturer / distributor | Target-market fields per product; distribution-outside-TM alerts |
| PRIIPs KID (if packaged) | Reg. 1286/2014 | Manufacturer | KID library, pre-trade delivery log |
| CDD / UBO / PEP / sanctions | AMLD (→ AMLR 2027) | Obliged entity | KYC/KYB, UBO ≥25%, PEP/EU sanctions screening incl. wallets, periodic review |
| Travel-rule data on crypto/EMT legs; self-hosted >€1,000 ownership proof | TFR Arts 14–16 | CASP / payment provider | Travel-rule messaging integration; wallet-ownership proof (signature / Satoshi test) |
| Qualified custody of securities / client funds | ECSPR Art 10; MiFID Art 16 | ECSP / firm / custodian | Custodian integration, daily reconciliation, segregation reports |
| No multilateral matching without MiFID licence | ECSPR Art 25; MiFID II | ECSP | Bulletin-board mode (indications of interest only); feature flag for MTF |
| DLT register integrity | Ley 6/2023 Arts 6, 8; eWpG | Register keeper | Golden-record ledger, holder identification, corporate-action module |
| Marketing: fair, clear, not misleading | MiFID Art 24(3); ECSPR Art 27 | Firm / ECSP | Marketing approval queue, per-country disclaimers |
| DAC8 / CRS / FATCA reporting | DAC8 (2026); DAC2; IGAs | RCASP / reporting FI | Self-cert with TIN, residency logic, annual XML generator |
| WHT on distributions | National tax law and treaties | Paying agent | Tax residency, treaty forms, WHT calculator |
| RETT share-deal thresholds | National RETT (e.g., DE 90%/10y) | SPV / sponsor | Holder-concentration monitor per SPV |
| Foreign-buyer restrictions | National land laws | Sponsor | Nationality × asset-country rule table |
| AIFM / depositary (if pooled) | AIFMD / AIFMD II | AIFM | Fund-vs-SPV classification record, AIFM reporting (Annex IV) |

---

## Sources (accessed 23 Sep 2026)

1. ESMA, "Markets in Crypto-Assets Regulation (MiCA)" — https://www.esma.europa.eu/esmas-activities/digital-finance-and-innovation/markets-crypto-assets-regulation-mica
2. Regulation (EU) 2023/1114 (MiCA), EUR-Lex HTML — https://eur-lex.europa.eu/legal-content/EN/TXT/HTML/?uri=CELEX:32023R1114
3. ESMA, Final Report: Guidelines on conditions and criteria for qualification of crypto-assets as financial instruments, 17 Dec 2024 — https://www.esma.europa.eu/sites/default/files/2024-12/ESMA75453128700-1323_Final_Report_Guidelines_on_the_conditions_and_criteria_for_the_qualification_of_CAs_as_FIs.pdf
4. Regulation (EU) 2022/858 (DLT Pilot Regime) — https://eur-lex.europa.eu/eli/reg/2022/858/oj
5. ESMA, "List of Authorised DLT Market Infrastructures" (Jan 2026) — https://www.esma.europa.eu/sites/default/files/2026-01/Authorised_DLT_Market_Infrastructures.pdf
6. European Commission, "Market integration and supervision package", 4 Dec 2025 — https://finance.ec.europa.eu/publications/market-integration-and-supervision-package_en
7. Proposal COM(2025) 943 (Master Regulation), EUR-Lex HTML — https://eur-lex.europa.eu/legal-content/EN/TXT/HTML/?uri=CELEX:52025PC0943
8. European Commission factsheet, "Breaking down barriers to integrate financial markets" — https://finance.ec.europa.eu/regulation-and-supervision/savings-and-investments-union/factsheet-breaking-down-barriers-integrate-financial-markets_en
9. Regulation (EU) 2024/2809 (Listing Act – Prospectus amendments), EUR-Lex HTML — https://eur-lex.europa.eu/legal-content/EN/TXT/HTML/?uri=CELEX:32024R2809
10. Regulation (EU) 2020/1503 (ECSPR), EUR-Lex HTML — https://eur-lex.europa.eu/legal-content/EN/TXT/HTML/?uri=CELEX:32020R1503
11. AMLA, "About AMLA" — https://www.amla.europa.eu/about-amla_en
12. European Commission DG TAXUD, "DAC8" — https://taxation-customs.ec.europa.eu/taxation/tax-transparency-cooperation/administrative-co-operation-and-mutual-assistance/directive-administrative-cooperation-dac/dac8_en
13. Regulation (EU) 2023/1113 (Transfer of Funds Regulation), EUR-Lex HTML — https://eur-lex.europa.eu/legal-content/EN/TXT/HTML/?uri=CELEX:32023R1113
14. Directive (EU) 2024/927 (AIFMD II), EUR-Lex — https://eur-lex.europa.eu/eli/dir/2024/927/oj
15. Ley 6/2023, de 17 de marzo, de los Mercados de Valores y de los Servicios de Inversión (BOE-A-2023-7053) — https://www.boe.es/buscar/act.php?id=BOE-A-2023-7053
16. ESMA, Statement to support the smooth implementation of MiCA standards and format, 28 Nov 2025 — https://www.esma.europa.eu/sites/default/files/2025-11/ESMA75-1303207761-6284_Statement_to_support_the_smooth_implementation_of_MiCA_standards_and_format.pdf
17. ESMA, "DLT Pilot Regime" page — https://www.esma.europa.eu/esmas-activities/digital-finance-and-innovation/dlt-pilot-regime
18. ESMA News (Sep 2026 list incl. Prospectus consultation 09/09/2026) — https://www.esma.europa.eu/press-news/esma-news
19. Lilex, Gesetz vom 3. Oktober 2019 über Token und VT-Dienstleister (TVTG) — metadata page only — https://www.gesetze.li/konso/2019301000
20. European Commission, Savings and Investments Union page (package adoption date 4 Dec 2025) — https://finance.ec.europa.eu/regulation-and-supervision/savings-and-investments-union_en
