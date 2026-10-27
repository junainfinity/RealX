# Vendor Landscape — Tokenised Real-Estate Infrastructure (Sep 2026)

**Purpose:** Categorised map of third-party vendors a fractional real-estate tokenisation platform (RealX) plugs into, with what each does, public cost model where found, 2025–26 corporate status, and known use by real-estate tokenisation platforms. Each vendor category also names the **admin-panel surface it creates** (settings page, queue, reconciliation view).
**Date:** 23 Sep 2026. **Labels:** [C#n] confirmed (Sources at end), [I] inferred / from prior knowledge not re-verified this session, [S] speculative.
**Method caveat:** the session-wide WebSearch quota was exhausted mid-task, so vendor currency was verified by direct fetches of vendor homepages/pricing pages where possible; unverified rows are explicitly [I]. Platform–vendor links should be cross-checked against `02-platforms/` profiles.

---

## 0. 2025–26 corporate changes that matter (verified where marked)

| Change | Evidence |
|---|---|
| **Onfido → Entrust Identity Verification** (onfido.com redirects to entrust.com) | [C#2] |
| **Parallel Markets → "iCapital Identity Solutions"** (site "will soon be retired"; iCapital Investor Passport) | [C#3] |
| **Verify Investor majority-owned by tZERO** (NYSE/ICE a significant investor in tZERO) | [C#4] |
| **Dynamic acquired by Fireblocks (Oct 2025)** | [C#6] |
| **Web3Auth acquired by Consensys → "MetaMask Embedded Wallets"** | [C#7] |
| **Privy acquired by Stripe (reported Jun 2025)** — homepage fetched but does not state ownership | [I] (offering confirmed [C#5]) |
| **INX is a subsidiary of OpenDeal (Republic)**; INX Securities = BD + ATS | [C#10] |
| **Securitize listed on NYSE (SECZ) on 2 Jul 2026** via Cantor Equity Partners II SPAC | [C#16] |
| **Circle IPO NYSE: CRCL (Jun 2025)**; EURC under MiCA (French EMI); **final OCC national-trust-bank approval Jul 2026**; GENIUS Act context (Senate passage Jun 2025) | [C#15] |
| **BitGo IPO NYSE: BTGO (Jan 2026)**; OCC federal charter approval **Dec 2025**; HQ moved to Sioux Falls, SD | [C#14] |
| **Tokeny is part of Apex Group** | [C#30] |
| **Kore (kore.inc) = rebranded KoreConX** | [C#18] |
| **Ctrl Alt VARA licence (Broker-Dealer + Issuer Services) 24 May 2025** | [C#20] |
| **21X** EU DLT-TSS licence (Dec 2024), live for trading Aug 2025 | [C#12] |
| **AE Coin** (AED Stablecoin LLC) — CBUAE-licensed AED payment token | [C#13] |
| **EIP-7702 live (Pectra, 7 May 2025)** — smart-EOA gas sponsorship/batching | [C#28] |

---

## 1. Identity: KYC / KYB / AML screening

| Vendor | What it does | Cost model (public) | RE-tokenisation usage | Admin-panel surface |
|---|---|---|---|---|
| **Sumsub** | IDV (doc + liveness), reusable KYC, AML screening & monitoring, KYB, **Travel Rule**, transaction monitoring | **Basic $1.35/verification ($149 min/mo); Compliance $1.85 ($299 min/mo); Custom**; charged for successful verifications only; 14-day trial w/ 50 checks [C#1] | Widely used by EU/MENA crypto & RWA platforms [I] | KYC queue w/ provider decision + manual override; AML hit review; Travel-Rule messages |
| **Entrust IDV (ex-Onfido)** | Document + biometric verification, **Workflow Studio** (no-code journeys), data verification, deepfake/synthetic-ID fraud, AML, KYC | Quote-based [C#2] | [I] | Provider config; workflow version shown on each case |
| **Persona** | Configurable KYC/KYB, "inquiries", case management, graph fraud detection | Tiered; startup plan [I] | US fintech/RE platforms [I] | Case-management deep links |
| **Jumio** | Enterprise IDV + AML (KYX platform) | Enterprise quote [I] | [I] | — |
| **Veriff** | IDV + AML + proof of address; strong EU coverage | Self-serve per-verification plans [I] | EU fractional RE [I] | — |
| **Shufti Pro** | KYC/KYB/AML, strong MENA/APAC doc coverage | Per-verification [I] | MENA [I] | — |
| **Alloy** | Identity & fraud **decisioning orchestration** (routes to multiple data vendors), credit/fraud policies | SaaS + per-evaluation [I] | US [I] | Policy/rules engine for onboarding decisions |
| **UAE PASS** | UAE national digital identity (SSO + e-signature) | Government [I] | Dubai tokenised-title flows (DLD/PRYPCO) likely use it [I — confirm in MENA profiles] | "Login with UAE PASS" + verified Emirates ID attributes |
| **DigiShares built-in KYC** | Bundled KYC/AML | **€2/verification**, 50 incl. in Launch tier [C#21] | DigiShares tenants | — |

## 2. Accredited / qualified investor verification (US-centric; also EU "professional" & AU "wholesale")

| Vendor | What it does | Cost / status | RE usage |
|---|---|---|---|
| **Parallel Markets → iCapital Identity Solutions** | 506(c) accreditation, KYC/KYB, reusable **Investor Passport**, perpetual AML monitoring | Status: rebranded, site retiring [C#3] | **Fundrise** listed as customer [C#3] |
| **Verify Investor** | Accredited / QP verification, AML/KYC, **On-ChainPass (soulbound token)** | Majority-owned by tZERO; pricing not shown [C#4] | tZERO ecosystem [I] |
| **InvestReady** | Accreditation verification (US) | Per-verification [I] | [I] |
| Admin surface | Accreditation queue: status, expiry date (US letters typically 90 days) [I], re-verification reminders; on-chain claim/credential issuance (ERC-3643 claim, XRPL credential, SBT) [I] | | |

## 3. Wallet screening / KYT & Travel Rule

| Vendor | What it does | Cost | Admin surface |
|---|---|---|---|
| **Chainalysis** (KYT, Reactor) | Address/tx risk scoring, sanctions, alerts; has published ERC-3643 explainers | Enterprise [I] | Wallet-risk badge on investor profile; alert queue; block/freeze decisions |
| **Elliptic** | Wallet & tx screening, ecosystem monitoring | Enterprise [I] | same |
| **TRM Labs** | Wallet screening, forensics, sanctions | Enterprise [I] | same |
| **Fireblocks policy engine** | "Automated compliance screening directly into transaction workflows" to flag/restrict high-risk addresses | Bundled [C#8] | Policy rule hits in approval queue |
| **Notabene** | Travel Rule (TRUST/TRP/IVMS101), counterparty VASP discovery, self-hosted-wallet proofs | SaaS [I] | Travel-Rule inbox (incoming/outgoing), wallet-ownership proofs |
| **Sumsub Travel Rule** | Bundled in Basic/Compliance plans | Included [C#1] | — |
| **21 Analytics** | Privacy-preserving Travel Rule (Swiss) | SaaS/on-prem [I] | — |

## 4. Custody, wallets & account abstraction

| Vendor | Type | Status / cost | RE-tokenisation usage | Admin surface |
|---|---|---|---|---|
| **Fireblocks** | MPC custody, **Tokenization Engine** (console deploy/mint/burn; 35+ chains), **Governance & Policy Engine** (amount/destination/asset/time rules, quorum approvals, admin quorum, pending-approval dashboard) | Enterprise SaaS [C#8][C#9]; acquired Dynamic Oct 2025 [C#6] | Tokeny issuers (Fireblocks × Tokeny) [C#36], Hedera ATS integration [C#27] | **Maker-checker for all on-chain admin tx**; treasury vault view |
| **BitGo** | Qualified custody, multisig/MPC wallets, stablecoin custody | **NYSE: BTGO (Jan 2026)**; OCC charter (Dec 2025); >$100B custodied [C#14] | [I] | — |
| **Anchorage Digital** | OCC-chartered crypto bank custody, staking, settlement | Enterprise [I] | Institutional RWA [I] | — |
| **Copper** | Institutional custody + ClearLoop off-exchange settlement | Enterprise [I] | — | — |
| **Dfns** | Wallet-as-a-service (MPC), policy engine; CIP-56 (Canton) support | SaaS [I] | Hedera ATS custody option [C#27] | — |
| **Taurus-PROTECT** | Bank-grade custody pre-integrated with Taurus-CAPITAL | Enterprise [C#31] | — | Function-level governance |
| **Privy** | Embedded wallets, agent/treasury wallets, "programmable rules, approvals and transfer controls", on/off-ramps, card programs | Contact sales [C#5]; Stripe ownership [I] | Consumer-grade onboarding for fractional RE apps [I] | User-wallet lookup, export/recovery support |
| **Dynamic (Fireblocks)** | Wallet auth, embedded MPC wallets, 800+ external wallets, server wallets, policy controls | **Free to 1,000 MAU**, then sales [C#6] | [I] | — |
| **MetaMask Embedded Wallets (ex-Web3Auth)** | Social-login MPC wallets, AA smart accounts, on-ramp aggregator | **Free for first 1,000 MAW** [C#7] | [I] | — |
| **Magic** | Email/social-login embedded wallets | Per-MAW [I] | Used by consumer RWA apps [I] | — |
| **Turnkey** | Secure-enclave key infra, policy engine, embedded/server wallets | Usage-based [I] | [I] | — |
| **Coinbase Smart Wallet / Base Account** | Passkey ERC-4337 smart wallet on Base; paymaster sponsorship | Free for users; paymaster credits [I] | Base-native RE apps [I] | — |
| **ERC-4337 + EIP-7702 paymasters** (Pimlico, Alchemy Account Kit, Biconomy, etc.) | Gas sponsorship, batched tx, session keys | Per-UserOp / markup [I]; EIP-7702 live May 2025 [C#28] | Gasless investor UX [I] | **Gas-sponsorship policy settings** (per-user caps, allowed contracts), paymaster balance alerts |

## 5. Fiat rails, on/off-ramps & payments

| Vendor | What it does | Cost model | Region | Admin surface |
|---|---|---|---|---|
| **Stripe** | Cards, ACH, SEPA, Connect payouts; stablecoin accounts via Bridge (acq. 2025) [I] | ~2.9% + 30¢ US cards; ACH 0.8% capped $5 [I] | Global | Payment reconciliation; refunds; payouts |
| **Plaid** | Bank-account linking, balance/identity/auth, Transfer | Per-connection/API [I] | US/EU | Bank-verification status on investor |
| **Dwolla** | ACH payouts (distributions at scale) | SaaS + per-tx [I] | US | Distribution batch status |
| **Modern Treasury** | Payment ops, ledgers, reconciliation, virtual accounts | SaaS [I] | US | **Ledger/reconciliation module** |
| **Checkout.com / Adyen** | Card acquiring, local methods (incl. MENA/EU) | Interchange++ [I] | Global/MENA | — |
| **Circle Mint** | Institutional USDC/EURC mint & redeem | Institutional account [C#15] | Global | Treasury: stablecoin mint/redeem |
| **MoonPay / Mercuryo / Ramp / Transak** | Card/bank → crypto on-ramp & off-ramp widgets | ~1–4.5% per tx [I] | Global | On-ramp order status |
| **Zand / UAE banks (Emirates NBD, Mashreq, RAKBANK)** | AED accounts, escrow for RE, some crypto-friendly corporate banking; Zand active in AED stablecoin/tokenisation [I] | Bank fees [I] | UAE | **Escrow account reconciliation** (DLD escrow rules) [I] |
| **Monoova / Zepto (AU)** | NPP/PayTo real-time payments, virtual accounts | Per-tx [I] | Australia | PayTo mandate status |
| **Wise (TransferWise)** | Cross-border fiat (historical DigiShares integration) | FX margin [I]; DigiShares integration [C#35] | Global | — |

## 6. Stablecoins (settlement & distributions)

| Coin | Issuer / regulation (2026) | Relevance |
|---|---|---|
| **USDC** | Circle (NYSE: CRCL); OCC trust-bank approval Jul 2026; 15+ chains [C#15] | Default USD settlement & rent distribution rail [I] |
| **EURC** | Circle; MiCA-compliant EMT via French EMI [C#15] | EU platforms' euro settlement; 21X uses EMTs for settlement [C#12] |
| **USDT** | Tether (not MiCA-authorised for EU trading venues) [I] | MENA/Asia liquidity; restricted in EU [I] |
| **PYUSD** | PayPal / Paxos [I] | US consumer ramps [I] |
| **EURCV** | SG-Forge (Société Générale) MiCA EMT [I] | EU institutional settlement [I] |
| **RLUSD** | Ripple (NYDFS trust charter), XRPL + Ethereum [I] | XRPL RWA settlement [I] |
| **AE Coin** | AED Stablecoin LLC, **CBUAE-licensed**, 1:1 AED; ADI Chain referenced [C#13] | AED-denominated Dubai RE settlement [I] |
| **AUDD / A$DC** | Novatti (AUDD); ANZ (A$DC) [I] | AU settlement [I] |
| **DAI / USDC / USDT** | — | Blocksquare revenue distribution examples [C#24] |

## 7. Transfer agents & cap-table / registrars

| Vendor | Notes | Evidence |
|---|---|---|
| **Securitize** | SEC TA + BD + ATS + EU DLT Pilot; NYSE-listed Jul 2026; >$4B tokenised | [C#16] |
| **Vertalo** | SEC TA since 2019; ledger *is* the register; Ethereum/Tezos/Aptos; Owner / Transfer Controller / Allowance Controller roles; MCP agent access; $500M+ on-chain, 100+ issuers | [C#17] |
| **Kore (ex-KoreConX)** | TA/cap table/compliance for Reg CF/D/A+; RE solutions | [C#18] |
| **Texture Transfer Services** | SEC TA affiliate of Texture Capital (BD+ATS) | [C#19] |
| **tZERO** | SEC TA + BD (own wallet infra) + ATS | [C#11] |
| **Carta** | Off-chain cap-table/409A/fund admin; no tokenisation [I] | [I] |

## 8. E-signature & documents
- **DocuSign / Dropbox Sign** — both integrated by DigiShares for subscription and shareholder agreements [C#21]. Admin surface: envelope status per investor, template library with variable fields (cf. DigiShares contract module) [I].
- **UAE PASS** digital signature for UAE flows [I].

## 9. Property management (off-chain ops feeding rent roll & distributions)

| Vendor | Segment | Integration value |
|---|---|---|
| **AppFolio** | US residential/commercial PM | Rent roll, expenses → NOI → distribution calc [I] |
| **Buildium** (RealPage) | US small/mid PM | same [I] |
| **Yardi** (Voyager/Breeze) | Institutional PM & accounting | same; GL export [I] |
| **RentRedi** | Small-landlord PM (US) | Rent collection data [I] |
| Admin surface | — | **Rent-roll import / PM sync**, NOI reconciliation, reserve & fee deductions before distribution [I] |

## 10. Oracles, proof of reserve & NAV

| Vendor | Use | Evidence |
|---|---|---|
| **Chainlink** — Proof of Reserve, SmartData/NAV feeds, **ACE (Automated Compliance Engine)** | NAV/valuation on-chain, reserve attestations; ACE demonstrated **ERC-7943** compatibility | ACE/7943 [C#26]; PoR/NAV [I] |
| **"NAVLink"** | Name given in brief; exact provider not verified this session | [S] |
| **RedStone** | Push/pull oracles incl. NAV feeds for tokenised funds | [I] |
| Admin surface | **Valuation module**: publish appraisal/NAV → oracle push, with maker-checker and valuation-report upload (cf. Blocksquare "asset valuation" doc) | [I] |

## 11. Secondary venues & liquidity

| Venue | Type / jurisdiction | Status (2026) | Evidence |
|---|---|---|---|
| **tZERO ATS** | US ATS for tokenised securities ("leader… by total trades"); BD custody; pending CFTC DCO/DCM | Active | [C#11] |
| **Securitize Markets** | US ATS (Securitize) | Active | [C#16][C#29] |
| **INX Securities** | US BD + ATS; subsidiary of Republic | Active | [C#10] |
| **Texture Capital** | US ATS; client-branded marketplaces | Active | [C#19] |
| **Archax** | UK FCA-regulated digital-securities exchange/broker | [I] | [I] |
| **21X** | EU DLT TSS (DLT Pilot Regime); atomic stablecoin settlement; public chain | Live Aug 2025 | [C#12] |
| **SDX (SIX Digital Exchange)** | Swiss CSD + exchange (DLT) | [I] | [I] |
| **BX Digital** | Swiss DLT trading facility (Börse Stuttgart group) | [I] | [I] |
| **InvestaX** | MAS RMO; AMM & liquidity pools | Active | [C#34] |
| **Oceanpoint** | Blocksquare's DeFi liquidity layer; Marketplace Pools; sBST governance | Active | [C#23] |
| **DigiShares internal exchange / SwapContract** | Issuer-controlled P2P swaps vs stablecoins | Active (code Aug 2026) | white-label file |
| **Tokeny DINO network** | Distribution to OTC desks, ATS/MTF, DEX/DeFi for ERC-3643 | Active | [C#30] |
| **Taurus TDX** | Swiss organised trading facility | Active | [C#31] |
| **Ctrl Alt** | VARA broker-dealer (UAE) | Active | [C#20] |
| DEX pools (e.g., RealT YAM on Gnosis; Uniswap-style pools) | Permissionless/permissioned AMMs | [I] | [I] |
| Admin surface | — | Listing approvals, venue connectivity (RFE-style holder sync), trading halts, price bands, fee settings, whitelisted venue/escrow addresses | [I] |

## 12. Data, audits, indexing & nodes

| Category | Vendors | Notes |
|---|---|---|
| RWA data | **RWA.xyz** — tokenised RE: ~$226M distributed / $1.34B represented; 2,851 holders; top platforms Groma, Reental, DigiShares, Securitize, MetaWealth [C#25] | Benchmarking & investor transparency |
| Smart-contract audits | **OpenZeppelin, CertiK, Hacken** (also Halborn audited Securitize DSToken; Trail of Bits audited Dynamic) [I / C#6 for Dynamic audits] | Public T-REX repo issues show ongoing audit findings (e.g., "grant tokenAgents token-operation roles, not AGENT"; "enforce the pause on forced transfers") [C#33 — search-result titles] |
| Indexing | **The Graph** subgraphs; Tokeny indexes 3bn events in-house [I; C#30] | Cap-table rebuild from events |
| Node / RPC | **Alchemy, Infura (Consensys), QuickNode** | Per-compute-unit pricing [I] |

---

## 13. Reference "vendor stack" for RealX (inferred)

| Layer | Primary | Alternative | Why |
|---|---|---|---|
| Token standard | ERC-3643 (+ ERC-7943 views) | ERC-7518 / ATS dual | Identity-bound compliance + recovery; interop [I] |
| KYC/AML | Sumsub (Compliance plan) | Entrust IDV, Persona | Price transparency; travel rule bundled [C#1][I] |
| Accreditation (US) | iCapital Identity (ex-Parallel) | Verify Investor | Reusable passport [C#3][C#4] |
| KYT | Chainalysis or TRM | Elliptic | [I] |
| Custody/signing (issuer ops) | Fireblocks (policy engine = maker-checker) | BitGo, Dfns, Taurus | [C#8][C#14][C#27] |
| Investor wallets | Privy or Dynamic embedded wallets + EIP-7702/4337 sponsorship | MetaMask Embedded Wallets | [C#5][C#6][C#7][C#28] |
| Fiat | Stripe/Plaid (US), Checkout/Adyen + Zand (UAE), Monoova/Zepto (AU) | Modern Treasury ledger | [I] |
| Stablecoin | USDC/EURC; AE Coin (UAE); AUDD (AU) | RLUSD on XRPL | [C#13][C#15][I] |
| TA/registrar (US) | Securitize or Vertalo | tZERO, Texture | [C#16][C#17] |
| Secondary | Internal whitelisted P2P (swap contract) → licensed venue (tZERO / 21X / VARA broker) | Oceanpoint-style pools | [C#11][C#12][I] |
| Oracles | Chainlink PoR/NAV (+ACE) | RedStone | [C#26][I] |
| PM data | Yardi / AppFolio sync | Buildium | [I] |

---

## Sources

| # | URL | Title | Accessed |
|---|---|---|---|
| 1 | https://sumsub.com/pricing/ | Sumsub Pricing | 23 Sep 2026 |
| 2 | https://www.entrust.com/products/identity-verification?edc_redirect=Onfido-Redirect | Entrust Identity Verification (redirect from onfido.com) | 23 Sep 2026 |
| 3 | https://www.parallelmarkets.com/ | Parallel Markets (now iCapital Identity Solutions) | 23 Sep 2026 |
| 4 | https://www.verifyinvestor.com/ | VerifyInvestor | 23 Sep 2026 |
| 5 | https://www.privy.io/ | Privy | 23 Sep 2026 |
| 6 | https://www.dynamic.xyz/ | Dynamic ("acquired by Fireblocks in October 2025") | 23 Sep 2026 |
| 7 | https://web3auth.io/ | Web3Auth / MetaMask Embedded Wallets | 23 Sep 2026 |
| 8 | https://www.fireblocks.com/platforms/governance-and-policies | Fireblocks Governance and Policy Engine | 23 Sep 2026 |
| 9 | https://www.fireblocks.com/blog/a-peek-inside-fireblocks-tokenization-engine-the-backbone-of-your-tokenization-solution | A Peek Inside Fireblocks Tokenization Engine (4 Sep 2024) | 23 Sep 2026 |
| 10 | https://www.inx.co/ | INX | 23 Sep 2026 |
| 11 | https://www.tzero.com/ | tZERO | 23 Sep 2026 |
| 12 | https://www.21x.eu/ | 21X | 23 Sep 2026 |
| 13 | https://www.aecoin.com/ | AE Coin | 23 Sep 2026 |
| 14 | https://en.wikipedia.org/wiki/BitGo | BitGo – Wikipedia | 23 Sep 2026 |
| 15 | https://en.wikipedia.org/wiki/Circle_Internet_Group | Circle Internet Group – Wikipedia | 23 Sep 2026 |
| 16 | https://www.tipranks.com/news/private-companies/securitize-to-list-on-nyse-after-completing-cantor-spac-merger | Securitize to List on NYSE After Completing Cantor SPAC Merger | 23 Sep 2026 |
| 17 | https://www.vertalo.com/ | Vertalo — Agentic Tokenization & Transfer Agency | 23 Sep 2026 |
| 18 | https://kore.inc/ | Kore (redirect from koreconx.com) | 23 Sep 2026 |
| 19 | https://texture.capital/ | Texture Capital | 23 Sep 2026 |
| 20 | https://www.ctrl-alt.co/ | Ctrl Alt | 23 Sep 2026 |
| 21 | https://digishares.io/product-overview/ | DigiShares Product Overview | 23 Sep 2026 |
| 22 | https://docs.blocksquare.io/certified-partners/pricing.md | Blocksquare Pricing | 23 Sep 2026 |
| 23 | https://blog.blocksquare.io/article/rwa-tokenization-explodes-in-2025-blocksquare-surpasses-200-m-in-tokenized-real-estate-fueled-by-marketplace-operators/ | Blocksquare surpasses $200M (Jul 2025) | 23 Sep 2026 |
| 24 | https://docs.blocksquare.io/for-developers/revenue-distribution-contract.md | Blocksquare Revenue Distribution Contract | 23 Sep 2026 |
| 25 | https://app.rwa.xyz/real-estate | RWA.xyz Tokenized Real Estate | 23 Sep 2026 |
| 26 | https://en.cryptonomist.ch/2026/05/26/erc-7943-the-new-universal-standard-for-the-tokenization-of-real-world-assets-on-ethereum/ | ERC-7943… (26 May 2026) | 23 Sep 2026 |
| 27 | https://docs.hedera.com/hedera/open-source-solutions/asset-tokenization-studio-ats | Hedera ATS docs (custody integrations) | 23 Sep 2026 |
| 28 | https://www.xcapit.com/en/blog/ethereum-new-standards-2026-enterprise | New Ethereum Standards in 2026 (13 Feb 2026) | 23 Sep 2026 |
| 29 | https://s3.us-east-2.amazonaws.com/securitizemarkets.io/caseStudies/How-to-Tokenize+Your+Asset+[Miami+Context]+.pdf | How to Tokenize Assets with Securitize (Jan 2022) | 23 Sep 2026 |
| 30 | https://tokeny.com/t-rex-platform/ ; https://docs.tokeny.com/docs/servicing-app | T-REX Platform; Servicing App | 23 Sep 2026 |
| 31 | https://www.taurushq.com/capital/ | Taurus-CAPITAL | 23 Sep 2026 |
| 32 | https://www.junipersquare.com/ ; https://www.investnext.com/ ; https://agorareal.com/ | Off-chain RE investor-management SaaS | 23 Sep 2026 |
| 33 | https://github.com/T-REX-Network/T-REX/issues/57 ; https://github.com/T-REX-Network/T-REX/issues/60 | T-REX GitHub audit issues (search-result titles only) | 23 Sep 2026 |
| 34 | https://investax.io/issuers | InvestaX Issuers | 23 Sep 2026 |
| 35 | https://digishares.io/platformupgrade/ | DigiShares White-Label Multi-Project Platform (TransferWise integration) | 23 Sep 2026 |
| 36 | https://www.fireblocks.com/blog/enhancing-fund-tokenization-operations-and-management-with-fireblocks-x-tokeny | Fireblocks x Tokeny | 23 Sep 2026 |
