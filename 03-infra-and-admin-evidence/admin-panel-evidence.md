# Admin / Issuer Panel Evidence Catalogue

**Purpose:** Consolidate every admin/issuer-panel feature observed in publicly documented vendor dashboards, protocols and comparators, grouped into 17 modules, as the evidence base for RealX's platform-owner admin-panel spec (`05-admin-panels/`).
**Date:** 23 Sep 2026. **Labels:** [C#n] confirmed (Sources at end), [I] inferred, [S] speculative.
**Columns:** *Vendors observed* (source #) · *Layer* = On-chain (contract function / chain primitive), Off-chain (app DB / workflow), or Hybrid · *Maker-checker* = **Y** (dual control / quorum / separate approver documented), **P** (approval step exists but not dual control, e.g., admin confirms a user-initiated request), **N** (not observed), **I** (recommended/inferred only).

---

## Headline findings

1. **The richest public admin evidence comes from Tokeny (ERC-3643 Servicing App), Hedera ATS, Securitize (Control Panel + DS Protocol), DigiShares (release notes + GitHub) and Blocksquare (operator docs + revenue contract).** Together they cover ~80% of the modules below [C#1–C#25].
2. **Maker-checker is almost never built into the issuer UI itself.** Explicit dual control was observed only in custody/governance layers — **Fireblocks Policy Engine** (quorum approvals, admin quorum for policy changes, pending-approvals dashboard) and **Taurus-CAPITAL** (governance rules at deployment and per smart-contract function) [C#26][C#28]. Tokeny agents sign each action with their own wallet; the "checker" comes from custody policy [C#9][I].
3. **Single-approver "P" approvals are common**: Blocksquare KYC-then-admin-confirm whitelisting; Tokeny conditional-transfer/DvP approvals; DigiShares exchange invite/whitelist; Blocksquare finalise-vs-revert offering [C#17][C#7][C#3][C#18].
4. **Wallet recovery and forced transfer are universal** (ERC-3643 `recoveryAddress`/`forcedTransfer`, Tokeny Recovery, Securitize "destroy and reissue", DS `seize`, ATS Controller, Brickken "Confiscation", Stobox emergency controls) — and must carry legal-evidence upload + dual approval in RealX [C#9][C#12][C#13][C#15][C#22][C#25][C#32][I].
5. **Real-estate-specific modules (rent roll, NOI, waterfalls, capital calls, K-1/tax centre) appear only in off-chain comparators** (InvestNext, Agora, Juniper Square), while token vendors stop at "dividends/distributions". RealX must merge both [C#29–C#31][I].
6. **Two-tier tenancy** (platform owner → issuers/SPVs/projects) is explicit in DigiShares (corporate dashboard + per-STO dashboards/URLs/DBs), Securitize (Issuer Management Panel) and Blocksquare (marketplace operator vs certified partner) [C#3][C#11][C#19].
7. **2025–26 shift: agentic admin.** Vertalo (MCP, "delegation on every write"), Juniper Square (Headless GPX: "same permissions, same audit trail"), Agora (MCP API) expose admin actions to AI agents with attribution — RBAC and audit log must treat agents as principals [C#24][C#29][C#31].

---

## Module 1 — Issuer / SPV management (tenancy)

| Feature | Vendors observed | Layer | Maker-checker |
|---|---|---|---|
| Top-level corporate dashboard over many offerings; per-project dashboard, URL and investor DB | DigiShares [C#3][C#2]; Securitize "Issuer Management Panel" (switch/create instances) [C#11] | Off | N |
| Multi-project pricing (projects per plan; extra project fee) | DigiShares (2 incl.; €60–200/extra) [C#1] | Off | — |
| SPV creation + bank account as onboarding step | DigiShares RE workflow [C#4] | Off | N |
| Marketplace operator vs certified partner (tokeniser) roles | Blocksquare [C#16][C#19] | Hybrid | N |
| Corporate resolution recorded for tokenisation | Blocksquare "corporate resolution" doc [C#20] | Off | I |
| Issuer on-chain identity (ONCHAINID of token/issuer) | ERC-3643 `setOnchainID` [C#32] | On | I |
| Asset type & identifiers (Equity/Bond/Fund; ISIN/CUSIP) | Polymesh [C#35]; Tokeny instrument type [C#8] | On/Off | N |

## Module 2 — Asset / property management

| Feature | Vendors observed | Layer | Maker-checker |
|---|---|---|---|
| Property listing (current/past/future tokenised properties) | Blocksquare [C#16] | Off | N |
| Property presentation (photos, financials, videos, docs) | DigiShares [C#4][C#1] | Off | N |
| Asset valuation process | Blocksquare "asset valuation" doc [C#20] | Off (→ oracle [I]) | I |
| Buyback process | Blocksquare [C#20] | Hybrid | I |
| Economic-rights model (100,000 BSPT = 100%) | Blocksquare [C#19] | On | — |
| Partitions per floor/share class | ERC-7518 [C#33]; DigiShares share classes [C#3] | On/Off | N |
| Rent roll / NOI / property accounting | **Not observed in token vendors**; comparators: Agora bookkeeping, fund admin [C#31] | Off | I |

## Module 3 — Offering / deal management

| Feature | Vendors observed | Layer | Maker-checker |
|---|---|---|---|
| Offering config: price, currencies/payment assets, min/max investment, start date/window, subscription agreement | Securitize [C#12]; Stobox [C#23] | Off (+contract) | N |
| Investment caps / eligibility rules per offering | Stobox [C#23] | Hybrid | N |
| Public vs private issuance | Brickken [C#15] | Off | N |
| Raise KPIs: prospects, **pledged vs funded**, daily registrations | Securitize [C#11] | Off | — |
| Under-subscription outcome: **finalise & mint** vs **revert & refund** | Blocksquare [C#18] | Hybrid | P |
| Funds routing to issuer bank account and/or wallet | Blocksquare [C#18] | Off | I |
| Fundraise site CMS (colours, images, docs) by jurisdiction × domicile | Securitize [C#11]; DigiShares [C#2] | Off | N |
| Async subscription/redemption queue (request → claimable) | ERC-7540 [C#34] | On | I |
| Capital calls | InvestNext, Agora [C#30][C#31] | Off | N |
| Deal rooms / data rooms / DDQ | InvestNext, Juniper Square [C#30][C#29] | Off | N |

## Module 4 — Investor CRM & onboarding review

| Feature | Vendors observed | Layer | Maker-checker |
|---|---|---|---|
| KYC/AML progress tracking with tools to fix common issues | Securitize [C#11] | Off | N |
| KYC auto-whitelist **or** admin confirmation after KYC | Blocksquare [C#17] | Hybrid | P |
| Authorise / revoke investor (Identities write permission) | Tokeny [C#8] | On | N (agent-signed) |
| Register investor after KYC ("Exchange" role) | DS Protocol [C#13] | On | N |
| Investor data: accreditation, qualification, KYC/AML, confirmed wallets (hashed PII) | DS Registry Service [C#13] | On (hashes) | — |
| Required claims + trusted claim issuers | Tokeny / ERC-3643 [C#8][C#32] | On | I |
| DID-verified wallets only | Stobox [C#22] | On | N |
| KYC/KYB with role- and jurisdiction-based entitlements | Zoniqx Z360 [C#21] | Hybrid | N |
| Investor CRM, relationship history | InvestNext, Agora, Juniper Square [C#30][C#31][C#29] | Off | N |
| Accreditation verification & letters | InvestNext [C#30] | Off | N |
| Tokenised + non-tokenised investors on same register | DigiShares [C#2] | Hybrid | — |
| Up to N wallets per investor (10) | Blocksquare [C#16] | Hybrid | — |
| Travel-Rule messaging | Sumsub plans [C#39] | Off | N |

## Module 5 — Compliance rules engine

| Feature | Vendors observed | Layer | Maker-checker |
|---|---|---|---|
| Country allow/deny by investor residence | Tokeny [C#8]; DS Protocol [C#14] | On | N |
| Max investors (US accredited cap; retail per EU country) | DS Protocol [C#14]; Polymesh count rules [C#35] | On | N |
| Max % ownership / investor balance limit / total supply cap | Tokeny [C#8]; Polymesh [C#35]; ATS supply cap [C#25] | On | N |
| Transfer type: Simple / **Conditional (approval)** / Whitelisting (mutually exclusive) | Tokeny [C#8] | On | P |
| Transfer limits per period; transfer fees | Tokeny [C#8] | On | N |
| Lock-ups by investor type; vesting schedules; affiliate restrictions; Reg S offshore→US block | DS Protocol [C#14] | On | N |
| Buy/sell restriction dates per address; global trading switch | DigiShares ERC-1404 [C#5] | On | N |
| Time-based locks (holding periods) | ERC-7518 [C#33]; ATS locks [C#25] | On | N |
| Pre-transfer checks / restriction codes | DS `preTransferCheck` [C#14]; ERC-1404 [C#5]; ERC-7943 `canTransfer` [C#41] | On | — |
| Modular compliance modules | ERC-3643 [C#32]; ATS "Customize" layer [C#25] | On | I |
| Rules at deploy and per-function governance | Taurus-CAPITAL [C#28] | Hybrid | **Y** |

## Module 6 — Token lifecycle

| Feature | Vendors observed | Layer | Maker-checker |
|---|---|---|---|
| Deploy token (wizard: name, symbol, decimals, network, logo, instrument type, base currency, owner wallet) | Tokeny [C#8]; ATS [C#25]; Fireblocks console [C#27]; Taurus template library (CMTAT/3643/1400/SPL) [C#28] | On | Y via custody [C#26]/I |
| Same-address multi-chain deploy (CREATE2) | Tokeny [C#7] | On | — |
| Mint / burn (individual or batch) | Tokeny [C#9]; DS Issuer role [C#13]; ATS Minter [C#25]; Stobox [C#22]; DigiShares owner-only [C#5] | On | Y via Fireblocks policy [C#26]; else N |
| Freeze / unfreeze wallet; partial freeze ("Block/Unblock") | Tokeny [C#9]; ERC-3643 [C#32]; DS TA role [C#13]; ATS [C#25]; DigiShares "token freezing/renewal" [C#2] | On | N/I |
| Pause / unpause | Tokeny [C#9]; DS [C#13]; ATS Pauser [C#25]; Stobox emergency [C#22] | On | I |
| Forced transfer / seize / confiscation | Tokeny [C#9]; DS `seize` [C#13]; ATS Controller [C#25]; Brickken "Confiscation" [C#15]; Stobox [C#22] | On | I |
| Lost-wallet recovery ("destroy and reissue") | Tokeny Recovery [C#9]; Securitize TA [C#12]; ERC-3643 `recoveryAddress` [C#32] | On | I |
| Agent appointment / role assignment | Tokeny "Appointed agents" [C#7]; ATS RBAC [C#25]; Stobox RBAC [C#22]; Vertalo Owner/Transfer Controller/Allowance Controller [C#24] | On | Y (admin quorum, Fireblocks) [C#26] |
| Contract-function auto-detection & interaction | Taurus [C#28] | On | Y |
| Rebasing / balance adjustments | DS v4 [C#13]; ATS balance adjustments [C#25] | On | I |
| Sandbox / test environment | Brickken [C#15] | Off | — |

## Module 7 — Cap table / register of holders

| Feature | Vendors observed | Layer | Maker-checker |
|---|---|---|---|
| Real-time cap table with **historical look-back** | Tokeny [C#7]; Securitize [C#11] | Hybrid (indexed) | — |
| Position reports | Tokeny [C#7] | Hybrid | — |
| Reconcile secondary-market holders not registered directly (RFE API) | Securitize [C#11] | Hybrid | — |
| Ledger is the legal register; switch chains without migrating records | Vertalo [C#24] | Hybrid | — |
| Holder register with price, dates, vesting, options/warrants | Vertalo [C#24]; InvestNext cap table [C#30] | Off | — |
| Snapshots / checkpoints | ATS [C#25]; Polymesh [C#35] | On | — |
| Reporting to transfer agents / CSDs | DigiShares [C#4] | Off | I |

## Module 8 — Distributions & corporate actions

| Feature | Vendors observed | Layer | Maker-checker |
|---|---|---|---|
| Dividend distribution (frequency by tier: quarterly/monthly/unlimited) | DigiShares [C#1] | Hybrid | N |
| Revenue per property per period (`addRevenue` admin/partner-only) + **push** to many wallets or holder **claim** | Blocksquare [C#19] | On (stablecoin) | N (restricted caller) |
| Dividends, coupons, voting rights, **Mass Payout** | Hedera ATS [C#25] | On | I |
| `payout` / `batchPayout` | ERC-7518 [C#33] | On | I |
| Dividend payouts, earnings distribution (fiat & crypto) | Brickken [C#15]; Securitize [C#12] | Hybrid | N |
| Distributions & corporate actions (automated) | Zoniqx [C#21]; Vertalo [C#24]; Tokeny [C#10] | Hybrid | N |
| **Waterfall builder**, ACH distributions | InvestNext [C#30]; Agora [C#31] | Off | N |
| Withholding tax / tax forms | InvestNext K-1 & Global Tax Center [C#30]; Agora K-1 delivery [C#31] | Off | N |
| Checkpoint-based distributions | Polymesh [C#35] | On | — |

## Module 9 — Secondary market controls

| Feature | Vendors observed | Layer | Maker-checker |
|---|---|---|---|
| Internal exchange; **invite & whitelist investors into exchange** | DigiShares [C#3] | Hybrid | P |
| P2P swap contract: whitelist security tokens & stablecoins, set **fees, fee recipients, expiry** | DigiShares SwapContract [C#6] | On | N |
| Approve conditional transfers & **DvP** | Tokeny [C#7] | On | P |
| Distribution network (OTC, ATS/MTF, DEX) | Tokeny DINO [C#7]; Zoniqx zConnect [C#21] | Hybrid | — |
| Activate trading on affiliated ATS | Securitize Markets [C#12]; Texture branded marketplace [C#37] | Off | — |
| AMM / liquidity pools | InvestaX [C#36]; Oceanpoint (see vendor file) | On | — |
| Global trading halt | DigiShares `isTradingAllowed` [C#5]; ERC-3643 pause [C#32] | On | I |
| Order-book circuit breakers / price bands | **Not observed** | — | I |

## Module 10 — Treasury, payments & reconciliation

| Feature | Vendors observed | Layer | Maker-checker |
|---|---|---|---|
| On-chain or traditional payments (card, bank, crypto) | DigiShares [C#1][C#4] | Hybrid | N |
| Investors add bank accounts (EUR) | Blocksquare [C#16] | Off | N |
| Treasury ops: withdrawals, allowances, issue/redeem from treasury | Stobox [C#22] | On | I |
| Payments & invoicing (operator ↔ Blocksquare) | Blocksquare [C#20] | Off | — |
| Transaction policies (amount, destination, asset, time window) + **pending approvals dashboard** | Fireblocks [C#26] | Off→On | **Y** |
| ACH / cross-border payouts; "Transact Accounts" | InvestNext, Agora [C#30][C#31] | Off | N |
| Escrow / refund on failed raise | Blocksquare revert-and-refund [C#18] | Hybrid | P |

## Module 11 — Documents & e-signature

| Feature | Vendors observed | Layer | Maker-checker |
|---|---|---|---|
| Contract templates with **variable fields**; distribute to investor subset; investor review/suggestions | DigiShares [C#3] | Off | N |
| DocuSign / Dropbox Sign integration | DigiShares [C#1] | Off | — |
| Subscription agreements per offering | Securitize [C#12] | Off | — |
| Token-attached documents | ATS document mgmt [C#25]; ERC-1643 (see token-standards) | On | I |
| Document management (templating, search, segmented sharing) | InvestNext [C#30]; Agora [C#31] | Off | N |
| Legal docs + audit trail in issuance workflow | Taurus [C#28] | Off | Y |

## Module 12 — Governance / voting

| Feature | Vendors observed | Layer | Maker-checker |
|---|---|---|---|
| Meetings with agendas; per-item votes; **vote weighting by share rights**; external invitees | DigiShares [C#3] | Off | N |
| Shareholder voting | Brickken [C#15]; Securitize (voting rights in contract config) [C#12] | Hybrid | N |
| Voting rights as corporate action | Hedera ATS [C#25]; Polymesh ballots [C#35] | On | — |

## Module 13 — Reporting & regulatory reports

| Feature | Vendors observed | Layer | Maker-checker |
|---|---|---|---|
| Audit-ready operational dashboards | Zoniqx [C#21] | Off | — |
| Reporting (investor, lifecycle) | Brickken [C#15]; Juniper Square investor reporting [C#29] | Off | — |
| Tax forms (K-1) | InvestNext, Agora [C#30][C#31] | Off | — |
| Policy docs & authorisation history for regulators | Fireblocks [C#26] | Off | Y |
| Reporting to TAs/CSDs | DigiShares [C#4] | Off | — |
| SAR/STR filing workflow | **Not observed in any vendor** | Off | I |

## Module 14 — Communications / notifications

| Feature | Vendors observed | Layer | Maker-checker |
|---|---|---|---|
| Mass email to investors; admin inbox; company news | DigiShares [C#3][C#2] | Off | N |
| Communicate with shareholders from issuer portal | Securitize [C#12] | Off | N |
| Investor portal updates / email marketing | InvestNext, Agora [C#30][C#31] | Off | N |

## Module 15 — RBAC, maker-checker & audit log

| Feature | Vendors observed | Layer | Maker-checker |
|---|---|---|---|
| Permission domains (Identities write vs Assets write) | Tokeny [C#8] | On | N |
| Role set Master / Issuer / Exchange / Transfer Agent | DS Protocol [C#13] | On | N |
| Role set Admin / Minter / Controller / Pauser / KYC Manager | Hedera ATS [C#25] | On | N |
| Owner / Agent (ERC-173) | ERC-3643 [C#32] | On | I (agent may be multisig contract) |
| Owner / Transfer Controller / Allowance Controller; **"delegation on every write"** (agent attribution) | Vertalo [C#24] | On | N (attribution) |
| Quorum approvals, approval-group thresholds, **admin quorum** for policy changes | Fireblocks [C#26] | Off/HSM | **Y** |
| Function-level governance rules | Taurus [C#28] | Hybrid | **Y** |
| Activity log of issuance events | Securitize [C#11] | Off | — |
| 2FA for admins | DigiShares [C#3] | Off | — |
| AI agents with "same permissions, same audit trail" | Juniper Square [C#29] | Off | — |

## Module 16 — Integrations, API keys & webhooks

| Feature | Vendors observed | Layer | Maker-checker |
|---|---|---|---|
| API access (1M calls/month) for white-label customisation | DigiShares Advanced [C#1][C#38] | Off | — |
| API v1/v2 endpoints | Tokeny [C#7] | Off | — |
| RFE API for exchange holder sync | Securitize [C#11] | Off | — |
| SDK + open-source contracts | Hedera ATS [C#25] | Hybrid | — |
| Custody integrations (Fireblocks, Dfns, AWS KMS) | Hedera ATS [C#25]; Tokeny×Fireblocks (see white-label file) | Hybrid | Y via custody |
| MCP / agent APIs | Vertalo [C#24]; Juniper Square [C#29]; Agora [C#31] | Off | — |
| Webhooks | **Not explicitly observed** in fetched pages | Off | I |

## Module 17 — Settings, white-label branding & multi-tenant

| Feature | Vendors observed | Layer | Maker-checker |
|---|---|---|---|
| White-label logo, colours, custom URL | DigiShares Standard+ [C#1]; Blocksquare custom domain [C#20] | Off | — |
| Fundraise-site content configuration | Securitize [C#11]; DigiShares CMS [C#2] | Off | — |
| Provider configuration (KYC, payments, custody, e-sign) | DigiShares [C#1][C#4] | Off | I |
| Multi-chain selection per token | Tokeny [C#8]; Zoniqx 14+ [C#21]; Brickken tier-gated [C#15] | On | — |
| User capacity limits by tier (200/2,000/10,000) | DigiShares [C#1] | Off | — |

---

## Compact catalogue (module → key pages → key actions → typical roles)

| Module | Key pages | Key actions | Typical roles (observed/inferred) |
|---|---|---|---|
| 1 Issuer/SPV | Tenants/Issuers list; Issuer profile; SPV docs | Create issuer/SPV, link bank acct, set on-chain identity | Platform Super-Admin; Issuer Admin |
| 2 Asset/property | Properties; Property detail (media, financials, valuation, docs) | List property, upload valuation, set share classes/partitions, initiate buyback | Asset Manager; Valuation Officer [I] |
| 3 Offering | Offerings; Offering builder; Raise dashboard; Subscriptions | Set price/min/max/window/currencies/caps; approve subscriptions; finalise-and-mint or revert-and-refund | Offering Manager; Compliance Officer (checker) |
| 4 Investor CRM | Investors; KYC/KYB queue; Accreditation queue; Wallets | Approve/reject KYC, whitelist wallet, revoke, request re-verification | KYC Analyst (maker); Compliance Officer (checker); "Exchange"/Identities agent |
| 5 Compliance rules | Rule sets per token; Jurisdictions; Limits; Lock-ups | Add/modify country bans, holder caps, % limits, lock-ups, transfer mode | Compliance Officer; Transfer Agent role |
| 6 Token lifecycle | Token Studio; Token detail; Actions console; Agents | Deploy, mint, burn, freeze/partial freeze, pause, forced transfer, recovery, appoint agents | Token Agent/Minter/Controller/Pauser; Owner (multisig) |
| 7 Cap table | Register; Position reports; Snapshots | Export, point-in-time look-back, reconcile venue holders | Registrar/Transfer Agent; Auditor (read) |
| 8 Distributions | Distributions; Waterfall; Snapshots; Tax | Compute NOI → waterfall → snapshot → push/claim payout; withholding; K-1/statements | Finance Manager (maker); CFO (checker) |
| 9 Secondary | Market settings; Whitelisted venues; Swaps/orders; Halts | Enable/disable trading, whitelist investors/tokens, set fees/expiry, approve DvP, halt | Market Operations; Compliance |
| 10 Treasury | Accounts & wallets; Payments in/out; Escrow; Reconciliation; Approvals | Reconcile fiat/stablecoin, release escrow, refunds, approve outbound tx | Treasury Ops (maker); Treasurer (checker) |
| 11 Documents | Templates; Envelopes; Token docs | Build templates w/ variables, send to subset, attach doc hash to token | Legal/Ops |
| 12 Governance | Meetings; Ballots | Create meeting/agenda, weight votes, publish results | Company Secretary |
| 13 Reporting | Reports; Regulatory; Audit exports | Investor statements, regulator reports, SAR/STR case export | Compliance/MLRO; Finance |
| 14 Comms | Announcements; Inbox; Templates | Mass email, targeted notices, reply to investors | IR Manager |
| 15 RBAC & audit | Users & roles; Approval policies; Audit log | Assign roles, set quorum thresholds, review audit trail, agent (AI) principals | Super-Admin; Security Officer |
| 16 Integrations | API keys; Webhooks; Providers; MCP | Rotate keys, configure providers, scope agents | Tech Admin |
| 17 Settings | Branding; Domains; Chains; Plans/limits | Set logo/colours/domain, choose networks, tier limits | Platform Super-Admin |

---

## Gaps (not publicly observed — must be designed from regulation/first principles)
- SAR/STR case management & regulator filing workflow [I].
- Withholding-tax engine for on-chain distributions (only off-chain K-1 tools observed) [I].
- Rent-roll / PM-system sync (AppFolio/Yardi) in any token vendor [I].
- Order-book circuit breakers / price bands on internal markets [I].
- Explicit maker-checker inside issuer UIs (only via custody policy engines) [C#26][C#28][I].
- Webhook catalogues (not seen on fetched pages) [I].

---

## Sources

| # | URL | Title | Accessed |
|---|---|---|---|
| 1 | https://digishares.io/product-overview/ | DigiShares Product Overview | 23 Sep 2026 |
| 2 | https://digishares.io/platformupgrade/ | DigiShares White-Label Multi-Project Platform | 23 Sep 2026 |
| 3 | https://digishares.io/jan2020update/ | DigiShares new software version (27 Jan 2020) | 23 Sep 2026 |
| 4 | https://digishares.io/real-estate-tokenization-best-practices/ | Tokenization of Real Estate – DigiShares | 23 Sep 2026 |
| 5 | https://github.com/DigiShares/ERC1404-v1.3 | DigiShares ERC1404-v1.3 | 23 Sep 2026 |
| 6 | https://github.com/DigiShares/SwapContract | DigiShares SwapContract | 23 Sep 2026 |
| 7 | https://docs.tokeny.com/docs/servicing-app | Tokeny Servicing App | 23 Sep 2026 |
| 8 | https://docs.tokeny.com/docs/tokens-management | Tokeny Manage Tokens | 23 Sep 2026 |
| 9 | https://docs.tokeny.com/docs/token-actions | Tokeny Token Actions | 23 Sep 2026 |
| 10 | https://tokeny.com/t-rex-platform/ | T-REX Platform | 23 Sep 2026 |
| 11 | https://medium.com/securitize/securitize-platform-3-0-release-enter-the-control-panel-a201370a48de | Securitize Platform 3.0 — Control Panel (13 May 2019) | 23 Sep 2026 |
| 12 | https://s3.us-east-2.amazonaws.com/securitizemarkets.io/caseStudies/How-to-Tokenize+Your+Asset+[Miami+Context]+.pdf | How to Tokenize Assets with Securitize (Jan 2022) | 23 Sep 2026 |
| 13 | https://github.com/securitize-io/dstoken | DS Protocol (dstoken) | 23 Sep 2026 |
| 14 | https://medium.com/securitize/ds-protocol-the-compliance-service-b6fe472d625d | DS Protocol — Compliance Service (3 Jun 2019) | 23 Sep 2026 |
| 15 | https://www.brickken.com/webapp-saas-platform | Brickken Issuer Studio | 23 Sep 2026 |
| 16 | https://docs.blocksquare.io/certified-partners/operating-your-marketplace.md | Blocksquare – Operating your marketplace | 23 Sep 2026 |
| 17 | https://docs.blocksquare.io/faq/marketplace-operators/user-management.md | Blocksquare – User management | 23 Sep 2026 |
| 18 | https://docs.blocksquare.io/faq/marketplace-operators/primary-offerings.md | Blocksquare – Primary offerings | 23 Sep 2026 |
| 19 | https://docs.blocksquare.io/for-developers/revenue-distribution-contract.md | Blocksquare – Revenue Distribution Contract | 23 Sep 2026 |
| 20 | https://docs.blocksquare.io/llms.txt | Blocksquare docs index (valuation, corporate resolution, buyback, custom domain, payments & invoicing) | 23 Sep 2026 |
| 21 | https://www.zoniqx.com/z360 | Zoniqx Z360 | 23 Sep 2026 |
| 22 | https://www.stobox.io/blog/stobox-4-release-2-0 | Stobox 4 Release 2.0 (13 Aug 2025) | 23 Sep 2026 |
| 23 | https://docs.stobox.io/stobox4/security-token-offering-sto-configuration-and-launch | Stobox 4 STO configuration | 23 Sep 2026 |
| 24 | https://www.vertalo.com/ ; https://beta.vertalo.com/platform/cap-table-management | Vertalo; Vertalo cap table | 23 Sep 2026 |
| 25 | https://docs.hedera.com/hedera/open-source-solutions/asset-tokenization-studio-ats | Hedera Asset Tokenization Studio | 23 Sep 2026 |
| 26 | https://www.fireblocks.com/platforms/governance-and-policies | Fireblocks Governance & Policy Engine | 23 Sep 2026 |
| 27 | https://www.fireblocks.com/blog/a-peek-inside-fireblocks-tokenization-engine-the-backbone-of-your-tokenization-solution | Fireblocks Tokenization Engine (4 Sep 2024) | 23 Sep 2026 |
| 28 | https://www.taurushq.com/capital/ | Taurus-CAPITAL | 23 Sep 2026 |
| 29 | https://www.junipersquare.com/ | Juniper Square | 23 Sep 2026 |
| 30 | https://www.investnext.com/ | InvestNext | 23 Sep 2026 |
| 31 | https://agorareal.com/ | Agora | 23 Sep 2026 |
| 32 | https://eips.ethereum.org/EIPS/eip-3643 | ERC-3643 | 23 Sep 2026 |
| 33 | https://eips.ethereum.org/EIPS/eip-7518 | ERC-7518 | 23 Sep 2026 |
| 34 | https://eips.ethereum.org/EIPS/eip-7540 | ERC-7540 | 23 Sep 2026 |
| 35 | https://developers.polymesh.network/core/assets/fungible/ | Polymesh Fungible Assets | 23 Sep 2026 |
| 36 | https://investax.io/issuers | InvestaX Issuers | 23 Sep 2026 |
| 37 | https://texture.capital/ | Texture Capital | 23 Sep 2026 |
| 38 | https://digishares.io/products/advanced | DigiShares Advanced | 23 Sep 2026 |
| 39 | https://sumsub.com/pricing/ | Sumsub Pricing | 23 Sep 2026 |
| 41 | https://eips.ethereum.org/EIPS/eip-7943 | ERC-7943 uRWA | 23 Sep 2026 |
