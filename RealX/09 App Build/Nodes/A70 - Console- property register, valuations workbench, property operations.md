---
title: "A70 - Console: property register, valuations workbench, property operations"
created: 2026-09-24
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am7]
node: A70
milestone: AM7
lane: "Operator console"
oracle: no
human_gate: none
---

# A70 - Console: property register, valuations workbench, property operations

**The three operator screens for the physical asset behind every token, built from A34's properties API and A48's lifecycle and wind-down API and nothing else. ADM-05 is the property register: a lifecycle table with filters, and a detail that reuses the PUB-04 tabs inside the admin shell, adds the admin tabs (title and encumbrances, due diligence, valuations, holders, lifecycle) and the supply calculator, and carries the INVM, LEGL and LEGL-to-CFO actions of J12 and J21. ADM-06 is the valuations workbench: the six-monthly schedule, a review with the comparable-sales banner, the token valuation as the server computes it, and the committee approvals chain. ADM-22 is property operations: the obligations table, the compliance score per trust, the title and encumbrance monitor, reserves against target months, vacancy and FIRB vacancy-fee evidence, attestation review, "Record notice", "Escalate to distressed" and the wind-down actions of the halts module (exit, waterfall hand-off, burn, register close). Every privileged act goes through A53's approval pattern and hardware-key modal. Every figure is the server's value. Every action has a named API and MCP twin that a staff agent may use as maker and never as checker.**

**Build status:** not started. **Milestone:** AM7 (Operator console). **Lane:** Operator console. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 10-14 agent-hours (build, challenge and verify loops; the SM-03 back-half forms and the wind-down checklist are their own sub-scope in the Build shape).

**Depends on:** [[A53 - Operator console shell, hardware-key sign-in and approval action pattern]], [[A34 - Properties, valuations and proof of property (SM-03)]], [[A51 - Design extension B- operator console screens not in the prototype]], [[A48 - Halts, kill switches and wind-down]]
**Direct interfaces reached through those ancestors (no new graph edge):** [[A06 - Seed data and scenario simulator]] (the seed and scenarios, via A53), [[A104 - MCP server generated from the OpenAPI document]] (the MCP catalogue, via A53 and A48), [[A10 - Append-only hash-chained audit log]] (`GET /v1/audit/events`, via A14), [[A12 - API tokens, scopes, agent principals and mandates for every user type (G22)]] (staff tokens and mandates). **Needs a graph edge (not added here):** [[A54 - Public site- home, investments, property page, sponsor profile, fees]] is not an ancestor, yet the public-tabs comparison diffs against its PUB-04 page; §9 point (g) asks the orchestrator to add A54 → A70.
**Unlocks:** [[A78 - Operator journey J20 end to end in Docker (AM7 exit)]], [[A80 - End-to-end journeys- sponsor and operations]]

**Oracle rule:** this is not an oracle node (`oracle: no`). No cent, unit or holding moves inside this node: the valuation formula, the attestation flag, the SM-03 guards, the waterfall and the burn are decided and tested in A34, A48, A40 and A28, each under its own hidden tests. The node QA still writes hidden challenge tests to `<workspace>\gauntlet\hidden\A70\` from this note before the build starts ([[App Gauntlet]] §2). They are Playwright specs and component tests against the Docker stack with A06's seed and A11's sandbox authenticator, covering:
- the column-to-field map for the three tables;
- every SM-03 state label and tone;
- every state A51 draws for ADM-05, ADM-06 and ADM-22;
- the refusal-code sentences;
- the parity file and the "never" rows under a staff token;
- a response-tampering fixture that plants a wrong `nav_per_unit_cents` and a wrong `proof_score`;
- the wind-down wrong-order script;
- The node QA's own crops of the prototype's `prop` tab regions and of extension B's three frames.

The builder never reads that folder.

This note is not legal advice. Who sits on the valuation committee, whether a late sponsor attestation may pause that sponsor's new offers, how a court order is recorded, and what a P1 exit resolution needs are counsel and founder questions. They are recorded in A34's and A48's Human gates. This node renders whatever the policy rows and the API return.

## Build brief

This node is UI-only. It covers the `(assets)` route group of `apps/console` (A53 reserved it) and the feature package `src/features/properties/`: data loaders over `packages/sdk`, pure view-model functions and the page components, composed only from `packages/ui` (A05) and the shell exports of A53 (`ConsolePage`, `ConsoleTable`, `FilterBar`, `StatusCell`, `IdCell`, `MoneyCell`, `WhenCell`, `MaskedCell`, `AgentMarker`, `PrivilegedActionFlow`, `usePrivilegedAction`, `HardwareKeyConfirm`, `ApprovalsChain`, `SplitListDetail`, `KillSwitchBanner`).

Boundaries:
- It owns no API route, no table, no job and no seed row. A field, a route or a component state it needs and cannot find is a spec defect for A34, A48, A18, A35, A05, A51 or A06. It is never worked around in the browser.
- Every rule is decided by the API and the screen only shows the answer. The rules include: who may see a property, which transitions are allowed next, whether a valuation needs comparables, whether a proof item is overdue, and whether a wind-down step may run.
- Every money figure and percentage is the API's integer value rendered through A05's `formatAud`, `formatPct` and `formatTokens`, or the API's own `formatted` twin. The browser never adds, subtracts, divides or rounds a money value.
- A lint rule in `src/features/properties/` forbids importing A02's money arithmetic (`addCents`, `subtractCents`, `divideCents`, `mulBps` and the rest of `packages/domain/money`). The only exception is the type imports.

**1. Routes and placement (`app/(assets)/`)**

- `app/(assets)/properties/page.tsx` is ADM-05, with `screen = "ADM-05"` and `designKey = "design-ext:ADM-05"`. `app/(assets)/properties/[code]/page.tsx` is the ADM-05 detail, routed by the property's short code (`KST01`), with `?tab=` in the URL. `app/(assets)/valuations/page.tsx` is ADM-06 (`design-ext:ADM-06`). `app/(assets)/property-ops/page.tsx` is the ADM-22 portfolio view and `app/(assets)/property-ops/[code]/page.tsx` is the per-trust view (`design-ext:ADM-22`).
- The sidebar items are A53's `nav.ts`, generated from A51's `screens.json`. ADM-05 and ADM-06 sit under "Assets & money" and ADM-22 under "Asset life". This node registers three badge sources with A53's `registerBadgeSource`:
  - `valuations.due`: the count from A34's valuation-schedule summary of properties due within `valuation.due_warn_days` or stale;
  - `property_ops.overdue`: overdue obligations plus overdue attestations, from A34's summary read;
  - `assets.review`: properties in `screening`, `due_diligence`, `ic_review` or `title_verification` assigned to the caller's role.
  Each count comes from a server summary, never from counting a list page. None of the three summary reads exists in A34's spec today; they are upstream item (d)(5) in §9.
- It also registers search providers with A53's `registerSearchProvider`: `PRP-` ids and property codes route to the ADM-05 detail, `VAL-` ids to the ADM-06 detail, `ATT-` ids to the ADM-22 per-trust view. Free text calls A34's list with a `q` filter of at least three characters (upstream item (d)(1)), and the text never enters the URL.
- Roles come from A11's `GET /iam/me`: INVM, ASTM, LEGL, VALO, CCO, CFO, CEO, PROP, FINA, TOKO, REGO, SUPER and the read-only roles the permission matrix gives `V` on M03 or M22. Pages and actions a role cannot use are hidden, never greyed, as A53 draws it. A URL reached without `V` shows A53's forbidden state.
- Partner and sponsor principals never sign in here: A53's console is staff-only. The sponsor, TP-VAL and TP-PM acts that J12, J13, J14 and J21 put "on" these screens are made in the sponsor console (A62, A64; valuers and property managers use it under A98's partner principals), and the TP-RE acts in the partner portal (A105), or by those people's agents through the API. These screens show those acts' results and whose turn it is, for example "Waiting for the trustee (TP-RE) · partner portal".

**2. ADM-05 Property register (M03)**

- **List.** `ConsoleTable` over `GET /v1/properties?state=&bucket=&type=&sponsor=&proof=&q=&cursor=` (A34 has `state`, `type`, `sponsor` and `cursor`; `bucket`, `proof` and `q` and the row fields `proof_state`, `holder_cap_bps` and `halt_state` are upstream items (d)(1) and (d)(2) in §9) with columns Code, Property, Sponsor, Type, State, Valued, LVR, Proof, Top holder, in extension B's order.
  - Code is an `IdCell` in Geist Mono.
  - State is a `StatusCell` using the SM-03 tone map in §6.
  - Valued is `valued_on` as "Mar 2026", or "–" when there is no current valuation.
  - LVR is `lvr_bps` through `formatPct`.
  - Proof is `proof_score` as a pill whose tone comes from the row's `proof_state` (`ok` green, `due`/`overdue` amber, `failed` red, `pending` grey), never from a threshold on the score in the client.
  - Top holder is `top_holder_bps` shown against the row's `holder_cap_bps` as "4.8% / 5%", or "–" before issue.
  - The Halt column (the register's G10 mark, §6) reads the row's `halt_state`; the list never calls A48 once per row.
- **Filters.** `FilterChip` groups for lifecycle bucket (All, In diligence = `sourced`…`title_verified`, Raising = `spv_formed`/`listed`, Operating, Distressed, Legal hold, Exit = `exit_proposed`…`sold`, Wound down, Rejected or withdrawn), for type (Residential, Commercial) and for proof (Current, Due, Overdue, Failed), plus a sponsor select fed by A33's tenant list. All filtering is server-side, and the filter state lives in the URL (A53's table rule).
- **Empty states.** A filter with no rows shows "No properties match these filters" with a "Clear filters" link. An empty platform shows "No properties yet · sponsors add them from their console" in the dashed style.
- **Detail header.** Extension B's frame: the property name in Newsreader 40, a mono eyebrow `ADM-05 · KST01 · PRP-…`, address, sponsor, type pills, the lifecycle `StatusPill`, and the halt banner described in §6. It has two parts.
  - **Tab bar.** A05's `Tabs` with the design's 2 px ink underline, holding the four PUB-04 tabs rebuilt from the same A05 components and the same field sources as A54 uses: Summary, Details, Documents, Proof of property. Then the admin tabs: Title and encumbrances, Due diligence, Valuations, Holders, Lifecycle. The public tabs show the admin a property exactly as an investor sees it, with a grey note "As shown on the public page" above them. A property that is not public (before `listed`, or withdrawn) shows the same rendering with the brick `PreviewBanner` "Not public · preview of the listing page".
  - **Summary aside: supply calculator.** A `KeyValueList` titled "Supply calculator · 10,000 tokens", backed by `GET /v1/properties/{id}/supply-preview?units=`. It shows units, property value, cash, debt, liabilities and the token valuation, all returned by the server (A34 returns only `nav_per_unit_cents` today; the breakdown fields are upstream item (d)(3)). The units field is an A05 mono `TextField` whose default is the pack's `offer.default_supply_units`. Each change is debounced 300 ms and calls the endpoint again. An out-of-range value shows the API's validation error. The calculator writes nothing, and it says "Preview · nothing is saved".
- **Title and encumbrances tab.** A34's `TitleVerification` record: `title_ref`, lot and plan, search date, registered proprietor (the A33 entity's name and role pill, custodian or trustee), `proprietor_capacity`, the encumbrance list with holder and document, the dealing numbers, the verification state and approval id, and the optional document-hash anchor with A05's `truncateAddress`. It also shows the `title_encumbrance_change` distress signals from A34's title monitor with their detection dates. The LEGL actions:
  - "Draft title verification" calls A34's draft route with the search document id picked from A35's library through A53's evidence step.
  - "Submit for CCO check" calls A34's `POST .../title-verification/submit`, which creates `M03.titleVerification` (`4E`, LEGL → CCO).
  - The precheck results render as `GateRow`s. `PROP_TITLE_OWNER_MISMATCH` is a hard fail with "Draft again". `PROP_TITLE_SEARCH_STALE` names the search date. The undeclared-encumbrance `warn` becomes an `AcknowledgeCard` the checker must tick in the modal.
  - A rejected title shows the category `title_defect` and the property's move to `rejected`.
- **Due diligence tab.** The `dd_items` for the property type as A05's `CheckList`: kind, issued on, expires on, document row, and a state pill (`uploaded`/`accepted` green, `missing` amber, `expired`/`rejected` red), matching SPN-04's pattern. INVM, ASTM, and LEGL (for `title_search`) get "Accept" and "Reject (note required)" on each uploaded item, through A34's review route. A reject needs a note of 10 to 500 characters and tells the sponsor through A34. Items the sponsor has not uploaded show "Waiting for the sponsor" with no action. Expired items show the expiry date and the rule behind it, for example "title search older than 90 days", as the API returns it.
- **Lifecycle actions in the front half of SM-03.** These appear only where the detail endpoint lists them in `allowed_transitions` for the caller (upstream item (d)(6) in §9). Action codes, routes and makers are A34's §1.1 and §8 tables, copied here, not redefined:
  - INVM moves `sourced → screening` (`POST .../screening/start`, refused `PROP_NOT_SENT` before the sponsor sends it), `screening → due_diligence` and `screening → rejected` (`POST .../screening/decide`, a note required for a reject) and `due_diligence → ic_review` (`POST .../ic-review`, refused with `PROP_DD_NOT_ACCEPTED` listing the items). These are single-actor calls with no approval.
  - "Request IC approval" is `M03.icApproval` (`POST .../ic-approval`, `6E`, maker INVM, checkers two of CFO, CCO, CEO with distinct roles) through `usePrivilegedAction`. The ApprovalsChain shows the two checker stages as they fill.
  - "Start title verification" is `ic_approved → title_verification` (`POST .../title-verification/start`), a LEGL single-actor call.
  - "Amend master data" appears once the state is past `due_diligence`. It is an `M03.propertyAmendment` request (`POST .../amendments`, `4E`, INVM → CCO) whose diff the flow shows. While the state is `sourced`, `screening` or `due_diligence` the edit form calls `PATCH /v1/properties/{id}`; after that any direct edit returns `PROP_EDIT_LOCKED`, which the form renders.
  - The transitions A36, A28 and A93 drive (`spv_formed → listed`, `listed → funded`, `listed → withdrawn`, `funded → operating`, the P2 vote outcomes) have no button here. They show in the history with the driving module's name.
- **Valuations tab.** The published valuation history for this property from `GET /v1/properties/{id}/valuations`: date, valuer, basis, property value, token valuation, change, state (published, superseded). The six published rows use the same `BarChart` as PUB-04's capital-returns chart. A link "Open in valuations workbench" goes to ADM-06 filtered to this property.
- **Holders tab.** A read of A18's cap-table summary through the SDK: holder count, total units against supply, top holders by units with share against the holder cap, and the last certified register version with its date. Holder names come through `MaskedCell` in masked form. There is no unmask here: investor detail is ADM-09's, reached through an `IdCell` link. Before issue the tab says "No units issued yet".
- **Lifecycle tab.** A34's insert-only `property_state_history` as a `Timeline` (state, when, actor with `AgentMarker` where an agent drafted, approval id, reason category). Below it, from `exit_approved` on, the wind-down checklist from A48's `GET /v1/vehicles/{id}/wind-down` as nine `ProofRow`s in A48's fixed order: halt published, holders notified, trustee control, sale recorded, waterfall closed, burn closed, register closed, retention flagged, final report filed. Each row shows its state tone, evidence, approval ids, its blocking reason, and the next allowed action (§5). Before `exit_approved` the tab shows "No wind-down in progress" because the API returns `started: false`.

**3. ADM-06 Valuations workbench (M03)**

- **Layout and schedule list.** The layout is extension B's `SplitListDetail`. The left side is the schedule `ConsoleTable` from A34's valuation schedule read (`GET /v1/valuations/schedule`, upstream item (d)(4) in §9), with columns Property, Last valued, Due, Valuer, Move, State.
  - Due is `next_valuation_due`. It is amber "due in N days" when the API flags `due_soon` (at `valuation.due_warn_days`, 21 on the seed, so Marion Street reads "due in 21 days" as the `adCmd` alert does) and red "stale since …" when the API flags `stale`.
  - Move is `change_bps` of the open valuation, or "–".
  - State covers `scheduled`, `draft`, `submitted`, `in_review`, `committee`, `published` and `rejected`, where `scheduled` is the API's label for a property with a due date and no open valuation.
  - Filter chips: Due soon, Stale, In review, Committee, Published this month, All. The empty state is "No valuations due".
  - "N days" is computed from the response's `as_of` timestamp, never from the browser clock, so A09's injectable clock drives it in tests.
- **Detail.** It shows the valuation id (mono), property, valuer entity with a panel pill ("Panel valuer · registration current to …", or red `VAL_NOT_PANEL` when the API reports it), `valuer_user_id` when a TP-VAL uploaded it, the report as a `DocumentRow`, `valuation_date`, `basis` and `purpose`. A `KeyValueList` lists property value, cash, debt, liabilities, units, token valuation (net assets ÷ units) and change %. Every value is the server's stored figure, and the token valuation is the server's `nav_per_unit_cents`. Below that come the comparable sales as a small `DataTable` (address, sale date, price, source document, note, and the API's per-row `within_age` flag) and the review note history.
- **Banners.** The comparables banner is extension B's `Banner` "Move above ±10% needs comparable sales evidence". It is neutral when the API's `comparables_required` is false. It turns amber with "{n} of 3 comparable sales within 12 months attached" when the API says the move needs them and not enough are attached. A publish request refused with `VAL_COMPARABLES_REQUIRED` shows the API's count. A listing-purpose valuation older than the limit shows `VAL_LISTING_TOO_OLD` with the date. The design's `blocked` state (a 12 % move with no comparables) is this banner in amber with the publish action disabled and the reason named.
- **VALO actions.** Each is a call to A34's valuation routes as its frozen OpenAPI names them:
  - "Take into review" (`submitted → in_review`).
  - "Add review note".
  - "Request more evidence", which sends the sponsor a coded list through A34.
  - "Reject (reason required)".
  - "Send to committee", which is `POST /v1/valuations/{id}/publish-request` and creates `M03.valuationPublish` under the policy row A03 carries (A03 §4.1 row 6 and A34 §8.2: `6E`, maker VALO, checker sets `[CFO]` then `[CCO]` as the interim second committee checker, with the independent member's signed attestation attached as evidence). The form's evidence step requires `independent_attestation_doc_id` from A35's library; without it the API refuses with `VAL_INDEPENDENT_EVIDENCE_REQUIRED`, which the form renders.
  The `ApprovalsChain` renders the chain the approval detail returns: Sponsor submission (a submission, not an approval: shown DONE with the submitting SP-FIN, SP-OWN or TP-VAL user's name) → VALO review → CFO → CCO (interim second checker) with the independent member's attestation as an evidence row → Published, as extension B draws it. The independent member does not sign in or decide in the system under the interim mapping; who that member is and whether they become a deciding principal is A03 §11 (i), named by the founder at HG-REVIEW. If the founder names a deciding principal, A03's row changes, A06 must seed that persona, and this chain renders the new stage from the approval detail with no code change here. The checkers decide in `HardwareKeyConfirm` from this screen, or from the ADM-30 inbox (A66), and both paths call the same A14 decision. A VALO who opens a checker stage sees the modal's `not-eligible` state, which extension B draws as "VALO on the CFO stage".
- **After publish.** The detail shows "Published · token valuation {A$x} from {date} · APR-…". The previous valuation shows as `superseded`. A link "Draft holder announcement" goes to ADM-28 (A75) with the `valuation.published` event id as a query parameter (an id, never text). This screen sends no announcement. In P2, a grey note says "Resale band reference updated (P2)", read from A19's band reference for the vehicle. Before the P2 flag it says "Resale board off until P2".
- **Preview.** A draft's KeyValueList can show `GET /v1/valuations/preview` for edited inputs, labelled "Preview · server-computed". There is no client formula.

**4. ADM-22 Property operations (M22)**

- **Portfolio view.** A `KpiGrid` (trusts operating, obligations overdue, attestations overdue, sponsors with new offers paused) over A34's property-operations summary read (upstream item (d)(5)). Below it, a `ConsoleTable` of trusts with columns Code, Property, Sponsor, State, Compliance score, Next obligation, Attestation (this period's state), Reserves (months covered against target), Vacancy, Halt.
  - The compliance score is A34's per-trust score (A34 §7 defines it as `proof_score` weighted with obligations on time but names no field) rendered from the fields `compliance_score` and `compliance_band` (upstream item (d)(2)) as a mono figure "71/100" with the band's tone.
  - A sponsor whose attestation-overdue listing pause is set gets an amber row note "New offers paused · attestation overdue (G19)", read from a staff read of the tenant's listing pauses (A33 owns the pause; the read is upstream item (d)(7)).
- **Per-trust view header.** Extension B's frame, filled from the seed: for example "BAY01 · Bay Road Apartment · Brighton VIC · score {n}/100" in the admin page header (the product map's "SBK02 · Southbank VIC · score 71/100" is a mock string, not a seeded trust), the SM-03 lifecycle strip that A48 describes (state pill, then the row actions in §5), and the halt banner (§6).
- **Obligations.** A `DataTable` with columns Item, Authority, Due, Amount, Status, Owner, over A34's obligations read for the property.
  - Kinds: council rates, water rates, land tax, insurance renewal, strata levy, compliance certificate renewals (smoke alarm, pool, fire safety, electrical), lease expiry, FIRB vacancy-fee return, valuation due.
  - Status covers `scheduled`, `due`, `paid`, `waived` and `overdue`, with a red pill for overdue.
  - A rates or land-tax item 30 or more days overdue adds extension B's Sev2 alert row linking to the A13 alert id and the note "Shown on PUB-04 within one job cycle".
  - PROP actions: "Record obligation" (kind, authority, due date, amount in cents, recurrence, evidence) and "Mark paid" with an evidence document (`OBL_EVIDENCE_REQUIRED` without one), both `M22.obligation.record` on `POST/PATCH .../obligations`, and "Waive (reason)", `M22.obligation.waive` on `POST .../obligations/{oid}/waive`.
  - The empty state is "No obligations loaded · load the calendar". A34 also opens a Sev4 task for PROP, and this screen links to it.
- **Attestations.** This period's and the last twelve periods' `PropertyAttestation` rows: period, due, grace until, state, attested by (a human's name, never a token), reviewed by, items. PROP reviews an `attested` row with "Accept" or "Flag items" (item codes required) through A34's review route.
  - A flagged row stays amber until a corrected item is re-attested and reviewed. A34's path is `flagged → prepared` (the sponsor corrects every failed item) `→ attested` (a human SP-OPS or SP-OWN signs) `→ reviewed` (PROP), so the pill reads amber in `flagged`, `prepared` and `attested`-awaiting-review, and green only at `reviewed`. That is J14's "evidence upload clears the flag only after PROP review".
  - An `overdue` row shows `late_days` and the tenant pause note. The row never shows or reads any distribution-approval state (design bug 2).
  - The sponsor's attestation items render read-only here in SPN-10's TaskRow pattern. Sponsors edit them only in SPN-10 (A64).
- **Tenancy and reserves.** `KpiTile`s: rent per period, arrears over 14 days, vacancy days and the `vacancy_over_60_days` flag, WALE for commercial, and reserves as "2.1 of 3.0 months" from the latest attestation's `months_covered` and the property's `reserve_months_target`. Below them is a lease summary table from A34's leases and rent-roll read (A34 stores `leases` and `rent_roll_imports` but names no staff read route; upstream item (d)(7)): unit, rent, frequency, paid to, arrears days, arrears, source, import id. Tenant names and contacts never render in the console. The table carries the rent-roll import history (file hash, mapper, rows accepted, error count, state) with a link to the import's error list.
- **FIRB vacancy-fee evidence.** It shows only for a property whose zone flags require it: the attestation item `firb_vacancy_fee` and the obligation `firb_vacancy_fee_return`, with evidence documents and state. Otherwise it shows "Not applicable to this property" with the reason from A34's `default_foreign_policy`.
- **Title and encumbrance monitor.** The Title and encumbrances tab data from §2 in compact form, with the open `title_encumbrance_change` signals highlighted red.
- **Distress signals.** A34's `distress_signals` for the property: kind, source, detected, severity, cleared. They feed "Escalate to distressed".
- **"Record notice".** PROP, single actor, audited. It calls A34's `POST /v1/properties/{id}/notices` (`M22.obligation.record`) with the notice kind (council, tribunal, insurer), a document from A35's library, a note, a due date and an amount. A34 records the notice as a new obligation, and also as a `manual` distress signal when the notice says so; the form's "Also record as a distress signal" choice sends that flag and the screen shows both rows as the API returns them.
- **"Request reserve top-up".** ASTM, `M22.reserveTopUp` (`4E`, ASTM → CFO) through `usePrivilegedAction` on A34's `POST .../reserve-top-up-requests`. The form shows the months short from the latest attestation and the amount; on approval A34 creates a sponsor-owned `reserve_top_up` obligation, which appears in the obligations table.

**5. SM-03 back-half and wind-down actions (A34's and A48's APIs, on ADM-22's strip and ADM-05's Lifecycle tab)**

- **Which actions render.** The action set comes from A48's `GET /v1/vehicles/{id}/lifecycle` ("A34's state plus this node's effects, halts and allowed next actions for the caller"). A button renders only for an action the API lists for this caller, and each button opens `usePrivilegedAction` (or a single-actor form where the row has no approval) against the owner's route below. Each action names the documents it needs, picked from A35's library through the flow's evidence step. Action codes, routes, makers and policies are copied from A34 §1.1, §8.1 and §8.2 (A34 owns every SM-03 row and its effector) and from A48 §3, §4 and §6 (A48 owns the P1 exit track and the wind-down steps). The console renders whatever code, policy and stages the approval detail returns, so a later rename upstream changes no screen logic, only `parity.ts` and the tests.

| Screen action | Transition | Owner route | Action code and policy | Maker (checker) |
|---|---|---|---|---|
| "Escalate to distressed" | `operating → distressed` | A34 `POST /v1/properties/{id}/distress/flag` | `M22.distress`, `4E` | ASTM (CCO) |
| "Remediate" | `distressed → operating` | A34 `POST .../distress/remediate` | `M22.distressRemediation`, `4E` | ASTM or PROP (CCO) |
| "Record court order" | `distressed → legal_hold` only | A34 `POST .../court-orders` | `M22.courtOrder`, `4E` | LEGL (CCO) |
| "Discharge court order" | `legal_hold → operating` | A34 `POST .../court-orders/{cid}/discharge` | `M22.courtOrderDischarge`, `4E` | LEGL (CCO) |
| "Court-directed sale" | `legal_hold → exit_approved` | A34 `POST .../court-orders/{cid}/directed-sale` | `M22.courtDirectedSale`, `4E` | LEGL (CCO) |
| "Record exit resolution" (P1) | none (files the resolution the proposal needs) | A34 `POST .../exit/resolution` | `M03.exitResolution.record`, single actor, not delegable | LEGL |
| "Propose exit" (P1) | `operating → exit_proposed` | A48 `POST /v1/vehicles/{id}/exit/propose` (A48 §6) | A48's `property.propose_exit` row, `4E` | TP-RE or ASTM, or SP-OWN on the wholesale track (CCO) |
| "Approve exit" (P1; the checkers' decision on the proposal, in `HardwareKeyConfirm`) | `exit_proposed → exit_approved` | A48 `POST /v1/vehicles/{id}/exit/approve` | A48's `property.approve_exit` row, `6E` | checkers CCO + LEGL, plus the trustee on the wholesale track (a partner stage in the chain) |
| "Withdraw exit" (P1) | `exit_proposed → operating` | A48 `POST /v1/vehicles/{id}/exit/withdraw` | A48's `property.withdraw_exit` row, `4E` | TP-RE (CCO) |
| "Start sale" | `exit_approved → under_sale` | A34 `POST .../sale/start` | scope `M03.exit.propose`, single actor, no approval | ASTM or LEGL |
| "Record sale completion" | `under_sale → sold` | A34 `POST .../sale/complete` | `M03.saleCompletion`, `4E` | LEGL (CFO) |

  - "Escalate to distressed": the form lists the open distress signals as checkboxes (at least one) by signal id, their evidencing attestation or obligation rows, and the halt the effector will open (`halt_resale` defaults to true; A34's payload). The API's simulation shows the halt's cause, scope and criteria. PROP does not make this request: PROP sees the signals on ADM-22 (J21 step 1's actor), records notices and obligations, and ASTM drafts the flag. Whether LEGL may also draft it (the product map's ADM-05 "ASTM or LEGL") is A34's open A03 §2.7 question; the button follows whatever the lifecycle endpoint lists.
  - "Remediate" is refused until every signal is cleared (`distress_cleared`), and the API's reason is shown.
  - "Record court order" exists only from `distressed`. A court order that lands on an `operating` property is first recorded as a notice with a `court_order` signal and escalated through "Escalate to distressed"; a forced `operating → legal_hold` renders A34's `PROP_ILLEGAL_TRANSITION`. The court-order document is mandatory in the evidence step (its hash goes in A34's payload); the refusal code for a request without it is A34's (upstream item (d)(8)). After approval the screen shows the red legal-hold banner "Distributions paused · court order {doc} · since {date}" from the `distributions_paused` fact.
  - In P2, `operating → exit_proposed → exit_approved` is driven by A93's certified vote; A48's P1 rows refuse with `WD_P2_VOTE_REQUIRED`, and the strip shows the vote id with no button.
  - "Record sale completion" takes the contract and settlement evidence A34's `sale/complete` schema names; any money figure in that schema is a mono money field sent as integer cents. It shows the note "tokens burned after the exit waterfall on ADM-19" (extension B).
- **Wind-down checklist.** These appear on the ADM-05 Lifecycle tab only, in A48's order:
  - "Record trustee control" (`POST /v1/vehicles/{id}/wind-down/trustee-control`, `winddown.trustee_control`, `4E`, TP-RE → CCO) is shown as waiting for TP-RE. The trustee acts through the partner portal (A105) or its agent, and CCO decides here in `HardwareKeyConfirm`.
  - "Start exit waterfall" is `POST /v1/vehicles/{id}/wind-down/waterfall`. The form shows the settlement lines A48 assembles, read-only. It then links to the run in ADM-19 (A74), where FINA, CFO, LEGL and the trustee decide. A shortfall shows `WD_SHORTFALL_UNACCEPTED` and the action "Accept shortfall" (`POST .../wind-down/waterfall/accept-shortfall`, `winddown.waterfall_shortfall_accept`, `6E`, CFO → CEO + LEGL, trustee countersign) where the lifecycle endpoint lists it.
  - "Start burn" is `POST .../wind-down/burn`. It shows the register total from A18 and `totalSupply` side by side as the API returns them, and it renders `WD_WATERFALL_OPEN`, `WD_LINES_UNSETTLED` and `WD_BURN_SUM_MISMATCH` as blocking reasons. After that it links to the batch in ADM-14 (A72).
  - "Close register" is `POST .../wind-down/close-register`, REGO → CCO `4E` plus the trustee's certification. It renders `WD_BURN_NOT_CLOSED` until the burn step closes.
  - A step attempted out of order shows `WD_STEP_ORDER` naming the step that must finish first.
  - The step pill never reads "done" until the API's step state is closed. For example "burn submitted · waiting for reconciliation" follows A53's rule that `confirmed` is not done.
- **Manual halts.** No halt is opened or cleared from these screens except as the effect of A48's transitions. "Open a halt" and "Clear halt" are links to ADM-34 (A69) with the vehicle id, and "Engage inflow freeze" is a link to the same screen.

**6. Gates rendered, never decided (G10, G13, G19, G20) and state tones**

- **G10 Trading halt.** Every detail and per-trust header reads A48's `GET /v1/halts/vehicles/{id}/state` and renders extension B's `KillSwitchBanner` in halt form: cause in plain words, scope, reopening criteria, "expected reopen {ends_at}", ratification state (a red "Unratified" pill once `lapsed`), and the halt id in mono. The text comes from `haltNotice()` fields, never from screen copy. The register list's Halt column shows `halted` with the cause. A vehicle with an A20 recon-break halt shows it even when A48's mirror lags, because the G10 read is the union. A platform `marketplace_suspend` shows a banner on all three screens.
- **G13 Maker-checker.** Every privileged act in §2 to §5 goes through `usePrivilegedAction`, and every decision goes through `HardwareKeyConfirm`. The feature folder imports none of A53's forbidden SDK operations: `approvals.create`, `approvals.submit`, `approvals.decide`, `approvals.bulkDecide`, `approvals.ratify`, `approvals.veto`, `approvals.cancel` and `auth.freshAssertion` (A14's `POST /v1/approvals`, `/submit`, `/decide`, `/bulk-decide`, `/ratify`, `/veto` and `/cancel`). A34's and A48's request-creating operations (for example `ic-approval`, `title-verification/submit`, `publish-request`, `distress/flag`, `court-orders`) are allowed only as the `createVia` argument of `usePrivilegedAction` (§9 point (a)), which a static test checks. The screens render the modal's `not-eligible` and `agent` states from the API, and every A14 refusal code with its sentence.
- **G19 Proof of property current.** The Proof of property tab, the ADM-05 Proof column, the ADM-22 attestation panel and the tenant pause note all read A34's `proof` and `attestations` responses and the tenant listing-pause read (upstream item (d)(7)). An overdue item is amber on the register and, through A54, on the public listing (the ADM-05 state the product map names). No screen action changes an attestation's state except PROP's review.
- **G20 Valuation current.** The ADM-06 due and stale tones, the stale banner on the ADM-05 detail ("Valuation older than 6 months · resale may halt (G20)") and the valuation-stale halt (§6 G10) come from A34's `valid_until`, `next_valuation_due` and the API's `due_soon` and `stale` flags. Publish only happens through the committee chain.
- **SM-03 tone map.** One file, `src/features/properties/tones.ts`, generated from A03's SM-03 state list and tested against it, gives each of the 20 states a label and a tone. The mapping uses only A05's five tones plus A48's ink rule for `wound_down`:
  - green: `ic_approved`, `title_verified`, `spv_formed`, `listed`, `operating`;
  - amber: `screening`, `due_diligence`, `ic_review`, `title_verification`, `distressed`;
  - red: `legal_hold`, `rejected`;
  - grey: `sourced`, `funded` (A05 already maps "funded" to grey), `withdrawn`, `exit_proposed`, `exit_approved`, `under_sale`, `sold` (A48's strip and A51's ADM-22 frame draw every `exit_*` state and `under_sale` grey);
  - ink: `wound_down`.
  Where `toneFor` already lists a string, `toneFor` wins, and this file has no hex. A tone that should differ from A48 and A51 is a design finding for A51 before HG-REVIEW, never a local change.
- **Error and edge states.** Each is one of A53's page states or a named `Banner`:
  - An unknown or invisible property is `404` and shows "not found" (A34 never returns `403` for another tenant's property).
  - A property without a valuation shows "No valuation published yet".
  - A property before its first period shows proof rows as `pending` with "before first period".
  - A vehicle before `exit_approved` shows `steps: []`.
  - `invalid_cursor` resets the list.
  - `agents_paused` and `rate_limited` get A53's sentences.
  - Every A34 and A48 code these screens can receive has a sentence, and a test iterates the OpenAPI catalogue for those two modules. The codes are A34's catalogue (`PROP_*`, `VAL_*`, `ATT_*`, `RR_IMPORT_INVALID`, `OBL_EVIDENCE_REQUIRED`) and A48's (`WD_*`, `HALT_*`), plus the pass-through codes these screens can meet (A14's `APR_*`, A12's `G22_*`, `SCOPE_*` and `428 CONFIRMATION_REQUIRED`). No code outside those catalogues is used or tested.

**7. Agent and API parity for what this node exposes (D04, G22)**

The screens add no endpoint. Every action is an existing A34, A48, A18, A35 or A14 route, reachable through the SDK and through A104's MCP tools under the same gates. The table is exported as `src/features/properties/parity.ts` for A100's suite. "Staff agent" means an `rx_stf_` token under A12's mandate for its human. Segregation of duties counts the agent as its human, so an agent whose human drafted a request can never check it.

| Screen action | API route (owner) | Staff agent |
|---|---|---|
| Read register, detail tabs, supply preview, holders summary, schedule, obligations, attestations, leases, signals, halt state, lifecycle, wind-down | A34 `GET /v1/properties…`, `/history`, `/supply-preview`, `/due-diligence`, `/proof`, `/valuations…` (`M03.property.read`, `M03.valuation.read`), `GET /v1/attestations*`, `.../obligations`, `.../distress-signals` (`M22.propertyOps.read`) and the upstream reads of §9 (d); A18 cap-table read; A48 `GET /halts/vehicles/{id}/state`, `/vehicles/{id}/lifecycle`, `/wind-down` | yes, read scopes never wider than its human, same filters and cursors; mandate `object_ref` limits (allowed properties or vehicles) apply to every read |
| INVM screening and IC-review transitions; start title verification; accept or reject a due-diligence item; take into review, add review note, request evidence, reject valuation; start sale | A34 `screening/start`, `screening/decide`, `ic-review`, `title-verification/start`, `due-diligence/{kind}/review`, `POST /v1/valuations/{id}/review`, `sale/start` | yes within the human's role and mandate; the console renders the API's answer, a 2xx or A12's `428 CONFIRMATION_REQUIRED`, which the person confirms here with `HardwareKeyConfirm` (purpose `confirmation`) |
| PROP attestation review | A34 `POST /v1/attestations/{id}/review` (`M22.attestation.review`) | never: A34 marks it not delegable ("a human judgement that lifts a public pause"), so it has no scope and no MCP tool |
| Record obligation, mark paid, waive, record notice | A34 `POST/PATCH .../obligations`, `.../obligations/{oid}/waive`, `POST .../notices` (`M22.obligation.record`, `M22.obligation.waive`) | yes within the mandate; reported to A101 as a sensitive action where A101's policy says so |
| Request IC approval, submit title verification, amend master data, send valuation to committee, request reserve top-up | A34 `ic-approval`, `title-verification/submit`, `amendments`, `publish-request`, `reserve-top-up-requests` (`M03.icApproval`, `M03.titleVerification`, `M03.propertyAmendment`, `M03.valuationPublish`, `M22.reserveTopUp`) | yes, as maker (G22); the request carries `AgentMarker` in the chain and the inbox |
| Escalate to distressed, remediate, record court order, discharge, court-directed sale, record sale completion | A34 `distress/flag`, `distress/remediate`, `court-orders`, `court-orders/{cid}/discharge`, `court-orders/{cid}/directed-sale`, `sale/complete` (`M22.distress`, `M22.distressRemediation`, `M22.courtOrder`, `M22.courtOrderDischarge`, `M22.courtDirectedSale`, `M03.saleCompletion`) | yes, as maker where A34's row is delegable; the evidence documents must already be in A35's library |
| Record exit resolution | A34 `POST .../exit/resolution` (`M03.exitResolution.record`) | never: not delegable (a legal record) |
| Propose or withdraw exit | A48 `POST /v1/vehicles/{id}/exit/propose`, `/withdraw` | yes, as maker where A48 classifies the route `maker_draft`; `exit/approve` is a checker act and falls under the "never" row below |
| Record trustee control, start waterfall, accept shortfall, start burn, close register | A48 `POST /v1/vehicles/{id}/wind-down/trustee-control`, `/waterfall`, `/waterfall/accept-shortfall`, `/burn`, `/close-register` | yes, as maker where A48 classifies the route `maker_draft`; the checkers stay human |
| Approve, reject, ratify any of the above | A14 `/v1/approvals/{id}/decide`, `/ratify`; A48 `exit/approve` | never. Through the SDK or REST: refused before scope evaluation with `APR_CHECKER_MUST_BE_HUMAN`. Through MCP: A104 excludes every decide, ratify, veto and approve operation (`agent_never_checker`), so the tool is absent from `tools/list` and a forced `tools/call` returns `MCP_UNKNOWN_TOOL` |
| Sign an attestation | A34 `POST /v1/attestations/{id}/sign` | never here: sponsor-side and human-only. Through the SDK or REST: `ATT_SIGNER_NOT_HUMAN`. Through MCP: no tool (`requires_human`), a forced call returns `MCP_UNKNOWN_TOOL`. The console has no sign action |
| Open or clear a manual halt, engage a kill switch | A48 (ADM-34, A69) | not on these screens; A48's rules apply there |

- A static test asserts that every SDK mutation called from `app/(assets)/` and `src/features/properties/` appears in `parity.ts`. A Docker test calls every "never" row with a staff token (§ Challenge lists them) through the SDK and REST, expecting the refusal codes above, and through the MCP server, expecting the tool to be absent from `tools/list` and `MCP_UNKNOWN_TOOL` on a forced `tools/call`; a checker operation forced into a scratch MCP catalogue (A104's test hook) is still refused by the API with `APR_CHECKER_MUST_BE_HUMAN`. Every agent-drafted request, transition and obligation shows A53's `AgentMarker` "agent {prefix id} on behalf of {person}" in the chain, the history timeline and the tables.
- The screens hold no UI-only path: no action exists that a token with the matching scope cannot take through the API, and no screen computes a value that an agent would have to compute for itself.

**8. Privacy, audit and telemetry**

- These screens write no audit rows. Every mutation is audited by the owning module with `actor_id`, `on_behalf_of_id` and `approval_id`. Every error shows the envelope's `request_id`.
- Only ids and codes go in URLs and document titles: property codes, `PRP-`, `VAL-`, `ATT-`, `APR-`, `HLT-`. Investor names on the Holders tab stay masked. Tenant contact details on leases are never requested. Court-order document bodies are never rendered inline, only as `DocumentRow` links through A35's presigned download.
- Nothing is stored in `localStorage` or `sessionStorage` except A53's column-picker preferences.

**9. Upstream items this node needs, and the Specify check**

A34 and A53 pass their gates (AM3, AM4) before this AM7 node starts, so every item below must be written into the owning node's spec now, as a plan change, and not discovered at this node's Specify step. This node does not edit those specs. Items (a) and (d) are recorded here as required changes to A53 and A34; (f) and (g) are for A48 and the graph. Until each is in the owner's frozen OpenAPI or exports, the screen element that needs it is not built and `A70.qa.md` names the gap.

- (a) **A53: module-created approvals (`createVia`).** A53's `PrivilegedActionFlow` drafts through `POST /v1/approvals`, but every A34 and A48 privileged route above creates its own A14 request and returns the approval id. A53's spec must add a `createVia` argument to `usePrivilegedAction` (an SDK operation that returns an approval id), after which the flow continues at Precheck, Submit, Diff and Evidence against A14's detail endpoint. This node never builds a second flow.
- (b) **Settled in A34 already (no change needed):** "Request reserve top-up" is A34's `M22.reserveTopUp` on `POST .../reserve-top-up-requests` (`4E`, ASTM → CFO), and "Record notice" on `POST .../notices` creates an obligation and, when the notice says so, a `manual` distress signal.
- (c) **Settled here:** action codes, routes and makers for SM-03 are A34's (§5 table). The product map's ADM-06 "4E" drawing question is closed: A51's frame now draws `6E` with CFO then CCO (interim) and the independent member's attestation as evidence, matching A03 row 6 and A34.
- (d) **A34: fields and reads this node renders.** A34's spec must add:
  1. list filters `bucket` (the lifecycle groups of §2), `proof` (`ok`, `due`, `overdue`, `failed`) and `q` (three or more characters, code, name or address) on `GET /v1/properties`;
  2. row fields `proof_state`, `holder_cap_bps`, `halt_state` (cause and halt id, from A19's G10 read) on the list, and `compliance_score` with `compliance_band` per trust (the score A34 §7 already defines in words);
  3. the supply-preview breakdown: `units`, `property_value_cents`, `cash_cents`, `debt_cents`, `liabilities_cents`, `nav_per_unit_cents`, each with its `formatted` twin;
  4. `GET /v1/valuations/schedule` with `state` (including `scheduled`), `next_valuation_due`, `due_soon`, `stale`, `change_bps` of the open valuation, `as_of` and cursor paging;
  5. three summary reads: valuations due or stale; property operations (trusts operating, obligations overdue, attestations overdue, tenants with offers paused); properties awaiting a staff role's review (the `assets.review` badge);
  6. `allowed_transitions` on `GET /v1/properties/{id}` for the caller (the front half of SM-03, as A48's lifecycle endpoint does for the back half);
  7. staff reads of a property's leases and rent-roll imports (tenant contact fields excluded), and of a tenant's `attestation_overdue` listing pause (A34 reading A33's pause, or A33 exposing it; A33 must then change too);
  8. a named refusal code for a court-order request without its evidence document.
- (e) **Founder question (none open for this node).** The reserve top-up is already an A34 row; if the founder removes it, the action is dropped from the screen and the gap is recorded.
- (f) **A48: align its restatement of A34's rows with A34.** A48 §3 calls A34's rows `property.flag_distressed`, `property.remediate`, `property.record_court_order`, `property.discharge_court_order`, `property.court_directed_sale` and `property.record_sale`, gives the distress flag the maker "ASTM or LEGL", remediation the maker "ASTM" only, and `exit_approved → under_sale` the actor INVM. A34 §1.1, §8.1 and §8.2 say `M22.distress` (ASTM), `M22.distressRemediation` (ASTM or PROP), `M22.courtOrder`, `M22.courtOrderDischarge`, `M22.courtDirectedSale`, `M03.saleCompletion`, and `POST .../sale/start` by ASTM or LEGL. A34 owns these rows, so A48's text must be corrected. A34 and A48 also both define `operating → exit_proposed` (A34 row 20, `POST .../exit/propose`, ASTM or LEGL, no approval; A48 §3, `property.propose_exit`, `4E`); the two owners must settle one route before A34's and A48's Specify steps. This node renders whatever the lifecycle endpoint lists.
- (g) **Graph: add A54 → A70.** The public-tabs comparison needs A54's PUB-04 page at this node's gate.
- **Specify check.** Before the node QA writes the hidden tests, a script lists every API field, route, filter, action code and error code this note names and checks each one against the frozen OpenAPI documents of A34, A48, A18, A14 and A12 and against A53's exports. A missing one stops the Specify step and is reported as an upstream defect for its owner (the escalation rule below).

## Owned paths (other nodes do not edit these)

- <workspace>\app\apps\console\app\(assets)\
- <workspace>\app\apps\console\src\features\properties\

Inside `src/features/properties/`, by convention: `register/` (ADM-05 list, filters and detail tabs, the supply calculator), `valuations/` (ADM-06 schedule and detail, comparables, the chain wiring), `operations/` (ADM-22 portfolio and per-trust views, obligations, attestation review, tenancy and reserves, notices), `lifecycle/` (the SM-03 strip, the transition forms and the wind-down checklist over A48), `public-tabs/` (the PUB-04 Summary, Details, Documents and Proof of property tabs rebuilt from A05 components for the admin shell), `tones.ts`, `parity.ts`, `badges.ts`, `search.ts`, `loaders.ts` (SDK calls only) and `index.ts`.

Everything else stays with its owner. The API modules are A34's and A48's. The shell, the modal, the flow and the shared blocks are A53's. Components, tokens and formatters are A05's. The extension file is A51's. `apps/web`'s PUB-04 page is A54's, and this node does not import from `apps/web`: the public tabs are rebuilt from the same A05 components and the same SDK reads, and a snapshot test compares them. A needed block or state that neither A05 nor extension B defines goes back to A51 as a finding, never drawn here in raw hex. The A05 token lint runs over the owned paths.

## Design references

- `design:prop tabs` in `<workspace>\design\RealX App.dc.html`. This is the ADM-05 detail's public half, as extension B specifies.
  - The tab bar is at lines 197-201 (the `tabs` loop; 12px 16px padding, Geist 500 14, 2 px `#17191C` underline on the active tab, `#5E6166` inactive, `margin-bottom:-1px` over the `#E3DED4` rule). `TABS` at line 807 names Summary, Monthly distributions, Capital returns, Details, Documents, Sponsor, Proof of property. The admin detail keeps Summary, Details, Documents and Proof of property and adds its own tabs after them in the same style.
  - Summary is at lines 202-242. It contains "About the property" in Newsreader 26, the key facts that line 771 builds ("Cash reserve 3 months", "Next valuation Sep 2026", the holder cap), and the capital-returns caption at line 231, "Independent valuation every 6 months · net assets ÷ 10,000".
  - Details is at lines 243-249: rows `minmax(160px,280px) 1fr`, padding 14px 20px, white card, 12 px radius. The "Foreign investors" row is at line 773, and A34's two `defaultForeignPolicy` strings fill it.
  - Documents is at lines 250-260: the `DocumentRow` with the 36×44 PDF icon and "Open ↗".
  - Proof of property is at lines 274-288. The dark `ProofBand` reads "Register and chain agree", with the mono subline "Trustee register … units = … tokens on-chain · checked …" and the MATCHED pill (`#E3EEE8` / `#1F5E4B`). Five `ProofRow`s follow (`grid 24px 1fr auto`, green ✓ circle, title, 12 px `#5E6166` detail, mono `#45484D` when).
  - The amber, red and pending variants come from A50's `design-ext:PUB-04 proof states` as A34 notes.
- `design-ext:ADM-05` from [[A51 - Design extension B- operator console screens not in the prototype]] (`<workspace>\app\design-ext\admin\RealX Admin Ext.dc.html`, anchor in `screens.json`):
  - FilterChips by lifecycle state.
  - DataTable Code / Property / Sponsor / Type / State / Valued / LVR / Proof / Top holder.
  - The detail reuses the PUB-04 tabs plus "Supply calculator · 10,000 tokens".
  - States: default (all eight seeded trusts, Marine Parade "in review", Gympie Road with a red proof pill), distressed, empty filter.
  - A48 adds the wind-down checklist as `ProofRow`s.
- `design-ext:ADM-06`:
  - `SplitListDetail` with the schedule table Property / Last valued / Due / Valuer / Move / State (Marion Street "due in 21 days" amber).
  - The detail `KeyValueList` (property value, cash, debt, liabilities, token valuation = net assets ÷ 10,000, change percent).
  - The "Move above ±10% needs comparable sales evidence" banner.
  - The `ApprovalsChain`.
  - States: default, blocked (a 12 % move with no sales evidence), not-eligible (VALO on the CFO stage).
- `design-ext:ADM-22`:
  - The compliance-score `KpiTile` per trust.
  - The obligations DataTable Item / Authority / Due / Amount / State / Owner, including FIRB vacancy-fee evidence.
  - Reserves-against-target and tenancy `KpiTile`s (vacancy, arrears, WALE).
  - "Record notice" and "Escalate to distressed" through the modal.
  - The Sev2 alert row at 30 days overdue with the public-flag note.
  - States: default, overdue, distressed, legal hold (the distributions-paused banner).
  - A48's lifecycle strip: `operating` green, `distressed` amber, `legal_hold` red, `exit_*` and `under_sale` grey, `wound_down` ink, with the row actions "Flag distressed", "Record court order", "Discharge", "Propose exit", "Record sale".
- Product-map layouts, whose structure is binding and whose BrickX look is superseded. The ADM-05 mock is a filter row and a table with Proof pills such as "overdue" and "missing 1". The ADM-06 mock is the split with "Move above 10%: attach 3 comparable sales." and the approvals column VALO-1 done → CFO reviewing → Independent next (the built chain renders A03's interim stages from the approval detail, CFO then CCO with the independent member's attestation as evidence, as extension B draws it). The ADM-22 mock header is "SBK02 · Southbank VIC · score 71/100" with Record notice and Escalate buttons, the obligations table, and the Reserves card "Operating 2.1 of 3.0 months" beside the Tenancy card. The coral BrickX styling of the product-map document is never used, and the prototype's tokens and blocks win.
- The admin shell: the `ad` grid and AdminSidebar at lines 621-625 and `navBtn` at line 795 are A53's. The `adCmd` task at line 880, "Marion Street valuation due in 21 days · Sponsor notified · G20", is the ADM-06 due-soon row on the seed. The `adListing` gate rows at line 882, "Proof of property (G19) · Title search 12 Sep 2026 · Pass" and "Valuation current (G20)", read the same A34 facts this node shows. The `DistributionChain` card style at line 844 (DONE `#F4F8F5`, YOUR TURN green border, WAITING muted) is the basis of extension B's `ApprovalsChain`.
- Tokens:
  - Colours: cream `#F5F2EC`, ink `#17191C`, green `#1F5E4B`, brick `#9A4524`, alert red `#A3312A`; amber `#7A4F00` on `#F6ECD6` for due, missing and distressed; red `#A3312A` on `#F6E1DE` for overdue, failed, defective and legal hold.
  - Fonts: Newsreader for page and property titles, Geist for text, Geist Mono for codes, ids, figures, dates, hashes and countdowns.
  - Radii: 12 px cards, 10 px rows, 999 px pills. All come from A05's `tokens.json`.
- Design bugs not reproduced: §6.4 bug 2 (the attestation pill never reads the distribution flag); §6.4 bug 7 (one as-of date drives both the property page's "valued" month and the register's Valued column, so they cannot disagree); §6.4-8 (badges are live reads).

## Requirements covered

- **ADM-05 Property register (screen, owned).** Done means all of the following:
  - The lifecycle table shows the product map's columns with server-side filters and cursor paging.
  - The detail has the product map's tabs (overview/summary, title and encumbrances, due diligence, valuations, proof of property, documents, holders) plus Lifecycle, with the four PUB-04 tabs matching A54's rendering of the same data.
  - The supply calculator is server-computed with the 10,000 preset.
  - The four product-map actions exist through the approval pattern, with A34's codes: IC approval (`M03.icApproval`, `6E`), record title verification (`M03.titleVerification`, `4E` LEGL → CCO), flag distressed (`M22.distress`, `4E` ASTM → CCO; LEGL as a maker only if A03 answers A34's §2.7 question that way) or legal hold (`M22.courtOrder`, `4E` LEGL → CCO, from `distressed`), and record sale completion (`M03.saleCompletion`, `4E` LEGL → CFO).
  - The "proof item overdue: amber on the register" state renders from A34's data.
  - Every state A51 draws for ADM-05 is screenshotted.
- **ADM-06 Valuations workbench (screen, owned).** Done means:
  - The schedule shows due dates every six months and the overdue list.
  - The review shows the report, comparables, change % and valuer independence (panel status).
  - The token valuation is the server's (value + cash − debt − liabilities) ÷ units.
  - Publish runs only through the committee chain (`M03.valuationPublish`, `6E`: CFO, then CCO as interim second checker, the independent member's attestation as evidence), with a link to draft the announcement.
  - The ">6 months: alert, resale may halt" and "move above ±10% needs sales evidence" states render from the API.
  - Every state A51 draws for ADM-06 (default, blocked, not-eligible, older than 6 months, empty) is screenshotted.
- **ADM-22 Property operations (screen, owned).** Done means:
  - Every product-map component is on screen from A34's reads: compliance score per trust; obligations for council rates, land tax, strata, insurance and the certificates, with due dates and status; the title and encumbrance monitor; the sponsor-supplied rent roll; reserves against target months; vacancy including FIRB vacancy-fee evidence.
  - The product-map actions exist: Record notice (PROP, which also adds the obligation); Escalate distressed property (ASTM → CCO, with the resale-halt review and holder disclosure links as hrefs to ADM-21 and ADM-28); Record court order (LEGL → CCO, from `distressed`, the legal-hold banner); Request reserve top-up (`M22.reserveTopUp`, ASTM → CFO); and Remediate (ASTM or PROP → CCO).
  - The two product-map states render: Sev2 at 30 days overdue, and the sponsor's new offers paused at 10 days.
  - The SM-03 back-half actions of §5 and A48's wind-down actions are reachable from the strip and the ADM-05 Lifecycle tab.
- **Journeys, the console steps.** Driven through the screens by A78 and A80, and here by this node's Playwright specs against the seed. Done means each step is a visible control or state that the seeded persona can drive to the stated outcome.
  - J12 steps 5-7: INVM screening, due diligence and IC with two of CFO, CCO and CEO; LEGL title verification with the owner-of-record rule; VALO sending the listing valuation to committee with the independent member's attestation attached, and the CFO then the CCO publishing it.
  - J13 steps 2-3: VALO's review with comparables for a move above 10 %; the committee decision (CFO, then CCO as interim second checker), taken from ADM-06 through `HardwareKeyConfirm` (the ADM-30 inbox path is A66's and is asserted here only as the inbox href with the approval id). Step 4 shows as the new token valuation on the register and the public tabs.
  - J14 steps 2, 4 and 7: PROP records the obligation and the alert shows; ASTM escalates to distressed with CCO approving and the halt banner appearing; then step 7 in full: the sponsor's corrected upload and the human SP-OPS signature (made through the sponsor API in the test setup, with SP-OPS's virtual authenticator for the passkey step-up), PROP's review in ADM-22 accepting the corrected items, ASTM drafting "Remediate" once every signal reads cleared, the CCO approving it in `HardwareKeyConfirm`, the state reading `operating`, and the halt banner and the offers-paused note clearing from `haltState` and the listing-pause read. The product map's outcome "Distressed → operating" is the pass condition, not the flag clearing alone.
  - J21 steps 1 and 4, plus the hand-offs of steps 5 and 6: PROP sees the distress signals on ADM-22 and records the notice, ASTM drafts "Escalate to distressed" and CCO approves (the product map's outcome "Operating → distressed"); LEGL records the court order from `distressed` and the court-directed sale (CCO approving); the trustee-control record (TP-RE through the API in the test setup, CCO deciding here), "Start sale" and "Record sale completion" (LEGL → CFO); the waterfall and burn started from the checklist, with the hand-offs to ADM-19 and ADM-14 asserted as hrefs carrying the run and batch ids (their screens are A74's and A72's). J08 step 8 (P2, tracking the sale) uses the same Lifecycle tab.
- **Gates, rendered and proven not weakened.** The server enforces them in A34, A48, A19 and A14.
  - **G10 Trading halt** is done when every halted vehicle shows its cause, scope, criteria and end date from `haltState` on all three screens, a distress or legal-hold transition shows the halt its effector opens, and no screen opens or clears a halt except through A48's transitions and the ADM-34 link.
  - **G13 Maker-checker** is done when every privileged act listed in §7 reaches A14 only through `usePrivilegedAction` and `HardwareKeyConfirm`, a maker (or a maker's agent) sees `not-eligible`, a staff token is refused every decision, and every A14 refusal code renders.
  - **G19 Proof of property current** is done when the proof score, overdue attestations, the tenant offers-paused note and PROP's review render from A34, and no screen path touches the attestation state except PROP's review. A test asserts that approving a distribution in the seed leaves the ADM-22 attestation pill unchanged.
  - **G20 Valuation current** is done when due-soon at 21 days, stale after 183 days, the listing limit of 92 days and the comparables rule are all shown from A34's flags and codes, and publish exists only as the committee request.
- **Modules.** **M03 Properties and assets** is done when every M03 object A34 exposes (register, detail tabs, lifecycle history, supply calculator, due diligence, title, valuations and the token valuation, the distressed, exit and sold states) has its console surface on ADM-05 and ADM-06. **M22 Property operations** is done when obligations, attestations with evidence, the title and encumbrance monitor, rent roll, reserves, vacancy and FIRB evidence, the compliance score, distress signals and the escalation and wind-down actions have their console surface on ADM-22 and ADM-05.
- **State machine SM-03 Property and listing (rendered).** Done when all 20 states have a label and a tone from `tones.ts` (a test iterates A03's list), the history timeline renders every transition with its driver, every button appears only when the API lists that transition for the caller, and a forced call to a transition the API does not list renders A34's `PROP_ILLEGAL_TRANSITION` (or A48's `WD_STEP_ORDER` for a wind-down step) with a sentence.
- **Founder requirement D04.** Done when `parity.ts` covers every mutation these screens call, each "yes" row succeeds for a staff token within its mandate with `AgentMarker` shown on the result, and each "never" row is refused for that token through the SDK and REST with the codes in §7, and through the MCP server by the tool's absence from `tools/list` and `MCP_UNKNOWN_TOOL` on a forced call.

## Gauntlet loop for this node

- **Specify.** The frozen inputs are:
  - this note, with the §9 upstream items (a), (d), (f) and (g) present in their owners' frozen specs, and the §9 Specify check run with zero missing names;
  - A34's OpenAPI operations and error codes for properties, due diligence, title, valuations, attestations, obligations, rent roll, notices and the proof read;
  - A48's lifecycle, wind-down and halt-state operations and codes;
  - A18's cap-table summary read and A35's document list;
  - A14's detail and decision contracts and A53's shell exports and lint rules;
  - A51's `screens.json` entries and frames for ADM-05, ADM-06 and ADM-22 (after HG-REVIEW) and A50's PUB-04 proof-state frame;
  - A03's SM-03 state list and policy rows for the action codes named here;
  - A06's seed for the eight trusts, its staff personas and tokens, and its `late-property-report`, `stale-valuation`, `agent-authority` and `wind-down` scenarios;
  - A10's `GET /v1/audit/events` contract (filter `approval_id`) and A104's catalogue and `excluded.json`.
  The node QA records `specVersion` and the hashes of those inputs, then writes the hidden specs and fixtures to `<workspace>\gauntlet\hidden\A70\`.
- **Build (Grok 4.7 xhigh, split by the planner).** Suggested shape: one lead session wires the three routes into A53's shell, the loaders, `tones.ts`, `parity.ts`, the badge and search registrations, and a stub page per route. Nothing else starts until the routes render the seed through the SDK in the Docker stack. Then three parallel sub-agents on disjoint folders:
  - (1) `register/` and `public-tabs/`: ADM-05 list and detail with all nine tabs, the supply calculator, the INVM and LEGL front-half flows;
  - (2) `lifecycle/`: the SM-03 strip, the eleven back-half forms of §5 over A34's and A48's routes, and the wind-down checklist with its blocking reasons (its own sub-scope, because it carries most of the parity rows and the wrong-order tests);
  - (3) `valuations/`: the ADM-06 schedule and detail, comparables, banners, the chain and committee wiring, the announcement link;
  - (4) `operations/`: the ADM-22 portfolio and per-trust views, obligations, attestation review, tenancy and reserves, FIRB evidence, the title monitor, notices, the reserve top-up, and the lifecycle strip mounted from `lifecycle/`.
  Integration then covers the Playwright specs over the seed at 1280 and 1024 px for every drawn state, the error-code sentence test over A34's and A48's catalogues, the parity static test and the Docker "never" test, the money-import lint, `npm run verify`, and `app/evidence/A70.json` (route list with `screen` and `designKey`, the screenshot set, the parity table, the error-code coverage table).
- **Challenge (node QA, Opus 5.5 high).** The node QA runs the hidden specs, then works by hand against the running stack:
  - drives J12 steps 5-7, J13 steps 2-3, J14 steps 2, 4 and 7, and J21 steps 1 and 4 through the screens as A06's seeded personas: Grant Ellison (INVM), Sam Reilly (LEGL), Chloe Brennan (VALO), Helen Tran (CFO), Jordan Blake (CCO), Marcus Bell (CEO), Felix Obi (PROP), Tessa Moreau (ASTM), Dev Sharma (FINA), Kai Watanabe (TOKO) and Grace Lim (REGO). The sponsor, TP-VAL and TP-RE steps run through the API in the test setup as those principals (the valuation committee's independent member signs no screen act under the interim mapping; the attestation is a seeded A35 document);
  - replays the response-tampering fixtures;
  - takes the wind-down steps in every wrong order;
  - calls every parity row with staff tokens through the SDK, REST and the MCP server. Tokens A06 does not seed (for example VALO's, PROP's and ASTM's) are created in the test setup by the human persona through A12's token API with the persona's virtual authenticator, because creating a token is non-delegable;
  - compares its crops of the prototype's `prop` tab regions and of extension B's three frames with screenshots at token level;
  - greps the owned paths for money arithmetic imports, hex literals, `Date.now()` in view models, and A53's forbidden SDK operations (§6 G13);
  - checks that only the owned paths changed;
  - writes `app/evidence/A70.qa.md`.
- **Gate.** A pass unlocks A78 and A80 and publishes the hidden tests into `app/`. A fail returns findings by severity. There are at most three fix loops, and each re-runs the whole Challenge.
- **Escalation trigger.** Any of these escalates:
  - a third loop failing on the same finding;
  - any finding that a screen computes or alters a money figure, a token valuation, a proof state or an allowed transition instead of rendering the API's (Critical, stops the milestone);
  - a decision path that bypasses `HardwareKeyConfirm`, or a staff token that gets a 2xx on a "never" row (Critical);
  - a needed field, route or export missing from A34, A48, A18 or A53, found by the §9 Specify check or later (spec defect: reopen upstream, never worked around here);
  - an extension-B frame that cannot be built from A05 and A53 blocks without raw hex (send back to A51);
  - any §9 upstream item not in its owner's frozen spec by Specify, or a founder decision it needs still open (orchestrator asks the user).

## Node-specific challenges (node QA)

- **The +18.4 % valuation that walks past the committee (money, G20, G13).** The node QA uploads a periodic valuation for MRN01 (Marion Street Warehouse, the seeded trust whose valuation is due in 21 days) through A34's `valuation-feed` sandbox with the `-JUMP` marker (a +18.4 % property-value move, no comparables), and submits it through the API as the valuer or SP-FIN, where A34 applies the comparables rule:
  - the submission is refused with `VAL_COMPARABLES_REQUIRED` naming "0 of 3"; ADM-06 shows the draft in extension B's blocked state with the amber banner and the count as the API returned it, and VALO sees no "Send to committee" button because the API offers none;
  - it attaches three comparables, one of them 400 days old, and the refusal must name "2 of 3";
  - with `-JUMP3` the submission succeeds; VALO takes it into review, attaches the independent member's attestation and sends it to committee, and the chain shows CFO then CCO (interim) with the attestation as an evidence row; a send without the attestation renders `VAL_INDEPENDENT_EVIDENCE_REQUIRED`;
  - it intercepts the detail response and changes `nav_per_unit_cents` to a flattering figure: the screen may show the tampered figure only as the API sent it, and the publish request must carry no client figure at all (the request body is inspected).
  After publish, the rendered token valuation on ADM-06, the ADM-05 register and the public tabs must equal A34's published `nav_per_unit_cents` for MRN01, and the node QA checks that figure independently with A02's `navPerUnit` over the valuation's stored inputs (the uploaded property value and MRN01's seeded cash, debt, liabilities and units on issue). Any publish without comparables, any rendered figure that differs from A34's published value, or any request body containing a computed value fails the node. (A34's PAD01 and Marine Parade vectors are A34's unit-test vectors, not seeded trusts, and are not asserted here.)
- **A committee of one (G13, agent misuse).** VALO's agent `valuations-drafter` (created in the test setup) sends the MRN01 valuation to committee. Then:
  - VALO opens the CFO stage in ADM-06 and must see `not-eligible`;
  - Helen Tran's own CFO agent token (seeded in A06's `agent-authority` scenario) calls `/decide` through the SDK and must get `APR_CHECKER_MUST_BE_HUMAN`; through the MCP server the decide tool must be absent from `tools/list` and a forced `tools/call` must return `MCP_UNKNOWN_TOOL`;
  - Grant Ellison (INVM), the maker of an IC approval he drafts in ADM-05 (Marine Parade, once the test setup has its insurance item accepted and moves it to `ic_review`), elevated to CFO under a break-glass session, tries to be its checker and must get `APR_MAKER_IS_CHECKER`;
  - Tessa Moreau (ASTM), who drafted the BAY01 distress flag, elevated to CCO under a break-glass session, tries to approve it and must get `APR_MAKER_IS_CHECKER`;
  - Felix Obi (PROP), who holds no CCO role, opens the distress flag's checker stage and must see `not-eligible` (`APR_ROLE_NOT_HELD` if forced).
  The chain must show `AgentMarker` on the agent-drafted stage. A visible Approve button for any of them fails.
- **Wind-down out of order, and a burn that does not equal supply (money).** On GYM01 in A06's `wind-down` scenario, driven to `exit_approved` in the test setup through A34's distress, court-order and court-directed-sale routes (or on a vehicle A06 seeds at `exit_approved`, if A06 adds one), the node QA does the following:
  - clicks "Start burn" before the waterfall closes, and "Close register" before the burn closes;
  - double-clicks "Start waterfall" in two tabs;
  - plants A18's register total one unit under `totalSupply`.
  Expected: `WD_WATERFALL_OPEN`, `WD_BURN_NOT_CLOSED`, one run only (the idempotency key is reused and the second tab sees "Still submitting"), `WD_BURN_SUM_MISMATCH` with both figures shown as the API returned them, and no step pill reading done while its A28 or A18 state is `confirmed` but not reconciled. A screen that retries item by item or toasts success before the API's 2xx fails.
- **Distress without a published halt (G10, compliance).** ASTM escalates Bay Road to `distressed` (CCO approving), LEGL records a court order (CCO approving), and then:
  - The node QA edits the court-order request to `halt_resale: false`, which the API must refuse because the halt is mandatory for `legal_hold`; it also forces "Record court order" on an `operating` vehicle, which must render `PROP_ILLEGAL_TRANSITION`;
  - it lets the halt's ratification lapse under the injectable clock;
  - it lets `ends_at` pass.
  The per-trust header must show the halt's cause, scope, every reopening criterion and the end date from `haltState`, then a red "Unratified", then A48's `review_due` state, and never "reopened" because the date passed. A banner built from screen copy instead of the API's fields fails.
- **The attestation flag that clears itself (G19, design bug 2).** In the seed, the sponsor's `distOk`-style distribution approval for Bay Road is approved while the August attestation is `open`. Then the sponsor uploads a renewed insurance certificate for a flagged item.
  - The ADM-22 attestation pill must stay amber, and the API state must be one of `open`, `overdue`, `flagged`, `prepared` or `attested` (awaiting review), never `reviewed` and never green, until PROP's review call. A34 legitimately moves the corrected row `flagged → prepared → attested` on the sponsor's correction and signature, so the test asserts the tone and the "not yet reviewed" condition, not one literal state.
  - The ADM-05 Proof column and the public tab (through A54) must stay amber until then and turn green within one job cycle after.
  - The tenant's "New offers paused" note must lift only when every overdue attestation is reviewed.
  Any pill that changes on the distribution approval, or on the upload without the review, fails.
- **A staff agent reaching past the parity table (D04, security).** The node QA uses two `rx_stf_` tokens created in the test setup, each holding every scope its human's role allows but with a mandate limited to BAY01: one for Felix Obi (PROP) and one for Tessa Moreau (ASTM). It calls:
  - with the PROP token, obligation writes and `POST .../notices` for BAY01 (allowed, with `AgentMarker`) and for MRN01 (outside the mandate), which must be refused with the G22 reason naming the object;
  - with the PROP token, `POST /v1/attestations/{id}/review`, which must be refused because A34 marks it not delegable (A12's non-delegable refusal, `AGNT_NOT_ALLOWED` in A06's `agent-authority` block), and `POST /v1/attestations/{id}/sign`, which must return `ATT_SIGNER_NOT_HUMAN`;
  - with the ASTM token, `POST .../distress/flag` for BAY01 (allowed as maker, the request showing `AgentMarker`), then `/decide` on that same request through the SDK (`APR_CHECKER_MUST_BE_HUMAN`) and through the MCP server (tool absent from `tools/list`, `MCP_UNKNOWN_TOOL` on a forced call);
  - a sponsor-owner token asking for another tenant's property (`404`).
  Every allowed call must appear in the ADM-22 table and the lifecycle history with "agent rx_stf_… on behalf of {person}". A mutation missing from `parity.ts` fails.
- **The public tabs drift from the public page (design fidelity).** The node QA renders KST01's Summary, Details, Documents and Proof of property in the console and on A54's PUB-04 against the same seed and diffs the text content and computed styles. It checks the eight seeded rows of ADM-05: Marine Parade "in review" and not public, with the preview banner; Gympie Road with a red proof pill from its planted recon break; Unley Road with no units before mint. It checks Marion Street's ADM-06 row "due in 21 days" amber, and the ADM-22 per-trust header for BAY01, "BAY01 · Bay Road Apartment · Brighton VIC · score {n}/100", with the score from the API. Any string not from the API, any drift beyond 1 px or one token step against the prototype's lines 197-288 and extension B's frames, or any BrickX coral fails.
- **Stale screen, fresh server (correctness).** The CFO opens the committee modal for a valuation, and VALO then edits the draft's comparables from another session. The modal must reload the new hash and diff (A53's rule) and render `APR_CHALLENGE_MISMATCH` if the stale decision is forced. After publish, the ADM-05 Valued column and the Valuations tab must show the new row and the superseded one within one refresh, without a manual reload.

## Pass criteria

- **Hidden specs.** The node QA's hidden Playwright specs and component tests in `<workspace>\gauntlet\hidden\A70\` pass 100 % on three consecutive runs against the Docker stack with A06's seed, at 1280 px and 1024 px.
- **Column and field maps.** Every column of the three tables and every `KeyValueList` row maps to one named API field, recorded in `evidence/A70.json`. A generated test asserts each rendered cell equals the API's value or its `formatted` twin for every seeded row. For every seeded trust, and for MRN01 after the challenge's publish, the rendered token valuation equals A34's `nav_per_unit_cents`, which the test also recomputes independently with A02's `navPerUnit` over that trust's seeded inputs (A06's locked property value, debt and cash figures). The money-arithmetic import lint fails on the node QA's planted `divideCents` import.
- **SM-03 and the refusal codes.** A test iterates A03's 20 SM-03 states and asserts a label and a tone for each. A test iterates the A34 and A48 OpenAPI error catalogues for the codes these screens can receive and asserts a sentence for each. Every transition button is present only when the API's `allowed_transitions` lists it: a generated test over the seeded personas and states finds zero extra and zero missing buttons.
- **States.** Every state A51's `screens.json` lists for ADM-05 (default, IC review in progress, distressed, legal hold, blocked, not-eligible, empty filter), ADM-06 (default, blocked, not-eligible, older than 6 months, empty) and ADM-22 (default, overdue, distressed, legal hold, not-eligible, empty) has a screenshot and a test. Loading, empty, error (with `request_id`), forbidden, not found and halted render on all three screens. `invalid_cursor` resets the list.
- **Maker-checker.** Every privileged action in §7 creates or follows its A14 request through A53's flow. A grep finds none of A53's forbidden SDK operations (§6 G13: `approvals.create`, `.submit`, `.decide`, `.bulkDecide`, `.ratify`, `.veto`, `.cancel`, `auth.freshAssertion`) imported in the owned paths, and a static test finds every A34 or A48 request-creating operation used only as the `createVia` argument of `usePrivilegedAction`. The maker, the maker's agent and a staff token each see `not-eligible` or the `agent` state and no Approve button.
- **Journeys.** J12 steps 5-7, J13 steps 2-3, J14 steps 2, 4 and 7 (to `operating` after remediation), and J21 steps 1 and 4 (plus the waterfall and burn hand-offs) are driven through the screens by the seeded personas in Playwright, and each ends in the outcome the product map states. Each act has exactly one audit event carrying its approval id, checked through A10's `GET /v1/audit/events?approval_id=` (the ADM-31 screen is A66's and is not needed). The hand-offs to ADM-19, ADM-14, ADM-28, ADM-30 and ADM-34 are asserted by href and query parameters (ids only), not by rendering those screens.
- **Gates.** The G10 banner fields equal `haltState` for every seeded halted vehicle. The G19 attestation pill is unchanged by a distribution approval (a test proves it) and changes only on PROP's review. The G20 due-soon, stale, listing-age and comparables states each render from a seeded or sandbox-triggered case.
- **Parity (D04).** `parity.ts` lists every mutation called from the owned paths (a static test over SDK calls). Every "never" row is refused for a staff token through the SDK and REST with the §7 code; through the MCP server its tool is absent from `tools/list`, a forced `tools/call` returns `MCP_UNKNOWN_TOOL`, and a checker operation forced into a scratch catalogue is refused by the API with `APR_CHECKER_MUST_BE_HUMAN`. Every "yes" row succeeds within the mandate, is refused outside it with the G22 reason naming the object, and shows `AgentMarker` on the result.
- **Design fidelity.** The node QA's token-level comparison of the four public tabs against the prototype's lines 197-288, and of the three screens against extension B's frames, shows no drift beyond 1 px in font family, size, weight, colour, radius, border or padding, other than the recorded design-bug fixes. The console's public tabs and A54's PUB-04 render identical text for the same seed.
- **Hygiene.** Badges come from server summaries and update within one 30 s tick after an action. axe reports zero critical or serious findings on the three screens, and the forms are keyboard operable. There is no hex, `font:` shorthand or radius literal in the owned paths, and no `localStorage` use. `npm run verify` is green. Only the owned paths changed. `app/evidence/A70.json` and `A70.qa.md` exist and match disk.

## Human gates touched

- None crossed. The node runs on A11's sandbox authenticator, A06's seed, A34's `property-data` and `valuation-feed` sandboxes, and the local chain behind A28 and A18. No real valuation, title search, court order or client money is touched.
- **HG-REVIEW (before this node starts):** the founder has reviewed A51's ADM-05, ADM-06 and ADM-22 frames and A50's PUB-04 proof states in the browser. This node builds only what was reviewed. The founder also names the valuation committee's independent member at HG-REVIEW (A03 §11 (i)); until then the interim mapping of §3 applies.
- **HG-LEGAL (recorded upstream, not decided here):**
  - the valuation committee's composition and the independent member (A34);
  - the ten-day attestation pause as a sponsor-terms clause (A34);
  - the court-order and legal-hold procedure, the P1 exit resolution, the trustee's role as checker and the retention period (A48);
  - whether any act on these screens must be personal and so refused to staff agents beyond the checker rule (A12's `delegable.ts`).
  The screens render whatever those rows say. A changed row re-runs this node's parity and state tests.
- **HG-VENDOR (later, upstream):** the appointment of panel valuers and any production rent-roll mapper (PropertyMe, Yardi) are human contracts. The console is unchanged by them, because A34's adapters are the swap points.
- **HG-PENTEST (later milestone):** the console's asset screens are in scope of the external penetration test with the rest of `apps/console`. This node passes on software evidence and records the gate as open.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
