# MENA — Selection, Ranking & Regional Summary

Research date: 23 Sep 2026 · Region: GCC + wider Middle East & North Africa · Evidence labels as per `00-method/platform-profile-template.md` ([C] confirmed, [I] inferred, [S] speculative). Platform files carry the numbered sources; this summary cites the platform file (e.g., "→01 [C#4]").

> **Research constraint.** The session-wide WebSearch quota (200 calls shared across parallel regional agents) ran out early in Platform 2. After that, all research used WebFetch on official domains, registers and URLs already surfaced. Candidates that could only be found through new searches (e.g., Aldar/Emaar developer tokenisation, QNB, Oman, Kuwait, Morocco, ADGM platforms) are therefore **under-verified** — see §6.

## 1. Ranking criteria

Aligned to `00-method/methodology.md` §2.3, with extra weight on tokenisation depth because the methodology gives on-chain / regulator-recognised digital fractions priority.

| Criterion | Weight | 5 = | 1 = |
|---|---|---|---|
| A. Volume / AUM / investors (fractional RE) | 20 % | ≥ US$100m or ≥ 1m users | no public numbers |
| B. Regulatory standing | 25 % | full licence(s) + regulator-backed pilot / sovereign role | unlicensed / announcement only |
| C. Tokenisation depth | 25 % | on-chain token with legal registry linkage | no token (off-chain ledger) |
| D. Secondary-market activity | 10 % | continuous regulated trading live | none |
| E. Longevity / track record | 10 % | ≥ 5 yrs, many exits | < 1 yr |
| F. Relevance & status Sep 2026 | 10 % | live, growing, strategically central | stalled / unverified |

**Tie-break:** more verified *retail-facing* activity ranks higher (because RealX is a retail-facing platform).

## 2. Scored longlist

| # | Candidate | Country | Label | A | B | C | D | E | F | **Score** | Outcome |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | **PRYPCO Mint** | UAE (Dubai) | On-chain, regulator-pilot distributor (XRPL, DLD) | 3 | 5 | 5 | 4 | 2 | 5 | **4.20** | **Rank 1** (tie-break: retail-facing) |
| 2 | **Ctrl Alt Solutions** | UAE (Dubai) / UK | Infra: VARA Cat 1 issuer + custody + TA | 3 | 5 | 5 | 3 | 3 | 5 | **4.20** | **Rank 2** |
| 3 | **Stake** | UAE + KSA | Off-chain fractional (DIFC SPV, CMA funds); VARA IPA | 5 | 4 | 2 | 3 | 4 | 5 | **3.70** | **Rank 3** (tie-break: 2m users) |
| 4 | **Saudi REGA/RER national tokenisation + sandbox operators (Jozo, Ghanem…)** | KSA | Regulator pilot + sovereign infra; registry-linked fractional deeds | 2 | 5 | 5 | 2 | 1 | 5 | **3.70** | **Rank 4** |
| 5 | **SmartCrowd / Nawy Shares** | UAE + Egypt | Off-chain fractional (DFSA SPV; FRA); VARA IPA | 4 | 4 | 2 | 3 | 5 | 4 | **3.50** | **Rank 5** |
| 6 | Tokinvest | UAE (Dubai) | VARA BD + Cat 1 issuance (VL/2024/12/004); Polygon; RE "coming soon" | 1 | 4 | 4 | 1 | 2 | 3 | 2.80 | Runner-up |
| 7 | Alt DRX | Qatar (QFC) / India | QFC Token Service Provider; "1 sqm" fractional; bank white-label | 2 | 3 | 3 | 2 | 3 | 3 | 2.70 | Runner-up |
| 8 | ATME | Bahrain | CBB-licensed RWA tokenisation venue (not RE-specific) | 1 | 4 | 3 | 2 | 2 | 2 | 2.55 | Benchmark |
| 9 | Tribe Tokenisation | UAE / Qatar | VARA Broker-Dealer VL/26/06/002 (22 Jun 2026); entering QFC Digital Assets Lab | 1 | 4 | 3 | 1 | 1 | 3 | 2.45 | Watch-list |
| 10 | SQMU | UAE (ADGM) | ADGM-framed RE tokenisation (evidence: own guide only) | 1 | 2 | 3 | 1 | 2 | 2 | 1.95 | Under-verified |
| 11 | SAFE (Madinet Masr) / Saqr-Frida | Egypt | FRA-path fractional (SAFE: EGP 50k shares) | 2 | 3 | 1 | 1 | 1 | 3 | 1.90 | Benchmark |
| 12 | DAMAC × MANTRA (US$1bn) | UAE | Developer–L1 RWA deal (Jan 2025) | 1 | 2 | 3 | 1 | 2 | 1 | 1.85 | No evidence of live retail RE product |
| 13 | MAG × MultiBank (US$3bn on MANTRA) | UAE | Developer–exchange RWA deal | 1 | 2 | 3 | 1 | 1 | 2 | 1.85 | No evidence of live product |
| 14 | RAK / RAK DAO initiatives | UAE (RAK) | — | — | — | — | — | — | — | n/a | No RE tokenisation platform found |
| 15 | Aldar / Emaar / Ellington / Binghatti developer-led | UAE | — | — | — | — | — | — | — | n/a | Not verified (search budget); Ellington is a Stake investor |
| 16 | QNB tokenisation; Oman; Kuwait; Morocco | — | — | — | — | — | — | — | — | n/a | Not researched (search budget) |

Score = Σ(weight × score). Evidence for rows 6–13 is in `_search-log.md` (S22, S26, S34, S35, S38, P1-24, S14, S41, S31, P5-7).

## 3. Ranking rationale (one line each)
1. **PRYPCO Mint** — the only live retail tokenised RE platform in MENA with government title-deed tokens (DLD certificate), XRPL, and a regulated 24/7 secondary market (AED 18.5m, 10 properties, 50+ nationalities) →01.
2. **Ctrl Alt** — the licensed issuer/tokenisation engine and MPC custodian behind the DLD rails (first VARA Cat 1 issuance licence; US$1.8bn tokenised across classes); the infra any new Dubai distributor plugs into →02.
3. **Stake** — MENA's largest digital fractional RE platform (2m+ users, AED 1.4bn transactions; DFSA + CMA) with a VARA tokenisation IPA; the UX/compliance baseline →03.
4. **Saudi REGA/RER national tokenisation** — world-first registry-as-truth tokenisation (enhanced ERC-3643 on SettleMint, Nafath identity) with 9 sandbox operators issuing fractional deeds; biggest GCC market →04.
5. **SmartCrowd / Nawy Shares** — MENA's first regulated RE crowdfunding platform (2018), deepest exit record, now Nawy-owned with VARA IPA (UAE) and FRA licences (Egypt) →05.

**Runners-up / benchmarks:** Tokinvest (first VARA multi-asset issuance licence; Polygon; RE not yet live), Alt DRX (QFC TSP, Qatar bank white-label), ATME (Bahrain CBB), Tribe Tokenisation (new VARA BD, Qatar entry), PRYPCO Blocks (sister DFSA product, covered in →01).

## 4. Regional comparison table

| Platform | Category | Chain(s) | Token standard | Legal wrapper | Exemption / licence | KYC vendor | Wallet / custody | Fiat & stablecoin rails | Secondary mechanism | Distribution cadence & ccy | User roles | Key admin modules |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| PRYPCO Mint | On-chain, regulator-pilot distributor | XRP Ledger [C] | XRPL issued token; dual ownership + ARVA mgmt token; IOU vs MPT unpublished [I] | Direct fractional title in investor names at DLD + DLD Token Ownership Certificate; no SPV [C] | VARA Broker-Dealer VL/25/05/001; DLD/VARA/CBUAE/DFF sandbox [C] | Undisclosed; Emirates ID + UAE PASS [C] | Custodial: Ctrl Alt warm MPC omnibus (Ripple Custody); in-app AED wallet [C] | AED only; Zand client-money a/c; bank transfer, card, Apple Pay; no crypto/stablecoin [C] | 24/7 in-app listing board; 3-mo lock-in; ±15 % of DLD valuation; 20 % cap [C] | Monthly (or quarterly), AED; reinvest to PAXG [C] | Guest, UAE-resident investor, gold investor, PRYPCO ops, compliance, Ctrl Alt, DLD, Zand | Offering builder (sqm→tokens), cooling-off/allocation, DLD/Ctrl Alt registry console, client-money recon, distribution engine, marketplace surveillance, votes (50 %+1, 30 % ROI trigger), complaints SLA, BCP/cessation |
| Ctrl Alt | Infra: issuer + custody + TA | XRPL (DLD, diamonds); Solana; Algorand (legacy) [C] | XRPL tokens; ASA (legacy); ARVA layer [C]/[I] | DLD title tokens (no SPV); Guernsey/Liechtenstein vehicles elsewhere [C] | VARA BD + Cat 1 VA Issuance VL/25/05/002; FCA 1045023; CBI MiFID; GFSC [C] | Delegated to distributors [C] | Warm MPC omnibus via Ripple Custody; legacy self-custody [C] | None on DLD rails (distributor handles) [C] | Settlement layer for distributor markets (dual-token) [C] | N/A — supplies holder snapshots [I] | DLD, distributors, issuer clients, pro investors, custody signers, TA, compliance, regulators | Tenant/distributor mgmt, asset registry, mint/burn console, DLD sync monitor, holder register, rules engine, ARVA disclosures/audit, MPC policy, whitepaper vault, multi-regulator reporting |
| Stake | Off-chain fractional; VARA IPA | None today; planned (unnamed) [C] | — (SPV shares / fund units) | DIFC SPV per property; CMA private funds [C] | DFSA crowdfunding F005879 (Islamic window); CMA FinTech permit; VARA IPA [C] | IDWise [C] | Custodial AED/SAR e-wallets; ENBD segregated [C] | Bank transfer, debit card (credit for pros) via Checkout.com, Wise; no stablecoin [C] | Exit Windows May & Nov (2 wks), 1-yr lock-in, valuation-set price [C] | Weekly target (AED); SAR for funds [C] | Retail, professional, KSA qualified, StakeOne buyer, Golden Visa applicant, fund manager, ops | Limits engine (AED 183.5k/367k, 33 %, 50/50 KSA), SPV secretarial, dividend engine, exit-window manager & price calculator, AutoInvest, rewards/StakePredict, DFSA/CMA reporting |
| Saudi REGA/RER + sandbox ops | Regulator pilot + sovereign infra; registry-linked fractional deeds | Permissioned EVM via SettleMint DALP [I] | "Enhanced ERC-3643"; 1:1 deed token + fractional tokens [C] | Fractional deeds in investors' names in RER (SPV claim unverified) [C] | REGA PropTech Sandbox (6–24 mo testing, weekly reports); CMA role unresolved [C] | Nafath / Absher / Yakeen (+ W3C VC) [C] | Registry-as-truth; custody provider undisclosed (DALP supports Fireblocks, Ripple Custody, DFNS, Utila, HSM) [C]/[I] | mada, SADAD, escrow-linked DvP; SAR; no stablecoin [C] | National marketplace in development; operator-level resale; transfer-time eligibility [C] | Operator-led, SAR (cadence undisclosed) [C] | REGA, RER/Aamal users, NHC/REDF, sandbox operators, investors, SettleMint/Inspire | Operator: Nafath KYC tiers, RER registration console, escrow/DvP, weekly REGA reporting, Shariah, exceptions desk; Registry: eligibility/geo-fence policy, price controls, API clients |
| SmartCrowd / Nawy Shares | Off-chain fractional; VARA IPA | None today [C] | — (SPV shares; co-ownership / fund units) | DIFC SPV "SC-…" (1m shares); Egypt Civil Code co-ownership → FRA funds [C] | DFSA crowdfunding F004285 (2018; Islamic Window endorsement); FRA 4 licences; VARA IPA [C] | Undisclosed [C] | Custodial eWallet, segregated [C] | Lean open-banking pay-by-bank, AED/USD; Egypt cards + instalments [C] | Share Transfer Facility Mar & Sep (2 wks), ±15 %, >1-yr property; vote-to-sell [C] | Monthly, AED [C] | Retail (global), professional, STF buyer/seller, voting shareholder, Egyptian instalment buyer, developer, advisors | Offering builder (Hold/Flip), SPV secretarial, returns engine (quarterly admin fee), STF manager, vote/exit & Flip hurdle calc, Lean recon, Egypt instalments/FRA reporting |

## 5. Regional patterns

1. **The land registry is the ledger of legal record.** Both flagship regimes anchor tokens to the state registry: Dubai (DLD issues a Property Token Ownership Certificate; Ctrl Alt syncs XRPL to DLD) and Saudi (RER runs the tokenisation stack; "settlement = registry update") →01 →02 →04. This is structurally different from US/EU SPV-share tokens and removes the need for an SPV — but ties every product to government integration and pilot rules.
2. **Split licensing: issuer/infra vs distributor.** Dubai separates the VARA **Category 1 issuer** (Ctrl Alt) from VARA **broker-dealer** distributors (PRYPCO; Stake and SmartCrowd with IPAs). Saudi puts the infra inside the registry and admits PropTechs via sandbox. New entrants will most likely be *distributors* on shared rails, not issuers →02 →04.
3. **Crypto-native branding, fiat-only plumbing.** No MENA RE platform lets retail self-custody or pay in stablecoins: AED/SAR wallets, client-money accounts (Zand, Emirates NBD), cards, Apple Pay, open-banking pay-by-bank (Lean), mada/SADAD; tokens sit in omnibus MPC custody →01 →03 →05.
4. **Guardrails are the product.** ±15 % price bands (PRYPCO vs DLD valuation; SmartCrowd vs market price), 20 % single-holder caps, lock-ins (3 months PRYPCO, 1 year Stake/SmartCrowd), DFSA caps (US$50k/property, US$100k/yr), 24-h cooling-off, KSA 50/50 qualified/retail tranches, REGA price controls →01 →03 →04 →05.
5. **Crowdfunding incumbents are migrating to VARA tokens — slowly.** Stake (IPA Jan 2026) and SmartCrowd/Nawy Shares (IPA Feb 2026) are not on the VARA register as of Sep 2026; only PRYPCO distributes DLD tokens. Licensing + DLD access is the bottleneck, not technology →03 →05.
6. **Government digital identity is table stakes.** UAE PASS + Emirates ID (PRYPCO), Nafath/Absher/Yakeen (Saudi) replace generic document KYC; private IDV vendors (IDWise) persist where non-residents are served →01 →03 →04.
7. **Residency gating shapes TAM.** DLD tokens: UAE residents only (gold is global); Stake/SmartCrowd accept most nationalities via DIFC SPVs; Saudi opens to non-Saudis only in designated zones under the Jan 2026 non-Saudi ownership law →01 →03 →04.
8. **Liquidity: continuous vs windows; Shariah as overlay.** Tokenised = 24/7 (PRYPCO); crowdfunding = twice-yearly windows (Stake May/Nov, SmartCrowd Mar/Sep). Shariah certification is common on crowdfunding and Saudi products (Shariyah Review Bureau at Jozo and PRYPCO Blocks; DFSA Islamic Window endorsements at Stake and SmartCrowd); PRYPCO Mint's product page carries only a bare "Shariah compliant" bullet with no certifier named →01 →03 →04 →05.

## 6. Gaps & uncertainties
- **Search budget exhaustion** (see top note): developer-led tokenisation (Aldar, Emaar, Binghatti), QNB/Qatar bank products, Oman, Kuwait, Morocco, ADGM platforms, RWA.xyz data and XRPL explorers were not verified.
- **On-chain verification impossible:** DLD/Ctrl Alt XRPL issuer addresses and token type (trust-line IOU vs Multi-Purpose Token) are unpublished →01 →02.
- **Vendors unknown:** KYC vendors for PRYPCO, Ctrl Alt, SmartCrowd; cloud providers for all except Saudi (in-Kingdom K8s).
- **Saudi legal status in Sep 2026:** final tokenisation law / executive regulations (expected mid-2026) and CMA's role remain unconfirmed →04.
- **Pending launches:** Stake and SmartCrowd tokenised products; DLD Phase 3 / non-resident access; Ctrl Alt second distributor.
- **Data inconsistencies** flagged per file (PRYPCO min ticket: product page AED 1,000 vs T&C/help centre AED 2,000, with LenderKit attributing the AED 1,000 to the secondary market; PRYPCO age: T&C 18 vs help centre 21; Stake rent AED 31m vs 55m; SmartCrowd exits 40+ vs 77).
- **Reconciled at QA (23 Sep 2026):** Ctrl Alt's UAE entity is one company — "Ctrl Alt Solutions FZCO", DMCC company no. DMCC199928, styled "Ctrl Alt Solutions DMCC" on the VARA register →02; the KSA non-Saudi ownership system entered into force on **22 Jan 2026** per REGA (not 21 Jan) →04; the UAE SCA became the **Capital Market Authority (CMA)** on 1 Jan 2026 (FDL 32 & 33 of 2025) — older "SCA" citations in →01 are pre-rename.
- **Blocked fetches:** thefintechtimes.com (403), blogs-old.prypco.com (robots timeout); JS-only pages (tokenizer.estate news, naukrigulf).

## 7. Files
- `01-prypco-mint.md` · `02-ctrl-alt.md` · `03-stake.md` · `04-saudi-rer-national-tokenization.md` · `05-smartcrowd-nawy-shares.md` · `_search-log.md` · this file.
- All 20 mermaid diagrams (4 per platform) were test-rendered with mermaid-cli and render without errors.
