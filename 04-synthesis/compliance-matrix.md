# Compliance Matrix: 20 Platforms × Compliance Dimensions

**Date:** 23 Sep 2026 · Research, not legal advice.

**Evidence key.** Labels are carried over from the source file and never upgraded: [C] confirmed, [I] inferred, [S] speculative. The file code and section follow the label, e.g. `[C us/01 §2.3]`. Codes are defined in `00-executive-summary.md`. `[I syn]` = inference made in this synthesis. F1–F6 = synthesis re-check fetches (see `conflicts-and-gaps.md` §4).

The tables are split by theme to keep them readable. Platform order is the same in every table: US → EU → MENA → AU.

---

## (a) Legal wrapper and what the token represents

| Platform | Issuing vehicle | What the token / interest represents | Classification | Golden record (legal truth) |
|---|---|---|---|---|
| Lofty | One Wyoming DAO LLC per property [C us/01 §2.1] | LLC membership interest as an Algorand ASA. Voting weight follows ownership; 15% cap per investor [C us/01 §2.2] | Security under *Howey* [I reg/us §1]. Lofty says the platform is "designed to avoid selling securities" [C us/01 §2.3] | The ASA ledger acts as the member register; no transfer agent [I us/01 §2.4] |
| RedSwan | Sponsor SPVs and funds (LLC/LP); GCC fund relies on 3(c)(1)/(7) [C us/02 §2.1] | Issuer-sponsored tokenised equity or LP interest [C us/02 §2.2] | Security, issuer-sponsored in the SEC taxonomy [C us/02 §2.2; C reg/us §1] | The registrar/TA ledger, reconciled to the chain [C us/02 §2.2] |
| Arrived | Delaware series LLC; each series elects REIT status [C us/03 §2.1, §2.7] | Series membership interest in book-entry form; no token [C us/03 §2.2] | Security offered under Reg A+ Tier 2 [C us/03 §2.3] | Third-party TA book entry [C us/03 §2.2] |
| RealT | Delaware series LLC, later one Delaware corporation per property; 165 entities [C us/04 §2.1] | ERC-20 representing an LLC interest (early) or common stock (later) [C us/04 §2.2] | Security (Reg D / Reg S) [C us/04 §2.3] | Whitelisted ERC-20 plus RealT's records; no TA disclosed [C us/04 §2.3] |
| mogul | PropCo LLC per property, taxed as a partnership, run as an "investment club" [C us/05 §2.1] | ERC-1155-compatible NFT representing LLC units [C us/05 §2.2] | mogul says it is a club, not a security; the research flags *Howey* risk [I us/05 §2.3] | **The operating agreement designates Avalanche as the register** [C us/05 §2.2] |
| Reental | Spanish S.L. SPV funded by a participative loan (RDL 7/1996); US deals use an LLC [C eu/01 §2.1] | Spain: a credit claim with variable interest plus sale upside. US: LLC equity [C eu/01 §2.2] | Reental states the property tokens are MiFID financial instruments and RNT is a MiCA utility token [C eu/01 §2.2] | Off-chain loan contracts; Polygon token as the ownership record [I eu/01 §6] |
| Blocksquare | No SPV. The issuer adopts a corporate resolution (on IPFS); Luxembourg notarised framework since Feb 2025 [C eu/02 §2.1] | Royalty-type "economic rights" over revenues, including sale proceeds; 100,000 BSPT per property [C eu/02 §2.2] | Blocksquare claims a MiCA crypto-asset. The research views it as likely a transferable security [I eu/02 §2.3] | On-chain registry plus IPFS hash; legal anchor off-chain [C eu/02 §6] |
| Propvest | One GmbH SPV per property [C eu/03 §2.1] | Subordinated, profit-participating token-based bond [C eu/03 §2.2] | MiFID transferable security under a CSSF-approved prospectus (2019) [C eu/03 §2.3] | Custodian/depot records plus Ethereum. Conversion to eWpG crypto securities is **unverified** [eu/03 §2.2 gap] |
| Brickken | Issuer's choice [C eu/04 §2.1] | Issuer-defined; points to off-chain documentation [C eu/04 §2.2] | Issuer's tokens fall under MiFID; the BKN utility token under MiCA [I eu/04 §2.3] | Whatever the issuer designates; on-chain whitelist [C eu/04 §2.5] |
| Domoblock | None; Domoblock borrows directly [C eu/05 §2.1] | Participative loan to Domoblock. The fan token is an accounting unit with no rights [C eu/05 §2.2] | Built to sit outside MiFID, the Prospectus Regulation, ECSPR and MiCA [I eu/05 §2.3] | Off-chain ledger; Polygon mirror [I eu/05 §6] |
| PRYPCO Mint | No SPV; investors registered by name at DLD [C me/01 §2.1] | Fractional title-deed interest, plus an ARVA "management token" since Phase 2 [C me/01 §2.2] | ARVA under VARA, with DLD title [C reg/me §1] | **DLD registry plus certificate**; XRPL is synced to it [C me/01 §6] |
| Ctrl Alt | DLD rails: no SPV. Guernsey/Liechtenstein vehicles for other assets [C me/02 §2.1] | Ownership token plus ARVA management token (dual-token design) [C me/02 §2.2] | VARA Category 1 ARVA issuer [C me/02 §2.3] | DLD registry; Ctrl Alt's holder register maps omnibus sub-accounts [I me/02 §6] |
| Stake | DIFC SPV per property; DLD's limit of 4 owners per deed forces the SPV. KSA: CMA private funds [C me/03 §2.1] | SPV shares with share certificate; fund units [C me/03 §2.2] | DFSA crowdfunding; CMA fund units [C me/03 §2.3] | DIFC share register; DLD deed in the SPV's name [C me/03 §2.2] |
| Saudi RER + operators | Direct fractional deeds. An SPV variant is claimed only by a vendor page [C me/04 §2.1] | One deed token (1:1) plus fractional ownership tokens linked to RER [C me/04 §2.1] | REGA sandbox and official standard [C me/04; F2]. **CMA role unresolved** [me/04 §2.3] | **RER registry**: "absolute probative force"; settlement = registry update [C me/04 §2.2] |
| SmartCrowd | DIFC SPV per property (1m shares). Egypt: Civil Code co-ownership moving to FRA-licensed funds [C me/05 §2.1] | SPV shares; Egyptian co-ownership shares or fund units [C me/05 §2.2] | DFSA crowdfunding; FRA [C me/05 §2.3] | DIFC share register [C me/05 §2.2] |
| BrickX | Registered MIS; one unit trust per property. AET is custodian; BrickX Trusco holds title [C au/01 §2.1] | One unit = 1/10,000 of the beneficial interest [C au/01 §2.2] | MIS interest [C au/01 §2.3; C reg/au §1] | RE unit register, off-chain [C au/01 §2.2] |
| Redbelly | Issuer vehicles on the network [C au/02 §2.1] | Issuer-defined | Per issuer (INFO 225) [C au/02 §2.3] | Issuer register. Enforceability of on-chain records is an open question per the RBA [C au/02 §2.3] |
| Tokeniser + Liquidise | Issuer's own trust, SPV or company [C au/03 §2.1] | Units or shares as ERC-3643 tokens [C au/03 §2.2] | MIS interests or shares under the issuer's AFSL [C au/03 §2.3] | Trustee/RE register is legal; the token's legal status is not stated [I au/03 §2.2] |
| Canvas Connect | SPV investing in a real-estate credit fund [C au/04 §2.1] | SPV shares recorded in the Canvas registry (CDSR) [C au/04 §2.2] | Securities under Acacia pilot relief [C au/04 §2.3] | CDSR registry [C au/04 §2.2] |
| Assetora + Bricklet | One registered MIS with single-asset sub-funds; Perpetual is custodian [C au/05 §2.1] | Units in a sub-fund (ASF). Bricklet: co-owners on Torrens title [C au/05 §2.2] | MIS interest [C au/05 §2.3] | RE unit register. Bricklet: APCOR DLT register plus Torrens title [C au/05 §2.2] |

**Pattern.**
- **US:** equity in a per-asset entity.
- **EU:** debt-like claims (loans, bonds, royalties), because transferring Spanish S.L. or German GmbH shares needs a notary [I eu/sum §6.1].
- **MENA:** registry-anchored direct title, or DIFC SPV shares.
- **Australia:** MIS units.

## (b) Licence, exemption, regulator and licensed partners

| Platform | Own licence / exemption | Regulator(s) | Licensed partners (BD/ATS · TA/registrar · custodian · bank/escrow/PSP) |
|---|---|---|---|
| Lofty | **No filing for retail sales.** Not a BD or ATS. Separate Reg D syndicate funds [C us/01 §2.3] | CA DFPI (order) [C us/01 §2.3] | Stripe; USDC (Circle). No BD, TA or custodian [C us/01 §2.4] |
| RedSwan | Reg D 506(c) + Reg S. Affiliated FINRA BD (RedSwan Markets, May 2024). RIA claim conflicts with its own ToS [C us/02 §2.3] | SEC, FINRA | Unnamed registrar/TA; Coinbase custody; unnamed escrow [C us/02 §2.4] |
| Arrived | Reg A+ Tier 2 (1-A / 1-K / 1-SA); blue-sky notice filings [C us/03 §2.3] | SEC; state regulators | NCPS (BD); PPEX (ATS); NCIT, Stripe, Dwolla; Modern Treasury (funds); unnamed TA [C us/03 §2.4] |
| RealT | Reg D 506(c) (17 Form Ds in 2022) + Reg S (non-US only since 2022) [C us/04 §2.3] | SEC (filings); Michigan courts | Request Finance. No BD, TA or custodian [C us/04 §2.4] |
| mogul | No SEC filings; relies on an investment-club theory [C us/05 §2.3] | None | Fireblocks, Plaid, PayPal/Venmo. No BD or TA [C us/05 §2.4] |
| Reental | Prospectus-exempt STOs placed through an **unnamed** ESI; US Reg D/S. No own licence [C eu/01 §2.3] | CNMV (indirectly) | Unnamed ESI; Akka (distributor); self-custody [C eu/01 §2.4] |
| Blocksquare | No licence. EU Blockchain Sandbox 2025 dialogue, which is not an approval [C eu/02 §2.3] | None; each operator is responsible for itself | Alchemy (AA wallets); notaries (LU) [C eu/02 §2.4] |
| Propvest | CSSF-approved prospectus passported to BaFin. Exporo Invest: WpIG §15, ECSP and GewO §34c licences [C eu/03 §2.3] | BaFin/Bundesbank; CSSF | Baader Bank (depot); Tangany (crypto custody); Mangopay; EdW compensation scheme; Bundesbank ADR [C eu/03 §2.4] |
| Brickken | None (software provider). Issuers use exemptions and partner placement agents [C eu/04 §2.3] | None | Partner directory (law firms, placement agents, custodians); Sumsub [C eu/04 §2.4–2.5] |
| Domoblock | None. Self-declared outside CNMV/ESI/ECSP supervision [C eu/05 §2.3] | None | Mangopay (EMI); notaries [C eu/05 §2.4] |
| PRYPCO Mint | VARA Broker-Dealer VL/25/05/001 (24 May 2025); DLD REES sandbox [C me/01 §2.3] | VARA, DLD, CBUAE, DFF | Ctrl Alt (issuer, custody); Zand Bank (client money); DLD trust account; Paxos (gold) [C me/01 §2.4] |
| Ctrl Alt | VARA Broker-Dealer + **Category 1 VA Issuance** VL/25/05/002; FCA; CBI (MiFID); GFSC (transfer agency) [C me/02 §2.3] | VARA, FCA, CBI, GFSC | Ripple Custody; DLD [C me/02 §2.4] |
| Stake | DFSA F005879 crowdfunding (Islamic window); CMA FinTech permit. **VARA in-principle approval only; not on the register** [C me/03 §2.3] | DFSA, CMA, VARA | Emirates NBD (client money); Checkout.com; IDWise [C me/03 §2.4–2.5] |
| Saudi RER + operators | REGA PropTech sandbox: 6–24 months of testing, weekly reports [C me/04 §2.3] | REGA; CMA role unresolved | RER, SettleMint, Inspire, droppRWA; mada/SADAD [C me/04 §2.4] |
| SmartCrowd | DFSA crowdfunding (since 2018); VARA IPA (5 Feb 2026); four FRA licences in Egypt [C me/05 §2.3] | DFSA, VARA, FRA | Lean Payments; DLD trustee office [C me/05 §2.4] |
| BrickX | AFSL 494878 (as RE); registered MIS ARSN 613 156 433. **Market-licence basis for its order book not confirmed** [S au/01 §2.3] | ASIC, AUSTRAC | AET (custodian); Equifax; unnamed bank [C au/01 §2.4] |
| Redbelly | Network is unlicensed; issuers hold AFSLs; Acacia pilot relief [C au/02 §2.3] | ASIC/AUSTRAC via issuers; RBA (pilot) | Fireblocks; Macropod AUDM (AFSL 566313) [C au/02 §2.4] |
| Tokeniser + Liquidise | Tokeniser: no stated AFSL. Liquidise: AFSL 547681. Claims AMMs are an "exempt public protocol" [C au/03 §2.3; I au/03 §2.3] | ASIC | IQ-EQ (trustee/registry); Averer (identity) [C au/03 §2.4] |
| Canvas Connect | Acacia opt-in relief from ASIC and AUSTRAC; FinTech sandbox exemption; AFSL application in progress; AUSTRAC-registered VASP [C au/04 §2.3] | ASIC, AUSTRAC, RBA | Banking Circle (ESA access); Zodia, Fireblocks; NAB, Fiserv [C au/04 §2.4] |
| Assetora + Bricklet | AFSL 444365 (RE since Jun 2025); registered MIS ARSN 167 020 626. Marketplace basis undisclosed [S au/05 §2.3] | ASIC; ASX (for the listed parent) | Perpetual (custodian); bank (historically ANZ); AMP Bank (Bricklet mortgages) [C au/05 §2.4] |

## (c) Investor eligibility, geo-restrictions, limits, lock-ups and holder caps

| Platform | Eligible investors | Geo-restrictions | Investment limits | Lock-ups / holder caps |
|---|---|---|---|---|
| Lofty | Retail; no accreditation [C us/01 §2.6] | Non-US accepted except sanctioned countries; California restricted [C us/01 §2.6] | None disclosed | **15% cap per investor per property** [C us/01 §2.2]; no lock-up |
| RedSwan | US accredited (506(c) with third-party verification); non-US under Reg S; institutions [C us/02 §2.6] | Non-accredited US persons excluded | — | Rule 144 holding periods [C us/02 §2.3] |
| Arrived | US retail and accredited [C us/03 §2.3] | **US residents only**, in states where registered [C us/03 §2.6] | Reg A Tier 2 cap for non-accredited: 10% of income or net worth [C us/03 §2.3] | 6-month hold before secondary [C us/03 §9] |
| RealT | Non-US retail (Reg S) since 2022; legacy US accredited investors [C us/04 §2.6] | US geo-blocked [C us/04] | None | **2,000 holders per property** (§12(g) logic; dropped Nov 2024); per-property whitelist [C us/04 §2.3] |
| mogul | US retail; no accreditation [C us/05 §2.6] | US only per ToS; docs mention non-US ID (conflict) [C us/05 §2.6] | None | Transfers need consent from mogul and the LLC [C us/05 §5.4] |
| Reental | Adults and entities; 108 countries; US investors must be accredited [C eu/01 §2.6] | — | No ECSP-style test evidenced [I eu/01 §2.6] | P2P only between KYC'd users [C eu/01 §2.5] |
| Blocksquare | Set per operator or issuer (e.g., "US residents only") [C eu/02 §2.6] | Set by operator | — | Operator whitelist [C eu/02 §2.5] |
| Propvest | Retail, with full WpHG protections; appropriateness and suitability tests [C eu/03 §2.3; I eu/03 §2.3] | DE/AT apps; German-language onboarding [C eu/03 §2.6] | — | None |
| Brickken | Set per issuer [C eu/04 §2.6] | Set per issuer | Min/max ticket per offering [C eu/04 §4] | Whitelist per token [C eu/04 §2.5] |
| Domoblock | Spain/EU residents; non-EU with an NIE. Pre-sale tier for investors with ≥€50k invested or ≥5 projects [C eu/05 §2.6, §1] | — | DAI capped at €10k per transaction [C eu/05 §2.5] | Non-transferable except by loan assignment [C eu/05 §2.2] |
| PRYPCO Mint | UAE residents with Emirates ID, 18+ (help centre says 21+); natural persons. Gold open to international investors [C me/01 §2.6] | Non-residents excluded from real estate | **20% cap per property** [C me/01 §2.6; F4] | **3-month lock-in before listing** [C me/01 §9; F4] |
| Ctrl Alt | Professional investors and eligible counterparties directly; retail only through distributors [C me/02 §2.6] | — | — | Rules engine: lock-in, price band, holder cap [C me/02 §5.4] |
| Stake | 18+; non-residents allowed. Retail vs Professional. KSA funds split 50/50 qualified/retail [C me/03 §2.6] | Excluded-nationality list [C me/03 §2.5] | **DFSA retail cap: AED 183,500 per property and AED 367,000 per year**; Professionals up to 33% of a property [C me/03 §2.6] | 1-year lock-in before Exit Windows [C me/03 §9] |
| Saudi RER + operators | Saudi citizens and residents; non-Saudis phased in by zone [C me/04 §2.6] | Holy-cities rules; **geo-fencing enforced at every transfer** [C me/04 §2.5] | Per-operator caps [I] | Eligibility checked at transfer time [C me/04 §5.4] |
| SmartCrowd | Global retail (130+ countries) and professionals [C me/05 §2.6; I me/05 §2.6] | — | DFSA caps apply [I me/05 §2.6] | STF only for properties held more than 1 year [C me/05 §9] |
| BrickX | AU residents (including citizens abroad), 18+; individuals, SMSFs, entities [C au/01 §2.6] | **Foreign investors excluded** | — | **5% cap (500 Bricks) per member per trust** [C au/01 §2.6] |
| Redbelly | Set per issuer through credentials [C au/02 §2.6] | — | — | Credential-gated [C au/02 §2.2] |
| Tokeniser + Liquidise | Wholesale/sophisticated by default; retail only on the Enterprise tier [C au/03 §2.6] | — | — | ERC-3643 rules [C au/03 §2.5] |
| Canvas Connect | Wholesale only (s761G) [C au/04 §2.3] | — | — | Allowlist [C au/04 §2.5] |
| Assetora + Bricklet | AU residents (individuals, SMSFs, trusts, companies) [C au/05 §2.6] | — | — | Marketplace can be suspended; exit through term votes [C au/05 §5.5] |

## (d) KYC / AML / sanctions / travel rule and vendors

| Platform | KYC flow & vendor | Sanctions / AML controls | Travel rule | Wallet permissioning |
|---|---|---|---|---|
| Lofty | ID + selfie, approved in minutes; vendor **undisclosed** [C us/01 §2.5] | Sanctioned-jurisdiction representations [C us/01 §2.5] | None disclosed [C us/01 §2.5] | Likely ASA freeze/clawback [I us/01 §7]. **USDC liquidity staking needs no KYC** [C us/01 §2.5] |
| RedSwan | KYC, AML and OFAC checks plus third-party accreditation; vendors unnamed [C us/02 §2.5] | OFAC screening; BD AML programme [C us/02 §2.5] | — | Likely HTS KYC key and freeze key [I us/02 §2.5] |
| Arrived | Unnamed vendor, likely via NCPS/NCIT [I us/03 §2.5]; Plaid [C us/03 §2.5] | Through the BD | Not applicable (fiat) | Not applicable |
| RealT | KYC plus a **per-property wallet whitelist** (≤28 tokens/day, 1–2 days) [C us/04 §2.5] | Not documented | Not documented | Whitelist enforced in the token [C us/04 §7] |
| mogul | SSN-based; unnamed vendor [C us/05 §2.5] | BSA clause in the ToS; no OFAC disclosure [C us/05 §2.5] | — | Custodial; off-chain transfer consent [C us/05 §7] |
| Reental | Mandatory KYC; vendor undisclosed. About 30% of sign-ups complete KYC [C eu/01 §2.5] | Can re-verify at any time [C eu/01 §2.5] | Not a CASP; no travel-rule data exchange [I eu/01 §2.5] | P2P allow-list. Enforcement at token level is unclear [I eu/01 §7] |
| Blocksquare | Operator runs KYC (ID, selfie, liveness), ~24h [C eu/02 §2.5] | Operators must screen UN, EU, UK and OFAC lists [C eu/02 §2.5] | Not addressed [eu/02 §2.5 gap] | On-chain Users whitelist (`addWalletList`) [C eu/02 §2.5] |
| Propvest | Unnamed identity service [C eu/03 §2.5]; GwG-obliged entity [I eu/03 §2.5] | AML as a licensed institution [I eu/03 §2.5] | Minimal, because transfers stay inside the custodian [I eu/03 §2.5] | Custodial (Tangany) [C eu/03 §7] |
| Brickken | **Sumsub**; six-state KYC machine; KYB [C eu/04 §2.5] | Through Sumsub | None; neither Brickken nor issuers are CASPs [I eu/04 §2.5] | Per-token whitelist. `needKyc` can be set false [C eu/04 §2.5] |
| Domoblock | DNI + selfie, KYB, suitability test. Vendor unknown; Mangopay also runs KYC [C eu/05 §2.5; I eu/05 §2.5] | €10k cap per DAI transaction [C eu/05 §2.5] | Not a CASP [I eu/05 §2.5] | Not applicable (fan token) |
| PRYPCO Mint | Emirates ID, address and source of funds; UAE PASS login since Aug 2026; vendor undisclosed [C me/01 §2.5] | VARA AML obligations [I me/01 §2.5] | Minimal, because tokens stay in omnibus custody [I me/01 §2.5] | Omnibus custody plus off-chain rules engine [C me/01 §7] |
| Ctrl Alt | Delegated to distributors [C me/02 §2.5] | Issuer-level AML [I me/02 §2.5] | — | ARVA layer plus rules engine [C me/02 §5.4] |
| Stake | **IDWise** passport + selfie, proof of address; KYC fee charged to the investor [C me/03 §2.5] | Excluded nationalities [C me/03 §2.5] | Not applicable | Not applicable |
| Saudi RER + operators | **Nafath + Absher + Yakeen**; W3C verifiable credential [C me/04 §2.5] | Eligibility and geo-fence checks at every transfer [C me/04 §2.5] | — | ERC-3643 compliance modules [C me/04 §7] |
| SmartCrowd | 1–2 business days; vendor undisclosed [C me/05 §2.5] | AML cost covered in the entry fee [C me/05 §2.5] | Not applicable | Not applicable |
| BrickX | **Equifax ID Matrix** (no credit check); FATCA/CRS self-certification [C au/01 §2.5] | Can refuse "Proscribed" persons; files suspicious matter reports [C au/01 §2.5] | Not applicable | Not applicable |
| Redbelly | Reusable verifiable credentials and ZK proofs; vendor unnamed [C au/02 §2.5] | Claims AML/CTF support [C au/02 §2.2] | Claimed [C au/02 §2.2] | Credential-gated contracts [C au/02 §2.2] |
| Tokeniser + Liquidise | Averer credentials and ZK proofs; underlying vendor undisclosed [C au/03 §2.5] | — | — | ERC-3643 plus credentials [C au/03 §2.5] |
| Canvas Connect | Unnamed vendors for identity, sanctions, chain analytics, travel rule and transaction monitoring [C au/04 §2.5] | Yes [C au/04 §2.5] | Yes (vendor category listed) [C au/04 §2.5] | Allowlist and roles [C au/04 §7] |
| Assetora + Bricklet | Vendor undisclosed [C au/05 §2.5] | Reporting entity [I au/05 §2.3] | Not applicable | Not applicable |

**Vendor disclosure is poor.** Only three of 20 platforms name a KYC vendor: Brickken (Sumsub), Stake (IDWise) and BrickX (Equifax). Two use government identity: PRYPCO (UAE PASS) and the Saudi operators (Nafath). The rest are undisclosed [per-file §2.5; C us/sum §9; C me/sum §6].

## (e) Tax reporting and withholding

| Platform | Investor tax form / regime | Withholding | Design note |
|---|---|---|---|
| Lofty | 1099-DIV + 1099-B [C us/01 §2.7] | Non-US investors: 30%, or 15% with a treaty [C us/01 §2.7] | C-corp election inferred, which avoids K-1s [I us/01 §2.7] |
| RedSwan | K-1 [I us/02 §2.7] | §1446 and FIRPTA [I us/02 §2.7] | Blockers for non-US investors [S us/02 §2.7] |
| Arrived | 1099-DIV per REIT series [I us/03 §2.7] | Not applicable (US only) | **REIT election per series** [C us/03 §2.7] |
| RealT | Annual "Rent Distribution Report" splitting dividend from return of capital [C us/04 §2.7] | FDAP withholding on dividends [I us/04 §2.7] | **FIRPTA on P2P sales not addressed** [I us/04 §2.7] |
| mogul | K-1 [C us/05 §2.7] | Not applicable | Partnership; depreciation passes through [C us/05 §2.7] |
| Reental | Automated 19% withholding for Spanish residents [C eu/01 §2.7] | 19% | USDT payouts create a second taxable event [I eu/01 §2.7] |
| Blocksquare | Not addressed; stablecoin paid gross [C eu/02 §2.7] | None | DAC8 applies if operators are CASPs [I eu/02 §2.7] |
| Propvest | The depot bank handles withholding and tax certificates [I eu/03 §2.7] | Via Baader [I eu/03 §2.7] | CRS via the depot bank [I eu/03 §2.7] |
| Brickken | Not provided; the issuer handles tax [I eu/04 §2.7] | Issuer | — |
| Domoblock | 19% withholding; income taxed as capital income [C eu/05 §2.7] | 19% | Referral bonuses taxed as general income [C eu/05 §2.7] |
| PRYPCO Mint | Investor responsible; no withholding [C me/01 §2.7] | None | Markets a 2% DLD fee against the 4% standard [C me/01 §1] |
| Ctrl Alt | Not disclosed [S me/02 §2.7] | — | — |
| Stake | Help articles cover UAE tax, KSA withholding tax and Zakat [C me/03 §2.7] | KSA withholding likely [I me/03 §2.7] | — |
| Saudi RER + operators | 5% RETT on transfers; non-Saudi surcharge up to 5% [C me/04 §1, §2.7] | — | Zakat [I me/04 §2.7] |
| SmartCrowd | Not disclosed [C me/05 §2.7] | — | 4% DLD fee passed through [C me/05 §8] |
| BrickX | Annual tax statement; TFN; FATCA/CRS [C au/01 §2.7] | TFN withholding [I au/01 §2.7] | **5% cap and resident-only rule avoid landholder duty and FIRB** [I au/01 §2.6]; AMIT election unconfirmed [S au/01 §2.7] |
| Redbelly | Per issuer [I au/02 §2.7] | — | — |
| Tokeniser + Liquidise | Distributions supported; AMIT not mentioned [C au/03 §2.7; I au/03 §2.7] | — | — |
| Canvas Connect | Not disclosed [I au/04 §2.7] | — | — |
| Assetora + Bricklet | "No stamp duty on unit transfers between members" [C au/05 §2.7] | TFN [I au/05 §2.7] | AMIT unknown [S au/05 §2.7] |

Regulatory context for tax engines:
- **US:** 1099-DA for tokenised securities unless traded on a limited-access regulated network (then 1099-B); cost-basis reporting from 2026 [C reg/us §8].
- **EU:** DAC8 applies from 1 Jan 2026, with first reports due 30 Sep 2027. Tokenised securities generally report under CRS [C reg/eu §8].
- **UAE:** CARF consultation (Oct 2025) [C reg/me §8].
- **Australia:** AMIT/AMMA statements; MIT withholding at 15% or 30% [I reg/au §8].

## (f) Enforcement, litigation and incidents

| Platform | Event(s) | Date | Lesson for RealX |
|---|---|---|---|
| Lofty | CA DFPI enforcement action (reported as a consent order restricting California residents; PDF blocked) [C us/01 §2.3]. St. Louis squatters and board-up notices [C us/01 §10]. MyAlgo wallet exploit [C us/01 §10] | 2022; 2025; 2023 | An undisclosed exemption invites state action. Absentee DAO ownership invites municipal hostility |
| RedSwan | No enforcement found (BrokerCheck not checked). Inconsistent scale claims; "24/7 liquidity" marketing contradicts its own disclaimer [C us/02 §10] | 2025–26 | Marketing controls matter even for licensed firms |
| Arrived | WA DFI consent order: $40k for notice-filing failures [C us/03 §2.3]. BBB liquidity complaints [C us/03 §10] | May 2025 | A blue-sky filing engine is mandatory |
| RealT | **City of Detroit suit** over 408 properties; TRO sending rent to escrow; payouts halted by end-2025 and formally suspended Feb 2026; court fiduciary Apr 2026; **voluntary liquidation 2 Jul 2026**; French class action and a Paris criminal complaint; allegations some properties were "never owned" [C us/04 §2.8; F6] | Jul 2025 – Jul 2026 | Property operations, municipal compliance and title verification are existential. Token holders rank behind the city, taxes and the fiduciary [C us/04 §2.8] |
| mogul | No enforcement found. Investment-club theory and broker-compensation risk from club rewards; Trustpilot 2.2 [I us/05 §10; C us/05 §10] | 2026 | Community rewards tied to volume can look like broker compensation |
| Propvest | KapMUG collective steps; critical source says all 38 projects missed targets; no BaFin action [C eu/03 §2.8] | Jun 2025 | A licence does not protect against valuation and performance failure |
| Domoblock | No action. Delivery delays; a third-party article falsely claimed CNMV authorisation [C eu/05 §10] | 2026 | Unregulated competitors create category reputation risk |
| PRYPCO Mint | No action. Trustpilot complaints: a refund outstanding from Aug 2025 to Jul 2026; AI-only support [C me/01 §2.8] | 2025–26 | Service-level agreements for complaints are regulatory as well as reputational |
| Ctrl Alt | No action. Manual ownership-change steps at launch [C me/02 §10] | 2025 | Automate registry sync |
| BrickX | Mosman wind-up vote failed (39.77% for) [C au/01 §2.8] | Jun 2025 | Exit votes can strand holders; design liquidity backstops |
| Assetora | Trustee withdrew the PDS and suspended the marketplace (NDIS vacancies); reopened after about 13 months; parent suspended on ASX for unpaid fees; recapitalised [C au/05 §2.8] | Jul 2024 – Aug 2025 | Market-freeze powers, vacancy buffers and operator solvency |
| Others (Reental, Blocksquare, Brickken, Stake, Saudi RER, SmartCrowd, Redbelly, Tokeniser, Canvas) | No enforcement found. Main risks are classification theses and metric claims [C/I per file §10] | — | — |
| *Context (not in top 20)* | Texas SSB emergency order against E-Estate Group for "tokenized real estate" claims such as "backed by real estate" and "capital protected" [C reg/us §11] | 1 Jun 2026 | Needs a marketing-claims library and approvals |

---

## Compliance archetypes (regulatory postures)

| Posture | Members | How it works | Observed outcome |
|---|---|---|---|
| **CP1 Fully licensed or prospectus-based** | Propvest, Arrived, PRYPCO Mint, Ctrl Alt, BrickX, Assetora, Stake, SmartCrowd | Own licence (or licensed partners) for every regulated function; prospectus/Reg A/PDS or a crowdfunding regime; client money at a bank, EMI or trust account [C eu/03 §2.4; C us/03 §2.4; C me/01 §2.4; C au/01 §2.3] | Scales with banks and institutions (Emirates NBD invested in Stake [C me/03 §2.4]). Slow and costly. Still exposed to performance risk (Propvest) and freezes (Assetora) |
| **CP2 Private placement through an exemption and intermediary** | RedSwan, Reental, RealT, Tokeniser (issuer's AFSL) | Reg D/Reg S; prospectus-exempt offers placed by an ESI; wholesale-only [C us/02 §2.3; C eu/01 §2.3; C us/04 §2.3; C au/03 §2.3] | Fast to launch. Retail reach depends on the exemption. RealT shows that Reg S geo-blocking does not shield a platform from US courts [C us/04 §12] |
| **CP3 Sovereign sandbox or regulator pilot** | Saudi RER operators, Canvas (Acacia), PRYPCO/Ctrl Alt (REES) | Admission to a regulator programme with reporting duties (weekly REGA reports [C me/04 §2.3]; time-bound Acacia relief [C au/04 §2.3]) | Strong legitimacy and registry linkage. Rules can change; exit from the programme is uncertain |
| **CP4 Software-only or "not a security" thesis** | Blocksquare, Brickken, Redbelly, Tokeniser's AMM theory | The vendor disclaims regulated activity; operators and issuers carry compliance; untested theses (MiCA-not-MiFID; "exempt public protocol") [I eu/02 §2.3; I au/03 §2.3] | Scales supply cheaply. Legal risk is pushed to operators; the "not a security" and exempt-protocol theses are untested (MiCA-not-MiFID: [I eu/02 §2.3]; AMMs under s767A: [I au/03 §2.3]) |
| **CP5 Outside the perimeter or undisclosed** | Lofty, mogul, Domoblock | No filing (Lofty), investment club (mogul), non-transferable fan tokens over an operator loan (Domoblock) [C us/01 §2.3; C us/05 §2.3; C eu/05 §2.3] | Cheapest UX; state enforcement risk (Lofty); no investor protection; cannot passport or partner with banks [I syn] |

**Recommendation for RealX [I syn]:** CP1 in each launch jurisdiction, reached through CP2- and CP3-style partnerships (rent-a-licence) while own licences are pending. Avoid CP4 and CP5 theses entirely.

---

## Minimum viable compliance stack by region (new entrant)

The goal is to launch legally with the smallest scope. A rent-a-stack partner is used wherever one exists. Labels are carried from the regulatory files.

### United States

| Component | Minimum viable choice | Evidence |
|---|---|---|
| Wrapper | Per-property series LLC or LLC. Elect REIT or C-corp status so investors get a 1099, not a K-1 | [C us/03 §2.7; I us/01 §2.7] |
| Offering | **Reg D 506(c) + Reg S** first; add **Reg A+ Tier 2** ($75M/12 months) or Reg CF ($5M) for retail | [C reg/us §2] |
| Intermediaries | Partner FINRA BD; **SEC-registered transfer agent as golden record**; qualified bank escrow | [I reg/us §2; C us/02 §2.4] |
| Investor verification | CIP/KYC; OFAC screening of persons and wallets; accreditation via third-party letter or the minimum-investment path ($200k natural person / $1M entity, per the 12 Mar 2025 no-action letter) | [C reg/us §3] |
| Token controls | Whitelist, Reg S and Rule 144 lock-ups, forced transfer and re-issue by the TA; holder-count monitor per SPV for §12(g) (2,000 / 500 non-accredited) | [C reg/us §5; I reg/us §1] |
| Funds flow | BD, bank escrow or a Modern Treasury-type holder, to stay out of MSB status; stablecoin allow-list once the GENIUS Act takes effect (~Jan 2027) | [C us/03 §2.4; C reg/us §4] |
| Secondary | Partner ATS after holding periods. No platform-run matching without a BD/ATS | [I reg/us §6] |
| Tax | W-9/W-8; 1099-DIV and 1099-DA/B; FIRPTA and §1446(f) withholding engine for foreign holders | [C reg/us §8; I us/04 §2.7] |
| Filings & marketing | Form D within 15 days; blue-sky notices (WA DFI lesson); claims library (Texas SSB lesson) | [C reg/us §7; C us/03 §2.3] |
| Property operations | Municipal compliance and title tracker (RealT lesson) | [C reg/us §9] |

### European Union

| Component | Minimum viable choice | Evidence |
|---|---|---|
| Wrapper | SPV issuing notes, participative loans or bonds (debt avoids notarised share transfers); a national DLT register where available (Spain Ley 6/2023; Germany eWpG) | [I eu/sum §6.1; C reg/eu §2] |
| Pathway | **Tied agent of a MiFID firm** (weeks to months) → **ECSP licence** (€5M per project owner per 12 months; KIIS; knowledge test; 4-day reflection; passport). Prospectus above €12M (a Member State may set €5M) | [C reg/eu §2–3] |
| Custody & payments | Qualified custodian (ECSPR art. 10); PSD2/EMD partner such as Mangopay or Lemonway | [C reg/eu §5; C eu/sum §6.5] |
| AML | AMLD now, AMLR from 10 Jul 2027; TFR data on stablecoin/EMT legs; ownership proof for self-hosted wallets above €1,000 | [C reg/eu §4] |
| Token | ERC-3643, with whitelist enforced at transfer (not just in the UI) | [C inf/std §0; I eu/01 §12] |
| Secondary | Bulletin board only, unless a MiFID MTF or DLT Pilot venue is used | [C reg/eu §6] |
| Stablecoin | MiCA-authorised EMTs only (EURC/EURCV); avoid USDT | [C inf/ven §6; I eu/01 §2.8] |
| Tax | Withholding engine (e.g., ES 19%); CRS/DAC8 self-certification; RETT concentration monitor | [C eu/01 §2.7; I reg/eu §8] |

### UAE (Dubai onshore)

| Component | Minimum viable choice | Evidence |
|---|---|---|
| Licence | **VARA broker-dealer as a DLD-approved distributor** on the Ctrl Alt/DLD rails. Alternative: own Category 1 issuance + BD | [C reg/me §2; C me/02 §12] |
| Investor classification | Retail / Qualified (AED 3.5M net assets or AED 700k income) / Institutional; suitability test; ≥1-week cooling-off after a failed test; 8-year records | [C reg/me §3] |
| KYC/AML | Emirates ID and UAE PASS; source of funds; EOCN/UN targeted-sanctions screening; goAML; FDL 10/2025 gap assessment | [C reg/me §4; C me/01 §2.5] |
| Travel rule | Beneficiary verification from AED 3,500 per day (aggregated); EDD for unhosted wallets | [C reg/me §4] |
| Client money & custody | Segregated UAE bank account; licensed custodian (MPC omnibus) | [C me/01 §2.4] |
| Guardrails | Holder cap (20%), price band (±15% of DLD valuation), lock-in (3 months), 24-hour cooling-off | [C me/01 §2.8, §9] |
| Payments | AED fiat. Stablecoins only if CBUAE-licensed AED payment tokens (CBUAE PTSR non-objection registration) | [C reg/me §4; I reg/me §4] |
| Marketing | VARA Market Conduct Rulebook; Trakheesi permit per listing | [C reg/me §7; I reg/me §7] |

### Saudi Arabia

| Component | Minimum viable choice | Evidence |
|---|---|---|
| Admission | REGA PropTech sandbox cohort (6–24 months of testing, weekly reports) | [C me/04 §2.3] |
| Identity | Nafath, Absher and Yakeen | [C me/04 §2.5] |
| Settlement | Dedicated acquisition (escrow) account; DvP; registry update through RER Aamal | [C me/04 §2.1, §7] |
| Eligibility | Non-Saudi zones; holy-cities limits (Saudi companies and Muslim individuals only); checked at every transfer | [C reg/me §9; C me/04 §2.5] |
| Parallel yield product | CMA FinTech fund-distribution permit (Stake model) | [C me/03 §2.3; C reg/me §2] |
| Tax / Shariah | 5% RETT; Zakat; Shariah review | [C me/04 §2.7; C me/04 §2.4] |

### Australia

| Component | Minimum viable choice | Evidence |
|---|---|---|
| Wrapper | **Umbrella registered MIS with a sub-fund per asset** (Assetora pattern) or a trust per property (BrickX); a wholesale unregistered MIS to start | [C au/05 §12; C au/01 §12; C reg/au §2] |
| Licence | RE-for-hire, or a corporate authorised representative (CAR) of an AFSL holder, then own RE AFSL | [C reg/au §2] |
| Disclosure & DDO | PDS plus a target market determination (TMD) per asset; significant-dealing reports within 10 business days | [I reg/au §3] |
| Custody | Custodian holding Torrens title (RG 133); NTA requirements | [C reg/au §5] |
| AML | AUSTRAC programme; tranche 2 (real-estate professionals) from 1 Jul 2026 | [C reg/au §4] |
| Eligibility & thresholds | Residency and FIRB flags (established-dwelling ban to 30 Jun 2029); holder caps below landholder-duty thresholds | [C reg/au §9; I reg/au §8] |
| Secondary | Bulletin board or ASIC relief. No AMM without a market licence or exemption | [I reg/au §6; I au/03 §2.3] |
| Token regime | **DAF Act 2026:** commences 8 Apr 2027 with a 6-month AFSL transition (TCP/DAP) and a low-value exemption ($10m per 12 months; $5k per client) | [C F1/F5] |
| Stablecoin | AUDM/AUDD, relying on ASIC Instrument 2025/867 relief for distributors | [C reg/au §5] |
| Tax | AMIT/AMMA; MIT withholding; TFN | [I reg/au §8] |
