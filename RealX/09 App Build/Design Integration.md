---
title: "Design Integration"
created: 2026-09-24
updated: 2026-09-24
status: complete
tags: [realx, app-build, design, integration]
aliases: ["RealX Design Integration", "Design Hand-off"]
---

# Design Integration

**This note explains how the Claude Design prototype `RealX App.dc.html` becomes the RealX app. It covers what the design package contains, the design tokens and components that [[A05 - Design system package from the prototype tokens|A05]] turns into `packages/ui`, and how each of the 26 prototype screen keys maps to the product-map screens. Counted by product-map ID, the prototype draws 38 of the 91 screens in full and 8 in part (PUB-05 was reclassified as partial on 24 Sep 2026: the Sponsor tab lacks most of the product map's PUB-05). The other 45 are undrawn, including the three D04 agent-access screens ACC-05, SPN-16 and ADM-35. Two design-extension nodes, [[A50 - Design extension A- investor and sponsor screens not in the prototype|A50]] and [[A51 - Design extension B- operator console screens not in the prototype|A51]], draw them, the missing parts of the partial screens and the drawn-screen supplements (§6) in the prototype's own style, and the founder approves them at HG-REVIEW before any UI node that uses them starts. The design wins on look and interaction. The product map wins on requirements, IDs, states and compliance. This plan is not legal advice.**

**How the build runs.** Under decision D19, Opus 5.5 (high) plans each node and Grok 4.7 (xhigh) builds it. Opus 5.5 (high) does the node QA at every gauntlet loop in a fresh session, and that QA includes design fidelity. Fable 5.1 (high) runs the graph QA when a milestone's nodes have all passed ([[App Gauntlet]]).

**How this plan was checked.** Fable 5.1 (high) wrote the 106-node graph and the specs A00 to A45 and A47 on 23 and 24 Sep 2026, and then reached its weekly limit. Opus 5.5 wrote the other specs and assembled the plan. A separate Opus 5.5 reviewer checked the plan in ultracode as a third party with no stake in it. For the plan only, this replaced the Fable check. When the check run started, every node spec had had one independent check followed by a fix pass. One spec (A05, the design-system node) has passed a re-check. The other 105, including A50, A51 and A87, still await their re-check, because the usage limit stopped the check run. A106, added by the plan fixer on 24 Sep 2026, has had no independent check yet. The build roles changed on 30 Sep 2026 (decision D19, above), and the remaining re-checks are Opus 5.5 (high) per spec and a Fable 5.1 (high) review of the whole plan.

## 1. The design package

The founder handed over the design as a zip file with SHA-256 `1644066E2BD78BB1CD7BB9F9E8869E2A984F3BC50DE31A4BA00F059654309976`. It was unpacked unchanged at `<workspace>\design\`. **No node edits this folder.** [[A00 - Charter and acceptance baseline|A00]] records every file's hash in `docs/charter/design-baseline.json`. A05 records the two source hashes in `design-source.json`, so any change to the design fails `npm run verify`. [[A87 - Design fidelity and accessibility pass|A87]] checks both hashes before and after every fidelity run.

| File | Bytes | SHA-256 (measured 24 Sep 2026) | What it is |
|---|---|---|---|
| `RealX App.dc.html` | 110,981 | `8ef1e73c23a3644aac5935cd41bddfbb1719c546b98e296d16be74b8d4589b03` | The prototype: 921 lines. About 738 lines of template sit inside `<x-dc>`, followed by a script block of about 180 lines holding the mock data and a `Component extends DCLogic` class |
| `support.js` | 69,150 | `8fe7df74405f3c55f49b7249c74ea1397e65d07dea2b1bd3b4a489bec2e28cbe` | The generated dc runtime (1,911 lines) that compiles the template into React elements |
| `uploads/RealX-Product-Map.html` | 1,202,874 | `bc95f8855e31e3a0528155d4dc2e063a296ffcfdb45e2efb3d7b08e17db017aa` | The product map as uploaded to Claude Design |
| `.thumbnail` | 22,184 | `84166f36ecfc51794a057aba91b316d00bfecfa82a2206a62990a7480a299b3d` | Preview image |

**The uploaded product map is out of date.** It is byte-identical to `<workspace>\RealX-Product-Map.html` from 23 Sep 12:53, which predates founder decision D04 and contains no ACC-05, SPN-16 or ADM-35. The requirements source is the vault copy `04 Architecture/RealX-Product-Map.html`, regenerated at 20:21 on the same day with the D04 screens. Its generator data is in `06 Evidence/Tools/product_map/data_*.py`, and the ID list gives 91 screens. Builders never read requirements from the copy in the design package.

## 2. What the prototype is

The prototype is a **Claude Design ("dc") prototype**. It is a single HTML file that the `support.js` runtime turns into a clickable React preview. It is a design reference, not product code. **Engineers do not port the dc runtime.**

| Construct in the dc file | What it means for the build |
|---|---|
| `<x-dc>` template, `{{ path }}` bindings, `<sc-if>`, `<sc-for>` | Read these as the component tree and data bindings. The app is Next.js and React written from scratch using `packages/ui` |
| `style-hover="…"` | Plain CSS `:hover` |
| `DCLogic.state`, `setState`, `renderVals()` | The state becomes a store, derived values become selectors, and click closures become handlers |
| `go(screen)` with no URL | Replaced by a real router with product-map routes ([[A52 - Web app shells, routing and passkey sign-in (public, investor, sponsor)|A52]] for the public, investor and sponsor surfaces; [[A53 - Operator console shell, hardware-key sign-in and approval action pattern|A53]] for the console) |

**Four "view as" roles** are available from a dark switcher bar: visitor and investor share the public shell, the sponsor role uses a light sidebar shell, and RealX admin uses a dark sidebar shell. The prototype has **26 screen keys**: 11 public and investor, 6 sponsor and 9 admin. Its mock data holds 8 properties (Kent Street Terrace, Marion Street, Bay Road Apartment, Ann Street, Hawthorn Grove, Marine Parade Apartment, Gympie Road Retail, Unley Road Cottage), 3 sponsors (Harbour & Co Property, Northline, Yarra Bend), 5 approvals (APR-2285 to APR-2291), 3 token operations (OP-7729 to OP-7731) and the personas Alex Morgan (investor), Mia Chen (sponsor owner) and Jordan Blake (chief compliance officer). [[A06 - Seed data and scenario simulator|A06]] makes this dataset the seed, fixes its inconsistencies (§8) and adds product-map short codes such as BAY01 and MPA01.

**Prototype-only artefacts that are not built:** the "view as" role switcher bar (not built in any form or build, D20: each user type signs up or signs in through its own front door and lands on its own home), the screen-code readout, the one-click `login` (replaced by passkey sign-in), toasts that stand in for real side effects, the striped "photo ·" placeholders (replaced by images with the same pill overlays), the `hint-placeholder-*` attributes and the in-script mock data.

## 3. Design tokens

A05 extracts these values from the dc file and emits them twice from one source: as TypeScript constants and as `tokens.css` custom properties (`--rx-…`). A token-diff test fails if a hex value exists in the dc file but not in the package, or in the package but not in the dc file. Components may use only tokens: lint rejects any colour or font literal in app code.

**Colours (35 tokens).** The core palette is cream `#F5F2EC`, ink `#17191C`, green `#1F5E4B` and brick `#9A4524`.

| Token | Hex | Main use |
|---|---|---|
| `bg.page` (cream) | `#F5F2EC` | Page backgrounds, text on dark surfaces, value chips |
| `ink` | `#17191C` | Default text, dark bars, active chips, selected borders, dark aside |
| `green` | `#1F5E4B` | Primary buttons, links, eyebrows, progress fills, positive numbers, focus outline |
| `green.hover` | `#174838` | Primary button hover |
| `green.tint` | `#E3EEE8` | Green pill background, "Vetted by RealX", decision banner |
| `green.soft` | `#D7E4DC` | Capital bars, allowed zone of the price band |
| `green.wash` | `#F4F8F5` | Completed distribution-chain card |
| `green.onDark` | `#8FC3AC` | Admin logo "X" and nav badges |
| `brick` | `#9A4524` | "Above valuation" text, brick pill text |
| `brick.tint` | `#F5E4DB` | Brick pill background, P2 preview banner |
| `brick.text` | `#7A3A1E` | Preview banner text |
| `red` / `red.tint` | `#A3312A` / `#F6E1DE` | Reject buttons, red pills |
| `amber` / `amber.tint` | `#7A4F00` / `#F6ECD6` | Amber pills (pending, review, due) |
| `grey.fg` | `#4A4D52` | Grey pills, type and phase pills |
| `text.2` / `text.3` / `text.body` | `#45484D` / `#5E6166` / `#2E3135` | Secondary text / muted labels and table headers / long paragraphs |
| `line` / `line.soft` / `line.input` | `#E3DED4` / `#ECE8E0` / `#D6D0C4` | Card borders / inner dividers and tracks / input borders |
| `muted.dark` | `#B9B4AA` | Text on dark surfaces, future step dots, dashed drop box |
| `role.inactive` / `nav.inactive.dark` | `#D9D4CA` / `#C9C4BA` | Inactive role-switcher text / inactive admin nav |
| `bg.input` | `#FBFAF7` | Inputs, selected option card, acknowledgement card |
| `photo.a` / `photo.b` | `#E8E2D6` / `#EFEAE0` | Placeholder stripes (images replace them) |
| `dark.1` / `dark.2` / `dark.3` | `#1C1E22` / `#2A2D31` / `#34383D` | Admin sidebar / user card and role track / active admin nav item |
| `dark.text` / `dark.muted` / `dark.muted2` | `#E8E4DC` / `#8E8A82` / `#A6A198` | Admin sidebar text / section headers / user subtitle |
| `white` | `#fff` | Cards, tables, sponsor sidebar |

**TONE map** (status pills, as background and foreground): green `#E3EEE8 / #1F5E4B` for pass, done, paid, matched; amber `#F6ECD6 / #7A4F00` for pending, escrow, review, due; red `#F6E1DE / #A3312A` for break, frozen, missing, high risk; grey `#ECE8E0 / #4A4D52` for allotted, closed, separate; brick `#F5E4DB / #9A4524` for P2 "2027" previews and "changes requested".

| Group | Values |
|---|---|
| Fonts | **Newsreader** (display serif, 400 and 500; italic only for the logo "X"), **Geist** (UI sans, 400 and 500), **Geist Mono** (400 and 500, for numbers, codes, pills, table headers and eyebrows), all from Google Fonts. The base size is 14px |
| Type scale | Hero h1 Newsreader 64 / 1.02; property h1 52; page h1 44 (public and app) or 40 (consoles); onboarding h1 38; section h2 34, 26, 24; KPI values Geist Mono 26 (24 in admin); pills Geist Mono 11 (10 on cards); table headers Geist Mono 11 uppercase `.05em` |
| Radius | 999 pills and buttons · 14 major cards · 12 standard cards · 10 rows and banners · 8 inputs and chips · 6, 4, 3, 2 small parts · 50% dots |
| Layout | Public main max 1240px, padding 36px 40px 64px; sponsor 1240px; admin 1280px; fixed 240px sidebars; card padding 16 to 36px; grids use `auto-fill` / `auto-fit` minmax |
| Shadows | Only two: the featured card `0 30px 60px -30px rgba(23,25,28,.25)` and the toast `0 12px 30px rgba(0,0,0,.25)`. Elevation otherwise comes from 1px borders and background steps |
| Breakpoints (decided in A05) | The prototype has none. The build adds `bp.phone` 768 and `bp.console` 1024. Public and investor screens must work at 375px with 16px gutters and no page-level horizontal scroll, and tables scroll inside their card. Console sidebars become a drawer below 1024px |
| Dark mode | Not built in P1. Tokens are named so it can be added later |

## 4. Component inventory and the design-system package

A05 builds **`<workspace>\app\packages\ui\`** with 45 of the 46 components of the design inventory as **55 exports** (the prototype's "view as" switcher is not built: D20). It adds a catalogue that shows every component in every state at 1280px and 375px, axe-core checks, and measured WCAG contrast.

| Group | Components (exports) |
|---|---|
| Shells and navigation | Logo, PublicHeader (NavPill, Avatar), SponsorSidebar, AdminSidebar, Footer, Breadcrumb, Tabs (no role switcher, D20) |
| Actions and inputs | Button (7 variants: primary, outlineInk, outlineSoft, danger, dark, link, qty), FilterChip, TextField, OptionCard, DropBox, QuantityStepper, AcknowledgeCard |
| Status and data | StatusPill, KpiGrid / KpiTile, FactsGrid, KeyValueList, DataTable, TaskRow, CheckList / CheckRow, GateRow, Banner, PreviewBanner, Toast, EmptyState |
| Property and money | PropertyCard, FeaturedCard, RaiseBar, BarChart (distributions and capital), PriceBand, OfferProgressCard, OrderSummaryAside, HoldingRow, WalletCard / ValueChip, FeeRow, DocumentRow, ProofBand / ProofRow |
| Flows | VerticalStepper, StepHeader, HorizontalStepper, Timeline, DistributionChain |
| Operator patterns | ApprovalListCard / ApprovalDetail, KycReview / DecisionBanner, TokenOpRow / SignatureMeter |

**New blocks from the design extensions.** These use the same tokens and no new visual language.
- A50 adds **SliderField** (PUB-07), **OrderBook** (INV-06), **ThresholdMeter** (GOV-02, SPN-13, ADM-23), **TextArea** (SPN-12), **ScopePicker** and **MandateSummary** (ACC-05, SPN-16, and the ADM-35 create flow). It also adds the SGN sign-in cards and a partner variant of the light sidebar.
- A51 adds **ExemptionMeter** (ADM-04), **FunnelRow** (ADM-08), **ApprovalsChain**, **MandateTable** (ADM-35), **KillSwitchBanner** (ADM-34, ADM-35), **MaskedField** / **UnmaskButton**, **SignInCard** (ADM-36) and the **hardware-key confirmation modal** that every approval and protective action goes through.

Where the new blocks' code lives is still disputed (see §8). The rule that holds either way: A05 is the only source of tokens, and a new block may compose A05 components but never add a colour, font or radius.

## 5. Prototype screen → product-map screen reconciliation

**The product-map code is canonical** in routes, tests, evidence and the [[Screen Coverage Matrix]]. The code a prototype screen displays is shown only for tracing. A00's `screens.json` freezes this table, and its checker rejects any row where the prototype's display code is used as an ID.

| Prototype key | Code the prototype shows | Product-map screen(s) | Match | Decision | Builds |
|---|---|---|---|---|---|
| `home` | PUB-01 · Home | PUB-01; the how-it-works strip is part of PUB-02 | Same (PUB-02 partial) | Keep. Add an audited-numbers strip in KpiGrid style (product map). A50 draws the full PUB-02 page | A54 (A55 for PUB-02) |
| `invest` | PUB-03 · Investments | PUB-03 | Same | Keep. The count line must follow the filter (bug 11) | A54 |
| `prop` | PUB-04 · Property page; the Sponsor tab shows "PUB-05 · Sponsor" | PUB-04; part of PUB-05 (the Sponsor tab: name, about, vetted pill and four facts only); part of PUB-08 (proof band) and PUB-09 (Documents tab) | Same, but PUB-05 is a partial tab | Build the tab as drawn and the standalone PUB-05 page (`/sponsors/[slug]`) from A50's frames, which add planned against actual per trust, skin in the game, the proof-of-property summary, the incidents and complaints disclosure and "Follow sponsor". A50 draws the full PUB-08 and PUB-09 pages | A54 (A55 for PUB-08 and PUB-09) |
| `fees` | PUB-06 · Fees | PUB-06 | Same | Keep. The fee model question is open (§8) | A54 |
| `onb` | ONB-xx, 8 steps ("Step n of 8") | ONB-01, 02, 03, 05, 06, 08, 09, 10 | Step numbers differ from the IDs. ONB-04 and ONB-07 are missing | Each step keeps its product-map ID and route. The product map wins on flow: SMSF, company and trust go to ONB-04 after ONB-03, joint accounts get a second-person step, and ONB-07 is added in P2 (bug 17) | A56 (A90 for ONB-07) |
| `port` | **"INV-01"** | **POR-01 Portfolio** | **Code mismatch.** Product-map INV-01 is Buy tokens | Use POR-01. Show value both at valuation and at last price (bug 9) | A58 |
| `buy` | **"INV-02 · Buy"**, three steps | **INV-01 Buy tokens, INV-02 Review and confirm, INV-03 Order status** | **Code mismatch, and three screens folded into one** | Keep the single stepped screen. Step 0 is INV-01, step 1 is INV-02 and step 2 is INV-03, each with its own route (`/invest/[code]/buy`, `/review`, `/orders/[id]`). Add the "no cooling-off" statement and the documents-opened precondition to INV-02, and the scaled-back, refunded and cancelled states to INV-03 | A57 |
| `wallet` | "INV · Wallet" | POR-05; part of POR-06 (token wallet card) and POR-04 (ledger table) | No product-map code shown | Use POR-05. A50 draws POR-06 as its own page | A59 |
| `activity` | **"INV-03"** | **POR-04 Transactions** | **Code mismatch.** Product-map INV-03 is Order status | Use POR-04. The page title may stay "Activity" | A58 |
| `statements` | "INV · Statements" | POR-07; part of POR-03 (totals only) | No product-map code shown | Use POR-07. Per-trust distribution history moves to POR-03 (A50) | A58 |
| `sell` | INV-05 · P2 | INV-05 (P2 preview) | Same | Keep the P1 preview banner and save drafts only. The lock-in is a per-listing parameter (the product-map default is 3 months, not the prototype's "No lock-in"). The board stays off under the `resale.board` flag. Sell is offered only from a holding (bug 10) | A92 (P1 links from A58) |
| `spDash` | SPN-02 | SPN-02 | Same | Keep | A62 |
| `spNew` | "SPN-03–06" | SPN-03, 04, 05, 06 | Four screens in one wizard | Keep one wizard. Each step keeps its own ID and route. A Missing item blocks submission | A63 |
| `spOffer` | SPN-07 | SPN-07 | Same | Keep | A63 |
| `spRegister` | SPN-08 | SPN-08 | Same | Merge the design's wallet and Frozen columns with the product map's classification and residency columns and export. The sponsor sees "Frozen" but never the reason | A64 |
| `spDist` | SPN-09 | SPN-09 | Same | The product map wins on the approval chain (sponsor accountant → sponsor owner → RealX fund accountant → CFO, with approval bound to the payment-file hash). The DistributionChain component renders it | A64 |
| `spReport` | SPN-10 | SPN-10 | Same | The attestation gets its own state (bug 2) | A64 |
| `adCmd` | ADM-01 | ADM-01; part of ADM-02 (alert rows) | Same (ADM-02 partial) | The product map wins on content: role-filtered tiles, queues with SLAs, clocks and chain health. The design wins on the KpiGrid plus TaskRows presentation | A66 |
| `adApprovals` | ADM-30 | ADM-30 | Same | Add the hardware-key confirmation, the 72-hour expiry, impact and TTL columns, and bulk approve for low-risk items (product map) | A66 (A53 modal) |
| `adListing` | ADM-07 | ADM-07 | Same | Keep go-live approval in ADM-30 (maker-checker). Add "Return with comments" | A71 |
| `adKyc` | ADM-10 | ADM-10 | Same | Tighten: a KYCA is the maker, the MLRO is the checker (4E), and a reason code is mandatory. The CCO cannot clear a PEP hit alone | A68 |
| `adToken` | ADM-14 | ADM-14; part of ADM-15 (the OP-7731 row) | Same (ADM-15 partial) | Keep. An operation is complete only after reconciliation. A51 draws the full ADM-15 desk | A72 |
| `adRecon` | ADM-17 | ADM-17 | Same | Add from the product map: a break halts resale for that trust (G10) and shows a banner on the sponsor register. Show all 8 trusts (bug 13) and never show escrow as register units (bug 14) | A73 |
| `adTreasury` | ADM-18 | ADM-18 | Same | Keep. The operating account is never counted as client money | A74 |
| `adAml` | ADM-25 | ADM-25 | Same | Keep. Investor, sponsor and support screens show only the effect of a case, never its reason (tipping-off) | A75 |
| `adAudit` | ADM-31 | ADM-31 | Same | Keep. The log is append-only and hash-chained, and agent rows read "agent … on behalf of …" | A66 |

## 6. Undesigned screens and the design-extension nodes

**58 screens are drawn by the extensions.** They are the 45 undrawn product-map screens (ACC-05, SPN-16 and ADM-35 among them), the 8 partly drawn screens that the extensions complete (PUB-05 among them since 24 Sep 2026), and the 5 supplementary admin screens ADM-36 to ADM-40. A00's `screens.json` counts 50 undrawn rows (45 + 5). The extensions also draw the 4 sign-in shell screens (SGN-01 to SGN-04). Each extension is a **new dc file written in the prototype's own language, run by a verbatim, hash-checked copy of `support.js`**. It lives in the app repo, never in the design folder, and uses only A05 tokens and blocks. Both extensions are in milestone AM4.

| Node | File | Screens it draws |
|---|---|---|
| [[A50 - Design extension A- investor and sponsor screens not in the prototype\|A50]] | `app/design-ext/investor-sponsor/RealX App Ext A.dc.html` | 27 product-map screens: PUB-02, PUB-05 (the standalone page), PUB-07, PUB-08, PUB-09, ONB-04, ONB-07, INV-04, INV-06, INV-07, POR-02, POR-03, POR-06, GOV-01, GOV-02, ACC-01 to **ACC-05**, SPN-01, SPN-11 to SPN-15 and **SPN-16**. Also SGN-01 to SGN-04, the partner frame and the drawn-screen supplements of the investor, public and sponsor surfaces |
| [[A51 - Design extension B- operator console screens not in the prototype\|A51]] | `RealX Admin Ext.dc.html` | 26 product-map admin screens: ADM-02 to 06, 08, 09, 11, 12, 13, 15, 16, 19 to 24, 26 to 29, 32 to 34 and **ADM-35**. Also the 5 charter screens ADM-36 to ADM-40, the extended seven-group sidebar (Overview, Onboarding & listings, Assets & money, Markets (P2), Asset life, Risk & records, Platform), the hardware-key modal and the drawn-screen supplements of the operator console |

**Drawn-screen supplements (added 24 Sep 2026).** Covering only the undrawn and partial screens left gaps on screens counted as drawn: product-map components and states that exist in neither the prototype nor an extension, which would have shipped with no design reference and no founder review. Examples: ADM-25's monitoring scenarios, SMR, TTR and IFTI filings and regulatory orders; ADM-18's fees and revenue and escrow committed against received against released; ADM-31's hash-chain verification banner and audited exports; ADM-10's risk factors and points and extracted against entered fields; ONB-08's trusted-contact recovery; ONB-09's PayTo top-ups; INV-03's scaled-back, refunded and cancelled states; POR-04's filters and the D04 "by your agent" marker; and the SPN-03 to SPN-07 wizard and offer-monitor states that A63 builds (named in A50 §10 since 24 Sep 2026, replacing the undefined loop A50.1). So A50 (§10) and A51 (§7) each generate a diff of every drawn screen on their surfaces: the product-map components, actions and states in `data_*.py` against what the prototype region shows. Each missing item is drawn as an extra frame of the existing screen under its parent code, listed in the extension's `screens.json` and in A00's `screens.json` with that parent code, and so covered by HG-REVIEW, by A87 and by the fidelity check of the UI node that owns the parent screen. The generated diff is the authority; the examples above are the minimum.

Each extension ships a `screens.json` manifest. For every screen it lists the product-map code, shell, personas, blocks, states drawn, every action with its maker, checker and policy code, and the action's delegation value, generated from A03's `delegable.ts`. Every dependent UI node reads the manifest and must cover it one to one. [[A100 - Agent-parity suite- every UI action has an API and MCP equivalent under the same gates|A100]] uses the same manifest as its list of UI actions.

**The three D04 screens** are drawn so that the browser decides nothing. Every scope ceiling, limit, refusal and approval right on them is the API's answer from [[A12 - API tokens, scopes, agent principals and mandates for every user type (G22)|A12]].
- **ACC-05 API tokens and agents** (investor app, every investor sub-type). The page has a KpiGrid (active tokens, sandbox tokens, agent actions this month, refused) and a token DataTable (name, agent, prefix, scopes, limits, expiry, last used, status tone). Creating a token is a five-step flow. Its ScopePicker offers only the viewer's own delegable rights. Its MandateSummary states the limits in plain English (for example "A$1,000 an order · A$5,000 a month"). A passkey step-up signs exactly the payload shown, and the secret is shown once with the MCP connection snippet. Per-token activity shows G22 outcomes and "on behalf of". The page also offers narrow (immediate) and widen (passkey), rotate, revoke and revoke all, confirmations above the cap, anomaly holds, a sandbox token, notice settings and "Pause all my agents". Non-delegable acts are shown as reserved to the person. Built by [[A60 - Investor and sponsor API tokens and agents screens (ACC-05, SPN-16)|A60]].
- **SPN-16 API tokens and agents** (sponsor console, including valuer and property-manager members). This is the same page with a tenant dimension: scopes capped at the member's role, and a pending state until a sponsor owner who is not the requester approves with a passkey. When the tenant has only one owner, RealX approves instead ("Waiting for RealX approval"). The owner sees a team tab and tenant-wide activity, and there are a tenant-wide pause, the sponsor agent policy card and a partner variant. Built by A60. [[A105 - Partner portal- trustee-RE, auditor, regulator observer and licensee views|A105]] mounts the same components for platform partners.
- **ADM-35 Agent and API access** (operator console). A KPI strip (with "approvals by agents" always 0) sits above seven tabs: staff's own tokens (the agent can never check, approve or sign), all tokens and mandates across users and tenants, agent principals, the G22 evaluation log, guard (anomalies, holds, and kill switches per token, user, tenant, owner type and platform), rate limits and the platform agent policy (SECO → CCO, 4E), and the MCP tool catalogue. Built by [[A77 - Console- agent and API access (ADM-35)|A77]].

**HG-REVIEW, the founder's design review.** The brief calls this HG-DESIGN. The node specs and [[Human Gates]] use the ID HG-REVIEW. For each extension, the founder opens the dc file in a browser through its "view as" bar and walks every new screen in journey order. The founder then approves or requests changes. Agents prepare the file, a README walk, `screens.json`, side-by-side screenshots of each new screen next to the prototype region it extends, and a one-page list of design findings (DF-A50-…, DF-A51-…). The approval is recorded as `humanApproval` in `app/evidence/A50.json` or `A51.json`. An extension can pass its software gate on the node QA's evidence while HG-REVIEW is still open, but **no dependent UI node starts building until the approval is recorded**. A50's dependents are A52, A55, A56, A58, A59, A60, A62, A64, A92, A93 and A105. A51's dependents are A53, A67 to A77, A92 and A93. Change requests go through the fix loop. A request that would change a product-map requirement is reopened against the product map instead. After HG-REVIEW, A87 writes the accepted fidelity baseline that A88's regression compares against. Timing: A50 builds in AM2 (moved from AM4 on 24 Sep 2026), so its review can be given while AM2 and AM3 build the backend; A51 stays in AM4. A later redraw on either extension is a follow-up loop that inherits the extension's gate (A50.2 for founder redraws on extension A; A50.3 for A96's P3 payout-rail frames), so its requester waits for the new sign-off.

## 7. Rules: who wins

1. **The design wins on look and interaction.** This covers colours, type, spacing, radii, component shapes, layout patterns (the `PG` page pattern of code, title, sub, KPIs, tasks and table), interaction feel and the mock dataset. The BrickX-style coral look of the product-map mocks is superseded. Those mocks are content wireframes only.
2. **The product map wins on requirements and compliance.** This covers screen IDs, routes, flows and branches, states (including empty, refused, halted and preview states), gates G01 to G22, maker-checker roles and policy codes, and compliance wording. Where the product map is stricter, the build adds what the prototype lacks. Examples: hardware-key approvals, the MLRO as checker on PEP hits, "no cooling-off" on INV-02, resale halts on a reconciliation break, and "not public until approved".
3. **New screens extend the design; they do not invent.** A50 and A51 reuse the prototype's shells, tokens and blocks. The extension blocks listed in §4 are the only new visual elements.
4. **The browser decides nothing.** Every pill, banner or disabled button that expresses a rule is rendered from the API's answer. The same rule is enforced server-side, where agent calls meet it under exactly the same gates (D04).
5. **Phone width is a build decision.** Public and investor screens work at 375px. Consoles are desktop-first with a drawer below 1024px (A05).
6. **Fidelity is tested.** Every UI node compares its screens with the prototype or extension region in every listed state, including the drawn-screen supplements of its screens (a standing pass criterion in the [[App Gauntlet]]). A product-map component or state with neither a prototype region nor an extension frame is a gap in A50 or A51, never a place for the builder to invent UI. A87 runs the full fidelity and accessibility pass, and the node QA (Opus 5.5 high) checks design fidelity at every gauntlet loop.
7. **Legal wording is a counsel question.** Copy that states a legal position, such as complaint deadlines, the wholesale test, FIRB or the CAR disclosure, is parameterised and marked "counsel to confirm". It is never written as settled fact.

## 8. Design bugs and conflicts found

**Bugs in the prototype** (design analysis §6.4). These are fixed in the build, never reproduced.

| # | Bug | Resolution | Owner |
|---|---|---|---|
| 1 | `regUnits` is assigned twice, and the first assignment is dead code | Keep the second formula | A06, A20 |
| 2 | The SPN-10 attestation pill reads the distribution-approval flag | Give the attestation its own state | A34, A64 (A87 tests it) |
| 3 | Two different wallet truncations | One rule, first 6 + last 4 (`truncateAddress`) | A05, A06 |
| 4 | Forced transfer OP-7731 names Marion Street, but the estate wallet holds Bay Road | The seed uses BAY01, 60 units | A06 |
| 5 | APR-2279 is used for two different objects | The listing change becomes APR-2278 | A06 |
| 6 | A mint "on offer close" while the offer is still open | UNL01 closed 21 Sep, "Closed · allotting" | A06 |
| 7 | Valuation dates drift from the as-of date | Per-trust valuation history, one as-of date | A06 |
| 8 | Static badges and KPIs | Every count computed from rows | A06, A53, A66 |
| 9 | Holding value shown at offer price only | Show value at valuation and at last price | A58 |
| 10 | Sell opens for a property not held ("20 of 0") | Sell only from a holding | A57, A92 |
| 11 | "From 3 sponsors" is hard-coded | Derive it from the filtered list | A54 |
| 12 | The featured card hard-codes "Open offer" | Bind the property status | A05, A54 |
| 13 | Recon shows 7 trusts against "7 of 8" | All 8, with MPA01 "not issued" | A06, A73 |
| 14 | Escrow applications shown as register units | Register 0 and chain 0, escrow in the client-money column | A18, A73, A55 |
| 16 | One-click login and no sign-in screen | Passkey sign-in (SGN-01 to SGN-04) and hardware-key sign-in (ADM-36) | A11, A52, A53, A105 |
| 17 | The investor-type choice does not branch, and there is no entity or joint step | ONB-04 branch, joint second person | A50, A56 |
| 18 | No G05, G15, G18 or G13 hardware-key step is drawn | The prototype is not treated as complete for gates. The gates are built from the product map | A19, A87 |
| 19 | Marine Parade is public while still in review | Not public until approved (APR-2290) | A36, A54, A62, A66, A71 |

Item 15 in the analysis was a consistency check, not a bug. Item 20 lists the numbers that are internally consistent (for example 6,400 × A$238.40 = A$1,525,760 and 280 tokens held). A06 locks them as seed invariants.

**Conflicts between the design and the product map** that need a decision, not just a fix:
- **Fees.** The prototype has a RealX platform fee of 0.25% a year charged to the trust. The product map has sponsors paying a per-investor monthly platform fee (SPN-15). These are different money flows. The founder must decide. Until then, UI copy shows only the per-listing fee schedule.
- **Withdrawal approval threshold.** The prototype uses A$50,000. The product map shows examples of A$25k and A$61k. The threshold is configuration, defaulting to A$50,000 until the founder or counsel decides.
- **Lock-in.** The prototype shows "No lock-in on this trust". The product-map default is 3 months. Lock-in is a per-listing parameter.
- **Admin navigation.** The prototype has 4 groups and the product map has 10. The build keeps the design's flatter grouping, extended to 7 groups (A51). Module IDs M01 to M27 stay the requirement key.

**Conflicts found by the independent plan check.** These were found at check 1 and addressed in the fix pass. Except for A05, they await the re-check.
- **A05 (passed re-check, with medium findings open).** The same display string takes different tones in the prototype. "Review" is amber, red and grey on different screens, and "Missing" is amber and red. So `toneFor` must key on state, not on text. Green focus rings measure 2.19:1 on `dark.1` and 2.32:1 on ink, below the 3:1 focus-indicator threshold, so the admin sidebar and dark aside need a lighter focus colour. Design borders fail WCAG 1.4.11 non-text contrast (1.37:1 and 1.34:1). They are kept unchanged and recorded as design finding DF-A05-01 for the founder to accept or change, with non-colour cues on five state graphics.
- **A50.** The D04 screens were missing actions and states ("change scopes or limits", confirmations, co-signer states). Some mock dates fell after the 23 Sep 2026 as-of date. The mock secret did not match A12's token format. (Since the second plan-level review, the rule is the opposite: the mock secret keeps the token's shape but deliberately fails A12's pattern, so no committed design file looks like a real token to A01's secrets scanner or to the leak scans; A51 uses the same rule.) INV-06 headroom was 380, but the seed gives 440.
- **A51.** Some drawn maker and checker roles contradicted `data_admin.py`. ADM-35 lacked the platform agent-policy view. Flag keys on ADM-34 were invented instead of taken from A21, A08 and A48. Module IDs for ADM-08 and ADM-33 were wrong. The deeper conflicts:
  - A51's MandateTable overlaps A50's MandateSummary.
  - Where new block code lives is disputed: A05 reserves `packages/ui/src/ext/`, while A53 would build the blocks in `apps/console`.
  - Hosting `support.js` by a relative path breaks the clean-clone rule. Each extension keeps its own verbatim copy.
- **A87.** Its contrast rule was stricter than WCAG and the design. Partner portal pages were outside its fidelity scope (see [[Screen Coverage Matrix]] gap G-9).

**Package conflict.** The product map inside the design zip predates D04 (§1). The vault product map is the one that counts.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]] · [[Decisions]] · [[Human Gates]]
