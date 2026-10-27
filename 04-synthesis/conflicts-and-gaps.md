# Conflicts & Gaps

**Date:** 23 Sep 2026.

**Evidence key.** Labels are carried over from the source files: [C] confirmed, [I] inferred, [S] speculative. File codes are defined in `00-executive-summary.md`. F1–F6 are the six re-check fetches made during synthesis (§4); WebSearch was not used.

**Resolution rule:**
1. A primary legal document (T&C, prospectus, statute, regulator register or release) beats a platform's help centre or homepage.
2. A help centre or homepage beats a third-party review or blog.
3. A later-dated source beats an earlier one when the metric is time-series.
4. When sources are equal, both values are shown and the item is flagged.

The "Materiality" column says how much the conflict matters for RealX decisions: H = high, M = medium, L = low.

---

## 1. Contradictions between files (value A vs value B → resolution)

### 1.1 Regulatory file vs platform files (cross-agent)

| # | Topic | Value A (file) | Value B (file) | Resolution & basis | Materiality |
|---|---|---|---|---|---|
| 1 | **Australia: Digital Asset Platform / Tokenised Custody Platform law** | "Exposure draft 25 Sep 2025 … **Bill status unverified**" [C/I reg/au §2, §10] | "Digital Assets Framework Act 2026, Act No. 38 of 2026; Royal Assent 8 Apr 2026; commences 8 Apr 2027; transition to 8 Oct 2027" (CryptoSlate, a secondary source) [au/sum §5.3; au/02 §2.3; au/03 §2.3; au/04 §2.3; au/05 §12] | **The platform files are right.** F1/F5 (legislation.gov.au) confirm the Act (No. 38, 2026), Royal Assent on 8 Apr 2026, and that the **whole Act commences 8 Apr 2027**. They also confirm a 6-month transition for existing operators, a low-value exemption ($10m per 12 months, $5k per client), and that Schedule 1 is titled "Digital asset platforms and tokenised custody platforms". The regulatory agent's "not in the bills-before-Parliament list" observation fits the facts: an enacted Act is no longer a bill | **H** |
| 2 | **Australia: does a fractional property token fit the TCP regime?** | "A Redbelly-based property tokeniser holding title through a custodian and issuing tokens is squarely the TCP model" [I au/02 §2.3] | Statutory definition (s761GD): the operator "creates a single digital token per asset", confers redemption rights, and holds each asset in trust [C F5] | The au/02 inference is **not confirmed**. The TCP definition reads as one token per underlying asset. Fractional MIS-unit tokens may remain plain MIS interests (INFO 225) rather than TCP products. Needs counsel [I syn] | **H** |
| 3 | **Saudi tokenisation framework** | "KSA REGA / CMA real-estate tokenisation framework: **not confirmed** by any fetched source [S]" [reg/me §10] | REGA's first tokenisation and a regulatory standard (SPA, 19 Nov 2025); sandbox 2nd edition with a tokenisation track (SPA, 13 Feb 2026) [C me/04 §2.3] | F2 confirms the SPA release of 19 Nov 2025. REGA announced "the world's first official tokenization standards" and a Regulatory Sandbox; technical specifications were due in early 2026. **The pilot and the standards are confirmed.** The final tokenisation law or executive regulations, and **the CMA's role**, are still unconfirmed | **H** |
| 4 | KSA non-Saudi ownership law, effective date | 22 Jan 2026 (REGA primary) [C reg/me §9] | 21 Jan 2026 (secondary sources) [me/04 §2.3] | **22 Jan 2026**: the REGA release says "enters into force as of today" | L |
| 5 | DFSA crowdfunding investor limits | "Detailed investor limits not confirmed in this session" [reg/me §2] | Retail US$50k per property and US$100k per year (AED 183,500 / 367,000); Professional clients up to 33% [C me/03 §2.6] | Use the me/03 values (from Stake's help centre and a secondary summary). Verify against the DFSA rulebook | M |
| 6 | Stake's KSA entity and CMA permit | "Stake Financial Technology, 13/06/2023" on the CMA ExPermit list [C reg/me §2] | "Hessa Menfea Financial Technology Company"; permit 2023-53-5 (privacy policy) or 05-53-2023 (website) [C me/03 hdr] | Treat as the same CMA FinTech permit relationship. The exact legal name, number and date are unverified | L–M |
| 7 | US items marked "not verified" by the US platform agent | SEC statement date and number, GENIUS, CLARITY, innovation exemption all "not verified" [us/sum §8] | Confirmed with primary sources: SEC statement 28 Jan 2026; GENIUS signed 18 Jul 2025; CLARITY cloture failed 15 Sep 2026; innovation exemption 17 Sep 2026 [C reg/us §1, §10] | **Use reg/us.** The SEC URL itself encodes 012826 (28 Jan 2026) | M |
| 8 | Date of the SEC tokenised-securities statement | Blog dated Feb 2026 [us/02 src 11] | 28 Jan 2026 [C reg/us §1] | **28 Jan 2026.** The RedSwan blog post came after the statement | L |
| 9 | Comment deadline for the Regulation Crypto Assets proposal | "Comments due 20 Oct 2026" (RedSwan summary) [us/02 §9] | "60 days after Federal Register publication" [C reg/us §1] | Use the reg/us wording. 20 Oct is plausible but unverified | L |
| 10 | Timing of RealT's payout halt | "Payouts halted by end-2025" [C reg/us §9] | "Largely halted late 2025; formally suspended Feb 2026" [C us/04 hdr, §2.8]; us/sum says "suspended Feb 2026" | Both hold. De facto halt by end-2025 (F6: "by the end of 2025 … largely ground to a halt"); formal suspension Feb 2026 | L |
| 11 | Scope of the Detroit suit | 408 properties, 53 "unquestionably harmful" [C reg/us §9; C us/04 §2.8] | "Over 100 properties" (Crypto Briefing) [F6] | **408**: local primary reporting (Outlier Media) and the city's release. ~700 is the whole portfolio under the fiduciary | L |
| 12 | Two RealT "2 July" dates | Detroit suit filed 2 Jul 2025 [C reg/us §11] | Voluntary liquidation announced 2 Jul 2026 [C us/04 hdr] | F6 confirms 2 Jul 2026 for the liquidation (article dated 28 Jul 2026, which is the date us/sum cites). The suit date rests on Outlier Media (Jul 2025) and was not re-fetched. Keep both; flag the suit date for re-check | L–M |
| 13 | **Prospectus threshold in Spain** | "Below the Spanish threshold (€8m)" [I eu/01 §2.3; I eu/04 §2.3; eu/sum §8] | EU Listing Act: **€12M per issuer per 12 months, with a Member State option of €5M** [C reg/eu §2, citing EUR-Lex 2024/2809] | The Listing Act framework governs in 2026. Spain's former €8m national threshold is probably superseded. **Which option Spain chose is unverified** [I] | **M–H** |
| 14 | AU no-action position under INFO 225 | "No-action to 30 Jun 2026" [au/sum §5.3; au/03 §2.3] | The regulatory agent's fetch of INFO 225 (last modified 30 Apr 2026) found "no no-action deadline" [reg/au search log #150] | As of Sep 2026, do not rely on any no-action position. It has either lapsed (30 Jun 2026) or been removed [I] | M |
| 15 | INFO 225 update date | 29 Oct 2025 (ASIC 25-250MR) [C au files] | Last modified 30 Apr 2026 [C reg/au §1] | Both are true; 30 Apr 2026 is the current version | L |
| 16 | Chain the Lofty and RealT tokens run on | "Lofty (US) is a known Algorand issuer [I — to be confirmed]"; "RealT … on Gnosis [I — confirm]" [inf/std §2.1, §2.8] | Algorand ASA [C us/01 §7]; Gnosis primary [C us/04 §7] | Confirmed by the platform files. The synthesis cites them, rather than upgrading the inf/std label | L |
| 17 | UAE PASS use at PRYPCO | "DLD/PRYPCO likely use it [I — confirm]" [inf/ven §1] | UAE PASS login added in app v1.1.31 (Aug 2026) [C me/01 §2.5] | Confirmed for login by me/01 | L |
| 18 | Blocksquare chain | "Ethereum/Polygon-family [I]" [inf/wl §0] | Ethereum mainnet contracts documented; multi-chain claimed [C eu/02 §7] | **Ethereum mainnet** [C]. L2 deployments are unverified | L |

### 1.2 Contradictions inside platform files

These are flagged by the regional agents and resolved here.

| # | Platform | Value A | Value B | Resolution & basis | Materiality |
|---|---|---|---|---|---|
| 19 | **PRYPCO: minimum ticket** | AED 2,000 [C reg/me §3; C me/01 hdr; C me/02 hdr] | AED 1,000 for the secondary market from Jul 2026 [C me/01 hdr]; the product page lists AED 1,000 [me/sum §6] | F4 (prypco.com/mint, 23 Sep 2026): "The minimum investment amount is AED 1,000". **AED 2,000 at launch (May 2025); AED 1,000 now (Sep 2026).** The page gives a single minimum and does not say whether it covers both primary and secondary | M |
| 20 | PRYPCO: minimum age | 18 (T&C, press) | 21 (help centre) [me/01 §2.6] | **18.** The T&C governs | L |
| 21 | PRYPCO: lock-in | 3 months (T&C) | "None" (blog) [me/01 §10] | **3 months.** The T&C governs, and F4 confirms it | L |
| 22 | PRYPCO: sale-vote threshold | 51% | 50% + 1 [me/01 §10] | **50% + 1** (T&C) | L |
| 23 | PRYPCO: legal structure | "SPV issues tokens" (user blog) | Direct title in investors' names with DLD (T&C) [me/01 §2.1] | **Direct title.** The T&C is the primary legal document | M |
| 24 | DLD go-live date | 24 May 2025 (VARA licences; Ctrl Alt press release) [C reg/me §10] | 25 May 2025 (DLD news) [C me/01 hdr] | Licences and go-live on 24 May; DLD's public announcement on 25 May | L |
| 25 | Saudi: legal structure | Direct fractional deed (Jozo, REGA framing, Raghdan) | SPV/LLC wrapper (vendor page) [me/04 §2.1] | **Direct deed.** The operator's primary statement is preferred | M |
| 26 | Saudi: minimum ticket | SAR 1,000 (Jozo; Ghanem) | ~SAR 5,000 (some sources) [me/04 hdr] | SAR 1,000 at Jozo; varies by operator | L |
| 27 | Stake: scale | Rent paid AED 31m+ (homepage); 500 properties | AED 55m+ (Feb 2026); 600+ properties [me/03 hdr, §10] | Use the latest: AED 55m+, 500–600+ | L |
| 28 | Stake: distribution cadence | Monthly (homepage) | Weekly target (help centre) [me/03 §5.3] | Weekly target (current help-centre article) | L |
| 29 | SmartCrowd: exits | 40+ exits / AED 22m (exits page) | 77 exits (homepage); AED 220m (Feb 2026 release) [me/05 §10] | 77 exits / AED 220m | L |
| 30 | SmartCrowd: fees | Own fee pages | Review says 2% management and 1.5% seller fee [me/05 §8] | Use the own fee pages | L |
| 31 | Ctrl Alt: legal entity | "Solutions DMCC" (VARA register) | "Solutions FZCO" (own site) [me/02 hdr] | DMCC is the VARA licensee | L |
| 32 | RealT: investor count | 14,000–22,000 affected | "20,000+ holders"; 65,000 registered [us/04 hdr] | 14–22k affected investors (F6 agrees); 65k are registered accounts | L |
| 33 | Lofty: listing fee | 3% of equity sold [C#1] | 5% [C#9] [us/01 §1] | **Unresolved.** Use the 3–5% range | L |
| 34 | **Lofty: wallet custody** | Non-custodial (ToS) [C#3] | "Custodial" (Coinpaprika) [C#9] [us/01 §7] | The ToS claims non-custodial; actual key management is unverified. This matters for MSB and custody analysis | M |
| 35 | Lofty: property count | 180+ (Sep 2026) | 111 listed (May 2026) [us/01 hdr] | Cumulative tokenised vs currently listed | L |
| 36 | Lofty: TVL source number | us/sum cites [C#18] | us/01 cites [C#11] | Numbering differs between files; it is the same DefiLlama figure | — |
| 37 | Arrived: secondary windows | Monthly one-week windows (product) | Quarterly (1-K text) [us/03 §9] | **Monthly** (current product) | L |
| 38 | Arrived: dividend cadence | "Monthly or other" (1-K) | Quarterly (reviews) [us/03 §5.3] | Varies by product | L |
| 39 | Arrived: scale | 885k sign-ups | 850k investors [us/03 hdr] | Different metrics | L |
| 40 | Arrived: asset-management fee | 0.6% per year (1-K) | 0.15% per quarter (reviews) [us/03 §1] | Equivalent; no conflict | — |
| 41 | **RedSwan: scale** | "$4B" / "$5B+" (site, Hedera) | "Over $100 million" (Oct 2025 press) [us/02 hdr, §10] | **~$100M+** corroborated. The billions are unaudited pipeline | M |
| 42 | RedSwan: RIA status | SEC-registered RIA (website) | "Not an investment manager" (ToS) [us/02 §2.3] | Unresolved. Check the SEC IAPD | L–M |
| 43 | mogul: eligibility | US only (ToS) | "Equivalent ID if non-US" (docs) [us/05 §2.6] | US only; the ToS governs | L |
| 44 | mogul: vote weighting | One vote per member plus one per unit (ToS) | Pro rata (docs) [us/05 §5.6] | The ToS governs | L |
| 45 | Propvest: operator entity | Exporo Invest GmbH (Impressum) | "EPH Investment GmbH" (other pages) [eu/03 hdr] | Same group; the Impressum governs | L |
| 46 | **Propvest: bond security** | Subordinated notes linked to a land charge (critic) | Subordinated debentures through property companies [eu/03 §2.1] | **Unresolved.** Needs the prospectus. Matters for the "secured vs unsecured" design lesson | M |
| 47 | Propvest: AUM / projects | ~€250m, 65 projects | €256m, 68 properties [eu/03 hdr] | Timing difference | L |
| 48 | Domoblock: founding year / investors | 2021; 3,412 (track-record page) | 2022; 2,500+ (homepage) [eu/05 hdr] | 2021; 3,412 | L |
| 49 | **Brickken: volume** | US$660m+ TTV | ~US$41m on-chain TVL (end-2025) [eu/04 §10] | Different definitions: TTV is issuer-declared asset value. **Use TVL** for on-chain value | M |
| 50 | Blocksquare: scale | US$216.9m "tokenised value" | Not capital raised [eu/02 hdr] | Different definitions: property valuations | M |
| 51 | BrickX: buyback | "Buyback guarantee" (CrowdSpace) | PDS says illiquid, no buyback [au/01 §5.4] | The PDS governs | L |
| 52 | BrickX: minimum | A$50 (2019 PDS) | A$250 first deposit including a A$10 fee (2020 PDS) [au/01 hdr] | Both: A$250 first deposit; Bricks from about A$50 | L |
| 53 | **Redbelly: on-chain assets** | US$73.8bn (Dec 2024 claim; mostly carbon pipeline) | ~US$1bn (RWA.io, Jan 2026) [au/02 §2.8] | **~US$1bn** | M |
| 54 | Tokeniser / Liquidise: scale | A$1.2B issued (Tokeniser) | US$736m / A$1.17B (Redbelly on Liquidise) [au/03 hdr] | Probably the same book [I] | L |
| 55 | Assetora: former trustee | Melbourne Securities Corporation | "MSC Trustees" [au/05 §2.1] | Same entity | L |

---

## 2. Load-bearing claims that rest only on [I] or [S]

These claims drive synthesis recommendations but have no confirmed source.

| # | Claim | Label & file | Where it drives the synthesis | Risk if wrong |
|---|---|---|---|---|
| 1 | Lofty's property LLCs elect C-corp status, so investors get a 1099-DIV instead of a K-1 | [I us/01 §2.7] | The "corporate wrapper for 1099s" pattern | Low. Arrived's REIT election [C us/03 §2.7] carries the same lesson |
| 2 | BrickX's and Assetora's internal markets rely on ASIC relief rather than a market licence | [S au/01 §2.3; S au/05 §2.3] | Whether RealX can run an internal board in Australia | **High**: it decides whether secondary trading in AU needs a market licence |
| 3 | AMM pools for ERC-3643 tokens are "exempt public protocols" | Tokeniser's claim; [I au/03 §2.3] flags it as untested | AMM treated as a later-phase option | High: s767A market definition |
| 4 | Blocksquare's revenue-share tokens are MiCA crypto-assets, not MiFID securities | Blocksquare claim; [I eu/02 §2.3] | Avoid CP4 postures | High for any EU operator copying the model |
| 5 | Reental's whitelist is enforced only in the UI, not in the token | [I eu/01 §7] | "Enforce whitelist at transfer" | Medium |
| 6 | How VARA's ARVA Annex 2 rules (reserves, redemption, audit) apply to DLD management tokens | [I me/02 §2.2] | UAE compliance stack | **High** for a Category 1 issuer route; lower for a distributor |
| 7 | DLD pilot admits more distributors (Ctrl Alt says "PRYPCO and others") | [C me/02 §1] for intent; the admission process is [I me/02 §5.1] | UAE go-to-market Phase 2 | **High** |
| 8 | Whether the 4% DLD transfer fee applies to token transfers | [S reg/me §8] | UAE secondary-market economics | Medium |
| 9 | XRPL token type used on DLD (IOU vs Multi-Purpose Token) | [I me/01 §7] | Tech stack | Low for a distributor |
| 10 | Saudi RER custody provider and chain | [I me/04 §7] | KSA integration | Low–Medium |
| 11 | Each wallet on an issuer-sponsored ledger may count as a US §12(g) holder of record | [I reg/us §1] | Holder-count monitor | **High**: it could force Exchange Act registration |
| 12 | Active ATS trading of LLC tokens can make the LLC a publicly traded partnership | [I reg/us §1, §6] | US tax-wrapper choice | High |
| 13 | A third-party funds holder (Modern Treasury, NCIT) keeps the platform out of MSB status | [I us/03 §2.5] | US money-movement design | Medium–High |
| 14 | Token transfers can trigger EU real-estate transfer tax (RETT) on share deals, and AU landholder duty / foreign surcharges | [I reg/eu §8; I reg/au §8] | Concentration monitors | High |
| 15 | FIRB applies to foreign holders of units in land-rich trusts; BrickX excludes foreigners for this reason | [I au/01 §2.6; I reg/au §9] | AU eligibility rules | High |
| 16 | Only CBUAE-licensed AED payment tokens may be used onshore; the new CBUAE law's transition runs to ~Sep 2026 | [I reg/me §4, §10] | UAE stablecoin allow-list | Medium |
| 17 | Timeline and cost indicators: Reg A 4–9 months and $100–300k; VARA 6–12 months; ECSP 6–12 months; AFSL 6–12+ months; legal costs $25–75k per SPV | [S reg/us §2; S reg/me §2; S reg/eu §2; S reg/au §2] | GTM phasing | Medium: the timelines may slip |
| 18 | RealT had a FIRPTA gap on P2P trades | [I us/04 §2.7] | Withholding at settlement | Medium (the lesson stands either way) |
| 19 | mogul has MSB and unregistered-broker exposure | [I us/05 §2.3, §2.5] | Avoid list | Low |
| 20 | KYC vendors for 15 of the 20 platforms (Sumsub/Onfido/Veriff guesses) | [S per file §2.5] | Vendor benchmarking | Low |
| 21 | Fractional MIS-unit tokens fall outside the TCP definition | [I syn from F5] | AU licensing plan | **High** |

---

## 3. Prioritised re-verification list

### P1: before committing to architecture or licences

| # | What to check | Where | Why it matters |
|---|---|---|---|
| 1 | Whether fractional MIS-unit tokens (and custodial holding of them) fall under the DAF Act's TCP or DAP definitions; any ASIC transitional guidance | legislation.gov.au C2026A00038 (Schedule 1, s761GD and neighbours); ASIC | Decides AU licensing and the design of the golden record |
| 2 | Text of the SEC transfer-agent modernisation proposal: is a DLT master file allowed, what TA controls are required, Form TA-2 | sec.gov (proposal of 1 Sep 2026; comments due 3 Nov 2026) | US golden-record design; whether RealX's token can be the master file |
| 3 | DLD rules for admitting distributors, the Phase 3 roadmap (non-residents), and fees on token transfers | DLD REES e-service; Ctrl Alt; VARA | UAE entry route and secondary-market economics |
| 4 | Prospectus threshold each target Member State adopted under the Listing Act (€12M vs €5M), and application dates | EUR-Lex 2024/2809; CNMV; BaFin | Size limits of EU offerings outside ECSP |
| 5 | Final Saudi tokenisation regulation / executive rules, and the CMA's role | rega.gov.sa; cma.gov.sa | KSA entry and licensing |
| 6 | When an internal P2P board becomes an exchange or ATS (US), an MTF (EU) or a financial market (AU) | Counsel; SEC Reg ATS; ESMA; ASIC s767A guidance | Secondary-market v1 design |
| 7 | Legal basis of the BrickX and Assetora internal markets (ASIC relief instrument?) | ASIC instruments register; the PDSs | Precedent for an AU internal board |
| 8 | §12(g) holder-of-record counting for wallets, and publicly-traded-partnership treatment of LLC tokens traded on an ATS | Counsel; SEC guidance | US structuring |

### P2: before launching in a region

| # | What to check | Where | Why it matters |
|---|---|---|---|
| 9 | DFSA crowdfunding investor limits (retail US$50k / US$100k; Professional 33%) | DFSA rulebook | UAE off-chain bridge product |
| 10 | How VARA's ARVA Annex 2 applies to title-referenced management tokens | VARA rulebooks; Ctrl Alt | Obligations if RealX becomes an issuer, not just a distributor |
| 11 | CBUAE payment-token law transition (FDL 6/2025) and the list of licensed AED tokens (AE Coin) | centralbank.ae | UAE stablecoin allow-list |
| 12 | GENIUS Act final rules and the permitted-issuer list (OCC) | occ.gov; Treasury | US stablecoin launch timing |
| 13 | 1099-DA vs 1099-B: does a partner ATS count as a "limited-access regulated network" | IRS 1099-DA instructions | US tax-engine design |
| 14 | FIRB and state landholder duty for foreign or concentrated holdings of MIS units | foreigninvestment.gov.au; state revenue offices | AU eligibility and threshold rules |
| 15 | Terms of the CA DFPI consent order against Lofty (PDF was blocked) | dfpi.ca.gov | State-regulator view of the marketplace model |
| 16 | Does PRYPCO's AED 1,000 minimum apply to primary offerings? | Current PRYPCO Mint T&C | Benchmark ticket size |
| 17 | KYC vendor coverage of Emirates ID / UAE PASS, Nafath, and Australia's DVS | Vendor documentation | Identity stack per region |
| 18 | Detroit v. RealT: filing date, docket, and recovery outcomes | Wayne County Circuit Court records | Evidence for the wind-down playbook |

### P3: useful context

| # | What to check | Where | Why it matters |
|---|---|---|---|
| 19 | Propvest bond security (land charge?) and eWpG conversion | CSSF prospectus; BaFin crypto-securities register | Lesson on secured vs unsecured structures |
| 20 | Reental's placing ESI; CNMV registration of the "Reental Tourist Homes" bond | CNMV registers | EU competitor posture |
| 21 | RedSwan: asset-level Form Ds; BrokerCheck; IAPD status | EDGAR; FINRA; SEC IAPD | Competitor credibility |
| 22 | Blocksquare L2 deployments; whether any operators hold CASP authorisation | Blocksquare docs; ESMA CASP register | EU competitor posture |
| 23 | XRPL token type for DLD tokens (IOU vs MPT) | XRPL explorer | Distributor integration |
| 24 | Stake's KSA permit number and entity name | CMA ExPermit list | Competitor mapping |
| 25 | Redbelly mainnet RWA value | RWA.io; Routescan | Choice of AU chain |

---

## 4. Re-check fetch log

Six WebFetch calls were made to resolve material contradictions (limit: 10). No WebSearch was used, and no blocked site was worked around.

| ID | URL | Purpose | Result |
|---|---|---|---|
| F1 | https://www.legislation.gov.au/C2026A00038/latest/text | AU DAF Act status (conflict #1) | Corporations Amendment (Digital Assets Framework) Act 2026, No. 38, 2026; Royal Assent 8 Apr 2026; shown as "In force"; Schedule 1 "Digital asset platforms and tokenised custody platforms" |
| F2 | https://www.spa.gov.sa/en/N2448380 | Saudi framework (conflict #3) | 19 Nov 2025. REGA announced the first title-deed tokenisation (NHC and investors) and "the world's first official tokenization standards"; technical specifications due early 2026; Regulatory Sandbox within the Saudi PropTech Hub; the CMA is not mentioned |
| F3 | https://www.legislation.gov.au/C2026A00038/asmade/text | DAF Act commencement text | Table of contents only; no section text returned |
| F4 | https://prypco.com/mint | PRYPCO minimum ticket (conflict #19) | "The minimum investment amount is AED 1,000"; gold AED 100; 20% holder cap; 3-month lock-in; ±15% of the latest DLD valuation. Eligibility (Emirates ID, age) is not on the page |
| F5 | https://www.legislation.gov.au/C2026A00038/asmade/2026-04-08/text/original/epub/OEBPS/document_1/document_1.html | DAF Act commencement and TCP definition (conflicts #1–2) | Whole Act commences "the day after the end of the period of 12 months beginning on the day this Act receives the Royal Assent", i.e. **8 Apr 2027**. 6-month transition (s1732). Low-value exemption: <$10m per year and ≤$5k per client (s911A(2)(ja)). TCP (s761GD): the operator creates a single digital token per asset and holds it in trust |
| F6 | https://cryptobriefing.com/realt-crypto-real-estate-collapse/ | RealT dates (conflicts #10–12) | Liquidation announced **2 Jul 2026** by Jean-Marc Jacobson; fiduciary Charles Bullock appointed Apr 2026; ~$140M raised; 14,000–22,000 investors; ~700 properties; payouts "largely ground to a halt" by end-2025; article dated 28 Jul 2026 |
