# White-Label & Issuer Dashboards — Publicly Documented Admin Surfaces

**Purpose:** Consumer tokenisation platforms keep their admin panels private, but the white-label vendors they build on publicly document issuer/admin dashboards. This file harvests that evidence (product pages, docs, GitHub, API refs, blog release notes) as the grounding for RealX's extrapolated platform-owner admin panel.
**Date:** 23 Sep 2026. **Labels:** [C#n] confirmed (Sources at end), [I] inferred, [S] speculative.
**Note on screenshots:** no images embedded; where docs/pages describe screens, their described contents are quoted.

---

## 0. Summary table

| Vendor | Type | Chain / standard | Admin-panel evidence quality | Real-estate relevance | 2025–26 status |
|---|---|---|---|---|---|
| **DigiShares** | White-label STO / RE tokenisation platform | Ethereum, Polygon (+Stellar historically); **ERC-1404** contracts on GitHub; atomic **SwapContract** for P2P | Medium (product pages, release notes, GitHub) | High — RE is core use case; #3 on RWA.xyz RE league table (~US$20M) | Active; GitHub updated Apr–Aug 2026; ERC-7943 coalition member [C#7][C#10][C#11] |
| **Tokeny (T-REX Platform)** | White-label issuance + servicing SaaS | EVM; **ERC-3643** + ONCHAINID | **High** (public docs.tokeny.com) | Medium (funds, RE via clients) | Part of **Apex Group**; 120+ issuers [C#15] |
| **Securitize** | Issuer platform + TA + BD + ATS | Multi-chain; **DS Protocol** (permissioned ERC-20) | Medium (Control Panel release notes, DS Protocol GitHub) | Medium (RE via funds) | **Listed NYSE: SECZ on 2 Jul 2026** via Cantor SPAC [C#21] |
| **Brickken** | Self-serve Issuer Studio (SaaS) | Multi-chain EVM; integrating **ERC-7943** | Medium (pricing tiers, screenshots described) | High (EU RE issuers) | Active [C#11][C#22] |
| **Blocksquare** | RE-only tokenisation protocol + white-label marketplaces | Ethereum/Polygon-family [I]; BSPT property tokens (100,000 = 100%) | **High** for RE (GitBook docs, contracts) | **Very high** — RE-native | >US$200M tokenised RE (Jul 2025) [C#23] |
| **Zoniqx** | TPaaS; Z360 dashboard; zConnect distribution | 14+ chains; **ERC-7518 DyCIST** | Low–medium (marketing) | High (RE partitions) | Active [C#30][C#31] |
| **Stobox** | Stobox 4 platform + STV3 protocol + DID | Arbitrum; Diamond (EIP-2535) ERC-20 | Medium (release notes) | Medium | Release 2.0 on 13 Aug 2025 [C#33] |
| **Vertalo** | SEC-registered digital transfer agent + cap table | Ethereum, Tezos, Aptos | Medium | Medium (RE issuers via BDs) | Repositioned "Agentic Tokenization & Transfer Agency" with MCP [C#34] |
| **InvestaX** | MAS-licensed (CMS + RMO) issuance + marketplace | Ethereum, Algorand, Tezos, Hedera, STACS | Low | Medium (fractional RE listed as structure) | Active [C#36] |
| **Polymesh** | Securities L1 with native issuer primitives | Polymesh | Medium (dev docs) | Low–medium | Active [C#37] |
| **Hedera ATS** | Open-source issuer web app + SDK | Hedera EVM; **ERC-1400 + ERC-3643** | **High** (docs, roles) | Medium | Dual standard Nov 2025 [C#39][C#40] |
| **Fireblocks Tokenization** | Custody-native token console + policy engine | 35+ chains | Medium | Infra | Acquired Dynamic (Oct 2025) [C#41][vendor-landscape] |
| **Taurus-CAPITAL** | Bank-grade tokenisation module (+PROTECT custody, TDX) | CMTAT, ERC-3643, ERC-1400, SPL | Medium | Low–medium (RE NFTs supported) | State Street, Deutsche Bank, CACEIS clients [C#43] |
| **KoreConX (now Kore / KoreInside)** | TA + cap table + compliance for Reg A/CF/D | KoreChain | Low (site redirects, thin) | Medium (RE solutions page) | Rebranded; koreconx.com → kore.inc [C#44] |
| **Texture Capital** | BD + ATS + TA (Texture Transfer Services) | — | Low | Low–medium | Active [C#45] |
| **Ctrl Alt** | Institutional tokenisation infra, VARA-licensed | — | Low | **High (Dubai RE)** | VARA licence 24 May 2025; US$1.8bn+ tokenised [C#46] |
| **Juniper Square / InvestNext / Agora** | Off-chain RE investor-management SaaS (comparators) | n/a | Medium (module lists) | **Very high** (RE GP ops) | All added AI/MCP layers 2025–26 [C#47][C#48][C#49] |

---

## 1. DigiShares (priority — white-label real-estate tokenisation)

**Positioning & pricing (public):** Tiered SaaS [C#1][C#5]:

| Tier | Setup | Monthly | Extra project | Users | Notable |
|---|---|---|---|---|---|
| Launch | €5,000 | €300 (2 projects) | €200 | ≤200 | Quarterly dividends, internal marketplace, 50 KYC incl. then €2/KYC |
| Standard | €35,000 | €950 (2 projects) | €60 | ≤2,000 | Monthly dividends, **advanced platform admin dashboard**, **white-label (logo, colours, custom URL)**, secondary-trading connectivity, fiat payment options |
| Advanced | €50,000+ (incl. 20h onboarding) | €1,500 | €60 | ≤10,000 | Unlimited dividends, **API (1M calls/month)**, full UI/UX customisation, custom mobile apps |

**Admin/issuer modules documented:**
- **Top-level administrator dashboard** managing all STOs issued by the same corporation; **each STO project has its own dashboard, URL and investor database** (multi-project / multi-SPV) [C#4][C#3].
- **Security definition**: "define the security to be tokenized" with **multiple share classes** [C#4]; securities supported: shares, bonds, loans, profit-sharing & dividend rights [C#3][C#6].
- **Token & investor management**, **share cap table** ("100% updated token holder cap tables"), supports both tokenised and **non-tokenised (off-chain) investors** on the same register [C#1][C#3].
- **Investor onboarding**: KYC/AML, **e-signature (DocuSign / Dropbox Sign)**, wallet connection (WalletConnect), payments on-chain or traditional (card, bank transfer, crypto; historically TransferWise/Wise) [C#1][C#2][C#3][C#6].
- **Contracts module**: create customised contracts in-system with **variable fields** (investor name, number of shares, date), distribute to a **selected subset of investors**, investors can review/suggest changes [C#4]. Contract workflow automation for subscription forms, purchase receipts, shareholder agreements [C#6].
- **Meetings & voting**: meetings with agendas; per-item votes; **vote weighting by shareholder rights**; invite external non-shareholders [C#4].
- **Communications**: mass email to investors; **administrator inbox** for investor messages; company news distribution [C#4][C#3].
- **Dividends distribution** (frequency gated by tier) [C#1].
- **Internal exchange / mini-exchange (OTC)**: investors trade with each other; admin **invites and whitelists investors into the exchange** [C#3][C#4].
- **Token freezing / renewal** [C#3].
- **Content management (CMS)**: upload content to design STO user-facing pages, varied by **jurisdiction × investor domicile** [C#3]; project documentation upload (files, media, text, images, videos) [C#1].
- **Security**: 2FA [C#4].

**On-chain layer (GitHub, updated 2026):** [C#7][C#8][C#9]
- `ERC1404-v1.3` (Apr 2026): ERC-20 + ERC-1404; **per-address `_buyRestriction` / `_sellRestriction` epoch timestamps** (lock-ups: both parties must be past their timestamps), global **`isTradingAllowed`** kill-switch, **whitelist authority** role (owner by default), owner-only mint/burn; OpenZeppelin-based. No documented forced transfer or max-holder rule [C#8].
- `ABT20` (Apr 2026, MIT) — purpose not documented [C#7].
- `SwapContract` (Aug 2026): P2P secondary trading — `open` (seller deposits security token), `close` (buyer pays stablecoin minus fees, before expiry), `expire` (reclaim); **PERMIT2** approvals; **owner manages whitelists of "opening tokens" (security tokens) and "closing tokens" (stablecoins), fee structure, fee recipients and expiry** [C#9].
- ERC-7943 coalition member [C#11]; RWA.xyz lists "DigiShare" ~US$20.0M (1 asset) in tokenised RE [C#10].

**Real-estate workflow (DigiShares' own best-practice page):** jurisdiction selection → **SPV creation + bank account** → fund-raise design by amount/investor type → security design → KYC → reporting to transfer agents/CSDs → legal docs (PPM/prospectus) → KYC customisation → contract workflow → **property presentation (photos, financials)** → payment options → custody partner → server install + **administrator training** → test → launch; typical STO 2–3 months [C#6].

**Inferred admin pages for RealX:** Projects (SPVs) list → Project settings (share classes, pricing, docs, CMS) → Investors (KYC queue, whitelist to exchange) → Contracts → Payments → Cap table → Dividends → Meetings/Votes → Exchange (whitelists, fees, expiry) → Communications (mass mail, inbox) → Platform settings (branding, domains, 2FA) [I].

---

## 2. Tokeny — T-REX Platform (Servicing App + Investor App)

**Status:** Tokeny is **part of Apex Group**; 120+ token issuers; "1000+ features"; 3bn blockchain events indexed [C#15]. Issues on "any EVM blockchain" [C#15]; docs list Ethereum, Polygon, Avalanche, Telos, Klaytn [C#13].

**Servicing App modules** [C#12]:
- **Issuance**: deploy ERC-3643 token (name, symbol, supply, decimals); **multi-network deployment with identical addresses via CREATE2**; compliance configuration at any lifecycle stage.
- **Lifecycle management**: **agent appointment** ("Appointed agents"); token actions individually or **batched** — Mint, Burn, Freeze ("Block"), Unfreeze ("Unblock"), Pause/Unpause, Force Transfer, **Recovery** (lost/stolen) [C#12][C#14].
- **Cap table**: real-time holder list with **historical look-back** [C#12]; **Position Reports** (doc link) [C#12].
- **Transaction management**: primary and secondary; **Transfer authorisation** — approve **conditional transfers and DvP** transactions [C#12].
- **Investor qualification**: KYC/AML agent functions; "Onchain identities" [C#12].
- **Distribution**: DINO network (OTC desks, billboards, ATS/MTF, DEX/DeFi) [C#12].

**Token-creation wizard fields** ("Manage Tokens") [C#13]:
- *Token Details (mandatory)*: Name, Symbol, Decimals, **Network**, logo, **Instrument type** (Fund, Equity, Debt, Commodity, Stable coin, Other), **Base currency**.
- *Token Ownership (mandatory)*: owner wallet = "the Master Key to your ERC-3643".
- *Identity Eligibility (optional)*: ONCHAINID requirement, **required claims**, **trusted claim issuers**.
- *Compliance Rules (optional)*: **Jurisdictions** (allow/deny by residence), **Supply** (total supply cap, **investor balance limit**), **Transfer types**: Simple / **Conditional** (approval-based) / **Whitelisting** — Conditional and Whitelisting are **mutually exclusive**; **transfer limits per period** (day, week, custom); **transfer fees**.

**Roles / permission domains** [C#13]: agents get **Write on "Identities"** (authorise/revoke investors) and/or **Write on "Assets"** (mint/burn to several wallets, freeze/unfreeze, force transfer, pause/unpause). Execution: agent **selects investors and signs the blockchain tx with their wallet** [C#14]. On-chain: Owner + Agent roles per ERC-3643 [C#50].

**Custody/maker-checker:** Fireblocks × Tokeny integration — token deployment and mint/burn via **Fireblocks Tokenization Engine**, signing through Fireblocks key management; Tokeny handles compliance config, distribution and investor portal (discovery, subscriptions, portfolio) [C#16]. Approval quorums therefore come from the **custody policy engine**, not the Tokeny UI [I].

**Investor App**: subscription, portfolio management, compliant P2P transfers; KYC/AML automation, document signing, payment & settlement [C#15].

---

## 3. Securitize (Issuer "Control Panel", DS Protocol, TA/BD/ATS)

**Status:** Completed SPAC with Cantor Equity Partners II; **NYSE: SECZ from 2 Jul 2026**; >US$4B tokenised RWA; licences: broker-dealer, **SEC transfer agent**, **ATS**, exempt reporting adviser, **EU DLT Pilot** (trading & settlement system); partners incl. BlackRock, Apollo, BNY, Hamilton Lane, KKR, VanEck, NYSE, Computershare [C#21].

**Control Panel (Platform 3.0, May 2019)** [C#17]:
- **Issuer Management Panel** — switch between multiple issuances / create new instances ("Securitize Ready" partners issue for their clients).
- **Dashboard with raise KPIs** — list of registered prospective investors; **pledged vs funded** amounts; **KYC/AML progress tracking with tools to resolve common issues**; charts of registrations & fundraising over recent days.
- **Fundraise-site configuration** — colours, images, documents, content.
- **Activity log** "for all that is happening around your token issuance".
- **Token-holder registry** — real-time cap table; **RFE (Ready-For-Exchange) API** to reconcile secondary-market holders who never registered directly.

**Issuer Portal (2022 deck)** [C#18]: manage cap table, track ownership, **communicate with shareholders**; configure offering — **available currencies, minimum investment, start date, token price, subscription agreements**; **manage dividend/payout distributions**; lifecycle events incl. **lost/stolen wallet recovery**; branded investor dashboard (imagery, videos, reports); smart-contract config of **shareholder rights incl. voting**, compliance and **jurisdictional restrictions**. **Transfer-agent services** handle payouts and "**death, divorce, lost digital asset wallet keys**" — "**destroy the tokens and reissue** the corresponding tokens to a new wallet". Secondary: activate trading on **Securitize Markets** (ATS). Client Success Manager-assisted onboarding.

**DS Protocol (on-chain)** [C#19][C#20]:
- Services: **Registry Service** (investor data: accreditation, qualification, KYC/AML, confirmed wallets — PII as hashes), **Compliance Service** (Regulated / Whitelisted / Global-Whitelisted variants), **Trust Service** (role registry), plus Wallet Manager, Lock Manager, BulkOperator (batch) and **rebasing** (v4, for accrual/corporate actions); upgradeable ERC-1967 proxies [C#19].
- **Roles:** **Master** (owner), **Issuer** (mint/burn/issuances), **Exchange** (registers investors after KYC), **Transfer Agent** (v4: compliance config, freeze/unfreeze) [C#19].
- Operations: issue, burn, **seize**, **lock (time-based or indefinite)**, **pause trading** [C#19].
- **Compliance Service parameters**: max accredited investors in US; **max retail investors per EU country**; investor-category limits; **country blocks** (issuance & secondary); block offshore→US-person transfers (Reg S); **automatic lock-ups per investor type**; **vesting schedules**; affiliate trading restrictions; **issuer wallets with seize privileges**; `preTransferCheck()` returns standard error codes [C#20].
- Securitize iD (reusable investor identity) — not fetched this session [I].

---

## 4. Brickken — Issuer Studio (self-serve SaaS)

[C#22] unless noted:
- **Asset digitisation**: token creation for equity, debt, private credit, RWA-backed tokens, funds, stablecoins; legal-structure documentation.
- **Offering management**: build and configure an **Investor Portal**; launch offerings — page screenshots are labelled **"public issuance"** and **"private issuance"**.
- **Investor administration**: KYC/AML automation (**50–1,000 checks/year by tier**); **"Investor operations (Confiscation / edits)"** listed as a priced feature (= forced transfer/burn + record edits); investor analytics & holder data.
- **Post-issuance**: **dividend payouts**, **shareholder voting**, reporting, lifecycle admin; earnings distribution; fiat and crypto payments.
- **Infrastructure**: **Sandbox (test) environment**; multichain 1–3+ chains by tier; API implied.
- **Standards**: announced ERC-7943 integration for institutional upgrades [C#11].

---

## 5. Blocksquare — RE-native protocol + white-label marketplaces (+ Oceanpoint)

**Model:** independent **marketplace operators** (LatAm, Middle East, SE Asia) run branded marketplaces on Blocksquare's compliance-ready framework; operators source properties, onboard owners, run community, staking and token sales; **>US$200M tokenised RE (Jul 2025)**, second US$100M "driven almost entirely by independent marketplace operators" [C#23]. **Oceanpoint** = DeFi/liquidity layer (sBST governance, Marketplace Pools where operators get up to **100% infra-cost discount for long-term BST staking**) [C#23].

**Documented operator admin surface** (GitBook index + pages) [C#24]–[C#28]:
- Setup: **New marketplace**, **Custom domain** [C#24].
- Operating: list current/past/future tokenised properties; **KYC-automated investor onboarding**; investors **add bank accounts for EUR investments**; organise and execute **initial real-estate token offerings**; P2P secondary buy/sell; **"distribute revenue payouts to 1000s of token investors"**; up to **10 wallets per account**; self-custodial wallets [C#25].
- **User management**: two whitelisting modes — **auto-whitelist on KYC pass** or **admin confirmation required after KYC**; whitelisting "at the sole discretion" of the operator [C#26].
- **Primary offerings**: on under-subscription the issuer can **"finalize the offering, mint tokens to token buyers"** or **"revert the offering and return funds to investors"**; funds go to issuer's bank account and/or wallet [C#27].
- **Revenue distribution contract**: `addRevenue()` (per property, with `from`/`to` period and per-user amount arrays; **only system admin or authorised certified partner**), `claimRevenuesForWalletForProperty()`, `claimRevenuesForWalletForMultipleProperties()`, **`pushRevenueToUser()`** (push to many wallets), `pendingRevenue()`, `totalPendingRevenue()`, `getAverageMonthlyPayout()`, `totalPayout()`; events `RevenueAdded`, `RevenueClaimed`; paid in stablecoins (DAI/USDC/USDT examples); **BSPT: 100,000 tokens = 100% of economic rights** [C#28].
- Protocol docs also cover **corporate resolution**, **asset valuation**, **buyback process**, transaction fees, payments & invoicing [C#24].
- **Pricing**: **US$5,990 per tokenisation event** (tokenisation + primary offering) + **US$69/month per tokenised property**; software licence one-off or monthly; BST-staking discounts; excl. VAT [C#29].

---

## 6. Zoniqx — Z360 / TPaaS / zConnect

[C#30] Z360 = "issuer-facing orchestration suite": **Asset structuring & token configuration** (asset parameters, investor eligibility, transfer restrictions); **multi-chain issuance (14+ networks)**; **Investor onboarding & entitlement management** (KYC/KYB, role- and jurisdiction-based access); **Lifecycle events** (distributions, corporate actions, reporting); **Operational dashboards** ("audit-ready reporting"); **zConnect** distribution to licensed partners. Product family: zProtocol, zIdentity, zCompliance, TALM (Tokenized Asset Lifecycle Management). On-chain: ERC-7518 (partitions, lockTokens, payout/batchPayout, forceTransfer, vouchers) [C#31].

## 7. Stobox — Stobox 4 + STV3

Release 2.0 (13 Aug 2025) [C#33]: STV3 = ERC-20 on **Diamond Standard (EIP-2535)**, on **Arbitrum**; **RBAC** for minting, burning, treasury ops; **Validation Management** (trusted addresses, transfer rules, pre-transfer checks); **Treasury operations** (withdrawals, allowances, DID/token metadata); **Monetary controls** (issue, redeem, transfer from treasury); **Emergency controls** (force transfer, mint/burn, pause/unpause). Accounts: Individual / Professional / Business. Transfers only to wallets verified via **Stobox DID**. STO parameters: price, accepted payment assets, offering window, min/max participation, eligibility rules, investment caps; validation layers = DID check, jurisdiction check, AML screening [C#32]. ERC-7943 coalition member [C#11].

## 8. Vertalo — Digital Transfer Agent & cap table

SEC-registered TA since 2019; "one shared distributed ledger that serves as the securityholder register itself"; four engagement models (primary TA, sub-TA, client-held registration, TA-technology provider); investor onboarding, **allocations, distributions, corporate actions**; chain-agnostic (Ethereum, Tezos, Aptos) — switch chains without migrating records; **on-chain roles Owner / Transfer Controller / Allowance Controller** with "**delegation on every write**" to attribute which (AI) agent acted; **MCP**-based agent integration; ATS settlement via Allowance Controller (e.g., tZERO); "$500M+ on chain", "100+ issuers" [C#34]. Cap-table page: holder register (amount, price, dates, contact, vesting), shares/options/warrants, onboarding, smart-contract settlement updates [C#35].

## 9. InvestaX (Singapore)

MAS **CMS licence (CMS100635)** + **Recognised Market Operator**; issuer services: advisory, tokenisation of existing securities, fundraising to global investors, **KYC/AML performed by InvestaX**, secondary trading via **AMM and liquidity pools**; structures incl. digital SPACs, digital ESOPs, fractionalised NFTs, **fractionalised real estate**, digital funds (VCC); chains Ethereum, Algorand, Tezos, Hedera, STACS [C#36].

## 10. Polymesh (issuer primitives at chain level)

Asset creation (type Equity/Bond/Fund, divisibility, ISIN/CUSIP), agent-performed `issue`/`redeem`, count/%/claim-based transfer restrictions, **checkpoints** → distributions & voting [C#37]; protocol-level compliance with identity + attestations, rules that "automate rule enforcement without requiring manual or semi-manual approval" [C#38]. Token Studio URL tried → 404 (issuer UI not confirmed this session).

## 11. Hedera Asset Tokenization Studio (open-source issuer app)

Web UI for non-technical users + SDK + audited contracts; equity & bond types; ERC-1400 with ERC-3643 (dual since 12 Nov 2025) [C#39][C#40]. **Roles:** Admin, Minter/Issuer, Controller (forced transfer), Pauser, KYC Manager [C#39]. **UI actions:** create/deploy; mint with **supply cap**; redeem; forced transfer; **approval lists / block lists**; pause; **locks** (vesting/escrow); **snapshots**; freeze/unfreeze; external management lists; **document management**; SSI verification; **corporate actions**: dividends, coupons, **voting rights**, **balance adjustments**, **Mass Payout** [C#39]. Custody: Dfns, Fireblocks, AWS KMS; wallets MetaMask, HashPack, Blade [C#39].

## 12. Fireblocks Tokenization Engine (+ Policy Engine)

Pre-built and partner smart contracts deployable **from the console**, custom contracts, APIs, 35+ chains, lifecycle "from primary issuance and settlement to secondary trading", **Policy Engine automating approval workflows across the token lifecycle** (Sep 2024) [C#41]. **Governance & Policy Engine**: rules by **amount, destination, asset, time window, approval requirement**; per wallet/user/tx type; **two-party to quorum approvals**, mobile or API approvers, **approval-group thresholds**; **admin quorum** for policy changes; policy docs + **authorisation history**; automated compliance screening of high-risk addresses; **dashboard of all pending approvals**; DApp connection policy; token-allowance policies [C#42]. → This is the clearest public **maker-checker** evidence in the vendor set.

## 13. Taurus-CAPITAL

Template library **CMTAT, ERC-3643, ERC-1400, SPL**; upload custom contracts or interact with third-party contracts; **automatic detection of contract functions**; governance rules **at deployment and at function level**; "multi-jurisdictions: fully configurable issuance workflow" with legal docs and **audit trails**; whitelisting; asset range incl. **real-estate NFTs**; pre-integrated with **Taurus-PROTECT** custody, Taurus-PRIME trading and **TDX** OTF settlement; clients State Street, Deutsche Bank, CACEIS, Swissquote, Temenos [C#43].

## 14. KoreConX (now Kore / KoreInside)

koreconx.com redirects to **kore.inc** (rebrand) [C#44]. Site lists solutions for broker-dealers, platform operators, **real estate**, transfer agents; RegCF/RegD/RegA+ focus; module detail not public on homepage [C#44]. Historical module names (KoreTransfer Agent, KoreCaptable, KorePortal) not re-confirmed [I].

## 15. Texture Capital

FINRA broker-dealer + SEC ATS; affiliate **Texture Transfer Services** (SEC TA) — "on-chain cap tables, ownership records, and post-trade processes"; Reg A/D/CF raises tokenised on-chain; **client-branded secondary marketplaces** (e.g., Detroit City FC) [C#45].

## 16. Ctrl Alt (UAE/UK)

VARA licence (Broker-Dealer + Issuer Services, **24 May 2025**, VL/25/05/002); FCA, CBI, GFSC footprint; "$1.8bn+ assets tokenised"; structuring, issuance, distribution, custody, **digital registries**; asset classes incl. **real estate**; clients SEI, Ripple, DMCC; references a **Dubai Land Department** pilot [C#46]. (DLD/Prypco detail to be confirmed by MENA agent [I].)

## 17. Off-chain comparators — RE investor-management SaaS

| Capability | Juniper Square [C#47] | InvestNext [C#48] | Agora [C#49] |
|---|---|---|---|
| Investor CRM | ✔ | ✔ | ✔ |
| Data/deal rooms, DDQ | ✔ data rooms, DDQ | ✔ deal rooms | ✔ fundraising pipelines |
| Investor onboarding / subscriptions / e-sign | ✔ onboarding | ✔ subscriptions | ✔ digital subscriptions / e-sign |
| KYC/AML, accreditation | ✔ AML/KYC | ✔ accreditation letters, KYC/AML | ✔ KYC/AML/accreditation |
| Capital calls | — (not listed) | ✔ | ✔ |
| Distributions & payments | (fund admin) | ✔ ACH, **point-and-click waterfall builder** | ✔ ACH + cross-border, **waterfall automation** |
| Cap table | — | ✔ | — |
| Reporting & tax | ✔ investor reporting | ✔ **K-1s & Global Tax Center** | ✔ **K-1 delivery**, bookkeeping |
| Fund administration | ✔ (+ "Fay" admin-oversight agent) | ✔ | ✔ |
| AI / API | **JunieAI**, **Headless GPX** (Claude, Copilot, ChatGPT, MCP) with "same permissions, same audit trail" | integrations | **Cortex** AI work surface + **MCP API** |
→ These define the **off-chain baseline** a tokenised RE admin must match: waterfalls, capital calls, K-1/tax centre, ACH distributions, white-label investor portal, CRM, data rooms, AI/MCP access with preserved permissions [I].

---

## 18. Cross-vendor patterns (inputs to admin spec)

1. **Two-tier admin**: *platform-owner* (tenant/marketplace/projects, branding, providers, fees) vs *issuer/SPV* (token, investors, offering, distributions) — DigiShares (corporate dashboard + per-STO dashboards), Securitize (Issuer Management Panel), Blocksquare (operator vs certified partner) [C#4][C#17][C#25][C#28].
2. **Role separation mirrors on-chain roles**: Tokeny Identities vs Assets write domains; DS Protocol Master/Issuer/Exchange/Transfer Agent; ATS Admin/Minter/Controller/Pauser/KYC Manager; Vertalo Owner/Transfer Controller/Allowance Controller [C#13][C#19][C#39][C#34].
3. **Maker-checker is mostly delegated to custody** (Fireblocks policy engine, Taurus function-level governance) rather than built in the issuer UI [C#42][C#43][I].
4. **Wallet recovery is a first-class action** everywhere (Tokeny Recovery, Securitize "destroy and reissue", ERC-3643 `recoveryAddress`) [C#14][C#18][C#50].
5. **Distributions**: push (Blocksquare `pushRevenueToUser`, ATS Mass Payout) and pull/claim (Blocksquare claims) both exist; stablecoin rails dominate on-chain; ACH on off-chain comparators [C#28][C#39][C#48].
6. **Offering failure handling** is explicit: finalise-and-mint vs revert-and-refund (Blocksquare) [C#27].
7. **Secondary controls** are whitelist-driven: DigiShares exchange whitelist + SwapContract token whitelists/fees/expiry; Tokeny conditional transfers/DvP approvals [C#4][C#9][C#12].
8. **Agentic/MCP admin access** emerged 2025–26 (Vertalo, Juniper Square, Agora) — with per-action attribution ("delegation on every write") [C#34][C#47][C#49].

---

## Sources

| # | URL | Title | Accessed |
|---|---|---|---|
| 1 | https://digishares.io/product-overview/ | Product Overview – DigiShares | 23 Sep 2026 |
| 2 | https://digishares.io/technology-overview/ | Technology Overview – DigiShares | 23 Sep 2026 |
| 3 | https://digishares.io/platformupgrade/ | DigiShares Announces White-Label Multi-Project Platform for Security Tokens | 23 Sep 2026 |
| 4 | https://digishares.io/jan2020update/ | DigiShares Announce a new Version of Software (27 Jan 2020) | 23 Sep 2026 |
| 5 | https://digishares.io/products/advanced | DigiShares Advanced – Custom Tokenization & API Access | 23 Sep 2026 |
| 6 | https://digishares.io/real-estate-tokenization-best-practices/ | Tokenization of Real Estate – DigiShares | 23 Sep 2026 |
| 7 | https://github.com/DigiShares | DigiShares GitHub organisation | 23 Sep 2026 |
| 8 | https://github.com/DigiShares/ERC1404-v1.3 | DigiShares/ERC1404-v1.3 | 23 Sep 2026 |
| 9 | https://github.com/DigiShares/SwapContract | DigiShares/SwapContract | 23 Sep 2026 |
| 10 | https://app.rwa.xyz/real-estate | RWA.xyz – Tokenized Real Estate | 23 Sep 2026 |
| 11 | https://en.cryptonomist.ch/2026/05/26/erc-7943-the-new-universal-standard-for-the-tokenization-of-real-world-assets-on-ethereum/ | ERC-7943… (The Cryptonomist, 26 May 2026) | 23 Sep 2026 |
| 12 | https://docs.tokeny.com/docs/servicing-app | Servicing App – Tokeny Docs | 23 Sep 2026 |
| 13 | https://docs.tokeny.com/docs/tokens-management | Manage Tokens – Tokeny Docs | 23 Sep 2026 |
| 14 | https://docs.tokeny.com/docs/token-actions | Token Actions – Tokeny Docs | 23 Sep 2026 |
| 15 | https://tokeny.com/t-rex-platform/ | T-REX Platform – Tokeny | 23 Sep 2026 |
| 16 | https://www.fireblocks.com/blog/enhancing-fund-tokenization-operations-and-management-with-fireblocks-x-tokeny | Enhancing Fund Tokenization… Fireblocks x Tokeny | 23 Sep 2026 |
| 17 | https://medium.com/securitize/securitize-platform-3-0-release-enter-the-control-panel-a201370a48de | Securitize Platform 3.0 Release — Enter the Control Panel (13 May 2019) | 23 Sep 2026 |
| 18 | https://s3.us-east-2.amazonaws.com/securitizemarkets.io/caseStudies/How-to-Tokenize+Your+Asset+[Miami+Context]+.pdf | How to Tokenize Assets with Securitize (Jan 2022 deck) | 23 Sep 2026 |
| 19 | https://github.com/securitize-io/dstoken | securitize-io/dstoken – DS Protocol implementation | 23 Sep 2026 |
| 20 | https://medium.com/securitize/ds-protocol-the-compliance-service-b6fe472d625d | DS Protocol — The Compliance Service (3 Jun 2019) | 23 Sep 2026 |
| 21 | https://www.tipranks.com/news/private-companies/securitize-to-list-on-nyse-after-completing-cantor-spac-merger | Securitize to List on NYSE After Completing Cantor SPAC Merger | 23 Sep 2026 |
| 22 | https://www.brickken.com/webapp-saas-platform | Tokenization SaaS Platform (Issuer Studio) – Brickken | 23 Sep 2026 |
| 23 | https://blog.blocksquare.io/article/rwa-tokenization-explodes-in-2025-blocksquare-surpasses-200-m-in-tokenized-real-estate-fueled-by-marketplace-operators/ | Blocksquare surpasses $200M… (Jul 2025) | 23 Sep 2026 |
| 24 | https://docs.blocksquare.io/llms.txt | Blocksquare Docs index | 23 Sep 2026 |
| 25 | https://docs.blocksquare.io/certified-partners/operating-your-marketplace.md | Operating your marketplace – Blocksquare Docs | 23 Sep 2026 |
| 26 | https://docs.blocksquare.io/faq/marketplace-operators/user-management.md | User management – Blocksquare Docs | 23 Sep 2026 |
| 27 | https://docs.blocksquare.io/faq/marketplace-operators/primary-offerings.md | Primary offerings – Blocksquare Docs | 23 Sep 2026 |
| 28 | https://docs.blocksquare.io/for-developers/revenue-distribution-contract.md | Revenue Distribution Contract – Blocksquare Docs | 23 Sep 2026 |
| 29 | https://docs.blocksquare.io/certified-partners/pricing.md | Pricing – Blocksquare Docs | 23 Sep 2026 |
| 30 | https://www.zoniqx.com/z360 | Z360 by Zoniqx | 23 Sep 2026 |
| 31 | https://eips.ethereum.org/EIPS/eip-7518 | ERC-7518 DyCIST | 23 Sep 2026 |
| 32 | https://docs.stobox.io/stobox4/security-token-offering-sto-configuration-and-launch | STO Configuration & Launch – Stobox 4 Docs | 23 Sep 2026 |
| 33 | https://www.stobox.io/blog/stobox-4-release-2-0 | Stobox 4 Release 2.0 (13 Aug 2025) | 23 Sep 2026 |
| 34 | https://www.vertalo.com/ | Vertalo — Agentic Tokenization & Transfer Agency | 23 Sep 2026 |
| 35 | https://beta.vertalo.com/platform/cap-table-management | Vertalo Cap Table Management | 23 Sep 2026 |
| 36 | https://investax.io/issuers | Asset Token Issuers – InvestaX | 23 Sep 2026 |
| 37 | https://developers.polymesh.network/core/assets/fungible/ | Fungible Assets – Polymesh Docs | 23 Sep 2026 |
| 38 | https://polymesh.network/compliance | Compliance – Polymesh | 23 Sep 2026 |
| 39 | https://docs.hedera.com/hedera/open-source-solutions/asset-tokenization-studio-ats | Asset Tokenization Studio – Hedera Docs | 23 Sep 2026 |
| 40 | https://hedera.com/blog/hedera-integrates-erc-3643-token-standard-into-asset-tokenization-studio/ | Hedera integrates ERC-3643 into ATS (12 Nov 2025) | 23 Sep 2026 |
| 41 | https://www.fireblocks.com/blog/a-peek-inside-fireblocks-tokenization-engine-the-backbone-of-your-tokenization-solution | A Peek Inside Fireblocks Tokenization Engine (4 Sep 2024) | 23 Sep 2026 |
| 42 | https://www.fireblocks.com/platforms/governance-and-policies | Governance and Policy Engine – Fireblocks | 23 Sep 2026 |
| 43 | https://www.taurushq.com/capital/ | Taurus-CAPITAL | 23 Sep 2026 |
| 44 | https://kore.inc/ (redirect from koreconx.com) | Kore / KoreInside | 23 Sep 2026 |
| 45 | https://texture.capital/ | Texture Capital | 23 Sep 2026 |
| 46 | https://www.ctrl-alt.co/ | Ctrl Alt | 23 Sep 2026 |
| 47 | https://www.junipersquare.com/ | Juniper Square | 23 Sep 2026 |
| 48 | https://www.investnext.com/ | InvestNext | 23 Sep 2026 |
| 49 | https://agorareal.com/ | Agora | 23 Sep 2026 |
| 50 | https://eips.ethereum.org/EIPS/eip-3643 | ERC-3643 T-REX | 23 Sep 2026 |

**Blocked / dead ends:** YouTube "DigiShares Core Platform Demo" (HTTP 429); docs.digishares.io (DNS failure); polymesh.network/token-studio (404); Blocksquare distributing-revenues & secondary-market pages (no body content; GitBook `?ask=` URL rejected by proxy for length); WebSearch budget exhausted after 23 searches in this agent (session-wide 200 cap shared with parallel agents) — remaining vendors (Taurus, KoreConX, Texture, Ctrl Alt, comparators) fetched directly by known URL.
