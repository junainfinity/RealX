---
title: "A66 - Console: command center, alerts centre, approvals inbox, audit log"
created: 2026-09-24
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am7]
node: A66
milestone: AM7
lane: "Operator console"
oracle: yes
human_gate: none
---

# A66 - Console: command center, alerts centre, approvals inbox, audit log

**The four Overview and records screens of the operator console, built on A53's shell with no business rule of their own. ADM-01 Command center shows role-filtered tiles: the design's four KPIs, operations queues with SLAs, compliance clocks, chain and custody health, treasury with bank vs ledger Δ that must be 0, and live alert rows. Every figure comes from one field of one SDK read. ADM-02 Alerts centre sits over A13: severity SLAs Sev1 15 min, Sev2 1 h, Sev3 1 business day; acknowledge, assign, escalate, snooze, resolve and reopen with the reasons A13 demands; bulk acknowledge for Sev3 and Sev4 only; the 07:00 digest. ADM-30 Approvals inbox sits over A14: diff against the last approved payload, simulation, file hash, evidence, the SoD line driven by A14's per-caller decision, custody binding, a TTL countdown to 72 h, the "agent on behalf of" marker, and approve, reject, veto, ratify and low-risk bulk approve only through the hardware-key modal. ADM-31 Audit log sits over A10: the hash-chain verification banner, an on-behalf-of column, row hashes and audited exports. Every action here has a named API operation, so a staff member's agent can do through the API everything this screen lets its human do, except the acts their owning nodes keep human-only: the checker acts (A14), declaring an incident (A48), and audit verify-now and legal holds (A10).**

**Build status:** not started. **Milestone:** AM7 (Operator console). **Lane:** Operator console. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 10-14 agent-hours (build, challenge and verify loops).

**Depends on:** [[A53 - Operator console shell, hardware-key sign-in and approval action pattern]], [[A14 - Approvals engine and maker-checker (SM-00)]], [[A10 - Append-only hash-chained audit log]], [[A20 - Three-way reconciliation engine with zero tolerance]], [[A46 - Job scheduler, integrations health and webhooks]], [[A13 - Alerts and SLA-clocks service]], [[A06 - Seed data and scenario simulator]], [[A15 - Custody sandbox adapter and policy engine P1-P10]], [[A21 - Licences, feature flags and licence perimeter (G21)]], [[A36 - Offerings- offer terms, listing review, raise monitor (SM-04)]], [[A39 - Treasury- withdrawals, escrow release, client-money reconciliation]], [[A42 - AML screening, monitoring, cases and SMR clocks (SM-11)]], [[A45 - Reporting, regulatory calendar, filings and metric governance]], [[A48 - Halts, kill switches and wind-down]], [[A101 - Agent guard- anomaly detection, sensitive-action notices and tiered kill switches]], [[A104 - MCP server generated from the OpenAPI document]]. The last ten are hard dependencies: this spec reads from every one of them, and they must be hard edges of A66 in `final-graph.json` and `app-graph.json` (see §7(b)).
**Unlocks:** [[A78 - Operator journey J20 end to end in Docker (AM7 exit)]], [[A79 - End-to-end journeys- investor money path]], [[A81 - End-to-end journeys- recovery, AML and planted breaks]]

**Oracle rule:** this node has a hidden oracle (`oracle: yes`; the graph's `oracle` flag for A66 must be `true` too, so the orchestrator schedules the Oracle stage). No money, supply or holdings move in this node's own code: every decision is made by A14, A13 and A10, and this node only renders it. But this is the console surface through which a checker approves money and authority, so the node QA writes hidden tests and Challenge fixtures to `<workspace>\gauntlet\hidden\A66\` before the build starts ([[App Gauntlet]] §2). They are:
- Playwright specs against the running stack with the sandbox authenticator.
- Its own crops of the `adCmd`, `adApprovals` and `adAudit` regions of the prototype and of A51's and A51.1's extension-B frames for ADM-01, ADM-02, ADM-30 and ADM-31 (§7(d)).
- A figure-trace fixture that maps every number on ADM-01 to its SDK operation and field.
- Planted drifts taken only from existing A06 scenarios, or from the A06.1 scenarios requested in §7(a): `cash-delta` (escrow statement short A$0.01), `audit-tamper` (one audit row altered in a throwaway database), `agent-anomaly`, `aml-third-party-deposits`, and the baseline's GYM01 register break.
- A persona set from A06's `sod-conflict` scenario and seed: the maker, the maker's agent, a delegate, a role granted mid-flight and a support role without need-to-know.
- Hidden fixtures are applied only through A06's scenario loader, A08's sandbox markers or the `RUN_MODE=test` `_test` routes named in §7(a), never by writing to tables directly.

The builder never reads that folder. The tests are published into `app/` only after the gate passes.

This node draws screens only. Eligibility, segregation of duties, severity, SLA arithmetic, hash verification, export scoping and tenant filtering are all decided in the API (A11, A13, A14, A10, A20 and the module owners of each tile). The console shows what the API decided. It never computes a count, a delta, a deadline or a permission of its own, and it never lets a person believe something is done before the API says so. This note is not legal advice.

## Build brief

**1. Routes, placement and what is reused (nothing from A53 is copied)**

- **Routes.** Four pages under `app/(overview)/`, each exporting `screen` and `designKey` as A53's route convention requires:
  - `/`: ADM-01, `adCmd`.
  - `/alerts`: ADM-02, `design-ext:ADM-02`. An alert detail drawer opens through `?alert=<id>`.
  - `/approvals`: ADM-30, `adApprovals`. The selected request is in `?id=<approval id>`, so a split list and detail view can be shared by URL.
  - `/audit`: ADM-31, `adAudit`. It has the tabs Events, Exports and Legal holds, selected through `?tab=`.
- **Sidebar placement.** A route group does not change the URL, so the audit page lives in this group while A53's `nav.ts` shows it under "Risk & records", as extension B's seven groups place it. ADM-01, ADM-02 and ADM-30 sit under "Overview".
- **Shell pieces reused.** Every page composes A53's pieces: `ConsolePage` (mono code eyebrow, Newsreader 40 title, subtitle, KpiGrid, TaskRows), `ConsoleTable`, `FilterBar`, `StatusCell`, `IdCell`, `MoneyCell`, `WhenCell`, `AgentMarker`, `HardwareKeyConfirm` and the SM-00 status card of `PrivilegedActionFlow`. A second copy of any of them is a verify failure: a lint rule forbids `DataTable`, `navigator.credentials` and `approvals.decide|bulkDecide|ratify|veto` imports in the owned paths.
- **Data access.** All data comes through `packages/sdk`. A53's lint already forbids `fetch` outside the SDK, and this node adds no API route, table, migration or worker job. A number or field that the SDK does not return is a spec defect upstream (see §7). It is never computed in the console by paging a list, adding list rows together or reading the client clock.
- **Time.** Countdowns (SLA, TTL, ratification, timelock) run from server instants (`ack_due_at`, `expires_at`, `ratify_by`, `snoozed_until`) against a server-time offset. The offset is taken from the `Date` header of the last SDK response, so a client clock that is hours wrong changes nothing that is shown. Under `RUN_MODE=local|test` A09's injectable clock drives both sides.
- **Refresh.** Pages refresh through one batched read every 30 s, on window focus and after every mutation the page makes, the same as A53's badge tick. There is no websocket. Every tile and table shows "as of hh:mm" from its response.

**2. ADM-01 Command center (`/`)**

- **Header.** Eyebrow `ADM-01`, title "Command center". The subtitle is "{weekday d Mon yyyy} · all figures reconciled at {hh:mm} {zone}.", where the time is A20's `GET /v1/recon/summary` `last_cycle.finished_at` shown in Australia/Sydney time, and `{zone}` is the abbreviation the Australia/Sydney tz database gives at that instant (AEST, or AEDT from 4 Oct 2026). The zone is never hard-coded; a test under A09's injectable clock renders a cycle on each side of the 4 Oct 2026 change and expects "AEST" and "AEDT". The subtitle reads "reconciled" only when two things hold: the last cycle state is clean, and A39's `GET /v1/treasury/reconciliation/status` is clean within its 26-hour freshness. Otherwise it reads "reconciliation not clean since {time} · {n} breaks open" in alert red. The sponsor switcher, PROD / SANDBOX badge, global search, badges and the break-glass button belong to A53's shell and are not redrawn here.
- **KPI row: the design's four tiles, bound to live reads.**
  - **Client money.** A16's `GET /v1/ledger/client-money/summary` `client_money_total_cents` through `formatAud` compact. The caption is "Reconciled hh:mm" from A39's status; if that status is not clean the caption reads "Unreconciled" in red and the tile gets a red border.
  - **Live offers.** A36's count of offerings in `live` or `soft_cap_reached`. The caption "A$x in escrow" is the offer-escrow account balance from A39's `GET /v1/treasury/accounts`.
  - **Approvals waiting.** A14's count for the caller as checker, with the caption "{m} high risk".
  - **Open alerts.** A13's `GET /v1/alerts/summary` `open`, with the caption "{high} high".
  - With the standard seed the row reads A$18.42m / Reconciled 06:00, 3 / A$3.05m in escrow, 5 / 3 high risk, and 3 / 1 high. The "3 high risk" is design bug §6.4-8 fixed: the prototype's static "2 high risk" disagrees with its own list, which has three High rows.
- **"Alerts" TaskRows.** These are the top open items from A13, ordered by severity, then breach, then age:
  - The open alerts from `GET /v1/alerts?state=open&limit=5`. Each row shows its `display_tone` pill (High red, Medium amber), title, detail and time. A click deep-links to the alert's `target_screen` route.
  - The queue items from `summary.queues` as grey "Review" rows. The seed's fourth row, "Listing waiting for review · Marine Parade Apartment · Harbour & Co", is a queue item, not an alert, exactly as A13 fixes it.
  - Deep links follow A13's corrected targets: register break to ADM-17, certificates to ADM-11, valuation to ADM-06, listing to ADM-07. They do not follow the prototype's `adKyc` and `adListing` jumps. A target whose node has not landed opens A53's placeholder page.
  - Each alert row has an inline Acknowledge and an Assign menu that call the same A13 operations as ADM-02. A "View all" link leads to `/alerts`.
- **Role-filtered sections below the design's row.** These are the product-map content the design lacks (design analysis §6.2: the product map wins on content, the design on presentation). They are drawn as KpiGrid cards and TaskRows in the design's tokens:
  - **Operations queues.** `summary.queues`: identity checks, wholesale certificates, bank matches, withdrawals, register sync, complaints, agent anomaly holds. Each shows its count, oldest age, SLA, and a red `BREACH` pill when `past_sla > 0`.
  - **Compliance clocks.** A45's `GET /v1/reporting/calendar/summary` (due 7, due 30, overdue, unassigned). A42's `GET /v1/aml/summary` for SMR drafts with their due times; this tile is the ADM-01 evidence of G14 (AML) and renders only for callers whose effective cells hold `V` on M20 (MLRO and CCO by default), so no other role learns that an SMR clock is running. A21's `GET /v1/perimeter/exemption-meter` as A53's `ExemptionMeter` showing the Digital Assets Framework Act exemption meter against the A$10m cap, amber at the 80 % `warn_pct`. A21's `GET /v1/licences/summary` for licence conditions and renewals due.
  - **Chain and custody.** A27's `GET /v1/chain/status` (head, lag, stuck transactions). A15's `GET /ops?state=` (its "waiting for signatures" queue) for pending signatures with the oldest age. A48's `GET /v1/kill-switches` and `GET /halts` for paused tokens, halts and engaged switches (any engaged switch also raises a red banner at the top of the page). A15's V7 vault for paymaster runway in days.
  - **Treasury.** A39's reconciliation status as "bank vs ledger Δ", which must be A$0.00. It is green "Δ 0" only when the API says clean; any other value shows the exact cents in red with a link to the open Sev2 `money.client_money_delta` alert. Next 7 days of payouts and escrow come from A39's KPIs.
  - **Integrations.** A46's `GET /v1/integrations/summary`: ok, degraded and down counts and the oldest down age.
  - **Headline KPIs from the product map.** Capital raised MTD, pledged vs funded, distributions paid MTD, and recon breaks from A20's summary. Each figure is shown with A45's metric basis. A `management` figure carries the caption "management figure, unaudited". A45's forbidden-label rule applies to tile titles, so there is no "TVL" and no "tokenised value" anywhere.
- **Tile visibility is data.** `src/features/command/tiles.ts` lists each tile with `{key, section, title, sourceOperationId, fields[], requiredCell, roles_default_order}`. A tile renders only when the caller's effective cell set from A11's `GET /iam/me` holds `V` on its source cell. Otherwise it is absent, not greyed, and its source is never requested (a network-log test checks this). Default layouts:

  | Role | Tiles shown |
  |---|---|
  | CCO | All |
  | CEO | KPIs, clocks, treasury, halts |
  | CFO | KPIs, treasury, approvals, clocks |
  | MLRO | Identity and AML queues, SMR clocks, alerts |
  | TRSY | Treasury, bank matches, withdrawals |
  | ENGO | Chain and custody, integrations, alerts |

  The persona's cells, not the role name, decide.
- **Tenant view.** Under A53's "view as tenant" grant, every read carries the acting tenant through A11's session and returns the tenant's slice through A04's row-level security. Tiles with no tenant meaning (platform client money total, custody vaults, exemption meter) show "Not available in tenant view". This matches the product map's line "Sponsors see a view filtered to their own trusts". Sponsor users themselves see their slice on SPN-02 (A62), never in this console.
- **Tile states.** Each tile loads, fails and empties on its own, so one failed source never blanks the page:
  - Loading: a skeleton.
  - Error: the envelope's `code` and `request_id` with Retry.
  - Empty: "Nothing waiting" for a queue with zero items.
  - Source absent: "Not connected · owned by Axx", shown when the SDK operation is absent from `GET /v1/meta`'s operation list. It exists only as a development aid under `RUN_MODE=local` while the builder works; under `RUN_MODE=test` and in the Docker stack it is a render error, not a state. Because every tile source is a hard dependency (see "Depends on"), no tile may show it at A66's own gate, and a pass criterion checks this for every persona; A78 checks it again.
- **ADM-01 actions.**
  - **Acknowledge or assign an alert.** Inline, through A13.
  - **Declare incident.** For CCO, SECO and the on-call ENGO. It opens a dialog with runbook R1-R9 from A48's `GET /runbooks`, severity and summary, then calls A48's `POST /v1/runbooks/{id}/declare` through `HardwareKeyConfirm` with purpose `incident_declare`. That purpose is not yet in A53's list of `HardwareKeyConfirm` purposes, so it is raised to A53 as a follow-up in §7(a) and is blocking at Specify. A48 refuses tokens with `INC_HUMAN_ONLY`. After that the page shows the incident's action list with one "Act" button per action, each its own A14 request. There is never a blanket approval.
  - **Emergency pause.** For CCO, SECO and pauser holders. It deep-links to ADM-14 (A72) with the vehicle and the `SGL+R` action prefilled, and shows the 2-hour ratification note. This page performs no pause itself.
  - **Export board pack.** For CFO and CCO. It calls A45's report export with `kind: board_pack` and the current period, shows the returned `REP-` id, state and file hash, and says "This export is recorded in the audit log". The export is single-actor and audited, not approved, exactly as A45 fixes it. When an agent token makes the export, the export is a bulk data-out act: it raises A101's sensitive-action notice to the human (`agent.sensitive_action`, delivered through A43 to the person, never to the agent) and appears in the human's activity (`GET /v1/me/activity`).

**3. ADM-02 Alerts centre (`/alerts`)**

- **Design source for this screen.** A51 draws only part of ADM-02: FilterChips for severity Sev1, Sev2, Sev3 and All and state New, Acknowledged and Snoozed; the DataTable columns; the SLA countdown; the row actions Acknowledge, Assign, Escalate and Snooze with its reason; the bulk control; and the default, empty and "one Sev1 breached" states. Everything else on this screen (Sev4, the Assigned, Escalated and Resolved states, the module and owner filters, "Breached only", Resolve, Reopen, Create incident, the detail drawer, the MTTA and MTTR strip and the digest panel) comes from the A51.1 follow-up frame in §7(d), not from A51.
- **Filters and table.** FilterChips for severity (Sev1, Sev2, Sev3, Sev4, All), state (New, Acknowledged, Assigned, Snoozed, Escalated, Resolved), module, owner (Mine, Unassigned, my role) and "Breached only". The module and owner filters are server-side filters on A13's list (A13.1 in §7(a)). A `ConsoleTable` over A13's `GET /v1/alerts` has the columns ID / Sev / Rule / Object / Created / Owner / SLA / State:
  - SLA is a Geist Mono countdown to `ack_due_at` that turns into "SLA breached {n} min ago" in red when `breached_at` is set. Sev4 shows "no clock".
  - The ADM-01 alert rows are the top slice with the same strings.
  - Filters live in the URL, and all filtering is server-side (A13 N12).
  - The empty state reads "No open alerts."
- **Row actions.** Each is disabled or hidden according to the `allowed_actions` the API returns for the caller and the alert's state, never according to a client rule:
  - **Acknowledge.**
  - **Assign.** The assignee picker lists only users A13's eligible-assignees read returns (A13.1 in §7(a)): the owner role or its escalation path. The console never builds the list from the user directory.
  - **Escalate.** A reason is mandatory.
  - **Snooze.** A reason is mandatory. The duration picker is bounded by the policy maximum the API returns: absent for Sev1, up to 4 h for Sev2, up to 5 business days for Sev3 and Sev4. It is never offered from `new`.
  - **Resolve.** A reason is mandatory. For `closes_by = system` types it is replaced by the closing condition text, for example "closes when the reconciliation re-run is clean".
  - **Reopen.** A reason is mandatory.
  - **Create incident.** It opens ADM-01's declare dialog with the alert id attached as `incident_id` context.
  - Reasons are trimmed and limited to 500 characters in the field. A13's `reason_required`, `snooze_too_long`, `transition_not_allowed`, `alert_closed`, `system_closed` and `self_subject` are rendered word for word as A53 Banners.
- **Bulk acknowledge.** A checkbox column. The "Acknowledge selected" control is enabled only when every selected row is Sev3 or Sev4 in the current response, which covers at most 200 ids. It sends one `POST /v1/alerts/bulk-acknowledge`. A `bulk_refused` answer lists every offending id in the Banner and changes nothing on screen. The console never falls back to one call per id. After a success it toasts "{n} alerts acknowledged".
- **Detail drawer.** `GET /v1/alerts/{id}`:
  - Title, detail, subject link, severity history (upgrades), owner and escalation path.
  - The SLA clock from `GET /v1/sla/clocks?subject=`.
  - Occurrences and first and last seen.
  - Activity with `AgentMarker` on every agent-made step, and a link to the audit events for the alert through `/audit?object_ref=`.
  - The emitter as a service-account label ("Recon job", "Identity provider") in the style of A10's labels.
- **MTTA, MTTR and digest.** The MTTA and MTTR strip by severity, over 7 and 30 days, and the "Today's digest" panel both read A13 (see §7 for the A13.1 read contract). The digest panel shows the viewer's role digest of the day: open by severity, breaches in the last 24 h, snoozes ending today, clocks due within 24 h. It is a read of the stored `alert_digest` row, never recomputed.
- **Agent rows.** Agent-related alerts (`agent.kill_switch_fired`, `agent.anomaly_hold`, `agent.revocation_storm`) link to ADM-35 (A77) for callers with `V` on M26.

**4. ADM-30 Approvals inbox (`/approvals`)**

- **Layout and list.** The design's split layout (`minmax(240px,1fr) minmax(0,1.8fr)`):
  - **Design source.** The split layout, list cards, detail card, check rows, maker-checker note and toasts are `adApprovals`; the bulk-approve state is A51's. The tabs, the Impact and "Expires in" columns, the diff, the simulation block, the file hash, the other approvers and the custody-binding block are drawn by the A51.1 follow-up (§7(d), `design-ext:ADM-30+`).
  - **Tabs.** "Waiting for me" (`GET /v1/approvals?mine=checker&state=prechecked`, the default and the badge). "To ratify" (`GET /v1/approvals?mine=ratifier`: `SGL+R` acts with `ratification_state = pending` where the caller is an eligible ratifier, with the 2-hour countdown to `ratify_by`). "Queued" (`GET /v1/approvals?mine=vetoer`: timelocked requests where the caller may veto). The `ratifier` and `vetoer` filters are A14.1 items (§7(a)); the console never filters an "All open" page client-side to build these tabs. "Submitted by me" (`mine=maker`). "All open" (CCO, SECO and CEO, when `V` on M25 permits). "Recently decided".
  - **List cards.** In the design's style: `{id} · {age}` in mono, a risk pill (High red, Medium amber, Low grey through `toneFor`), type, object, and a maker line, which is "raised by {name} ({team})", or "System threshold" for `AUTO`-fall-through rows.
  - **Agent marker.** When the list item's `maker_principal` is an agent, the maker line is replaced by `AgentMarker`: "submitted by agent {token_prefix} on behalf of {person}". Both fields come on the list item (A14.1), never from a detail fetch per card.
  - **Right-aligned columns.** Impact, the list item's `impact` field (amount, units or holders affected, A14.1), and "Expires in" (a TTL countdown to the list item's `expires_at`, amber under 12 h and red under 2 h, which is the product map's 72 h TTL column).
  - Sorting is by age (the design's order) or by TTL remaining. Filters for risk, action type, tenant and state live in the URL.
  - The empty list is the design's dashed box, "Inbox clear."
- **Detail.** Read from `GET /v1/approvals/{id}`, so maker and checker see one thing:
  - The header "{id} · raised by {maker}" and the type in Newsreader 28, with the object below.
  - The `checks[]` as the design's check rows with gate refs and the pills Pass, Done, Matched, Pending and flags, using the API's tones.
  - The diff against the last approved payload of the same object, for example "vs last run +0.8 %", with changed fields highlighted.
  - The `simulation` block the checkers will see (A14.1 on the detail: the stored precheck result with rule-pack version and gate evaluations).
  - The payload hash, and the detail's `file_sha256` where a file is bound (distribution and payout files, A14.1), in Geist Mono with a copy button.
  - Evidence documents as `DocumentRow`s.
  - The `other_approvers[]` with their status.
  - Policy code, TTL and timelock.
  - Custody binding from the detail's `custody_binding {vault, quorum, condition}` (A14.1): which vault signs (for example "V2 token agent · 2-of-3") and on what condition ("signs after approval; approval ID bound to the signing request"). Absent when the action has no custody vault.
  - The SM-00 status card from A53.
  - The SoD line. The API's per-caller `decision {allowed, code, note}` (A14.1) is the only source. The console renders a fixed design framing around the API's `note`: the prefix "Maker-checker (G13): " always, then the API's `note` word for word (for example "you are not the maker, so you can approve", "you are the maker", "already decided", or the note for `APR_ROLE_NOT_HELD`), then the suffix " Approval ID is bound to the signing request." only when `custody_binding` is present. `copy.ts` marks the prefix and suffix as design framing and the note as API text, so a test can tell them apart.
- **Decisions.** Approve and Reject are A53's slots filled by `HardwareKeyConfirm`, and Reject takes a mandatory reason code from the domain list. Veto (from "Queued") and Ratify (from "To ratify") go through the same modal. Approve, Reject, Veto and Ratify are enabled only when the API's `decision.allowed = true` for the caller; otherwise they are disabled with `decision.code` and `decision.note`. A14 computes `decision` from the maker identity set (which includes the human behind an agent and every delegate) and the `required_checkers[]` resolved at submit, so the delegate of a maker and a checker whose role was granted after submission both get `allowed = false`. The client adds no rule, reads no note text to decide, and has no override. After the API's 2xx:
  - It toasts "{id} approved · sent to signing" or "{id} rejected · maker notified".
  - The card leaves "Waiting for me".
  - The sidebar badge, the ADM-01 KPI and the next selection refresh from the API.
- **Bulk approve.** The checkbox renders only on rows whose list item carries `bulk_allowed = true` (A14.1), which is A14's `risk = low` on the current policy version. It is never derived from the risk label on the card. Extension B draws the checkbox on Medium rows as a presentation of the state, and the API flag wins. Selecting rows opens A53's bulk modal listing every id and signing once, capped at 50. `APR_BULK_NOT_LOW_RISK` and `APR_BULK_MIXED` are shown whole, and nothing is applied. With the standard seed no row qualifies, and A06 has no low-risk approvals scenario yet, so the bulk state is exercised with the A06.1 `low-risk-approvals` scenario requested in §7(a).
- **Refusals and expiry.** Every A14 and A11 refusal code A53 lists renders as a Banner. `APR_EXPIRED` also moves the card to "Recently decided" with the expired pill. A request that expires while the modal is open fails with `APR_EXPIRED` and the modal says so. A request that left the inbox still opens by URL with its terminal state (A14 returns 200), never as a blank page.

**5. ADM-31 Audit log (`/audit`)**

- **Design source.** The Events table is `adAudit`. The verification banner, the on-behalf-of and token columns, the export dialog and the legal-holds tab are drawn by the A51.1 follow-up (§7(d), `design-ext:ADM-31+`); A51 does not draw them.
- **Verification banner.** From A10's `GET /v1/audit/head`:
  - Green "Hash chain VERIFIED to {verified_at}", with the verified range's end time `to_ts` and the head hash's first 12 characters, when `status = ok` and the verification is younger than 26 h.
  - Amber "Not verified since {time}" when the verification is older than 26 h or `missing`.
  - Red "Hash chain BROKEN at {first_bad_ts} · Sev1 alert open", with a link to the `audit.chain_broken` alert, when `status = broken`.
  - `to_ts` and `first_bad_ts` are A10.1 items (§7(a)): A10's head returns sequence numbers only, and the console never renders a `seq` or looks up an event to turn one into a time.
  - SECO and CCO get "Verify now", which calls `POST /v1/audit/verify`. It is human-only under A10's own delegability table (`audit_human_only` for a token), so the button is rendered only for a passkey session. A synchronous result updates the banner. A `202` shows "Verification running · job {id}" and polls the head.
  - The banner is never a static string.
- **Events tab.** A `ConsoleTable` over `GET /v1/audit/events` with the design's columns Time / Actor / Action / Object / Approval / Result:
  - Result pills: OK green, Referred amber, Alert red, Refused red.
  - Added columns: "On behalf of", rendered through `AgentMarker` from `on_behalf_of_label` and `token_prefix`. Hidden by default through `ColumnPicker`: Token and Event hash.
  - Filters in the URL: object (exact or type prefix), actor, on behalf of, token, action, approval id, result, date range, and tenant for platform staff.
  - Clicking a row opens a drawer from `GET /v1/audit/events/{id}` with previous and event hashes, before and after hashes, canonical version, request id and the approval link.
  - `seq` is used only as the opaque cursor and is never rendered, counted or shown as a gap. Totals are never shown.
  - **Need-to-know rows follow A42's contract.** A `need_to_know` event the caller is not on the ACL for is either absent (A10's list and 404 on the detail) or, where A42's redaction-by-schema returns it, rendered only as the redacted row "AML · restricted event", word for word, with its hash intact and no subject, object label, actor detail or reason. The console never un-redacts, never fetches the detail of a redacted row, and the export dialog relies on the API applying the same redaction to the file. Whether a redacted row carries `seq` in its list payload is a blocking Specify decision owned by A10 and A42 (§7(a)); the console renders no `seq` in any case.
  - The empty state reads "No events match these filters."
- **Exports tab.**
  - "Export" opens a dialog that restates the active filter in words, offers CSV or JSON lines and a reason field, and warns "This export is itself recorded in the audit log".
  - It calls `POST /v1/audit/exports` and lists the caller's exports with state, row count, file sha256 and a download link to A08's presigned document link once `audit.export_completed` exists.
  - `audit_export_too_large` is shown with A10's suggestion to narrow the filter.
  - The export's own `audit.exported` row appears in the Events tab on the next refresh, and a test asserts it.
  - An export requested by an agent token (`agent-maker` under `audit:events.export`) is a sensitive act in A10's table: it raises A101's sensitive-action notice to the human and appears in the human's activity. The console adds nothing to this; a test asserts the notice and the activity row.
- **Legal holds tab.** A10's `GET /v1/audit/holds` as a read-only list for LEGL, CCO and SECO. "Place hold" and "Release" run through A53's `PrivilegedActionFlow` as A10's `4E` request (LEGL maker to CCO checker); the CCO decision is made on ADM-30. Both routes are human-only in A10's delegability table (`audit_human_only` for a token), so the buttons render only for a passkey session. `audit_hold_active` is shown word for word.
- **Who reaches this page.** CCO, SECO and any staff role with `V` on M25's audit cell. TP-AUD and TP-OBS reach the same API through A105's partner portal, not this console. Likewise, the product map lists TP-RE (the trustee or responsible entity) as an ADM-30 approver: TP-RE decides on A105's partner-portal approvals page over the same A14 operations, never in this console.

**6. Agent and API parity (D04, G22): every action this node exposes**

- **Parity manifest.** `src/features/command/parity.ts` lists every screen action with its `{screen, action, operationId, mcpTool | null, delegability}`, where delegability is `agent-maker`, `agent-read` or `human-only`, plus the reason for any `human-only`. It also lists every figure on ADM-01 with its source operation and field. A100's parity suite imports this manifest unchanged, and the node QA's figure-trace test reads it.
- **Delegable actions.** Anything a staff member can do on these screens, their agent token can do through the same operation under A12's `agentAuth` middleware and G22. The console calls no private route, and an agent can rebuild the command view from the same reads through the SDK or A104's MCP read tools.

  | Screen | Delegable actions | Scope | Delegability |
  |---|---|---|---|
  | ADM-01 | Every tile read (A13, A14, A16, A20, A21, A27, A36, A39, A42, A45, A46, A48 and A15 read operations) | Each within its own module | `agent-read` |
  | ADM-02 | List, detail, summary, acknowledge, assign, escalate, snooze, resolve, reopen, bulk acknowledge | M01 | `agent-maker` |
  | ADM-30 | List and detail | M25 read | `agent-read` |
  | ADM-31 | Events, event detail, head, holds list, export request and export status | M25 audit read; export under `audit:events.export` | `agent-read`, with the export `agent-maker` (A10's `AUDIT_DELEGABILITY`) |

  Agent limits on ADM-02 are A13's: `self_subject` for alerts about the agent's own tokens. The agent's human is still the one audited (A10) and notified (A101). ADM-01's board-pack export is A45's `reports:export`, delegable to a CFO's agent under mandate, audited "agent on behalf of". Both bulk data-out acts, the agent-made audit export and the agent-made board-pack export, raise A101's sensitive-action notice to the human and appear in the human's activity.
- **Human-only actions.** Each is human-only because its owning node says so, with that node's refusal code; this node invents no code:
  - Approve, reject, veto, ratify and bulk approve: A14 refuses any token with `APR_CHECKER_MUST_BE_HUMAN` before scope evaluation.
  - Declare incident: A48's `INC_HUMAN_ONLY`.
  - Audit "Verify now" (`POST /v1/audit/verify`): A10's delegability table, refused with `audit_human_only` before any work.
  - Place and release legal hold (`POST /v1/audit/holds`, `/holds/{id}/release`): A10's delegability table, refused with `audit_human_only`. The CCO's checker decision on the resulting `4E` request is human-only in A14 as well (`APR_CHECKER_MUST_BE_HUMAN`).
  The manifest copies these entries from A10's `AUDIT_DELEGABILITY`, A14's and A48's exports rather than restating them, and the MCP catalogue must not list them. Whether declaring an incident, verifying the chain or placing a hold must stay personal is recorded as an HG-LEGAL counsel question; if counsel or the founder changes the answer, the flip happens only in A03's `delegable.ts` and the owning node, and this manifest follows it.
- **Agent-made rows.** Every row made by an agent, on all four screens, carries `AgentMarker`: approvals submitted by an agent, alert actions taken by an agent, and audit events with `actor_type = AGNT`. Hovering shows the token id, agent name and mandate version. For callers with `V` on M26, the marker links to ADM-35.

**7. Spec points to settle at Specify (before the node QA writes the hidden tests)**

- **(a) Missing read fields and fixtures go upstream as follow-up loops.** Every figure and control above must map to an existing SDK field. The following are not in the dependency specs as written and are requested as follow-up loops per the plan's upstream-defect rule, never computed in the console. **Every item that backs a product-map component (ADM-01, ADM-02, ADM-30 or ADM-31) is blocking at Specify.** If one is declined, the node escalates to the founder; the component is never dropped and never faked.
  - **A14.1:**
    - On list items: `impact`, `other_approvers[]`, `expires_at`, `maker_principal` with `token_prefix` and `on_behalf_of` label, and `bulk_allowed`.
    - On the detail: `simulation` (the stored precheck result with rule-pack version and `GateEvaluation` ids), `file_sha256` where a file is bound, and `custody_binding {vault, quorum, condition}`.
    - A per-caller `decision {allowed, code, note}` on list items and the detail, with `code` one of `APR_MAKER_IS_CHECKER`, `APR_ROLE_NOT_HELD`, `APR_EXPIRED`, `APR_ALREADY_DECIDED` (or A14's existing spelling) and the others A14 already raises, computed from the maker identity set and the `required_checkers[]` resolved at submit.
    - List filters `mine=ratifier` and `mine=vetoer` for the "To ratify" and "Queued" tabs.
    - `GET /v1/approvals/summary` returning `{waiting_for_me, high_risk_waiting_for_me, expiring_24h, to_ratify, queued_vetoable}`. A53's badge needs this too.
    - A `RUN_MODE=test` transition route (`POST /v1/_test/approvals/{id}/transition`) that walks a seeded request through every SM-00 state for the SM-00 test; it is absent from OpenAPI in any other mode.
  - **A13.1:** `module` and `assignee` (`me`, `unassigned`, role) filters on `GET /v1/alerts`; an eligible-assignees read per alert; `allowed_actions[]` with `snooze_max` per alert for the caller; `mtta_ms` and `mttr_ms` by severity over 7 and 30 days in the summary; `GET /v1/alerts/digests?date=&role=` returning the stored digest.
  - **A10.1:** `to_ts` (the event time of the last verified row) and `first_bad_ts` on `GET /v1/audit/head`. Also a blocking decision owned by A10 and A42: whether a redacted "AML · restricted event" row, and a row absent under need-to-know, can be inferred from `seq` in the list payload (for example by returning an opaque cursor instead of `seq` to callers outside the ACL).
  - **A36.1:** a live-offers count on the console offerings read.
  - **A53 follow-up:** add `incident_declare` to `HardwareKeyConfirm`'s purposes, with A48's declare body as the subject.
  - **A06.1:** a `low-risk-approvals` scenario (at least 50 pending `risk = low`, `bulk_allowed = true` requests across one tenant, plus one `4E+Q` distribution run of A$1.2m for the mixed case); a `distributions-delta` scenario (a one-cent bank vs ledger difference on the distributions account with the next 02:00 reconciliation run skipped); a `sanctions-true-match` scenario (a `kyc.sanctions.true_match` event under need-to-know with an otherwise identical control seed); a `bulk-alerts` scenario (49 Sev3 and Sev4 alerts plus one Sev1 register break). Until A06.1 lands, the challenges use the existing `cash-delta`, `audit-tamper`, `sod-conflict`, `agent-anomaly` and `aml-third-party-deposits` scenarios.
  The orchestrator decides each, and the decision is recorded in `A66.qa.md`.
- **(b) Graph edges.** A06, A15, A21, A36, A39, A42, A45, A48, A101 and A104 feed this node (seed and scenarios, ADM-01 tiles, the agent-anomaly-hold queue and sensitive-action notices, and the MCP catalogue the pass criteria check). They are listed as hard dependencies above. All of them sit in AM1 to AM3, so adding them as hard edges of A66 in `final-graph.json` and `app-graph.json` makes no cycle and no milestone-order violation. There are no soft edges: the scheduler orders by remaining path length, so only a hard edge guarantees each source has passed before A66 is scheduled.
- **(c) Alert id prefix.** A53's global search routes alerts by `ALR-`, but A13 issues `ALT-<yyyymmdd>-<6 base32>`. This node registers the alerts, approvals and audit search providers with A13's spelling, and the `ALR-` entry is reported to A53 as a follow-up. A13's spelling wins.
- **(d) Design frames A51 does not draw (A51.1 follow-up, blocking).** A51 draws ADM-02 only in part and does not own ADM-01, ADM-30 or ADM-31. An A51.1 follow-up loop draws, in the dc file from A05's tokens and A51's blocks, with `default`, `empty`, `blocked` and (where a persona can be the maker) `not-eligible` states:
  - `design-ext:ADM-02+`: the full Alerts centre (Sev4, the Assigned, Escalated and Resolved states, module and owner filters, "Breached only", Resolve, Reopen and Create incident, the detail drawer, the MTTA and MTTR strip and the digest panel).
  - `design-ext:ADM-01+`: the role-filtered sections below the `adCmd` row (operations queues, compliance clocks, chain and custody, treasury Δ, integrations, headline KPIs), the per-tile states and the declare-incident dialog.
  - `design-ext:ADM-30+`: the tabs, the Impact and "Expires in" columns, the diff, the simulation block, the file hash, other approvers, the custody-binding block and the SoD line with its framing.
  - `design-ext:ADM-31+`: the verification banner in its three states, the on-behalf-of and token columns, the redacted "AML · restricted event" row, the export dialog, the Exports tab and the Legal holds tab.
  Specify is blocked until these frames are drawn and HG-REVIEW has passed on them. The node QA's fidelity crops and the design-fidelity pass criterion use these anchors for the parts `adCmd`, `adApprovals` and `adAudit` do not show.

## Owned paths (other nodes do not edit these)

- <workspace>\app\apps\console\app\(overview)\
- <workspace>\app\apps\console\src\features\command\

Inside the route group, by convention: `page.tsx` (ADM-01), `alerts/page.tsx`, `approvals/page.tsx`, `audit/page.tsx`, and a `loading.tsx` and `error.tsx` per route that delegate to A53's page states.

Inside `src/features/command/`:
- `tiles/`: `tiles.ts` catalogue, one source adapter per SDK read, the tenant-view rules.
- `alerts/`: table columns, row actions, bulk control, drawer, digest panel.
- `approvals/`: list cards, tabs, detail sections, custody-binding block, bulk selection.
- `audit/`: banner, events table, drawer, export dialog, holds tab.
- `search/`: providers registered with A53.
- `parity.ts`, `copy.ts` (every user-facing sentence and refusal sentence), `index.ts`.

Shell primitives, tokens and extension blocks belong to A53, A05 and A51. A missing visual block is requested from A51 as a follow-up loop and never drawn here in raw hex.

## Design references

- **`design:adCmd`** in `<workspace>\design\RealX App.dc.html`: the screen definition on line 880 (eyebrow `ADM-01`, title "Command center", subtitle "Wednesday 23 Sep 2026 · all figures reconciled at 06:00 AEST.", the four KPIs and the four alert tasks with their pills, details and times). The admin page frame: `main` at line 630 (`padding:32px 40px 64px`, gap 24, `max-width:1280px`), the mono 12 px eyebrow and Newsreader 40 h1 on lines 632-634, the KpiGrid on lines 636-640 (1px `#E3DED4` gaps, white cells, Geist Mono 500 24 px values), and the TaskRows on lines 642-652 (grid `auto 1fr auto`, pill, title and detail, mono time, hover border ink). Design analysis §3.3 `adCmd` and §6.2 "Command center": the product map wins on content, the design on the KpiGrid plus TaskRows presentation.
- **`design:adApprovals`**: the split layout on lines 655-679; the list cards on lines 658-663 (mono `id · age`, risk pill, type, object, selected card white with ink border, else `#FBFAF7`); "Inbox clear." on line 665; the detail card on lines 667-677 (radius 14, padding 26, "{id} · raised by {maker}", Newsreader 28 type, check rows, the maker-checker note on line 673, Approve green `#1F5E4B` and Reject `#A3312A` outline pills on lines 675-676); the `APS` data on lines 777-782 (APR-2291, 2290, 2288, 2286, 2285 with their checks; A06 seeds these with the deltas listed under "Design bugs not reproduced", so the expected cards come from A06's manifest, not from `APS`); `apList` on lines 854-856; the toasts on line 913. The product map wins on the hardware-key step, the TTL column, impact, bulk approve and custody binding (design analysis §6.2 "Approvals").
- **`design:adAudit`**: the definition on line 888 (subtitle "Append-only. Every action, who made it, who approved it."; columns Time / Actor / Action / Object / Approval / Result; five rows including "Identity provider Referred INV-00912" amber and "Recon job Detected break Gympie Road Retail" red) and the admin DataTable block on lines 719-726 (mono 11 px uppercase headers, 13 px cells, `#ECE8E0` row rules). The verification banner, on-behalf-of and token columns, redacted row, export dialog and legal-hold list are not drawn by A51; they come from `design-ext:ADM-31+` (A51.1, §7(d)).
- **`design-ext:ADM-02`** from [[A51 - Design extension B- operator console screens not in the prototype]]: exactly what A51 draws, namely FilterChips by severity (Sev1, Sev2, Sev3, All) and state (New, Acknowledged, Snoozed), the DataTable ID / Sev / Rule / Object / Created / Owner / SLA / State with the mono SLA countdown, the row actions Acknowledge, Assign, Escalate and Snooze with its reason field, the bulk control disabled unless all rows are Sev3 or Sev4, and the states default, empty and "one Sev1 breached" ("SLA breached 12 min ago"). The `adCmd` rows are its top slice. Extension B's KeyConfirmModal, SplitListDetail and the bulk-approve state of ADM-30 are the frames A53 implements and this node composes.
- **`design-ext:ADM-01+`, `design-ext:ADM-02+`, `design-ext:ADM-30+`, `design-ext:ADM-31+`** from the A51.1 follow-up (§7(d)): everything on these four screens that neither the prototype nor A51 draws. Specify is blocked until they exist and HG-REVIEW has passed.
- **Tokens.** Cream `#F5F2EC`, ink `#17191C`, green `#1F5E4B`, brick `#9A4524` (the `AgentMarker` pill), alert red `#A3312A`, the TONE map (green, amber, red, grey, brick) through `toneFor`, Newsreader for display, Geist for text, Geist Mono for ids, hashes, times and figures.
- **Design bugs not reproduced.** §6.4-5: one approval id for two objects; the seed gives the "Listing change · Unley Road" audit row its own id. §6.4-7: one as-of date, 23 Sep 2026. §6.4-8: static "2 high risk" and static badges; all counts are live. The prototype's deep links from the certificate and valuation alerts to the wrong screens are corrected by A13's `target_screen`. The subtitle's fixed "AEST" is replaced by the tz-database abbreviation. Seed deltas on ADM-30, recorded as design fixes (A06's design-bug table, item 4a):
  - APR-2288 is decided in the seed (approved 22 Sep 09:10, because OP-7731 needs an approved request), so it is not in the inbox; APR-2292 (GYM01 register adjustment, `6E`, High, maker Grace L. (Registry ops)) takes its place.
  - APR-2291's maker is "agent `treasury-drafter` on behalf of Priya N. (Treasury ops)", so its card shows `AgentMarker`, not a plain "raised by" line.
  - Maker team labels follow the seed, not `APS`: APR-2290 Tom H. (Listings), APR-2286 "System threshold", APR-2285 Jordan Blake (CCO) with Sam R. (LEGL) as checker.
  - The expected five cards, in the design's age order, are therefore APR-2291 (2h), APR-2292 (4h), APR-2290 (5h), APR-2286 (1d), APR-2285 (2d), with their check rows taken from A06's manifest and A14's `checks[]`.

## Requirements covered

- **ADM-01 Command center.** Done when:
  - The design's KPI row and TaskRows render from live reads with the seed values above.
  - The product map's queues with SLAs, compliance clocks, chain and custody, treasury with Δ that must be 0, and headline KPIs render as role-filtered tiles whose visibility equals the persona's cells.
  - Declare incident, emergency pause (deep link), board-pack export and inline alert acknowledge or assign work through their owning APIs.
  - Every figure traces to one operation field in `parity.ts`.
  - A non-zero Δ or a stale reconciliation can never render as "reconciled".
- **ADM-02 Alerts centre.** Done when:
  - Severity, state, module and owner filters, SLA countdowns and breaches render from A13.
  - All six row actions and bulk acknowledge enforce A13's reason and scope rules through the API, with every refusal shown word for word.
  - MTTA and MTTR and the daily digest panel read A13 (after A13.1).
  - The empty and breached states match extension B.
- **ADM-30 Approvals inbox.** Done when:
  - The five seeded requests (APR-2291, APR-2292, APR-2290, APR-2286, APR-2285 from A06's manifest) render in the design's list and detail with impact, diff, simulation, file hash, evidence, other approvers, TTL, custody binding, the SoD line (design framing plus the API's `decision.note` word for word) and the agent marker.
  - The "To ratify" and "Queued" tabs read A14's `mine=ratifier` and `mine=vetoer` filters.
  - Approve, reject, veto, ratify and bulk approve reach A14 only through `HardwareKeyConfirm`.
  - After a decision the item leaves the list and the badge and KPI drop from the API.
  - Expiry at 72 h shows as expired.
- **ADM-31 Audit log.** Done when:
  - The verification banner reflects `GET /v1/audit/head` in all three states.
  - The events table shows the five design rows with on-behalf-of and filters.
  - Exports are requested, listed and themselves audited.
  - Legal holds are listed and changed only through the approval flow.
  - No numbering or total leaks a need-to-know event, and a restricted AML event renders only as A42's redacted row.
- **Journeys: J20 Operator daily run, steps 1, 2 and 6.** Done when a Playwright run with distinct personas covers the three steps. In step 1 the CCO reviews queues, clocks and treasury on ADM-01. In step 2 the ENGO acknowledges the chain alerts on ADM-02 and resolves or assigns the human-closed ones with reasons. In step 6 SECO reviews overnight audit events on ADM-31 filtered from 18:00 to 06:00, with the chain verified. Each leaves exactly the audit events the owning API writes. A78 runs the whole journey. The inbox steps of J05, J06, J09, J13 and J15 and the ratification in J10 step 3 can be driven through ADM-30 by the seeded personas.
- **Gates: G13 Maker-checker.** Done when:
  - No path on these four screens reaches a decision endpoint without the modal.
  - The maker, the maker's agent's human, a delegate of the maker and a checker whose role was granted after submission each receive `decision.allowed = false` from A14 and see Approve, Reject, Veto and Ratify disabled with the API's `decision.code` and `decision.note`.
  - Bulk approve cannot include a non-low-risk request.
  - Every decision appears on ADM-31 with actor, approval id and on-behalf-of.
  G13 is enforced by A14; this node proves the screens cannot weaken it.
- **Gates: G14 AML (ADM-01 evidence).** Done when the SMR-clock tile from A42's `GET /v1/aml/summary` renders only for callers with `V` on M20, is absent (with no network request) for every other persona, and a treasury Δ other than 0 shows the link to A39's Sev2 alert. G14 is enforced by A42; this node only shows it to the right roles.
- **Gates: G22 Agent authority (on these screens).** Done when every agent call replayed from `parity.ts` meets the same gate evaluation as the passkey session, every human-only action is refused with its owning node's code, and every agent-made row carries the marker. G22 is enforced by A12 and A101.
- **Modules.**
  - **M01 Command Center:** done when ADM-01 and ADM-02 deliver every M01 component and action the product map lists (header pieces through A53), role-filtered.
  - **M25 Access, approvals and audit (inbox and audit parts):** done when ADM-30 and ADM-31 deliver the D16 inbox and audit behaviours. Users, roles and break-glass are A67; the shell parts are A53.
- **State machines: SM-00 Privileged action.** Done when the inbox filters and the detail status card show every state of A03's list (the 14 catalogue states plus `queued`) with A53's labels and tones, and `confirmed` with reconciliation pending never reads as done. A test drives one seeded request through each state with A14.1's `RUN_MODE=test` transition route (§7(a)).
- **Founder requirement D04.** Done when `parity.ts` lists every action and figure on the four screens with its operation and delegability, A100 consumes it, every delegable action succeeds under a staff token with the matching scope, every human-only action is refused to the token with the owning node's named code, every agent-made audit export or board-pack export raises A101's sensitive-action notice to the human, and every agent-made row carries the marker.

## Gauntlet loop for this node

- **Specify.** The frozen inputs are:
  - This note, with spec points (a) to (d) decided; every blocking (a) item merged upstream or escalated.
  - A53's shell exports and lint rules.
  - The read contracts, error codes and seeds of A14, A13, A10 and A20 (with A14.1, A13.1 and A10.1), plus A46's summary.
  - The tile sources of A15, A16, A21, A27, A36, A39, A42, A45 and A48 as listed in §2, A42's need-to-know redaction contract, A101's notice policy and A104's MCP catalogue.
  - A51's partial ADM-02 frame and A51.1's `design-ext:ADM-01+`, `ADM-02+`, `ADM-30+` and `ADM-31+` frames (after HG-REVIEW).
  - A06's standard seed and scenarios (with A06.1).
  The node QA records `specVersion`, the SDK hash and those hashes in the evidence manifest, then writes the hidden specs to `<workspace>\gauntlet\hidden\A66\`.
- **Build (Grok 4.7 xhigh, split by the planner).** Suggested shape: one lead session first creates the four routes with `screen` and `designKey`, `tiles.ts`, `parity.ts` and `copy.ts` skeletons, and confirms every operation id in them exists in the generated SDK. A missing one becomes a spec-point (a) item, not a workaround. Then four parallel sub-agents on disjoint sub-folders:
  - (1) `tiles/` and `page.tsx` for ADM-01: the KPI row, TaskRows, role-filtered sections, tenant view, per-tile states, the declare-incident dialog, the emergency-pause deep link and the board-pack export.
  - (2) `alerts/` for ADM-02: table, filters, SLA countdown with the server-time offset, row actions, bulk control, drawer, MTTA and MTTR, digest.
  - (3) `approvals/` for ADM-30: tabs, cards, TTL, detail sections, custody binding, the SoD rendering, bulk selection on `bulk_allowed`, ratify and veto slots.
  - (4) `audit/` for ADM-31: banner states, events table and drawer, exports, holds, search providers.
  Integration covers:
  - Playwright specs at 1280 and 1024 px for every screen in the states default, empty, error, forbidden, loading and blocked, with the seeded personas.
  - The J20 steps 1, 2 and 6 run.
  - The figure-trace test (each number on ADM-01 compared with its SDK field).
  - The network-log test for hidden tiles.
  - The parity manifest check against the OpenAPI document and the MCP catalogue.
  - axe, `npm run verify`, and `app/evidence/A66.json` with the routes, screenshots, trace table and parity table.
- **Challenge (node QA, Opus 5.5 high).** It runs the hidden specs, then, by hand against the running stack:
  - Every persona (CCO, CEO, CFO, MLRO, TRSY, ENGO, SECO, REGO, LEGL, SUPP, KYCA), with the rendered tiles compared to `GET /iam/me`.
  - The planted drifts from A06's scenarios: `cash-delta` (A$0.01), A06.1's `distributions-delta` with the skipped 02:00 run, and the baseline's GYM01 register break.
  - Every approval SoD persona, including the maker's own agent.
  - Bulk approve and bulk acknowledge with one ineligible item.
  - A request expiring under the modal.
  - A06's `audit-tamper` scenario, then verify-now.
  - An export with and without filters, as a scoped role.
  - A client clock set four hours wrong.
  - Every action on the four screens replayed through a staff token and through the MCP tools, comparing gate evaluations and audit rows.
  - Token-level comparison with its crops of `adCmd`, `adApprovals`, `adAudit`, A51's ADM-02 and A51.1's `design-ext:ADM-01+`, `ADM-02+`, `ADM-30+` and `ADM-31+`.
  - Greps for client-side arithmetic over list rows, `Date.now()` in countdowns, raw hex and imports of decision operations.
  - A check that only the owned paths changed.
  It writes `app/evidence/A66.qa.md`.
- **Gate.** A pass unlocks A78, A79 and A81 and publishes the hidden specs into `app/`. At most three fix loops, each re-running the whole Challenge.
- **Escalation trigger.** Any of these escalates:
  - The same finding failing a third time.
  - Any path to a decision endpoint without the modal, or any figure rendered that is not traceable to an API field (Critical, stops the milestone).
  - A fix that needs a change to A14's, A13's or A10's contract beyond the spec-point (a) follow-ups (spec defect, reopened upstream).
  - A spec-point (a) or (d) item that backs a product-map component and is declined (escalated to the founder, never dropped).
  - A need-to-know event inferable from the audit screen, which goes to A10 and A42.
  - An extension-B frame that cannot be built from A05, A51 and A53 without raw hex, which goes back to A51.

## Node-specific challenges (node QA)

- **Δ 0 that is not zero (money).** A06's `cash-delta` scenario (escrow statement short A$0.01), and then A06.1's `distributions-delta` scenario (a one-cent bank vs ledger difference on the distributions account with the next 02:00 run skipped). Until A06.1 lands, the skipped run is simulated only by advancing A09's injectable clock past 26 h without triggering the run. Expected:
  - ADM-01's treasury tile shows "Δ A$0.01" in red with a link to the Sev2 `money.client_money_delta` alert.
  - The Client money KPI caption reads "Unreconciled".
  - The subtitle does not say "all figures reconciled".
  - At 27 h without a run, every reconciled caption flips from A39's freshness rule without a reload.
  A console that rounds, caches the earlier "Reconciled 06:00" or computes Δ itself fails the node.
- **The maker approves by the side door (compliance, G13).** Jordan Blake's agent token drafts and submits a `policy.change` request. Jordan opens `/approvals?id=` by URL, from the ADM-01 KPI and from the search palette. Then a delegate of Jordan and a CFO whose role was granted after submission open it too. Expected:
  - Every view shows the agent marker "submitted by agent … on behalf of Jordan Blake".
  - For each persona, A14 returns `decision.allowed = false` (Jordan and the delegate with `APR_MAKER_IS_CHECKER`, the mid-flight CFO with `APR_ROLE_NOT_HELD` or A14's equivalent code), and Approve, Reject, Veto and Ratify are disabled with that code and note. The node QA checks that the console enabled nothing on a note text alone.
  - A forced modal call renders `APR_MAKER_IS_CHECKER` or `APR_ROLE_NOT_HELD`.
  - The same agent token calling `decide`, `ratify` or `bulk-decide` through the SDK and the MCP server is refused with `APR_CHECKER_MUST_BE_HUMAN`, and the MCP catalogue lists none of those tools.
  Any enabled Approve for these personas fails the node.
- **Bulk operations that quietly split (money and operations).** Two cases, both from A06.1's `low-risk-approvals` and `bulk-alerts` scenarios:
  - Forty-nine low-risk requests plus one `4E+Q` distribution run of A$1.2m are selected through a doctored URL filter. The bulk modal must send one call, render `APR_BULK_NOT_LOW_RISK` naming the id, and apply nothing.
  - Forty-nine Sev3 and Sev4 alerts plus one Sev1 register break are selected on ADM-02. The acknowledge control must be disabled. A forced call must render `bulk_refused` with the id, and no alert changes state.
  A console that retries item by item, or toasts partial success, fails.
- **Tipping-off through the audit screen (compliance, security).** A06.1's `sanctions-true-match` scenario (or, until it lands, A06's `aml-third-party-deposits` scenario with its AML case) puts a need-to-know AML event in the log. Under A10's visibility rules and A42's contract, a reader outside the case ACL gets either no row (404 on the detail) or the redacted row "AML · restricted event" with the hash intact and no subject. The node QA signs in as SUPP-like and KYCA-like personas with `V` on the audit cell (not on the ACL) and checks:
  - A restricted event renders only as A42's redacted row, word for word, with no subject, object label, actor detail or reason; the console never fetches its detail and never un-redacts.
  - The export file for the same filter carries the same redaction, from the API.
  - No `seq`, count, total or numbering gap is rendered anywhere on the four screens.
  - The ADM-01 tiles and the alert list show nothing about the case to these personas (the SMR-clock tile is absent).
  If the API itself returns more than A42's contract allows, or the redacted row's `seq` lets a reader infer the event (the §7(a) A10.1 decision), the finding goes to A10 and A42 as Critical. If the console renders more than the redacted row, or renders `seq` or totals, this node fails.
- **Chain banner that stays green (security).** The node QA loads A06's `audit-tamper` scenario (one row altered in a throwaway database) and triggers the verifier. Expected:
  - ADM-31's banner turns red with `first_bad_ts` from A10's head and the link to the Sev1 `audit.chain_broken` alert, which also heads ADM-01's TaskRows.
  - "Verify now" re-confirms the break.
  - With verification skipped for 27 h, the banner is amber, never green.
  A banner that is hard-coded, cached across reloads or green while `status` is not `ok` fails.
- **Clocks the client makes up (compliance).** With the browser clock set four hours fast and then two days slow, the node QA compares the ADM-02 SLA countdowns, the ADM-30 TTL and ratification countdowns and the "as of" labels with the server's instants. Expected:
  - Every value is within one refresh tick of the server-derived value.
  - A Sev1 alert shows "SLA breached" exactly when A13 sets `breached_at`.
  - A request at 71 h 59 min shows under 1 minute left and is not expired, while one at 72 h 01 min shows expired, from A14's sweep.
- **Snooze and resolve as a way to hide (operations).** The node QA tries each of the following from the table, the drawer and a forced SDK call:
  - Snooze a Sev1.
  - Snooze a Sev2 for a day.
  - Snooze from `new`.
  - Resolve a `closes_by = system` register break.
  - Act as an agent on an `agent.anomaly_hold` about its own token.
  Every one must be absent or disabled in the UI and refused by A13 with the named code (`snooze_too_long`, `transition_not_allowed`, `system_closed`, `self_subject`), rendered word for word. A UI that offers a Sev1 snooze fails even if the API refuses it.
- **Tiles that leak across roles and tenants (security).** An MLRO opens `/` and edits the URL and query to force treasury and custody sources. An ENGO does the same for client money. A CCO in "view as tenant" for Harbour & Co opens every tile. Expected:
  - Hidden tiles make no network request at all (network log).
  - Forced source calls are refused by the API and render the forbidden state inside the tile.
  - The tenant view shows only Harbour & Co's figures, and "Not available in tenant view" for platform tiles.
  - No other tenant's name, id or number appears in the DOM.
- **Design fidelity.** With the standard seed at 1280 px:
  - ADM-01 must reproduce the four KPI tiles and four TaskRows of `adCmd`, with the recorded fix "3 high risk" and the corrected deep links.
  - ADM-30 must reproduce the design's card style for the five seeded cards in A06's order (APR-2291, APR-2292, APR-2290, APR-2286, APR-2285, with the seed deltas recorded under "Design bugs not reproduced": APR-2292 in place of APR-2288, the agent marker on APR-2291, the seed's maker team labels), the APR-2291 check rows from A06's manifest, and the SoD line with its design framing and the API's note word for word.
  - ADM-01's role sections, ADM-02's A51.1 additions, ADM-30's additions and ADM-31's additions must match the A51.1 frames (§7(d)).
  - ADM-31 must reproduce the five `adAudit` rows with the recorded §6.4-5 id fix.
  - ADM-02 must match A51's frame for the parts A51 draws.
  Token-level comparison (font family, size, weight, letter-spacing, colours, radii, borders, paddings) must show no drift beyond 1 px other than the recorded design-bug fixes.

## Pass criteria

- **Hidden specs.** The node QA's hidden specs in `<workspace>\gauntlet\hidden\A66\` pass 100 % on three consecutive runs against the Docker stack with the sandbox authenticator, at 1280 px and 1024 px.
- **Figure trace.** 100 % of the numbers, pills and captions on ADM-01 map through `parity.ts` to one SDK operation field. The node QA's trace test compares each rendered value with a direct SDK call for eight personas and finds zero mismatches. A grep finds no `reduce`, `sum` or `length`-based count over list-response rows in `src/features/command/` outside the formatting helpers.
- **Role matrix.** For each seeded staff persona, the rendered tile set equals the tiles whose source cell the persona holds in `GET /iam/me`. The network log shows zero requests for hidden tiles. The tenant view shows only the tenant's rows. The layouts at 1280 px and 1023 px have no horizontal page scroll. No tile shows "Not connected" (the source-absent state) for any persona at A66's own gate; a hidden spec asserts this under `RUN_MODE=test` on the Docker stack.
- **Decision surface.** No import of the SDK's decision, bulk-decide, ratify or veto operations exists in the owned paths (lint, with the node QA's planted import failing it). Every A14 and A13 refusal code named in §3 and §4 renders as a Banner with its code: a generated test iterates both modules' OpenAPI error catalogues. The five seeded requests show the SoD line (design framing plus the API's `decision.note` word for word) for a checker and the maker's note for their makers. Decision buttons are enabled if and only if `decision.allowed = true`: a generated test over the maker, the maker's agent's human, a delegate of the maker and a checker granted the role after submission finds every button disabled with the API's code, and a grep finds no comparison against note text in the owned paths.
- **SM-00 and TTL.** A test drives one request through every state of A03's list and asserts the label, tone, tab and action slots. TTL and ratification countdowns come from server instants and stay correct with the client clock skewed by ±48 h. Expiry at 72 h and ratification lapse at 2 h show within one refresh tick under the injectable clock.
- **Alerts.** The seed renders "3 · 1 high" and the three alert rows plus one queue row in the right order and tones. The four bulk-acknowledge refusal cases and the Sev3/4 success case behave as A13 defines. Every snooze bound comes from the API. The digest panel shows the stored digest for the viewer's role. Each action writes exactly the audit events A13 writes: the console adds none, and the count difference over the test run is zero.
- **Audit.** The banner shows all three `status` states from `GET /v1/audit/head`. The five design rows render with on-behalf-of. An export writes `audit.exported` and `audit.export_completed` and its row appears on refresh. The file sha256 shown equals the export row. `seq` never appears in the DOM. A need-to-know AML event renders to a non-ACL reader only as A42's redacted row, word for word. The banner's times come from `to_ts` and `first_bad_ts`. After the full suite, A10's chain verifier passes.
- **Agent parity.** `parity.ts` lists every action on the four screens (A100 consumes it unchanged). Every `agent-maker` or `agent-read` action succeeds under a staff token with the matching scope and yields the same gate evaluations and audit events as the passkey session, apart from actor and on-behalf-of. Every `human-only` action is refused to the token with its owning node's named code (`APR_CHECKER_MUST_BE_HUMAN`, `INC_HUMAN_ONLY`, `audit_human_only`) and is absent from A104's MCP catalogue. An agent-made audit export and an agent-made board-pack export each raise exactly one A101 sensitive-action notice to the human and appear in `GET /v1/me/activity`. Every agent-made row shows `AgentMarker`.
- **Journey J20.** Steps 1, 2 and 6 pass in Playwright with the CCO, ENGO and SECO personas. A78 imports the page objects unchanged.
- **Design fidelity.** Token-level comparison against the node QA's crops of `adCmd`, `adApprovals`, `adAudit`, A51's ADM-02 and A51.1's `design-ext:ADM-01+`, `ADM-02+`, `ADM-30+` and `ADM-31+` shows no drift beyond 1 px other than the recorded fixes. Screenshots of every screen in every state at both widths are in `evidence/A66/` and listed in `evidence/A66.json`.
- **Quality.** axe reports zero critical or serious findings on the four routes. `npm run verify` is green from a clean clone on Node 24.21.0. Only the owned paths changed, and every spec-point (a) and (d) follow-up is merged upstream; none is dropped (a declined item escalates). `app/evidence/A66.json` validates against A00's schema and matches disk.

## Human gates touched

- None crossed. The node runs on A53's sandbox authenticator, A06's seed, the sandbox adapters and the injectable clock. No production key, vendor account, real money or mainnet is touched.
- **HG-REVIEW (before this node starts):** the founder reviews A51's partial ADM-02 frame and bulk-approve state, and A51.1's `design-ext:ADM-01+`, `ADM-02+`, `ADM-30+` and `ADM-31+` frames (role sections, full alerts centre, inbox tabs and detail additions, verification banner, on-behalf-of column, redacted row, export dialog, legal holds). This node builds only what was reviewed.
- **HG-LEGAL (recorded, not decided here):** these are A10's, A45's and A48's counsel questions, and the screens render whatever the rule packs carry:
  - Whether audited exports to auditors and regulators need extra redaction or a counsel sign-off per export.
  - The seven-year retention period per record class.
  - Whether "management figure, unaudited" KPIs may appear in an internal board pack.
  - Whether declaring an incident, verifying the audit chain or placing or releasing a legal hold must stay personal rather than delegable (today human-only in A48 and A10; any change is made in A03's `delegable.ts` and the owning node, and this node's manifest follows).
- **HG-KEYS / HG-PROD (at go-live, not now):** real staff hardware keys and the production console origin come from A11's and A53's ceremonies. Until then every approval on ADM-30 is signed with the sandbox attestation and the shell shows `SANDBOX`.
- **HG-PENTEST (later):** the inbox decision flow, the export path and the tenant view are in scope of the external penetration test before go-live. This node passes on software evidence and records the gate as open.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
