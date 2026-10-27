# United States — Platform Selection & Regional Summary

Research date: 23 Sep 2026 · Region: United States · Researcher: Claude (US regional agent)
Evidence labels: **[C#n]** confirmed (source n in §10), **[I]** inferred (reasoning given), **[S]** speculative. Log IDs in the form (S5), (L13) refer to rows of `_search-log.md`.

> **Headline:** the US on-chain fractional real-estate market was **reshaped in 2025–26 by RealT's collapse**:
> - Detroit lawsuit filed Jul 2025, distributions suspended Feb 2026, court fiduciary appointed Apr 2026, voluntary liquidation announced 2 Jul 2026 [C#1][C#2][C#33].
> - Lofty is now the largest *live* retail on-chain marketplace, at ~$101M TVL on Algorand [C#18].
> - Most other early US tokenisers are dormant, acquired or pivoted: HoneyBricks was acquired and its token model shelved [C#13]; Roofstock onChain shows no activity after 2022 [C#7]; Homebase shows no activity after 2023 (S17); Propy moved into title-company roll-ups [C#16].
> - The durable US compliance template is still **off-chain** (Arrived's Reg A+ series-LLC/REIT model) [C#21].

---

## 1. Selection method

**Universe.** Candidate longlist from the brief, plus candidates found in searches (HoneyBricks, Ark7, Mogul, Landshare, RWA.xyz-tracked US issuers). All candidates were checked for **Sep-2026 status**.

**Gate.** The platform is investor-facing and US-focused, and offers **fractional economic ownership of real property** (equity/LLC/fund interests). The following were excluded from the Top 5 but kept as notes:
- Pure infrastructure: Securitize, tZERO.
- Debt-only products: Figure HELOC.
- Synthetic price exposure: Parcl.
- Title/deed transaction platforms: Propy, Balcony.

**Preference rule.** On-chain token issuers are preferred. **At most one off-chain benchmark** is allowed in the Top 5, and only with justification.

**Scoring rubric (0–5 per criterion, weighted):**

| Criterion | Weight | What earns a 5 |
|---|---|---|
| C1 On-chain tokenisation depth | 25% | Tokens are the legal register or security, on a public chain, with on-chain settlement and distributions |
| C2 Verified scale (AUM/TVL, properties, investors) | 25% | ≥$100M verified, ≥100 properties, ≥20k investors |
| C3 Regulatory standing | 15% | Registered/qualified offering path + regulated BD/ATS/TA; no adverse actions |
| C4 Secondary-market liquidity | 10% | Continuous, functioning secondary venue |
| C5 Longevity / track record | 10% | 5+ years of operation, exits, distributions |
| C6 Status & momentum (Sep 2026) | 15% | Live, growing, no distress |

## 2. Scored longlist

| # | Candidate | Category | Status Sep 2026 | C1 | C2 | C3 | C4 | C5 | C6 | **Score** | Decision | Key evidence |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | **Lofty** | On-chain (Algorand ASA) | Live | 5 | 3.5 | 2 | 4.5 | 4 | 5 | **4.03** | **#1** | TVL $101M [C#18]; 180+ props [C#19]; no SEC filing; CA DFPI order [C#32] |
| 2 | **RedSwan** | On-chain CRE (Hedera/Stellar) | Live | 4 | 3 | 4.5 | 1 | 4 | 4 | **3.53** | **#2** | FINRA BD [C#6]; 506(c)+Reg S [C#22]; $170M + $89M offerings (02-redswan §0); no secondary [C#22] |
| 3 | **Arrived** | Off-chain Reg A+ | Live, growing | 0 | 5 | 4.5 | 3 | 4 | 5 | **3.38** | **#3 (benchmark)** | 885k sign-ups, $300M+, 550+ props [C#9]; 1-K FY2025 [C#21]; WA DFI order [C#31] |
| 4 | **RealT** | On-chain (Gnosis/Ethereum) + DeFi | **Liquidating** | 5 | 4 | 1 | 4 | 4 | 0.5 | **3.28** | **#4 (cautionary)** | ~$140M raised, ~700 props [C#1]; 17 Form Ds [C#25]; Detroit suit [C#33] |
| 5 | Ark7 | Off-chain Reg A+ (PPEX ATS) | Live | 1 | 3 | 5 | 3 | 3 | 5 | 3.10 | Runner-up (2nd off-chain; benchmark slot already used) | Reg A+, Dalmore BD, PPEX ATS, $30M+, 300k investors [C#12] |
| 6 | Landshare | On-chain DeFi hybrid (BNB Chain) | Live | 5 | 1 | 1 | 4 | 3 | 3.5 | 2.88 | Runner-up (see tie-break) | LSRWA DEX liquidity + KYC primary mint at NAV [C#11]; primary site blocked (S43, S45) |
| 7 | **mogul** | On-chain register (Avalanche) | Live, fast-growing, some strain | 3.5 | 3.5 | 1 | 0 | 2 | 4.5 | **2.78** | **#5 (tie-break over Landshare)** | $90M assets, 40k users [C#17]; Avalanche + Fireblocks (05-mogul); investment-club structure [C#17] |
| 8 | Propy | Title/escrow + deed NFTs | Live, pivoted to title roll-up | 3 | 4* | 3 | 0 | 4 | 4 | n/a | Excluded (not fractional) | $100M facility, title-firm acquisitions [C#16]; "title platform, not a securities issuer" [C#3] |
| 9 | HoneyBricks | On-chain (Polygon) Reg D | Acquired Apr 2024; token model shelved | 4 | 1 | 3 | 1 | 2 | 0 | 2.00 | Excluded (dormant) | EquityMultiple acquisition [C#13] |
| 10 | Roofstock onChain | Whole-home NFTs (Ethereum, LLC per home) | Dormant (no 2024–26 updates) | 4 | 1 | 2 | 1 | 2 | 0.5 | 1.93 | Excluded | [C#7]; 2022 USDC NFT sale (S11) |
| 11 | Homebase | Solana SFR tokens | No activity found after 2023 | 4 | 0.5 | 1 | 1 | 1 | 0 | 1.48 | Excluded | 2023 coverage only (S17) |
| 12 | Securitize (RE issuances) | Tokenisation infra (TA/BD/ATS) | Live | 5 | — | 5 | 3 | 4 | 5 | n/a | Excluded as infra; see `03-infra-and-admin-evidence/` | Q1 2026 results (S29); one RE asset $16.3M on RWA.xyz [C#5] |
| 13 | tZERO (Aspen Digital) | ATS | Live (Aspen token 2020-era) | 4 | — | 5 | 2 | 4 | 3 | n/a | Excluded as infra | (S30) |
| 14 | Fundrise | Off-chain eREITs/funds | Live (not re-verified) | 0 | 5 | 5 | 1 | 5 | 5 | n/a | Benchmark only (not researched this session) | [S] |
| 15 | Figure | Tokenised HELOC debt | Live (not re-verified) | — | — | — | — | — | — | n/a | Out of scope (debt) | [S] |
| 16 | Parcl | Synthetic price perps (Solana) | Live; Polymarket housing-index partnership | — | — | — | — | — | — | n/a | Out of scope (no property ownership) | (S41) |
| 17 | Tangible | RWA real estate + USDR stablecoin | Collapsed (not re-verified this session) | — | — | — | — | — | 0 | n/a | Cautionary note only | [S], from prior knowledge: the USDR depeg in Oct 2023. Unverified here because the search budget was exhausted |
| 18 | Balcony / Homium / Fabrica | Deed tokenisation / shared-appreciation / land NFTs | Not verified | — | — | — | — | — | — | n/a | Not assessable | No useful search results (S31) |
| 19 | Elevated Returns / Aspen Digital | Single-asset token (Tezos) | Legacy | 4 | 1 | 4 | 1 | 4 | 1 | 2.50 | Excluded (single legacy asset) | (S30) |
| 20 | Republic RE / Blocksquare US marketplaces / Kin-Hivemind | — | Not verified | — | — | — | — | — | — | n/a | Not assessable | Search budget exhausted before these could be checked |

\* Propy's scale reflects transactions facilitated, not fractional AUM.

**Tie-break for #5 (mogul 2.78 vs Landshare 2.88).**
- mogul was selected despite its slightly lower score, for three stated reasons:
  1. Its **verified scale is roughly an order of magnitude larger** ($90M assets and 40k users [C#17], versus no verifiable Landshare metrics; the Landshare scale score is 1 because figures are unknown, not because they were measured as small).
  2. Landshare's primary sources were **blocked or JS-only** (S43, S45), so a deep-dive would have been low-confidence.
  3. mogul's **US-retail residential model** is directly comparable to a likely RealX offer.
- Landshare is the top on-chain runner-up (§5).

**Why RealT is in the Top 5 despite liquidating.**
- It is the largest US on-chain real-estate tokeniser ever built by properties and holders, and it has the most complete DeFi stack (RMM lending, YAM, AA wallet, governance token).
- Its failure mode is the single most instructive case for RealX.
- It scores #4 by rubric because the status criterion carries only 15%.

**Why Arrived (off-chain) is in the Top 5.**
- It is the US scale leader in retail single-property fractional ownership.
- It uses the **reference compliance stack a tokenised entrant must replicate**: Reg A+ Tier 2 series LLC, a per-series REIT election (1099-DIV), NCPS BD, PPEX ATS secondary, Modern Treasury custody of funds, and blue-sky filings [C#21].
- The live on-chain field is too thin to fill five slots with healthy platforms.

## 3. Final Top 5

| Rank | Platform | File | One-line rationale |
|---|---|---|---|
| 1 | Lofty | `01-lofty.md` | Largest live US on-chain retail marketplace: $50 tokens, daily USDC rent, order book + PMM liquidity pools, DAO-LLC governance. Its legal basis is undisclosed |
| 2 | RedSwan | `02-redswan.md` | Most complete regulated tokenised-CRE stack: affiliated FINRA BD + RIA, 506(c)/Reg S, registrar-as-truth, Hedera/Stellar. No secondary yet; inflated AUM claims |
| 3 | Arrived | `03-arrived.md` | Off-chain benchmark: Reg A+ Tier 2 series LLC with per-series REIT, NCPS/PPEX ATS secondary, 885k sign-ups |
| 4 | RealT | `04-realt.md` | Most advanced on-chain + DeFi design (Gnosis, RMM, YAM, AA wallet). Now in liquidation after Detroit nuisance litigation |
| 5 | mogul | `05-mogul.md` | Fast-growing Avalanche-register retail platform (Fireblocks, fiat-first UX) resting on a risky "investment club" legal theory |

## 4. Regional comparison table

| Platform | Category | Chain(s) | Token standard | Legal wrapper | Exemption / licence | KYC vendor | Wallet / custody model | Fiat & stablecoin rails | Secondary market mechanism | Distribution cadence & currency | User roles | Key admin modules |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **Lofty** | On-chain P2P marketplace | Algorand | ASA per property | Wyoming DAO LLC per property (C-corp tax election inferred from 1099-DIV) | **None disclosed** for retail; not BD/ATS; CA DFPI order 2022; no SEC filing attributable to Lofty AI (the "Lofty Ventures Syndicate" Form Ds belong to an unrelated VC syndicate, see 01-lofty §2.3) | Undisclosed (doc + selfie) | App-generated Algorand wallet described as non-custodial; Pera/Defly | Stripe card/ACH, wire; USDC (Algorand) | 24/7 limit order book + per-property PMM liquidity pools (≤20% supply) | **Daily**, USD or USDC | Investor, LP (no KYC), property seller (≥10% retained), PM, governance voter | Listing pipeline, ASA token control, PMM ops, daily distribution, governance timers, 15%-cap monitor, 1099/1042-S |
| **RedSwan** | On-chain institutional CRE | Hedera, Stellar (earlier Polymath/Ethereum) | HTS tokens / HSCS ERC-20-compatible / Stellar assets | Sponsor SPVs and funds (LLC/LP) | **Reg D 506(c)** + **Reg S**; Rule 144; affiliated **FINRA BD** (RedSwan Markets); RIA (RedSwan PC); 3(c)(1)/(7) fund | Undisclosed (KYC + accreditation + OFAC vendors) | Custody via Coinbase + KYC-gated accounts | USD wire/escrow; USDC on Hedera | **None active** ("controlled secondary venues" planned) | Programmable; cadence undisclosed (quarterly/K-1 inferred) | Accredited investor, Reg S investor, institution, sponsor, registered rep, registrar, custodian | Offering manager, accreditation queue, BD compliance, Token Studio (HTS keys), registrar reconciliation, escrow, distributions |
| **Arrived** | Off-chain fractional | None | None (book-entry) | Delaware **series LLC**, each series a **REIT** | **Reg A+ Tier 2** (1-A/1-K/1-SA); **NCPS** BD; **PPEX ATS**; blue-sky filings; WA DFI order 2025 | Unnamed vendor (likely via NCPS/NCIT) + Plaid | Book-entry at transfer agent; investor funds held by **Modern Treasury** | Plaid, Stripe, Dwolla, NCIT ACH; no stablecoin | Monthly one-week windows, limit orders, PPEX ATS, 6-month hold, T+1/T+2 | Monthly or quarterly, USD to Arrived Wallet | Retail investor, fund investor, manager, NCPS/ATS, PM | Offering/Tier-2 cap tracker, blue-sky engine, NAV committee, dividends/REIT test, window ops, 1-K/1-SA |
| **RealT** | On-chain + DeFi (liquidating) | Gnosis, Ethereum | ERC-20 transfer-restricted (whitelist) | Delaware Series LLC, later per-property **Delaware Inc.** (common shares); 165 entities | **Reg D 506(c)** (17 Form Ds, 2022) + **Reg S** (non-US only since 2022); 2,000-holder 12(g) cap | Undisclosed | External wallets or **RealToken AA wallet** (social login, RealT Guardians) | Card (1.5%), Request Finance crypto, REUSD vouchers; USDC/xDai | **YAM** P2P offers contract, Levinswap/Swapcat DEX, **RMM** (Aave v3 fork) lending | **Weekly**, USDC/xDai or reinvest (suspended 2026) | Non-US investor, DeFi borrower/lender, whitelisted liquidator, YAM trader, REG holder | Whitelist manager + holder-cap, weekly distribution, RMM risk/oracle, YAM whitelist, REG/CCIP admin, **missing: municipal compliance, title reconciliation** |
| **mogul** | On-chain register, fiat-first | Avalanche C-Chain | ERC-1155-compatible property NFT | PropCo LLC per property (partnership) as "investment club"; OA names Avalanche as ownership record | **No SEC filing**; investment-club theory; not BD/RIA | Unnamed (SSN-based) | **Custodial via Fireblocks** | Plaid ACH, card, wire, PayPal, Venmo; **USDT on Avalanche** | None ("coming soon"); transfers need Mogul consent | **Monthly** (2nd Tuesday), USD to mogul wallet | Investor, Chief Mogul (club founder), club member, PM, manager | Fireblocks mint policy, reserves, monthly distributions, votes (default-to-manager), club rewards (2%), K-1s |

## 5. Runners-up & benchmarks (short notes)
- **Ark7:** Reg A+ (Ark7 Properties Plus LLC, CIK 1923734), Dalmore BD, **PPEX ATS** secondary after a 1-year lockup, $20 minimum, 300k+ investors, ~4% yields in 2026. It acquired DeFi tokeniser Robinland in 2023, but no live tokens were found [C#12]. It is the closest off-chain analogue to Arrived and uses the same ATS.
- **Landshare (BNB Chain):**
  - A dual-track model: **permissionless DEX trading of LSRWA** alongside a **KYC-gated primary mint at NAV** [C#11].
  - The 2026 roadmap adds a stablecoin "Real Assets Vault" lending pool, monthly USD cash-flow claims and a RWA launchpad [C#11].
  - It is the most DeFi-native US model, but its legal basis and scale are unverified.
- **Propy:** on-chain deed NFTs and title/escrow, now a **$100M title-company roll-up with an AI escrow agent** [C#16]. It is relevant as a closing and title integration partner, not as a fractional issuer.
- **HoneyBricks:** Polygon + USDC, Reg D 506(b)/(c), 90-day lock, peer-to-peer tokens. Acquired by EquityMultiple (Apr 2024) and the token model was **discontinued** [C#13]. A cautionary case: tokenisation did not survive acquisition by a traditional platform.
- **Roofstock onChain:** whole-home NFTs, with each home in its own LLC on Ethereum [C#7]. No 2024–26 activity; effectively dormant [I].
- **Securitize / tZERO:** regulated infrastructure (TA, BD, ATS). RWA.xyz shows only one Securitize real-estate asset ($16.3M) [C#5]. Treated as vendors rather than platforms; see `03-infra-and-admin-evidence/`.
- **Parcl** (synthetic price exposure) and **Figure** (HELOC debt): out of scope.
- **Tangible:** the collapse is noted from prior knowledge only [S]. It was not re-verified this session.

## 6. US compliance map (per platform)

| Angle | Lofty | RedSwan | Arrived | RealT | mogul |
|---|---|---|---|---|---|
| Reg D 506(b)/(c) | **None** (QA correction: the "Lofty Ventures Syndicate, LP" Form Ds are an unrelated Seattle VC syndicate, not Lofty AI) | **506(c)** with third-party accreditation | — | **506(c)** (16) + 506(b) (1), 2022 | — |
| Reg S | Not stated (sells globally) | **Yes** | No (US only) | **Yes** (non-US only since 2022) | No (US only) |
| Reg A+ Tier 2 | — | — | **Yes** (1-A/1-K/1-SA/253G2) | — | — |
| Reg CF | — | Company raise (Form C); BD described as Reg CF-licensed | — | — | — |
| Rule 144 / lockups | None disclosed | **Rule 144 holding periods** | 6-month secondary hold | None (whitelist only) | Transfers need consent |
| EDGAR filings | None attributable to Lofty AI | Form D + Form C | 176 1-A, 55 1-K… (full-text hits) | 17 Form D | None found |
| Transfer agent | On-chain ASA ledger (no TA) | Registrar (unnamed) | Third-party TA (unnamed) | None disclosed | Chain designated as register by the OA |
| BD / ATS / portal | None | Affiliated BD | NCPS BD + PPEX ATS | None | None |
| Blue-sky | CA DFPI consent order (2022) | Not observed | Filings; **WA DFI order (2025)** | — | — |
| FinCEN / BSA | Non-custodial stance [I] | BD AML programme | NCIT/Modern Treasury hold funds | Unclear | Holds balances + USDT/PayPal: MSB risk [I] |
| OFAC | Sanctions reps | OFAC screening | KYC vendors | Unclear | Unclear |
| Tax forms | **1099-DIV/1099-B** | K-1 inferred | **1099-DIV (REIT)** inferred | Annual "Rent Distribution Report" (dividend vs return of capital) | **K-1** |
| Foreign withholding / FIRPTA | 30% / 15% treaty | §1446 + FIRPTA inferred | n/a (US only) | FDAP on dividends; **FIRPTA on USRPHC share sales unaddressed** [I] | n/a |
| Entity | Wyoming DAO LLC | SPV/LP | DE series LLC + REIT | DE series LLC → DE Inc. | PropCo LLC (partnership) |
| Controversies | DFPI; St. Louis squatters | Inconsistent AUM claims | WA DFI; secondary complaints | **Detroit v. RealT** (Jul 2025), fiduciary, liquidation, French class action | Investment-club risk; low Trustpilot |

## 7. Regional patterns (US)

1. **Property operations, not smart contracts, are the failure point.**
   - RealT's contracts (YAM) still work while the portfolio is in court-fiduciary liquidation [C#1].
   - Lofty faces St. Louis squatter and board-up issues (01-lofty §10).
   - Municipal enforcement (nuisance suits, rent-to-escrow orders, tax foreclosure) can switch off on-chain yield overnight.
   - RealX needs a **municipal-compliance and title-verification module** as a first-class product.
2. **Three legal postures co-exist:**
   - (a) **Fully regulated:** Arrived (Reg A+ + BD/ATS) and RedSwan (506(c) + BD).
   - (b) **Exemption-light / undisclosed:** Lofty (no filing), mogul (investment club).
   - (c) **Offshore-oriented:** RealT (Reg S, US persons geo-blocked).
   - Only (a) has scaled cleanly with institutions. (b) carries state-regulator risk (CA DFPI vs Lofty), and (c) did not shield RealT from US municipal courts.
3. **Tax-wrapper engineering drives UX.** Platforms converge on **corporate-style wrappers so retail investors get 1099s instead of K-1s**: Arrived through a per-series REIT election [C#21]; Lofty's 1099-DIV implies a corporate election [I]; RealT through Delaware Inc. issuers [C#25]. mogul (K-1) is the outlier.
4. **The entity-per-property pattern is universal**: series LLC, DAO LLC or Delaware Inc. This produces **entity sprawl** (RealT had 165 entities) and needs an entity-registry admin module.
5. **Secondary liquidity takes three forms:**
   - Regulated **periodic ATS windows**: Arrived and Ark7 on PPEX.
   - **Unregistered on-chain venues**: Lofty's order book + PMM; RealT's YAM, DEXs and lending.
   - **None**: RedSwan, mogul.
   - No US platform yet offers continuous trading on a registered ATS with on-chain T+0 settlement. This is a clear gap for RealX.
6. **Chain choice follows cheap, permissioned-friendly L1s** with native compliance primitives: Algorand ASA clawback/freeze, Hedera HTS KYC keys, Stellar authorisation flags, Gnosis for low fees, and Avalanche with Fireblocks. **No US leader uses ERC-3643.** RealT used a bespoke whitelist; standards adoption is low.
7. **Custody splits into two models:**
   - "Non-custodial by assertion": Lofty's app-generated wallet; RealT's AA wallet with issuer Guardians.
   - Custodial: mogul via Fireblocks; RedSwan via Coinbase.
   - Stablecoin rails are mostly USDC; mogul uses USDT and RealT uses xDai.
8. **Marketing and verified numbers diverge widely**:
   - RedSwan claims $4–5B, with about $100M+ corroborated.
   - mogul's 18.8% IRR has no exits behind it.
   - Lofty markets ~11% vs 9.2% actual [C#19].
   - For RealX, audited, on-chain-verifiable metrics are a real differentiator.

## 8. 2025–26 US regulatory context (limited; see `01-regulatory/`)
- **SEC statement on tokenised securities** distinguishing issuer-sponsored from third-party tokenisation: a "tokenized security is still a security", as cited by RedSwan [C#24]. The date and document number were not verified.
- **SEC proposed "Regulation Crypto Assets"** (18 Aug 2026; comments due 20 Oct 2026), per RedSwan's summary [C#23]. RedSwan says it does not govern tokenised securities and does not resolve ATS or exchange registration for secondary venues.
- **Not verified this session** (search budget exhausted): the GENIUS Act stablecoin regime, the CLARITY Act market-structure status, any SEC innovation exemption, and state actions beyond CA DFPI (Lofty) and WA DFI (Arrived). These are deferred to the regulatory agent.
  > **QA note (Fable):** these items are now confirmed against primary sources in `01-regulatory/us.md` (SEC staff statement 28 Jan 2026; GENIUS Act signed 18 Jul 2025, effective the earlier of 18 Jan 2027 or 120 days after final rules; CLARITY Act cloture failed 49–50 on 15 Sep 2026; innovation exemption for tokenized NMS stock only, 17 Sep 2026; Texas SSB emergency order v. E-Estate Group, 1 Jun 2026). Treat `01-regulatory/us.md` as authoritative for §8.

## 9. Gaps & uncertainties
- **WebSearch budget exhausted mid-study.** The session-wide 200-call limit was hit during Platform research. All later work used WebFetch on known URLs, official docs, GitHub and SEC EDGAR full-text search. As a result:
  - FINRA BrokerCheck was not checked.
  - Block-explorer contract verification failed (JS-rendered explorers). Token configurations (ASA clawback/freeze, RealToken compliance contract, mogul NFT contract) are **[I]/[S]**.
  - KYC vendors are **undisclosed for all five platforms** (privacy policies name only payments and analytics vendors).
- **Lofty:** legal basis for retail sales is undisclosed; the DFPI consent-order PDF was blocked (HTTP 403); the listing fee conflicts (3% vs 5%); custodial vs non-custodial wallet claims conflict.
- **RealT:** realt.co is geo-blocked for US fetches; the community token API was blocked by robots.txt; the Mt Pelerin Bridge-protocol lineage of RealToken contracts is **[S]**; final liquidation recoveries are unknown.
- **RedSwan:** AUM claims are unverifiable; asset-level SPV Form Ds were not located; ToS and website disagree on RIA status.
- **Arrived:** the secondary-window cadence differs between the 1-K (quarterly) and the product (monthly); the KYC vendor is unnamed.
- **mogul:** no SEC filings under "mogul Technologies" (other entity names not searched); US-only vs "non-US ID" inconsistency; NFT contract address not verified.
- **Not assessed:** Balcony, Homium, Fabrica, Republic real estate, Kin/Hivemind, and Blocksquare-based US marketplaces. Tangible's collapse is noted from prior knowledge only.

## 10. Sources (for this summary; platform files carry their own numbered sources)
1. Crypto Briefing — RealT liquidation (28 Jul 2026) — https://cryptobriefing.com/realt-crypto-real-estate-collapse/
2. Michigan Public — RealT crumbling (20 Mar 2026) — https://www.michiganpublic.org/social-justice/2026-03-20/the-model-no-longer-works-crypto-landlords-detroit-enterprise-is-crumbling
3. eco.com — RealT, Lofty, Propy compared — https://eco.com/support/en/articles/15254024-tokenized-real-estate-2026-realt-lofty-propy-compared
4. RWA.xyz — Real estate dashboard announcement (21 Jan 2026) — https://app.rwa.xyz/blog/announcing-real-estate-dashboard
5. RWA.xyz — Real estate league table — https://app.rwa.xyz/real-estate
6. Weekly Real Estate News — RedSwan FINRA membership (21 May 2024) — https://wrenews.com/tokenization-platform-redswan-cre-secures-finra-membership/
7. TokenizeList — Roofstock onChain — https://tokenizelist.com/platforms/roofstock-onchain/
8. Blockworks — Mogul on Avalanche (29 May 2025) — https://blockworks.com/news/tokenized-real-estate-app-mogul
9. Arrived — Series B / secondary market (6 Nov 2025) — https://arrived.com/blog/series-b-fundraise
10. Zoniqx — Top RE tokenization platforms 2025–26 (vendor listicle, low reliability) — https://www.zoniqx.com/resources/top-real-estate-tokenization-platforms-in-2025-and-2026
11. Landshare — Roadmap Q1–Q2 2026 (Medium) — https://landshare.medium.com/landshare-roadmap-q1-q2-2026-99e9a44a8326
12. CrowdfundedWealth — Ark7 review 2026 — https://www.crowdfundedwealth.com/reviews/ark7-review
13. CrowdfundedWealth — HoneyBricks review 2026 — https://www.crowdfundedwealth.com/reviews/honeybricks-review
14. Sabai Protocol — Successful and failed tokenization projects 2026 — https://paragraph.com/@sabai-protocol/successful-and-failed-tokenization-projects-2026-update
15. Genfinity — RedSwan Hotel on Rivington (9 Oct 2025) — https://genfinity.io/2025/10/09/nyc-skyline-first-building-tokenized-redswan-digital/
16. Inman — Propy's $100M title roll-up (14 May 2026) — https://www.inman.com/2026/05/14/propy-ai-title-companies/
17. mogul — About — https://www.mogul.club/about
18. DefiLlama — Lofty — https://defillama.com/protocol/lofty
19. Coinpaprika — Lofty AI review 2026 — https://coinpaprika.com/education/lofty-ai-review/
20. CrowdfundedWealth — mogul.club review 2026 — https://www.crowdfundedwealth.com/reviews/mogul-club-review
21. SEC EDGAR — Arrived Homes, LLC 1-K FY2025 — https://www.sec.gov/Archives/edgar/data/1821720/000121390026050390/ea028816201-1k_arrived.htm
22. RedSwan — Homepage — https://redswan.io/
23. RedSwan — "The SEC Writes A New Rulebook For Crypto" (Aug 2026) — https://redswan.io/the-sec-writes-a-new-rulebook-for-crypto/
24. RedSwan — "Tokenized Securities: Same Rules, Smarter Rails" (2026) — https://redswan.io/tokenized-securities-same-rules-smarter-rails-and-where-redswan-fits-in/
25. SEC EDGAR full-text search — "RealToken" Form D — https://efts.sec.gov/LATEST/search-index?q=%22RealToken%22&forms=D
26. Crypto Briefing / RealOpen — RealT timeline — https://realopen.com/resources/tokenized-real-estate/realt
27. CrowdfundedWealth — Lofty review 2026 — https://www.crowdfundedwealth.com/reviews/lofty-review
28. Arrived — Terms of Service — https://arrived.com/terms-of-service
29. mogul — Terms of Service — https://app.mogul.club/terms-of-service
30. RealT Community Wiki — https://community-realt.gitbook.io/tuto-community
31. Angel Investors Network — Arrived review 2026 (WA DFI consent order) — https://angelinvestorsnetwork.com/real-estate/arrived-homes-review-2026
32. California DFPI — Lofty AI, Inc. enforcement action (14 Nov 2022) — https://dfpi.ca.gov/enforcement_action/lofty-ai-inc/
33. Outlier Media — Detroit sues RealT (Jul 2025) — https://outliermedia.org/realt-lawsuit-detroit-sues-crypto-landlord/
