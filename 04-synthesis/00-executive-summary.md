# Executive Summary: Fractional Real-Estate Tokenisation, 20 Platforms in 4 Regions

**For:** RealX founder · **Date:** 23 Sep 2026 · **Basis:** the 42 research files in `00-method/` to `03-infra-and-admin-evidence/`, plus six re-check fetches (F1–F6) made during synthesis to settle contradictions between files. No new web research was done.

> **Evidence key (applies to all `04-synthesis/` files).** Each label is carried over from its source file and never upgraded: **[C]** confirmed, **[I]** inferred, **[S]** speculative. After the label comes the source file and section, e.g. `[C us/01 §2.3]` = `02-platforms/us/01-lofty.md` §2.3.
> - **Platform codes.** us/01 Lofty · us/02 RedSwan · us/03 Arrived · us/04 RealT · us/05 mogul · eu/01 Reental · eu/02 Blocksquare(+Oceanpoint) · eu/03 Propvest/Exporo · eu/04 Brickken · eu/05 Domoblock · me/01 PRYPCO Mint · me/02 Ctrl Alt · me/03 Stake · me/04 Saudi REGA/RER + sandbox operators · me/05 SmartCrowd/Nawy Shares · au/01 BrickX · au/02 Redbelly · au/03 Tokeniser+Liquidise · au/04 Canvas Connect · au/05 Assetora(DomaCom)+Bricklet.
> - **Other codes.** `<region>/sum` = the regional `_selection-and-summary.md`. `reg/us|eu|me|au|x` = `01-regulatory/` (x = cross-cutting-controls). `inf/std|ven|wl|adm` = `03-infra-and-admin-evidence/` (token-standards, vendor-landscape, white-label-issuer-dashboards, admin-panel-evidence).
> - **Synthesis codes.** `[I syn]` marks an inference made in this synthesis. `F1`–`F6` are the re-check fetches, logged in `conflicts-and-gaps.md` §4.

---

## 1. The 20 platforms at a glance

| # | Platform (HQ) | Archetype (§2) | What the investor legally holds | Chain · custody | Min ticket | Secondary liquidity | Scale & status, Sep 2026 |
|---|---|---|---|---|---|---|---|
| 1 | **Lofty** (US) | A1 DeFi-native | Membership interest in a per-property Wyoming DAO LLC, as an Algorand ASA [C us/01 §2.2] | Algorand · app wallet that the ToS calls non-custodial [C us/01 §7] | ~$50 [C us/01] | 24/7 order book plus per-property PMM pools capped at 20% of supply [C us/01 §9] | TVL ~$101M, 40k+ investors [C us/01]. Live. CA DFPI order in 2022 [C us/01 §2.3] |
| 2 | **RedSwan** (US) | A2 Regulated security-token | Equity or LP interest in sponsor SPVs/funds. The registrar's ledger is authoritative [C us/02 §2.2] | Hedera, Stellar · Coinbase custody or self-custody [C us/02 §2.4] | Not disclosed | None active. Rule 144 holding periods apply [C us/02 §9] | ~$100M+ corroborated, against $4–5B claimed [C us/02]. Live |
| 3 | **Arrived** (US) | A4 Off-chain benchmark | Interests in a Delaware series LLC, with each series electing REIT status [C us/03 §2.1, §2.7] | No chain · book entry at a transfer agent; investor funds held at Modern Treasury [C us/03 §2.4] | $100 | Monthly one-week windows on PPEX ATS after a 6-month hold [C us/03 §9] | 885k sign-ups, $300M+ invested, 550+ properties [C us/03]. Growing |
| 4 | **RealT** (US) | A1 DeFi-native | Common shares of a per-property Delaware corporation (earlier: series LLC units) [C us/04 §2.1] | Gnosis, Ethereum · external or account-abstraction (AA) wallet [C us/04 §7] | ~$50 | YAM peer-to-peer (P2P) contract, DEXs, RMM lending [C us/04 §5.4] | ~$140M raised, ~700 properties. **Voluntary liquidation announced 2 Jul 2026** [C us/04; F6] |
| 5 | **mogul** (US) | A7 Outside perimeter | Membership in a PropCo LLC run as an "investment club". The operating agreement names Avalanche as the register [C us/05 §2.2] | Avalanche · custodial via Fireblocks [C/I us/05 §7] | $250 | None ("coming soon") [C us/05 §9] | $90M assets, 40k users [C us/05]. Signs of strain |
| 6 | **Reental** (ES) | A1 DeFi-native | Spanish deals: a share of a participative loan to the SPV. US deals: an LLC interest [C eu/01 §2.2] | Polygon ERC-20 [I eu/01 §7] · non-custodial passkey smart wallet [C eu/01 §7] | €100 in USDT; €500 by bank | RNT P2P board (KYC'd users) plus RNTlend loans up to 75% LTV [C eu/01 §9] | 43.7k users, 123 projects, US$110m+ tokenised [C eu/01]. Growing |
| 7 | **Blocksquare + Oceanpoint** (SI) | A6 White-label protocol | Contractual royalty on issuer revenues ("economic rights"). No SPV [C eu/02 §2.1–2.2] | Ethereum, custom whitelisted ERC-20 · Alchemy AA wallets [C eu/02 §7] | Set per operator (~€100 [S]) | P2P across partner marketplaces; Oceanpoint pools [C eu/02 §9] | 172 properties; US$216.9m tokenised *value* (valuations, not capital raised) [C eu/02] |
| 8 | **Propvest / Exporo** (DE) | A2 Regulated security-token | Subordinated token-based bonds of per-property GmbH SPVs, issued under a CSSF prospectus [C eu/03 §2.2–2.3] | Ethereum · Tangany key custody, Baader depot [C eu/03 §7] | €25 | Brokered bilateral Handelsplatz. Offers at ~75–80% of par in 2025 [C eu/03 §9] | ~€250m AUM, 16k investors [C eu/03]. Under performance pressure |
| 9 | **Brickken** (ES) | A6 White-label protocol | Whatever the issuer defines: equity, income rights, loans, bonds [C eu/04 §2.2] | Multi-EVM ERC-20 with whitelist (ERC-7943 co-author) · non-custodial [C eu/04 §7] | Issuer-set | Whitelisted transfers; partner venues [C eu/04 §9] | US$660m+ issuer-declared TTV (Sep 2026 site); at end-2025 the like-for-like figures were ~US$450m TTV against ~US$41m on-chain TVL [C eu/04 hdr, §10] |
| 10 | **Domoblock** (ES) | A7 Outside perimeter | A participative loan to Domoblock itself. The "fan token" is an accounting mirror that carries no rights [C eu/05 §2.2] | Polygon · Mangopay e-money wallet, plus MetaMask for DAI [C eu/05 §7] | €200 | None; exit only by loan assignment [C eu/05 §5.4] | €59m financed, 3,412 investors [C eu/05] |
| 11 | **PRYPCO Mint** (AE) | A3 Registry-anchored | Direct fractional title in the investor's name at DLD, plus a DLD Token Ownership Certificate [C me/01 §2.1–2.2] | XRPL · Ctrl Alt warm MPC omnibus on Ripple Custody [C me/01 §7] | **AED 1,000 now** (AED 2,000 at launch) [C F4; C me/01] | 24/7 listing board; 3-month lock-in; price within ±15% of DLD valuation; 20% holder cap [C me/01 §9; F4] | ~10 properties, AED 18.5m in Phase 1 [C me/01]. Live |
| 12 | **Ctrl Alt** (AE/UK) | A3 + A6 | Issues the DLD title tokens and the ARVA management tokens (dual-token design) [C me/02 §2.2] | XRPL for DLD; Solana · Ripple Custody MPC [C me/02 §7] | B2B | Settlement layer for distributors [C me/02 §9] | US$1.8bn+ across asset classes; DLD share ~US$5m [C me/02] |
| 13 | **Stake** (AE/SA) | A4 Off-chain benchmark | UAE: DIFC SPV shares. KSA: units in CMA-regulated funds [C me/03 §2.2] | No chain (VARA in-principle approval (IPA) to tokenise) · custodial AED/SAR wallets at Emirates NBD [C me/03 §7] | AED 500 | Exit Windows in May and Nov after a 1-year lock-in [C me/03 §9] | 2m+ users, AED 1.4bn in transactions [C me/03] |
| 14 | **Saudi REGA/RER + sandbox operators** (SA) | A3 Registry-anchored | Fractional deeds in investors' names in the Real Estate Registry: one deed token plus fractional tokens [C me/04 §2.1] | Permissioned EVM on SettleMint DALP [I]; "enhanced ERC-3643" [C me/04 §7] | SAR 1,000 (Jozo) | National marketplace still in development; operator-level resale [C me/04 §9] | Regulated pilot; 9 sandbox operators [C me/04; F2] |
| 15 | **SmartCrowd / Nawy Shares** (AE/EG) | A4 Off-chain benchmark | DIFC SPV shares (1m shares per SPV). Egypt: co-ownership moving to FRA-licensed funds [C me/05 §2.2] | No chain (VARA IPA) · custodial eWallet [C me/05 §7] | AED 500 | Share Transfer Facility in Mar and Sep, within ±15% of market price [C me/05 §9] | US$110m+ transactions, 77 exits [C me/05] |
| 16 | **BrickX** (AU) | A5 MIS wrapper | A unit (1/10,000) in a per-property trust inside a registered MIS [C au/01 §2.2] | No chain · custodial wallet; AET is custodian [C au/01 §7] | A$50 | Continuous internal order book; sell floor ≥80% of valuation; 5% holder cap [C au/01 §9] | ~25 properties, ~A$53m funded [C au/01] |
| 17 | **Redbelly Network** (AU) | A6 Infra L1 | Network only; the issuer defines the instrument [C au/02 §2.1] | Redbelly L1 with verifiable credentials and ZK proofs · Fireblocks [C au/02 §7] | n/a | Credentialed P2P; Liquidise AMM [C au/02 §9] | ~US$1bn RWA per RWA.io, against a US$73.8bn claim [C au/02] |
| 18 | **Tokeniser + Liquidise** (AU) | A5 + A6 | Units or shares in the issuer's own trust/SPV, as ERC-3643 tokens [C au/03 §2.2] | Redbelly (+ ETH, Arbitrum) · Averer non-custodial embedded wallet [C au/03 §7] | Issuer-set; wholesale by default | 24/7 AMM pools with T+0 settlement [C au/03 §9] | A$1.2B issued (self-reported), 160+ issuers [C au/03] |
| 19 | **Canvas Connect** (AU) | A2 Regulated security-token (wholesale) | Shares in an SPV that invests in a real-estate credit fund (Acacia pilot) [C au/04 §2.1] | Permissioned EVM L2 · Zodia, Fireblocks [C au/04 §7] | Wholesale only | Bilateral DvP and repo against wholesale CBDC [C au/04 §9] | Pilot scale; first live tokenised-AGB repo in wholesale CBDC (Jun 2026) [C au/04] |
| 20 | **Assetora + Bricklet** (AU) | A5 MIS wrapper | Units in a single-asset sub-fund of one registered MIS. Bricklet: co-owners on title with a DLT register [C au/05 §2.2] | No chain (Bricklet: unnamed DLT) · custodial cash; Perpetual is custodian [C au/05 §7] | A$1,000 (legacy) | List/accept board; **frozen Jul 2024–Aug 2025** [C au/05 §9] | A$200M+ FUM (self-reported); listed parent suspended on ASX [C au/05] |

## 2. Platform archetypes

The seven archetypes differ mainly on two axes:
- **Golden record:** where the legally authoritative ownership record lives.
- **Regulatory posture:** who holds the licence.

| Archetype | Members | Defining traits | Golden record | RealX relevance |
|---|---|---|---|---|
| **A1 DeFi-native public-chain marketplace** | Lofty, RealT, Reental | Issuer-run primary; stablecoin rent (daily, weekly or monthly); app or self-custody wallets; 24/7 P2P/AMM; DeFi lending; often a utility or governance token. Legal basis is undisclosed, Reg S-only, or a prospectus exemption placed through a licensed firm (ESI) [C us/01 §2.3; C us/04 §2.3; C eu/01 §2.3] | Chain, or chain plus the issuer's off-chain records | Copy the UX. Avoid the legal perimeter and the issuer-set oracles |
| **A2 Regulated security-token via licensed intermediary** | RedSwan, Propvest/Exporo, Canvas Connect | Registered BD or MiFID firm places the tokens; a registrar or custodian is the truth; chain is a mirror or back-office ledger; custodial; secondary absent or brokered/bilateral [C us/02 §6; C eu/03 §6; C au/04 §6] | Transfer agent / registrar / custodian register | Template for institutional trust and passporting |
| **A3 Registry-anchored sovereign model** | PRYPCO Mint, Ctrl Alt, Saudi REGA/RER + operators | Land registry is legal truth and the token is synced to it. Issuer licence is split from distributor licence. Fiat-only, omnibus custody. Guardrails: price bands, holder caps, lock-ins [C me/sum §5] | Government land registry | Fastest route to title-linked tokens in the GCC, as a distributor |
| **A4 Off-chain fractional SPV/series (benchmark)** | Arrived, Stake, SmartCrowd | Reg A+ series LLC or DIFC SPV per asset; cash wallet; periodic liquidity windows; transfer-agent or corporate register [C us/03 §2; C me/03 §2; C me/05 §2] | TA / corporate registry | The compliance and UX bar a token entrant must beat |
| **A5 MIS / fund-unit wrapper** | BrickX, Assetora, Tokeniser+Liquidise | Trust units per property or sub-fund. The RE or trustee holds title through a custodian. Tokenising changes the register, not the wrapper [C au/sum §5.1] | RE unit register (s169); a token only if the constitution designates it | Required wrapper for Australian retail |
| **A6 White-label protocol / infrastructure** | Blocksquare, Brickken, Redbelly (plus Ctrl Alt, Tokeniser) | Two-tier admin (protocol owner, then operator or issuer). The issuer carries compliance. Metrics often "tokenised value" rather than capital raised [C inf/wl §18; C eu/04 §10] | Whatever the issuer designates | Build-vs-buy options for the token layer |
| **A7 Outside-the-perimeter structures** | mogul, Domoblock | Legal theories built to avoid securities regimes: an "investment club", or non-transferable "fan tokens" over a loan to the operator. No investor-protection regime [C us/05 §2.3; C eu/05 §2.3] | Operator's ledger; mogul's OA designates the chain | Competitor set and anti-pattern |

## 3. Ten cross-regional findings

1. **Property operations, not smart contracts, are where platforms fail.**
   - RealT's contracts still work in liquidation. What broke was maintenance, taxes and blight, followed by a court order redirecting rent to escrow [C us/04 §10; C reg/us §9].
   - Lofty faced squatters in St. Louis [C us/01 §10]. Assetora froze its market after NDIS vacancies [C au/05 §2.8]. Propvest's bonds trade at 75–80% of par after returns fell short of targets [C eu/03 §9].
   - Token rails do not create liquidity once assets underperform.
2. **Tokenisation changes the register, not the legal wrapper. The wrapper is almost always a regulated financial product:**
   - US: a security; "format doesn't matter" (SEC staff, 28 Jan 2026) [C reg/us §1].
   - EU: a transferable security, and therefore outside MiCA [C reg/eu §1].
   - Australia: a managed investment scheme (MIS) interest (INFO 225 Example 9) [C reg/au §1].
   - Dubai: an asset-referenced virtual asset (ARVA) with DLD title [C reg/me §1].
   - The platforms that argue otherwise (A7, and Blocksquare's "MiCA-not-MiFID" thesis) carry the largest legal risk [I eu/02 §2.3].
3. **Where the golden record lives is the central architecture decision.** Three models exist in production:
   - **Chain as register:** Lofty ASA; mogul's operating agreement; Blocksquare [C us/01 §6; C us/05 §2.2].
   - **Transfer agent / registrar as truth, chain as mirror:** RedSwan; Propvest; the US TA proposal of Sep 2026, which allows a DLT master file only under the TA's exclusive control [C us/02 §6; C reg/us §5].
   - **Land registry as truth:** DLD; Saudi RER, where "settlement = registry update" [C me/04 §2.2].
   - All three need daily reconciliation between the chain and the authoritative record.
4. **Regulatory posture falls into five bands, and only the licensed ones scale with banks and institutions.**
   - The bands are: fully licensed; exemption placed through a licensed intermediary; sovereign sandbox; "not a security" or software-only; and outside the perimeter.
   - Exemption-light players drew state and national actions: CA DFPI against Lofty [C us/01 §2.3]; the Texas SSB order against E-Estate [C reg/us §11].
   - Even fully regulated Arrived was fined for blue-sky slippage [C us/03 §2.3].
   - None of the five EU token programmes uses ECSPR or the DLT Pilot Regime [C eu/sum §6.2].
5. **Nobody has solved secondary liquidity.** Four models exist:
   - Unregistered 24/7 on-chain P2P or AMM venues: Lofty, RealT, Reental, Blocksquare [C us/01 §9; C eu/01 §9].
   - Periodic windows on licensed venues: Arrived monthly on an ATS; Stake and SmartCrowd twice a year [C us/03 §9; C me/03 §9].
   - Listing boards with price guardrails: PRYPCO's ±15% band; BrickX's 80% floor [C me/01 §9; C au/01 §9].
   - No secondary at all: RedSwan, mogul, Domoblock.
   - No platform yet runs **continuous trading on a licensed venue with T+0 stablecoin settlement** [C us/sum §7.5]. That is the clearest white space.
6. **Borrowing against tokens is the emerging liquidity substitute, and it is also a risk.**
   - Examples: RealT's RMM (an Aave v3 fork), Reental's RNTlend (up to 75% LTV), Canvas's repo in wholesale CBDC, Tokeniser's leverage add-on [C us/04 §5.4; C eu/01 §9; C au/04 §5.4].
   - Oracles set by the issuer made RealT's lending reflexive once cash flows stopped [I us/04 §9].
7. **Branding is crypto, but the plumbing is fiat.**
   - Every MENA platform, every *retail* Australian platform (BrickX, Assetora) and every licensed EU platform is fiat-only and custodial [C me/sum §5.3; C au/sum §5.5; C eu/03 §7]. The Australian exceptions are wholesale: Tokeniser/Liquidise settle in USDT, USDC or LIQ through a non-custodial Averer wallet, and Canvas settles pilots in wholesale CBDC [C au/03 §7; C au/04 §7].
   - The DeFi-native players pay in USDC, USDT, DAI or xDai. No top-20 platform defaults to a MiCA e-money token or a GENIUS-permitted coin [C eu/sum §6.4].
   - Regulation now forces stablecoin allow-lists: GENIUS Act (~Jan 2027), the CBUAE payment-token rules (PTSR), MiCA, and ASIC Instrument 2025/867 [C reg/x A3 #19].
8. **Wallets converged on "no seed phrase", and recovery plus forced transfer is universal.**
   - Patterns seen: passkey ERC-4337 accounts (Reental); Alchemy account abstraction (Blocksquare); social-login AA with issuer guardians (RealT); embedded ZK wallets (Averer); custodial MPC (mogul, PRYPCO/Ctrl Alt, Propvest/Tangany) [C eu/01 §7; C us/04 §7; C au/03 §7].
   - Every issuer stack exposes freeze, forced transfer and recovery [C inf/adm headline 4].
9. **Compliance built into the token is under-used by retail leaders.**
   - No US leader uses ERC-3643 [C us/sum §7.6]. The EU leaders use custom whitelists [C eu/sum §6.6].
   - ERC-3643 shows up in infrastructure: Tokeniser; Saudi RER's "enhanced ERC-3643"; Tokeny [C au/03 §7; C me/04 §7].
   - Standards are converging on **ERC-3643 plus the ERC-7943 interface** [C inf/std §0].
10. **Tax and holding thresholds shape the product. They must be enforced as transfer rules, not in the UI.**
    - **Tax wrappers:** US REIT or C-corp elections give investors a 1099 instead of a K-1 (Arrived, Lofty, RealT; mogul is the K-1 outlier) [C us/sum §7.3].
    - **Withholding:** 19% Spanish withholding is automated [C eu/01 §2.7].
    - **Caps and eligibility:** BrickX's 5% cap and resident-only rule keep it clear of FIRB and landholder duty [I au/01 §2.6].
    - **Counts and concentration:** US §12(g) holder counts [C us/04 §2.3]; EU real-estate transfer tax (RETT) share-deal thresholds [I reg/eu §8]. RealT's FIRPTA gap on P2P trades is a warning [I us/04 §2.7].

**Honourable mention: metric inflation.** Headline figures diverge from verified numbers:
- RedSwan: $4–5B claimed vs ~$100M corroborated [C us/02].
- Redbelly: US$73.8bn claimed vs ~US$1bn [C au/02].
- Brickken: issuer-declared TTV vs on-chain TVL [C eu/04].

RWA.xyz shows only ~US$226M of *distributed* tokenised real estate worldwide [C inf/std §0.7]. Audited, on-chain-verifiable metrics are therefore a cheap way to stand out.

## 4. What this means for RealX

1. **Choose the golden record per jurisdiction before writing code:**
   - US: a transfer agent's register, synced to an ERC-3643 token.
   - EU: a national DLT register or a registrar.
   - UAE/KSA: the land registry, joined as a distributor.
   - Australia: the RE's unit register, with the token as a mirror until the constitution designates it.
   - Build one reconciliation service that serves all four [I syn; C reg/x A5 #25].
2. **Rent the regulated core; own the experience and the operations.** Every region has a "distributor on shared rails" route:
   - US: partner BD, TA and ATS [C reg/us §2].
   - EU: tied agent, or an ECSP licence [C reg/eu §2].
   - Dubai: VARA broker-dealer on the DLD/Ctrl Alt rails [C me/02 §12].
   - KSA: REGA sandbox operator [C me/04 §12].
   - Australia: an RE-for-hire, or a CAR of an AFSL holder [C reg/au §2].
3. **Make property operations a first-class product.** Build municipal compliance, title and deed reconciliation, reserves, PM SLAs, vacancy alerts, and a pre-agreed wind-down waterfall. RealT is the cautionary case [C us/04 §11 ⚑].
4. **Build one jurisdiction-parameterised rules engine**, not four compliance stacks [C reg/x design principle]. Enforce at transfer time on-chain (ERC-3643 modules) and off-chain (golden record).
5. **Start fiat-first, add stablecoins behind an allow-list.** Use EURC in the EU, USDC once GENIUS-permitted in the US, CBUAE-licensed AED tokens in the UAE, and AUDM/AUDD in Australia [C inf/ven §6].
6. **Stage liquidity:**
   - First, a guard-railed internal board: ±15% of independent NAV, holder caps, lock-ins, halts [C me/01; C au/01].
   - Then a licensed venue with T+0 atomic settlement.
   - Only later, borrowing against tokens, with independent oracles and circuit breakers.
7. **Differentiate on verifiability.** Offer "proof of property" (deed, title insurance, tax-paid status, compliance certificate) and audited per-property ledgers and reserves. RealT, Lofty and PRYPCO were all criticised on exactly this [C us/04 §12; C us/01 §12; C me/01 §10].

The full playbook, with a phased go-to-market, 15 v1 capabilities, the top 10 risks and open questions, is in `implications-for-realx.md`.

## 5. File map (`04-synthesis/`)

| File | Contents |
|---|---|
| `compliance-matrix.md` | 20 platforms across six compliance themes; compliance archetypes; minimum viable compliance stack per region |
| `user-types-and-actions-matrix.md` | Role taxonomy; platform × role presence; role × action matrix (on-chain vs off-chain) |
| `reference-architecture.md` | Layered reference architecture; one diagram per archetype; golden-record options; key-management patterns |
| `end-to-end-flows.md` | 10 canonical flows with sequence diagrams and archetype variants |
| `tech-stack-comparison.md` | Stack by layer × 20 platforms; observed patterns; recommended default stack per region |
| `regional-comparison.md` | US vs EU vs MENA vs AU, dimension by dimension |
| `implications-for-realx.md` | Builder playbook |
| `conflicts-and-gaps.md` | 50+ contradictions and their resolutions; load-bearing [I]/[S] claims; re-verification priorities |
