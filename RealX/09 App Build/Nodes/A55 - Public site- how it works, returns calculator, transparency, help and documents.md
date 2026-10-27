---
title: "A55 - Public site: how it works, returns calculator, transparency, help and documents"
created: 2026-09-24
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am5]
node: A55
milestone: AM5
lane: "Investor app"
oracle: no
human_gate: none
---

# A55 - Public site: how it works, returns calculator, transparency, help and documents

**The four informational pages of the public site, built from A50's design extension in the prototype's visual system: PUB-02 How it works (four steps, "who holds what", plain-English risks above the fold), PUB-07 Returns calculator (annual change −10% to 20%, net yield, amount, 1-10 years, income versus capital breakdown, the one approved disclaimer on every result, integer-cents maths shared with a public `/v1` API operation and its MCP tool), PUB-08 Transparency (A47's `GET /v1/public/transparency`: attested totals only, the per-trust register / on-chain / bank / last run / result rows that are A20's public rows unchanged, contract addresses and audit reports) and PUB-09 Help and documents (search, platform and per-listing documents from A35 with versions, effective dates and superseded marks, notices and help articles). No business logic lives here: every number, pill, date and sentence comes from an API field, from an A43 `site_copy` item that carries a G17 approval record, or from a short allow-listed UI label.**

**Build status:** not started. **Milestone:** AM5 (Investor app and preview). **Lane:** Investor app. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 6-10 agent-hours (build, challenge and verify loops).

**Depends on:** [[A52 - Web app shells, routing and passkey sign-in (public, investor, sponsor)]], [[A50 - Design extension A- investor and sponsor screens not in the prototype]], [[A20 - Three-way reconciliation engine with zero tolerance]], [[A35 - Documents, disclosure library, e-sign and acknowledgements (G12)]], and, with the graph edge still to be added, [[A21 - Licences, feature flags and licence perimeter (G21)]], [[A36 - Offerings- offer terms, listing review, raise monitor (SM-04)]], [[A43 - Communications, notifications, tasks and marketing claims (G17)]], [[A47 - Portfolio, holdings and transactions read models with statements]]
**Unlocks:** [[A79 - End-to-end journeys- investor money path]]

**Graph note for the plan reviewer:** this node reads contracts from A21 (`GET /v1/perimeter/public`), A36 (`GET /v1/public/offerings/{code}` for the calculator's per-trust fee and yield), A43 (published `site_copy` items and `checkClaims`) and A47 (`GET /v1/public/transparency`). None of the four is an ancestor of A55 in `final-graph.json` today. The fix is to add the edges A21 → A55, A36 → A55, A43 → A55 and A47 → A55; all four sit in AM1 or AM3, so no milestone order breaks. Until the edges exist, the Specify step confirms that all four have passed; if one has not, A55 waits. Two follow-up loops on other nodes are also needed before this node can pass (see "Requests to other nodes").

**Oracle rule:** not an oracle node (oracle: no): no money, supply or holdings move here. But a wrong number on a public page is a misleading-claims risk (G17), so the node QA writes a small hidden vector set to `<workspace>\gauntlet\hidden\A55\` before the build starts ([[App Gauntlet]] §2): a reference table for the calculator maths (inputs to the cent and the annualised figure), planted copy (published `site_copy` items and JSX literals) with the expected lint outcome, and fixture transparency and document responses with the expected rendered strings. The builder never reads that folder; the node QA brings the rest of its checks to the Challenge step and files everything under that folder after the gate so A79 re-runs them as regression.

This note is not legal advice. What the general-advice warning must say, whether the calculator needs the wording or relief that ASIC provides for generic calculators, what the risk list must contain, and whether the "who holds what" descriptions are accurate for a given sponsor's structure are counsel questions (HG-LEGAL). Each is an A43 `site_copy` item whose seed row is marked `set_with_counsel` with a `counsel_ref`; counsel's answers become new approved versions of those items, not code changes. The seed uses the design's sample sponsors, trusts and documents and nothing real.

## Reconciled differences with A50

A50's extension pages are the visual reference, but their mock copy and a few layout choices differ from the product map. This table decides each one. The Screen Coverage Matrix rows for PUB-02, PUB-07, PUB-08 and PUB-09 must cite this table (the matrix is not edited by this node). The user may overturn any row at Specify; after Specify a change is a spec defect.

| # | Screen | Item | Product map | A50 extension | Wins | What A55 builds |
|---|---|---|---|---|---|---|
| R1 | PUB-02 | Four steps | Choose, verify, invest, get paid | Verify once / Buy tokens / Get paid monthly / Sell on the board from 2027 (brick "2027" pill) | Product map (content), A50 (wording) | 01 Choose a property trust, 02 Verify once, 03 Buy tokens, 04 Get paid monthly. Resale timing moves into the risks list (R3), so no P2 feature is drawn as a step in P1. |
| R2 | PUB-02 | Wholesale banner | not listed | "Wholesale investors only" banner | A50 | Banner shown when A21's `wholesale_only` is true, with A21's text; never a literal. |
| R3 | PUB-02 | Risk items | Illiquid, no cooling-off, property risk, sponsor risk | Property values fall, Rent stops, No resale until 2027, Sponsor fails, Platform fails | Product map (which risks must be present), A50 (wording) | Six seeded items `RSK-01..06`: No resale until 2027 (illiquid), No cooling-off, Property values fall, Rent stops, Sponsor fails, Platform fails. Counsel sets the final list. |
| R4 | PUB-02 | Where the risks sit | "above the fold, not in a footer" (J01 step 2) | above the fold of the second screen-height | Product map (stricter) | First viewport at both widths (see PUB-02). This is a layout difference from A50 that the fidelity check allows. |
| R5 | PUB-07 | Yield input | not listed | second SliderField, net yield 0-10% | A50 | Yield slider, 0-10% in 0.1% steps, default from the trust's offer (the `vehicle` path under PUB-07) or the stated assumption. |
| R6 | PUB-07 | Result tiles | Estimated return in dollars and %, annualised; income vs capital | Income received / Value at end / Total return | Both | Tiles: Total return (A$ and %), Annualised (% a year), Income received, Value at end. Capital change and the fee sit in the breakdown. |
| R7 | PUB-07 | Default amount | not stated | A$10,000 | A50 | A$10,000. |
| R8 | PUB-07 | Disclaimer | "illustrative only, not a forecast" | "Illustrative only, not a forecast. Past performance is not a guide." | A50 (one exact string) | The single string "Illustrative only, not a forecast. Past performance is not a guide." from `site_copy` `calc.performance.disclaimer` (`set_with_counsel`), exactly once per result. |
| R9 | PUB-08 | Break pill | "break · 23 Sep" | "Break · opened 23 Sep 06:02" | Both, plus the unit count from A20 | "Break · 2 units · opened 23 Sep 2026 06:02 AEST". |
| R10 | PUB-08 | Audited caption | "Capital raised (audited)" | "Audited to 30 Jun 2026" | A50 format, A47 data | "Audited to {as_of}" per tile from A47's `totals[].as_of`; no auditor name (A47 does not carry one). |
| R11 | PUB-08 | Pre-issue rows | register column | "Applications 6,400 (escrow)" | A50 | Register "0", On-chain "0", subline "Applications {escrow_application_units} (escrow)", grey "Not issued" pill. |
| R12 | PUB-09 | Platform group | FSG, terms, privacy, complaints, fees | FSG, Complaints policy, Privacy policy, Sponsor terms | Product map, plus A50's Sponsor terms if A35 publishes it | Whatever A35 returns for `scope=platform`, ordered by the doc-type order list in `help/doc-order.ts`: FSG, platform terms, privacy policy, complaints policy, platform fee schedule, sponsor terms, transparency reports. |
| R13 | PUB-09 | Help articles | "Search and help articles" | no articles group | Product map | A "Help articles" group in A50's row style. |
| R14 | PUB-09 | Empty search | not stated | "No documents match · try the trust name" | A50 | That string, from `site_copy` `help.search.empty`. |
| R15 | PUB-09 | Open toast | not stated | prototype toast "Opens the PDF (G12 acknowledgement recorded)" | Neither (it overstates what is recorded) | "Opens the PDF · your open is recorded", shown only when A35 returns `open_recorded: true` (as A35 instructs). Anonymous opens show no toast. |

## Build brief

**Scope and placement**

- Route group `apps/web/app/(public-info)/` with four pages: `/how-it-works` (PUB-02), `/calculator` (PUB-07), `/transparency` (PUB-08) and `/help` (PUB-09, with `/help/articles/[slug]` for a single article and `/help/documents/[documentId]` for a single document with all of its versions). The group's `layout.tsx` wraps the pages in A52's `PublicShell` (header, footer, phone menu, session hook) and adds nothing of its own. The shell's nav and footer links (Fees, Complaints, Privacy) are A52's; this node only supplies the targets for Complaints and Privacy (`/help/documents/<complaints policy>` and `/help/documents/<privacy policy>`, resolved by doc type at render time, never by a hard-coded id). Each route declares its canonical product-map code and `designExtensionNode: A50` through `packages/domain/screens.ts` (A02), so the design-fidelity step knows which extension page to compare against.
- Feature folder `apps/web/src/features/public-info/` with one sub-folder per screen (`how-it-works/`, `calculator/`, `transparency/`, `help/`), plus `copy/` (the slot map, the seed file, the allow-list and the local structural lint), `labels.ts` (the one enum-to-label map), `parity-manifest.json` and the tests. Pages are server components reading through `packages/sdk` (A09) and rendered statically with a 60-second revalidation matching the API caches. The only client components are the calculator form, the help search box, the document "Open" button (it needs the session) and the copy-to-clipboard affordance on contract addresses.
- No business logic: the pages compute nothing except client-side search over titles already fetched. The calculator's arithmetic is the shared `computeIllustration` in `packages/domain` (see PUB-07); the page never has its own copy of it. Every pill tone, date, meta string, disclosure sentence and phase label is an API field. `npm run verify` greps the owned paths for:
  - hard-coded hex colours outside `packages/ui` tokens;
  - the strings "Matched", "Break", "Not issued", "Wholesale · P1", "PDF ·" and "corporate authorised representative". The only file exempt is `labels.ts`, which holds the single map from API enums to label and tone (below);
  - `Date.now()` and argument-less `new Date()` (the pages show times as the API sends them, formatted by `packages/domain` helpers in `Australia/Sydney`).
- `labels.ts` maps enum values to `{label, tone}` and nothing else: A47/A20 `result` (`matched` → "Matched" green, `break` → "Break" alert red, `not_issued` → "Not issued" grey, `null` → "Awaiting first run" grey), `client_money_result` (`matched` → "Matched" green, `break` → "Break" red, `not_applicable` → "n/a" grey), `halted: true` → "Halted (G10)" grey, and A47's `attestation.kind` (`auditor` → "Audited to", `reconciliation` → "Reconciled to"), plus the two break templates ("Break · {n} units · opened {date}" and "Break · opened {date}"). A test drives every enum value through the map and asserts each rendered pill's label and tone come from it. An enum value the map does not know renders the grey pill "Unknown status" and logs a spec defect, never a guess. A20's own spec names `clean`/`breaks` where A47 names `matched`/`break`; the map is keyed on what A47 actually serves, and Specify confirms the spelling with A47.

**Copy: approved A43 `site_copy` items only (G17)**

- Every sentence on the four pages that is not an API field is a published A43 `marketing_item` of kind `site_copy`, audience `public`, read through A43's public read of published `site_copy` items (the same read A54 uses). This covers steps, who-holds-what, risks, assumptions, the disclaimer and `GA-01`, empty states, toasts, term expansions and the help articles (`body_md`, within A43's 20,000-character limit and link rules).
- Slot keys follow `<page>.<role>.<name>`, for example `how.risk.rsk-01`, `calc.performance.disclaimer`, `help.tax.no-tfn`. The role is one of `risk`, `fee`, `tax`, `performance`, `legal` or `general`, and the local lint uses it (below). `copy/slots.ts` lists every slot the pages render, its role and whether it is mandatory.
- Seeding: `copy/site-copy.seed.json` in this node's folder holds the seed text for every slot, with `set_with_counsel` and `counsel_ref` (`GA-01` general-advice warning, `RSK-01..06` risks, `WHW-01..05` who holds what, `CALC-01` calculator assumptions, `AFCA-01`). A local-only setup script, `copy/seed-site-copy.ts` (RUN_MODE `local` or `test` only), loads it through A43's own API. It uses the seed's staff personas: IRCM drafts, runs the claims check and submits, and the checker set by A43's routing (CFO or CCO) approves. So every seeded item passes A43's `checkClaims` and carries a real sandbox `approval_id`. If the seed owner (A06 or A43) prefers to import the file itself, that is a Specify choice; either way no item reaches the page without an approval record.
- A withdrawn or missing item leaves its slot empty; it never falls back to old text. Mandatory slots fail closed:
  - if `calc.performance.disclaimer` or `GA-01` is not published, the calculator renders no result, only the allow-listed notice "Calculator unavailable";
  - if any `RSK-*` item is missing, PUB-02 renders without the four steps and with the allow-listed notice "This page is being updated". The steps never show without the risks.
- The only literals allowed in the owned paths are short UI labels on `copy/copy.allowlist.json`: column headers, button labels, units, field labels and the two fail-closed notices above. The lint extracts JSX text and template literals and fails on anything else, as A54 does.
- Local structural lint (`copy/lint.ts`, in `npm run verify` and the e2e). It adds nothing to A43's claims ruleset, and it checks only these:
  - (a) the disclaimer string appears exactly once in every result block and in the breakdown footer;
  - (b) no item with role `risk` renders outside the risks block, and the footer carries none;
  - (c) no percentage figure appears in an item with role `performance` or an untagged item, apart from the calculator's labelled inputs and outputs. Items with role `fee` or `tax` may carry percentages (for example the 0.5% transaction fee or the 47% no-TFN withholding rate), and A43's `CLAIM_YIELD_NO_BASIS` still applies to them at source;
  - (d) the disclosure sentence and the phase label appear only as A21 values.
  - The rendered-HTML claims scan runs A43's `checkClaims` with the current ruleset (through `POST /v1/comms/claims/check` under a local test principal) over the visible text of the four pages. API-supplied fields are excluded, because they are linted at source: document titles and `meta` by A35 and A43, trust names by A20 and A47, perimeter text by A21.

**PUB-02 How it works**

- Data: `GET /v1/perimeter/public` (A21) for `disclosure_text`, `phase_label`, `wholesale_only`, `retail_launch_label` and `car_active`; `site_copy` items for everything else. When `car_active` is false the page renders the shell's "offers unavailable" state in place of the steps, and keeps the risks and the "who holds what" block.
- Layout from `design-ext:PUB-02`, adjusted by R1-R4: the home page's three-cell "how" strip (`design:howSteps`, cream cells on a 1px `#E3DED4` grid, Newsreader 32 numeral in green, Geist 500 16 title, `#45484D` description) extended to the four steps of R1 with the same cell style. Then:
  - the "Wholesale investors only" banner (R2);
  - a "Who holds what" `KeyValueList` with five rows (Sponsor, Trustee or responsible entity, Custodian, Registry, RealX) from `WHW-01..05`; the RealX row states that RealX never owns property and never holds investor keys;
  - a "Key risks" `CheckList` with grey pills for `RSK-01..06` (R3), and a "Read the risks" control that expands each item's longer text;
  - links to `/fees` (A54), `/help` and `/calculator`.
- Risks above the fold, made testable (R4): at 1280×800 the "Key risks" heading and its first three items are inside the first viewport (the section sits in the right column beside the steps). At 375×812 the risks block is the first content block after the page header, before the steps, and its heading and first three items are inside the first 812 px. The footer carries no risk text.
- Jargon expanded on first use per N18 (AFSL, RE, CAR, IM, FIRB, TMD) through a `Term` component. It renders the expansion inline the first time and a dotted-underline abbreviation with a tooltip afterwards. The expansions are `site_copy` items.

**PUB-07 Returns calculator**

- Inputs (URL query state so a result is shareable and A100 can drive it):
  - `amount` in whole dollars, A$100 to A$1,000,000, default A$10,000 (R7);
  - `change`, the annual change in property value in basis points, −1,000 to 2,000 in steps of 50 (shown as −10.0% to 20.0% in 0.5% steps);
  - `years`, 1 to 10;
  - `yield`, the assumed net rental yield in basis points, 0 to 1,000 in steps of 10 (R5);
  - `vehicle`, an optional trust code.
- Where the fee and the default yield come from:
  - With `?vehicle={code}` (A54's property page links to `/calculator?vehicle={code}`), the page reads A36's `GET /v1/public/offerings/{code}`. It takes the transaction fee from the offering's fee lines (`fee_bps`, 50 by default from the rule pack) and the default yield from the offering's net yield. Both show as labelled source rows, for example "Transaction fee 0.5% · {code} offer terms" and "Net yield {x}% p.a. · {basis as A36 sends it} · {date}". The user may still move the yield slider; the source row then reads "your assumption".
  - Without `vehicle`, or when the code is unknown or not public (A36 returns 404), the fee and yield are the platform defaults in `calculator/defaults.ts` (`fee_bps: 50`, `yield_bps: 300`). They are labelled as the stated assumptions of `CALC-01`, and a test asserts that the numbers in the `CALC-01` text equal the numbers in the file. An unknown or non-public `vehicle` adds the inline note "Figures for that trust are not available; the stated assumptions are used."
  - Unknown query parameters are ignored. Out-of-range or malformed values of the known parameters render the field error from `TextField` and no result. A deep link with invalid values falls back to the defaults and shows an inline note "Some values were reset."; a valid `vehicle` link never triggers that note.
- Maths: one pure function, `computeIllustration({amount_cents, change_bps, years, yield_bps, fee_bps})`, specified here and implemented once in `packages/domain/calculator.ts` (see "Requests to other nodes"). The page and the `/v1` operation both call it. It is exact in integer cents with BigInt and no floating point until the last step:
  - `fee = round_half_up(amount × fee_bps / 10,000)`;
  - `value_0 = amount`; `value_k = round_half_up(value_{k−1} × (10,000 + change_bps) / 10,000)` for k = 1..years (rounded to the cent each year, never compounding a fraction);
  - `income_k = round_half_up(value_{k−1} × yield_bps / 10,000)` (income on the value at the start of each year, not reinvested);
  - `capital = value_years − amount`; `income = Σ income_k`; `total = capital + income − fee`; `total_pct = total / amount`;
  - `annualised = ((amount + total) / amount)^(1 / years) − 1`, the one floating-point step, computed last from the integer totals and shown to one decimal place.
  - A negative `total` renders in brick `#9A4524`, never green. The function refuses floats and out-of-range inputs with a typed error.
  - It is tested against a committed fixture table plus fast-check properties: zero change and zero yield gives `total = −fee`; capital is monotone in `change_bps`; income is monotone in `yield_bps`; every cent of `total` reconciles to `capital + income − fee`; `value_years` is independent of `fee_bps`.
- Output block (R6): `KpiTile`s for Total return (A$ and %), Annualised (% a year), Income received and Value at end. Then:
  - a breakdown that expands on "View return breakdown": a per-year table (Value at start / Change / Income / Value at end) in Geist Mono, followed by the lines Capital change, Income received, Transaction fee and Total return;
  - an assumptions list from `CALC-01`. Its first line states the model in the same words as the maths: "Growth and income apply to the full amount; the one-off transaction fee is deducted from the result." The other lines: one valuation change per year, income not reinvested, sponsor and platform fees already inside the net yield, no tax, no resale costs, no debt effects beyond the sponsor's stated LVR;
  - on every result and in the breakdown's footer, the exact string "Illustrative only, not a forecast. Past performance is not a guide." (R8) in Geist 500 13 ink, followed by the general-advice warning `GA-01`.
  - The disclaimer is part of the result component, not the page, so no route, deep link or state can render a result without it.
- `SliderField` for the annual change and for the yield (new block from `design-ext:PUB-07`):
  - a 10 px track styled like `PriceBand`. For change: grey `#ECE8E0` outside, light green `#D7E4DC` for 0% to 20%, brick tint `#F5E4DB` for −10% to 0%. For yield: light green across its range;
  - a 3 px ink marker, end and zero labels, and a Geist Mono value readout;
  - keyboard operable (arrow keys by one step, Page keys by ten steps, Home and End), `role="slider"` with `aria-valuemin/max/now/text`.
  - Holding period as `FilterChip`s (1y, 3y, 5y, 10y) plus a numeric field for any year 1-10. Amount as a mono `TextField` with `en-AU` thousands formatting on blur.
- API and MCP equivalent (D04 parity): `GET /v1/public/calculator?amount=&change=&years=&yield=&vehicle=` (no session; scope `public:read` for tokens), in the OpenAPI document and listed as an MCP tool by A104's generator. It calls the same `computeIllustration`, reads the same A36 fee and yield for `vehicle`, and returns `{inputs, fee_source, yield_source, fee_cents, capital_cents, income_cents, value_end_cents, total_cents, total_pct, annualised_pct, years[], assumptions[] (CALC-01 item id and text), disclaimer (calc.performance.disclaimer item id and text), general_advice_warning (GA-01 item id and text)}`, with validation errors in A09's envelope. The page and the operation must agree to the cent. There is no Next.js route handler: A100's allow-list entry for "A55's public calculator route handler" is no longer needed.

**PUB-08 Transparency**

- Data: `GET /v1/public/transparency` (A47; no auth; scope `public:read` for tokens and the MCP server), and `GET /v1/public/transparency/{vehicleCode}` for a deep link `/transparency#{code}`. Nothing on this page reads another source for totals, trust rows, contracts or audit reports.
- Audited totals: a `KpiGrid` with one tile per entry in `totals[]`, in the order the copy slot `transparency.general.metric-order` fixes (Capital raised, Distributions paid, Investors, Trusts on platform). Each tile shows the value (cents formatted by `packages/domain`, counts with thousands separators) and the caption "{labels(attestation.kind)} {as_of as d MMM yyyy}" (R10). A47 includes a metric only when it carries an attestation and omits it otherwise. So:
  - a metric absent from `totals[]` has no tile;
  - when `totals[]` is empty the grid is replaced by the `site_copy` empty state (seed: "Audited figures are published after each audit period. Nothing on this page is an estimate.");
  - the page never renders a number that is not in `totals[]` and never labels anything "tokenised value".
  - A43's published `public_kpi_set` on the home page (A54) cites the same A47 snapshots. The node QA checks that a metric shown on both pages has the same value and `as_of`.
- Reconciliation table: `DataTable` with columns Trust / Register / On-chain / Bank / Last run / Result, one row per entry in `trusts[]`, in the order sent. A47's trust rows are A20's `GET /v1/public/recon` rows unchanged. The rules below compose strings from those fields and compute nothing:
  - `register_units` and `chain_supply` in Geist Mono with thousands separators;
  - a `not_issued` row shows Register "0", On-chain "0", the muted subline "Applications {escrow_application_units} (escrow)" (R11) and a grey "Not issued" pill;
  - Bank is `client_money_result` through `labels.ts`;
  - Last run is `last_run_at` in `Australia/Sydney` as "23 Sep 2026 06:00 AEST";
  - Result is `result` through `labels.ts`. A break renders the alert-red `#A3312A` pill "Break · {|open_breaks[0].delta_units|} units · opened {break_opened_at as d MMM yyyy HH:mm z}" (R9). A cents-only break (no unit delta) reads "Break · opened {date}";
  - a `halted` vehicle adds a second grey pill "Halted (G10)" with the `site_copy` title `transparency.general.halted-title` (seed: "Resale and new issues paused for this trust until the break clears"; P1 wording, the resale board itself is P2).
  - The unit count comes from `open_breaks[]`, which A20's public row carries and A47 passes unchanged. A47's written field list omits `open_breaks[]`, so Specify confirms it with A47. If it is absent, that is a spec defect routed to A47; the page never derives the delta from the register and chain columns.
  - The table never rounds, abbreviates or reconciles anything: 9,998 renders as "9,998", and a component that formats units with "k" or a tolerance fails the node.
  - The table's subline is the `site_copy` item `transparency.general.recon-intro` (seed, adapted from A20's ADM-17 sentence for visitors: "The trustee's register is the legal record. Token supply and client money are compared to it every night; a difference of one unit or one cent is a break.").
- Contracts: a "Contracts" section with one `DocumentRow`-style row per entry in `contracts[]`:
  - title = the trust name matched by `vehicle_code` from `trusts[]`;
  - meta = "token {token_address} · identity registry {identity_registry} · compliance {compliance}", full addresses in Geist Mono with a copy affordance;
  - "View on explorer ↗" when `explorer_url` is not null. When it is null (the local Hardhat chain), the grey pill "Local chain · no explorer" shows instead.
- Audit reports: `DocumentRow`s from `audit_reports[]` (`title`, `period`, opened through A35's open route by `document_id`). When `audit_reports[]` has no entry, the section shows the grey pill "Audit pending (HG-AUDIT)" and the `site_copy` empty state, so nobody mistakes a local build for an audited one. Whether A35 adds a `smart_contract_audit` doc type (A35.2) changes nothing here, because A47 already selects the reports.
- Error state: if the transparency call fails, the page renders "We can't show reconciliation results right now." (a `site_copy` item) with the last successful `as_of` if the cache has one. It never presents a stale table as current, and a 500 from the API is never a Matched row.

**PUB-09 Help and documents**

- Data: `GET /v1/documents/public?scope=platform` (A35) and, for every trust in A47's `trusts[]`, `GET /v1/documents/public?scope=vehicle:<code>` (eight calls with the seed, cached with the page). A35's scope is written as `vehicle:<id>`; A47's public rows carry the trust code, not an internal id, so Specify confirms that A35 accepts the code. If it does not, that is a spec defect routed to A35, and the page does not look ids up elsewhere. A request to A35 for a `scope=all_public` variant is recorded as follow-up loop A35.3 for N12 if the vehicle count grows.
- Each row renders the A35 fields `title`, `doc_type`, `version_no`, `effective_from`, `page_count`, `byte_size` and the composed `meta` string exactly as sent ("PDF · 42 pages · v2, 1 Sep 2026"). The page never recomposes `meta`, and a locally composed meta string fails the node.
- Layout from `design-ext:PUB-09`, adjusted by R12-R14. A search `TextField` sits at the top. Below it, grouped `DocumentRow`s under Newsreader 24 headings:
  - "Platform documents", ordered by `help/doc-order.ts` (R12);
  - "Documents by property trust": one sub-group per trust in `trusts[]` order, holding the information memorandum or PDS, the TMD in P2, the trust deed, the valuation report, the fee schedule and property reports;
  - "Notices" (`significant_event_notice`, `notice`, newest first);
  - "Help articles" (R13, below).
- Superseded versions: each document with `superseded_versions[]` gets a "Previous versions" disclosure list. Its rows carry the grey pill "Superseded · replaced by v{n} on {date}" and remain openable. Opening a superseded version shows the `PreviewBanner`-style note "This version has been superseded" above the open action. Withdrawn documents are absent from the listing, because A35 does not return them.
- Single-document route `/help/documents/[documentId]`: A35 has no anonymous read of one document, so the page resolves the id from the same public listings (platform scope plus every vehicle scope, already cached). Found: the page shows the current version and its `superseded_versions[]`. Not found in any public listing (withdrawn, never public, or unknown): it returns HTTP 404 with the `site_copy` state "This document is no longer published" and a link back to `/help`, never a 500. A dedicated A35 read `GET /v1/documents/public/{documentId}` is recorded as optional follow-up A35.4. If it lands, the page switches to it with the same 404 rule, and the parity manifest maps the route to it.
- Open behaviour (G12's "opened" fact; no acknowledgement here). Every open, signed in or not, calls `POST /v1/documents/versions/{id}/open` through the SDK with the current session, or with none (A35 allows an anonymous open and writes no row for it):
  - A35 returns a five-minute presigned URL and `open_recorded`;
  - signed in, A35 writes one `document_open` row and returns `open_recorded: true`, and the page shows "Opens the PDF · your open is recorded" (R15);
  - anonymous, A35 writes no row, returns `open_recorded: false`, and no toast shows;
  - the file opens in a new tab in both cases.
  - A token acting for a signed-in person is A35's concern (`on_behalf_of`); the page only ever calls the SDK with the current session.
  - Every row shows the version and effective date, so J01 step 9 ("PDF with version and date") is met by the listing, not only by the file.
- Help articles are `site_copy` items with keys `help.<role>.article-<slug>`: getting started, who can invest and the wholesale test, how money moves and escrow, distributions and tax statements, wallets and passkeys, resale timing, fees, complaints and AFCA. They render through the shell's Markdown component with their `published_at`.
- Search is client side over article titles and bodies and document titles already fetched, with `?q=` in the URL and highlights in the results. The empty state is `help.general.search-empty` (R14), with a link to Support and complaints (ACC-04, A59).
- The complaints article always ends with the `AFCA-01` item (address and number are counsel-confirmed). The design's Complaints footer link lands on the complaints policy document, and the article links to it.

**The parity manifest**

- `parity-manifest.json` maps each screen's actions to OpenAPI operation ids and MCP tool names for A100. Every entry names both, and the list covers every action on the four screens:
  - PUB-02: `perimeter.get.public` and A43's public `site_copy` read;
  - PUB-07: the public calculator operation and `offerings.get.public` for `vehicle`;
  - PUB-08: `public.transparency.get` and `public.transparency.get.vehicle`. The product map's "View contract" action is the `explorer_url` field of `contracts[]`, so an agent reaches the same explorer link the page shows;
  - PUB-09: `documents.list.public` (also the resolver for `/help/documents/[documentId]` until A35.4) and `documents.open.version`.
- The manifest declares that none of the four screens has an account action other than "open a document while signed in". That is A35's route, and it is already reachable by a token with `on_behalf_of`.

**Accessibility, phone width and performance**

- At 375 px: no horizontal scroll, 16 px gutters, the recon table scrolls inside its card (`overflow-x:auto`, `min-width 720`) with a visible scroll affordance, the sliders and chips are 44 px tall touch targets, and the help groups collapse to single columns. At 1280 px the layouts follow the design-extension pages except where R1-R15 say otherwise.
- axe-core over the four pages with zero critical or serious findings; `aria-live="polite"` on the calculator result and the search count; visible focus rings from `packages/ui`.
- Each page renders from cache in under 200 ms locally and revalidates every 60 s. The calculator computes in under 5 ms for any valid input. The help page paginates document groups at 50 rows per group with "Show more" (N12).
- Rate limit: A52's public limit (60 requests per minute per IP) and A09's limit on `/v1/public/*` are config values. The Challenge's 1,000-input run sets a higher limit for the test runner's IP through that config in `RUN_MODE=test`, with no code change. One test keeps the default and asserts that the 61st request inside a minute returns `429 rate_limited` in A09's envelope.

## Requests to other nodes

This node does not edit these; the orchestrator schedules them, and Specify confirms that each is scheduled before the build starts.

- **A02 follow-up:** `packages/domain/calculator.ts` exporting `computeIllustration` exactly as specified under PUB-07, with its fixture table and fast-check properties. It is built from this spec, and the node QA's hidden vectors run against it.
- **Public calculator operation:** `GET /v1/public/calculator` as specified under PUB-07, in the OpenAPI document under scope `public:read`. Proposed owner: A47, which already owns the public read family under `public:read`; the orchestrator may name another API module. A104 lists it as an MCP tool, and A100 replaces its allow-list entry for A55's route handler with this matrix row.
- **A47:** confirm that `trusts[]` carries A20's `open_breaks[]` and a null `result`, and the enum spelling of `result`.
- **A35:** confirm that `scope=vehicle:<code>` is accepted; optional A35.4 `GET /v1/documents/public/{documentId}`.
- **A43 or A06:** the owner of A43's public `site_copy` read route name, and whether the seed loads `copy/site-copy.seed.json` or this node's setup script does.
- **Graph:** edges A21, A36, A43 and A47 → A55; the gate G18 (reachability only) on A55, or the coverage-matrix note below.

## Owned paths (other nodes do not edit these)

- <workspace>\app\apps\web\app\(public-info)\
- <workspace>\app\apps\web\src\features\public-info\

The shell, session hook, public middleware and rate limit are A52's. The design-extension pages are A50's. The components in `packages/ui` are A05's: `SliderField` is built inside this feature folder, styled after `PriceBand`, and offered to A05 as a follow-up, not added to `packages/ui` here. `packages/domain/calculator.ts` is A02's (follow-up above). The following belong to other nodes:

- A35: `/v1/documents/public`, the open route and the `meta` composer;
- A47: `/v1/public/transparency`;
- A20: the trust rows inside it;
- A21: `/v1/perimeter/public`;
- A36: `/v1/public/offerings/{code}`;
- A43: the `site_copy` items, their approvals, `checkClaims` and the claims ruleset;
- the owner named for the public calculator operation;
- A54 ([[A54 - Public site- home, investments, property page, sponsor profile, fees]]): the home page's own three-cell strip, `/fees`, the property page and the sponsor profile.

## Design references

- `design:howSteps` in `<workspace>\design\RealX App.dc.html` (screen key `home`, PUB-01, lines 79-87 and the `howSteps` data at line 812): the three-cell strip, `grid repeat(auto-fit, minmax(240px, 1fr))`, 1px `#E3DED4` gaps and border, radius 12, cream `#F5F2EC` cells with padding 26, Newsreader 400 32 numeral in green `#1F5E4B`, Geist 500 16 title, `#45484D` description with `text-wrap: pretty`. PUB-02 reuses this exact cell for four steps, numbered "01" to "04". The home page keeps its three cells (A54).
- `design-ext:PUB-02`, `design-ext:PUB-07`, `design-ext:PUB-08` and `design-ext:PUB-09` from [[A50 - Design extension A- investor and sponsor screens not in the prototype]] (`app/design-ext/RealX App Ext.dc.html`, design analysis §5 "Public site"), read together with the Reconciled differences table above. They are the reference for tokens, layout and components; their mock copy is not. Screen codes on the extension pages are the product-map codes.
- `design:adRecon grid` (screen key `adRecon`, ADM-17) for the table's cell styles: th Geist Mono 500 11 uppercase `#5E6166`, td 13 px, row dividers `#ECE8E0`, the last column a `StatusPill`, the break row's pill in alert red `#A3312A` on `#F6E1DE`. The public table is the same component with visitor-safe columns.
- `design:prop Documents tab` (screen key `prop`, `DocumentRow` block at line 253, `docs` at line 809, toast at line 899): the `DocumentRow` (PDF icon 36×44 outline with "PDF" in Geist Mono 500 8, title, meta, "Open ↗", whole row clickable, hover border ink, radius 10, padding 16/20) and the five Kent Street Terrace meta strings that A35's seed reproduces. The prototype's toast wording is replaced (R15).
- `design:fees page` (screen key `fees`, PUB-06, lines 291-301) for the page-header pattern the four pages share: Geist Mono 500 12 uppercase eyebrow with the screen code, Newsreader 400 44/1.05 h1, 15 px `#45484D` subtitle, `max-width 760` header and `max-width 900` content cards.
- `design:footer` (lines 515-518): the disclosure paragraph is `/v1/perimeter/public.disclosure_text` rendered by A52's shell, not by these pages; the Complaints and Privacy links target this node's document routes.
- Tokens: cream `#F5F2EC`, ink `#17191C`, green `#1F5E4B`, brick `#9A4524`, alert red `#A3312A`, grey pill `#ECE8E0` / `#4A4D52`, light green `#D7E4DC`, brick tint `#F5E4DB`; Newsreader for display, Geist for text, Geist Mono for units, addresses, dates, versions and calculator figures.

## Requirements covered

- Product-map screens:
  - **PUB-02** is done when the four steps (R1), the five-row "who holds what" list and the `RSK-01..06` risks render from approved `site_copy` items, with `phase_label`, `wholesale_only` and `disclosure_text` from A21. The risks block passes the above-the-fold test at both widths, "Read the risks" expands, the `car_active: false` state renders, and a missing risk item hides the steps.
  - **PUB-07** is done when:
    - the product map's three inputs plus the yield slider drive `computeIllustration` from `packages/domain`;
    - the `vehicle` path takes the fee and yield from A36 with labelled sources;
    - the tiles of R6 and the income-versus-capital breakdown render;
    - every result carries the R8 disclaimer and `GA-01`;
    - the page and `GET /v1/public/calculator` agree to the cent;
    - out-of-range inputs never produce a result.
  - **PUB-08** is done when:
    - every tile, trust row, contract and audit report comes from A47's `/v1/public/transparency`;
    - a metric without an attestation is absent and the empty state renders;
    - the seed reproduces the expected rows listed in Pass criteria;
    - contract rows and audit reports render with the pending states;
    - the failure state never shows stale numbers as current.
  - **PUB-09** is done when platform, per-trust and notice documents list with version and effective date, superseded versions stay listed, marked and openable, search works over articles and titles with the empty state, the single-document route returns 404 with the friendly state for a withdrawn document, and opening records an open only for a signed-in person.
- Journeys: **J01** steps 2 (risks above the fold, not in the footer), 7 (calculator result carries the disclaimer), 8 (per-trust register / chain / bank with last run; a break in red with the date opened) and 9 (opens the IM as a PDF with version and date) run in the Playwright journey spec against the seed. The oracle hooks the catalogue names are assertions in that spec: no "tokenised value" string anywhere, the break row's colour and date, and the disclaimer on every result. Steps 1 and 3-6 are A54's.
- Gates:
  - **G10** is done when the `halted` and break fields of A47's trust rows render as specified and nothing on the page decides a halt or a match locally (a planted `halted: true` with `result: matched` renders both pills exactly as sent).
  - **G12** is done when every document row shows version and effective date, a superseded version is marked and remains available, an anonymous open writes no `document_open` row, a signed-in open writes exactly one through A35's route, and the page contains no acknowledgement control (acknowledgement is INV-02's, A57).
  - **G17** is done when:
    - every prose string on the four pages is a published A43 `site_copy` item with an `approval_id`;
    - a withdrawn item leaves its slot empty and the mandatory slots fail closed;
    - A43's `checkClaims` passes over the rendered text and the local structural lint passes;
    - the calculator disclaimer is unconditional;
    - no unattested metric is ever rendered;
    - no percentage appears outside the calculator's labelled fields and `fee` or `tax` items, and no "tokenised value" appears anywhere.
  - **G18**: the product map also tags PUB-09 with G18 (complaints). Complaint intake and the SM-12 clocks belong to A44 and A59, so here G18 is only the reachability rule: the complaints policy and the AFCA article are one click from the footer and from `/help`, checked in the J01 spec. The graph's gate list for A55 (G10, G12, G17) should add G18 scoped to reachability; until it does, the Screen Coverage Matrix records that A44 and A59 own G18 and A55 contributes this reachability check.
- Modules: **M18 Documents and e-sign** (the public face of the library) is done when PUB-09 renders A35's public listing without recomposing any field, the platform document order and grouping are data, and A35's open route is the only way a file is reached.
- State machines: none. The document version states (`published`, `superseded`) are A35's and the `MKT` item states are A43's; they are only displayed or read here.
- Founder requirement D04 and gate G22: the four screens are anonymous reads plus one account action (open while signed in, A35's token-reachable route). The calculator's equivalent is a `/v1` operation with an MCP tool, not a web route. Done when A100's parity suite finds, for every action in `parity-manifest.json`, an OpenAPI operation id and an MCP tool that return what the page shows.

## Gauntlet loop for this node

- **Specify.** The frozen spec is:
  - this note, including the Reconciled differences table;
  - the D19 and D13 sections and J01 of the requirements catalogue;
  - A50's four extension pages;
  - A35's public listing, open route and `meta` rules;
  - A47's `/v1/public/transparency` shape and A20's public row and seed;
  - A21's `/v1/perimeter/public`;
  - A36's public offering read;
  - A43's `site_copy` items, routing, `checkClaims` and public read;
  - A52's shell contract (layout, session hook, public rate limit, Playwright glob);
  - the calculator maths as written here.

  Settled before the node QA writes the hidden vectors:
  - the user confirms or overturns rows R1-R15;
  - the A02 calculator follow-up and the owner of `GET /v1/public/calculator` are scheduled;
  - A47 confirms `open_breaks[]`, a null `result` and the enum spelling;
  - A35 confirms scope by code;
  - A43 or A06 names the `site_copy` read route and the seed path;
  - the graph edges A21, A36, A43 and A47 → A55 exist, or those nodes have passed.

  Any later change to the calculator formula, the rounding rule, the disclaimer string or what counts as "above the fold" is a spec defect, not a fix.
- **Oracle (node QA, Opus 5.5 high, before the build).** Hidden files in `<workspace>\gauntlet\hidden\A55\`:
  - (a) a reference table of 200 calculator inputs with expected cents and annualised figures computed independently from the model above ("growth and income on the full amount; fee deducted from the result"). It includes the corners (A$100 at −10% for 10 years, A$1,000,000 at 20% for 10 years, zero change and zero yield, one year) and both fee sources (platform default and a `vehicle` with a different `fee_bps`);
  - (b) planted copy: `site_copy` items with block and review phrases written straight into the local database as if published (bypassing A43), JSX literals, a role-`risk` item placed outside the risks block and a percentage in a `performance` item, each with the expected lint result, plus near-misses that must pass (a `tax` item with "47%", a `fee` item with "0.5%");
  - (c) fixture transparency and document responses with the exact strings and pill tones the page must render. These include `halted: true`, a cents-only break, a null result, a `totals[]` with one metric missing, an empty `totals[]`, an unknown enum value, and a document withdrawn after a link was shared.
- **Build (Grok 4.7 xhigh, split by the planner).** Four parallel sub-agents, one per screen, on disjoint sub-folders and routes:
  - (1) PUB-02 with the four-cell strip, `KeyValueList`, risks `CheckList`, `Term` component, fail-closed risk slots and the fold tests;
  - (2) PUB-07 with the form and URL state, the `vehicle` source path, the two `SliderField`s, the result component with the unconditional disclaimer, and the page's call into `packages/domain`'s `computeIllustration`;
  - (3) PUB-08 with the attested totals, the trust table through `labels.ts`, contract rows, audit report rows and the failure states;
  - (4) PUB-09 with the grouped listing, previous-versions disclosure, the single-document resolver, the open path, articles and search.

  Then one integration pass:
  - the route group layout on A52's shell;
  - `slots.ts`, `site-copy.seed.json`, `seed-site-copy.ts`, `copy.allowlist.json` and the structural lint in `verify`;
  - `labels.ts` and its test;
  - `parity-manifest.json`;
  - the J01 Playwright spec (steps 2, 7, 8, 9) at 1280×800 and 375×812, with screenshots into `app/evidence/A55/`;
  - axe runs;
  - `app/evidence/A55.json` listing every slot key with its `marketing_item` id, `body_sha256`, `approval_id` and `counsel_ref`.
- **Challenge (node QA, Opus 5.5 high).** Runs the hidden vectors against `computeIllustration`, `GET /v1/public/calculator` and the rendered pages. Then:
  - 1,000 random calculator inputs through the page (driven by URL) and through the API operation, compared to the cent, under the raised test rate limit;
  - the default-limit 429 test;
  - deep links, `vehicle` links (known, unknown, not public), unknown and malformed parameters;
  - the fold test at both widths;
  - the copy plants;
  - a `site_copy` item withdrawn in A43 (the slot goes empty), and the disclaimer and a risk item withdrawn (fail-closed notices);
  - a planted hard-coded disclosure sentence and a planted local `meta` composition;
  - every fixture transparency case, and a comparison of the page's trust rows with A47's response and A20's `/v1/public/recon`;
  - open with a session, without one, and under a token of the signed-in investor, counting `document_open` rows in the database;
  - a withdrawn document's stale link;
  - superseded versions listed and openable;
  - 375 px horizontal-scroll and touch-target checks, and axe;
  - the `Date.now()`, hex-colour and forbidden-string greps, with `labels.ts` as the only exemption;
  - the parity manifest against the OpenAPI operation ids and the MCP tool list;
  - a design-fidelity comparison of the four pages against A50's extension pages, and of the `howSteps` cell against the prototype, on tokens, layout and components only (fonts, colours, spacing, radii, block order except R4), not copy;
  - a scan that no internal id, wallet-to-person link, investor name or unattested figure appears in the rendered HTML.
- **Gate.** Pass unlocks A79. Max three fix loops; each loop re-runs the hidden vectors and the failing challenges.
- **Escalation trigger.** Any of these reopens the spec and goes to the user:
  - a third failure on the same assumption (for example the rounding point in the calculator, what "above the fold" means on a phone, or whether a cents-only break shows a unit count);
  - any fix that would need a change outside the owned paths (A52's shell or middleware, A35's listing or `meta`, A47's transparency response, A20's public rows, A43's items or ruleset, A05's components);
  - `packages/domain/calculator.ts`, the public calculator operation or its MCP tool not landed by the Gate (the node cannot pass on a web-only calculator);
  - a new disagreement between the design extension and the product map that the Reconciled differences table does not cover.

## Node-specific challenges (node QA)

- **The calculator that flatters.**
  - A$10,000 at −10% for 10 years with a 3.0% yield: the capital line must be negative and in brick, income must fall each year as the value falls, `total` must equal `capital + income − fee` to the cent, and the annualised figure must be negative.
  - A$1,000,000 at 20% for 10 years: no overflow, no float rounding in the cents (BigInt), and the per-year table sums exactly.
  - `amount=2395.92` (a float), `change=2001`, `years=0`, `yield=-1`: each refused with a field error and no result, by the page and by the API operation.
  - The deep link `/calculator?amount=50000&change=1500&years=5` opened cold: the result renders with the disclaimer and `GA-01` before any interaction, and `GET /v1/public/calculator` with the same inputs returns the identical cents.
  - `/calculator?vehicle=<a code with fee_bps 75>`: the fee line reads 0.75% with the offer as its source, and the API operation returns the same `fee_cents`. `/calculator?vehicle=NOPE&foo=1`: the stated assumptions are used, the "not available" note shows, `foo` is ignored, and "Some values were reset." does not show.
  - Fails the node: a result rendered anywhere without the R8 string, a page and API that disagree by one cent, or a green figure for a loss.
- **The break that hides.** With the seed:
  - the Gympie Road Retail row reads "9,998 / 10,000 / {Bank as sent} / 23 Sep 2026 06:00 AEST / Break · 2 units · opened 23 Sep 2026 06:02 AEST" in alert red;
  - a planted fixture that closes the break renders "Matched" in green with the new run time;
  - `halted: true` adds "Halted (G10)";
  - a cents-only `cash` break renders "Break · opened {date}" without a unit count;
  - Kent Street renders "Applications 6,400 (escrow)" and "Not issued", and Marine Parade renders "Applications 0 (escrow)" and "Not issued";
  - a vehicle with `result: null` renders "Awaiting first run";
  - an unknown enum renders "Unknown status";
  - the API returning 500 renders the failure state with no table.
  - Fails the node: a page that rounds 9,998 to "10k", shows "Matched" for a row it could not fetch, computes a match or a unit delta from the register and chain columns itself, or shows a trust row that differs from A47's response by one byte.
- **Unattested numbers on the wall.**
  - The fixture drops "Capital raised" from `totals[]`: that tile is absent, not zero.
  - With an empty `totals[]`, the empty state renders.
  - A response with an extra field named `tokenised_value_cents` must never reach the HTML.
  - Then A43's `checkClaims` runs over the rendered text of all four pages, plus a scan for any percentage outside the calculator's labelled fields and `fee`/`tax` items, and for the disclosure sentence as a literal in the bundle rather than an API value. One hit fails.
- **The document that went missing.**
  - The seed's Kent Street IM v1 (superseded by v2 on 1 Sep 2026): v2 is the row, and v1 sits under "Previous versions" with "Superseded · replaced by v2 on 1 Sep 2026". Opening v1 shows the superseded note and still opens.
  - The complaints policy withdrawn in a fixture disappears from the list, and its `/help/documents/[id]` link returns 404 with the "no longer published" state.
  - An anonymous visitor opens the IM: the `document_open` table gains zero rows and no toast shows.
  - Alex Morgan, signed in, opens it: the table gains exactly one row with `channel: ui`, and the toast reads "Opens the PDF · your open is recorded".
  - Alex's token opens it through the SDK: the row carries `on_behalf_of`.
  - The five Kent Street meta strings equal the design byte for byte, because they are A35's.
  - Fails the node: a page that composes "PDF · 42 pages · v2, 1 Sep 2026" itself, hides superseded versions, writes an open for an anonymous visitor, or shows "acknowledgement recorded".
- **Copy that says too much, and copy that goes away.**
  - Plant "guaranteed monthly income" in a published help article, "your capital is protected" in a published risk item and "beats the market" as a JSX literal in step three. The rendered-HTML `checkClaims` scan fails twice and the literal lint fails once, each naming the slot or file and the phrase.
  - Submit a draft `site_copy` item with "guaranteed" through A43: it is refused at submit with `gate_failed` G17 and never reaches the page.
  - Withdraw `how.general.step-02` in A43: its cell is empty after revalidation, with no fallback text.
  - Withdraw `calc.performance.disclaimer`: no result renders anywhere.
  - Set the perimeter fixture to `car_active: false`: PUB-02 renders the offers-unavailable state, the risks and the who-holds-what list remain, and no page shows a stale disclosure sentence.
  - Move the risks block below the steps on the phone layout: the fold test fails at 375×812.
- **Phone and keyboard.** At 375×812:
  - no horizontal page scroll on any of the four pages, and 16 px gutters;
  - the recon table scrolls inside its card;
  - both sliders are operable with arrow, Page, Home and End keys and announce their values;
  - the chips and "Open ↗" are 44 px targets;
  - the search box and result count are announced;
  - axe reports zero critical or serious findings.
  - Fails the node: a page that needs a mouse for a slider, or overflows the viewport.

## Pass criteria

- The hidden vectors in `<workspace>\gauntlet\hidden\A55\` pass 100 percent on three consecutive runs: all 200 calculator references to the cent and to 0.05 percentage points on the annualised figure, every copy plant caught and every near-miss allowed, and every fixture case rendered with the exact string and pill tone.
- Calculator integrity:
  - `computeIllustration`, the page and `GET /v1/public/calculator` agree on 1,000 random valid inputs to the cent;
  - the fast-check properties (zero change and zero yield gives `total = −fee`; monotone in change and in yield; `total = capital + income − fee`; `value_years` independent of the fee) hold over 10,000 cases;
  - every out-of-range or float input is refused by the function, the page and the API operation;
  - the `CALC-01` assumption text states the same model as the function, and the numbers in it equal `calculator/defaults.ts`;
  - the rendered HTML of every result contains "Illustrative only, not a forecast. Past performance is not a guide." and the `GA-01` text exactly once each, including deep links, `vehicle` links and the breakdown view;
  - with the default configuration the 61st public request in a minute returns `429 rate_limited`.
- Transparency integrity. With the seed, the table has exactly these eight rows, in A47's order, every string equal to the fixture:
  - Marion Street, Bay Road and Hawthorn Grove: register equals on-chain as sent, and a green "Matched" result;
  - Gympie Road Retail: "9,998" / "10,000" and the red "Break · 2 units · opened 23 Sep 2026 06:02 AEST";
  - Kent Street, Ann Street and Unley Road: "0" / "0", "Applications 6,400 (escrow)", "Applications 2,800 (escrow)" and "Applications 9,100 (escrow)", each with a grey "Not issued" pill;
  - Marine Parade: "0" / "0", "Applications 0 (escrow)" and "Not issued".

  Trust names are as A20's seed sends them. If A20's seed changes, these rows follow it, and the assertion is that every rendered row equals A47's (and so A20's) public row through `labels.ts` byte for byte. Also:
  - the planted fixture cases (clean, unit break, cents break, halted, not issued, null result, unknown enum, API failure) render as specified;
  - a metric missing from `totals[]` is never rendered in any planted case;
  - contract rows show full addresses and the local-chain pill;
  - "Audit pending (HG-AUDIT)" shows while `audit_reports[]` is empty.
- Documents integrity, proven through the page and by SQL over `document_open`:
  - zero rows from anonymous opens, exactly one row per signed-in open, and `on_behalf_of` set for a token open;
  - superseded versions present, marked and openable;
  - withdrawn documents absent, and their stale link a 404 with the friendly state;
  - every rendered `meta` string byte-equal to A35's response;
  - grouping and order equal to `doc-order.ts`;
  - the toast only when `open_recorded` is true.
- Fold and copy:
  - the risks heading and its first three items are inside the first viewport at 1280×800 and inside the first 812 px at 375×812, asserted by bounding boxes in Playwright three runs in a row;
  - every prose string on the four pages maps to a published `site_copy` item with an `approval_id` in `app/evidence/A55.json`, and a withdrawn item leaves its slot empty (the mandatory slots fail closed);
  - A43's `checkClaims` and the local structural lint pass over the rendered HTML of the four pages, and the literal lint passes over the source;
  - no hard-coded disclosure sentence, phase label, `meta` string, status label outside `labels.ts`, hex colour outside tokens, `Date.now()` or argument-less `new Date()` in the owned paths.
- Design fidelity: the four pages at 1280×800 match A50's extension pages on tokens, layout and components, except where the Reconciled differences table says otherwise. The `howSteps` cell matches the prototype on fonts, colours, spacing and radii per the e2e token comparison. Screenshots at both widths are in `app/evidence/A55/`. `packages/ui` components are used for every block except `SliderField`, which follows `PriceBand`'s track and marker. Copy is not part of this comparison.
- Accessibility and phone: axe-core zero critical or serious findings on all four pages at both widths; no horizontal page scroll at 375 px; 16 px gutters; slider keyboard operation and `aria-valuetext` verified; `aria-live` regions present.
- Parity: `parity-manifest.json` names, for every action on the four screens, an OpenAPI operation id that exists in the OpenAPI document and an MCP tool that exists in A104's tool list, and A100's run over the manifest finds no unmapped action and no web-only equivalent.
- Repo hygiene: line coverage of `apps/web/src/features/public-info/` at least 85 percent, with `labels.ts` and `copy/lint.ts` at 100 percent; `npm run verify` green; only the owned paths changed; `app/evidence/A55.json` and `A55.qa.md` exist and match disk.

## Human gates touched

- None crossed. The node runs on A35's sandbox storage, A20's stub chain source, the local Hardhat address book, A43's sandbox approvals and the seed. No vendor, key, real money or mainnet is involved.
- Recorded for HG-LEGAL rather than encoded. Each of these is a `set_with_counsel` `site_copy` item; counsel's answer becomes a new approved version through A43's routing, followed by a re-run of the lint, fold and disclaimer tests, not a gate on this node:
  - the general-advice warning wording (`GA-01`), and whether ASIC's arrangements for generic calculators apply to PUB-07 and what warnings they require (counsel is expected to say);
  - the disclaimer string (R8);
  - the risk list wording (`RSK-01..06`);
  - the "who holds what" descriptions (`WHW-01..05`);
  - the calculator assumptions disclosure (`CALC-01`);
  - the AFCA line (`AFCA-01`).
- HG-AUDIT stays open and visible on PUB-08 as the "Audit pending" pill until the chain nodes clear it. The audited-totals strip shows only what A47 attests (in local runs, A06's sandbox attestations). This is the honest public face of "audited metrics only".

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
