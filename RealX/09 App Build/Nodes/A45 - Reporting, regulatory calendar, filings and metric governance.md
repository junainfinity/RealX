---
title: "A45 - Reporting, regulatory calendar, filings and metric governance"
created: 2026-09-23
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am3]
node: A45
milestone: AM3
lane: "Services"
oracle: yes
human_gate: none
---

# A45 - Reporting, regulatory calendar, filings and metric governance

**The compliance back office as data. It has five parts:**
- **A regulatory obligation calendar.** It covers ASIC, ATO, DDO, licensee, CAR and internal obligations, plus AUSTRAC rows. Rows show at 7, 30 and 90 days, with owners and evidence. The AML rows are a read-only view of A42's records, and only the MLRO and CCO can see them.
- **A filing workspace.** It holds the reports that are not AML reports, for example the P2 outside-TMD report sent to the issuer within 10 business days. Each overdue filing raises one Sev2.
- **A regulator and auditor data room.** Access is time-boxed and scoped, the CCO requests it and SECO checks it.
- **Audited board and investor report exports**, and a prudential read model that includes NTA.
- **A metrics dictionary.** It defines and versions "capital raised" and can never render "tokenised value". It also supplies the published figures behind the public site and the sponsor track record.

**Build status:** not started. **Milestone:** AM3 (Business services). **Lane:** Services. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 8-12 agent-hours (build, challenge and verify loops), as in `app-graph.json`. The size is about that of A42: three local lifecycles, about 45 endpoints, three jobs, two event consumers, a formula evaluator, renderers and property tests over 2,000 grants and 20,000 requests.

**Depends on:** [[A21 - Licences, feature flags and licence perimeter (G21)]], [[A42 - AML screening, monitoring, cases and SMR clocks (SM-11)]], [[A41 - Tax engine- profiles, withholding, AMMA and CRS-FATCA]], [[A98 - Partner principals- trustee-RE, valuer, property manager, auditor, regulator observer and licensee users]], [[A43 - Communications, notifications, tasks and marketing claims (G17)]], [[A36 - Offerings- offer terms, listing review, raise monitor (SM-04)]], [[A37 - Orders, escrow, allocation and refunds (SM-05)]], [[A39 - Treasury- withdrawals, escrow release, client-money reconciliation]], [[A44 - Support, complaints (SM-12) and Investor 360 read model]].
- A43 is needed for the G17 claims check, A36 and A37 for the audited basis of capital raised, A39 for escrow and treasury figures, and A44 for complaints. All nine edges are in `app-graph.json`.

**Unlocks (ten dependents, all edges in `app-graph.json`):** [[A47 - Portfolio, holdings and transactions read models with statements]], [[A48 - Halts, kill switches and wind-down]], [[A54 - Public site- home, investments, property page, sponsor profile, fees]], [[A55 - Public site- how it works, returns calculator, transparency, help and documents]], [[A65 - Sponsor journeys J11, J12, J13, J14, J19 end to end in Docker (AM6 exit)]], [[A66 - Console- command center, alerts centre, approvals inbox, audit log]], [[A76 - Console- regulatory calendar and data room, sponsor performance, integrations and jobs]], [[A90 - P2- registered scheme, sub-funds, PDS and TMD, target market (G05), retail cooling-off]], [[A99 - Developer surface- API versioning, reference docs, per-user sandbox and SDK-MCP packaging]], [[A105 - Partner portal- trustee-RE, auditor, regulator observer and licensee views]].
- A47 composes PUB-01's audited-numbers strip and PUB-08's audited totals (`GET /v1/public/transparency`, `totals[]`) from `latestPublishedMetrics(scope)`, exported from this module's `index.ts`. A47 is the only public reader of the published platform figures.
- A48 calls `openWindDownReport` and registers the `incidents` source. The reverse edge (A48 → A45) is not added.
- A54 renders the PUB-01 strip from A47's `totals[]` and PUB-05's published block from A33's public sponsor route, which reads the `PublishedSponsorProfileProvider` this node registers. A54 calls no route of this module.
- A55 renders PUB-08 from A47's `totals[]`.
- A65 exports a sponsor report and reads the sponsor's own ADM-29 row (J19).
- A66 reads the calendar summary and the board-pack export.
- A76 renders ADM-26 and ADM-29 from this module's routes.
- A90 calls `recordOutsideTmdDealing`.
- A99 documents this module's operations in the reference docs and runs them in the per-user sandbox. `metrics_compute` never reads a sandbox partition's events.
- A105 renders the partner data-room views, the auditor's audited values and the licensee's obligation rows.

**Oracle rule:** this is an oracle node (`oracle: yes`). No money, supply or holdings move here. It is still an authority node, for three reasons: it decides which figures the public may rely on (G17), it keeps the existence of AUSTRAC reports away from everyone without need-to-know (tipping off), and it gives auditors and regulators access to records.
- The node QA writes hidden oracle tests from this spec to `<workspace>\gauntlet\hidden\A45\` before the build starts ([[App Gauntlet]] §2). The builder never reads that folder.
- The oracle uses only interfaces in this node's dependency closure. It covers five invariants:
  1. **AML projection.** No principal other than MLRO and CCO can learn that an `aml_filing`, `aml_regulatory_order` or `aml_reportable_breach` exists through any route, count, export, collection, alert, queue item or audit read of this module.
  2. **Data-room scope.** Every allow and every 404 agrees with a pure reference filter over grant and engagement scope, window, state and sealing.
  3. **Publication precheck.** A management value, a retired version, `public_allowed: false`, a forbidden label, a claims-check failure or a suspended tenant is always refused.
  4. **Mark-audited binding.** A value becomes `audited` only through an approved `4E` whose payload hash names that value's `value_sha256` and the auditor's signed letter.
  5. **M19 cells.** Exactly the M19 cells listed under "Permissions" below.
- Checks that need later nodes (A47's `totals[]` reader, A33's public sponsor route as rendered by A54, A55's PUB-08, A76's ADM-26, A105's data room) are named re-runs of these vectors at those nodes' gates.

This note is not legal advice. Every obligation in the seeded calendar (which regulator, which form, which due date, which period, who the recipient is), every clock length, whether a filing is a personal act that an agent may not lodge, what "audited" means and the wording of every metric definition are counsel questions (HG-LEGAL). Each is a row of data with a default and a `set_with_counsel` marker. Counsel's answers change rows, not code. The seed uses the design's sample sponsors and figures and nothing real.

## Build brief

**Scope and placement**

- Module `reporting` in the API (`services/api/src/modules/reporting/`) follows the A09 module pattern:
  - a public `index.ts`, the only import other modules may use, enforced by the boundary lint;
  - zod schemas that feed the OpenAPI document from the first commit;
  - a Drizzle schema and migration in the module's own folder, registered with the db package's per-module registry;
  - job definitions and event handlers exported from the module and registered with the worker through A09's job registration. This node owns no worker folder.
- Time and business days:
  - The only source of time is the injectable clock from A09. `Date.now()` and argument-less `new Date()` are forbidden in the owned path, and `npm run verify` greps for them.
  - Business-day arithmetic is never done here. Every clock is opened through A13's `openClock` and `businessDaysAfter` with the `AU-NSW` calendar, so this node inherits A13's holiday and DST behaviour.
- A13 registrations, from this module's own folder through A13's registry and never by editing A13's files:
  - the alert type `reporting.filing_overdue` (Sev2, CCO, target ADM-26) through `registerAlertTypes`;
  - the clock policies `queue.obligation_7d` and `outside_tmd` (10 business days, `set_with_counsel`) and `wind_down_report_30d` through `registerSlaPolicies`.
  - A13's own seed of 17 plus 10 types is unchanged. The registry validates these at boot.
- Everything here is the data behind surfaces owned by other nodes:
  - ADM-26 Regulatory calendar and data room, and ADM-29 Sponsor performance ([[A76 - Console- regulatory calendar and data room, sponsor performance, integrations and jobs]]);
  - the auditor and regulator observer views of the data room ([[A105 - Partner portal- trustee-RE, auditor, regulator observer and licensee views]]);
  - the published figures behind the PUB-01 audited-numbers strip and the PUB-08 audited totals, composed by [[A47 - Portfolio, holdings and transactions read models with statements]] into `totals[]` and rendered by [[A54 - Public site- home, investments, property page, sponsor profile, fees]] and [[A55 - Public site- how it works, returns calculator, transparency, help and documents]];
  - the published sponsor block behind PUB-05 (declared facts, performance lines, per-trust rows and the `about` text), served by [[A33 - Sponsor tenancy, entities, licences of sponsors and admission pipeline]]'s public sponsor route through the provider this node registers, and rendered by A54;
  - the ADM-01 compliance-clocks tile and board-pack export ([[A66 - Console- command center, alerts centre, approvals inbox, audit log]]).
  - This node fixes record shapes, column names, reason codes and metric keys, so those nodes render without translation.
- **Adapters are written here, not waited for.** Every data source this module reads is an adapter that this node writes in its own path (`reporting/sources/`) over the public `index.ts` of a module in its dependency closure:
  - A16 ledger and client-money balances;
  - A18 certified register versions and holder counts;
  - A20 reconciliation runs, breaks and deltas;
  - A21 licences, CAR appointments, conditions and the exemption meter;
  - A33 tenants and tenant state, the tenant's `about` text and SPN-01 declared facts, `sponsorSnapshotSeries(tenantId, range)` and `sponsor_watchlist` rows;
  - A34 vehicles, valuations and proof-of-property attestations;
  - A35 documents;
  - A36 offer terms, outcomes and soft caps;
  - A37 locked allocation files and refunds;
  - A39 escrow held and treasury summaries;
  - A40 distribution runs, record dates and paid dates;
  - A41 tax calendar rows (the `tax.filing.calendar` event consumer);
  - A42 `aml_filing`, `aml_regulatory_order` and `aml_reportable_breach` (read-only);
  - A43 `checkClaims`;
  - A44 complaint counts;
  - A10 scoped audit exports.
  - No module in the closure imports this module, because each was built first. Two of them reach it only through ports they defined and this module registers at start-up (below): A98's `GrantSource` and A33's `PublishedSponsorProfileProvider`.
  - Exact function names are fixed at Specify from each module's `index.ts`. If a needed read is missing (for example A33's read of a tenant's `about` text and declared facts), that is a follow-up request to the owning node (`A33-fu-A45`), never a local copy of its table.
- **Ports this module implements and registers at module start-up** (from `reporting/ports/`, through the owners' `index.ts`):
  - **A98 `GrantSource`** (`reporting/ports/grant_source.ts`), registered with A98's `registerGrantSource`:
    - `listActiveGrants(principalId, at)` returns `{grant_id, principal_id, collections[], starts_at, ends_at, state}` for every `data_room_grant` of that grantee with `state: active` and `starts_at <= at < ends_at`, where `at` is the injected clock. A grant past `ends_at` is not returned even before `reporting.dataroom_sweep` has marked it `expired`, so access ends at the second the window ends.
    - `subscribe(onGrantChanged)` keeps the subscriber list. This module calls every subscriber, in the same transaction, from the grant effector (a grant or an extension becoming `active`), from SECO's or CCO's revoke and from `reporting.dataroom_sweep` (`pending → active` at `starts_at`, `active → expired` at `ends_at`). A98 subscribes its `onGrantChanged`, drops the grant's collections from the principal's scope and emits `partner.scope.narrowed`, so A12 recomputes token rights and caps a partner token's `expires_at` at the earliest `ends_at` of the grants it relies on.
    - A boot test proves that outside the `test` profile the registered source is this module's and `FixtureGrantSource` is absent, and that before registration A98 fails closed (a TP-OBS has no scope).
  - **A33 `PublishedSponsorProfileProvider`** (`reporting/ports/sponsor_profile.ts`), registered with A33's `registerPublishedSponsorProfileProvider`. It returns, for a tenant, the `sponsor_profile` block of the latest approved `metric_publication` (see "Sponsor track record" below), with `as_of`, `publication_id` and `approval_id`, or `null` when there is none. A33 then serves `{ source: 'missing' }`.
- Three registration interfaces stay in `index.ts` for nodes built **after** this one: `registerObligationSource(source)`, `registerDataRoomCollection(collection)` and `registerMetricSource(key, fn)`. The named later callers are:
  - A48 (the `incidents` series from halts, plus `openWindDownReport`);
  - A90 (outside-TMD in P2);
  - A96 (nothing here: IFTI stays A42's).
  - A source with no provider yet returns `null` with `source_pending: true`, never a fixture value.
- Functions `index.ts` exports for dependents: `latestPublishedMetrics(scope)` (A47), `recordOutsideTmdDealing` (A90), `openWindDownReport` (A48), `openReportableSituationNotice`, `checkForbiddenLabels`, and the three registration interfaces above.
- Fixtures:
  - The fixture provider (`sources/fixture.ts`) is registered only when `RUN_MODE=local` and the test harness asks for it. The production registry refuses to boot if it is present.
  - The seed reproduces the design's figures through the real adapters over A06's seeded data, not through the fixture.

**Regulatory obligation calendar (module M19 half one)**

- `regulatory_obligation` row fields:
  - `id` (`OBL-<6 base32>`);
  - `regime` (`austrac`, `asic`, `ato`, `ddo`, `licensee_car`, `internal`);
  - `kind` (`recurring`, `one_off`, `triggered`, `derived`);
  - `title`;
  - `basis_text` (the statutory or contractual reference as plain text, `set_with_counsel`);
  - `phase` (`P1`, `P2`, `P3`). A P2 row is listed but shows `not_yet_applicable` until `PHASE` reaches it;
  - `recurrence` (`{every: 'year'|'quarter'|'month', on: 'MM-DD'|'day N', period_end_rule}` or null);
  - `due_at`, `period_start`, `period_end`;
  - `owner_role` (a role code from the 51-role list), `secondary_roles[]`, `assignee_id?`;
  - `recipient` (`austrac`, `asic`, `ato`, `issuer_re`, `licensee`, `internal`), `recipient_entity_id?` (the A33 entity of the TP-RE or TP-LIC when the recipient is not a regulator);
  - `visible_to_partner` (`none`, `TP-LIC`, `TP-RE`);
  - `evidence_doc_ids[]` (A08 storage objects), `linked_filing_id?`;
  - `source` (`seed`, `perimeter`, `tax`, `derived_issuer`, `derived_licensee`, `manual`; plus `aml` on projected rows only, never stored), `source_ref?`;
  - `state` (`open`, `in_progress`, `completed`, `waived`, `not_yet_applicable`);
  - `overdue` (a flag, never a state);
  - `completed_at/by`, `waived_by_approval_id?`, `version`, `set_with_counsel`.
- **AML rows are not stored here.**
  - SMR, TTR and IFTI items, and late-filing breaches, belong to A42 (`aml_filing` with its MLRO-only `smr-decision`, `/file`, `external_ref`, and the `filed` or `decided_no_file` decisions; `aml_reportable_breach`; the Sev1 `aml.smr_clock`).
  - This module registers a read-only projection, `aml_projection.ts`, over A42's public index. It is computed at read time and never copied into a table, so this module writes no row, event, alert or queue item about an AML item.
  - The projection covers all three A42 records: `aml_filing` (SMR, TTR, IFTI), `aml_regulatory_order` (AUSTRAC notices, law-enforcement requests and court orders, with their `due_at` typed from the order) and `aml_reportable_breach`.
  - Only MLRO and CCO get the projected rows on `GET /v1/reporting/calendar`. Each has `regime: austrac`, `kind: triggered`, `source: aml`, `source_ref` = the `aml_filing`, `ROR-…` order or breach id, A42's `clock_id`, `due_at` and `target_at`, A42's state mapped to `open` or `completed` (an order's `responded` is `completed`), and the `overdue` flag (set from A42's `late` state or a passed `due_at`). Each also carries a link to A42's case or order route. No field of the order's `content_enc` is read.
  - Projected rows never offer complete, assign, waive, edit or submit. Every write route returns 404 on their ids.
  - For every other principal, including CEO, LEGL, CFO, SUPER, every partner, every token and every agent, the projection behaves as if the rows do not exist. There is no placeholder, no count and 404 on the id.
  - The calendar summary excludes `source: aml` rows **for every viewer**. A66 takes AML clocks from A42's `GET /v1/aml/summary`, so nothing is counted twice and the summary can never reveal a count.
- **Tax rows come from A41, not the seed.**
  - A41's daily job `tax.calendar_publish` publishes its `TaxFiling` rows (AMMA, CRS/FATCA, annual investment income report, withholding remittance) through the outbox as the event `tax.filing.calendar`. This module consumes no other A41 event (`tax.amma.issued` and `tax.profile.status_changed` are for other nodes).
  - This module's consumer, `reporting.tax_calendar_intake`, is bound to `tax.filing.calendar` and is idempotent on the event id. It upserts one `derived` row per TaxFiling id with `source: tax`, `source_ref` = the TaxFiling id, and A41's TaxFiling fields `filing_type` (as the title), `period`, `due_rule`, `owner_role`, `state` and `evidence_document_id` mirrored. A replayed or older event for the same TaxFiling id (by A41's outbox sequence) never overwrites a newer state.
  - These rows are read-only here. Completion follows A41's state and never an action in this module.
  - CRS/FATCA filing is A41's `tax.crs_fatca.file` (4E TAXO → CCO, lodged through A41's `tax_authority` sandbox adapter). The calendar shows A41's request id and the adapter acknowledgement as evidence.
  - This module has no CRS/FATCA filing kind, no second lodgement and no batch check of its own.
- Seeded catalogue (`obligations.ts`, data; every due date, basis and recipient `set_with_counsel`):
  - **AUSTRAC:**
    - annual compliance report (recurring, 31 March for the prior calendar year, owner MLRO, recipient `austrac`);
    - AML/CTF program independent review (recurring, owner MLRO);
    - enrolment detail change notice (triggered, 14 days, MLRO).
    - SMR, TTR and IFTI are the A42 projection above, not seed rows.
  - **Licensee and CAR in P1:** RealX is a corporate authorised representative, so ASIC reporting of reportable situations is the licensee's obligation.
    - Seed row: "notify the licensee of a possible reportable situation". It is triggered, owner CCO, recipient `licensee`, `visible_to_partner: TP-LIC`, and the clock length is a counsel question with a default of 5 business days.
    - Each completed notification materialises a `derived_licensee` row, "licensee's report to ASIC (30 calendar days)". It is owned by TP-LIC, has `recipient: asic`, is visible to TP-LIC and RealX staff, and RealX can never complete it; TP-LIC's acknowledgement reference is shown as evidence when A105 records it.
    - Other rows: licensee notification of a CAR appointment or cessation, triggered from A21's `car.created` and `car.ended` (owner LEGL, `licensee_car`, TP-LIC visible); licensee financial statements and audit (recurring, derived from the licensee record, TP-LIC visible).
  - **ASIC in P2** (`not_yet_applicable` in P1):
    - the reportable situation lodged by RealX or the RE (30 calendar days, CCO → LEGL; which entity lodges is a counsel question);
    - the RE's compliance plan audit, scheme financial reports, and PDS and TMD reviews (LEGL).
  - **DDO in P2:** the significant dealing outside the TMD. The distributor's report goes to the issuer (recipient `issuer_re`, the scheme's RE, TP-RE visible for its own vehicles) within 10 business days of becoming aware. Its completion materialises a `derived_issuer` row, "issuer's report to ASIC (10 business days)", owned by TP-RE and visible to TP-RE and RealX staff.
  - **Licence and CAR dates:** every A21 licence, CAR appointment and condition with an `expires_on` or `review_on` appears as a `derived` row with `source: perimeter` and `source_ref` = the licence id. It is never copied and never alerted here: A21 owns the T-90/30/7 alerts, and this node only lists them.
  - **Internal:** monthly board pack, quarterly access-review evidence, and the annual business-wide risk assessment (CCO; the AML risk assessment itself stays A42's).
  - Adding a row is a data change plus a seed test. The seed test asserts the count, that every owner role exists and that no seed row has `regime: austrac` with kind `smr`, `ttr` or `ifti`.
- Horizons:
  - `GET /v1/reporting/calendar?horizon=7|30|90&regime=&owner=&state=` returns rows due within the horizon under the clock. Each row has `days_remaining`, `horizon_bucket` (`due_7`, `due_30`, `due_90`, `later`), `overdue`, `evidence_count`, `has_owner` and `read_only` (true for `perimeter`, `tax`, `derived_*` and `aml` rows).
  - The ADM-01 "compliance clocks" tile is `GET /v1/reporting/calendar/summary`, returning `{due_7, due_30, due_90, overdue, unassigned}`. The field names are the ones A66 reads. `unassigned` counts rows with no `assignee_id` due within 30 days. `source: aml` rows are excluded for every viewer.
- Job `reporting.calendar_sweep` runs daily at 06:00 `Australia/Sydney` under the clock, and on every obligation write. It:
  - materialises the next occurrence of every recurring seed row when the current one completes or passes;
  - marks `overdue` on rows past `due_at` that are not `completed`, `waived` or `not_yet_applicable`;
  - registers rows due within 7 days as an A13 `QueueSource` (`reporting.obligations`, policy `queue.obligation_7d`, owner = the row's owner role), so an unattended obligation surfaces as `queue.sla_breach` Sev3 through A13.
  - The queue source never includes `aml`, `perimeter`, `tax` or partner-owned `derived_*` rows. Their owners raise their own clocks and alerts.
  - Re-running the sweep changes nothing.
- Overdue Sev2:
  - When an A45-owned row (`seed` or `manual`) or an A45 filing first becomes overdue, the sweep emits `reporting.filing_overdue` exactly once. It goes to CCO, with the owner role as secondary, target ADM-26, and dedup key `obligation:<id>:overdue` or `filing:<id>:overdue`. It is resolved by system when the row completes or the filing is submitted.
  - It is never raised for `aml`, `perimeter`, `tax` or `derived_*` rows. A42, A21 and A41 own those clocks.
  - **No-leak rule:** this module never emits an alert, a queue item or an audit event whose subject is an AML item or names an AML case, filing or breach.
- Completion and waiver:
  - `POST /v1/reporting/obligations/{id}/complete` needs at least one evidence document, or a linked filing in `submitted` or `acknowledged`; otherwise it returns 422 `OBL_EVIDENCE_REQUIRED`. It is a single-actor act by the owner role or CCO, and it is audited.
  - Waiving is `M19.obligation.waive` (CCO maker, LEGL checker, 4E) through A14, with a reason. A waived row keeps its history, and the next recurrence still materialises.
  - Editing a seeded row's due date or basis is `M19.obligation.editSeed` (LEGL maker, CCO checker, 4E) and creates a new `version`. Open clocks and queue items that hang off the row are cancelled and re-armed at the new date.
  - `PATCH /v1/reporting/obligations/{id}` may change only `title` (on `manual` rows), `assignee_id`, `secondary_roles[]`, `evidence_doc_ids[]` and `state` between `open` and `in_progress`. Anything else is refused:
    - `due_at`, `basis_text`, `recurrence`, `regime`, `owner_role` or `recipient` on a `seed` row: 422 `OBL_FIELD_LOCKED`, naming `M19.obligation.editSeed`;
    - any field on a `perimeter`, `tax`, `derived_*` or `aml` row: 422 `OBL_READ_ONLY_SOURCE`, and 404 for `aml` rows for anyone who cannot see them;
    - an unknown field: 422 `OBL_FIELD_UNKNOWN`.

**Filing workspace (module M19 half two)**

- `filing` row fields:
  - `id` (`FIL-<yyyymmdd>-<6 base32>`);
  - `kind`: one of
    - `outside_tmd` (P2);
    - `austrac_compliance_report`;
    - `enrolment_change`;
    - `reportable_situation_notice` (P1, to the licensee);
    - `reportable_situation` (P2, to ASIC);
    - `wind_down_report`;
    - `other`.
    - There is **no** `smr`, `ttr`, `ifti` or `crs_fatca` kind. Those belong to A42 and A41, and a create request naming them returns 422 `FIL_KIND_OWNED_ELSEWHERE` with the owning module.
  - `regime`;
  - `recipient` (`austrac`, `asic`, `ato`, `issuer_re`, `licensee`) and `recipient_entity_id?`;
  - `subject_ref` (a typed reference to an order, vehicle, licence, CAR appointment or wind-down; never an AML case);
  - `obligation_id?`, `clock_id` (A13), `due_at`;
  - `prepared_by`;
  - `payload_sha256` (the canonical hash of the lodgement pack), `lodgement_pack_doc_id?`;
  - `lodgement_ref?` (the regulator's reference, or the recipient's acknowledgement reference when the recipient is the issuer or licensee);
  - `submitted_by/at`, `acknowledged_at`, `recipient_ref?`, `rejection_reason?`;
  - `evidence_doc_ids[]`, `state`, `overdue` (a flag), `version`, `set_with_counsel`.
- Lifecycle (a local table, not one of SM-00..SM-13):
  - `draft -> ready` (evidence attached and pack hashed);
  - `ready -> submitted` (lodgement reference recorded);
  - `submitted -> acknowledged` (recipient reference);
  - `submitted -> rejected -> draft`;
  - `draft|ready -> withdrawn` (reason mandatory).
  - Every other transition returns 409 `FIL_TRANSITION_NOT_ALLOWED` with the current state.
  - The sweep sets `overdue` when `due_at` passes without `submitted`. It never blocks submission, because a late filing is still a filing.
- Who submits:
  - `M19.filing.submit` is a single-actor act by LEGL or CCO for every kind, following the product map's "Submit filing".
  - Lodgement is a human act on the recipient's own channel: the regulator's portal, or delivery to the issuer or licensee. This node produces the lodgement pack (file plus hash into A08 storage), and the human types the lodgement or acknowledgement reference. The build never calls a regulator, an issuer or a licensee.
  - In `RUN_MODE=local` only, `POST /v1/_test/reporting/acknowledge/{id}` plays the recipient, so e2e and the node QA can close a filing.
- Producers through the public interface. Each links to the obligation row and opens its A13 clock:
  - `recordOutsideTmdDealing({orderId, vehicleId, tmdId, reasonCode, awareAt})`:
    - creates an `outside_tmd` filing in `draft` with `recipient: issuer_re` (the vehicle's RE, from A36 and A33) and `lodgement_ref` = the issuer's acknowledgement reference;
    - opens a 10-business-day A13 clock from `awareAt` (policy `outside_tmd`, `set_with_counsel`);
    - emits `ddo.outside_tmd` (Sev2, CCO, ADM-26; an A13 catalogue type);
    - links the DDO obligation row.
    - It is guarded by `PHASE >= P2` or `RUN_MODE=local`. In P1 nothing calls it except the fixture and the node QA. [[A90 - P2- registered scheme, sub-funds, PDS and TMD, target market (G05), retail cooling-off]] calls it in P2, and A90 then moves the filing to `ready` (significant) or `withdrawn` (not significant).
    - This is J16 step 5, and the catalogue's oracle hook "outside-TMD dealing creates a report item with a 10-business-day clock".
  - `openWindDownReport({vehicleId, windDownId, dueAt, packProvider})`:
    - [[A48 - Halts, kill switches and wind-down]] calls it at `wound_down`. A48 names the task `filing.wind_down_report`.
    - It creates a `wind_down_report` filing owned by LEGL with `recipient` set by counsel (default `asic`). The due date comes from A48 (30 days, `set_with_counsel`, policy `wind_down_report_30d`).
    - The lodgement pack comes from A48's `packProvider`.
    - Retention belongs to A48 (`trust_retention`, `retentionHold()`) and A10. This node writes no retention flags. The filing records A48's retention reference as a read-only field.
    - This is J21 step 7.
  - `openReportableSituationNotice({subjectRef, awareAt})` is called by staff from ADM-26, and by A44 or A48 when they build a hook (follow-up).
- Partner visibility:
  - TP-LIC sees `reportable_situation_notice` filings addressed to it (state, dates, pack hash, and the pack file when submitted), and the derived licensee rows.
  - TP-RE sees `outside_tmd` filings addressed to it for its own vehicles.
  - No other partner sees filings outside a data-room grant.

**Regulator and auditor data room**

- `data_room_grant` row fields:
  - `id` (`DRG-<6 base32>`);
  - `grantee_user_id`: an A98 partner principal with role TP-AUD, TP-OBS or TP-LIC. An investor, sponsor user or staff member is refused with 422 `DR_GRANTEE_NOT_PARTNER`;
  - `purpose` (text, mandatory);
  - `scope` (`{tenant_ids[], vehicle_ids[], collections[], date_from, date_to}`). An empty `collections[]` is refused; there is no "everything" scope;
  - `starts_at`;
  - `ends_at`: default 30 days after `starts_at`, hard maximum 90 days as a platform setting; longer returns 422 `DR_WINDOW_TOO_LONG`;
  - `granted_by_approval_id`;
  - `state` (`pending`, `active`, `expired`, `revoked`);
  - `revoked_by/at`, `revoke_reason`, `watermark_text`, `views`, `downloads`, `last_accessed_at`.
- Collections (`collections.ts`, data). Each is served by this node's own adapter:
  - `register_extracts`: A18 certified versions. Holder names and identifiers are rendered only through A04's `mask()`.
  - `recon_runs`: A20 runs and breaks.
  - `filings`: this module's filings. It never includes an AML item, because there are none here.
  - `obligations`: the calendar, with `source: aml` rows excluded structurally.
  - `licences`: A21 rows and evidence.
  - `board_reports` and `investor_reports`.
  - `audit_exports`: A10 scoped exports produced on request, never the live log, with need-to-know events redacted by A10's ACL.
  - `financials`: A16 ledger and A39 treasury summaries, as aggregates only.
  - `aml_program`: A35 documents of kind `aml_program`. These are program documents, never cases, filings or breaches.
  - Each collection is a `DataRoomCollection` provider with `list(scope, cursor)` and `get(scope, id)`. A provider that does not filter by the grant's tenants, vehicles and date range fails its conformance test.
  - Every collection declares its column list. The export minimiser drops every field not on that list and applies `mask()` to person fields. A102 replaces the default minimiser with `minimiseForExport` through `registerExportMinimiser` when it is built.
  - There is no investor-level PII in clear in any collection, file or watermark.
- Granting:
  - It is `M19.dataRoomGrant.request` (CCO maker) with `M19.dataRoomGrant.approve` (SECO checker, 4E) through A14.
  - The precheck refuses a grantee without an active A98 principal, a window past the maximum, a scope naming a collection that does not exist, a start earlier than the approval, and a grantee who is the requesting person (`DR_SELF_GRANT`).
  - Extending is a new `M19.dataRoomGrant.extend` with `M19.dataRoomGrant.extendApprove` (CCO → SECO, 4E) and the same checks. The old grant is never edited in place.
- Revoking and expiry:
  - Revoking is `M19.dataRoomGrant.revoke`, a single-actor protective act by SECO or CCO. It takes effect within one clock tick and needs no ratification, because it opens nothing. The grantee's next request gets 404.
  - Expiry is the job `reporting.dataroom_sweep`, every minute under the clock. It moves `active` grants past `ends_at` to `expired`.
  - Both paths, and the grant effector, notify every `GrantSource` subscriber (A98's `onGrantChanged(grant)`) in the same transaction, so A98 drops the grant's collections from the principal's scope and A12 recomputes the principal's token rights. Access checks never wait for the sweep: every data-room route checks the window against the injected clock on each request, as `listActiveGrants` does.
- Two access paths:
  - **Grant path.** `GET /v1/dataroom/grants/{id}/collections/{code}`, `/items/{itemId}` and `/items/{itemId}/download`, for the grantee, scoped by the grant.
  - **Engagement path.** `GET /v1/dataroom/engagement/collections/{code}`, `/items/{itemId}` and `/items/{itemId}/download`, for a TP-AUD whose A98 engagement lists collection codes (`partnerScope(principalId).collections[]`). The scope is the engagement's tenants, vehicles and time box.
  - A partner with neither an active grant nor an engagement collection has no data-room endpoints at all.
- Audit of reads is one event per read, matching A98's rule:
  - every listed page writes exactly one A10 event of A98's type `partner.read`, with `grant_id` or `engagement_id`, the collection code, the item refs on the page and A12's `on_behalf_of` when a token is used;
  - every item view (`/items/{itemId}`) writes exactly one `partner.read` with the same fields and the single item ref. It returns the item and a short-lived A08 link (5 minutes) to the watermarked rendering for the viewer; fetching that link writes no event, because the view is already audited;
  - every download (`/items/{itemId}/download`) writes exactly one `dataroom.item_downloaded` event with the same fields, and no `partner.read`.
  - This module writes no separate "viewed" event. If A98 implements `partner.read` as a generic middleware, it must skip `/v1/dataroom/*` so that the count stays one; this is confirmed at Specify. A105's count (one `partner.read` per listed page and per item view, none for the viewer's file fetch, one `dataroom.item_downloaded` per download) is the rule here, not an assumption.
- Files are served from A08 storage with the watermark text ("Provided to <grantee> under grant DRG-… · <date>", or "under engagement PEN-…") and never as the original object.
- A grantee asking for an item outside the scope, outside the window, or among AML records gets 404, never 403.
- Staff see the same grants and their activity on ADM-26. The grantee sees only its own grants on the partner portal (A105).

**Board and investor report exports (audited)**

- `report_export` row fields:
  - `id` (`REP-<yyyymmdd>-<6 base32>`);
  - `kind` (`board_pack`, `investor_report`, `prudential_dashboard`, `sponsor_report`);
  - `period_start/end`, `scope` (platform, tenant or vehicle);
  - `template_version`, `metric_publication_id?`, `metric_snapshot_sha256`;
  - `figure_basis`: `audited` for anything external; `management` is allowed only for `board_pack` and `prudential_dashboard`;
  - `file_sha256`, `doc_id` (A08 storage);
  - `requested_by`, `on_behalf_of?`;
  - `state` (`queued`, `rendered`, `failed`).
- Exporting is a single-actor act that is audited, not approved: `M19.boardPack.export` and `M19.report.export` write `report.exported` with the hash and the metric versions used. This matches the catalogue's "Export board pack (audited)".
- Rendering rules:
  - A render pulls every figure through the metric renderer below.
  - A template that names a forbidden label, or a metric key that does not exist, fails the render with 422 `MET_RENDER_REFUSED` and an audit event, never a document with a blank.
  - Every management figure in a board pack carries the printed label "management figure, unaudited" beside it.
  - An `investor_report` or `sponsor_report` with any management figure is refused.
  - Investor-level rows in any export are masked and minimised as in the data room.
- The prudential dashboard is a read model, `GET /v1/reporting/prudential`. It is what CCO reviews in J20 step 1 alongside ADM-01. It holds:
  - the client-money delta and last reconciliation (A20);
  - escrow held per open offer (A39 and A37);
  - the DAF Act exemption meter and licence states (A21);
  - open and overdue A45 filings;
  - active data-room grants;
  - the audit-chain head status (A10);
  - an `nta` block: `{applicable, state, value_cents, requirement_cents, basis, as_of, basis_text}`.
    - `applicable` is a `set_with_counsel` row, false in P1.
    - `state` is one of `not_applicable_car`, `met`, `shortfall`, `not_computed`.
    - `value_cents` is net tangible assets from the A16 ledger's RealX own-account balances. `requirement_cents` is the counsel-set requirement. `basis` is `management` until attested.
    - In P1 the block reads `state: not_applicable_car` with the text "Not applicable in P1: RealX operates as a corporate authorised representative; financial requirements sit with the licensee", with value and requirement null.

**Metrics dictionary and publication (N16, G17)**

- `metric_definition` row fields:
  - `key`: snake case, from a closed vocabulary seeded here: `capital_raised`, `distributions_paid`, `trusts_on_realx`, `investors_active`, `distributions_on_time_bps`, `yield_plan_vs_paid_bps`, `proof_score_bps`, `sponsor_keeps_bps`, `reconciliation_last_clean_at`, `client_money_delta_cents`;
    - `trusts_on_realx` is a platform-scope count of **publicly visible** trusts only (a listing hidden from the public site is never counted), so a published figure can never reveal a hidden listing. This node does not publish a tenant-scope `trusts_on_realx`: the per-sponsor "Trusts on RealX" fact is A33's live fact of record (see "Sponsor track record");
    - tenant-scope values of the sponsor keys are computed from A33's `sponsorSnapshotSeries`, never from a second copy of A33's inputs;
  - `name`;
  - `definition_text`: plain English, the sentence shown when a user hovers or expands the figure;
  - `formula`: a declarative expression over named source series, validated at insert against the registered sources;
  - `unit` (`aud_cents`, `count`, `bps`, `timestamp`). There is no `pct` unit: every ratio is stored in basis points, so 97.5 percent is 9750. This is the one unit convention proposed for A33, A45 and A54 (see "Sponsor track record");
  - `sources[]`;
  - `audited_basis_rule`: which computation counts as audited. For `capital_raised` it is allotted subscriptions in AUD received into the client-money trust account (A16) against locked allocation files (A37) for offers that closed at or above soft cap (A36), net of refunds (A37), taken from certified register versions (A18). Open-offer escrow, sponsor co-investment terms and anything derived from valuations are excluded; the first two are `set_with_counsel`;
  - `public_allowed`, `version`, `effective_from`, `changed_by_approval_id`;
  - `state` (`draft`, `active`, `retired`).
  - Changing a definition is `M19.metricDefinition.change` (CFO maker) with `M19.metricDefinition.changeApprove` (CCO checker, 4E) through A14. It always creates a new version. Old versions are never edited, and every stored value and every publication names the version it used.
  - The precheck runs A43's `comms.checkClaims` on the new `name` and `definition_text` when `public_allowed` is true.
  - A definition change also checks `unit` against the key's closed unit: a ratio key with any unit other than `bps` is refused (`MET_UNIT_INVALID`).
- Forbidden labels (`forbidden_labels.ts`, data, `set_with_counsel`):
  - The labels are "tokenised value", "tokenized value", "total value tokenised", "TVL", "assets tokenised", plus any metric that sums property valuations across vehicles.
  - The renderer, the export templates, the public endpoints and the OpenAPI examples are all checked. A label match anywhere in a rendered string returns 422 `MET_FORBIDDEN_LABEL`.
  - `npm run verify` greps the owned path and `packages/ui` for the strings outside the data file.
  - The function `checkForbiddenLabels(text)` is exported so later nodes (A54, A55) can reuse it.
  - Valuation figures remain available per vehicle (the design's "Token valuation" on PUB-04), because they are a property of one trust, not a platform total.
- `metric_value` row fields:
  - `id` (`MVL-<6 base32>`), `metric_key`, `version`;
  - `scope` (`platform`, `tenant:<id>`, `vehicle:<id>`), `as_of`;
  - `value`: an integer in cents, count or basis points, never a float;
  - `basis` (`audited`, `management`);
  - `inputs_sha256`, `value_sha256` (the canonical hash of key, version, scope, as_of, value and inputs_sha256);
  - `computed_by` (job run id), `audit_letter_doc_id?`, `marked_audited_approval_id?`.
- Job `reporting.metrics_compute` runs daily at 06:30 Sydney, and on demand. It recomputes every active metric from its sources and stores one row per scope. Recompute and on-demand paths must agree to the unit. A source with `source_pending` yields `value: null`, never a fixture number.
- **Marking a value audited (settled at plan level: option (b), a signed auditor letter).**
  - Partners are read-only (A98's structural rule), so TP-AUD has no write route here.
  - A value becomes `audited` only through `M19.metricValue.markAudited` (CFO maker) with `M19.metricValue.markAuditedApprove` (CCO checker, 4E) through A14. The request names the value id and an A35 document of kind `audit_signoff`, the auditor's signed letter. The canonical payload is `{value_id, value_sha256, audit_letter_doc_id, audit_letter_sha256, auditor_entity_id, audit_letter_signed_at}`, where `audit_letter_signed_at` is the signing date from A35's document metadata; it becomes the figure's `attestation.attested_at` in `latestPublishedMetrics`.
  - The effector refuses:
    - a value whose `value_sha256` changed since the request (`MET_VALUE_CHANGED`);
    - a letter whose document is not of kind `audit_signoff` or whose `auditor_entity_id` is not an active A98 `audit_firm` entity (`MET_LETTER_INVALID`);
    - a maker or checker who is the same person (A14).
  - When it is approved, the auditor's TP-AUD principals are notified through A43 ("RealX has marked <metric> as audited citing your sign-off"). They can read every value bound to their firm's letters at `GET /v1/metrics/values?audited_by=me`, read-only, so the auditor can object off-platform.
  - Whether a live auditor should instead attest on the portal is the HG-VENDOR and HG-LEGAL question below. If counsel chooses that later, it is an A03 and A98 change, not a change here.
  - `POST /v1/metrics/values/{id}/attest` does not exist.
- Publication:
  - `metric_publication` freezes one approved set for the public site and the sponsor profiles. It holds:
    - `figures[]` at platform scope, each `(metric_snapshot_id, key, version, scope, value, unit, basis, as_of, label, marked_audited_approval_id, audit_letter_doc_id, audit_letter_signed_at)`, where `metric_snapshot_id` is `MPI-<6 base32>`, the id A47's `totals[]` and A33's published block cite;
    - `sponsor_profiles[]`, one `sponsor_profile` block per tenant (see "Sponsor track record");
    - `period_label`, `as_of`, the claims report and its ruleset version.
  - Creating one is `M19.metricPublication.publish` (IRCM maker) with `M19.metricPublication.publishApprove` (CFO and CCO checkers, 6E; baseline row 44) through A14. This is the only action id for publishing figures; A43's G17 hook and A47's reader use this id (see the upstream corrections in the Gate step).
  - The precheck refuses:
    - any value with `basis: management`;
    - any retired version;
    - any key with `public_allowed: false`;
    - any sponsor figure or sponsor block whose tenant is suspended;
    - any forbidden label;
    - any string that fails A43's `comms.checkClaims` against the current ruleset. It checks every `label`, the `period_label`, the `name` and `definition_text` of every metric in the set, and every string in every `sponsor_profile` block (the `about` text, each declared fact's label and value, each per-trust row's text): the exact strings the public pages render. The report and its ruleset version are stored on the publication. A43 also registers its own G17 precheck on `M19.metricPublication.publish` over the same strings; either refusal blocks.
    - This is the product map's "Claims reviewed" result, and it is the G17 gate for numbers and for sponsor-declared facts.
- **Published figures for the public site: `latestPublishedMetrics(scope)`.** This function, exported from `index.ts`, is the only way a published figure leaves this module for a public page. There is no public metrics route in this module; A47's `GET /v1/public/transparency` is the one public route and composes PUB-01 and PUB-08 from it.
  - It returns `{publication_id, approval_id, published_at, period_label, figures: [{key, value, unit, as_of, basis: 'audited', metric_snapshot_id, attestation: {kind: 'auditor', ref, attested_at}}]}` or `null` when nothing is published. `ref` is the `audit_letter_doc_id` of the auditor's signed letter and `attested_at` is `audit_letter_signed_at`.
  - A47 maps each figure to one `totals[]` entry `{metric: key, value_cents (unit aud_cents) | value, as_of, attestation}` without translation.
  - It returns the latest approved publication only: never a live value, never a management value, never a retired version. A figure without an attestation cannot exist in a publication, because the precheck refuses management values.
  - Captions and the empty state are the page nodes' copy (A54 "audited · as at {as_of}", A55's R10 caption and empty-state sentence), checked by their copy lint. This node supplies only the fields.
  - The staff route `GET /v1/metrics/publications/latest` returns the same publication with its claims report for ADM-26; no public page reads it.

**Sponsor track record (ADM-29 and PUB-05)**

This section follows A33 section 7 (the public contract) and A33's instruction 7 to this node. The split is recorded here and in A33's spec, and no field has two owners.

- **ADM-29 (staff, and SP-OWN's own row).** `GET /v1/reporting/sponsor-performance` composes one row per tenant from A33's latest snapshot (`sponsorSnapshotSeries`), A33's open `sponsor_watchlist` rows, A33's tenant state and this module's audited values. It returns the 8 ADM-29 columns plus `late_distributions_6m`, `incidents` and `complaints`, and no other field:
  - `sponsor`;
  - `trusts`: vehicles on RealX with a live or closed offer, including a listing hidden from the public site. This column is staff-only (and SP-OWN's own row); it is never published;
  - `raised_cents`: audited `capital_raised` at tenant scope, or null with `basis: management` when not marked audited;
  - `yield_plan_vs_paid_bps`;
  - `on_time_bps`: A33's `distributions_on_time_pct` (a run is late when paid more than 10 business days after the record date);
  - `proof_score_bps`: A33's `proof_of_property_score`;
  - `keeps_bps`: A33's `retained_stake_bp`, against the proposed minimum of 1,000 bps;
  - `flag`: `suspended` when A33's tenant state is `suspended`, otherwise `watch` when A33 holds an open `sponsor_watchlist` row, otherwise `none`. This node has no watch rule of its own: the rule (two late distributions in six months, or a retained stake under the minimum) is A33's;
  - `late_distributions_6m`, `incidents` and `complaints`: A33's `late_distributions_6m`, `incidents` and `complaints_open`. A field A33 marks `source: missing` is null with `source_pending: true`, never zero.
  - SP-OWN reads its own tenant's row at `GET /v1/reporting/sponsor-performance/me`, under row-level security; another tenant's row is 404.
- **PUB-05: A33 serves, this node publishes.** A33 serves `GET /v1/public/sponsors/{slug}` and owns every facts-of-record field live, including "Trusts on RealX" (A33's live fact; it must count publicly visible trusts only, see the upstream corrections) and the `Licence` fact. This node publishes the rest, and has no public sponsor route of its own.
  - Each `metric_publication` carries one `sponsor_profile` block per tenant with an active admission:
    - `about` (from A33's tenant record);
    - `declared_facts[]`: `Trusts managed`, `Assets managed` and `years_operating`, each `{label, value, unit, source: 'sponsor_declared', declared_at}`, copied from the tenant's SPN-01 declaration at draft time;
    - `performance`: `capital_raised_cents`, `distributions_on_time_pct`, `proof_of_property_score`, `retained_stake_bp`, `incidents` and `complaints_resolved`, each `{value, unit, as_of, metric_snapshot_id}` from audited tenant-scope values (A33's field names, carried unchanged);
    - `trusts[]`: one row per publicly visible trust with planned versus actual yield, forecast versus actual growth and the retained stake;
    - `as_of`, `publication_id`, `approval_id`.
  - Every string in the block passes `checkClaims` in the 6E precheck (above). A performance line whose value is not audited is `null` in the block, never a management value.
  - A33 reads the block only through the `PublishedSponsorProfileProvider` this node registers. A newer A33 snapshot never changes a published block.
- **One unit convention.** Every ratio in this module, in the published block and in ADM-29 is an integer in basis points (9750 = 97.5 percent), and every line in the block carries its `unit` (`bps`, `aud_cents` or `count`), so no reader guesses. A33's field names `distributions_on_time_pct` and `proof_of_property_score` therefore hold basis points. A33 and A54 must record that (or rename the fields with a `_bp` suffix); see the upstream corrections.

**HTTP API (all under `/v1`, cursor pagination, idempotency keys on writes, in the OpenAPI document from the first commit)**

- Calendar:
  - `GET /v1/reporting/calendar`, `GET /v1/reporting/calendar/summary`;
  - `GET|POST /v1/reporting/obligations` (POST creates `manual` rows), `GET|PATCH /v1/reporting/obligations/{id}` (PATCH fields as listed above);
  - `POST /v1/reporting/obligations/{id}/complete | assign | waive | edit-seed` (waive and edit-seed open A14 requests);
  - `GET /v1/reporting/obligations/{id}/history`.
- Filings:
  - `GET|POST /v1/reporting/filings`, `GET /v1/reporting/filings/{id}`;
  - `POST /v1/reporting/filings/{id}/ready | submit | acknowledge | reject | withdraw`;
  - `GET /v1/reporting/filings/{id}/pack`.
- Data room:
  - `GET|POST /v1/dataroom/grants` (POST opens `M19.dataRoomGrant.request`);
  - `POST /v1/dataroom/grants/{id}/extend | revoke`, `GET /v1/dataroom/grants/{id}/activity`;
  - `GET /v1/dataroom/grants/{id}/collections/{code}`, `GET /v1/dataroom/grants/{id}/items/{itemId}`, `GET /v1/dataroom/grants/{id}/items/{itemId}/download`;
  - `GET /v1/dataroom/engagement/collections/{code}`, `GET /v1/dataroom/engagement/items/{itemId}`, `GET /v1/dataroom/engagement/items/{itemId}/download`;
  - `GET /v1/dataroom/collections` (the catalogue).
- Reports: `GET|POST /v1/reports/exports`, `GET /v1/reports/exports/{id}`, `GET /v1/reporting/prudential`.
- Metrics:
  - `GET /v1/metrics/definitions`, `POST /v1/metrics/definitions/{key}/versions`;
  - `GET /v1/metrics/values?key=&scope=&as_of=&basis=&audited_by=me`, `POST /v1/metrics/values/{id}/mark-audited`;
  - `GET|POST /v1/metrics/publications`, `GET /v1/metrics/publications/latest`;
  - `GET /v1/reporting/sponsor-performance`, `GET /v1/reporting/sponsor-performance/me`.
  - There is no public route in this module. Published figures reach the public only through A47 (`latestPublishedMetrics`) and A33 (`PublishedSponsorProfileProvider`).
- Local test routes, in `RUN_MODE=local` only: `POST /v1/_test/reporting/acknowledge/{id}`, `POST /v1/_test/reporting/outside-tmd` and `POST /v1/_test/reporting/metric-inputs`, so e2e and the node QA can drive filings and figures without another module.
- **Permissions.** The scope names are A03 action ids (`scopeId === actionId`, grammar `<module>.<object>.<verb>`).
  - This node does not own A03's catalogue, and A03's spec has no M19 or M24 row today. The table below is this node's **full change list for the follow-up loop `A03.1-A45`** (loop A03.1 on A03, keyed by this requester, as [[Build Orchestration]] §6 describes). The change covers:
    - every action id below, added to `actions.ts` and `action-sources.ts`: a PM row maps to its product-map reference (ADM-26 "Grant regulator or auditor access", "Change a metric definition" and "Submit filing", ADM-01 "Export board pack", ADM-29, TP-OBS and TP-AUD reads); a proposal row is added as an extension marked `proposal`;
    - the cells in `matrix.ts` and the flags in `delegable.ts` exactly as below;
    - the 4E and 6E baseline and resolution rows for `M19.obligation.waive`, `M19.obligation.editSeed`, `M19.dataRoomGrant.request`, `M19.dataRoomGrant.extend`, `M19.metricDefinition.change`, `M19.metricValue.markAudited` and `M19.metricPublication.publish` (baseline row 44), with the matching A14 `approval_policy` seed rows;
    - the updated exported counts (distinct actions, source references, resolution entries) in A03's export and the re-run of A03's hidden oracle against them.
  - At this node's Specify step the orchestrator looks up `A03.1-A45` in `status.json` and, if it has not passed, runs it through the gauntlet before this node's Oracle step. A45 does not start its build until that loop has passed.
  - "PM" marks a product-map row. "Proposal" marks a founder proposal that defaults as shown until the founder confirms it.
  - "Delegable" is A03's flag: `false` means no token scope exists; `counsel` behaves as `false` until counsel answers.

| Action id | Kind | Actor (maker) | Checker, policy | Delegable | Source |
|---|---|---|---|---|---|
| `M19.calendar.read` | read | CCO, LEGL, CFO, MLRO (PM); TAXO, TRSY, CEO, IRCM, SECO, SUPER (proposal; AML rows stay hidden from all but MLRO and CCO); TP-LIC own `licensee` rows, TP-RE own vehicles' `issuer_re` rows (proposal) | none | true | ADM-26 roles |
| `M19.obligation.create` | draft | CCO, LEGL | none | true | proposal |
| `M19.obligation.update` | draft | owner role, CCO | none | true | proposal |
| `M19.obligation.assign` | submit | owner role, CCO | none | true | proposal |
| `M19.obligation.complete` | submit | owner role, CCO | none | true | proposal |
| `M19.obligation.waive` / `.waiveApprove` | draft / approve | CCO | LEGL, 4E | true / false | proposal |
| `M19.obligation.editSeed` / `.editSeedApprove` | draft / approve | LEGL | CCO, 4E | true / false | proposal |
| `M19.filing.prepare` (create, ready, withdraw, record reject or acknowledgement) | draft | LEGL, CCO | none | true | proposal (supports PM "Submit filing") |
| `M19.filing.submit` | submit | LEGL, CCO | none | counsel (`FIL_PERSON_ONLY`) | PM ADM-26 "Submit filing" |
| `M19.dataRoomGrant.request` / `.approve` | draft / approve | CCO | SECO, 4E | true / false | PM ADM-26 "Grant regulator or auditor access" |
| `M19.dataRoomGrant.extend` / `.extendApprove` | draft / approve | CCO | SECO, 4E | true / false | proposal |
| `M19.dataRoomGrant.revoke` | act (protective) | SECO, CCO | none | false | proposal |
| `M19.dataRoom.read` | read | TP-OBS, TP-AUD, TP-LIC within a grant; TP-AUD within engagement collections | none | true | PM TP-OBS "View the regulator data room", TP-AUD "Read registers, ledgers and approvals" |
| `M19.boardPack.export` | export | CFO, CCO (PM); CEO, LEGL (proposal) | none | true | PM ADM-01 "Export board pack" |
| `M19.report.export` | export | CFO, CCO; SP-OWN, SP-FIN own tenant `sponsor_report` (proposal) | none | true | proposal |
| `M19.prudential.read` | read | CCO, CFO, CEO | none | true | PM ADM-26 component |
| `M19.metrics.read` | read | CCO, CFO, IRCM, CEO, LEGL; TP-AUD values bound to its firm's letters | none | true | PM ADM-26 component |
| `M19.metricDefinition.change` / `.changeApprove` | draft / approve | CFO | CCO, 4E | true / false | PM ADM-26 "Change a metric definition" |
| `M19.metricValue.markAudited` / `.markAuditedApprove` | draft / approve | CFO | CCO, 4E | counsel (`MET_ATTEST_PERSON_ONLY`) / false | proposal (replaces a TP-AUD attest) |
| `M19.metricPublication.publish` / `.publishApprove` | draft / approve | IRCM | CFO + CCO, 6E | true / false | PM ADM-26 and baseline row 44 |
| `M24.sponsorPerformance.read` | read | PART, ASTM, CCO, CFO (PM); SP-OWN own row (proposal) | none | true | PM ADM-29 |

- Investors have no endpoint in this module; they see published figures only through A47 and A33. A tenant or partner user asking for another tenant's or partner's row gets 404, never 403. Nothing on M19 grants any cell on AML rows: the projection checks A42's own ACL (MLRO and CCO) and nothing in this matrix.
- **Agent parity (D04, G22).**
  - Every endpoint is in the public OpenAPI document. A staff member's, sponsor user's or partner's agent token holding the matching delegable scope can do exactly what its human can do as maker: list, draft, ready, assign, complete, export, and open a grant request, a definition change, a mark-audited request or a publication.
  - Checking stays with A14 and is refused to tokens there (`APR_CHECKER_MUST_BE_HUMAN`).
  - Three rules are person-only by default. Each is a `set_with_counsel` row, so counsel can relax or extend it:
    1. submitting any filing to a regulator, issuer or licensee (`FIL_PERSON_ONLY`, because a lodgement is a legal declaration by a named person);
    2. drafting a mark-audited request (`MET_ATTEST_PERSON_ONLY`);
    3. no token, and no person, may draft `M19.dataRoomGrant.request` or `.extend` naming as grantee the token's own person (`DR_SELF_GRANT`).
  - Revoking a grant is an `act` kind and has no token scope.
  - A regulator observer's or auditor's agent may read the data room within the grant or engagement. Every such read is audited as the token on behalf of the person and appears in the token's activity (A12).
- **Sensitive agent actions.**
  - The following audit events carry A03's `sensitive` class and the actor type (AGTU or AGNT) with `on_behalf_of`, so that [[A101 - Agent guard- anomaly detection, sensitive-action notices and tiered kill switches]] raises the person's sensitive-action notice through A43 and runs its anomaly rules:
    - an agent drafting a data-room grant or extension, a metric definition change, a mark-audited request or a publication;
    - an agent exporting a board pack or any report;
    - a partner's agent reading or downloading from the data room.
  - This module sends no notice itself and never to the agent.
  - A101's intake reads audit events with `actor_type = AGNT`. If AGTU is not included there, A101 must add it; this is confirmed at Specify.
- **Audit.** There is exactly one A10 event per write and per system act:
  - obligations: `obligation.created`, `obligation.materialised`, `obligation.assigned`, `obligation.completed`, `obligation.waived`, `obligation.seed_edited`, `obligation.overdue`, `obligation.derived_upserted` (one per A41 or A21 intake event, not per read);
  - filings: `filing.created`, `filing.ready`, `filing.submitted`, `filing.acknowledged`, `filing.rejected`, `filing.withdrawn`, `filing.overdue`;
  - data room: `dataroom.granted`, `dataroom.extended`, `dataroom.revoked`, `dataroom.expired`, `partner.read` (A98's type, written by the data-room routes as above), `dataroom.item_downloaded`;
  - reports: `report.exported`, `report.render_refused`;
  - metrics: `metric.definition_versioned`, `metric.values_computed` (one per job run with the inputs hash, not one per value), `metric.marked_audited`, `metric.published`, `metric.render_refused`.
  - No event names an AML case, filing or breach. A10's PII detector is expected to find nothing in any reason or payload.
- Empty and error states:
  - An empty calendar is `{items: [], next: null}`, and the summary is all zeros.
  - A filing with no evidence cannot become `ready` (422 `FIL_EVIDENCE_REQUIRED`).
  - A submit without a lodgement reference returns 422 `FIL_LODGEMENT_REF_REQUIRED`.
  - A grant request for an expired A98 principal returns 422 `DR_PRINCIPAL_INACTIVE`.
  - An export with no publication and `figure_basis: audited` returns 409 `REP_NO_AUDITED_FIGURES`, naming the missing keys.
  - A metric with no computed value returns `value: null, basis: null`, never zero.
  - A publication attempt with nothing marked audited is refused at precheck with the list of management-only keys.
  - An unknown metric key is 404.

## Owned paths (other nodes do not edit these)

- <workspace>\app\services\api\src\modules\reporting\

There is no cross-path exception. A13 alert types and clock policies are added through A13's registry from this folder.
- ADM-26 and ADM-29 pages belong to A76, and ADM-01 tiles to A66.
- The partner data-room views belong to A105.
- PUB-01 and PUB-05 rendering belongs to A54, and PUB-08 to A55. A47 composes `GET /v1/public/transparency` (`totals[]`) from `latestPublishedMetrics`.
- A33 owns the sponsor's facts of record (including "Trusts on RealX"), the capture of sponsor-declared facts, snapshots, the watchlist and the public sponsor route. This node owns only the approved publication of the declared facts and performance lines, served through A33's provider port.
- A98 owns partner principals, engagements, their sessions and the `GrantSource` port; this node owns its implementation over `data_room_grant`.
- A42 owns every SMR, TTR and IFTI item, its clock and its lateness.
- A41 owns the tax filings and their lodgement.
- A48 owns wind-down and retention.
- **M27 fee schedules are not this node's.** A33 owns tenant schedules (`tenant.fee_schedule.set`). The platform schedule (baseline row 49, CFO → CEO 4E, with its notice period, and the PUB-06 read) is proposed for A33 alongside the tenant schedules, because it has the same shape and notice rules. A33's and A21's specs must record that.

## Design references

- `design-ext:ADM-26` from [[A51 - Design extension B- operator console screens not in the prototype]] (design analysis §5): the dark admin shell page with the tabs Calendar / Filings / Data room / Metrics.
  - **Calendar:** FilterChips (7 d / 30 d / 90 d / Overdue, by regime) plus a DataTable with the columns Obligation / Regime / Recipient / Due / Owner / Evidence / State.
    - State pills: open grey, in progress amber, overdue red, completed green, waived grey.
    - "Derived from licence LIC-…", "From tax filing TAX-…" and "Issuer's report" rows show a muted mono source caption and no action buttons.
    - For MLRO and CCO only, AML rows show "AUSTRAC · A42" with a link to the case page, and no action buttons. For everyone else they are absent, not greyed.
  - **Filings:** a split DataTable (ID / Kind / Recipient / Subject / Due / Submitted / State) with a detail card, a KeyValueList of pack hash, lodgement reference, recipient reference and clock, and the buttons Mark ready / Submit / Withdraw.
  - **Data room:** KeyValueList cards per grant (Grantee / Purpose / Scope / Window / Granted by / State) with an activity TaskRows list and a Revoke danger-outline button in alert red `#A3312A`.
  - **Metrics:** a DataTable (Key / Name / Version / Basis / Public / Last value) with the definition text expanded under the row, a "Mark audited" secondary that opens the 4E with the letter picker, and a "Publish figures" primary that opens the 6E request.
  - **Prudential panel:** includes the NTA row with its `not_applicable_car` text.
  - This node fixes those column names, pill vocabularies and reason codes, so A51 draws them and A76 renders them without renaming.
- `design-ext:ADM-29` (the same extension node): a DataTable with the 8 columns Sponsor / Trusts / Raised / Yield plan vs paid / On-time / Proof score / Keeps / Flag.
  - Each column is a field of `GET /v1/reporting/sponsor-performance`. The basis-point fields are displayed as percentages with one decimal. The three extra fields (`late_distributions_6m`, `incidents`, `complaints`) feed the row detail, not the table.
  - Flag pills: none grey, watch amber, suspended red.
- `design:PUB-05 sponsor facts` in `<workspace>\design\RealX App.dc.html` (screen key `prop`, Sponsor tab, lines 265-270; mock data at lines 745-748).
  - The facts are "Trusts on RealX 3", "Trusts managed 14", "Assets managed A$212m" and "Licence Own AFSL".
  - "Trusts on RealX" and "Licence" are A33's live facts of record. "Trusts managed" and "Assets managed" are sponsor-declared facts that this node publishes in the `sponsor_profile` block with `source: sponsor_declared`, after the claims check.
  - The design's "3" for Harbour & Co counts the hidden Marine Parade listing. The public figure is 2 (Kent Street and Bay Road), as A54 records under HG-REVIEW; no response of this module or of the published block may return 3 for Harbour & Co. ADM-29's staff-only `trusts` column keeps 3 / 3 / 2.
- PUB-01 audited-numbers strip and PUB-08 audited totals (design analysis §5 and §6.2 item 12: a KpiGrid in the design's style with mono 26px values and 1px `#E3DED4` gaps) are rendered by A54 and A55 from A47's `totals[]`, which A47 composes from `latestPublishedMetrics`. This node fixes the keys, the figure shape and the empty (`null`) result; the captions are the page nodes' copy.
- The graph's `design_refs` for A45 lists only `design-ext:ADM-26`; `design-ext:ADM-29` and `design:PUB-05` above are also part of this node's frozen spec (the graph entry must add them).
- Tokens: cream `#F5F2EC`, ink `#17191C`, green `#1F5E4B`, brick `#9A4524`, alert red `#A3312A`, amber tint `#F6ECD6`. Newsreader for display, Geist for text, and Geist Mono for ids, hashes, dates and every figure.

## Requirements covered

- **Product-map screens:** none owned. ADM-26 and ADM-29 belong to A76, ADM-01 to A66, the partner data-room views to A105, PUB-01 and PUB-05 to A54, and PUB-08 to A55.
  - "Done" means every column, pill, count, fact, field and blocked-action reason those screens show is an endpoint field here, checked by a contract test the screen nodes re-use.
- **Journeys:** none owned. This node supplies:
  - J10 step 5: the MLRO's and CCO's calendar view of A42's SMR item on ADM-26. Deciding and filing stay A42's routes.
  - J16 step 5: the outside-TMD report item to the issuer, with its 10-business-day clock.
  - J20 step 1: the compliance clocks tile and the prudential read model.
  - J21 step 7: the final report filing opened by A48. Retention is A48's.
  - J01 step 1: the published "audited capital raised" on PUB-01, through A47's `totals[]`.
  - "Done" means those steps can call the interfaces and endpoints above and get the states and reason codes their owning nodes assert.
- **Gates:**
  - **G05 Target market fit** — done when `recordOutsideTmdDealing` creates the filing addressed to the issuer, opens the 10-business-day clock through A13, emits `ddo.outside_tmd` once, and the calendar shows the item and the derived issuer row with their owners, under the P2 guard.
  - **G14 AML monitoring and reports** — done when A42's SMR, TTR and IFTI items, regulatory orders (`aml_regulatory_order`) and reportable breaches appear on the calendar for MLRO and CCO only, as read-only rows with A42's clock ids and due dates. This module has no AML filing kind, no submit path, no alert and no queue item for them, and every other principal, partner, token, export, collection, summary, alert list, queue and audit read shows no trace of them.
  - **G16 Tax details** — done when A41's AMMA, AIIR, CRS/FATCA and withholding remittance rows reach the calendar through A41's outbox as read-only `source: tax` rows with A41's owners, states and evidence, exactly one row per TaxFiling id under replayed events, and this module holds no second tax lodgement.
  - **G17 Marketing claims approved** — done when no public or investor-facing figure or sponsor-declared fact can leave this module (through `latestPublishedMetrics` or the `PublishedSponsorProfileProvider`) unless it is published under a `M19.metricPublication.publish` 6E request, every figure is marked audited through the 4E, and every string is free of forbidden labels and passed by A43's `checkClaims`; and "tokenised value" is refused by the renderer and the grep.
  - **G21 Licence perimeter** — done when every A21 licence, CAR appointment and condition date appears in the calendar as a derived row with its source, TP-LIC sees its own licensee rows and no others, and none of these rows raises a second alert here.
  - **G13 Maker-checker** (standing) — done when waiver, seed edits, grants, extensions, definition changes, mark-audited requests and publications pass through A14 with the policy codes named above, and the effector refuses a mismatched approval.
- **Modules:** **M19 Reporting and filings** — done when the D17 behaviours of the requirements catalogue for reporting are implemented and tested:
  - the calendar at 7/30/90 days with owners and evidence;
  - the filing workspace;
  - the prudential dashboard with NTA where applicable;
  - the regulator data room with scoped, time-boxed access, CCO → SECO;
  - investor and board reports;
  - governed metric definitions, with `capital_raised` versioned CFO → CCO;
  - public KPIs and track record, IRCM → CFO + CCO 6E, with claims reviewed;
  - the overdue filing Sev2;
  - the outside-TMD Sev2.
  - The same holds for the non-functional requirements N06 (clocks through A13 in business days), N12 (server-side filtering, pagination) and N16 (metric governance).
  - N01 retention for wound-down trusts is A48's and A10's. This node only references it.
  - The ADM-29 read under **M24** is covered for its performance table, composed from A33's snapshots and watchlist. M02 licences are A21's, and M27 fee schedules are A33's (see Owned paths).
- **State machines:** none of SM-00..SM-13. Three lifecycles are local tables whose transition tables are exhaustively tested:
  - obligations (`open`, `in_progress`, `completed`, `waived`, `not_yet_applicable`);
  - filings (`draft`, `ready`, `submitted`, `acknowledged`, `rejected`, `withdrawn`);
  - grants (`pending`, `active`, `expired`, `revoked`).
  - "Done" means every allowed transition works and every other returns 409.
- **Founder requirement D04 and gate G22:**
  - every delegable action here is token-reachable as maker under its A03 scope;
  - the person-only acts are refused to tokens with their codes;
  - partner agents read the data room only within a grant or engagement;
  - sensitive agent actions reach A101.
  - Done when the agent-parity suite ([[A100 - Agent-parity suite- every UI action has an API and MCP equivalent under the same gates]]) finds an API and MCP equivalent for every action here and the person-only refusals hold.

## Gauntlet loop for this node

- **Specify.**
  - The frozen spec is:
    - this note;
    - the D17 section of the requirements catalogue (ADM-26, ADM-29, the maker-checker rows, the alerts, the key rules), N06 and N16;
    - A13's `openClock`, `businessDaysAfter`, `emit`, `registerAlertTypes`, `registerSlaPolicies` and `registerQueueSource` contracts;
    - A14's precheck-hook and effector contracts;
    - A21's licence read endpoints;
    - A41's `tax.filing.calendar` outbox event and TaxFiling fields;
    - A42's public reads of `aml_filing`, `aml_regulatory_order` and `aml_reportable_breach`, its ACL and its tipping-off comparer (`tests/aml/tipping-off-diff.ts`);
    - A43's `comms.checkClaims` and its G17 precheck registration on `M19.metricPublication.publish`;
    - A98's `partnerScope`, `GrantSource` type, `registerGrantSource`, `onGrantChanged` and `partner.read` rule;
    - A33's `registerPublishedSponsorProfileProvider`, `sponsorSnapshotSeries`, `sponsor_watchlist` read, tenant record read and section 7 public contract;
    - the reads of A16, A18, A20, A34, A35, A36, A37, A39, A40 and A44 listed under Scope;
    - A47's `totals[]` schema (`{metric, value_cents | value, as_of, attestation: {kind, ref, attested_at}}`), as the consumer contract of `latestPublishedMetrics`;
    - the design regions above.
  - Three items are settled before the node QA writes the hidden tests:
    - (a) The follow-up loop `A03.1-A45` has passed: A03 holds the M19 and M24 rows of the permissions table (ids, kinds, roles, policy codes, delegable flags, PM or proposal), the baseline and resolution rows, the A14 policy seed rows and the updated counts. A14 takes action codes only from A03, so the build cannot start until they exist.
    - (b) A98 confirms where `partner.read` is written, so each data-room read writes one event.
    - (c) A101 confirms that AGTU audit events with the `sensitive` class are consumed.
  - Any change after the freeze to what "audited" means, to who may see an AML row, or to the maximum grant window is a spec defect, not a fix.
- **Oracle (the node QA).** Hidden tests for the five invariants in the oracle rule go to `<workspace>\gauntlet\hidden\A45\` before Build.
- **Build (Grok 4.7 xhigh, split by the planner).** Four parallel sub-agents on disjoint files, then one integration pass:
  1. **The pure parts**, each with unit tests:
     - `obligations.ts` (the seeded catalogue and recurrence rules) and the three transition tables;
     - `metrics/formula.ts` (the declarative formula evaluator over integer series);
     - `metrics/renderer.ts` with `forbidden_labels.ts`;
     - `collections.ts` with the scope filter and the minimiser;
     - `aml_projection.ts` as a pure function of viewer and A42 rows (filings, regulatory orders and breaches).
  2. **Persistence and API:**
     - the Drizzle schema and migration;
     - repositories with row-level security for sponsor and partner reads;
     - permission checks against A03's M19 cells;
     - routes and zod schemas;
     - OpenAPI generation and an SDK compile check.
  3. **Wiring:**
     - the source adapters over A16, A18, A20, A21, A33, A34, A35, A36, A37, A39, A40, A42 and A44;
     - the A41 intake consumer on `tax.filing.calendar`;
     - the A13 registrations, clocks and queue source;
     - precheck hooks and effectors for the approved action pairs, with A43's claims check in the publication and definition prechecks;
     - the `GrantSource` implementation registered with A98's `registerGrantSource` and the `PublishedSponsorProfileProvider` registered with A33, both at module start-up;
     - the three jobs (`calendar_sweep` with the overdue emitter, `dataroom_sweep`, `metrics_compute`) on the injectable clock;
     - `recordOutsideTmdDealing`, `openWindDownReport` and the registration interfaces for later nodes.
  4. **Exports:**
     - the lodgement-pack and report renderers into A08 storage, with hashes, watermarks and masking;
     - the prudential read model with the NTA block;
     - the sponsor-performance read model, `latestPublishedMetrics` and the `sponsor_profile` block.
  - **Integration:**
    - The seed:
      - the catalogue;
      - one published metric set matching the design, marked audited through a real 4E with a seeded `audit_signoff` letter from a seeded audit firm, with `trusts_on_realx` at platform scope counting publicly visible trusts only and one `sponsor_profile` block for each of the three sponsors;
      - one active data-room grant for a seeded TP-AUD.
    - An end-to-end test. It:
      - shows an A42 fixture SMR item on the MLRO's calendar and nowhere else;
      - opens an outside-TMD filing under `RUN_MODE=local`, ages it past its clock under the injectable clock, and sees the Sev2 arrive once;
      - submits it as LEGL with the issuer's acknowledgement reference;
      - grants an auditor a 30-day data room through a real A14 4E, reads a masked register extract under it, revokes it and proves the next read is 404.
- **Challenge (node QA, Opus 5.5 high).** The node QA runs the hidden oracle and the module tests, then its own vectors:
  - exhaustive transition tests generated from the three tables;
  - every filing kind submitted by every staff role, every partner role and a token for each;
  - the AML projection probed as every principal through the calendar list, get by id, summary, prudential, exports, every data-room collection, `GET /v1/alerts` and A13's queue items, ADM-31 audit reads through A10, and OpenAPI examples, with an A42 fixture SMR, a regulatory order and a reportable breach planted, and A42's tipping-off comparer run over every response;
  - A42's named re-run: a late SMR, TTR, IFTI and regulatory order each appear on the MLRO's and CCO's calendar from A42's `aml_regulatory_order` and `aml_reportable_breach` rows, dated by the statutory or order deadline, and nowhere else;
  - the A41 intake on `tax.filing.calendar` with replayed, reordered and duplicated events;
  - A98's named real-grant re-runs, against this module's effector, `reporting.dataroom_sweep` and SECO's revoke instead of `FixtureGrantSource`: grant-scoped reads, grant expiry (a read at 00:00:01 after `ends_at` is 404 before the sweep runs, for the person and for its agent), revocation, TP-OBS agent parity (a TP-OBS with no live grant has no data-room scope), and the token cap (the partner token's `expires_at` equals the earliest `ends_at` of its live grants);
  - the port registrations: a boot outside the `test` profile has this module's `GrantSource` registered with A98 and no `FixtureGrantSource`, and A33's provider returns this module's latest block;
  - the 10-business-day clocks across Easter 2027 and both DST changes, checked against A13's vectors;
  - recurrence materialisation with the clock advanced day by day across three year-ends, and with a seed edit mid-cycle;
  - grant and engagement scope with every combination of tenant, vehicle, collection and date range against planted out-of-scope items;
  - the 90-day maximum and the one-tick revoke;
  - the renderer against a fixture template that plants each forbidden label in three casings and inside a longer sentence;
  - the publication precheck against management values, retired versions, a suspended tenant, a banned claims phrase in a figure label, and a sponsor-declared fact or `about` text containing a banned phrase (refused at the precheck with the report stored);
  - the mark-audited 4E against a changed value and an invalid letter;
  - `metrics_compute` run twice on the same inputs;
  - the permission matrix for every staff role, SP-OWN, SP-FIN, TP-AUD, TP-OBS, TP-LIC, TP-RE, an investor and a token for each, on every endpoint;
  - the `Date.now()` grep;
  - OpenAPI and SDK presence of every endpoint;
  - a check that the fixture provider is absent from the production registry and that the API refuses to boot with it outside `RUN_MODE=local`;
  - a design-fidelity read of the seed against ADM-29's columns, ADM-26's column sets and the prudential NTA row;
  - a contract test of `latestPublishedMetrics` against A47's `totals[]` schema: every figure maps field for field to `{metric, value_cents | value, as_of, attestation: {kind: 'auditor', ref, attested_at}}`, and `null` maps to an empty `totals[]`. A47's gate re-runs it against A47's real reader;
  - a PII grep: no investor name, TFN, unmasked identifier, case narrative or AML reference in any calendar row, filing list, export, data-room file, audit payload or watermark.
- **Gate.** Pass unlocks the ten dependents. There are at most three fix loops, and each re-runs the full hidden set and the failing challenges.
- **Upstream corrections requested** (this note cannot edit other specs; each owner records the change in its own spec or in its follow-up loop):
  - **A03** (loop `A03.1-A45`): the M19 and M24 rows, delegable flags, baseline and resolution rows and counts above.
  - **A51** rows 141 and 143 ("Grant regulator or auditor access", "Change a metric definition"): replace "no A03 row" with the A03 action ids `M19.dataRoomGrant.request` and `M19.metricDefinition.change` added by `A03.1-A45`.
  - **A47**: read published figures through `latestPublishedMetrics(scope)`, not "the data behind `GET /v1/metrics/public`", which does not exist.
  - **A43** (lines 344, 583 and 693): register the G17 precheck on `M19.metricPublication.publish`, not `metrics.publish`, and drop the reference to `GET /v1/metrics/public`.
  - **A33**: "Trusts on RealX" counts publicly visible trusts only (2 for Harbour & Co in the base seed, as A54 requires); `distributions_on_time_pct` and `proof_of_property_score` hold basis points (or are renamed with a `_bp` suffix); the `metrics.publish` references read `M19.metricPublication.publish`.
  - **A54**: the same unit convention for the PUB-05 performance lines.
  - **A98** line 46: "A45 cites the auditor's signed letter; TP-AUD is read-only" replaces "attesting a metric as audited is A45's person-only act for TP-AUD".
  - **A76**: no change needed; this note now lists SUPER (proposal) under `M19.calendar.read`, matching A76 line 129.
  - **A66** line 191 and **A65** line 87: the scope is `M19.boardPack.export` (board pack) or `M19.report.export` (sponsor report), not `reports:export`.
  - **A105** line 152: the item view writes exactly one `partner.read` and the viewer's file fetch none, as this note now states; no change beyond removing "assumption".
  - **Graph** (`app-graph.json`): add `design-ext:ADM-29` and `design:PUB-05` to A45's `design_refs`.
- **Escalation trigger.** Reopen the spec and ask the user in any of these cases:
  - a third failure on the same assumption (for example, whether an overdue obligation is Sev2 at `due_at` or at close of business);
  - any fix that would need a change outside the owned path (A13's clock semantics, A14's hook contract, A42's ACL, A98's session scopes, A03's matrix);
  - any finding that the catalogue's maker-checker rows contradict this note.

## Node-specific challenges (node QA)

- **The number that was never audited.**
  - The seeded sources compute `capital_raised` at A$18,420,000.00 as `management`. IRCM opens `M19.metricPublication.publish` with it, and it is refused at precheck naming `capital_raised`.
  - CFO drafts `markAudited` citing a seeded `audit_signoff` letter, and CCO approves. Then:
    - the 6E passes with CFO and CCO;
    - `latestPublishedMetrics('platform')` returns it with `basis: 'audited'` and an `attestation` naming the letter;
    - the auditor's TP-AUD principal is notified and sees the value under `audited_by=me`.
  - The job then recomputes a higher management value. `latestPublishedMetrics` and A33's provider must still return the published figure and version.
  - A board pack rendered the same minute must show the management figure labelled "management figure, unaudited", and rendering it as an `investor_report` must be refused.
  - A CFO approving their own mark-audited request, an agent drafting one, a letter of the wrong kind, or a value that changed after the request must each be refused with its code.
  - An implementation that serves the live value, drops the label, or has a TP-AUD write route fails.
- **The word that must not appear.** Each of these must be refused with `MET_FORBIDDEN_LABEL` and audited as `metric.render_refused`:
  - a template author writes "Total value tokenised: A$212m";
  - a metric definition names "TVL";
  - a publication label contains "tokenised value" inside a longer sentence.
  - A publication of approved, audited numbers whose `period_label` contains a phrase from A43's seeded block list (for example "guaranteed income") must be refused by the claims check, with the report stored.
  - The per-vehicle "Token valuation A$246.10" on PUB-04 and the vehicle-scope valuation in a board pack render normally.
  - A renderer that matches only exact labels, a publication that skips `checkClaims`, or a grep that misses `packages/ui` fails.
- **The data room that outlives its grant.** Take a TP-AUD grant for tenant Harbour & Co, with collections `register_extracts` and `recon_runs`, from 1 to 30 October 2026.
  - On 15 October the grantee's reads are refused with 404:
    - Northline's extracts;
    - a filing;
    - an A42 SMR id (404 even though the item exists);
    - Harbour & Co's extract dated 2 September, when `date_from` is 1 October.
  - On 31 October at 00:00:30 Sydney, the same extract that worked at 23:59 is 404 and the grant reads `expired`.
  - SECO revokes another grant at 10:00:00, and a request at 10:00:01 is 404.
  - A TP-AUD with an engagement listing `recon_runs` reads it without a grant, and cannot read `register_extracts`.
  - The grantee's agent token reads within a live grant. Every page and every item view writes exactly one `partner.read` with `on_behalf_of`, the viewer's file fetch writes none, each download writes one `dataroom.item_downloaded` and no `partner.read`, and A101 receives the sensitive read.
  - A CCO tries to grant a data room to an investor, to a sponsor user, and for 120 days: three refusals with their codes.
  - Any 200 in the wrong place, a revoke that waits for the sweep, or a second audit event per page fails.
- **The filing lodged by the wrong hand, and the report that is not ours.**
  - A42's fixture opens an SMR item with a 3-business-day clock at 10:00 on Thursday 25 March 2027. Then:
    - it appears on the MLRO's and CCO's calendar, with A42's clock id and no buttons;
    - CEO, LEGL, CFO, SUPER, TP-AUD and TP-OBS find no trace of it in the calendar, summary, alerts, queues, exports, collections or audit reads;
    - every write route on its id returns 404;
    - `POST /v1/reporting/filings` with `kind: smr` returns 422 `FIL_KIND_OWNED_ELSEWHERE`;
    - when the MLRO misses it, only A42's Sev1 exists, and this module has raised nothing.
  - An `outside_tmd` filing opened at 10:00 on Thursday 25 March 2027 has its 10-business-day clock land on Monday 12 April 2027 (Good Friday and Easter Monday skipped). Then:
    - CFO, MLRO, a TP-OBS and LEGL's own agent token each try to submit it, and all are refused (403 or `FIL_PERSON_ONLY`);
    - it passes its clock unsubmitted: exactly one `reporting.filing_overdue` Sev2 exists, and it is resolved by system when LEGL submits late with the issuer's acknowledgement reference;
    - TP-RE of that vehicle sees the filing and the derived issuer row.
  - A second sweep that repeats the Sev2, a submit that succeeds for the wrong role, or any trace of the SMR outside MLRO and CCO fails.
- **The seed edit that re-arms nothing.**
  - LEGL edits the AUSTRAC annual compliance report's due date from 31 March to 30 April 2027 through `M19.obligation.editSeed` 4E. Then:
    - the old clock must be cancelled and a new one opened;
    - the row's `version` must be incremented;
    - the queue-source item must be re-dated.
  - The sweep is run ten times under a clock advanced from 1 March to 15 May. It must produce one `queue.sla_breach` through A13 on the right day, one `reporting.filing_overdue` on 1 May, and the 2028 occurrence materialised once with the edited month.
  - A PATCH of `due_at` on the same row is 422 `OBL_FIELD_LOCKED`. A PATCH on an A41 AMMA row is 422 `OBL_READ_ONLY_SOURCE`.
  - A change applied without the 4E, a duplicated occurrence, or a stale clock fails.
- **Design and copy fidelity.** With the seed:
  - `GET /v1/reporting/sponsor-performance` (staff) returns the three design sponsors with `trusts` 3, 3 and 2 and exactly the field set of the 8 ADM-29 columns plus `late_distributions_6m`, `incidents` and `complaints`, with `flag` taken from A33's watchlist and tenant state;
  - `latestPublishedMetrics('platform')` passes the contract test against A47's `totals[]` schema with `basis: 'audited'`, an auditor attestation and the seeded as-of date;
  - Harbour & Co's published `sponsor_profile` block carries the declared facts with `source: sponsor_declared`, the performance lines in basis points with their units, and `trusts[]` without Marine Parade; no public output of this module (the block, `latestPublishedMetrics` or any seed fixture used by A33's public route) returns 3 as Harbour & Co's "Trusts on RealX";
  - `GET /v1/reporting/calendar?horizon=90` returns the A41 AMMA row with owner TAXO and `source: tax`, and the derived licence rows with `source: perimeter`;
  - `GET /v1/reporting/calendar/summary` has exactly the fields `due_7`, `due_30`, `due_90`, `overdue`, `unassigned`;
  - the prudential NTA block reads `not_applicable_car`.
  - A seed that marks a figure audited without the 4E, a calendar that duplicates A21's or A41's rows as its own, or a percentage stored outside basis points fails.

## Pass criteria

- **Transitions.** All three local transition tables are tested exhaustively: every allowed obligation, filing and grant transition passes, and every generated forbidden pair returns 409. 100 percent of the tables are covered, and the forbidden-pair count in the test equals the count derived from the tables.
- **AML projection (oracle).**
  - A generated test runs over every staff role, every partner role, an investor, a token for each, and every read surface: calendar list, get by id, summary, prudential, exports, every data-room collection, A13 alert lists and queue items, A10 audit reads through ADM-31's route, and OpenAPI examples.
  - It finds zero traces of an A42 item outside MLRO and CCO, three runs in a row.
  - The summary never counts AML rows for anyone.
  - This module wrote zero alerts, queue items and audit events that name an AML item.
- **Clocks.**
  - The node QA's vector set has at least 40 A45-owned obligations and filings with due dates across 2026-2028, covering both DST changes each year, Easter 2027, seed edits mid-cycle and the calendar-exhausted refusal from A13.
  - On 100 percent of cases it yields:
    - exactly one `reporting.filing_overdue` per overdue A45-owned row, at the right instant;
    - none for `aml`, `perimeter`, `tax` or `derived_*` rows;
    - one A13 queue item per eligible row within 7 days;
    - no duplicates under repeated or rewound sweeps;
    - re-arming on an edited date.
- **Sources.**
  - Every source named under Scope is served by this node's adapter over the owning module's `index.ts`. Outside `RUN_MODE=local` tests, no metric, collection or obligation is served by the fixture provider.
  - The API refuses to boot when the fixture provider is in the production registry.
  - A source with no provider yet returns `null` with `source_pending: true`.
  - The A41 intake yields exactly one row per TaxFiling id under 1,000 replayed and reordered events.
- **Data room.**
  - A property test over at least 2,000 random grants and engagements (tenants, vehicles, collections, windows) and 20,000 random item requests agrees with a pure reference scope filter on every allow and every 404.
  - Expiry and revoke take effect within one clock tick in every planted case.
  - Every allowed page read and every item view writes exactly one `partner.read`, the viewer's file fetch writes none, and every download writes exactly one `dataroom.item_downloaded` and no `partner.read`, each with the grant or engagement id and, under a token, `on_behalf_of`.
  - Access through `listActiveGrants` and the data-room routes ends at `ends_at` under the injected clock, before the sweep runs.
  - Every served file is watermarked, and person fields are masked.
- **Metrics.**
  - The formula evaluator agrees with an independent reference on 5,000 random integer series. There are no floats anywhere: `npm run verify` greps the module for `parseFloat` and `toFixed` outside the renderer's display layer.
  - Every ratio metric has `unit: bps`.
  - `metrics_compute` run twice on the same inputs produces identical `inputs_sha256`, `value_sha256` and values.
  - Every publication precheck refusal case in this note is refused by name: management value, retired version, `public_allowed: false`, forbidden label, claims-check failure and suspended tenant.
  - Every mark-audited refusal case is refused by name.
  - `latestPublishedMetrics` and the `sponsor_profile` block never return a figure whose stored basis is not `audited`, every figure carries an auditor attestation, and the contract test against A47's `totals[]` schema passes field for field.
- **Forbidden labels.** The renderer refuses every seeded label in upper, lower and mixed case, singular and plural, and inside longer strings, in templates, definitions and publication strings. The verify grep finds none of them in the owned path or `packages/ui` outside `forbidden_labels.ts`.
- **Maker-checker.**
  - Every state-changing endpoint that this note names as approved opens an A14 request with its A03 action id and policy code:
    - 4E: `obligation.waive`, `obligation.editSeed`, `dataRoomGrant.request`, `dataRoomGrant.extend`, `metricDefinition.change`, `metricValue.markAudited`;
    - 6E: `metricPublication.publish`.
  - The effector refuses a mismatched or unapproved approval id in every planted case.
  - The single-actor acts (complete, assign, submit, revoke, export) write their audit event with the actor and never accept an approval id for another object.
- **Permissions and agents.**
  - Permission tests are generated from A03's M19 and M24 cells and cover every staff role, SP-OWN, SP-FIN, TP-AUD, TP-OBS, TP-LIC, TP-RE, an investor and a token for each, on every endpoint.
  - A token with a delegable scope behaves exactly as its human as maker, is refused as checker by A14, and is refused with `FIL_PERSON_ONLY`, `MET_ATTEST_PERSON_ONLY` and `DR_SELF_GRANT` where this note says so.
  - Every token action is audited with `on_behalf_of`, and every sensitive agent action listed above carries the `sensitive` class that A101 consumes.
- **Audit.** After the full suite:
  - A10's chain verification passes;
  - the number of `obligation.*`, `filing.*`, `dataroom.*`, `partner.read` (data-room routes), `report.*` and `metric.*` audit events equals the number of writes, transitions, reads, renders and job runs the suite performed;
  - A10's PII detector flags nothing.
- **OpenAPI, SDK and grep.** Every endpoint is present in the generated OpenAPI document, `packages/sdk` compiles with typed clients for them, and the MCP tool catalogue generated from the document lists them. `npm run verify` finds no `Date.now()`, argument-less `new Date()` or local business-day arithmetic in the owned path.
- **Seed and coverage.**
  - The seed reproduces exactly:
    - the ADM-29 rows;
    - the published metric set, marked audited through a 4E, with the platform `trusts_on_realx` counting publicly visible trusts only;
    - the three `sponsor_profile` blocks, with no hidden listing in any count or `trusts[]`;
    - the ADM-26 column sets;
    - the NTA `not_applicable_car` state.
  - The seeded calendar count and every owner role are asserted by the seed test.
  - Line coverage of the module is at least 90 percent.

## Human gates touched

- **None crossed.**
  - No regulator, issuer, licensee, auditor or portal is called.
  - Lodgement references are typed by a human, and in the build by the test endpoint.
  - The seeded audit firm, its `audit_signoff` letter and the data-room grant are sample records from A98 and A35.
- **HG-LEGAL** later confirms every `set_with_counsel` row:
  - the obligation catalogue, its due dates and recipients;
  - that in P1 RealX notifies its licensee rather than ASIC, and the notice period;
  - that the DDO report goes to the issuer, and the issuer's ASIC report;
  - the clock lengths;
  - the meaning of "audited" for `capital_raised`, and the sponsor co-investment treatment;
  - whether NTA applies and at what requirement;
  - which acts are person-only;
  - the forbidden-label list;
  - the wording of every public metric definition.
  - Each answer is a data change followed by a re-run of this node's calendar, projection, renderer and fidelity tests, not a gate on this node.
- **HG-VENDOR and HG-LEGAL** together decide whether a live auditor keeps the signed-letter model chosen here, or attests on the portal (which would need an A03 cell and an A98 exception).
  - Until a real auditor signs, no figure is `audited`, and the public site keeps its unaudited empty state.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
