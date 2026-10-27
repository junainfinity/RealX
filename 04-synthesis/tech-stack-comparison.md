# Tech-Stack Comparison: 20 Platforms by Layer

**Date:** 23 Sep 2026.

**Evidence key.** Labels are carried from each platform file's §7 (tech stack) unless another section is cited. They are never upgraded: [C] confirmed, [I] inferred, [S] speculative. A dash (—) means the layer was not disclosed and not inferred. File codes are defined in `00-executive-summary.md`. `[I syn]` marks a recommendation made in this synthesis.

The comparison is split into six themed tables to keep them readable. Platform order is US → EU → MENA → AU.

---

## T1. Chain, token standard, permissioning

| Platform | Chain | Token standard | Permissioning / compliance controls |
|---|---|---|---|
| Lofty | Algorand L1 [C us/01 §7] | ASA per property [C] | ASA clawback/freeze for KYC gating, the 15% cap and recovery [I]. PMM app enforces a 20% staking cap [C] |
| RedSwan | Hedera (since 2023), Stellar (2025); legacy Polymath/Ethereum [C us/02 §7] | HTS native tokens; HSCS ERC-20-compatible; Stellar assets [C]; ST-20 in the past [I] | HTS KYC grant and custom fees [C]; freeze and Stellar auth flags [I] |
| Arrived | None [C us/03 §7] | Book entry at the TA [C] | Application rules; ATS eligibility (6-month hold) [C] |
| RealT | Gnosis (primary), Ethereum [C us/04 §7] | ERC-20 with transfer restrictions [C] | Whitelist per wallet and per property [C]. Mt Pelerin "Bridge Protocol" lineage [S] |
| mogul | Avalanche C-Chain [C us/05 §7] | ERC-1155-compatible custom NFT [C] | Off-chain transfer consent [C]; mogul's Fireblocks signer controls minting [I] |
| Reental | Polygon PoS; RNT also on BNB [C eu/01 §7] | ERC-20 per property [I]; RNT is ERC-20 [C] | Allow-list on the platform, probably not enforced in the token [I] |
| Blocksquare | Ethereum mainnet [C eu/02 §7]; "multi-chain" claimed [I] | Custom ERC-20 BSPT: 100,000 per property, 18 decimals; **not ERC-3643** [C] | PermissionRoles, CertifiedPartners and Users whitelist contracts; `freezeToken` [C] |
| Propvest | Ethereum [C eu/03 §7] | Not disclosed [S: ERC-20 with restrictions] | Transfers happen inside the custodian (Tangany) [C/I] |
| Brickken | Ethereum, Base, BNB, Polygon (+ MANTRA, XDC) [C eu/04 §7] | ERC-20 with per-token whitelist; **co-author of ERC-7943** [C]. Whether it uses 7943 in production is [I] | Factory clones plus UUPS proxies [C, 2021 whitepaper]. Whitelist state machine [C] |
| Domoblock | Polygon [C eu/05 §7] | "Fan token", standard undisclosed [C/S] | Transfers only by approved loan assignment [C] |
| PRYPCO Mint | XRP Ledger [C me/01 §7] | XRPL issued asset; IOU vs Multi-Purpose Token not published [I] | Omnibus custody plus off-chain rules engine; ARVA management-token layer [C/I] |
| Ctrl Alt | XRPL (DLD, diamonds); Solana; Algorand (legacy) [C me/02 §7] | XRPL tokens; ASA (legacy) [C/I] | ARVA module plus rules engine (lock-in, price band, 20% cap) [C/I] |
| Stake | None; tokenisation planned [C me/03 §7] | — | Off-chain limits engine [C] |
| Saudi RER + operators | Permissioned EVM on SettleMint DALP [I me/04 §7] | **"Enhanced ERC-3643"**: 1:1 deed token plus fractional tokens [C] | 50+ smart contracts covering identity, transfer restrictions, eligibility and geo-fencing [C] |
| SmartCrowd | None [C me/05 §7] | — | Off-chain; STF ±15% validator [C] |
| BrickX | None [C au/01 §7] | Units on an internal register [C] | Application rules: 5% cap, residency, 80% price floor, halts [C] |
| Redbelly | Redbelly L1 (DBFT consensus, scalable EVM) [C au/02 §7] | Not set at network level; Tokeniser uses ERC-3643 [C] | W3C verifiable credentials, DIDs, ZK proofs, Receptor ID protocol [C] |
| Tokeniser + Liquidise | Redbelly; Liquidise also on Ethereum and Arbitrum [C au/03 §7] | **ERC-3643 + ERC-7575** [C] | ERC-3643 transfer hooks plus Averer credentials and ZK proofs [C] |
| Canvas Connect | Permissioned EVM Layer 2; settlement L1 undisclosed [C/S au/04 §7] | Undisclosed [I: allowlisted Solidity tokens] | Identity, role and allowlist controls; ZK privacy patterns [C] |
| Assetora + Bricklet | None for the fund. Bricklet: unnamed DLT [C au/05 §7] | Not applicable | App-level: verified members, AU residents, whitelist of tradeable sub-funds [C] |

## T2. Identity / KYC and wallets / custody

| Platform | Identity / KYC vendor | Wallet / custody |
|---|---|---|
| Lofty | Undisclosed (document + selfie) [C flow / S vendor] | App-generated Algorand account ("Lofty Wallet"); ToS says non-custodial, one reviewer says custodial. Pera/Defly supported [C us/01 §7] |
| RedSwan | Unnamed KYC, AML, OFAC and accreditation vendors [C exist / S names] | Coinbase custody partner [C]; HashPack/Freighter self-custody [S] |
| Arrived | Unnamed; possibly the NCPS Transact API [I]; Plaid [C] | No tokens. Investor funds held by Modern Treasury [C] |
| RealT | Undisclosed [S] | RealToken AA wallet (social login, RealT Guardians) plus external wallets via RainbowKit [C] |
| mogul | Unnamed; SSN-based [C/S] | Fireblocks, custodial [C/I] |
| Reental | Undisclosed | **HumanWallet**: ERC-4337 smart account, WebAuthn passkey, ERC-7212, paymaster. Also WalletConnect wallets [C] |
| Blocksquare | Automated ID + selfie + liveness; unnamed [C] | **Alchemy account-abstraction** smart wallets plus external wallets; up to 10 per account [C] |
| Propvest | Unnamed identity service [C]; VideoIdent [S] | **Tangany** crypto custody (originally Upvest); Baader Bank depot [C] |
| Brickken | **Sumsub** [C] | Non-custodial (MetaMask/WalletConnect); transactions signed locally [C] |
| Domoblock | DNI + selfie, KYB; vendor unknown [C] | Mangopay custodial EUR wallet; MetaMask for DAI [C] |
| PRYPCO Mint | Emirates ID; **UAE PASS** login [C]; KYC vendor undisclosed | Ctrl Alt **warm MPC omnibus on Ripple Custody** [C] |
| Ctrl Alt | Delegated to distributors [C] | Ripple Custody MPC; legacy self-custody on Algorand [C] |
| Stake | **IDWise** [C] | Custodial AED/SAR wallet ledger; client money at Emirates NBD [C] |
| Saudi RER + operators | **Nafath, Absher, Yakeen**; W3C VC / eIDAS 2.0 [C] | Registry-held tokens. Custody provider undisclosed; DALP supports Fireblocks, Ripple/Metaco, DFNS, Utila, HSM [C/I] |
| SmartCrowd | Undisclosed | Custodial eWallet [C] |
| BrickX | **Equifax ID Matrix** [C] | Custodial "Digital Wallet" (client money held on trust); AET is asset custodian [C] |
| Redbelly | Credential issuers, unnamed [C] | Fireblocks MPC [C] |
| Tokeniser + Liquidise | **Averer** VerifyKit (VC + ZK) [C] | Averer non-custodial embedded wallet with account and gas abstraction [C] |
| Canvas Connect | Unnamed IDV, sanctions, chain-analytics and travel-rule vendors [C] | **Zodia Custody AU, Fireblocks** [C] |
| Assetora + Bricklet | Undisclosed [S] | Custodial cash account; Perpetual is custodian [C] |

## T3. Fiat rails and stablecoins

| Platform | Fiat rails | Stablecoin / settlement asset |
|---|---|---|
| Lofty | Stripe card and ACH; wire above $50k [C] | USDC on Algorand [C] |
| RedSwan | USD wire/ACH via escrow [C/I] | USDC on Hedera (hUSDC) [C] |
| Arrived | Plaid, Stripe, Dwolla, NCIT [C] | None [C] |
| RealT | Card (processor undisclosed; 1.5% fee to buyer) [C] | USDC, xDai/WXDAI; Request Finance; REUSD vouchers [C] |
| mogul | Plaid ACH, card, wire, PayPal, Venmo [C] | USDT on Avalanche (deposits) [C] |
| Reental | Manual EUR/USD bank transfer with proof upload; USDT-to-bank withdrawal for higher tiers [C] | USDT / USDT0 [C] |
| Blocksquare | EUR bank integration, set up per operator [C] | DAI / USDC / USDT [C] |
| Propvest | Mangopay payment accounts (SEPA) [C] | None; EUR only [C] |
| Brickken | Through partners [C] | USDC / USDT (`acceptedCoin`) [C] |
| Domoblock | Mangopay SEPA and card (0.10%) [C] | DAI, max €10k per transaction [C] |
| PRYPCO Mint | Zand client-money account; bank transfer, debit/credit card, Apple Pay; AED only [C] | None for real estate; PAXG for the gold product [C] |
| Ctrl Alt | Distributor handles fiat; legacy UK used trustshare/PayrNet [C] | None on DLD rails [C] |
| Stake | Bank transfer; debit card (credit card for Professionals) via **Checkout.com**; Wise [C] | None [I] |
| Saudi RER + operators | **mada, SADAD**; payment verification linked to escrow [C] | None [C] |
| SmartCrowd | **Lean Payments** open-banking pay-by-bank (AED/USD); Egypt: cards and instalments [C] | None |
| BrickX | **PayID/NPP** (since Aug 2025); direct debit; legacy BPAY/POLi [C] | None [I] |
| Redbelly | Issuer-side | **AUDM** (Macropod, AFSL 566313); RBA wholesale-CBDC pilot [C] |
| Tokeniser + Liquidise | Not specified [I] | USDT, USDC, LIQ [C]; AUDM available [I] |
| Canvas Connect | NAB, Fiserv, Banking Circle (ESA) [C] | RBA wholesale-CBDC pilot; eAUD + USDC (2023); AUD/USDT via Canvas Prime [C] |
| Assetora + Bricklet | Bank transfer (historically ANZ) [C] | None [I] |

## T4. Marketplace / AMM and oracles / valuation

| Platform | Marketplace / liquidity tech | Oracles / valuation |
|---|---|---|
| Lofty | Hybrid limit order book plus PMM pools (Algorand apps) [C] | None disclosed; PMM price reference unknown [S] |
| RedSwan | Primary market only [C] | — |
| Arrived | PPEX ATS in monthly windows, T+1/T+2 [C] | Internal quarterly NAV (manager-set) [C] |
| RealT | **YAM** (Solidity/Hardhat, admin token whitelist); Levinswap/Swapcat; **RMM v3** (Aave v3 fork) [C] | Prices set by RealT on reappraisal; the lending market relies on them [I] |
| mogul | None [C] | Monthly valuations from "appraisal-level" data [C us/05 §2.8] |
| Reental | RNT P2P; RNTlend (Aave-based); SushiSwap RNT pool; Aragon OSx DAO [C] | None named; admin-set NAV likely [I] |
| Blocksquare | P2P across marketplaces; Oceanpoint pools; Uniswap v2 BST/ETH; POINT aggregator planned [C] | Asset-valuation document process [C inf/wl §5] |
| Propvest | Custom bilateral board (Handelsplatz) [C/I] | External appraiser (criticised) [C eu/03 §2.8] |
| Brickken | Partner venues [C] | **Chainlink** proof of reserve, CCIP, ACE [C] |
| Domoblock | None [C] | AI micro-location pricing model [C] |
| PRYPCO Mint | In-app 24/7 listing board [C] | **DLD smart valuation** sets the ±15% band [C] |
| Ctrl Alt | Settlement layer for distributors [C] | Takes DLD valuation [I] |
| Stake | Exit Window engine (twice a year) [C] | Third-party valuer every six months; Reidin data [C] |
| Saudi RER + operators | National marketplace (Inspire), in development [C] | Automated valuation models built into the registry stack; REGA price controls [C] |
| SmartCrowd | Share Transfer Facility (March and September) [C] | "Current market price" reference ±15% [C] |
| BrickX | Proprietary order book with price-time priority [C] | Independent valuer every six months [C] |
| Redbelly | Credentialed P2P; Liquidise AMM [C] | Real-time NAV unit pricing (JellyC); vendor unknown [C] |
| Tokeniser + Liquidise | AMM pools, "protocol finance", Liquidise proprietary AMM [C] | NAV-anchored pools possible [I] |
| Canvas Connect | DvP, repo and collateral contracts [C] | — |
| Assetora + Bricklet | Proprietary list/accept marketplace [C] | NAV per sub-fund [I] |

## T5. Front end, back end and cloud

| Platform | Frontend | Backend / API | Cloud / hosting |
|---|---|---|---|
| Lofty | Web app; framework unknown [S] | Needs order matching, an indexer and a distribution cron [I] | — [S] |
| RedSwan | Marketing site on WordPress [C]; app unknown | — | — |
| Arrived | Web plus iOS/Android [C] | — | Unnamed cloud hosting [C] |
| RealT | TypeScript (yam-interface, realt-interface-commons) [C] | The Graph; Chainlink CCIP; Hardhat [C] | — |
| mogul | Web app; docs on a GitBook-style site with llms.txt [C] | — | — |
| Reental | Viem + Wagmi [I]; Webflow marketing site [I] | Node/TypeScript [S] | — |
| Blocksquare | Multi-tenant web app at app.blocksquare.io/&lt;tenant&gt; [C/I] | Solidity 0.6.12; Envio-based TypeScript indexer [C] | — [S] |
| Propvest | Vue SPA marketplace [I]; native iOS/Android apps [C] | — | AWS [S] |
| Brickken | Micro-frontends [C, 2021] | **AWS Lambda microservices**, Infura; REST (OpenAPI), TypeScript SDK, CLI, MCP server, x402 payments [C] | AWS [C, 2021] |
| Domoblock | Web app with mandatory 2FA [C] | In-house team [C] | — |
| PRYPCO Mint | **React Native + Expo**, TypeScript, Redux/Zustand, EAS [C] | Firebase and/or GraphQL/REST [I] | Unknown; data may be stored outside the Middle East [C/S] |
| Ctrl Alt | — | Distributor and registry API gateway [I] | — |
| Stake | iOS/Android plus web [C] | — | — |
| Saudi RER + operators | Operator mobile apps (e.g., Jozo) [C] | SettleMint DALP orchestration; RER Aamal APIs [C] | **Kubernetes in Saudi data centres; NCA cybersecurity standards** [C] |
| SmartCrowd | iOS/Android plus web [C] | — | — |
| BrickX | In-house web / mobile web; small team [C] | — | Offshore SaaS/cloud [I] |
| Redbelly | — | SDKs [C] | Routescan explorer [C] |
| Tokeniser + Liquidise | Averer React SDK; Liquidise site built with Lovable [C] | Public API add-on [C] | Overseas providers [C] |
| Canvas Connect | Apps and APIs [C] | RPC interfaces [C] | Data processed in LU, US, UK, SG, DK [C] |
| Assetora + Bricklet | Native iOS/Android (4.9★), biometric login; static HTML marketing site [C] | Bricklet runs on a Microsoft stack [C] | Onshore (no overseas recipients) [I] |

## T6. Analytics / CRM / support, e-signature, property management, audits

| Platform | Analytics / CRM / support | E-signature | Property-management integration | Audits / security |
|---|---|---|---|---|
| Lofty | Google Analytics/Ads, GTM, Meta and X pixels; Community channel plus email [C] | — | Seller or third-party PM (HomeRiver) [C] | Dropped the MyAlgo wallet after its exploit [C us/01 §10] |
| RedSwan | — | PPM e-signing [C us/02 §4] | Sponsor / operator | — |
| Arrived | Google Analytics; unnamed chat providers [C] | — | Mynd, Marketplace Homes [C] | — |
| RealT | Community dashboards (third party) [C] | — | Third-party PM, then in-house "New Detroit PM" [C] | **ABDK** audit (REG/CCIP); Slither on YAM [C] |
| mogul | Calendly for consultations [C] | Click-through company agreement [C us/05 §5.2] | Licensed PMs, unnamed [C] | — |
| Reental | Discord, Telegram, WhatsApp; Google Drive transparency portal [C] | Present; vendor unknown [C] | City managers [C] | — |
| Blocksquare | DefiLlama adapter; Snapshot voting [C] | — | Legal Issuer self-reports revenue [C] | Etherscan-verified contracts [C] |
| Propvest | **Salesforce, Braze, New Relic**, GTM, Usercentrics, Zenloop, Trustpilot, Personio [C] | — | Sonar Real Estate [C] | — |
| Brickken | — | — | Issuer-defined | **ISO 27001 / 27701 / 27018**; "DORA-ready" [C] |
| Domoblock | Trustpilot [C] | Loan contracts signed digitally [C eu/05 §4] | In-house works team [C] | Mandatory 2FA [C] |
| PRYPCO Mint | **Sentry, Amplitude, GTM**; AI support bot plus email [C] | UAE PASS signing possible [I inf/ven §8] | In-house tenancy management (Ejari) [C] | — |
| Ctrl Alt | — | — | Distributor | — |
| Stake | **Customer.io, Segment**, Google Analytics; Intercom-style help centre [C/I] | — | In-house PM [C] | — |
| Saudi RER + operators | — | — | Operator; Ejar contracts [I] | NCA standards [C] |
| SmartCrowd | Intercom-style help centre [I] | — | PM at ~5% of rent [C] | — |
| BrickX | **Freshdesk**; news CMS (WordPress) [C/S] | Online acceptance [I] | Third-party agents; manual or CSV integration [I] | — |
| Redbelly | Routescan [C] | — | Hutly rent-roll tokenisation [C] | — |
| Tokeniser + Liquidise | Zoho Bookings [C] | — | — | — |
| Canvas Connect | Employment Hero (HR) [C] | — | — | ZK privacy (eAUD pilot) [C] |
| Assetora + Bricklet | In-app messaging; podcast; BoardRoom (share registry of the listed parent) [C] | — | Asset managers; vacancy alerting was missing (NDIS) [C/I] | — |

---

## Observed patterns

1. **EVM dominates the token layer, with chain-native exceptions.**
   - EVM: 11 of the 15 platforms with an on-chain component (RealT, mogul, Reental, Blocksquare, Propvest, Brickken, Domoblock, Saudi RER, Redbelly, Tokeniser, Canvas). The four non-EVM platforms are Lofty, RedSwan, PRYPCO and Ctrl Alt.
   - Chain-native primitives: Algorand ASA (Lofty), Hedera HTS / Stellar (RedSwan), and XRPL (DLD rails: PRYPCO, Ctrl Alt) [C per file §7].
   - Regulators and registries picked their own chains: DLD chose XRPL; Saudi RER chose permissioned EVM on SettleMint [C me/02 §7; C me/04 §7].
2. **ERC-3643 is adopted by infrastructure and sovereign stacks, not by retail leaders.**
   - Tokeniser and Saudi RER ("enhanced ERC-3643") use it [C au/03 §7; C me/04 §7].
   - US and EU leaders use custom whitelists or chain primitives [C us/sum §7.6; C eu/sum §6.6].
   - ERC-7943 is emerging as the interoperability layer [C inf/std §1.7].
3. **Wallets split three ways:**
   - Non-custodial smart or embedded wallets: Reental, Blocksquare, RealT, Tokeniser.
   - Custodial MPC: mogul/Fireblocks, PRYPCO/Ripple, Propvest/Tangany, Canvas/Zodia.
   - Cash-only ledgers: all off-chain benchmarks.
   - Licensed platforms and the MENA region default to custodial [C per file §7].
4. **Fireblocks is the most-cited institutional custody vendor** among the 20: mogul, Redbelly, Canvas. It also has the only public evidence of maker-checker policy [C inf/wl §12].
5. **KYC vendors are mostly hidden.** Only Sumsub, IDWise and Equifax are named, and the rest are undisclosed. Government ID dominates in the GCC: UAE PASS, Nafath [C eu/04 §2.5; C me/03 §2.5; C au/01 §2.5; C me/01 §2.5; C me/04 §2.5; see compliance-matrix.md (d)].
6. **Fiat rails are regional and licensed:**
   - US: Stripe, Plaid, Dwolla, Modern Treasury, NCIT.
   - EU: Mangopay.
   - UAE: Zand, Emirates NBD, Checkout.com, Lean.
   - KSA: mada, SADAD.
   - AU: NPP/PayID [C per file §7].
7. **Stablecoin choice is legacy-DeFi (USDT, DAI, xDai) and does not match the 2026 regulated allow-lists:** EURC under MiCA, GENIUS-permitted USD coins, CBUAE-licensed AE Coin, AUDM/AUDD under ASIC relief [C inf/ven §6; C reg/x A3 #19].
8. **Oracles are the weakest layer.** Most valuations are set by the issuer or manager (RealT, Arrived, Reental). Exceptions:
   - Official valuations: DLD smart valuation, Saudi automated valuation models [C me/01 §7; C me/04 §7].
   - Independent valuers every six months: BrickX, Stake [C au/01 §2.4; C me/03 §2.4].
   - Only Brickken integrates Chainlink proof of reserve [C eu/04 §7].
9. **No token platform integrates a property-management system.** Rent-roll import is manual. AppFolio/Yardi/Buildium sync is a gap across all token vendors [C inf/adm gaps].
10. **Security evidence is thin.** Only RealT (ABDK audit) and Brickken (ISO certifications) publish security evidence. Upgradeability sits behind admin keys at Brickken (UUPS) and RealT (upgradeable REG) [C us/04 §7; C eu/04 §7].

---

## Recommended default stack

### Global core: the same in every region [I syn]

| Layer | Default | Alternatives | Reasoning / evidence |
|---|---|---|---|
| Token standard | **ERC-3643**, also exposing **ERC-7943** views | Hedera ATS (dual ERC-1400/3643); ERC-7518 for partition-heavy deals | ERC-3643 is final and the de-facto standard, with ~US$32B claimed and DTCC in the association; 7943 adds venue and DeFi interoperability [C inf/std §0, §1.4, §1.7] |
| Issuer custody and signing | **Fireblocks** (tokenisation engine plus policy engine) | BitGo (OCC charter), Dfns, Taurus-PROTECT | The policy engine gives quorum approvals for maker-checker [C inf/wl §12]; BitGo holds an OCC charter [C inf/ven §4] |
| Investor wallets | Embedded smart wallets (Privy or Dynamic) with an EIP-7702/4337 paymaster and guardian recovery | Custodial MPC omnibus where the regime requires it (MENA) | No-seed-phrase UX converged [C eu/01 §7; C eu/02 §7]; EIP-7702 live since May 2025 [C inf/std §1.8]; Dynamic acquired by Fireblocks [C inf/ven §0] |
| KYC / AML / travel rule | **Sumsub** (Compliance plan, $1.85 per verification, travel rule bundled) | Entrust IDV (ex-Onfido), Persona | Public pricing; travel rule included [C inf/ven §1] |
| Wallet KYT | Chainalysis or TRM | Elliptic | Wallet screening against OFAC and other lists [C reg/us §4; I inf/ven §3] |
| Oracles / NAV | **Chainlink** proof of reserve / NAV feeds (+ ACE) fed by an independent valuer, with maker-checker | RedStone | Avoids RealT's issuer-oracle failure [I us/04 §9]; ACE is 7943-compatible [C inf/ven §10] |
| Indexing | Envio or The Graph, plus an event store for point-in-time cap tables | In-house indexer | Blocksquare moved to Envio [C eu/02 §7]; historical cap tables at Tokeny [C inf/wl §2] |
| Ledger / reconciliation | Double-entry ledger with virtual accounts, Modern Treasury-style | In-house | Arrived uses Modern Treasury; reconciliation is the core control [C us/03 §2.4; C reg/x A5 #25] |
| E-signature | DocuSign or Dropbox Sign; UAE PASS signing in the UAE | — | DigiShares integrates both [C inf/wl §1] |
| Property-management integration | Yardi / AppFolio / Buildium connectors (US) with a rent-roll import standard elsewhere | CSV import with validation | This gap exists at every token vendor [C inf/adm gaps] |
| CRM / messaging / observability | Salesforce or HubSpot; Braze or Customer.io; Intercom; Sentry; Amplitude | — | As seen at Propvest, Stake and PRYPCO [C eu/03 §7; C me/03 §7; C me/01 §7] |
| Audits | Two independent smart-contract audits before mainnet; ISO 27001 roadmap | — | RealT used ABDK; Brickken holds ISO certifications [C us/04 §2.4; C eu/04 §2.8] |
| Contracts | ERC-3643 proxies behind a timelock plus multisig; CREATE2 same-address multi-chain deploys | Non-upgradeable plus a migration path | [C inf/wl §2; see `reference-architecture.md` (d)] |

### Region-specific layers [I syn]

| Layer | US | EU | UAE (Dubai) | KSA | Australia |
|---|---|---|---|---|---|
| Chain | EVM L2 (Base) or Polygon PoS | Polygon PoS or Ethereum; a DLT Pilot venue chain if listing | **XRPL on DLD rails (fixed by DLD / Ctrl Alt)**; EVM for non-DLD assets | Whatever RER / SettleMint runs; integrate via Aamal APIs | **Redbelly** (compliance-native, carries AUDM) or an EVM L2; same address via CREATE2 |
| Golden record | SEC-registered TA (Securitize, Vertalo, Texture) | Registrar under national DLT law (Spain, Germany), or a registrar entity | DLD registry via Ctrl Alt's transfer agency | RER | RE unit register (IQ-EQ-type registry) |
| Identity | Sumsub or Persona + iCapital/Verify Investor for accreditation | Sumsub or Veriff | **UAE PASS / Emirates ID** + Sumsub | **Nafath** + Yakeen | Averer credentials, or Equifax/eIDV via DVS |
| Fiat | Plaid + Stripe + Modern Treasury (funds held by a third party) | **Mangopay / Lemonway** (EMI) | Zand or Emirates NBD client money; Checkout.com; Lean | mada / SADAD with escrow | Monoova or Zepto (NPP / PayTo) |
| Stablecoin | USDC, only once GENIUS-permitted (~Jan 2027) | **EURC / EURCV** (MiCA EMTs) | AE Coin (CBUAE-licensed AED) | None yet | AUDM / AUDD |
| Secondary venue | Partner ATS: tZERO, Securitize Markets, Texture | Bulletin board, then 21X / DLT Pilot venue | DLD Phase 2 marketplace, within the distributor licence | National marketplace (Inspire), when live | Bulletin board or ASIC relief, then a licensed market |
| Tax engine | 1099-DIV / 1099-DA / 1042-S; FIRPTA and §1446(f) | National withholding (e.g., ES 19%); CRS / DAC8 | CRS; VAT on fees | RETT 5%; Zakat | AMIT / AMMA; MIT withholding; TFN |

**Reasoning by region:**
- **US: fewest moving parts to reach a compliant product.** A registered TA is the golden record, which the Sep 2026 proposal would formalise [C reg/us §5]. ACH plus a third-party funds holder avoids money-transmitter status [I us/03 §2.5]. A partner ATS supplies the secondary market. USDC waits for GENIUS permitting.
- **EU: MiCA-compliant rails from day one.**
  - EMI partners are standard [C eu/sum §6.5].
  - EURC avoids USDT's MiCA problem [I eu/01 §2.8].
  - A DLT Pilot venue (21X, live since Aug 2025) is the only licensed on-chain secondary [C inf/ven §11; C reg/eu §6].
- **UAE: no chain choice as a distributor.** XRPL, Ripple Custody and DLD sync are fixed by the rails [C me/02 §7].
  - RealX differentiates in UX, liquidity (bids, market-making) and transparency, not infrastructure [C me/01 §12].
  - AED stablecoins must be CBUAE-licensed [I reg/me §4].
- **KSA: the registry runs the stack.** RealX's build is an operator back office plus API integration [C me/04 §11.1].
- **AU: choose between an ecosystem and liquidity.**
  - Redbelly offers an Australian compliance-native stack (verifiable credentials, ZK proofs, AUDM, the Acacia track record) [C au/sum §5.4].
  - A global EVM L2 offers wider liquidity and tooling.
  - Deploying the same ERC-3643 contracts to both keeps the option open [I syn].
