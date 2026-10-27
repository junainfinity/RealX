---
title: "A76 - Console: regulatory calendar and data room, sponsor performance, integrations and jobs"
created: 2026-09-24
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am7]
node: A76
milestone: AM7
lane: "Operator console"
oracle: no
human_gate: none
---

# A76 - Console: regulatory calendar and data room, sponsor performance, integrations and jobs

**Three operator console screens built from design extension B. ADM-26 is the compliance back office: the obligations calendar at 7, 30 and 90 days (with AML clocks for SMR, TTR and IFTI projected read-only to the MLRO and CCO only), the filing workspace, time-boxed data-room grants for auditors and regulators, board pack and report exports, and governed metric definitions. ADM-29 is the sponsor track record: raised (audited only), yield plan vs paid, on-time, proof score, retained stake and watchlist flag, plus complaints, incidents and fee and revenue-share agreements. ADM-32 is the platform's operational health: connectors with environment, latency and state; job runs, dead letters and pauses; webhooks platform-wide; the operator sandbox panels; the human-gate status board, which shows all seven gates as open; and a link to ADM-35 for tokens and agents. Every action on the three pages is an API operation that a staff member's agent can call under the same rules, except the acts that must be a person's.**

**Build status:** not started. **Milestone:** AM7 (Operator console). **Lane:** Operator console. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 10-14 agent-hours (build, challenge and verify loops). This replaces the graph's 6-10: the node builds three screens with 13 tabs, a bundle-stripping check, two contract tests and parity sweeps for every persona and every persona's agent. The three upstream loops it needs (A33.1, A45.1, A46.1, §1) are extra: about 2-4 agent-hours together on their owners' nodes. The orchestrator updates `est_lo`/`est_hi` in `app-graph.json` and gives the three loops graph nodes or hours.

**Depends on:** [[A53 - Operator console shell, hardware-key sign-in and approval action pattern]], [[A45 - Reporting, regulatory calendar, filings and metric governance]], [[A33 - Sponsor tenancy, entities, licences of sponsors and admission pipeline]], [[A46 - Job scheduler, integrations health and webhooks]], [[A51 - Design extension B- operator console screens not in the prototype]]
- Transitively (through A45, A46 and A53) this node already sits after A06 (personas and seed), A12 (tokens and activity), A14 (approvals API), A42 (AML clocks), A101 (sensitive-action notices) and A104 (the MCP server), so the Challenge may use them.
- It does **not** sit after A66 (approvals inbox), A77 (ADM-35) or A99 (sponsor sandboxes). No check at this gate needs their screens: approvals are checked through A14's `GET /v1/approvals`, a person's activity through A12's `GET /v1/agent-access/activity`, and the ADM-35 links by their `href` only. The same checks through A66's and A77's screens run at [[A78 - Operator journey J20 end to end in Docker (AM7 exit)]], which depends on both.

**Unlocks:** [[A78 - Operator journey J20 end to end in Docker (AM7 exit)]], [[A80 - End-to-end journeys- sponsor and operations]]

**Oracle rule:** this is not an oracle node (`oracle: no`). Money, supply and holdings do not move on these pages. Every rule the pages show is decided by A45, A42, A33, A46, A12 and A14. The pages do carry three things a mistake would expose: AML clocks (tipping-off), regulator data-room access and the platform's environment status. So before the build starts, the node QA still writes hidden tests and Challenge fixtures to `<workspace>\gauntlet\hidden\A76\` ([[App Gauntlet]] §2). They include:
- Playwright specs against the running stack with the sandbox authenticator
- an AML-projection persona sweep
- a data-room window and revoke script under the injectable clock
- a sponsor-performance fixture that plants unaudited and missing figures
- a failing-job, dead-letter and indexer-lag script
- the hostile environment fixtures (a `live` string, an eighth human gate, a closed gate)
- its own crops of the extension-B ADM-26, ADM-29 and ADM-32 frames
- an agent-parity table for every action on the three pages, with expected classes taken from A03's `delegable.ts`

The builder never reads that folder. The tests are published into `app/` only after the gate passes.

This node draws screens and holds no business rule. The API decides all of the following:
- when an obligation is overdue
- who may submit a filing
- whether an AML item exists for the caller
- what a data-room grant may reach and when it ends
- which figure is audited
- whether a sponsor is on the watchlist
- whether a connector is up
- whether a job may be paused by one person
- whether a human gate is open
- which buttons the caller may press on each row (`allowed_actions[]`)

A45 decides the calendar, filings, data room, reports, metrics and the ADM-29 row. A42 decides AML clocks, which A45 projects. A33 decides the watchlist, pauses and revenue share. A46 decides jobs, integrations, webhooks and the sandbox panels. A12 decides tokens. A14 decides maker-checker. The page shows what the API decided, cites the rule the API names, and never lets a person believe that a filing, a grant, a pause or a configuration has changed before the API says so. Tokens, mandates and agent write tools are managed on ADM-35 ([[A77 - Console- agent and API access (ADM-35)]]), and whether write tools are enabled at all is [[A104 - MCP server generated from the OpenAPI document]]'s spec point (a). The partner's own view of a data-room grant is on the partner portal ([[A105 - Partner portal- trustee-RE, auditor, regulator observer and licensee views]]). This note is not legal advice.

## Build brief

**1. What this node reuses, and the API it needs first (no API code here)**

- **From A53** (`src/shell/index.ts` only):
  - Page and table parts: `ConsolePage`, `ConsoleTable`, `FilterBar`, `StatusCell`, `IdCell`, `WhenCell`, `MaskedCell`, `AgentMarker`.
  - Action pattern: `PrivilegedActionFlow` with `usePrivilegedAction(action_code)`, and `HardwareKeyConfirm` (purpose as a prop).
  - Registries: `registerBadgeSource` and `registerSearchProvider`.
  - The PROD / SANDBOX environment badge, which this node never re-derives.
  - The forbidden, not-found and session-expired states.
- **From A51** (`packages/ui/src/ext/admin/`): the ADM-26, ADM-29 and ADM-32 blocks and the KeyConfirmModal states.
- **From A05:** `DataTable`, `KpiGrid`, `Tabs`, `FilterChip`, `StatusPill`, `Banner`, `KeyValueList`, `TaskRows`, `OptionCards`, `RaiseBar`, `TextArea`, `DocumentRow`.
- Nothing is redrawn and no raw hex is written. A05's lint runs over this route group.
- **Approval codes.** `usePrivilegedAction` takes the code exactly as its owner names it. The page uses these eleven and no others:

  | Code (owner) | Policy | Maker → checker |
  |---|---|---|
  | `M19.obligation.waive` (A45) | 4E | CCO → LEGL |
  | `M19.obligation.editSeed` (A45) | 4E | LEGL → CCO |
  | `M19.dataRoomGrant.request` (A45) | 4E | CCO → SECO |
  | `M19.dataRoomGrant.extend` (A45) | 4E | CCO → SECO |
  | `M19.metricDefinition.change` (A45) | 4E | CFO → CCO |
  | `M19.metricPublication.publish` (A45) | 6E | IRCM → CFO + CCO |
  | `tenant.revenue_share.set` (A33; A03 id `M24.revenueShare.set`) | 6E | PART → CFO + CCO, plus LEGL when `transaction_based` |
  | `jobs.schedule.pause` (A46) | 4E, `critical` rows only | ENGO → SECO |
  | `jobs.schedule.disable` (A46) | 4E, every row | ENGO → SECO |
  | `jobs.dead_letter.discard` (A46) | 4E, `money: true` rows only | ENGO → CFO |
  | `integrations.config.change` (A46) | 4E | ENGO → SECO |

- **From A45** (already specified there; used as named):
  - Calendar: `GET /v1/reporting/calendar` (`horizon=7|30|90`, `regime`, `owner`, `state`), `GET /v1/reporting/calendar/summary` (`{due_7, due_30, due_90, overdue, unassigned}`, which excludes `source: aml` rows for every viewer), `GET|PATCH /v1/reporting/obligations/{id}`, `POST .../complete | assign | waive | edit-seed`, and `.../history`.
  - The read-only AML projection (A45's `aml_projection.ts` over A42): rows with `source: aml`, returned only to MLRO and CCO, 404 on every write route and to every other principal, token and agent.
  - Filings: `GET|POST /v1/reporting/filings`, `GET /v1/reporting/filings/{id}`, `POST .../ready | submit | acknowledge | reject | withdraw`, and `GET .../pack`. A45 has no `smr`, `ttr`, `ifti` or `crs_fatca` kind; those are A42's and A41's (`FIL_KIND_OWNED_ELSEWHERE`).
  - The local test route `POST /v1/_test/reporting/acknowledge/{id}`, mounted by A45 in `RUN_MODE=local` only.
  - Data room: `GET|POST /v1/dataroom/grants`, `POST .../extend | revoke`, `GET .../activity`, and `GET /v1/dataroom/collections`.
  - Reports: `GET|POST /v1/reports/exports`, `GET /v1/reports/exports/{id}` and `GET /v1/reporting/prudential`.
  - Metrics: `GET /v1/metrics/definitions`, `POST /v1/metrics/definitions/{key}/versions`, `GET /v1/metrics/values`, and `GET|POST /v1/metrics/publications`.
  - Sponsor performance: `GET /v1/reporting/sponsor-performance` (`raised_cents`, `yield_plan_vs_paid_bps`, `on_time_bps`, `proof_score_bps`, `keeps_bps`, `flag`, `late_distributions_6m`, `incidents`, `complaints`).
  - A45's error codes and its three person-only rules (`FIL_PERSON_ONLY`, `MET_ATTEST_PERSON_ONLY`, `DR_SELF_GRANT`), and its rule that revoking a grant is an `act` with no token scope.
- **A45.1**, a follow-up loop requested from A45 before this build (none of it is in A45's spec today):
  - `allowed_actions[]` for the caller, in the current state, on every row and detail of the calendar, obligations, filings, grants, exports, metric definitions and publications that this page renders.
  - On each projected AML row, A42's `clock_policy` (`smr_3bd`, `smr_tf_24h`, `ttr_10bd`, `ifti_10bd`), the clock unit (`business_days` or `hours`) and `remaining_ms`, so the page renders the unit the API returns and never assumes business days.
  - On `sponsor-performance`, `flag` derived from A33 instead of A45's own watch rule: `watch` when A33 has an open `sponsor_watchlist` row for the tenant, `suspended` when A33's tenant status is `suspended`, otherwise `none`. A45's rule-pack row "two late distributions in six months" is removed, because A33's job already opens the watchlist row on that rule (A33 §7 already says A45 composes the table "from the watchlist"). This is a spec defect between A45 and A33, and Specify stops until it is settled (see Gauntlet).
- **From A46** (already specified there):
  - Jobs: `GET /v1/jobs`, `GET /v1/jobs/{name}` and `.../runs`, `POST .../trigger | pause | resume | disable`, and the dead-letter list with `replay` and `discard`.
  - Integrations: `GET /v1/integrations`, `GET /v1/integrations/{name}`, `samples`, `config` (versions and proposals), and `GET /v1/integrations/summary` (counts, applied config versions and `banners[]` carrying `registry_sync_failed`).
  - The sandbox forwarding route `POST /v1/integrations/{name}/sandbox/{event}` (mounted in `RUN_MODE=local|test` only).
  - Webhooks: `GET /v1/webhooks` and `GET /v1/webhooks/{id}/deliveries` under `webhooks:read_all` (SECO, ENGO).
- **A46.1**, a follow-up loop requested from A46 before this build:
  - `GET /v1/integrations/summary` gains `human_gates[]`, generated at build time from A00's `docs/charter/human-gates.md` (or its JSON twin) joined with A08's `SWAP-POINTS.md`. It has one row per gate: `HG-LEGAL`, `HG-VENDOR`, `HG-KEYS`, `HG-AUDIT`, `HG-PENTEST`, `HG-CAPITAL`, `HG-PROD`. Each row carries `code`, `title`, `what_the_human_does` (one line), `state`, `nodes[]` (nodes it gates, from the graph data), `connectors[]` (integrations it blocks, from the swap points) and `evidence_ref` (null while open).
    - `state` is `open | closed`. No code path in this build writes `closed`: closing a gate is a human evidence step recorded in the charter data, never a button.
    - HG-REVIEW and HG-LAUNCH are process gates on the plan, not platform gates, so they are not listed.
  - `allowed_actions[]` for the caller on every job, run, dead letter, integration and config version row this page renders.
  - Each webhook row carries `receiver_host` (host only, no path or query).
- **From A33** (routes already in A33's staff route table; used as named):
  - `GET /v1/admin/sponsor-metrics?tenant=&month=` (the snapshot series, with `source: missing` per absent provider, `complaints_open`, `incidents` and `per_vehicle[]` retained stake against its minimum).
  - `GET|POST /v1/admin/sponsor-watchlist` and `POST /v1/admin/sponsor-watchlist/{id}/close` (`M24.watchlist.read | open | close`, reason mandatory; a staff agent's open or close is `confirm` in A33's table, so it returns A12's `428 CONFIRMATION_REQUIRED`).
  - `GET|POST /v1/admin/revenue-share` (`M24.revenueShare.read | set`; POST opens `tenant.revenue_share.set`).
  - `GET /v1/admin/tenants/{id}` for `open_pauses[]` and tenant status.
- **A33.1**, a follow-up loop requested from A33 before this build: `allowed_actions[]` for the caller on the watchlist, revenue-share and tenant responses above, and a `reason_text` sentence on each open pause (`attestation_overdue`, `billing_overdue`, `watchlist_hold`, `licence_expired`, `suspended`), so the page holds no pause wording.
- If A33.1, A45.1 or A46.1 has not landed and passed before this node's Build step, the node waits. It never decides a button in the client from roles, and it never mocks the API in production code; fixtures exist only in tests. Every endpoint carries `x-realx-scope` in M19, M24 or M26, so it is in the OpenAPI document, `packages/sdk` and A104's MCP catalogue from the moment its owner passes.

**2. Page structure (one route group, three screens, tabs as route segments)**

- **ADM-26** at `app/(reporting)/reporting/`:
  - `screen = "ADM-26"`, `designKey = "design-ext:ADM-26"`, h1 "Regulatory calendar and data room", subtitle "Every filing and report with its due date, and who may see our records."
  - Tabs `calendar` (default), `filings`, `data-room`, `reports`, `metrics`.
  - A `KpiGrid` of four tiles from `GET /v1/reporting/calendar/summary`: due in 7 days, due in 30 days, overdue (red when above 0), unassigned due within 30. The summary excludes AML rows for every viewer, so the tiles are the same for the MLRO as for anyone else with the same A45 rows.
  - The metric-governance line under the header: "Public numbers only from audited figures (N16)". Its text comes from A45's metric definitions endpoint, not from a string in this app.
- **ADM-29** at `app/(reporting)/partners/sponsors/`: `screen = "ADM-29"`, `designKey = "design-ext:ADM-29"`, h1 "Sponsor performance", subtitle "How each sponsor performs. Feeds the public sponsor profile."
- **ADM-32** at `app/(reporting)/integrations/`:
  - `screen = "ADM-32"`, `designKey = "design-ext:ADM-32"`, h1 "Integrations and jobs".
  - Tabs `connectors` (default), `jobs`, `webhooks`, `sandbox` (see §8 for when it exists), `human-gates`.
  - A header `KpiGrid` from `GET /v1/integrations/summary`, in the shape of A46's design note: connectors ok N of M, jobs failed 24 h, dead letters open, webhook deliveries failing.
  - A persistent line "Tokens, agent tools and API keys are managed in Agent and API access" linking to ADM-35 (A77).
- **Sidebar and visibility.** Items sit where A51's seven-group sidebar puts them: ADM-26 under "Risk & records", ADM-29 under "Asset life", ADM-32 under "Platform". Each item and each tab is rendered only for principals whose `GET /iam/me` holds `V` on the screen's module:
  - M19 for ADM-26: CCO, LEGL, CFO, MLRO, CEO, SUPER, IRCM, TAXO, TRSY and SECO by A45's `M19.calendar.read`.
  - M24 for ADM-29: PART, ASTM, CCO and CFO.
  - M26 for ADM-32: ENGO, SECO, PART, CCO, CEO, REGO and TRSY by A46's `jobs:read`.

  The sets are those in A03's matrix; the lists above are the expected result, checked by a test, not coded. Partner principals (TP-AUD, TP-OBS, TP-LIC) do not sign in to the console. Their views are A105's.
- **Badge sources:**
  - `reporting.attention`: overdue A45 obligations and filings, plus A45 filings due within 3 business days, counted for the caller's own roles from the summary endpoint and the filings list. AML clocks are never counted here; A66 takes them from A42's own summary.
  - `sponsors.attention`: open watchlist rows and open listing pauses.
  - `integrations.attention`: connectors `down`, jobs failed in 24 h, open dead letters.
- **Search provider:** `OBL-`, `FIL-`, `DRG-`, `REP-`, `DLQ-` and `WHS-` ids, and job and integration names. The provider never indexes projected AML rows or any A42 id, for any caller; a search for one returns "no result", the same as an unknown id.

**3. ADM-26 Calendar tab**

- A `ConsoleTable` over `GET /v1/reporting/calendar`.
  - `FilterChip` groups: horizon (7 d / 30 d / 90 d / Overdue), regime (AUSTRAC, ASIC, ATO, DDO, licensee and CAR, internal) and owner role.
  - Columns follow A51's frame: Obligation / Regulator / Due / Owner / State. The regulator label is rendered from A45's `regime` through `copy.ts`. The due date is in Geist Mono with the remaining time as a subline, in the unit the row's clock uses: "N business days" for a business-day clock, "N h M min" for an hours clock. The page never converts one into the other.
  - The last column is always a `StatusPill`: `open` grey, `in_progress` amber, `completed` green, `waived` grey, `not_yet_applicable` grey with a brick "P2" pill.
  - `overdue` is a flag, so the pill turns red and reads "overdue" whatever the state.
  - A `read_only` row (`source` of `perimeter`, `tax`, `derived_issuer`, `derived_licensee` or `aml`) has no complete, assign, waive or edit action, because its owner is elsewhere. A `perimeter` row shows the muted mono caption "derived from licence LIC-…" and links to ADM-04 (A69). A `tax` row shows A41's request id and acknowledgement as evidence. A `derived_*` row names the partner that owns it.
- **AML rows (G14, tipping-off).** For MLRO and CCO only, A45's projection adds A42's SMR, TTR and IFTI items as `source: aml` rows with `regime: austrac`.
  - Each shows a typed reference and its clock only, never a name or narrative, and links to the case on ADM-25 (A42), where the SMR is decided and filed. It has no action on this page.
  - An `smr_3bd` row counts down in business days ("Suspicious matter reports go to AUSTRAC within 3 business days", `adAml`'s wording). An `smr_tf_24h` row (terrorism financing) counts down in hours from A42's `remaining_ms` and turns red and "overdue" at 24 elapsed hours, as the product map's J14 step states.
  - For every other principal, token and agent, the rows do not exist: no row, no count, no placeholder, no search hit, and the shell's not-found state on any id URL, exactly as A45's API behaves. The page draws no counter that sums rows the API did not return.
- Row detail (split view in the ApprovalList/Detail pattern):
  - A `KeyValueList` of basis text (marked "to be confirmed with counsel" while `set_with_counsel` is true), period, owner and secondary roles, assignee, evidence (`DocumentRow` per document, count shown), linked filing, source and version.
  - The last 20 `obligation.*` audit events, with an `AgentMarker` on agent-made rows.
  - Actions come only from `allowed_actions[]` (A45.1):
    - **Assign.**
    - **Attach evidence**, which uploads through A08 storage via A45's route.
    - **Mark complete.** It is refused with `OBL_EVIDENCE_REQUIRED` when there is no evidence or linked filing, and the page shows A45's sentence.
    - **Waive**, through `PrivilegedActionFlow` `M19.obligation.waive` (CCO → LEGL, 4E, reason required).
    - **Edit due date or basis**, through `PrivilegedActionFlow` `M19.obligation.editSeed` (LEGL → CCO, 4E). The dry run lists the clocks that will be cancelled and re-armed.
- Empty states:
  - "Nothing due in this window." (dashed box)
  - A platform whose seed is empty shows "No obligations yet. The seeded catalogue loads with the API." That case is a defect and is tested for its absence.

**4. ADM-26 Filings tab**

- A split `ConsoleTable` over `GET /v1/reporting/filings`: ID / Kind / Subject / Due / Submitted / State.
  - Kinds are A45's only: `outside_tmd` (P2), `austrac_compliance_report`, `enrolment_change`, `reportable_situation_notice`, `reportable_situation` (P2), `wind_down_report`, `other`. SMR, TTR and IFTI filings live on ADM-25 (A42) and CRS/FATCA on A41's screens; neither appears in this list for anyone.
  - Pills: `draft` grey, `ready` amber, `submitted` green-tint, `acknowledged` green, `rejected` red, `withdrawn` grey, with `overdue` red over any state.
  - The subject shows a typed reference (order, vehicle, licence, CAR appointment, wind-down) as an `IdCell`, never a person's name.
- Detail card:
  - `KeyValueList` of pack hash (mono), lodgement reference, recipient reference, the A13 clock with its remaining time in the unit the API returns, evidence, prepared by and submitted by.
  - Buttons from `allowed_actions[]`: **Mark ready** (refused with `FIL_EVIDENCE_REQUIRED`), **Download lodgement pack** (audited by A45), **Submit** and **Withdraw** (reason mandatory).
  - **Submit** (LEGL or CCO) is a form that takes the lodgement reference the person copied from the recipient's own channel (the regulator's portal, or the issuer's or licensee's acknowledgement). It calls `submit`; `FIL_LODGEMENT_REF_REQUIRED`, `FIL_TRANSITION_NOT_ALLOWED` and `FIL_PERSON_ONLY` render verbatim.
  - The page never calls a regulator. It says so in the form's help text: "RealX does not lodge for you. Lodge on the regulator's portal, then record the reference here."
- **Acknowledge (test).** When `GET /v1/meta` reports `RUN_MODE=local` (A45 mounts the route only there) and the console was built with the sandbox UI on (§8), the detail shows a brick "sandbox" **Acknowledge (test)** button calling A45's `POST /v1/_test/reporting/acknowledge/{id}`, so A78 and the node QA can close a filing. In `RUN_MODE=test` the button is not rendered, because the route is not mounted there. In a bundle built with the sandbox UI off, it is absent from the code.
- Outside-TMD filings (P2) render with the brick "P2" pill and the 10-business-day clock. In P1 they appear only from A45's fixture in local runs.

**5. ADM-26 Data room tab**

- One `KeyValueList` card per grant from `GET /v1/dataroom/grants`, with the fields Grantee / Purpose / Scope / Window / Granted by / State.
  - The grantee is the partner principal's display name and role (TP-AUD, TP-OBS, TP-LIC) with the `PRN-` id in mono.
  - Scope lists tenants, vehicles, collections and a date range.
  - The window is `starts_at` to `ends_at`, with the time remaining read from the server; there is no client timer alone.
  - Granted by is the `APR-` id.
  - State pills: `pending` amber, `active` green, `expired` grey, `revoked` red.
- Under each card is a `TaskRows` activity list from `GET /v1/dataroom/grants/{id}/activity`: item viewed and item downloaded, each with the item reference, the time and `on_behalf_of` when a partner's agent token read it (with `AgentMarker`). The list is cursor-paged.
- **Grant access** (CCO) runs `PrivilegedActionFlow` with `M19.dataRoomGrant.request`:
  - The form takes the grantee (a picker over active A98 partner principals only), purpose (mandatory), collections (from `GET /v1/dataroom/collections`, at least one, with no "everything" choice), tenants, vehicles, date range and window (default 30 days, the maximum shown from the platform setting).
  - A45's precheck errors render before submission: `DR_GRANTEE_NOT_PARTNER`, `DR_WINDOW_TOO_LONG`, `DR_PRINCIPAL_INACTIVE`, `DR_SELF_GRANT`, unknown collection, start before approval.
  - SECO approves through the card's own `HardwareKeyConfirm` (the same request also appears in A66's inbox, which A78 checks). The maker sees the modal's `not-eligible` state.
  - **Extend** is a new `M19.dataRoomGrant.extend` 4E. The old card stays as it is.
- **Revoke** is a danger-outline button in alert red `#A3312A`, for SECO or CCO, shown only from `allowed_actions[]`. It has a confirm dialog naming the grantee and a mandatory reason. It is effective at once, and the card reloads to `revoked` only after the 2xx. It is a person's act: A45 gives it no token scope.
- The AML collections do not exist in the collection picker, because A45's catalogue does not offer them. The tab states the default answer as data: "AML filings are never in the data room (counsel to confirm)."
- Empty state: "No data-room grants. Auditors and regulator observers get time-boxed access here, approved by a second person."

**6. ADM-26 Reports and Metrics tabs**

- **Reports:**
  - A `ConsoleTable` over `GET /v1/reports/exports`: ID / Kind / Period / Scope / Figure basis / Requested by / State, with `queued` amber, `rendered` green and `failed` red.
  - **Export board pack**, **Export investor report**, **Export sponsor report** and **Export prudential dashboard** are single-actor, audited acts (`M19.boardPack.export` and `M19.report.export`: CFO, CCO, CEO, LEGL).
  - The form picks kind, period and scope. The row links the rendered document through A08 storage and shows `file_sha256` and the metric versions used.
  - Refusals render verbatim: `REP_NO_AUDITED_FIGURES` naming the missing keys, and `MET_RENDER_REFUSED`.
  - A board pack with management figures shows the API's "management figure, unaudited" label in the preview, never stripped.
  - A prudential summary card reads `GET /v1/reporting/prudential`: client-money delta and last reconciliation, escrow per open offer, the DAF Act meter, open and overdue filings, active grants, and audit-chain head status. It is the CCO's J20 step 1 read beside ADM-01.
- **Metrics:**
  - A `ConsoleTable` over `GET /v1/metrics/definitions` joined with the latest values: Key / Name / Version / Basis / Public / Last value.
  - The definition text expands under the row, followed by its version history.
  - A value with `basis: management` shows the mono value with a grey "management" pill. A value with `basis: audited` shows a green "audited" pill and the attestation document. A null value shows "no value yet", never 0.
  - **Change definition** runs `PrivilegedActionFlow` `M19.metricDefinition.change` (CFO → CCO, 4E).
  - **Publish figures** (IRCM) runs `M19.metricPublication.publish` (6E, CFO + CCO). This is the product map's "Publish public KPIs and track record", which carries G17 ("Claims reviewed"). The precheck names every management-only key, forbidden label, retired version and suspended tenant, and A43's `checkClaims` result on the exact strings; the page renders `MET_FORBIDDEN_LABEL` and the claims flags verbatim and offers no way to submit past them. The request carries the exact rendered strings the page shows.
  - The page has no attest control. Attestation is a TP-AUD act on the partner portal, and a staff principal sees "Attested by the auditor on the partner portal" with the document link.
- The strings "tokenised value", "TVL" and the rest of A45's `forbidden_labels.ts` appear nowhere in this node's code or copy. The verify grep from A45 is extended to this route group.

**7. ADM-29 Sponsor performance**

- A `ConsoleTable` over `GET /v1/reporting/sponsor-performance`. Columns follow A51's frame: Sponsor / Trusts / Raised / Yield plan vs paid / On-time / Proof score / Keeps / Flag.
  - Raised is A45's `raised_cents`, formatted from integer cents in A$, Geist Mono. When null with `basis: management`, the cell shows "awaiting audit" muted, never a management number and never 0.
  - Yield plan vs paid is `a% / b%` from basis points.
  - On-time is `on_time_bps`, from A45 as distributions paid within 10 business days of the record date (label settled at Specify point b).
  - Proof score is from A34's attestations through the snapshot.
  - Keeps is `keeps_bps`, with a red suffix when below the proposed 10 percent minimum.
  - Flag is `none` grey, `watch` amber, `suspended` red. After A45.1 the API derives it from A33's open watchlist rows and tenant status; the page renders the field and never recomputes it.
  - An input whose provider is absent shows "source missing" in grey (A33's `source: missing`), never a blank or a zero.
- **Flag note (G19).** A row with an open listing pause shows extension B's amber note under the row, for example "Late reports pause new offers (G19)". The pause comes from A33's `tenant_listing_pause`, reason `attestation_overdue`. The note's text is the pause's `reason_text` from A33.1, and a pause for `billing_overdue`, `watchlist_hold`, `licence_expired` or `suspended` shows its own sentence. The page never infers a pause from the numbers.
- Row detail:
  - The monthly snapshot history from `GET /v1/admin/sponsor-metrics` as a small table of the last 12 months with `as_of`.
  - Retained stake per vehicle (`per_vehicle[]`) against its minimum.
  - Complaints open (`complaints_open`, A44) and incidents (`incidents`, A48), each showing "source missing" when the provider is absent.
  - Open watchlist rows with reason, `opened_by` (job or person) and date.
  - Open pauses with their `reason_text`.
  - **Fee and revenue-share agreements**: versions with basis (`flat`, `pct_of_platform_fee`, `transaction_based`), rate, effective dates, document and approval id.
  - Actions from `allowed_actions[]` (A33.1):
    - **Open sponsor**, a link to ADM-33 (A69). Suspension and guardrails live there, not here.
    - **Onboard partner** (PART), a link to ADM-03's admission pipeline (A69). Applicants are pipeline rows there, not tenants here.
    - **Add to watchlist** and **Close watchlist entry** (PART or CCO, reason mandatory, single-actor, audited). An agent's call returns `428 CONFIRMATION_REQUIRED` (A33's `confirm` class), and the row changes only after the human confirms.
    - **Set revenue share**, through `PrivilegedActionFlow` `tenant.revenue_share.set` (6E, PART → CFO + CCO; LEGL is added by the policy row for `transaction_based`, and `REVENUE_SHARE_LEGAL_REVIEW_REQUIRED` renders verbatim).
- **One flag, one source (regression check).** After A45.1, A45's `flag = watch` is by definition "A33 has an open watchlist row for the tenant", and `suspended` is A33's tenant status. A contract test run by this node against the seeded stack compares the two for every tenant after each watchlist or tenant change. It is a regression check on A45.1, not a place where two rules are reconciled; a disagreement is a defect raised to A45, never patched in the page.
- Empty state: "No sponsors admitted yet. Sponsors appear here after admission (ADM-03)." Onboarding applicants such as the product map's Wattle & Stone Living are pipeline rows on ADM-03, not tenants, so they do not appear. The product-map mock's `onboarding` pill is recorded as a DF finding.

**8. ADM-32 Integrations and jobs**

- **Connectors tab.**
  - A `ConsoleTable` over `GET /v1/integrations` with A51's columns: Connector / Latency / Last ok / Sandbox pill / State.
  - Latency is `p95` in mono ms with `p50` in the detail.
  - The environment pill is brick "sandbox" in every local run. `live` is rendered green only if the API returns it, which no code path in this build can produce.
  - State pills: `ok` green, `degraded` amber, `down` red, `unknown` grey. `mcp-server` shows `unknown` until A104's service answers.
  - **Indexer lag.** When A27's `chain.stuck_or_lagging` alert (`indexer_lag_5m`, Sev3) is open, A46 reports the `indexer` row `degraded`. The row shows the amber pill, the lag in seconds from A27's read model and a link to the open A13 alert. The page raises nothing and computes no threshold.
  - Row detail follows A46's list: Connector / Implementation / Environment / Status / Latency (p50 · p95) / Last check / Last error (code and redacted summary) / Config v / Gate (the swap-point human gate and the production candidates from `SWAP-POINTS.md`) / Credential (the `kms` key reference and its creation date, never a value).
  - A 24-hour sparkline of samples from `.../samples` is drawn with A05's chart primitive.
  - **Propose configuration** (ENGO) runs `PrivilegedActionFlow` `integrations.config.change` (4E, ENGO → SECO):
    - The form lists only the variables A46 names for that integration.
    - `config_value_not_allowed` and `CONFIG_NOT_OVERLAYABLE` render before any request exists, naming the variable.
    - After approval the version shows `approved` with the note "applies when the API and worker next start" until the API reports `applied`.
    - The version list keeps rejected and superseded rows.
  - When `banners[]` carries `registry_sync_failed`, the red `Banner` "No on-chain moves without a registry ack · runbook R4" is drawn at the top of every ADM-32 tab, with a link to ADM-17. The sentence comes from A46's banner code through `copy.ts`.
- **Jobs tab.**
  - A `ConsoleTable` over `GET /v1/jobs` with A51's columns: Job / Last run / Duration / Next / State. The detail adds Owner module / Schedule (cron and zone in mono) / Outcome / Trigger / On behalf of.
  - Pills: `ok` green, `failed` red, `skipped_paused`, `skipped_disabled` and `skipped_no_handler` grey with their reason, `cancelled` grey, `paused` amber with `paused_until`, `disabled` grey.
  - Row actions from `allowed_actions[]` (A46.1):
    - **Run now** (ENGO, reason mandatory). A cycle-based job that already ran returns the existing run id, and the page says "Already ran this cycle: RUN-…", not "Started".
    - **Pause job.** It is immediate for ordinary rows. For `critical` rows it runs through `PrivilegedActionFlow` `jobs.schedule.pause` (4E, ENGO → SECO). The form shows the 24-hour maximum and renders `pause_too_long` and `job_paused` verbatim.
    - **Resume** (ENGO, immediate).
    - **Disable**, through `PrivilegedActionFlow` `jobs.schedule.disable` (4E, ENGO → SECO) for every row. **Enable** is immediate for ENGO.
  - A dead-letter list below with A46's columns: ID / Job / First failed / Attempts / Money / State.
    - **Replay** (ENGO, reason). At most three replays; the fourth renders `replay_limit`, and `dead_letter_not_open` also renders verbatim.
    - **Discard.** It is immediate with a reason for `money: false`. For `money: true` it runs through `PrivilegedActionFlow` `jobs.dead_letter.discard` (4E, ENGO → CFO), and the confirm names the consequence the owning module reports (for example "distribution run left in funded").
  - The run history drawer is cursor-paged from `.../runs`. It shows `svc-scheduler` rows with the actor label "Scheduler" and the human as on-behalf-of when one exists, in the `adAudit` pattern.
- **Webhooks tab** (SECO, ENGO; `webhooks:read_all`).
  - A read-only `ConsoleTable` over `GET /v1/webhooks`, platform-wide: ID / Owner (masked by A04's `mask()`, with owner kind) / Token (prefix and name, linking to that token's URL on ADM-35) / Receiver host / Event families (scope-group chips) / State (`active` green, `paused` amber, `failing` amber, `revoked` red) / Last delivery / Consecutive failures.
  - Receiver host is A46.1's `receiver_host`: the host only. The path and any query are never sent to the page, so a receiver's secret path cannot be displayed.
  - The delivery drawer comes from `.../deliveries`: status, duration and the redacted excerpt.
  - A46 gives staff no route to pause or rewrite another person's subscription, so the tab has no such button. Stopping a person's webhooks is done by revoking or suspending the token on ADM-35, and the tab says so.
- **Sandbox tab** (the operator sandbox; ENGO and SECO by `integrations:sandbox`).
  - One `OptionCards` group per adapter that registered a `SandboxControl`, with its `listScenarios()` as the cards and a small form for the payload. Buttons come from data, never a hard-coded list.
  - The seed must offer at least the four panels the plan names: bank "Receive a payment" (and "Return a payment"), identity "Refer a case", registry "Drift the register" (and "Ack an instruction"), and sanctions "Trigger a list update".
  - Each click calls `POST /v1/integrations/{name}/sandbox/{event}`. The result shows the run record, audit id and the downstream effect the API reports; for example, a drift shows "recon break expected at next run, or run recon now", linking to ADM-17.
  - The tab is rendered when `GET /v1/meta` reports `RUN_MODE` `local` or `test` (A46 mounts the route in both) and the console bundle contains the sandbox UI.
  - Per-user and per-tenant sandbox partitions (the sponsor and investor sandboxes of D04) are [[A99 - Developer surface- API versioning, reference docs, per-user sandbox and SDK-MCP packaging]]'s, not shown here.
  - **Build-time switch.** A08 allows only `RUN_MODE=local | test`, so the run mode cannot prove the stripping. The console instead reads a build-time variable `NEXT_PUBLIC_CONSOLE_SANDBOX_UI` (`on` by default, `off` for a release-shaped bundle), in the same spirit as A09's `testControls: false` for production-shaped checks. The variable is read only in this node's owned paths: the sandbox route segment and the Acknowledge (test) control import their code behind a check of the inlined constant, so the bundler removes the dead branch when it is `off`. With `off` the build must succeed, the console must start against the Docker stack, and `/integrations/sandbox` must show the shell's not-found state.
- **Human gates tab.**
  - A `ConsoleTable` over `human_gates[]` (A46.1): Gate / What the human does / Blocks (connectors and nodes) / Evidence / State.
  - All seven gates (`HG-LEGAL`, `HG-VENDOR`, `HG-KEYS`, `HG-AUDIT`, `HG-PENTEST`, `HG-CAPITAL`, `HG-PROD`) show the amber "open" pill in this build, with the sentence "Agents cannot close a human gate. A person closes it with evidence."
  - The tab has no button. A `closed` value renders green with its evidence reference, so the vocabulary is complete, but the seed never contains one.
  - This is the board [[A89 - Docker Desktop launch (P1)]] shows the founder.

**9. Agents and tokens (D04, G22): parity for every action on the three pages**

- **Reads.** Every read on the three pages is an operation of A45, A46 or A33 with `x-realx-scope` in M19, M24 or M26. A staff member's agent with a read mandate lists and reads exactly what its human sees on screen, filtered by the same RLS and the same AML projection rule. The reads include the calendar and summary, filings and packs, grants and activity, exports, prudential, metrics, sponsor performance, snapshots, watchlist, revenue share, integrations, samples, configs, jobs, runs, dead letters, webhooks and human gates. An agent of a non-MLRO, non-CCO human can no more learn that an AML item exists than its human can.
- **Delegability table.** Every button in `actions.ts` has one class from A100's vocabulary (`delegable`, `confirmable`, `maker`, `human_only`, `counsel`). The class is **read from A03's `delegable.ts` and the owners' route tables at Specify** (A45's permissions table, A46's §6 and agent-parity rules, A33's staff route table), never from design data. The expected values today are:

  | Button (screen) | Operation (owner) | Class for a staff agent | Agent refusal, if any |
  |---|---|---|---|
  | Assign; Attach evidence (ADM-26) | `M19.obligation.assign`, `.update` (A45) | `delegable` | — |
  | Mark complete (ADM-26) | `M19.obligation.complete` (A45) | `delegable` today; counsel question below | — |
  | Waive; Edit due date or basis (ADM-26) | `M19.obligation.waive`, `.editSeed` (A45) | `maker` | approve: `APR_CHECKER_MUST_BE_HUMAN` |
  | Create, Mark ready, Withdraw, record rejection (ADM-26) | `M19.filing.prepare` (A45) | `delegable` today; counsel question below for Withdraw | — |
  | Download lodgement pack (ADM-26) | `GET .../filings/{id}/pack` (A45, audited) | `delegable` (read) | — |
  | Submit filing (ADM-26) | `M19.filing.submit` (A45) | `counsel`, behaves as `human_only` | `FIL_PERSON_ONLY` |
  | Grant access; Extend (ADM-26) | `M19.dataRoomGrant.request`, `.extend` (A45) | `maker` | own person as grantee: `DR_SELF_GRANT`; approve: `APR_CHECKER_MUST_BE_HUMAN` |
  | Revoke grant (ADM-26) | `M19.dataRoomGrant.revoke` (A45) | `human_only` (no token scope) | `AGNT_NOT_ALLOWED` (reason code `G22_SCOPE_NON_DELEGABLE`) |
  | Export board pack or report (ADM-26) | `M19.boardPack.export`, `M19.report.export` (A45) | `delegable` (sensitive notice) | — |
  | Change definition; Publish figures (ADM-26) | `M19.metricDefinition.change`, `M19.metricPublication.publish` (A45) | `maker` | approve: `APR_CHECKER_MUST_BE_HUMAN` |
  | Add to watchlist; Close watchlist entry (ADM-29) | `M24.watchlist.open`, `.close` (A33) | `confirmable` (A33 `confirm`) | `428 CONFIRMATION_REQUIRED` until the human confirms |
  | Set revenue share (ADM-29) | `M24.revenueShare.set` (A33) | `maker` | approve: `APR_CHECKER_MUST_BE_HUMAN` |
  | Run now; Replay; Resume; Enable; ordinary Pause (ADM-32) | `jobs:trigger`, `jobs:replay`, `jobs:pause` (A46) | `delegable` | — |
  | Critical Pause; Disable; money Discard; Propose configuration (ADM-32) | `jobs.schedule.pause`, `jobs.schedule.disable`, `jobs.dead_letter.discard`, `integrations.config.change` (A46) | `maker` | approve: `APR_CHECKER_MUST_BE_HUMAN` |
  | Non-money Discard (ADM-32) | `jobs:discard` with `money: false` (A46) | `delegable` today; `confirmable` if Specify point (a) is accepted | `428 CONFIRMATION_REQUIRED` if accepted |
  | Sandbox scenario (ADM-32, local and test only) | `integrations:sandbox` (A46) | `delegable` | — |
  | Any approve, reject or hardware-key confirm (all three) | A14 `decide` | `human_only` | `APR_CHECKER_MUST_BE_HUMAN` (A14 refuses before any scope check) |

  Each maker or delegable act is audited as the agent on behalf of its person and appears in the person's activity (A12's `GET /v1/agent-access/activity`). Sensitive ones reach A101's notice policy. A14's maker identity set counts the agent as its human, so the human can never check a request their agent drafted (`APR_MAKER_IS_CHECKER`).
- **Proposed rule for the founder (Specify point a).** Discarding a non-money dead letter removes evidence of a failure without a second person, so the proposal is that a staff agent's call gets `428 CONFIRMATION_REQUIRED` and takes effect only when the human confirms. This is a data change owned upstream, not code here: A03's `delegable.ts` row for `jobs:discard` with `money: false` becomes `confirmable`, and A12 honours it through the `confirm_always` value A33 already asks for. The watchlist open and close need no proposal: A33 already classes them `confirm`.
- **Counsel questions (recorded, not decided).** Whether marking an obligation complete is a personal compliance attestation, and whether withdrawing a filing is a personal legal act. If counsel says yes, the A03 row becomes `counsel` or `false` by data and this page follows with no code change.
- The page shows the `AgentMarker` on every agent-drafted request, filing, export and watchlist row. It shows the KeyConfirmModal's `agent` state ("Agents prepare. A person approves.") wherever an agent persona would otherwise see an approve slot.
- **Action manifest.** `src/features/reporting-integrations/actions.ts` lists every button on the three pages with its A03 cell, A14 action code and policy, class, the OpenAPI operation id and the run modes and bundle switch it exists in. Class and cell are generated from A03's `delegable.ts` and matrix, not from extension B's `screens.json`. A100's parity suite reads it. A test fails when:
  - a rendered action has no operation id
  - a class in the manifest differs from A03's data
  - an operation marked `human_only` or `counsel` is exposed as an MCP write tool without the refusal
  - a sandbox or test-acknowledge action is listed for a bundle built with the sandbox UI off

**10. Errors, loading and privacy**

- Every refusal from A45, A46, A33, A12 and A14 renders verbatim as a `Banner` with its code and sentence. This includes:
  - A45: `OBL_EVIDENCE_REQUIRED`, `OBL_FIELD_LOCKED`, `OBL_READ_ONLY_SOURCE`, `FIL_TRANSITION_NOT_ALLOWED`, `FIL_EVIDENCE_REQUIRED`, `FIL_LODGEMENT_REF_REQUIRED`, `FIL_KIND_OWNED_ELSEWHERE`, `FIL_PERSON_ONLY`, `DR_GRANTEE_NOT_PARTNER`, `DR_WINDOW_TOO_LONG`, `DR_PRINCIPAL_INACTIVE`, `DR_SELF_GRANT`, `REP_NO_AUDITED_FIGURES`, `MET_RENDER_REFUSED`, `MET_FORBIDDEN_LABEL`
  - A33: `REVENUE_SHARE_LEGAL_REVIEW_REQUIRED`, `TENANT_SUSPENDED`
  - A46 and A08: `pause_too_long`, `job_paused`, `replay_limit`, `dead_letter_not_open`, `config_value_not_allowed`, `CONFIG_NOT_OVERLAYABLE`
  - A11 and A12: `FORBIDDEN`, `AGNT_NOT_ALLOWED`, `G22_SCOPE_MISSING`, `G22_SCOPE_NON_DELEGABLE`, `G22_SCOPE_PENDING_COUNSEL`, `G22_AGENT_NEVER_CHECKER`, `G22_LIMIT_EXCEEDED`, `428 CONFIRMATION_REQUIRED`, `423 AGENTS_PAUSED`, `423 AGENT_HELD`, `429 RATE_LIMITED`
  - A14: `APR_MAKER_IS_CHECKER`, `APR_ROLE_NOT_HELD`, `APR_CHECKER_MUST_BE_HUMAN`, `APR_EXPIRED`

  A new code in any of those OpenAPI catalogues without a sentence in `copy.ts` fails a generated test.
- Loading skeletons and the empty, error (with `request_id`), forbidden and session-expired states come from A53's primitives. After any mutation the page refetches the affected row and the summary; it never edits a row optimistically, and no success toast appears before a 2xx.
- A 404 from the AML projection, or on any id the caller cannot see, renders as the shell's not-found state, never "forbidden". The distinction would itself leak existence.
- PII: the three pages need no investor name, email, TFN or case narrative. Subjects are typed ids, and webhook owners are masked by `mask()`. Nothing on the pages unmasks.
- URLs carry ids, tab names and filters only, never a purpose text, reason, lodgement reference or grantee name. No `localStorage` or `sessionStorage` use beyond A53's column picker.
- All dates are rendered in Australia/Sydney from server timestamps. Business-day counts and hour countdowns come from the API (A13 via A45 and A42), never from client arithmetic. `npm run verify` greps the owned paths for `Date.now()` used in a due-date or countdown computation.

## Owned paths (other nodes do not edit these)

- <workspace>\app\apps\console\app\(reporting)\
- <workspace>\app\apps\console\src\features\reporting-integrations\

By convention:
- `app/(reporting)/reporting/{calendar,filings,data-room,reports,metrics}/page.tsx`
- `app/(reporting)/partners/sponsors/page.tsx`
- `app/(reporting)/integrations/{connectors,jobs,webhooks,sandbox,human-gates}/page.tsx`
- `[id]/` detail routes under each
- `src/features/reporting-integrations/`, which holds:
  - `queries.ts` (SDK calls only)
  - `actions.ts` (the action manifest)
  - the feature folders `calendar/`, `filings/`, `data-room/`, `reports/`, `metrics/`, `sponsors/`, `connectors/`, `jobs/`, `webhooks/`, `sandbox/` (removed from a bundle built with `NEXT_PUBLIC_CONSOLE_SANDBOX_UI=off`) and `human-gates/`
  - `badges.ts`, `search.ts`, `copy.ts` (every sentence and error mapping) and `index.ts`

API routes are A45's (with loop A45.1), A46's (with loop A46.1) and A33's (with loop A33.1). Shell primitives are A53's; blocks are A51's and A05's. A missing primitive or block is requested from its owner as a follow-up loop, never drawn here.

## Design references

- **`design-ext:ADM-26 Regulatory calendar and data room`** from [[A51 - Design extension B- operator console screens not in the prototype]] (`<workspace>\app\design-ext\admin\RealX Admin Ext.dc.html`, anchor in `screens.json`):
  - The obligations DataTable Obligation / Regulator / Due / Owner / State, with the AMMA by 30 Sep, the outside-TMD report at 10 business days in P2, the SMR at 3 business days (an AML projection row, MLRO and CCO only), licence conditions and breach reporting.
  - The data-room KeyValueList (auditor and regulator grants, time-boxed, CCO → SECO).
  - "Submit filing" (LEGL or CCO) through the modal.
  - The metric-governance line "public numbers only from audited figures (N16)".
  - Three drawn states: default, overdue, empty.
  - A45's design note adds the Filings, Data room and Metrics tab layouts and the Evidence column. The Filings, Reports and Metrics tabs, the Evidence column and the grant form are composed from the same blocks (DataTable, KeyValueList, TaskRows) and listed as DF findings for the next HG-REVIEW rather than invented silently.
- **`design-ext:ADM-29 Sponsor performance`** (A51): DataTable Sponsor / Trusts / Raised / Yield plan vs paid / On-time reporting percent / Proof score / Keeps ≥10 percent stake / Flag; the amber "late reports pause new offers (G19)" note on a flagged row; "Open sponsor" as the only row action. Drawn states: default, one sponsor flagged, empty.
  - The frame's label "On-time reporting percent" and A45's field (distributions paid on time) mean different things. The label rendered is settled at Specify (point b below) and recorded as a DF finding.
  - The fee and revenue-share detail, complaints and incidents, the "Onboard partner" link and the watchlist actions are not drawn. They are composed from KeyValueList and the modal, as DF findings.
- **`design-ext:ADM-32 Integrations and jobs`** (A51):
  - Two DataTables. Connectors: Identity/DVS, sanctions, bank rail NPP/PayID/PayTo, custody, registry, chain RPC, email/SMS, MCP server, with Latency / Last ok / Sandbox pill (brick "sandbox" in every local run) / State. Jobs: reconciliation 02:00, register sync, screening refresh, expiry sweep, webhook delivery, anomaly baseline, with Last run / Duration / Next / State.
  - "Run now" and "Pause job" (ENGO, through the modal).
  - The red banner "No on-chain moves without a registry ack · runbook R4".
  - Drawn states: default, connector down, job failed, empty.
  - The dead-letter list, webhooks, sandbox panel (OptionCards, per A46's design note) and human-gate board are composed from those blocks and listed as DF findings.
- **Prototype regions in `<workspace>\design\RealX App.dc.html` that set the language:**
  - The admin shell grid, dark sidebar and user card (lines 621-624) and `adNavDef` (line 862), through A53.
  - `adCmd` (line 880): KPI tiles with a mono value and a subline such as "Reconciled 06:00", the shape of every header `KpiGrid` here. Its subtitle "all figures reconciled at 06:00 AEST" is why the seeded `recon.nightly` run shows last run 06:00 on 23 Sep 2026 on the Jobs tab while its schedule reads 02:00.
  - `adRecon` (line 885): the "Last run" column and "Run 06:00" subline pattern for the jobs table.
  - `adAml` (line 887): the subtitle "Suspicious matter reports go to AUSTRAC within 3 business days", the wording standard for the `smr_3bd` projection row.
  - `adAudit` (line 888): Time / Actor / Action / Object / Approval / Result with the last column a StatusPill, the pattern for the data-room activity and job run history.
  - `adApprovals` (lines 655-676): the split list and detail, and the empty state "Inbox clear." (line 665, dashed `#D6D0C4`, text `#5E6166`) that every empty state here copies in form.
- **Blocks** from design analysis §2: DataTable (34), KpiGrid, KeyValueList, TaskRows, OptionCards, RaiseBar (11), ApprovalListCard and ApprovalDetail (38); KeyConfirmModal and SplitListDetail from A51's `blocks.md`.
- **Tokens:**
  - Colours: cream `#F5F2EC`, ink `#17191C`, green `#1F5E4B`, brick `#9A4524`, alert red `#A3312A`, amber tint `#F6ECD6`, and the dark shell surfaces `#1C1E22` / `#2A2D31` / `#34383D`.
  - Fonts: Newsreader for h1s and sponsor names; Geist for text; Geist Mono for `OBL-`, `FIL-`, `DRG-`, `REP-`, `DLQ-`, `WHS-`, `APR-` and `PRN-` ids, hashes, cron strings, latencies, every figure and every date in tables.
  - Status tones through A05's `toneFor`, with every enum value above mapped. `sandbox` and `P2` are brick (the tone the design uses for "2027" previews).

## Requirements covered

- **Product-map screens:**
  - **ADM-26 Regulatory calendar and data room.** "Done" means:
    - Every component of the product map's row is on screen with live data from A45: the calendar at 7/30/90 days with overdue, the filing workspace, the prudential dashboard, the regulator data room with scoped, time-boxed access, investor and board reports, and metric definitions where "capital raised" is not "tokenised value".
    - Its four actions run through the API with their policy codes: Submit filing (LEGL or CCO), Grant access (`M19.dataRoomGrant.request`, CCO → SECO), Publish KPIs (`M19.metricPublication.publish`, IRCM → CFO + CCO, G17), Change a metric definition (`M19.metricDefinition.change`, CFO → CCO).
    - The state "Overdue filing: Sev2 to CCO" is visible as a red row whose A13 alert is linked.
  - **ADM-29 Sponsor performance.** "Done" means the sponsor table with performance and risk signals (including complaints and incidents), the watchlist and the fee and revenue-share agreements are on screen from A45 and A33. "Set revenue share" runs as a 6E with the legal-review rule. "Onboard partner" and "Suspend sponsor" link to their owning screens ADM-03 and ADM-33 (A69), as A51's frame decides. The "two late distributions in 6 months: watchlist (proposed)" state appears as A33's job-opened watchlist row, and the flag follows it.
  - **ADM-32 Integrations and jobs.** "Done" means:
    - Connectors (identity, bank/NPP, unit registry, custody, chain RPC and indexer, e-sign, plus the rest of A08's manifests), webhooks, scheduled jobs and the operator sandbox panels are on screen from A46. The product map's sponsor sandboxes are A99's per-user and per-tenant partitions and are covered there, not here.
    - "Switch RPC or network config" runs as ENGO → SECO with config v+1.
    - "Indexer lag above 5 min: Sev3" shows as the `indexer` row `degraded`, with A27's alert linked (challenged below).
    - The product map's API key and MCP write-tool rows are reached through the link to ADM-35 (A77), which draws them; whether write tools are enabled is A104's spec point (a).
    - The human-gate board shows all seven gates as open.
- **Journeys: J20 Operator daily run.** This node owns three steps: step 5 (ENGO checks connectors and jobs on ADM-32), step 9 (PART reviews sponsor performance on ADM-29, G19) and the compliance-calendar part of the run (CCO or LEGL on ADM-26, with the prudential card beside ADM-01 for step 1). "Done" means A78's Playwright run can, using only the seeded data and the sandbox authenticator:
  - take ENGO through ADM-32: see a planted failed job, replay its dead letter, and see a paused critical job's 4E request waiting for SECO;
  - take PART through ADM-29 to the flagged sponsor and its pause note;
  - take CCO through ADM-26: see an overdue obligation, and have a second person (SECO) approve a data-room grant with a hardware key on the card.

  It also means A80 can drive J14's reporting breach and J21's final report to the regulator through these pages.
- **Gates:**
  - **G13 Maker-checker.** Done when waiver, seed edit, data-room grant and extension, metric definition change, publication, revenue share, critical pause, disable, money discard and configuration change move only through `PrivilegedActionFlow` and `HardwareKeyConfirm` with the eleven codes in §1. The maker and the maker's agents never see an enabled approve. Every single-actor act (complete, submit, export, revoke, watchlist, trigger, replay, resume, non-money discard) writes its audit event with the actor.
  - **G14 AML monitoring and reports.** Done when the SMR, TTR and IFTI clocks appear on the calendar as A45's projection for MLRO and CCO only, read-only and linked to ADM-25, each in its own unit (business days, or hours for terrorism financing); and when no AML item is visible (no row, count, badge, search hit, tile change or placeholder) to any other role or any agent of those roles. Deciding and filing an SMR are A42's on ADM-25.
  - **G17 Marketing claims approved.** Done when "Publish figures" can be requested only through `M19.metricPublication.publish`, the precheck's management-only keys, forbidden labels and A43 claims flags render verbatim (`MET_FORBIDDEN_LABEL`), no path submits past them, and no forbidden label appears in the owned paths. A45 and A43 enforce it.
  - **G19 Proof of property.** Done when ADM-29 shows the proof score from A34's attestations and the API's listing-pause sentence for any tenant paused for overdue attestations, and never shows a pause the API did not report.
  - **G21 Licence perimeter.** Done when every licence, CAR appointment and condition date from A21 appears on the calendar as a read-only `perimeter` row with its source and a link to ADM-04. No connector can be shown as `live`, and no human gate as closed, from the real seed and API in this build.
  - **G22 Agent authority.** Done when every button's class in `actions.ts` equals A03's data, and every agent refusal on these pages is the code the §9 table names.

  G13, G14, G17, G19, G21 and G22 are enforced by A45, A42, A43, A46, A33, A14, A12 and A21. This node proves the pages cannot weaken them.
- **Modules:**
  - **M19 Reporting and filings** (the screen half). Done when the pack's calendar, filing workspace, prudential dashboard, data room, reports and metric definitions are on screen with live data.
  - **M24 Partners** (the sponsor half). Done when sponsor performance and track record, the watchlist and fee agreements are on screen. Advisers and distributors are later phases and are not drawn.
  - **M26 Integrations and API** (the ADM-32 half). Done when connectors, jobs, dead letters, webhooks, the operator sandbox panels and the human-gate board are on screen. API keys, agent tools and personal tokens are the ADM-35 half (A77, with A104), linked from here.
- **State machines:** none of the 14 owned. The page renders these local lifecycles, and each has a label and a tone for every value, checked by a test that iterates the OpenAPI enums:
  - A45's obligation, filing and grant lifecycles, and the AML projection's mapped states
  - A46's job outcome, dead-letter, integration status, config version and subscription lifecycles
  - A33's watchlist and pause reasons
  - the human-gate `open | closed`
- **Founder requirement D04.** "Done" means:
  - every action on the three pages has an API operation;
  - the `delegable` and `maker` ones work under a staff token with the same refusals as the screen, and the `confirmable` ones return `428 CONFIRMATION_REQUIRED` until the person confirms;
  - the `human_only` and `counsel` ones refuse any agent before any effect, with the code in §9;
  - every agent-made item carries the `AgentMarker` and appears in the person's activity from A12's `GET /v1/agent-access/activity` (the same list ADM-35 shows, checked on screen at A78);
  - a partner's agent reading a data-room grant appears in the grant's activity as the token on behalf of the person.

## Gauntlet loop for this node

- **Specify.** Frozen inputs:
  - this note
  - A45's routes, enums, error codes, permissions table and seeded catalogue, with loop A45.1 applied (`allowed_actions[]`, AML clock unit, `flag` from A33)
  - A46's routes and enums with loop A46.1 applied (`human_gates[]`, `allowed_actions[]`, `receiver_host`)
  - A33's staff route table with loop A33.1 applied (`allowed_actions[]`, pause `reason_text`)
  - A03's matrix cells for M19, M24 and M26, the `delegable.ts` rows for every operation in §9, and the policy rows for the eleven approval codes in §1
  - A12's refusal catalogue, `428 CONFIRMATION_REQUIRED` shape and activity route
  - A14's `GET /v1/approvals` and its refusal order
  - A53's shell exports
  - A51's ADM-26, ADM-29 and ADM-32 frames after HG-REVIEW
  - A06's staff personas and seed: Jordan Blake CCO, Aisha Karim SECO, Ben Cole ENGO, Helen Tran CFO, Marcus Bell CEO and Lena Moss SUPER, plus the MLRO, LEGL, PART, IRCM and TAXO personas; the three design sponsors Harbour & Co Property, Northline Capital and Yarra Bend Partners; one open A42 SMR item on `smr_3bd` and one on `smr_tf_24h`; one overdue obligation; one A45 filing in `ready`; one active and one expired data-room grant for a seeded TP-AUD; one sponsor on the watchlist with an `attestation_overdue` pause; one failed job with an open money dead letter; and one connector the seed can stop

  A06's owner adds any of these that are missing. **Specify stops, and the owner is reopened before the node QA writes the hidden tests, if** A45 still computes `flag` from its own rule instead of A33's watchlist and tenant status, or if A45.1, A46.1 or A33.1 is missing any field named in §1. Three points are settled here before the node QA writes the hidden tests:
  - (a) whether discarding a non-money dead letter by a staff agent needs the human's confirmation (proposed: yes, as the A03 `delegable.ts` change in §9, with A12's `confirm_always`);
  - (b) the ADM-29 "On-time" label: A51's "on-time reporting percent" or A45's distributions on time (proposed: show A45's field as "Distributions on time" and add the proof score's on-time attestations as the reporting measure; the founder decides at HG-REVIEW);
  - (c) whether ADM-29 may show a management "Raised" figure to staff with a label (proposed: no, "awaiting audit" only, so the console can never be screenshotted into a public claim).

  The node QA records `specVersion`, the OpenAPI hash covering the A45, A46 and A33 routes, the A03 data hashes and `hiddenCasesHash` in the manifest. Its expected agent outcomes come from A03's data as frozen here, never from `screens.json`.
- **Build (Grok 4.7 xhigh, split by the planner).** Suggested shape: one lead session wires the route group, the three screens' tab layouts, `queries.ts` against the SDK, the `NEXT_PUBLIC_CONSOLE_SANDBOX_UI` switch and the action manifest first. Then three parallel sub-agents work on disjoint folders:
  - (a) `calendar/`, `filings/`, `data-room/`, `reports/`, `metrics/` (ADM-26);
  - (b) `sponsors/` (ADM-29, the watchlist and revenue-share flows, the flag regression test);
  - (c) `connectors/`, `jobs/`, `webhooks/`, `sandbox/`, `human-gates/` (ADM-32, the bundle-stripping check for the sandbox folder).

  The lead then writes `badges.ts`, `search.ts` and `copy.ts`. Integration covers:
  - Playwright specs for every state at 1280 and 1024 px with the seeded personas
  - the error-catalogue sentence test and the enum tone test
  - the action-manifest parity test against A03's data
  - a planted-failure script (stop the registry sandbox, fail a job, drift the register, lag the indexer past 5 minutes)
  - a release-shaped build with `NEXT_PUBLIC_CONSOLE_SANDBOX_UI=off`, started against the stack
  - axe and `npm run verify`
  - `app/evidence/A76.json`: routes with `screen` and `designKey`, screenshots, error-code coverage, the action manifest hash and the DF findings list
- **Challenge (node QA, Opus 5.5 high).** The node QA:
  - runs the hidden specs;
  - drives every action as each seeded persona and as each persona's agent token (through the SDK and the MCP server, not the page) and compares outcomes with the screen and with A03's classes;
  - runs the node-specific challenges below, and the data-room script under the injectable clock across a grant's `ends_at`;
  - stops and restarts sandbox adapters and plants failed jobs, dead letters and indexer lag;
  - builds the console with `NEXT_PUBLIC_CONSOLE_SANDBOX_UI=off`, requires the build to succeed and the console to start, and inspects the bundle and routes;
  - compares the pages with its crops of extension B at token level;
  - greps the owned paths for hard-coded rule text, forbidden metric labels, `localStorage`, raw hex, client business-day or hour arithmetic, role checks that decide a button, and SDK decision operations outside A53's shell;
  - confirms only the owned paths changed.

  It writes `app/evidence/A76.qa.md`.
- **Gate.** Pass unlocks A78 and A80. Max three fix loops; each re-runs the hidden specs and the failing challenges.
- **Escalation trigger.** Any one of these escalates:
  - the same finding fails a third time;
  - any path where an AML item's existence reaches a non-MLRO, non-CCO principal or agent through the page, the search, a badge, a count or an error;
  - any path where the maker or the maker's agent approves a grant, publication, revenue share, critical pause, disable, money discard or configuration change;
  - any path where, **with the real seed and the real API** (not a hostile fixture), the page shows `live`, a closed human gate, or a management figure as audited;

  Those three path findings are Critical and stop the milestone. The hostile fixtures in the challenges below are expected to render what the fixture says; there the finding is the contract test's, not the page's. So do:
  - a fix that needs a change to A45's, A46's or A33's contract (a spec defect, reopened upstream);
  - a Specify point the founder has not answered;
  - an extension-B frame that cannot be built from A05 and A51 blocks.

## Node-specific challenges (node QA)

Every refusal below is phrased the same way: the persona has no button on the page (it is not in `allowed_actions[]`), and a direct SDK call returns the code named.

- **The AML clock that leaks by counting (compliance, G14).**
  - Setup: one A42 SMR item on `smr_3bd` is due tomorrow, and one on `smr_tf_24h` has 3 hours left. The node QA signs in as LEGL, CFO, SUPER, TAXO, ENGO and PART in turn, and as each one's agent token.
  - Probes for each: the Calendar and Filings tabs, the calendar summary tiles, the `reporting.attention` badge, global search for the A42 ids, the calendar id URL typed directly, the prudential card ("open and overdue filings"), the data-room collection picker, and ADM-26's error state after a forced `GET`.
  - Expected: no row, no count difference against a seed without the items, no search hit, and the shell's not-found state (not forbidden) on the id. As MLRO and CCO the two rows are there with typed reference and clock only, no action, and a link to ADM-25; the summary tiles are the same as for everyone else.
  - The `smr_tf_24h` row counts down in hours and minutes, never in business days, and turns red "overdue" at exactly 24 elapsed hours under the injectable clock, including across the October 2026 daylight-saving change.
  - Fail condition: any number that changes when the AML items are added, or any AML clock shown in the wrong unit.
- **The filing lodged by the wrong hand (compliance, G14 and G22).**
  - Attempts: CFO (no submit cell) on a `ready` filing; CCO's agent submits it with a lodgement reference; LEGL submits it without a lodgement reference; LEGL submits a `draft` filing; an SDK call creates a filing with `kind: smr`; LEGL submits the `ready` filing properly.
  - Expected: no Submit button for CFO and `FORBIDDEN` from the SDK; `FIL_PERSON_ONLY` for the agent, with no state change; `FIL_LODGEMENT_REF_REQUIRED`; `FIL_TRANSITION_NOT_ALLOWED`; `FIL_KIND_OWNED_ELSEWHERE` naming A42; then success. The overdue red and its Sev2 clear only after the API reports `submitted`. No success toast before the 2xx.
  - In `RUN_MODE=test` there is no Acknowledge (test) button, and in the bundle built with the sandbox UI off it is absent from the code.
- **The data room that outlives its grant (security).**
  - Setup: under the injectable clock, a TP-AUD grant for Harbour & Co, `register_extracts` only, 1 to 30 October 2026.
  - Checks: the card counts down from the server's `ends_at`. On reload at 23:59:30 and 00:00:30 Sydney it reads `active`, then `expired`.
  - Attempts: SECO revokes a second grant and the card must read `revoked` only after the 2xx. CCO's agent tries to revoke a grant. Through the SDK (the picker lists partners only), CCO requests access for an investor and for a sponsor user; on the page CCO asks for 120 days. CCO names herself as grantee. CCO approves her own grant request. CCO's agent approves it.
  - Expected: `AGNT_NOT_ALLOWED` (reason code `G22_SCOPE_NON_DELEGABLE`) for the agent's revoke; `DR_GRANTEE_NOT_PARTNER` twice; `DR_WINDOW_TOO_LONG`; `DR_SELF_GRANT`; the modal's `not-eligible` state for CCO and `APR_MAKER_IS_CHECKER` from the SDK; `APR_CHECKER_MUST_BE_HUMAN` for the agent. The activity list shows a partner agent's read as "agent for <person>". No grant's purpose text or grantee name appears in any URL.
- **The number that was never audited (compliance, G17 and N16).**
  - Setup: the fixture gives Northline a management `capital_raised`.
  - Expected: ADM-29 shows "awaiting audit" for Raised, never the number or 0. A provider that is absent shows "source missing", including complaints and incidents in the row detail. The Metrics tab shows the management value with the grey pill. IRCM's Publish is refused at precheck naming `capital_raised`. A board pack preview keeps the "management figure, unaudited" label. An investor report export is refused with `REP_NO_AUDITED_FIGURES` naming the key.
  - Then the node QA plants "Total value tokenised" in a definition name through the API: the page renders A45's `MET_FORBIDDEN_LABEL` banner, the Publish request cannot be submitted, and the grep finds no forbidden label in the owned paths.
- **One flag, one source (correctness, G19).**
  - Setup: the node QA opens a watchlist row by hand for Yarra Bend as PART, closes Harbour & Co's job-opened row as CCO, and lets an attestation go overdue for Northline so A34 sets `attestation_overdue`.
  - Expected: after each step, ADM-29's Flag column equals A33's watchlist and tenant status (A45.1), and the regression test passes. The pause note appears for Northline with A33's `reason_text` and not for Yarra Bend, whose watchlist row alone blocks nothing.
  - PART's agent tries to close Yarra Bend's entry: `428 CONFIRMATION_REQUIRED`, and nothing changes until PART confirms with a hardware key; then the row closes once, audited as the agent on behalf of PART.
  - Fail condition: any divergence between `flag` and A33's rows fails the regression test by name and is raised to A45.
- **Pausing reconciliation quietly (compliance, G13).**
  - Attempts: ENGO presses Pause on `recon.nightly` (a critical job) for 24 hours; ENGO's agent submits the same; SECO approves ENGO's request; ENGO pauses for 25 hours; ENGO presses Disable on `screening.refresh`; ENGO discards the money dead letter for the failed payout run alone, then with CFO; ENGO's agent approves the disable request.
  - Expected: a 4E request (no pause yet, amber `pending approval` with the `APR-` id); the agent's request is created as maker and SECO sees the `AgentMarker`; the row shows `paused` with `paused_until` only after the API reports it; `pause_too_long`; a `jobs.schedule.disable` 4E request, with the row still enabled until SECO approves; the discard waits for CFO and succeeds with CFO's hardware key, and the confirm names "distribution run left in funded"; `APR_CHECKER_MUST_BE_HUMAN` for the agent's approve.
  - Fail condition: any `paused` or `disabled` pill before approval.
- **The indexer that falls behind (operations).**
  - Setup: under the injectable clock, A27's sandbox indexer stops advancing for 6 minutes.
  - Expected: within one refresh, the `indexer` row shows `degraded` with the lag from A27's read model and a link to the one open `chain.stuck_or_lagging` Sev3 alert; no second `integration.degraded` alert exists. When the indexer catches up, the row returns to `ok` within one refresh.
- **Sandbox where it must not be, live where it cannot be (security, G21).**
  - Bundle: the node QA builds the console with `NEXT_PUBLIC_CONSOLE_SANDBOX_UI=off`. The build must succeed and the console must start against the stack. Then the bundle and routes must contain no `sandbox` feature code, no Acknowledge (test) control and no `/integrations/sandbox` page, and that URL must show not-found. A build that fails or does not start is a failure, not a pass.
  - Hostile fixtures: an intercepting fixture makes `GET /v1/integrations` return one row with `environment: live`, and `human_gates[]` return eight gates, one `closed`. The page renders exactly what the fixture says, and the contract test must flag both fixtures as contract violations (it fails on `live` and on any gate not in the seven). With the real seed, no `live` and no `closed` pill ever appears.
  - Stopping the registry sandbox must show the red R4 banner on every ADM-32 tab within one refresh, and restarting it must clear the banner.
- **Configuration that reaches for mainnet (security, G13).**
  - Attempts: ENGO opens Propose configuration for `chain-rpc` and types `CHAIN_ID=8453`; then `84532` with the allow-listed host; SECO approves on the page.
  - Expected: `config_value_not_allowed` naming `CHAIN_ID` renders under the field with no request created; the v3 request is returned by `GET /v1/approvals?mine=checker` for SECO (A66's inbox view of it is A78's check); the version shows `approved` with "applies when the API and worker next start", then `applied` with its hash and approval id after the stack restarts.
  - Fail condition: a form that offers any variable A46 does not list for that integration.
- **Buttons the API did not offer (G22, all three screens).**
  - For each persona, the node QA picks one action whose A03 cell the persona lacks (for example TAXO and Waive, PART and Propose configuration, CFO and Submit filing). The page shows no button, and a direct SDK call returns the same refusal. It then flips one `allowed_actions[]` entry off in an intercepting fixture while the persona holds the cell: the button must disappear, which proves the page reads the field and not the role.
- **Design fidelity.**
  - With the standard seed, the calendar table with an overdue row, the data-room card, the ADM-29 table with one flagged sponsor and its amber note, and the ADM-32 connectors and jobs tables with a connector down and a job failed must match extension B's frames at token level (font family, size, weight, colours, radii, borders, paddings within 1 px) at 1280 and 1024 px, with no horizontal page scroll at 1024.
  - Every status value in the OpenAPI enums has a tone. The empty states use the dashed pattern from `adApprovals`. The Jobs tab shows `recon.nightly` last run 06:00 on 23 Sep 2026 with schedule 02:00.

## Pass criteria

- **Hidden specs:** The node QA's hidden Playwright specs in `<workspace>\gauntlet\hidden\A76\` pass 100 % on three consecutive runs against the Docker stack with the sandbox authenticator, at 1280 px and 1024 px.
- **AML sealing:** a generated test over every staff persona, each persona's agent token and every read surface of the three pages finds zero observable difference, for anyone but MLRO and CCO, between a seed with the two AML items and one without. Surfaces: rows, tiles, badges, search, prudential card, id URLs, errors and counts. For MLRO and CCO, the tiles and badge are also unchanged, and each AML row shows its clock in the unit A42 uses.
- **Gate enforcement:** for every seeded persona and each persona's agent, a generated test drives every action in `actions.ts` through both the page and the SDK. Outcomes agree with A03's matrix, A03's `delegable.ts` classes and A14's policy rows in 100 % of cells. There are zero paths where the maker, the maker's agent or any agent approves. Every `human_only` or `counsel` act refuses every agent before any effect, with the code in §9. Every `confirmable` act returns `428 CONFIRMATION_REQUIRED` to an agent and changes nothing until the person confirms. Every single-actor act leaves exactly one audit event with actor and on-behalf-of. No button is rendered that is not in `allowed_actions[]`.
- **State coverage:** these states each have a screenshot in `evidence/A76/` listed in `app/evidence/A76.json`:
  - every drawn state of extension B's frames: ADM-26 default, overdue and empty; ADM-29 default, one flagged and empty; ADM-32 default, connector down, job failed and empty
  - every value of the obligation, AML projection, filing, grant, export, metric basis, flag, pause reason, integration status, environment, job outcome, dead-letter, config version, subscription and human-gate enums
  - the `indexer` row `degraded` with its linked alert, and an `smr_tf_24h` row in hours
  - the shell's loading, error, forbidden, not-found and session-expired states

  Every error code in §10 renders a sentence (generated from the OpenAPI catalogues).
- **Clocks and windows** under the injectable clock:
  - A data-room grant leaves `active` within one refresh of its `ends_at`, and a revoke shows within one refresh of the 2xx.
  - An overdue obligation, filing or AML row turns red at the instant the API reports it and not before, including the 24-hour terrorism-financing clock.
  - The countdowns match the server's values to the minute across both daylight-saving changes of 2026-2027.
- **Environment honesty:**
  - With the standard seed, 100 % of connector rows show `sandbox` and all seven human gates show `open`.
  - A console built with `NEXT_PUBLIC_CONSOLE_SANDBOX_UI=off` builds, starts, and contains no sandbox route, component or test-acknowledge control (bundle inspection).
  - The contract test refuses `live` and any unknown gate code.
  - The R4 banner appears and clears with the registry adapter's state, and the `indexer` row follows A27's lag alert.
- **Agent parity:** the action manifest lists every rendered action with an OpenAPI operation id, class (equal to A03's data) and run modes. A100's reader finds zero actions without an operation and zero `human_only` operations callable by a token without refusal. Every agent-made item on the pages carries the `AgentMarker` and appears in A12's activity for the person.
- **Design fidelity:** token-level comparison with extension B's ADM-26, ADM-29 and ADM-32 frames shows no drift beyond 1 px. The owned paths contain:
  - no raw hex
  - no hard-coded rule, policy or pause text
  - no forbidden metric label
  - no client business-day or hour arithmetic
  - no role check that decides whether a button is shown
  - no `localStorage` beyond A53's column picker
  - no SDK decision operation outside A53's shell folders

  axe reports zero critical or serious findings, `npm run verify` is green, only the owned paths changed, and every undrawn tab and form is listed as a DF finding.

## Human gates touched

- None crossed. The node runs on:
  - the sandbox authenticator with an allow-listed AAGUID
  - the seeded personas
  - sandbox adapters
  - the Docker stack

  No regulator portal is called; lodgement references are typed by a person and closed by the local test endpoint. No real auditor, regulator observer or licensee gets access. No production key, vendor account, real money or mainnet is touched.
- **HG-REVIEW (before this node starts):** the founder reviews A51's ADM-26, ADM-29 and ADM-32 frames and answers Specify points (a) and (b). The undrawn tabs and forms, listed as DF findings, go to the next review.
- **The human-gate board itself:** HG-LEGAL, HG-VENDOR, HG-KEYS, HG-AUDIT, HG-PENTEST, HG-CAPITAL and HG-PROD are displayed as open and are never closed by this node or any agent. [[A89 - Docker Desktop launch (P1)]] relies on this board for the founder's HG-LAUNCH sign-off.
- **HG-LEGAL (counsel questions recorded, not decided here):**
  - whether any role beyond MLRO and CCO may ever know an AML item exists
  - whether an AML filing may ever reach an auditor's or regulator observer's data room (default never)
  - the obligation catalogue, its due dates and clock lengths
  - whether data-room access for a real regulator needs a written request or instrument on file before the grant
  - whether a filing submission is a personal legal act (default person-only, `FIL_PERSON_ONLY`)
  - whether marking an obligation complete is a personal compliance attestation, and whether withdrawing a filing is a personal legal act (default: delegable as A45's data says)
  - whether transaction-based revenue share with a sponsor is permissible under the conflicted-remuneration rules (A33's question)

  Answers change A45, A33 or A03 data, not this page.
- **HG-VENDOR / HG-KEYS / HG-PROD (at go-live, not now):** the connectors move from `sandbox` to vendor implementations only after contracts, keys and the production run mode exist. Until then ADM-32 shows each connector's gate as open. HG-AUDIT: until a real auditor attests, ADM-29's Raised column reads "awaiting audit".
- **HG-PENTEST (later milestone):** the data-room grant and revoke flows, the AML-projection behaviour of the pages and the configuration-change form are in scope of the external penetration test. This node passes on software evidence only.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
