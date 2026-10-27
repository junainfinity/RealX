---
title: "A73 - Console: rule packs and threshold monitors, register and reconciliation"
created: 2026-09-24
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am7]
node: A73
milestone: AM7
lane: "Operator console"
oracle: no
human_gate: none
---

# A73 - Console: rule packs and threshold monitors, register and reconciliation

**The two console screens where RealX staff check that the rules hold and that the three records agree. ADM-16 has the rule-pack banner with its version and effective date, the threshold-monitor table (holder cap, largest holder, landholder duty, foreign percentage, member count), the per-trust rule sets and the FIRB matrix. It also has parameter changes and pack publishing, each with a simulation attached that names the holders who would be in breach, investor-specific overrides with a legal basis and an end date, and the replayable decision log with a G22 agent filter. ADM-17 is the drawn `adRecon` grid widened to the product map's columns (trustee register, token supply, bank or escrow, deltas). Around the grid sit the unknown-holders strip, the break panel (likely cause, proposed fix and the `6E` register-adjustment action), the halt state of each trust, the cap table as of any date, the register snapshots, signed extracts and force re-sync. The API decides every rule; the console only shows what the API decided. Every action exists through the API and MCP for a staff member's agent, which can prepare work but never approve it.**

**Build status:** not started. **Milestone:** AM7 (Operator console). **Lane:** Operator console. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 8-12 agent-hours (build, challenge and verify loops).

**Depends on:** [[A53 - Operator console shell, hardware-key sign-in and approval action pattern]], [[A19 - Rules engine- gates G01-G22, rule packs, threshold monitors, simulator]], [[A20 - Three-way reconciliation engine with zero tolerance]], [[A18 - Unit register (legal record) and registry sandbox adapter]], [[A51 - Design extension B- operator console screens not in the prototype]], [[A06 - Seed data and scenario simulator]], [[A48 - Halts, kill switches and wind-down]], [[A72 - Console- token operations, forced transfer and recovery desk, wallets and custody, token registry]], [[A98 - Partner principals- trustee-RE, valuer, property manager, auditor, regulator observer and licensee users]], [[A104 - MCP server generated from the OpenAPI document]]
- Why the last five: A06 supplies the seed and the scenario simulator every fixture uses; A48 supplies the halt-state read (G10 causes); A72 owns the freeze, forced-transfer and lock-up forms this node deep-links to; A98 issues the TP-RE, TP-REG and TP-AUD principals needed for the trustee's `6E` decision, certification and extracts; A104 is the MCP server the parity test calls. All are in AM7 or earlier and none depends on A73, so the edges add no cycle. The same five edges must be added to `Data\app-graph.json` by the plan owner.
**Unlocks:** [[A78 - Operator journey J20 end to end in Docker (AM7 exit)]], [[A79 - End-to-end journeys- investor money path]], [[A80 - End-to-end journeys- sponsor and operations]], [[A81 - End-to-end journeys- recovery, AML and planted breaks]]

**Oracle rule:** this is not an oracle node (`oracle: no`). The node moves no money, supply or holdings itself: every change it offers is an A14 request that A18, A19, A20 or A28 carries out. The node QA still writes hidden tests and Challenge fixtures from this frozen spec to `<workspace>\gauntlet\hidden\A73\` before the build starts ([[App Gauntlet]] §2). They are:
- Playwright specs against the running stack with the sandbox authenticator;
- its own crops of the `adRecon` region of the prototype and of the `design-ext:ADM-16` frames of extension B;
- a planted-drift scenario script for A06's simulator (register drift with no existing fix, unknown holder, one-cent bank delta, an incomplete chain source, a timed-out instruction giving a Sev2 `pending_ack`), plus the SM-07 state fixtures;
- a stale-simulation fixture and the server-side simulation-bypass cases (no simulation, another draft's simulation, unacknowledged breach list);
- a "tone computed in the browser" probe;
- a staff-token persona set for the parity checks.

The builder never reads that folder.

This note is not legal advice. The landholder-duty thresholds, the s601ED member count, the FIRB treatment of fractional units and whether a signed register extract may be issued by an agent are counsel or founder questions. The screen shows them as data marked "counsel to confirm" and never as settled rules.

## Build brief

This node draws two screens, ADM-16 and ADM-17, and their drill-downs inside `apps/console`. It adds no endpoint and no business rule. Every rule is enforced where it lives:
- A19: packs, gates, monitors, the simulator and evaluations;
- A20: runs, breaks and the recon halt;
- A18: entries, cap table, versions, extracts and instructions;
- A14: approvals;
- A48: halt records.

The console shows what those modules return and never lets a person believe an action is done before the API says so. All reads and writes go through `packages/sdk`. Every privileged action goes through A53's `PrivilegedActionFlow` and `HardwareKeyConfirm`, never through direct calls to `approvals.*`, which A53's lint rule already forbids. Every table is A53's `ConsoleTable`, every page is `ConsolePage`, and every agent-touched row carries A53's `AgentMarker`.

**1. Routes, navigation and badges (`app/(register)/`)**

- Pages. Each exports `screen` and `designKey` as A53 requires:
  - `rules/page.tsx`: ADM-16, `designKey = design-ext:ADM-16`.
  - `rules/packs/[code]/page.tsx`: pack detail and versions.
  - `rules/vehicles/[vehicleId]/page.tsx`: the per-trust rule set.
  - `rules/overrides/page.tsx`.
  - `rules/evaluations/page.tsx`: the decision log, with `rules/evaluations/[setId]/page.tsx` for one evaluation set.
  - `recon/page.tsx`: ADM-17, `designKey = design:adRecon`.
  - `recon/vehicles/[vehicleId]/page.tsx`: the per-trust drill-down, with the tabs Reconciliation, Cap table, Snapshots, Extracts, Instructions.
  - `recon/breaks/[breakId]/page.tsx`: the break panel as a full page; it also opens as a split detail from the grid.
  - `recon/runs/[runId]/page.tsx`.
  - Routes follow the product map (`/admin/rules`, `/admin/recon`) without the `/admin` prefix, because the console has its own origin. The `(register)` group is this node's. A53's reserved list does not name it; A53's nav test reads the routes from A51's `screens.json`, and this node adds the two entries there only through A51's manifest, never by editing `src/shell/nav.ts`.
- The sidebar puts both items in extension B's "Assets & money" group, labelled "Rule packs and threshold monitors" and "Register & reconciliation" (the prototype's label on line 862). They are visible only to callers with `V` on M10 or M11 in `GET /iam/me`:
  - ADM-16: CCO, LEGL and REGO, plus the read-only roles A03 grants.
  - ADM-17: REGO, CCO, CFO, SECO and CEO, plus TP-RE, TP-REG and TP-AUD once A98 issues their principals.
- Badge sources registered through `registerBadgeSource`:
  - `recon.breaks`: `GET /v1/recon/summary` → `breaks_open`. This is the prototype's "Register & reconciliation 1" badge made live.
  - `rules.monitors`: the `warning` plus `breached` count that the API returns in the `counts` field of `GET /v1/rules/monitors` (or its summary route), requested as A19.1 in §6. The browser never counts rows from the cursor-paged list. If A19.1 is declined, this badge source is not registered and the gap is recorded in `A73.qa.md`.
  - Both show nothing, not `0`, when empty.
- Search providers registered through `registerSearchProvider`:
  - `BRK-` and `RUN-` ids route to the break and run pages.
  - `RGX-` routes to the extract.
  - Trust codes route to the recon drill-down.
  - Free text matches trust names only. Investor names are never searched here and never go into a URL.

**2. ADM-16 Rule packs and threshold monitors (M10)**

- **Pack banner.** It comes from `GET /v1/rules/rule-packs`. It shows the effective pack's `code`, display name, `effective_from`, `state`, `params_sha256` (Geist Mono, truncated by A05's one rule) and the count of vehicles whose `ComplianceRule` uses it. The P2 stub `AU-R-2027.1` shows with the brick "2027" pill and `state = draft`. A pack in `approved_pending_effect` shows "effective {date}" in amber. The banner never shows a version the API did not return.
  - A51's mock string "AU-wholesale v3 · published 1 Aug 2026" is replaced by the API's code and dates. The difference is recorded in `A73.qa.md` as a reconciled design difference, not a defect.
- **Threshold-monitor table** (`GET /v1/rules/monitors`, one row per vehicle, cursor-paged). Columns as extension B draws them: Trust / Holder cap / Largest / Landholder duty / Foreign % / Members / State.
  - Each cell prints the monitor's `current` against its `warn_at` and `block_at` in basis points through A05's percentage formatter, for example "4.8% of 5%".
  - The tone comes only from the monitor's `state`: `ok` green, `warning` amber, `breached` red, and brick for a parameter the API marks `counsel_pending`. The browser never works out a tone from the numbers. Extension B's "amber at 80 percent" is a mock value; the seeded pack warns at `holder_cap.warn_at_pct_of_cap = 90`, and the screen follows the API.
  - Landholder duty shows the state threshold (for example "NSW 20%") with a "counsel to confirm" brick pill while the parameter is `counsel_pending`. Members shows the distinct-person count against `member_count.s601ED_trigger`, with the same pill.
  - A vehicle whose `foreign_policy` is `after_firb_check` shows the brick "FIRB check" pill, as the product-map mock does for PAR01.
  - Each row carries `computed_from` (the register version) and `computed_at` on hover. A row flagged `SOURCE_DIVERGENT` shows an amber "register and chain differ · see reconciliation" link to the ADM-17 drill-down.
  - Filters: state (`ok`, `warning`, `breached`), metric, tenant. The page has four states: default, near-threshold, blocked, and empty ("No trusts issued yet").
- **Per-trust rule set** (`GET /v1/rules/vehicles/{id}/compliance-rule`). This is a `KeyValueList` of:
  - country and residency policy, holder cap, lock-in days, eligibility claim topics, `foreign_policy`, `property_type`, `zone_flags`, `member_cap`;
  - the resolved `params_sha256` beside the on-chain hash from A20's last run's R6 result, as a green "matches chain" pill or a red "rule drift (R6)" pill with a link to the break.
  - The FIRB eligibility matrix is a read-only table by property type, taken from the pack parameters (established residential → residents only until `firb.established_residential_ban_until`; build-to-rent, student, aged care, 20+ dwelling redevelopments and commercial → FIRB check). It uses the design's G06 wording ("Not permitted (established residential)", "Permitted after FIRB checks").
- **Simulate.** An outline button, as extension B draws it. It calls `POST /v1/rules/simulate` with the proposed change and renders the result as a `KeyValueList`:
  - verdicts before and after;
  - monitor states before and after;
  - the count of pending orders that would fail;
  - the list of holders who would be in breach, each with a masked investor id through `MaskedCell`, current units and units over the limit;
  - the register version it ran on and its result hash.
  - Simulation writes nothing, and the screen says so ("Simulation only · nothing changed").
  - A run over the 5-second budget returns A19's timeout code, which is rendered with the request id.
- **Change rule parameter.** This uses `PrivilegedActionFlow` with the action code from A03's catalogue for a per-vehicle parameter change (policy `4E+Q`, CCO → LEGL, "simulation attached"). The flow works like this:
  - The draft form edits one parameter of one vehicle's `ComplianceRule` (`PUT /v1/rules/vehicles/{id}/compliance-rule` as a draft).
  - The flow runs the simulation for exactly that draft and puts its result hash into the payload. Submit stays disabled until a simulation exists for the current draft hash.
  - If the simulation lists any holder who would be in breach, the flow shows that list before Submit, as the product-map state requires. The maker must tick an `AcknowledgeCard` naming the count; the tick is sent as an acknowledged warning, and the checker sees the same list.
  - Any edit after the simulation throws the simulation away and changes the payload hash visibly.
  - Prerequisite checks from A14's precheck ("Trust deed allows change", "Licensee notified", "Holders affected") show as `GateRow`s. A `Pending` prerequisite leaves the request in extension B's blocked state, and the console renders whatever A14's precheck decides; it never pre-filters. The seeded example is APR-2285 (A06): holder cap 5% → 7.5% on Ann Street, maker Jordan Blake (CCO), checker Sam R. (LEGL), already submitted and pending in the inbox with its simulation attached, and "Licensee notified" Pending, so A14 refuses its approval on the prerequisite.
  - The server, not only the screen, enforces the simulation rule. A request for a parameter change, pack version, override or threshold block that has no simulation, carries the result hash of a different draft, or whose breach list was not acknowledged (acknowledged count different from the simulation's breach count) is refused by A19 and A14 with `SIMULATION_MISSING`, `SIMULATION_DRAFT_MISMATCH` or `BREACH_LIST_NOT_ACKNOWLEDGED`. These codes are requested from A19 and A14 in §6; the console shows each code with its sentence.
  - At approve time A19 and A14 refuse a simulation older than the latest register version with `SIMULATION_STALE`. The console shows the code and offers "Simulate again", which starts a new draft. It never re-attaches an old result.
  - The `+Q` suffix means a custody signature follows approval. The status card follows SM-00 to `confirmed` and words it "confirmed · on-chain parameters pending reconciliation (R6)", never as done.
- **Publish pack version.** The pack detail page drafts a new pack (`POST /v1/rules/rule-packs`, then `POST .../{code}/submit`), which A14 resolves to `6E` (CCO → LEGL, "notifies every sponsor using it").
  - The flow shows the parameter diff against the effective pack key by key, the simulation over every vehicle that uses the pack, and `effective_from`.
  - A date earlier than the approval time is refused by A19. Its code is shown; the console does not pre-filter it.
  - An effective or superseded pack has no edit control at all, because it is immutable.
- **Turn on a pre-trade threshold block.** A per-monitor switch on the rule-set page drafts a compliance-rule change that sets the monitor's `block_at` (the product map's CCO, `4E`).
  - While the parameter is `counsel_pending`, the switch shows A19's rule that the block is downgraded to a warning (`COUNSEL_PENDING_<KEY>`). The console does not hide the switch, because the API decides.
  - If A19's `ComplianceRule` has no per-metric block field, this is a spec finding for A19 (see §6), not a field invented here.
- **Set lock-up dates.** A product-map row on ADM-16 (REGO → CCO, `4E+Q`, a partial freeze with a release date). The row is listed on the rule-set page and opens A72's token-operation form pre-filled through the deep-link contract requested from A72 in §6 (`op=lock_up`, vehicle, wallet and lot ids only). The on-chain operation and the G08 lock-in enforcement belong to A28 and A72, and this node does not duplicate them.
- **Investor-specific overrides** (`rules/overrides`). A table built from A19's `InvestorOverride` rows, with columns: investor (masked id), trust, gate, parameter, value, legal basis (truncated, full on open), evidence documents (`DocumentRow`s from A35's library), starts, ends, approval id, state.
  - An override past `ends_at` shows grey "ended", from the API's `state`.
  - "New override" opens `PrivilegedActionFlow` with the fields A19 requires: legal basis and evidence mandatory, `ends_at` mandatory and at most 24 months ahead. The policy is whatever A14 resolves: A19 names `6E` CCO + LEGL, while the product map names CCO → LEGL. The console renders the resolved row and hard-codes neither.
  - There is no edit and no delete: a change is a new override that supersedes the old one.
- **Decision log** (`rules/evaluations`). Every allow and deny from A19's `GateEvaluation` rows, newest first, cursor-paged.
  - Columns: time, evaluation set id, gate, action kind, object, actor (with `AgentMarker` when `actor_type = AGNT`), on behalf of, result (tone from the verdict), reason codes, rule-pack code, overrides used.
  - Filters: gate, result, action kind, vehicle, actor type, token id and on-behalf-of person id, all as ids in the URL.
  - One preset chip, "Agent authority (G22)", sets `gate = G22` together with `actor_type = AGNT`, so a CCO can see every agent refusal in one view. Each G22 row shows the missing scope or cap and the remaining amount that A12's verdict carries, and links to ADM-35 (A77) for callers with `V` on M26 (Integrations and API, the module that owns ADM-35 in the product map and that A77 gates its oversight pages on).
  - Opening a row shows the whole evaluation set, including every gate, the fix-it text, the facts used by name (never values that are PII) and the pack hash. It also offers "Replay" (`POST /v1/rules/evaluations/{setId}/replay`), which shows "Replayed · identical" in green or the Sev1 `replay_mismatch` in red.

**3. ADM-17 Register and reconciliation (M11)**

- **Header and KPIs** exactly as `adRecon` draws them, with each value from `GET /v1/recon/summary`:
  - subtitle "Trustee register, token supply and client-money ledger compared daily. The register is the legal record.";
  - "Trusts matched {trusts_matched} of {trusts_total}", with the detail "Run {last_cycle.started_at as HH:mm}" (A20 names `started_at` as the source of "Run 06:00"). The KPI prints A20's `trusts_matched` unchanged. A20's formula counts `not_issued` vehicles whose legs agree as matched, and A06 seeds Marine Parade (MPA01) that way, so the seed reads "7 of 8" while only six Status cells are green (Marine Parade shows grey "Not issued", Gympie Road red). This is correct and is not a defect: the UI never recounts grid rows to produce or check the KPI, and a test asserts the KPI text equals the API fields while the grid shows a different number of green cells. A78's "8 of 8" after the fix follows from the same formula;
  - "Breaks open {breaks_open}", with "{breaks_open_units} units";
  - "Oldest break {oldest_break_age}", with "SLA {sla}".
  - A fourth tile "Unknown holders {n}" (this node's addition from the product map's check strip) is shown only when it is above zero.
  - The seed reproduces "7 of 8 · Run 06:00", "1 · 2 units" and "4h · SLA 24h".
  - A cycle still running shows "Run in progress · started HH:mm". A cycle that is `incomplete` shows amber "Incomplete · {missing_source}" and is never counted as matched.
- **Recon grid** (`GET /v1/recon/vehicles`). The design's columns and the product map's merged into one `ConsoleTable`, in this order:
  - Trust.
  - Register (legal record), `register_units`.
  - Token supply, `chain_supply`.
  - Δ units, register minus chain, signed, Geist Mono, red when not zero.
  - Bank / escrow: `client_money_result` as "Matched" or "Break"; for an open offer, the label "Applications (escrow) {escrow_application_units}".
  - Δ cents: `bank_delta_cents` through `formatAud`; A$0.01 is red.
  - Last run.
  - Halt.
  - Status.
- Two design bugs are fixed, not reproduced: every trust is listed, Marine Parade included as "Not issued" in grey (design analysis §6.4 item 13), and escrow figures never appear in the register column (item 14). Kent Street reads register `0`, token supply `0` and "Applications (escrow) 6,400".
- The Status cell is a fixed lookup on the API's `result`, `matched` and `open_breaks[0]` fields, never arithmetic: "Matched" in green (`clean`, or `not_issued` with an open offer's escrow leg, as drawn for Kent Street, Ann Street and Unley Road), "Break · {|delta_units|} units" in alert red `#A3312A` from `open_breaks[0].delta_units` (A20 names this composition), "Break · A${delta}" for a cash break, "Not issued" in grey (`not_issued` with no escrow leg, Marine Parade), or "Incomplete" in amber. Gympie Road Retail reads "9,998 / 10,000 / Break · 2 units", as drawn.
- **Sev2 breaks.** A20 halts resale only for an open Sev1 break; a Sev2 break (for example R17 `pending_ack` before its SLA, or `source_unavailable`) instead refuses `allocation`, `escrow_release`, `distribution_approval`, `distribution_execution` and `board_open`, and escalates to Sev1 at 24 h. So the console never shows "no restriction" while actions are refused: for a Sev2 break the row, the drill-down header and the break-panel banner show amber "Pre-flight blocked: {purposes}" (for example "Pre-flight blocked: distributions, board opening") from the blocked-purpose list the API returns (requested from A20 in §6 as `blocked_purposes[]` on the vehicle row and the break). The browser never derives the list from the severity. The product map's wording that any break halts resale is A20's recorded reconciled difference, and this node shows A20's rule.
- Filters: result, halted, tenant. A row opens the split detail; Enter or a click opens the drill-down.
- **Unknown-holders strip.** A check strip above the grid, from open breaks of type `unknown_holder`. It shows one chip per trust with the truncated wallet (`truncateAddress`), the on-chain balance and "not on the register · Sev1".
  - When the count is zero it reads "Unknown holders on-chain: 0" in green and stays visible. It is never hidden, because the product map says any non-zero is Sev1 and a missing strip would look like zero.
  - Each chip links to the break and offers two deep links: "Freeze wallet (SGL+R)" opens A72's freeze form pre-filled through the §6 deep-link contract (`op=freeze`, vehicle id, wallet id and the `BRK-` reference only), and "Fetch a fresh extract" is the A20 resync below.
- **Break panel.** A split detail from the grid, or `recon/breaks/[id]`. It is built from A20's `ReconBreak` fields:
  - id, rule, type, severity, subject;
  - a three-value card: Register "INV-… holds {n}", Chain "{0x…} holds {n}", Bank or ledger where relevant (the product-map mock's `card` block);
  - signed deltas;
  - `likely_cause` and `proposed_fix` as the API's text from A20's data table;
  - owner role, state (`open`, `investigating`, `fix_pending`, `escalated`), opened at, last seen run;
  - the A13 alert id and the 24 h clock as a mono countdown;
  - `ApprovalsChain` for `fix_approval_id` from A14's detail (maker done, CCO pending, CFO next, trustee; the product-map `appr` block);
  - the halt banner "Resale for {trust} halted until resolved · reopens after a completed run with zero open Sev1 breaks" (A20's fixed criterion).
- Actions on the break panel, each shown only when the API lists it as allowed for the caller:
  - **Investigate** (`POST /v1/recon/breaks/{id}/investigate`, assign to self).
  - **Propose register adjustment (6E).** `PrivilegedActionFlow` with action code `register.adjust`: REGO is the maker; CCO, CFO and the vehicle's TP-RE or TP-REG are the checkers. The payload comes from the break: holder, signed unit delta, lots, `reference = recon_break_id`. The flow shows the register impact before and after, from `GET /register/{vehicleId}/cap-table?as_of=` and the proposed delta, and says "Any unit change needs CCO, CFO and the trustee". The order is fixed by A20's contract (former spec point (a), now settled): the flow first calls `POST /v1/recon/breaks/{id}/propose-fix` with the reason and payload; A20 and A14 create the `register.adjust` `6E` request server-side, store its approval id on the break and move the break to `fix_pending`; the console then opens that request in A53's `PrivilegedActionFlow` for the maker to review and submit with `HardwareKeyConfirm`. The console never creates the approval request itself.
  - A break that is already `fix_pending` shows its linked request (in the seed, the Gympie Road break with APR-2292, maker Grace L., checkers CCO, CFO and the trustee) through `ApprovalsChain`, and offers no second proposal.
  - **Submit adjustment to the registry.** This appears only once the request is `approved`. It prepares and submits A18's `adjust` instruction with that `approval_id` (`POST /register/{vehicleId}/instructions`, then `/{id}/submit`). A18 refuses anything weaker with `REG_ADJUST_REQUIRES_6E`, `REG_ADJUST_TRUSTEE_MISSING` or `REG_APPROVAL_MISMATCH`, and each code is shown with its sentence.
  - **Propose corrective transfer.** For breaks where the chain is wrong and the register is right, a deep link opens A72's forced-transfer form pre-filled through the §6 deep-link contract (`op=forced_transfer`, vehicle, wallet and lot ids and the `BRK-` reference only; `6E+Q`, 48 h timelock). This node does not own the on-chain operation.
- **No close button exists.** Neither A20 nor this node offers close, suppress, snooze, dismiss or delete on a break. The panel says "Closes automatically when a later run no longer finds it". After an approved adjustment is acknowledged, the panel shows "Adjustment acknowledged · waiting for the targeted run" until A20's run closes the break. Approval alone is never shown as resolved.
- **Runs.** "Re-run" (`POST /v1/recon/runs`, REGO or CCO; the product-map mock's button) takes optional explicit keys (register version, block, statement) under an "Advanced" disclosure.
  - `409 run_in_progress` shows the running run's id and a link.
  - A token that goes over A20's limit of 10 runs per vehicle per hour gets `429`, and the screen shows it with `Retry-After`.
  - The run page lists R1-R18 with pass, break, not applicable, or "not in this phase" in grey for `skipped_stub` (R9, R15, R16). It also shows the three keys, `result_sha256`, the duration and "Replay" (`POST /v1/recon/runs/{id}/replay`: "reproduced" or `replay_mismatch` Sev1).
- **Halt state per trust.** The Halt column and the drill-down header read A48's `GET /v1/halts/vehicles/{id}/state`, together with A20's `halted` flag and `GET /v1/recon/halts`.
  - A red "Resale halted" pill shows each cause (recon break, manual, meeting, valuation stale, token paused, distressed, legal hold), the time since, the published reopening criteria and `ends_at`.
  - A halt with only the A20 source still reads halted, following A48's union rule.
  - The console never offers to clear a halt caused by a recon break. Manual halts link to ADM-34 (A69's screen over A48's routes), where clearing lives.
  - A48 is a declared dependency, and every halt test (the G10 criteria, the tone-mutation test, the planted-drift run) reads A48's `GET /v1/halts/vehicles/{id}/state` against the running A48 module. A build cannot pass without it.
  - Runtime degradation only: if A48's read fails at run time (for example a timeout), the column falls back to A20's `halted` flag and the recon cause alone, with the grey note "other halt causes unavailable". A test forces that failure and asserts the note; this fallback never counts as meeting the G10 criteria.
- **Cap table as of a date** (drill-down tab). `GET /register/{vehicleId}/cap-table?as_of=` with a date picker (AEST dates, ids only in the URL).
  - Columns: holder (masked through `MaskedCell`, unmasked only by the owning module's reason-coded route), holder type, classification, residency, units, `pct_bps` as a percentage with two decimals, locked units, frozen units, wallet (truncated display field), status.
  - Totals: `holder_count`, `total_units`, `largest_holding_bps`, `foreign_units_bps`, and the last certified version with its date.
  - The percentages add up to exactly 100.00%, because A18 assigns the remainder, and the table prints the API's values without re-rounding.
  - An "As the register knew it at…" field sends `record_time` for a bitemporal read of `GET /register/{vehicleId}/entries`. After a back-dated adjustment, an auditor can see both the corrected past and the past as it was recorded.
  - Export (`POST /register/{vehicleId}/exports`, purpose `audit`) shows the extract id and hash. `REG_EXPORT_RATE_LIMIT` shows its code, and every export produces one A18 audit event.
- **Snapshots** (drill-down tab). `GET /register/{vehicleId}/versions` with source (`internal` 00:00, `registry` 02:10, `certification`), `as_of`, `file_sha256`, row count, total units, holder count, state and certified by.
  - "Download" streams `/versions/{id}/file`; A18 writes `register.version.downloaded`.
  - "Request monthly certification" is a `PrivilegedActionFlow` with `register.certify` (`4E`: REGO → TP-RE or TP-REG) over `POST /register/{vehicleId}/versions/{id}/certify`. `REG_CERTIFICATION_HASH_MISMATCH` shows its code.
  - A certified version shows green "Certified · {date} · {approval id}". Superseded versions are grey and never editable.
- **Signed extracts** (drill-down tab). "Issue register extract" (REGO, TP-RE, TP-REG, TP-AUD; single actor; `POST /register/{vehicleId}/extracts` with `as_of`, `record_time`, `purpose`, `format`).
  - The result card shows the extract id, `file_sha256`, `signing_key_id`, and three downloads (csv, json, pdf).
  - "Verify" (`GET /register/extracts/{id}/verify`) shows green "Signature valid · hash matches" or red.
  - "Check a file" (`POST /register/extracts/verify`) takes an uploaded file and answers `matches: true | false` with the original extract id. The file goes straight to the API and is never parsed in the browser.
  - The list of past extracts needs a list route that A18 does not yet name (§6).
- **Force re-sync with the registry** (drill-down, REGO). Two actions, named plainly so nobody mistakes one for the other:
  - **"Fetch a fresh extract and re-run"** is A20's `POST /v1/recon/vehicles/{id}/resync`, runbook R4. REGO acts alone and it is audited. It asks A18 for an extract and queues a targeted run.
  - **"Force re-sync (diff against the registry)"** is `PrivilegedActionFlow` with `register.resync` (`4E`, REGO → the head-of-operations checker set that A14 resolves) over `POST /register/{vehicleId}/resync`. It returns a diff table (holder, register units, registry units, delta) and changes nothing. Its only follow-on action is "Propose adjustment (6E)" per line.
  - The product map describes the re-sync as "re-submits instructions" (runbook R4). A18's approved re-sync effector does exactly that first: it re-submits every `submitted` or `timed_out` instruction for the vehicle under its original idempotency key, then returns any remaining difference as the diff. The result card therefore lists the re-submitted instruction ids before the diff table. It is not a design difference.
  - While an instruction is pending past A18's SLA, or the registry adapter reports down, the drill-down shows the red banner "Registry sync failure · no on-chain moves without a registry acknowledgement · runbook R4". The banner comes from R17's `pending_ack` break and the adapter health that A20 reports.
- **Instructions** (drill-down tab). Pending and recent `RegisterInstruction`s with kind, lines count, reference, approval id, state, `submit_count`, `submitted_at` and ack age. Instruction preparation other than `adjust` belongs to A28, A37 and A91.
  - **Re-submit timed-out instruction.** On a `timed_out` row, and only when the API lists the action as allowed for the caller, a "Re-submit" button calls A18's `POST .../instructions/{id}/resubmit` (REGO, as A18 names it). It moves the same row back to `submitted` under the same idempotency key, with no new lines and no new approval, so it can never apply twice. `REG_ILLEGAL_TRANSITION` and any other A18 refusal is shown with its code. This is the single-instruction form of runbook R4; the vehicle-wide form is the `4E` force re-sync above.

**4. Agent and API parity for what this node exposes (D04, G22)**

This node adds no endpoint. Every action above is an existing A14, A18, A19, A20 or A48 route that the SDK, the REST API and the MCP server (A104) expose unchanged. Parity is proved by listing each action with its agent rule, exported as `src/features/rules-recon/parity.ts` for A100's agent-parity suite.

| Screen action | API route (owner) | Staff agent (`rx_stf_` token, within its human's role and mandate) |
|---|---|---|
| Read packs, monitors, rule sets, overrides, evaluation sets | `GET /v1/rules/*` (A19) | yes, M10 read scope; same filters and pagination |
| Simulate a change | `POST /v1/rules/simulate` (A19) | yes; writes nothing; a G22 row records the call |
| Replay an evaluation set | `POST /v1/rules/evaluations/{id}/replay` (A19) | yes, M10 read |
| Draft and submit a parameter change, pack version, override, threshold block | `PUT .../compliance-rule`, `POST rule-packs`, `POST .../submit`, `POST investors/{id}/overrides`, then `POST /v1/approvals` and `/submit` (A19, A14) | yes, as maker only; the request carries `AgentMarker`; the simulation must be attached exactly as for a person |
| Submit one of the above with no simulation, a simulation of another draft, or an unacknowledged breach list | same routes (A19, A14) | nobody, human or agent: `SIMULATION_MISSING`, `SIMULATION_DRAFT_MISMATCH`, `BREACH_LIST_NOT_ACKNOWLEDGED` (§6) |
| Approve, reject, ratify any of the above | `/v1/approvals/{id}/decide` (A14) | never: `APR_CHECKER_MUST_BE_HUMAN` before scope evaluation; SoD counts the agent as its human |
| Read recon summary, grid, runs, breaks, halts | `GET /v1/recon/*`, `GET /v1/halts/vehicles/{id}/state` (A20, A48) | yes, M11 read |
| Manual re-run, replay a run | `POST /v1/recon/runs`, `/runs/{id}/replay` (A20) | yes, M11 `recon:run`; 10 runs per vehicle per hour for tokens |
| Investigate a break; propose a fix | `POST /v1/recon/breaks/{id}/investigate`, `/propose-fix` (A20) | yes, as maker; the `6E` request it links to is refused to every token by A14 |
| Close, suppress or snooze a break | none exists | nobody, human or agent |
| Submit the approved adjustment instruction | `POST /register/{id}/instructions`, `/submit` (A18) | yes, `register:instruct`, only with an approved `6E` approval id |
| Re-submit a timed-out instruction | `POST /register/{id}/instructions/{id}/resubmit` (A18) | per A03's matrix cell for the resubmit action; same row, same idempotency key, no new approval; the test asserts the API's answer matches `delegable.ts` |
| Fetch a fresh extract and re-run (R4) | `POST /v1/recon/vehicles/{id}/resync` (A20) | yes, `recon:resync`, audited on behalf of the person |
| Force re-sync diff | `POST /register/{id}/resync` via A14 `4E` (A18) | draft as maker only |
| Cap table, entries, versions, version download | `GET /register/{id}/cap-table`, `/entries`, `/versions`, `/versions/{id}/file` (A18) | yes, `register:read`; download audited "agent … on behalf of …" |
| Audit export | `POST /register/{id}/exports` (A18) | yes, `register:export`; the same per-principal rate limit |
| Request monthly certification | `POST /register/{id}/versions/{id}/certify` via A14 `4E` (A18) | draft only (`register:certify.request`); the trustee decision is human |
| Issue a signed register extract | `POST /register/{id}/extracts` (A18) | per A03's `delegable.ts`; proposed `false` (spec point (b)); refused with `AGNT_NOT_ALLOWED` until decided |
| Verify an extract | `GET /register/extracts/{id}/verify`, `POST /register/extracts/verify` (A18) | yes, read |
| Unmask a holder name | the owning module's unmask route | per `delegable.ts`; `"counsel"` behaves as non-delegable |

- Routes in this table are abbreviated; `parity.ts` and the SDK use the owning module's exact paths (for example A18's `/v1/register/vehicles/{vehicleId}/…`).
- A static test asserts that every SDK mutation called from `src/features/rules-recon/` and `app/(register)/` appears in `parity.ts`. A Docker test calls every "never", "nobody" and "draft only" row with a seeded REGO and CCO staff token through the SDK and the MCP server (A104) and asserts the refusal code. The "nobody" simulation row is also called with a human session.
- A staff agent can reach nothing through these screens that its human could not reach through the same routes. The console offers no path that is not in this table.

**5. States, errors and privacy**

- Every page has A53's four shell states (loading, forbidden, not found, session expired) and table-level empty states with this node's copy:
  - "No trusts issued yet" on the recon grid, where rows still show as "Not issued" when vehicles exist;
  - "No breaks open" in the break list;
  - "Unknown holders on-chain: 0";
  - "No overrides in force";
  - "No evaluations match these filters";
  - "No snapshots yet · the first runs at 00:00".
- Every refusal code from A14, A18, A19, A20 and A48 that these actions can return is shown as the design's inline `Banner` with the code and a sentence. They include `SIMULATION_STALE`, `RULE_PACK_MISSING`, `run_in_progress`, `recon_not_clear`, `REG_*`, `APR_*`, `FRESH_ASSERTION_*`, `AGNT_NOT_ALLOWED`, `G22_*`, `RATE_LIMITED` and `invalid_cursor`. A generated test iterates those modules' OpenAPI error catalogues and asserts that a sentence exists for each.
- Amounts are integer cents through `formatAud`, units are integers through A05's unit formatter, and percentages are basis points through the percentage formatter. The browser does no floating-point arithmetic on money, units or percentages, and it computes no delta, tone, total or count that the API does not return. A grep in `npm run verify` fails on `parseFloat`, `toFixed` or arithmetic operators applied to fields named `*_units`, `*_cents` or `*_bps` in the feature folder, apart from the display formatters.
- PII never appears in URLs, document titles, `data-*` attributes or `localStorage`. Holder names on the cap table and break panel are masked until a reason-coded unmask, and investor ids render in Geist Mono. The evaluation log shows fact names, never fact values, because A19 stores PII as record ids.

**6. Upstream contract requests (recorded, not built here)**

These are read routes, filters, refusal codes and a deep-link contract the screens need and the owning nodes do not yet name. Per the plan's rule they go to the owners as follow-up loops before this node's build starts, never as endpoints written here:
- **A19.1:** `GET /v1/rules/evaluations?gate=&result=&action_kind=&vehicle=&actor_type=&token_id=&on_behalf_of=&since=&cursor=`, the decision log. A19 today exposes only `GET evaluations/{setId}`.
- **A19.1:** `GET /v1/rules/overrides?vehicle=&state=&cursor=`, the cross-investor override list.
- **A19.1:** confirmation that `ComplianceRule` carries a per-metric block switch for "Turn on a pre-trade threshold block".
- **A19.1:** a `counts` field (`ok`, `warning`, `breached`) on `GET /v1/rules/monitors`, or a monitors summary route, for the `rules.monitors` badge.
- **A19.1 and A14 (follow-up loop):** server-side refusals for the simulation rule on every parameter-change, pack, override and threshold-block request, whoever submits it (UI, SDK or MCP, human or token): `SIMULATION_MISSING` (no simulation result hash in the payload), `SIMULATION_DRAFT_MISMATCH` (the hash belongs to a different draft than the payload) and `BREACH_LIST_NOT_ACKNOWLEDGED` (the acknowledged count differs from the simulation's breach count). Today A19 and A14 define only `SIMULATION_STALE` at approve time. This is a contract change to A19 and A14, not a read route; if declined, it escalates as a spec defect (§ Gauntlet loop), because the rule would then be proved only in the browser.
- **A20.1:** `unknown_holders_open` in `GET /v1/recon/summary`, for the check strip (the `type` filter on `GET /v1/recon/breaks` already exists in A20).
- **A20.1:** `blocked_purposes[]` (from A20's blocking matrix) on each `GET /v1/recon/vehicles` row and on `GET /v1/recon/breaks/{id}`, for the Sev2 "Pre-flight blocked" display.
- **A18.1:** `GET /register/{vehicleId}/extracts?cursor=`, the extract list.
- **A72 (deep-link contract):** A72's `tokens/[vehicleId]` page accepts id-only query parameters that open its new-operation form pre-filled: `op=freeze|forced_transfer|lock_up`, `wallet=<wallet id>`, `lot=<lot id>` and `ref=<BRK- id>`. No names, amounts or addresses go in the URL; A72 re-reads everything through its own routes and runs its own prechecks (including G08 for lock-ups). If A72 does not accept these parameters, the links open A72's page for the vehicle without pre-fill and the gap is recorded in `A73.qa.md`.

If an owner declines a request, the affected panel shows extension B's grey "not available" state and the gap is recorded in `A73.qa.md`. The screen never works around it by fetching everything and filtering in the browser.

## Owned paths (other nodes do not edit these)

- <workspace>\app\apps\console\app\(register)\
- <workspace>\app\apps\console\src\features\rules-recon\

Conventions inside the owned paths:
- `app/(register)/` holds only thin route files: `rules/…` and `recon/…` as in §1, each exporting `screen` and `designKey`.
- `src/features/rules-recon/` holds `monitors/`, `packs/`, `rule-sets/`, `overrides/`, `evaluations/`, `recon-grid/`, `breaks/`, `runs/`, `halts/`, `cap-table/`, `snapshots/`, `extracts/`, `resync/`, `badges.ts`, `search.ts`, `parity.ts`, `copy.ts` (every sentence and refusal text of these screens), `fixtures/` (screen contract fixtures from A06's seed), and `index.ts`.

The shell, its primitives and the shared blocks are A53's. Components and tokens are A05's. The extension file is A51's. The API modules are A18, A19, A20 and A48's. A visual this node needs that neither A05, A53's blocks nor extension B defines goes back to A51 as a finding and is never drawn here in raw hex; A05's token lint runs over the folder.

## Design references

- **`design:adRecon`** in `<workspace>\design\RealX App.dc.html`: screen key `adRecon`, product-map ADM-17, admin dark shell. The page data is on line 885. The admin page frame is at lines 630-640: mono code eyebrow `ADM-17`, Newsreader 40 h1, subtitle in `#45484D` 15 px, and the KpiGrid (`repeat(auto-fit,minmax(200px,1fr))`, 1 px `#E3DED4` gaps, Geist Mono 500 24 values). The table block is at lines 719-729: white card, radius 12, `min-width:760px`, Geist Mono 11 uppercase headers in `#5E6166`, 13 px cells with `#ECE8E0` row rules, a status pill in the last column through `pl()` on line 742.
  - The subtitle "Trustee register, token supply and client-money ledger compared daily. The register is the legal record." must match exactly. The KPIs are "Trusts matched 7 of 8 · Run 06:00", "Breaks open 1 · 2 units" and "Oldest break 4h · SLA 24h". The rows are Kent Street Terrace, Marion Street Warehouse, Bay Road Apartment, Ann Street Office Floor, Hawthorn Grove Townhouse, Gympie Road Retail ("9,998 / 10,000 / Break · 2 units" in red) and Unley Road Cottage.
  - Deliberate departures, each recorded as a reconciled difference: the eighth trust, Marine Parade, is added as "Not issued" (design analysis §6.4 item 13); escrow moves out of the Register column into "Applications (escrow)" (item 14, the product map and A18's "units exist only after an acknowledged issue"); the product map's Δ and bank/escrow columns are added (product-map ADM-17 mock "Three-way reconciliation · 23 Sep 02:00", columns Chain supply / Trustee register / Bank / escrow / Δ / Status); and A51's note on `adRecon` that a break halts resale becomes the Halt column.
  - The "06:00" strings come from the run's real timestamp; the seed pins its as-of run at 06:00 AEST, while engineering runs the nightly job at 02:00 as A20 fixes.
  - Fidelity target: token-level match of the header, KPI grid and table at 1280 px.
- **`design:adCmd` register-break alert** (ADM-01, line 880): the row "Register break · Gympie Road Retail / Registry file shows 9,998 units; token supply 10,000" deep-links to `adRecon`. That link lands on this node's grid with the Gympie Road break panel open (`/recon?break=BRK-…`, an id only).
- **Nav.** `adNavDef` on line 862: the "Register & reconciliation" item with badge `'1'` becomes the live `recon.breaks` badge (design bug §6.4-8: static badges are not reproduced).
- **`design-ext:ADM-16`** from [[A51 - Design extension B- operator console screens not in the prototype]] (`<workspace>\app\design-ext\admin\RealX Admin Ext.dc.html`, `screens.json`, `blocks.md`):
  - the pack banner, with the P2 pack under the brick 2027 pill;
  - the DataTable Trust / Holder cap / Largest / Landholder duty ("20 percent NSW private unit trust · counsel to confirm") / Foreign percent / Members ("s601ED · counsel to confirm") / State;
  - the "Simulate" outline button opening a `KeyValueList`;
  - "Change parameter" (CCO → LEGL, `4E+Q`, simulation attached) through the modal;
  - APR-2285 drawn as the blocked state;
  - the default, near-threshold, blocked and empty states.
  - The product-map ADM-16 mock (desk frame "Threshold monitors", rows PAD01 "near cap" amber, SBK02 and WLG01 ok, PAR01 "FIRB check" brick; the banner "Pack AU-wholesale v3 effective 1 Oct 2026 · AU-MIS-retail v1 draft (P2)") is the content check. A06's seed and A19's pack supply the values.
- **ADM-17 panels that neither the prototype nor extension B draws** (unknown-holders strip, break panel, cap table, snapshots, extracts, re-sync diff): A51 draws only a note on `adRecon`. These panels are composed from existing patterns without new visuals:
  - `adRecon`'s DataTable;
  - `adToken`'s TokenOpRow card for the three-value break card (block 40, `id · approval id` in Geist Mono);
  - `ApprovalsChain` and `KillSwitchBanner` from `blocks.md` (built by A53);
  - A05's `Banner`, `KeyValueList`, `DocumentRow`, `StatusPill` and `DropBox` (for "Check a file").
  - The layout follows the product-map ADM-17 mock: the grid, then a two-column row with the break card on the left and "Proposed fix: register +12" with the approvals chain and the banner "Resale for WLG01 halted until resolved." on the right. If A51's extension file contains frames for these panels when this node starts, those frames win. Any need beyond these blocks becomes a DF finding for A51.
- **Tokens:** cream `#F5F2EC`, ink `#17191C`, green `#1F5E4B`, brick `#9A4524`, alert red `#A3312A`, amber from A05's `toneFor`, and the dark shell surfaces from A53. Newsreader for titles, Geist for text, Geist Mono for units, deltas, hashes, ids and times. Tone mapping follows A19 and A20: `pass`/`ok`/`matched` green, `warn`/`warning` amber, `fail`/`breached`/`break` red, `not_applicable`/`not_issued`/`skipped_stub` grey, `counsel_pending` and sandbox brick.

## Requirements covered

- **ADM-16 Rule packs and threshold monitors.** Done when:
  - every product-map component is on the screen and wired to A19: rule packs with effective dates (AU-wholesale P1, AU-MIS-retail P2 stub), token rule sets per trust, the FIRB eligibility matrix, threshold monitors (holder cap, largest, landholder duty, foreign %, s601ED members), the simulator and the decision log;
  - all five product-map actions are reachable with the policy A14 resolves: publish pack version, change rule parameter with the simulation attached, set lock-up dates through A72's form, investor-specific override with legal basis and end date, and the pre-trade threshold block;
  - the state "a change that would put current holders in breach shows them in the simulator first" is enforced by the flow and proved by a test;
  - extension B's four states are rendered.
- **ADM-17 Register and reconciliation.** Done when:
  - the grid reproduces `adRecon` with the fixes above and the product map's columns;
  - the unknown-holders strip is always visible;
  - the break panel shows likely cause, proposed fix and approvals, offers the `6E` adjustment and the corrective-transfer link, and has no close control;
  - the cap table works as of any date and record time;
  - snapshots, certification, signed extracts with verification, both re-sync actions and the R4 registry-sync-failure banner are rendered;
  - the halt state per trust is shown;
  - the product-map actions are reachable with their policies: register adjustment `6E` REGO → CCO + CFO (+ trustee), force re-sync REGO → head of operations (which re-submits pending instructions, then diffs), re-submit of a single timed-out instruction as A18 allows, monthly certification `4E` REGO → trustee or RE, and the extract as a single actor;
  - a Sev2 break shows the API's blocked pre-flight purposes wherever a Sev1 break would show "Resale halted".
- **J20 Operator daily run.** Done when step 3 (CCO checks the threshold monitors, G06 and G07) and step 4 (REGO reviews the 02:00 reconciliation, G13, on-chain and off-chain) can be driven by the seeded personas through these screens in a Playwright run against the Docker stack. That run is the fixture A78 reuses unchanged for the AM7 exit, so it works only on A06's seeded objects and creates no competing request:
  - Step 3 (Jordan Blake, CCO): open ADM-16; see Bay Road near cap in amber from the API; run a simulation of a cap change (nothing written); see APR-2285 (Ann Street holder cap 5% → 7.5%) as a pending parameter change with its attached simulation and its "Licensee notified" prerequisite Pending; open the "Agent authority (G22)" preset and see the seeded agent refusals. A73 does not decide APR-2285; A78 rejects it.
  - Step 4 (Grace L., REGO): open ADM-17; see 7 of 8 matched, 1 break of 2 units, 4h against SLA 24h; open the Gympie Road break, which is `fix_pending` with APR-2292 (maker Grace L., `6E`) and read the likely cause; Jordan Blake (CCO), Helen Tran (CFO) and the trustee (the TP-RE persona from A98, with its own assertion) approve APR-2292 in turn through the modal; Grace submits the `adjust` instruction; the sandbox registry acks; the targeted run closes the break, clears the halt, and the KPI reads 8 of 8.
  - Proposing a new `6E` adjustment is proved by a separate planted-break fixture, not in J20: A06's simulator plants a register drift with no existing fix on another trust, Grace proposes the fix through `propose-fix`, the request opens in A53's flow, and it is approved and applied the same way.
- **G06 Residency and FIRB.** Done when the Foreign % monitor, the per-trust `foreign_policy`, the FIRB matrix and the "FIRB check" pill are rendered from A19's data, and a change to country or residency policy runs through the simulated parameter change.
- **G07 Holder cap.** Done when holder cap and largest holder (associates included, as A19 computes them) show with the API's state, the near-cap and breached tones come from the API, the pre-trade block is a `4E` change, and a cap change always carries a simulation that lists holders who would be in breach.
- **G08 Lock-in.** A product-map gate on ADM-16 through "Set lock-up dates". This node only deep-links to A72's lock-up form (§6 contract); A28 and A72 enforce G08 and own its tests. Done here when the row opens A72's form pre-filled with ids only and the console shows A72's result, never its own lock-up state.
- **G10 Trading halt.** Done when every trust's halt state and causes are shown from A48 (a declared dependency, read live in every halt test) and A20, a Sev1 break shows "Resale halted" on its row within one refresh of the run, a Sev2 break shows the API's amber "Pre-flight blocked: …" list on the row, drill-down and break panel, no console control clears a recon-break halt, and the halt disappears only after the clean re-run.
- **G13 Maker-checker.** Done when every change these screens offer goes through `PrivilegedActionFlow` and `HardwareKeyConfirm`: parameter change, pack publish, override, threshold block, register adjustment, re-sync, certification. The resolved policy code, the checkers and the SoD note come from A14. A maker never sees Approve on their own request, and approval is never displayed as completion.
- **G22 Agent authority.** Done when the decision log has the "Agent authority (G22)" preset showing every agent evaluation with missing scope or cap, every agent-made row on these screens carries `AgentMarker`, and the parity table in §4 is complete, with its "never" and "draft only" rows refused for staff tokens.
- **M10 Compliance rules.** Done when the product map's M10 purpose (versioned rule packs, token rule sets, the FIRB eligibility matrix, threshold monitors and a simulator) is fully operable from ADM-16 with no second source of truth.
- **M11 Register and reconciliation.** Done when snapshots, the daily three-way reconciliation, breaks and register extracts are fully operable from ADM-17, and the rules "tolerance is zero" and "a break halts resale for the trust until resolved" are visible and untouchable in the UI.
- **SM-07 Holding (wallet × token).** Done when the cap table and the break panel render every SM-07 state the API reports as of any date, each with a seeded or planted fixture (A06 seed or A06 simulator):
  - `eligible_empty` (identity registered, no units): not a cap-table row; the break panel shows it only as the wallet named in a break, labelled "Eligible · no units". Fixture: a registered investor with no holding.
  - `active`: a normal row with units and `pct_bps`, green status. Fixture: any seeded Bay Road holder.
  - `locked`: status "Locked until {date}" with the locked units in the Locked column, from the API's state and lock date. Fixture: a seeded holder inside the lock-in period.
  - `partially_frozen`: status "Partially frozen" in amber with the frozen units in the Frozen column. Fixture: a planted partial freeze.
  - `frozen`: status "Frozen" in red, all units in the Frozen column. Fixture: a planted sanctions freeze.
  - `recovered_out`: absent from later as-of dates; on an earlier date it shows grey "Recovered out" and the break panel names the recovery op id. Fixture: a planted recovery.
  - `deregistered` (and a holder who reached a zero balance, back to `eligible_empty`): absent from later as-of dates, grey "Left the register" on earlier dates. Fixture: a planted full redemption.
  - Frozen and locked units are shown exactly as the API returns them. A holding on the chain with no register entry also appears as an `unknown_holder`. The screen changes no holding; SM-07 transitions belong to A28 and A18. If A18 does not report one of these states on the cap table, that is recorded as a finding for A18, never derived in the browser.
- **Founder requirement D04.** Done when every action in §4 is available to a staff member's agent under the same routes, gates and rate limits as the screen, the "never" rows are refused before scope evaluation, and every agent action appears in the audit log as "agent {token} on behalf of {person}".

## Gauntlet loop for this node

- **Specify.** Frozen inputs:
  - this note;
  - A51's `design-ext:ADM-16` frames, `screens.json` and `blocks.md` after HG-REVIEW;
  - A53's shell exports (`ConsolePage`, `ConsoleTable`, `PrivilegedActionFlow`, `HardwareKeyConfirm`, `AgentMarker`, `MaskedCell`, the badge and search registries);
  - the OpenAPI documents of A18, A19, A20 and A48 with their error catalogues;
  - A14's policy rows for `register.adjust`, `register.resync`, `register.certify` and the rule-change action codes in A03's catalogue;
  - A06's seed and scenario simulator (eight trusts, the Gympie Road break `fix_pending` with APR-2292, APR-2285 pending with its prerequisite);
  - A98's TP-RE, TP-REG and TP-AUD principals, A72's deep-link contract and A104's MCP tool catalogue;
  - the answers to the §6 requests.
- Spec point (a), the order of `propose-fix` and the `register.adjust` request, is settled by A20's fixed contract (§3: `propose-fix` first, the request is created server-side, then A53's flow opens it). One spec point is settled with the user before the node QA writes the hidden tests:
  - **(b)** Whether issuing a signed register extract is delegable to a staff agent. Proposed: no, because it is an attestation issued in RealX's name. It is recorded in A03's `delegable.ts`.
- The node QA records `specVersion`, the input hashes, `oracleVersion` and `hiddenCasesHash` in the manifest.
- **Build (Grok 4.7 xhigh, split by the planner).** Suggested shape: one lead session wires the two routes into the shell, registers the badge and search sources, and builds the screen contract fixtures from A06's seed. Nothing else starts until `/rules` and `/recon` render the seeded monitor table and the seven-plus-one recon rows against the Docker stack. Then four parallel sub-agents work on disjoint folders:
  - (1) `monitors/`, `packs/`, `rule-sets/`: the banner, monitor table, per-trust rule set, FIRB matrix, the simulator panel, and the parameter-change, pack-publish and threshold-block flows with the stale-simulation and blocked-prerequisite states;
  - (2) `overrides/` and `evaluations/`: the override table and flow, the decision log with filters, the G22 preset, the evaluation-set view with replay;
  - (3) `recon-grid/`, `breaks/`, `runs/`, `halts/`: KPIs, grid, unknown-holders strip, break panel with the `6E` flow, instruction submit and corrective-transfer link, run page with replay, the halt column;
  - (4) `cap-table/`, `snapshots/`, `extracts/`, `resync/`: as-of and record-time reads, export, versions and certification, extract issue, verify and file check, both re-sync actions, the R4 banner, and `parity.ts` with its static test.
- Integration:
  - Playwright specs for every state at 1280 and 1024 px;
  - the J20 steps 3-4 run with the seeded personas;
  - the planted-drift run through A06's simulator;
  - the parity Docker test with staff tokens through SDK and MCP;
  - screenshots in `app/evidence/A73/`;
  - `npm run verify`;
  - `app/evidence/A73.json`, with the route list, `designKey`s, screenshot set, error-code coverage table, parity table and the reconciled design differences.
- **Challenge (node QA, Opus 5.5 high).** The node QA:
  - runs the hidden specs, then the node-specific challenges below by hand against the running stack;
  - compares its crops of `adRecon` and of extension B's ADM-16 frames with screenshots at token level;
  - checks that every tone on both screens comes from an API field (DOM-to-response trace);
  - greps the feature folder for arithmetic on units, cents and basis points, for `localStorage`, for raw hex and for SDK approval calls outside A53's flow;
  - confirms only the owned paths changed;
  - writes `app/evidence/A73.qa.md`.
- **Gate.** Pass unlocks A78, A79, A80 and A81. At most three fix loops, each re-running the whole Challenge.
- **Escalation trigger.** Any of these escalates:
  - a third loop failing on the same finding;
  - any console path that closes, hides or softens a break, clears a recon-break halt, or shows an approval or `confirmed` state as resolution (Critical, stops the milestone);
  - a need to change an A18, A19, A20, A14, A48 or A72 contract beyond the §6 requests, or a §6 simulation-refusal request declined by A19 or A14 (spec defect: reopen upstream);
  - a panel that cannot be composed from A05, A53's blocks and extension B without raw hex (send back to A51);
  - a refusal whose correctness depends on a counsel answer not yet given (landholder duty, s601ED, FIRB fractional units, extract delegability): record it in HG-LEGAL and ask the user.

## Node-specific challenges (node QA)

- **Stale simulation slips through (money, G07, G13).** The node QA simulates a holder-cap change on Bay Road from 5% to 4% (it lists two holders in breach), acknowledges and submits it. It then settles a new order through A06's simulator so the register version moves, and has LEGL approve.
  - A14 must refuse with `SIMULATION_STALE`, the modal must render the code, and the only offered action is "Simulate again", which starts a new draft.
  - Separately, it edits the draft after simulating: Submit must be disabled until a new simulation exists for the new hash.
  - A console that re-attaches the old result, or submits a draft whose breach list the maker never saw, fails the node.
- **Closing a break by the side door (compliance, G10, G13).** With the Gympie Road break open, the node QA:
  - looks for any close, dismiss, snooze or hide control on the grid, panel, strip and command-centre deep link;
  - approves APR-2292 with CCO and CFO but not the trustee, checks that "Submit adjustment to the registry" is absent, and calls A18's submit through the SDK with that approval id;
  - approves with all three and checks the panel before the registry ack arrives;
  - replays the old `BRK-` URL after the break closes.
  - Expected: no control exists; the SDK submit is refused with A18's code (`REG_ADJUST_TRUSTEE_MISSING`, or `REG_APPROVAL_MISMATCH` if A18's binding table rejects a not-yet-approved id first) and the panel renders that code; "Adjustment acknowledged · waiting for the targeted run" is shown, never "resolved", and the Halt column stays red until A20's run closes the break; the closed break renders as closed with its closing run id.
  - A green row before the clean re-run fails the node.
- **Tones computed in the browser (design fidelity, compliance).** The node QA intercepts `GET /v1/rules/monitors` and sets a row to `current = 4.9%` of a 5% cap with `state = ok`, then another to `current = 3%` with `state = breached`. The screen must show green and red respectively, following the API's `state`, with no client threshold logic.
  - On the recon grid it returns `bank_delta_cents = 1` with `client_money_result = matched`: the Δ cents cell must show A$0.01 in red while the status shows the API's result. The mismatch is recorded as an API inconsistency in `A73.qa.md`, and the console must not "correct" it.
  - A component that derives a tone, a total or a delta from numbers fails.
- **Escrow shown as register units and the missing eighth trust (design fidelity).** With the standard seed:
  - Kent Street must read register `0`, token supply `0` and "Applications (escrow) 6,400";
  - Marine Parade must be present as "Not issued";
  - the KPI must read "7 of 8".
  - A screenshot that reproduces the prototype's "6,400 (escrow)" in the register column, or omits Marine Parade, fails.
- **An incomplete run looks clean (money).** The node QA makes A20's stub chain source unavailable for Hawthorn Grove and triggers a run. The row must read amber "Incomplete · chain source unavailable", the KPI must not count it as matched, and the run page must list the missing source.
  - It then plants a one-cent bank delta on Bay Road and an unknown holder on Marion Street through A06. The strip must show the Marion Street wallet as Sev1 and the grid must show both breaks.
  - With every break cleared, the strip must still be visible and read "0".
- **A staff agent reaches past the parity table (agent misuse, D04).** With a REGO token for Grace L. holding every scope her role allows, the node QA first plants a register-drift break with no existing fix through A06's simulator (not Gympie Road, which already carries APR-2292). Then it:
  - has Grace's agent propose a `register.adjust` fix on the planted break through `propose-fix` (accepted as maker, `AgentMarker` on the request);
  - has the agent try to approve it (`APR_CHECKER_MUST_BE_HUMAN`);
  - has a sibling agent of CCO Jordan Blake approve it (refused: an agent never approves);
  - has Grace's agent call `POST /register/{id}/extracts` (`AGNT_NOT_ALLOWED` under spec point (b));
  - has Grace's agent exceed 10 manual runs in an hour (`429` with `Retry-After`).
  - SoD case, on an action where the human is an eligible checker: Jordan Blake's CCO agent drafts an investor-specific override (A19 names `6E` with CCO + LEGL), and Jordan then approves it in person. It must be refused with `APR_MAKER_IS_CHECKER`, because SoD counts the agent as its human. If A14's resolved policy row makes CCO ineligible as a checker for overrides, the node QA picks, at Specify, another action code whose resolved maker role is also in its checker set, and records the choice in the manifest.
  - All of this happens through the SDK and through the MCP server, and each action must appear in the ADM-17 break panel (for break actions) and the audit log as "agent rx_stf_… on behalf of {person}".
- **The simulation rule is bypassed through the API (money, G07, G13).** Through the SDK and the MCP server, with Jordan Blake's CCO staff token and separately with his human session, the node QA submits a holder-cap change with no simulation, one with the result hash of a different draft, and one with an acknowledged count lower than the simulation's breach count. Each must be refused with `SIMULATION_MISSING`, `SIMULATION_DRAFT_MISMATCH` or `BREACH_LIST_NOT_ACKNOWLEDGED`, and no request may reach a checker. The same three cases are repeated for a pack version, an override and a threshold block.
- **Decision-log filter leaks (security, G22).** As a REGO without `V` on M26, the node QA opens the "Agent authority (G22)" preset. The ADM-35 link must be absent. The rows must show fact names, never investor names, TFNs or addresses. Changing the `token_id` filter to a token of another tenant's sponsor must return only what the API's row-level security allows. No PII may appear in the URL, the document title or any `data-*` attribute.
- **Cap table time travel (compliance).** The node QA issues a signed extract as of 30 June, then pushes a back-dated `6E` adjustment through the screens.
  - The cap table as of 30 June must show the corrected units.
  - The same date with "As the register knew it at" set to the extract time must reproduce the extract's totals.
  - "Verify" on the extract must still read valid.
  - "Check a file" with one altered byte must read `matches: false`.
  - The percentages must add up to exactly 100.00% on every date.
- **Force re-sync that changes something (money).** The node QA runs the `4E` "Force re-sync (diff against the registry)" on a vehicle with the sandbox `-DRIFT+2` scenario. The diff must show +2 for the named holder, the register must be unchanged afterwards, and the only follow-on action must be "Propose adjustment (6E)". The A20 "Fetch a fresh extract and re-run" must queue a targeted run and change no entry.

## Pass criteria

- The node QA's hidden Playwright specs in `<workspace>\gauntlet\hidden\A73\` pass 100 % on three consecutive runs against the Docker stack with the sandbox authenticator, at 1280 px and 1024 px, with no horizontal page scroll at 1024 px.
- **Design fidelity.** The node QA's token-level comparison of the ADM-17 header, KPI grid and recon table against its `adRecon` crop, and of ADM-16 against extension B's frames in all four states, shows no drift beyond 1 px in font family, size, weight, letter-spacing, colours, radii, borders or paddings. The allowed exceptions are the recorded reconciled differences (the Marine Parade row, the escrow column, the Δ, bank and Halt columns, the live badge, the API's pack code). The standard seed reproduces "7 of 8 · Run 06:00", "1 · 2 units", "4h · SLA 24h" and "Gympie Road Retail 9,998 / 10,000 / Break · 2 units" exactly. Screenshots of every state are listed in `app/evidence/A73.json`.
- **Tones and numbers from the API only.** A generated test mutates each monitor `state`, recon `result`, break `severity` (Sev1 and Sev2) and `blocked_purposes[]`, and halt `halted` value in intercepted responses and asserts that the rendered tone and blocked-action text follow the fields in 100 % of cases. The KPI test asserts "Trusts matched" prints `trusts_matched` and `trusts_total` unchanged while the seeded grid has six green Status cells. The feature-folder grep finds no arithmetic on `*_units`, `*_cents` or `*_bps` outside the display formatters.
- **Maker-checker surface.** Every action in §2-§3 that changes state reaches A14 only through A53's flow and modal (A53's lint rule passes over the feature folder, and the node QA's planted direct `approvals.create` import fails it). The simulation-attached rule holds in the UI (Submit is disabled without a simulation for the current draft hash, verified over 50 random edit sequences) and on the server: through the SDK and the MCP server, with a staff token and with a human session, a parameter change, pack version, override and threshold block each submitted with no simulation, another draft's simulation, or an unacknowledged breach list is refused with `SIMULATION_MISSING`, `SIMULATION_DRAFT_MISMATCH` or `BREACH_LIST_NOT_ACKNOWLEDGED` (12 cases per caller type, all refused, no request reaching a checker). The `SIMULATION_STALE`, `REG_ADJUST_TRUSTEE_MISSING`, `REG_ADJUST_REQUIRES_6E` and blocked-prerequisite states each have a test and a screenshot.
- **Breaks and halts.** In the planted-drift run (register drift, unknown holder, one-cent bank delta, incomplete source, plus a Sev2 R17 `pending_ack` from a timed-out instruction):
  - each break appears on the grid within one refresh of the run completing;
  - the strip shows the unknown holder as Sev1;
  - the Halt column turns red for Sev1 breaks, read from the live A48 module;
  - each Sev2 break (the `source_unavailable` and the `pending_ack`) shows the API's amber "Pre-flight blocked: …" list on its row, drill-down and break panel, and "Re-submit" on the timed-out instruction returns it to `submitted` on the same row;
  - no break or recon-break halt disappears except after a clean A20 run;
  - a DOM scan of every ADM-17 view finds no control labelled or wired to close, dismiss, snooze, suppress or delete a break.
- **Error coverage.** A generated test over the OpenAPI error catalogues of A18, A19, A20 and A48 (the codes these screens' actions can return), plus A14's decision codes, finds a rendered sentence for 100 % of them, with the code and `request_id` visible.
- **Parity (D04).** `parity.ts` lists every mutation the feature folder calls (static test). Every "never" and "draft only" row is refused for seeded REGO and CCO staff tokens through the SDK and the MCP server with the named code. Every "yes" row succeeds for those tokens within their mandate and writes one audit event with `actor_type = AGNT`, `token_id` and `on_behalf_of_id`. The G22 preset in the decision log shows each of those calls.
- **J20.** Steps 3 and 4 pass end to end with the seeded personas on the seeded objects (APR-2285 shown pending and undecided; APR-2292 approved by CCO, CFO and the A98 trustee principal, the instruction acked, the break closed by the targeted run, "8 of 8" afterwards), and the spec is exported unchanged for A78. The separate planted-break fixture proves a new `propose-fix` → `6E` → instruction → closing run.
- **SM-07.** Each of the seven SM-07 states in "Requirements covered" has its fixture and a screenshot of the cap table (and the break panel where named) on a date where the state applies.
- **Privacy and hygiene.**
  - No PII in URLs, titles, `data-*` attributes or browser storage (scan over every route and state).
  - `MaskedCell` puts no raw holder name in the DOM before unmask.
  - axe reports zero critical or serious findings on both screens, and every flow is keyboard-operable.
  - `npm run verify` is green from a clean clone on Node 24.21.0.
  - Only the owned paths changed.
  - `app/evidence/A73.json` validates against A00's schema and matches disk.

## Human gates touched

- None crossed. The node runs on the Docker stack with the sandbox registrar, the stub chain source, the sandbox bank rail, the sandbox signing key for extracts and A11's sandbox authenticator. No real register, registry, bank, chain or key is touched.
- **HG-REVIEW (before this node starts):** the founder reviews extension B's `design-ext:ADM-16` frames and the recorded reconciled differences on `adRecon`. This node builds only what was reviewed.
- **HG-LEGAL (counsel questions recorded, not decided here):**
  - the landholder-duty thresholds by state and the s601ED member-count treatment, shown as "counsel to confirm" (A19's `counsel_pending` parameters);
  - how the FIRB established-dwelling rule applies to fractional trust units (G06);
  - whether issuing a signed register extract, or requesting certification, must be done by the person rather than their agent (spec point (b), A03's `delegable.ts`).
  - Counsel's answers change pack data and `delegable.ts`, never this node's code.
- **HG-VENDOR (later):** the real registry and the trustee's register sit behind A18's adapter. When a vendor is contracted, these screens must pass their hidden specs unchanged against it.
- **HG-PENTEST (later):** the decision-log filters, the cap-table unmask path, the extract upload check and the re-sync actions are in scope of the external penetration test. This node passes on software evidence and records the gate as open.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
