---
title: "A90 - P2: registered scheme, sub-funds, PDS and TMD, target market (G05), retail cooling-off"
created: 2026-09-24
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am9]
node: A90
milestone: AM9
lane: "P2"
oracle: yes
human_gate: none
---

# A90 - P2: registered scheme, sub-funds, PDS and TMD, target market (G05), retail cooling-off

**The retail track of Phase 2, switched on only while `offers.retail` is bound to the hired RE's licence record. It covers: the umbrella `Scheme` run by the hired responsible entity (RE), with a `SubFund` vehicle per property, both established two-person with the RE as a checker; a PDS (plus any current SPDS) and a TMD attached to every retail listing, with the TMD's distribution conditions stored as data; the AU-MIS-retail rule pack that needs a target-market result; the ONB-07 target-market questionnaire and the G05 evaluation per listing; the `TMD_FIT` claim on the investor's on-chain identity; the INV-R role and when retail listings are visible; the `reflection_period` branch of SM-05, driven by a cooling-off parameter per listing (default none, counsel to confirm); and the outside-TMD dealing register, which feeds the 10-business-day report item. Every action is callable under a scoped token with the same gates as the screen. The one exception is the target-market answers, which are personal until counsel says otherwise.**

**Build status:** not started. **Milestone:** AM9 (P2 features (from about 8 April 2027)). **Lane:** P2. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 18-26 agent-hours (build, challenge and verify loops), plus 3-5 agent-hours for the upstream loops recorded in spec points (f) to (j). The scope is seven tables, about 25 routes, four A14 action codes, five jobs, two fact providers, the pack draft, the claim lifecycle, one consumer, six web components with seven ONB-07 states and two 10,000-case properties. If the orchestrator prefers smaller nodes, the clean split is A90a (scheme, sub-funds, TMD records, retail terms, go-live) and A90b (G05, answers, ONB-07, cooling-off UI, outside-TMD, INV-R), with A90b depending on A90a.

**Depends on:** [[A36 - Offerings- offer terms, listing review, raise monitor (SM-04)]], [[A32 - Classification- wholesale certificates and target-market hooks (SM-02)]], [[A35 - Documents, disclosure library, e-sign and acknowledgements (G12)]], [[A56 - Onboarding flow ONB-01 to ONB-10 with entity branch]], [[A57 - Buy flow- order, review and confirm, order status (INV-01 to INV-03)]], [[A71 - Console- listing review, live raise monitor, orders and allocation]], [[A45 - Reporting, regulatory calendar, filings and metric governance]], [[A98 - Partner principals- trustee-RE, valuer, property manager, auditor, regulator observer and licensee users]], [[A60 - Investor and sponsor API tokens and agents screens (ACC-05, SPN-16)]], [[A54 - Public site- home, investments, property page, sponsor profile, fees]], [[A76 - Console- regulatory calendar and data room, sponsor performance, integrations and jobs]], [[A43 - Communications, notifications, tasks and marketing claims (G17)]], [[A46 - Job scheduler, integrations health and webhooks]], [[A48 - Halts, kill switches and wind-down]], [[A101 - Agent guard- anomaly detection, sensitive-action notices and tiered kill switches]], [[A104 - MCP server generated from the OpenAPI document]], [[A100 - Agent-parity suite- every UI action has an API and MCP equivalent under the same gates]]
(The last eleven are interfaces this spec calls: A45's `recordOutsideTmdDealing` and the `outside_tmd` clock, A98's TP-RE principal, A60's confirmation screen, A54's listing visibility, A76's ADM-26 rows, A43's templates and claims lint, A46's job registry and webhooks, A48's wind-down path, A101's sensitive-action notices, A104's catalogue and A100's parity manifest. All sit in AM1 to AM8, before AM9, and none has A90 as an ancestor. The same edges must be added to `final-graph.json`.)
**Unlocks:** [[A94 - P2- end-to-end journeys and launch update]], [[A95 - P3- Auto Invest and PayTo mandates]]

**Oracle rule:** The node QA writes hidden oracle tests from this spec to `<workspace>\gauntlet\hidden\A90\` before the build starts ([[App Gauntlet]] §2). The builder never reads that folder. The cases are published into the repo only after the gate passes. This is an oracle node (`oracle: yes`): orders, escrow, refunds and mints stay with A37 and A38, but this node decides who may buy a retail unit, whether an order waits out a cooling-off period before any money moves, when a `fund_at = confirm` revoke refunds in full, and what an agent may do without its person. The oracle covers:
- the G05 fit vectors over the question bank and criteria, and the result precedence of spec point (f);
- the retail-terms validator table, including `last_order_at` under both funding modes;
- the reflection-branch property (retail and days > 0 and flag on, and nothing else);
- the scheme and sub-fund transition tables and their segregation of duties;
- the outside-TMD clock vectors over A13's business-day calendar;
- whose declaration an answer set is, for individual, joint (each signing rule), SMSF, company and trust accounts (§5);
- the `TMD_FIT` claim for buyers and for the sponsor's retained stake (§4);
- the agent-parity probes, including the direct signed-payload probe of spec point (g);
- the tests that answers never leak.

This note is not legal advice. The following are counsel questions (HG-LEGAL, CQ-01 and CQ-06), each held as a parameter row with a default and a `set_with_counsel` marker:
- whether retail orders carry a cooling-off period, of what length, when it starts, and whether it runs before funding or after issue;
- whether the target-market answers must be given personally;
- what counts as a "significant" dealing outside the TMD;
- whether a wholesale client buying into a retail sub-fund must answer the questions;
- how long answers stay valid;
- whether the RE must be a named checker on every retail go-live;
- whether a closing-early offer may end a cooling-off period (A37's table currently moves `reflection_period -> awaiting_funds` at an early close).

Counsel's answers change rows, not code, with one stated exception. If counsel says cooling-off must run after issue (a right to hand back issued units), `fund_at` cannot express it: SM-05 has no path from `settled` back through burn, register removal and refund. That answer is an escalation trigger and a new SM-05 requirement for A37, A28 and A18 (see Escalation trigger). Nothing here is switched on in P1. The retail flag, the RE's authorisation and the scheme's registration with ASIC are human acts (see Human gates touched).

## Build brief

This node has two halves:
- **API.** Module `scheme` in the Fastify monolith (`services/api/src/modules/scheme/`), with its worker jobs registered from the same folder through A09's job registry.
- **Investor web.** The retail feature components in `apps/web/src/features/retail/` and the target-market route in `apps/web/app/(investor)/target-market/`.

It edits nothing that another node owns. The stepper, the buy screens, the listing review and the filings screen already carry the slots and generic renderers this node fills:
- A56's `stepsFor` ONB-07 slot;
- A57's `reflection` status and banner slot, and its regime-neutral disclosure rows;
- A71's generic blocker list and "AU-retail" pack pill;
- A76's ADM-26 filing rows over A45.

General rules for the whole node:
- Every rule is enforced in the API, never only on a screen.
- Every route carries `x-realx-scope` (a cell in M04, M07 or M19) and is in the OpenAPI document from the first commit, so `packages/sdk`, the MCP catalogue (A104) and the parity suite (A100) see it.
- The only source of time is A09's injectable clock. `Date.now()` and argument-less `new Date()` are grepped by `npm run verify`.
- Every P2 route and job first asks A21 (the owner of the `offers.retail` flag and its licence binding) whether `offers.retail` is on and A08 (the owner of the `PHASE` setting) whether `PHASE >= P2`. Otherwise it answers `PHASE_NOT_ENABLED` (API) or renders the P1 preview (web). The `local|test` environment override is the only way to exercise it before the retail launch, and the start-up log records it.

**1. What this node reuses and what it adds (no duplication of A21, A32, A33, A35, A36, A37, A45)**

This node reuses:
- **Tables from A04.** `scheme` (P2) and `vehicle` (`vehicle_type` `sub_fund`, `scheme_id`) are A04's tables. Vehicle rows are created only through A33's `createVehicle` interface, which already refuses a `sub_fund` while `PHASE=P1`.
- **Entity and licence rows.** The RE is an A33 `entity` of type `responsible_entity`. Its authorisation is an A21 `licence` row of kind `re_authorisation`, holding the activities `operate_registered_scheme`, `issue_pds` and `deal_retail_units`. The flag binding is A21's: `offers.retail` may bind only to a licence holding all three, a toggle is `4E` SUPER to CCO, and the flag reverts automatically when the binding lapses.
- **From A32.** The `tmd_questionnaire` and `tmd_response` tables, the pure `evaluateG05`, the `requiresHuman` route `POST /v1/tmd/responses` and the outbox event `tmd.outside_market_dealing`. This node fills in content, screens and the report. It does not write a second evaluator: A32's `evaluateG05` stays the only one, and its input and result contract is extended by loop A32.2 (spec point (f)), because the contract A32 ships reads a per-offering response and has no expiry or review-overdue result.
- **From A12.** The `428 CONFIRMATION_REQUIRED` record (`agent_confirmation`) and `POST /v1/agent-access/confirmations/{id}/confirm`, extended with a person-only flag by loop A12.2 (spec point (g)).
- **From A98.** The TP-RE partner principal (the hired RE's users) that consents to sub-funds, checks `tmd.publish` and retail go-live, and may stop distribution.
- **From A35.** The document types `pds`, `spds`, `tmd` and `constitution`, the disclosure-set resolver, the SPDS supplement rule, the acknowledgement evaluator, the `cooling_off_statement` rows with the `{{cooling_off_days}}` template, and the standing-consent rules.
- **From A36.** The `offering` row with `regime = au_mis_retail`, `tmd_id`, the material-change fields (which already list "IM or PDS version, TMD version") and the go-live and pause actions.
- **From A37.** The SM-05 transitions `eligibility_check -> reflection_period -> awaiting_funds | revoked`, the `revoked_in_reflection` refund reason, the order parameters `cooling_off_days` and `fund_at`, the `orders.agent_standing_consent` switch (default: wholesale only) and the cancel route.
- **From A45.** `recordOutsideTmdDealing`, which creates the `outside_tmd` filing in `draft`, opens A13's `outside_tmd` 10-business-day clock and raises `ddo.outside_tmd` (Sev2, CCO).
- **From A31.** `issueClaim` and `revokeClaim` for topic `TMD_FIT` (5).
- **From A19.** The `AU-R-2027.1` pack stub and the "Rule pack change" approval.

This node adds:
- **Tables of its own**, registered through A04's `registerModuleSchema`, and nothing else:
  - `scheme_state_log`;
  - `sub_fund_request`;
  - `offering_retail_terms`;
  - `tmd_record` (the structured TMD metadata per TMD document version);
  - `tmd_question_bank`;
  - `tmd_fit_result`;
  - `outside_tmd_dealing`.
- **Parameter rows** in `packages/config`, each marked `set_with_counsel`:
  - `retail.cooling_off_default_days` = 0;
  - `retail.cooling_off_count_basis` = `calendar_days`;
  - `retail.fund_at_default` = `after_reflection`;
  - `tmd.answers_valid_days` = 365;
  - `tmd.reanswer_cooldown_hours` = 24;
  - `tmd.flip_watch_days` = 30;
  - `ddo.significance` (a small rule set: any single dealing at or above `ddo.single_dealing_cents` = A$10,000.00, or at least `ddo.count_per_listing_30d` = 5 outside dealings in one listing in 30 days, or outside dealings above `ddo.share_of_raise_bps` = 500 of a listing's raise);
  - `tmd.review_warn_days` = 30.

**2. Scheme and sub-funds (the P2 legal wrapper as records)**

The `scheme` row (A04 columns, filled here):
- `id` (`SCH-<6 base32>`), `name`, `arsn` (null until registered), `re_entity_id` (the A33 entity) and `re_licence_id` (the A21 licence);
- `constitution_doc_id` and `compliance_plan_doc_id` (A35 documents of type `constitution` and `compliance_plan`; the second is proposed as a data addition to A35's vocabulary, see spec point (e));
- `state`, `registered_approval_id`, `activated_at`, standard audit and RLS columns (platform tenant; readable by every sponsor tenant through A04's `is_public` reference policy on non-PII columns).

The scheme lifecycle is a small local table in A03's engine format:
- `draft -> registered` (ARSN and both documents recorded);
- `registered -> active`;
- `active -> suspended` (automatic);
- `suspended -> active` (automatic, when the binding is restored and a CCO confirms);
- `active -> winding_up` (A48's wind-down path).

Any other change is refused with `SCHEME_ILLEGAL_TRANSITION`.

The routes:
- `POST /v1/scheme/schemes` (LEGL or CCO, or their agents as makers) creates a `draft`.
- `POST /v1/scheme/schemes/{id}/register` creates an A14 request `scheme.register` under `4E` (LEGL maker, CCO checker who is not the maker). It carries the ARSN, the ASIC registration evidence document and the two document ids. The effector refuses unless both documents are `published` and the RE's licence is `active`.
- `POST /v1/scheme/schemes/{id}/activate` is a CCO act with a fresh assertion (`requiresHuman`). It refuses unless all of the following hold:
  - the scheme is `registered`;
  - A21 reports `offers.retail` on;
  - the flag's `licence_binding` is this scheme's `re_licence_id` (`SCHEME_FLAG_BINDING_MISMATCH` names both licence ids when the flag is bound to anything else, for example the sample AFSL);
  - `PHASE >= P2`.
- The worker job `scheme.binding_watch` (every minute and on A21's `flag.reverted` and `licence.state_changed` events) moves an `active` scheme to `suspended` in the same transaction that processes the event, whenever the flag is off or bound elsewhere, or the RE licence is not `active`.
- `POST /v1/scheme/schemes/{id}/reactivate` is the CCO's confirmation of the return to `active` once the binding is restored (fresh assertion, `requiresHuman`, the same four checks as activate).

A sub-fund is one `vehicle` per property:
- `vehicle_type = sub_fund`, `scheme_id`, `trustee_entity_id` = the scheme's RE (the RE is the responsible entity of every sub-fund; `SUBFUND_TRUSTEE_MUST_BE_RE` otherwise), its own client-money trust account reference (filled by the treasury node) and its own unit register (A18 keys registers by vehicle; the register stays the legal record).
- It is established through `sub_fund_request` (`SFR-`): `scheme_id`, `tenant_id` (the sponsor), `property_id` (A34; a property already bound to another vehicle is `SUBFUND_PROPERTY_TAKEN`), `proposed_name`, `class_code`, `state` (`requested -> re_consented -> established | declined | withdrawn`).

The sub-fund routes:
- `POST /v1/scheme/schemes/{id}/sub-funds` (SP-OPS or SP-OWN of the tenant, or CAPM; agents as makers) creates the request. Committing a property to the scheme is a sponsor-side decision, so a request follows A33's `tenant_approval_rule` exactly as A36's sponsor requests do, under the new action code `scheme.subfund.request` (default row: `required_roles [SP-OWN]`, `required_count 1`; a data row for A33). The request records `owner_approved_by`; a request an SP-OWN makes in person counts as the tenant approval when the rule's count is 1. Until the tenant approval is complete, RE consent is refused with `SUBFUND_OWNER_APPROVAL_REQUIRED`. The agent counts as its human for segregation of duties, so an SP-OWN cannot approve a request made by their own agent; a sole-owner tenant uses the same fallback A33's rule already defines for its other action codes. A CAPM request made on the sponsor's written instruction still needs the tenant approval.
- `POST /v1/scheme/sub-fund-requests/{id}/re-consent` is the TP-RE partner principal from A98 (AM3). It needs a fresh assertion, is `requiresHuman`, and takes the reason code `RE_CONSENT` or `RE_DECLINE`.
- Establishment is then an A14 request `scheme.subfund.establish` under `6E`: the requester is the maker; the TP-RE who consented is one checker and the platform CCO is the other; the maker's own agents and the consenting TP-RE's agents are excluded as A14 does.
- The effector calls A33's `createVehicle` and A34's property binding in one transaction and writes `subfund.established`.

A sponsor tenant may use the scheme only when its `tenant_guardrail.eligibility_packs` includes an `AU-R` pack (A33 shows `AU-R` packs only once the flag is on).

**3. PDS, SPDS, TMD and retail terms per listing (M04)**

`tmd_record` (`TMDR-`) is the structured side of a published `tmd` document version (A35 keeps the PDF and its sha256). It holds:
- `document_version_id`, `vehicle_id`;
- `target_market` (plain-English description);
- `criteria` (the same JSON rule shape A32's evaluator reads, over question-bank codes);
- `outside_market_action` (`stop` or `flag`: the TMD's distribution condition for an investor outside the target market);
- `review_period_months`, `review_due_at`, `review_triggers[]` (coded list);
- `approved_approval_id`.

A TMD record and the listing questionnaire that maps to it are published together as one A14 request, `tmd.publish`, under `6E`:
- maker: SP-OPS, LEGL or CAPM;
- checkers: the scheme's TP-RE (the issuer, who makes the TMD) and the platform CCO (the distributor);
- the effector also asks A35 to publish the PDF version, so the document and its data can never disagree (`TMD_RECORD_DOC_MISMATCH` when the hashes differ).

TMD reviews are tracked through A13:
- A13 clock `tmd.review_due` at `review_due_at`, with a Sev3 warning to CCO and TP-RE `tmd.review_warn_days` before.
- A breach sets the fact `tmd_review_overdue`, and A19's G05 then fails new orders with `G05_TMD_REVIEW_OVERDUE`.
- `POST /v1/scheme/sub-funds/{id}/stop-distribution` is the issuer's stop: the scheme's TP-RE acts (fresh assertion, `requiresHuman`; reason code from `review_triggers`), and the route opens A36's `offer.pause` on every live offering of the sub-fund as `SGL+R` with cause `tmd_review`, with the platform CCO as the ratifier within A36's 2 hours. For this the TP-RE needs A03's `offerings:pause` cell, scoped to offerings of its own scheme (a data diff; A36's cell today names CCO only). A CCO who wants the same stop uses A36's own pause route with cause `tmd_review`. A pause is a protective act (`delegable: false` in A03, as A36 says), so an agent can neither draft nor call it: every token is refused `AGNT_NOT_ALLOWED` (reason code `G22_SCOPE_NON_DELEGABLE`) on the `requiresHuman` route (A12 §5 order of agent refusals).

`offering_retail_terms` (keyed by `offering_id`, one row per offering version) carries:
- `scheme_id`, `sub_fund_vehicle_id`;
- `pds_document_id` and the current SPDS set at submission;
- `tmd_record_id`, `questionnaire_id` and `questionnaire_version`;
- `cooling_off_days` (integer >= 0), `cooling_off_count_basis`, `fund_at` (`after_reflection` default or `confirm`);
- `outside_market_action` (copied from the TMD record, never typed);
- `last_order_at` (derived, rounded down to a Sydney business-day boundary on A13's calendar):
  - with `fund_at = after_reflection`: `closes_at` minus the cooling-off period (counted on `cooling_off_count_basis`) minus A37's `orders.funding_window_hours`, because the money moves only after reflection;
  - with `fund_at = confirm`: `closes_at` minus the cooling-off period, because the money moved at confirm and only the reflection must finish before close;
  - with `cooling_off_days = 0`: null (G11's ordinary close rule applies).

The terms are set through `PUT /v1/scheme/offerings/{id}/retail-terms` (A36's `offerings:draft` cell: SP-OPS or SP-OWN of the tenant; their agents as makers) on a `draft` offering only (`OFR_TERMS_LOCKED` otherwise, as A36 does), and are read through `GET /v1/scheme/offerings/{id}/retail-terms` (A36's `offerings:read` cell).

The two order parameters A37 reads through A36 (`cooling_off_days` and `fund_at`) are the values of this row. Spec point (a) settles how A36 exposes them.

Retail-terms rules, a pure validator `retail-terms.ts` registered with A36's validate step for `regime = au_mis_retail`. It returns blockers that A71's ADM-07 already renders generically. Each rule has its own code:
- `RETAIL_VEHICLE_NOT_SUBFUND`: the vehicle is not a `sub_fund` of an `active` scheme.
- `RETAIL_PDS_REQUIRED`: there is no `published` PDS for the vehicle at the current clock.
- `RETAIL_COOLING_OFF_MISMATCH`: the PDS version's metadata `cooling_off_days` differs from the offering's. A35 stores this on the version's `meta` as a data field; the PDS is the source and the offering copies it.
- `RETAIL_TMD_REQUIRED`: there is no published TMD record for the vehicle, or `offering.tmd_id` is not that record's document.
- `RETAIL_QUESTIONNAIRE_NOT_APPROVED`: the questionnaire does not reference exactly the published TMD version.
- `RETAIL_CLOSE_TOO_SOON`: `closes_at - opens_at` is shorter than `cooling_off_days` plus the funding window plus one business day.
- `RETAIL_PACK_NOT_EFFECTIVE`: no `AU-R` rule pack is `effective`.
- G21 fails through A21's evaluator, naming the RE and the licence id, when the RE licence is not `active`.

These rules run on every save, at submit and again in A36's go-live precheck.

Material change (spec point (i)):
- `cooling_off_days`, `fund_at`, `outside_market_action`, `questionnaire_version` and the pinned SPDS set are added to A36's `material_fields.ts` as a data diff (the PDS and TMD versions are already there).
- A36's `offer.material_change` is a manual `4E` request (CAPM maker, CCO checker), so publishing never opens it by itself. Instead, `tmd.publish` for a vehicle that has an offering in `live`, `soft_cap_reached` or `reconfirming` has a precheck: the request must carry `material_change_request_id`, an open A36 `offer.material_change` request for that offering whose diff names the new TMD version. Without it the request is refused at creation with `TMD_MATERIAL_CHANGE_REQUIRED`. The TMD is published first (its effector); A36's material-change effector then refuses until the TMD version is published (the retail-terms validator's `RETAIL_TMD_REQUIRED`). Until the material change applies, the offering keeps its old TMD and questionnaire version, and G05 evaluates against them.
- An SPDS works the same way. The offering's disclosure set uses the SPDS set pinned on the offering version (the `RegimeTermsProvider` of spec point (a) supplies it), so a newly published SPDS reaches a live offering only through a linked `offer.material_change`. A published SPDS that no offering version pins raises Sev3 `retail.spds_unpinned` to CAPM. If A35's resolver reads "current" rather than "pinned", that is loop A35.1.
- When the material change applies, A36 opens its 5-business-day re-confirmation window, A35 makes the old acknowledgements stale (`DISCLOSURE_VERSION_STALE`), and `tmd.fit_refresh` re-evaluates every open order against the new criteria. An order that no longer fits fails at A37's allocation re-evaluation and is refunded through the ordinary path.
- A36's re-confirmation set today is `funded`, `awaiting_funds` and `submitted`. This node adds `reflection_period` as a data diff to A36. The outcome at the deadline for an order still in `reflection_period` that has not re-confirmed: the SVCA moves it `reflection_period -> revoked` with cause `reconfirm_lapsed` (a guard row for A37's existing transition, which today names only the investor's revoke). Under `after_reflection` no journal exists and nothing is refunded; under `confirm` it goes `revoked -> refund_pending` and refunds `total_cents` under `revoked_in_reflection`. Re-confirming does not restart the cooling-off timer by default (a CQ-06 row, `retail.reconfirm_restarts_cooling_off = false`).

Retail go-live:
- A36's `listing.go_live` is `4E+Q` with the platform CCO. For `au_mis_retail` this node contributes the policy row `listing.go_live.retail`, proposed as `6E+Q`: CAPM maker, checkers the platform CCO and the scheme's TP-RE. This is the architecture brief's "Offer approval (CCO + LEGL, plus X-RE on the retail track)", recorded as a seed addition in A03's policy data.
- If the user prefers LEGL as the second RealX checker in addition, the row changes and the node records the choice in `A90.qa.md`.

Disclosure set and statements (G12):
- The regime set for `au_mis_retail` is proposed as data to A35's resolver: `pds` (plus the current `spds` as one requirement), `tmd` and `fee_schedule`.
- The two retail `cooling_off_statement` rows are proposed as data to A35 (`set_with_counsel`):
  - `au_mis_retail` with days: "You can cancel this order within {{cooling_off_days}} {{cooling_off_basis}} of confirming it, as set out in the PDS. No money leaves your account until the cooling-off period ends." (the second sentence only when `fund_at = after_reflection`). `{{cooling_off_basis}}` renders `retail.cooling_off_count_basis` as "days" for `calendar_days` or "business days" for `business_days`; the token is a data addition to A35's statement template vocabulary;
  - `au_mis_retail_none`: "No cooling-off period applies to this offer, as set out in the PDS."
- A57 renders whichever `statements[]` the quote returns and has no regime branch. Agent acknowledgement for retail stays off: `orders.agent_standing_consent` keeps its A37 default `wholesale_only`, so a retail agent order always returns `428` with `documents_required` until the person acknowledges.

**4. The AU-MIS-retail rule pack, G05 and the `TMD_FIT` claim**

The rule pack:
- This node drafts `AU-R-2027.1` from A19's stub with the parameters `tmd.required = true`, `tmd.answers_valid_days`, `claim_topics = [KYC_VERIFIED, SANCTIONS_CLEAR, RESIDENCY_FIRB, TMD_FIT]` (A24 and A31 already agree) and `cooling_off.max_days = 14` (a ceiling for sponsor input; the value is still the PDS's).
- The pack reaches `effective` only through A19's "Rule pack change" approval, with a CCO checker. A pack that is `draft` leaves every retail offering blocked with `RETAIL_PACK_NOT_EFFECTIVE`.

G05:
- A19's registry maps G05 to A32's `evaluateG05`, with the input and result contract of spec point (f). This node supplies the facts through A19's `FactProvider` (`scheme/gates/g05-facts.ts`):
  - the account's latest standard answer set (answers, bank version, expiry) or null;
  - the offering's current questionnaire version and criteria;
  - `outside_market_action`;
  - `tmd_review_overdue`;
  - the principal (A19's `ActionContext.principal`), so the evaluator knows whether the order is agent-placed.
- G05 applies to every buyer of an `au_mis_retail` offering, wholesale clients included, until counsel says otherwise (HG-LEGAL). A wholesale investor who chose "Answer later" at ONB-07 therefore gets `G05_NO_TMD_ANSWERS` at INV-01 with the fix-it.
- The last-order rule is a G11 check, not a G05 one (spec point (h), loop A19.2): G11 reads the optional fact `retail_last_order_at` from `scheme/gates/g11-retail-facts.ts` and fails `G11_RETAIL_ORDER_WINDOW_CLOSED`, naming the date, when the clock is past it. So an order whose cooling-off could not finish before the scheduled close is refused at the quote and at placement. A cooling-off period is never cut short by the scheduled close. (An early close is A37's rule and a counsel question; see above.)
- G05 runs at quote, at placement and at A37's allocation re-evaluation. A19's table 3 already lists G05 in the `order_allocation` set and marks it `not_applicable` only for the wholesale regime, so spec point (d) is a check, not a change.
- Results, in precedence order (the first that applies wins; spec point (f)):
  1. `not_applicable` for every `s708_wholesale` offering (A32's G04 is already `not_applicable` for retail);
  2. `fail G05_PHASE_OFF` for `au_mis_retail` while `offers.retail` is off or `PHASE < P2` (A19's and A32's P1 result, kept);
  3. `fail G05_TMD_REVIEW_OVERDUE` (a listing fact, checked before any investor fact);
  4. `fail G05_NO_TMD_ANSWERS` when there is no answer set, or the set lacks a code the listing's questionnaire uses (fix-it: answer the questions, deep link `/target-market?offering=<code>`);
  5. `fail G05_ANSWERS_EXPIRED` when `expires_at <= now`;
  6. `pass` when the criteria put the answers `in_market`;
  7. `fail G05_OUTSIDE_MARKET` when outside and the action is `stop`;
  8. `warn G05_OUTSIDE_MARKET_FLAGGED` when outside and the action is `flag`, with `person_confirmation_required: {reason: 'outside_target_market'}` when the principal is `AGNT` and `tmd.agent_outside_market = person_required` (spec point (g)).
- A19's stub code `TMD_ANSWERS_MISSING` and A19's own G05 stub are retired in favour of this contract, as data changes in A19's fix-it table and evaluator registry (spec point (d)).

Agent-placed outside-market orders (spec point (g)):
- A `warn G05_OUTSIDE_MARKET_FLAGGED` on an order placed under a token never passes on the agent's word, and the rule lives in the API, not the screen. The chain is: A19's `GateVerdict` carries `person_confirmation_required`; A37's placement returns `428 CONFIRMATION_REQUIRED` with A12's `confirmation_id` and the sub-reason `outside_target_market` whenever any verdict in the set carries that field and the principal is `AGNT`, and creates no order; A12's `agent_confirmation` row carries `person_only = true`.
- A valid `RealX-Mandate-Signature` within caps does not satisfy a `person_only` confirmation: the placement is refused with `CONFIRMATION_PERSON_ONLY` (403, gate G22), and nothing is written.
- The person confirms with a passkey on ACC-05 (A60 renders it) after reading why the offer is outside their target market. A37 then places the order under the stored payload.
- A32 emits `tmd.outside_market_dealing` only after the placement commits (loop A32.3). A warn during a quote, a refused placement or an unconfirmed agent placement emits nothing.
- This is a proposed default recorded under CQ-01. If counsel allows it, the change is one parameter, `tmd.agent_outside_market`, from `person_required` to `mandate`.

The `TMD_FIT` claim:
- When a person's standard answer set is on file and unexpired, this node asks A31 to issue `TMD_FIT` (topic 5) through the V4 identity agent.
- The claim data is the sha256 of the answer set id and version, and `valid_until` equals the answer set's expiry.
- On expiry (worker job `tmd.answers_expiry`, daily 04:15 Sydney, idempotent), on withdrawal by the person, or on account offboarding, it asks A31 to revoke the claim. Holdings stay.
- Per-listing fit is enforced off-chain by G05. The claim is the coarse on-chain mirror that a retail token's identity registry requires, so a mint or a P2 resale purchase to an investor without current answers fails on-chain as well as off-chain.
- The claim belongs to the account's on-chain identity (the investor record), not to a person, so a joint, SMSF, company or trust account holds one claim for its answer set (§5).
- The sponsor's retained stake. A36 mints the retained stake (default 1,000 bps) to the sponsor's investor record, which never answers the questions and is not an order, so G05 does not apply to it. So that the on-chain mint does not fail on the required `TMD_FIT` topic, the `tmd.publish` effector asks A31 to issue `TMD_FIT` with claim data `{basis: 'issuer_retained', vehicle_id, tmd_record_id}` to the sponsor's retained-stake investor record for that sub-fund. It is valid while the sub-fund is not winding up, revoked by A48's wind-down path, and never usable for an order (G05 reads answer sets, never claims, so the claim cannot stand in for answers).

**5. Target-market questions and ONB-07 (M07)**

The question bank:
- `tmd_question_bank` is versioned data seeded from the product map. There are four codes:
  - `OBJECTIVE` ("What are you investing for?": Regular income, Growth over time, Both);
  - `HORIZON` ("How long can you hold?": Less than 2 years, 2 to 5 years, 5 years or more);
  - `LIQUIDITY_NEED` ("Could you need this money at short notice?": Yes, No);
  - `SHARE_OF_PORTFOLIO` ("How much of your total investments will this be?": Less than 10%, 10 to 25%, More than 25%).
- This bank is the one canonical option set. It follows the product map's ONB-07 components and mock ("How long can you hold?" answered "5 years or more"). A50's ONB-07 extension draws a different set (Timeframe under 3 years / 3-7 / over 7; "Need to sell quickly"; "Share of investable assets"), so A50 must redraw its option strings from this bank before A90's Specify. Until then the screenshot comparison masks the option text and compares layout only.
- The option strings are data. Any change goes to the IRCM-to-CCO copy approval and A43's claims lint.
- A listing questionnaire may use only bank codes. A listing-specific extra question needs a new bank version first (`TMD_CRITERIA_INVALID` otherwise). Criteria are validated at `tmd.publish` against the bank: every referenced code and option exists, and the rule is total.

The standard answer set (spec point (b)):
- The person answers once. The answers are stored as an A32 `tmd_response` against the platform standard questionnaire: a `tmd_questionnaire` with `offering_id = null`, scope `platform`, version = bank version.
- The submission is A32's `POST /v1/tmd/responses` (`requiresHuman`). This node gives it the purpose `tmd_declaration` for A11's fresh passkey assertion over the canonical answers hash, and the `declaration_id` is stored on the response.
- Whose declaration it is. The answer set belongs to the account (the investor record), and so do INV-R and the `TMD_FIT` claim. Who declares:
  - an individual account: the person, with one `declaration_id`;
  - a joint account: its signing rule, as A12 does for token creation. With "either to sign", one holder declares for the account. With "both to sign", the first holder's submission is held as `pending_cosign` and the set takes effect only when the second holder asserts over the same answers hash within `tmd.cosign_window_hours` = 72 (a `set_with_counsel` row); otherwise it lapses and no set exists. One `declaration_id` per signer;
  - an SMSF, company or trust: an authorised KYB signatory of the entity (A30's verified signatory list); anyone else is refused with `TMD_SIGNATORY_REQUIRED`. The set records the signatory's `declaration_id`, and where the entity's signing rule needs two signatories, both declare as for a "both to sign" joint account.
  The answers describe the account's objectives, not one person's.
- For every live retail listing, `tmd_fit_result` rows are then computed by A32's evaluator: `investor_id`, `offering_id`, `questionnaire_version`, `response_id`, `result` (`in_market` or `outside_market`), `action`, `evaluated_at`. They are recomputed when a listing's questionnaire version changes or a new retail listing goes live (worker job `tmd.fit_refresh`, idempotent, on `offering.live` and `tmd.published`).
- Re-answering is allowed after `tmd.reanswer_cooldown_hours` (`TMD_REANSWER_COOLDOWN` names the time). Every set is kept.
- A new set that turns an `outside_market` result into `in_market` for a listing where the person had a refused or flagged quote within `tmd.flip_watch_days` raises `tmd.answers.flip` (Sev3, CCO) and stores the fact on the next order's gate inputs. It does not block (counsel question).

Answers are private:
- Answers are personal information and are never returned to a sponsor, a partner or support.
- Sponsors see only counts per listing (`in_market`, `outside_flagged`, `outside_stopped`, `not_answered`) from `GET /v1/scheme/offerings/{id}/tmd-summary`. SP-OWN, SP-OPS and SP-IR of the tenant, TP-RE of the scheme, and CCO and LEGL may read it.
- The listing criteria are never sent to the investor, the SDK examples or an error message. A result says in or outside and which TMD to read, never which answer would change it.

The web route `apps/web/app/(investor)/target-market/` (product-map route `/verify/target-market`; spec point (c) settles `/target-market` as canonical, and A56's own slot already uses it). With `?flow=onboarding` the page renders as an onboarding step without any A56 edit:
- it registers the step through A56's public `registerOnboardingStep({ id: 'ONB-07', route: '/target-market', after: 'ONB-06', when: phaseFlags['offers.retail'] })`, which A56 ships for exactly this;
- it computes the stepper list with A56's exported pure `stepsFor`, so the eyebrow "Step n of N" matches A56's stepper;
- it composes the sticky stepper column and the card from A05's `@realx/ui` primitives with the ONB-06 tokens below. It does not import A56's `StepFrame` or `StepCard`, which A56 does not export; if A56 later exports them, switching is a refactor inside this node's folder.

States:
- **Flag off.** No form and no API call beyond `GET /v1/perimeter/public`. It shows the brick preview banner whose text is A21's `retail_launch_label`, rendered verbatim (seeded "Wholesale investors · retail from 2027"). This node writes no banner text of its own. A50's extension draws "Retail investors from about 8 Apr 2027"; A50 must draw the A21 value, and until it does the screenshot comparison masks the banner text and compares its layout.
- **First visit.** The four questions are OptionCard groups (single select each). "Continue" is inert until every group has a choice. Pressing it opens the passkey sheet for the declaration, then posts.
- **Result.** A list of live retail listings, each with a StatusPill:
  - `In target market` (green);
  - `Outside · you can still apply` (amber, action `flag`);
  - `Outside target market` (red, action `stop`);
  - `Update your answers` (grey, expired);
  - each with a link to the listing's TMD (A35 public document).
- **Arrived from INV-01** (`?offering=PAD01`). The green banner "PAD01: you're in the target market." (product-map mock) with "Continue" back to the buy step, or the outside explanation with the TMD link.
- **Empty.** "No retail offers are open right now. We'll check your answers against each new offer."
- **Cooldown.** The time the person may answer again.
- **Signed out.** A56's sign-in redirect.

Onboarding:
- When the flag is on, A56's `stepsFor` slot inserts ONB-07 after ONB-06.
- ONB-06 gains, through A32's option vocabulary (a data diff with `phase: P2`, rendered by A56 unchanged), a fourth option "Retail investor · Invest in retail offers under a PDS. We'll ask a few questions next." Choosing it records no classification.
- A wholesale investor may skip ONB-07 ("Answer later") and is asked at INV-01 through G05's fix-it.
- This node registers a `target_market` `StatusContributor` with A30's composer for ONB-10. It gives `done` with the set's expiry, `action_needed` when expired, and `not_applicable` in P1 or for a wholesale investor who skipped. A retail investor's `wholesale` row becomes `not_applicable`.

**6. INV-R role and retail listing visibility**

The INV-R role:
- A whitelisted investor who submits a standard answer set while the flag is on is granted INV-R through A11's role-assignment interface as a system assignment (`SVCA` actor, audited `retail.role.granted`).
- The role's cells come from A03's matrix; this node adds none.
- The role is kept while any retail holding exists, and removed when the account closes or the flag is permanently withdrawn (`retail.role.revoked`).
- A12's rule applies unchanged: an existing token gains no INV-R cell until its owner widens it with a fresh assertion.

Visibility:
- While the flag is off, `au_mis_retail` offerings never appear in A36's public or investor listing contracts, except in the sponsor's own console and the operator review. A36's `OFR_REGIME_NOT_ENABLED` already holds submit and go-live.
- Once the flag is on and the offering is live, it is public like any listing. The card and property page eligibility come from A19's dry-run gates, so A54 shows "Outside your target market" (stop) or the flagged note without a regime branch.
- A54's existing eligibility filter defaults to "eligible for me" for a signed-in retail investor. That default filter is this node's reading of the "hidden" of J16's failure path. The listing is still reachable by URL and by "show all", because the PDS and TMD are public documents. Whether a `stop` listing must be hidden outright rather than filtered and flagged stays under HG-LEGAL ("whether to hide or flag, per TMD"); a stricter answer is a data row on A54's filter, not a change here.
- A visitor sees retail listings with "Check if it's right for you", linking to sign-up.

**7. Retail cooling-off (the `reflection_period` branch of SM-05)**

What A37 already does: it takes `eligibility_check -> reflection_period` only when `regime = au_mis_retail`, `cooling_off_days > 0` and the P2 flag is on; wholesale never, whatever the parameters.

What this node adds:
- `coolingOffFor(offeringId) -> {days, count_basis, fund_at, ends_at(placed_at)}` from `index.ts`. `ends_at` counts in the Sydney zone on A13's calendar, calendar days by default.
- A13 clock policy row `retail.cooling_off`, which A37's timer uses so the timer and the screen agree to the second.
- `cooling_off_ends_at` on the order read model, through A37's read contract.

Revoking:
- Revoke is A37's `POST /v1/orders/{id}/cancel` from `reflection_period`. It needs no reason, and the person or their agent may call it, because money only goes back to the person.
- With `fund_at = after_reflection` no journal exists for the order, and a revoked order ends `revoked` with no refund.
- With `fund_at = confirm`, `revoked -> refund_pending` refunds the full `total_cents` under `revoked_in_reflection` within A37's 5 business days.
- A revoke and the timer's end on two processes resolve in the database: exactly one transition commits and the other is refused `ORD_INVALID_TRANSITION`.

Web (`features/retail/`):
- `CoolingOffBanner` fills A57's INV-03 `reflection` slot (status label "Cooling-off (P2)", amber), with the days and hours left, the end time in AEST or AEDT, and "No money has moved" or "Your money is in escrow" by `fund_at`. When the order was agent-placed it also shows A57's "placed by your agent" marker.
- The `Revoke order` danger-outline button opens `RevokeOrderDialog` with the refund sentence for the funding mode. The call uses the SDK and the idempotency key.
- When the timer ends the banner disappears and A57's awaiting-funds state takes over.

Notifications: this node proposes A43 templates for `order.reflection.started` and `order.reflection.ending` (24 hours before), with a revoke deep link.

**8. Outside-TMD dealings (M19, J16 step 5)**

When an order's placement commits with `warn G05_OUTSIDE_MARKET_FLAGGED` (for an agent-placed order, only after the person has confirmed; spec point (g)):
- A32 emits `tmd.outside_market_dealing` after the placement transaction commits (loop A32.3), never on a quote or a refused or unconfirmed placement.
- This node's consumer, idempotent on `order_id`, writes one `outside_tmd_dealing` row (`OTD-`): `order_id`, `offering_id`, `tmd_record_id`, `response_id`, `amount_cents`, masked investor reference, `reason_codes`, `significance_facts` (the `ddo.significance` rule results, including the rolling 30-day count and share of raise for the listing), `assessment` (`pending`, `significant`, `not_significant`), `assessed_by`, `reason_code`, `filing_id`.
- In the same transaction it calls A45's `recordOutsideTmdDealing`, which opens the filing in `draft` and the 10-business-day clock from the moment of awareness (the placement time), and links `flagged_report_id` on A32's response through A32's interface.
- A second event for the same order is a no-op.

Assessment:
- The dealing needs a human assessment, in two routes:
  - `POST /v1/scheme/outside-tmd-dealings/{id}/assessment-drafts` (CCO or LEGL, or their agents as makers; delegable) stores a proposed `significant` or `not_significant` with a reason code and a note, as the maker's draft. It changes no filing. A later draft replaces the earlier one and each is audited.
  - `POST /v1/scheme/outside-tmd-dealings/{id}/assess` (CCO only, fresh assertion with purpose `ddo_assessment`, `requiresHuman`) records the final decision, taking `significant` or `not_significant` with a mandatory reason code (`SINGLE_LARGE`, `PATTERN_IN_LISTING`, `SHARE_OF_RAISE`, `ISOLATED_SMALL`, `ORDER_REVOKED_BEFORE_ISSUE`, ...) and optionally the `draft_id` it adopts. An AGNT principal is refused with `AGNT_NOT_ALLOWED` before scope evaluation.
- `significant` moves A45's filing to `ready`. The report goes to the issuer (the scheme's RE, the recipient under DDO for a distributor) and is submitted on ADM-26 by LEGL or CCO through A45's `submit`.
- `not_significant` withdraws the filing through A45 with the reason, which meets the clock.
- Nothing closes a dealing automatically. A breached clock escalates through A13 (Sev2 to CCO).
- An order later revoked in reflection keeps its dealing row. The assessor sees `order_state = revoked` among the facts.

The RE's own reporting obligation to ASIC is recorded on A45's calendar as the RE's (owner TP-RE, `phase: P2`), not RealX's.

**9. Agents and tokens (D04, G22)**

Every route goes through A12's `agentAuth` and G22 unchanged and carries `x-realx-scope`. Parity with the screens is as follows:
- **Investor's agent.** It may read the question bank, its person's fit results, answer-set expiry and cooling-off state. It may place retail orders under its mandate, under all of G01-G22 exactly as the person, with the two person-only confirmations above: disclosure acknowledgement, and outside-market orders under `flag` (a `person_only` confirmation that its mandate signing key cannot satisfy; spec point (g)). It may revoke an order in reflection, which sends a sensitive-action notice to the person through A101. It cannot submit answers: `POST /v1/tmd/responses` is `requiresHuman`, A12 refuses the scope at issue with `SCOPE_PENDING_COUNSEL`, and A11 refuses the call with `AGNT_NOT_ALLOWED` before scope evaluation.
- **Sponsor's agent.** It may draft sub-fund requests, retail terms, TMD records and questionnaires, and read the fit summary, as maker only. The sponsor-side second person (A36's submission chain) and the A14 checkers stay human.
- **TP-RE's agent.** It may read and comment. Consent, `tmd.publish` checking and stop-distribution are `requiresHuman`, and stop-distribution is not even draftable (a protective act).
- **Staff agents** (LEGL, CAPM, CCO). They may draft `scheme.register`, `tmd.publish`, the pack draft and outside-TMD assessment drafts as maker. They can never check, approve, activate, reactivate, assess finally, pause or submit a filing: A14 refuses an agent checker with `APR_CHECKER_MUST_BE_HUMAN`, and A11 refuses an AGNT principal on every `requiresHuman` route with `AGNT_NOT_ALLOWED` before scope evaluation.

**Route table (frozen at Specify).** Each route names its A03 scope id (the value of `x-realx-scope`), the roles holding that cell, whether it is `requiresHuman`, what an agent gets, and the A14 policy code where one applies. Scope ids marked *new* are A03 matrix cells added as a data diff (below); A09 rejects any other unknown scope id at registration.

| Route | Scope id | Roles | requiresHuman | Agent | Policy |
|---|---|---|---|---|---|
| `POST /v1/scheme/schemes` | `scheme:draft` *new* (M04) | LEGL, CCO | no | maker | none |
| `POST /v1/scheme/schemes/{id}/register` | `scheme:register_request` *new* (M04) | LEGL | no | maker | `scheme.register` `4E`, CCO checker |
| `POST /v1/scheme/schemes/{id}/activate`, `.../reactivate` | `scheme:activate` *new* (M04) | CCO | yes | refused `AGNT_NOT_ALLOWED` | none (fresh assertion) |
| `GET /v1/scheme/schemes`, `.../{id}`, `GET /v1/scheme/sub-funds/{id}` | `scheme:read` *new* (M04) | every sponsor role (non-PII columns), CAPM, CCO, LEGL, TP-RE, TP-OBS, TP-AUD | no | read | none |
| `POST /v1/scheme/schemes/{id}/sub-funds` | `scheme:subfund_request` *new* (M04) | SP-OWN, SP-OPS (own tenant), CAPM | no | maker | tenant rule `scheme.subfund.request`, then `scheme.subfund.establish` `6E` |
| `GET /v1/scheme/sub-fund-requests` | `scheme:subfund_read` *new* (M04) | SP-OWN, SP-OPS (own tenant), TP-RE (own scheme), CAPM, CCO, LEGL | no | read | none |
| `POST /v1/scheme/sub-fund-requests/{id}/re-consent` | `scheme:re_consent` *new* (M04) | TP-RE (own scheme) | yes | refused `AGNT_NOT_ALLOWED` | none (fresh assertion) |
| `PUT /v1/scheme/offerings/{id}/retail-terms` | `offerings:draft` (A36) | SP-OPS, SP-OWN (own tenant) | no | maker | none |
| `GET /v1/scheme/offerings/{id}/retail-terms` | `offerings:read` (A36) | A36's `offerings:read` holders | no | read | none |
| `POST /v1/scheme/tmd-records` (draft record and questionnaire), `PATCH .../{id}` | `scheme:tmd_draft` *new* (M04) | SP-OPS, SP-OWN (own tenant), LEGL, CAPM | no | maker | none |
| `POST /v1/scheme/tmd-records/{id}/publish-request` | `scheme:tmd_publish_request` *new* (M04) | SP-OPS, LEGL, CAPM | no | maker | `tmd.publish` `6E`, TP-RE and CCO checkers |
| `GET /v1/scheme/tmd/{recordId}`, `GET /v1/tmd/questions` | public | anyone | no | read | none |
| `GET /v1/scheme/offerings/{id}/tmd-summary` | `scheme:tmd_summary` *new* (M04) | SP-OWN, SP-OPS, SP-IR (own tenant), TP-RE (own scheme), CCO, LEGL | no | read (counts only) | none |
| `POST /v1/scheme/sub-funds/{id}/stop-distribution` | `offerings:pause` (A36; TP-RE cell *new*, scoped to own scheme) | TP-RE | yes | refused (not delegable) | A36 `offer.pause` `SGL+R`, CCO ratifier |
| `POST /v1/tmd/responses` | A32's cell (M07) | INV-* for own account, per §5 | yes | refused `AGNT_NOT_ALLOWED`; scope `SCOPE_PENDING_COUNSEL` at issue | none (fresh assertion) |
| `GET /v1/tmd/me/results`, `GET /v1/tmd/me/answer-set` | `tmd:read_own` *new* (M07) | INV-* for own account | no | read; the agent's answer-set read returns id, version and `expires_at` with answers omitted | none |
| `GET /v1/scheme/outside-tmd-dealings` | `ddo:read` *new* (M19) | CCO, LEGL, TP-RE (own scheme, masked investor reference) | no | read | none |
| `POST /v1/scheme/outside-tmd-dealings/{id}/assessment-drafts` | `ddo:assess_draft` *new* (M19) | CCO, LEGL | no | maker | none |
| `POST /v1/scheme/outside-tmd-dealings/{id}/assess` | `ddo:assess` *new* (M19) | CCO | yes | refused `AGNT_NOT_ALLOWED` | none (fresh assertion) |

The A14 approve and ratify routes, A37's order and cancel routes, A19's rule-pack routes and A45's filing routes keep their own owners' scopes. The generated route-by-principal matrix in the pass criteria is checked against this table and A03's data, never against the running routes alone.

Every agent action is audited as "agent <token prefix> on behalf of <person>" and appears in the person's activity and on ADM-35. A100's parity suite picks up every operation here. A104's MCP catalogue lists the delegable ones and none of the `requiresHuman` routes.

**10. Read API, events and errors**

Read routes (all under `/v1`, cursor-paginated, idempotency keys on writes):
- `GET /v1/scheme/schemes` and `GET /v1/scheme/schemes/{id}` (with RE name, licence state, flag binding state, sub-fund count).
- `GET /v1/scheme/sub-funds/{id}`.
- `GET /v1/scheme/sub-fund-requests?state=`.
- `GET /v1/scheme/offerings/{id}/retail-terms`.
- `GET /v1/scheme/offerings/{id}/tmd-summary`.
- `GET /v1/scheme/tmd/{recordId}` (public fields only: target market text, action, review date, document link).
- `GET /v1/tmd/questions` (bank; public).
- `GET /v1/tmd/me/results?offering=`.
- `GET /v1/tmd/me/answer-set` (the person's own answers, masked to option labels).
- `GET /v1/scheme/outside-tmd-dealings?state=&offering=`.

Empty states:
- `{items: [], next_cursor: null}` for lists.
- A person with no answers gets `{answer_set: null, next_action: "answer_questions"}`.
- A listing without a questionnaire returns `RETAIL_QUESTIONNAIRE_NOT_APPROVED` on the summary.
- An unknown or other-tenant id is 404, never 403.

Audit events (exactly one hash-chained A10 event per change, each with `actor_id`, `on_behalf_of_id` and `approval_id` where one exists):
- `scheme.created`, `scheme.registered`, `scheme.activated`, `scheme.suspended`, `scheme.reactivated`;
- `subfund.requested`, `subfund.owner_approved`, `subfund.re_consented`, `subfund.re_declined`, `subfund.established`;
- `retail_terms.set`;
- `tmd.drafted`, `tmd.published`, `tmd.review_warned`, `tmd.review_overdue`, `tmd.distribution_stop_requested`;
- `tmd.answers.submitted`, `tmd.answers.cosign_pending`, `tmd.answers.cosign_lapsed`, `tmd.fit.refreshed`, `tmd.answers.flip`, `tmd.claim.issued`, `tmd.claim.revoked`;
- `retail.role.granted`, `retail.role.revoked`, `retail.spds_unpinned`;
- `outside_tmd.recorded`, `outside_tmd.assessment_drafted`, `outside_tmd.assessed`.

The outbox carries the same names for A43, A46 and A101, with ids and codes only. Answers and names are never included.

Error codes are stable strings in the OpenAPI document:
- `PHASE_NOT_ENABLED`;
- `SCHEME_ILLEGAL_TRANSITION`, `SCHEME_FLAG_BINDING_MISMATCH`, `SCHEME_RE_LICENCE_NOT_ACTIVE`, `SCHEME_NOT_ACTIVE`, `SCHEME_DOCS_REQUIRED`;
- `SUBFUND_TRUSTEE_MUST_BE_RE`, `SUBFUND_PROPERTY_TAKEN`, `SUBFUND_RE_CONSENT_REQUIRED`, `SUBFUND_OWNER_APPROVAL_REQUIRED`;
- `RETAIL_VEHICLE_NOT_SUBFUND`, `RETAIL_PDS_REQUIRED`, `RETAIL_COOLING_OFF_MISMATCH`, `RETAIL_TMD_REQUIRED`, `RETAIL_QUESTIONNAIRE_NOT_APPROVED`, `RETAIL_CLOSE_TOO_SOON`, `RETAIL_PACK_NOT_EFFECTIVE`;
- `TMD_RECORD_DOC_MISMATCH`, `TMD_CRITERIA_INVALID`, `TMD_REANSWER_COOLDOWN`, `TMD_MATERIAL_CHANGE_REQUIRED`, `TMD_SIGNATORY_REQUIRED`;
- `OTD_REASON_REQUIRED`, `OTD_ALREADY_ASSESSED`;
- plus A32's `G05_*` (as extended by spec point (f)), A19's `G11_RETAIL_ORDER_WINDOW_CLOSED` (spec point (h)), A21's `G21_*`, A11's `AGNT_NOT_ALLOWED` and `FRESH_ASSERTION_*`, A12's `CONFIRMATION_REQUIRED`, `CONFIRMATION_PERSON_ONLY` (spec point (g)) and `G22_SCOPE_NON_DELEGABLE`, and A14's `APR_*`.

They are returned with 403, 404, 409, 422 or 428, so the SDK and MCP server surface them verbatim.

**Spec points settled at Specify (before the node QA writes the hidden tests)**
- **(a)** A37 reads `cooling_off_days` and `fund_at` "through A36's index.ts", but A36's `offering` row does not list them. Proposed: A36's `index.ts` getter composes them from this node's `offering_retail_terms` through a registered `RegimeTermsProvider` (loop A36.1 if the slot is missing), with defaults 0 and `after_reflection` for every non-retail offering.
- **(b)** The platform standard questionnaire with `offering_id = null` in A32's table (loop A32.1 if the column is not nullable).
- **(c)** The canonical route `/target-market` as described in §5.
- **(d)** G05 is in A19's `order_allocation` set for `au_mis_retail` (A19's table 3 already lists it; this is a check). A19's stub code `TMD_ANSWERS_MISSING` and A19's baseline G05 stub are retired in favour of A32's evaluator under the contract of (f).
- **(e)** `compliance_plan` is added to A35's document-type vocabulary.
- **(f) The G05 contract (loop A32.2, with A19.1).** `evaluateG05` becomes `evaluateG05({regime, flag_on, phase, principal, answer_set: {answers, bank_version, expires_at} | null, listing: {questionnaire_version, criteria, outside_market_action, tmd_review_overdue}, params: {agent_outside_market}, now}) -> GateVerdict`. It returns exactly the codes of §4 (`G05_PHASE_OFF`, `G05_TMD_REVIEW_OVERDUE`, `G05_NO_TMD_ANSWERS`, `G05_ANSWERS_EXPIRED`, `G05_OUTSIDE_MARKET`, `G05_OUTSIDE_MARKET_FLAGGED`, or `pass` / `not_applicable`) in the stated precedence, and it reads the account's standard answer set, not a per-offering response. A32 applies the change as a new evaluator version (A19 keeps every version registered for replay), and A19 retires its baseline stub. This loop is exempt from the escalation trigger on A32's evaluator, because it is recorded here before Specify.
- **(g) The person-only confirmation (loops A19.3, A37.2, A12.2, A32.3).** A19's `GateVerdict` gains the optional field `person_confirmation_required: {reason: string}`. A37's placement returns `428 CONFIRMATION_REQUIRED` with the sub-reason and creates no order when any verdict carries it and the principal is `AGNT`. A12's `agent_confirmation` row gains `person_only: boolean`; when it is true a mandate signature is refused with `CONFIRMATION_PERSON_ONLY` and only `POST /v1/agent-access/confirmations/{id}/confirm` with the person's fresh assertion resolves it. A32 emits `tmd.outside_market_dealing` only after the placement commits. These loops are exempt from the escalation trigger for the same reason as (f).
- **(h) The last-order check (loop A19.2).** A19's G11 evaluator reads the optional fact `retail_last_order_at` (absent for every non-retail offering and for `cooling_off_days = 0`) and fails `G11_RETAIL_ORDER_WINDOW_CLOSED`, naming the date, when `now > retail_last_order_at`. A fact provider cannot add a verdict, so this is A19's change, supplied with facts by this node.
- **(i) Material change for a new TMD or SPDS (data diffs to A36 and A37).** The `TMD_MATERIAL_CHANGE_REQUIRED` precheck on `tmd.publish`, the pinned SPDS set, `reflection_period` added to A36's re-confirmation states, and the SVCA `reflection_period -> revoked` guard row with cause `reconfirm_lapsed` in A37, as §3 describes.
- **(j) Graph edges.** The dependencies added to the Depends on line are added to `final-graph.json`, with `oracle: true` and the revised estimate.

If the user declines any of them, the node records the alternative in `A90.qa.md` and the hidden tests follow the settled reading.

## Owned paths (other nodes do not edit these)

- <workspace>\app\services\api\src\modules\scheme\
- <workspace>\app\apps\web\src\features\retail\
- <workspace>\app\apps\web\app\(investor)\target-market\

Inside the API module, by convention:
- `schemes/` (records, local transition table, binding watch);
- `sub-funds/` (requests, RE consent, establish effector through A33 and A34);
- `retail-terms/` (`retail-terms.ts` validator, `RegimeTermsProvider`, material-field diff);
- `tmd/` (records, question bank, publish effector, fit refresh, answer-set expiry, claim issue and revoke through A31, status contributor);
- `gates/` (`g05-facts.ts`, `g11-retail-facts.ts`);
- `cooling-off/` (`coolingOffFor`, clock policy);
- `outside-tmd/` (consumer, significance rules, assessment, A45 calls);
- `roles/` (INV-R assignment);
- `fixtures/p2-retail.ts` (the P2 scenario through `packages/sim`'s `generate` and `load`);
- `jobs/`, `events.ts`, `errors.ts`, `index.ts` (the only import surface).

In the web feature folder:
- `TargetMarketForm`, `TmdResultList`, `TmdOutsideExplainer`, `RetailPreviewBanner`, `CoolingOffBanner`, `RevokeOrderDialog`, with their Storybook stories and the SDK hooks.

Data diffs that other owners apply in the same milestone:
- A03's policy rows (`scheme.register`, `scheme.subfund.establish`, `tmd.publish`, `listing.go_live.retail`);
- A03's matrix cells from the route table: `scheme:draft`, `scheme:register_request`, `scheme:activate`, `scheme:read`, `scheme:subfund_request`, `scheme:subfund_read`, `scheme:re_consent`, `scheme:tmd_draft`, `scheme:tmd_publish_request`, `scheme:tmd_summary`, `tmd:read_own`, `ddo:read`, `ddo:assess_draft`, `ddo:assess`, each with its roles, `delegable` value (`true` for draft, request and read cells; `false` for activate, re-consent and assess) and module; plus the TP-RE cell on `offerings:pause`, scoped to offerings of its own scheme;
- A33's `tenant_approval_rule` action code `scheme.subfund.request` (default SP-OWN, count 1);
- A19's fix-it rows, the retired G05 stub and code, the G11 fact and code (spec points (d), (f), (h)), and the `person_confirmation_required` field (spec point (g));
- A32's evaluator contract and event timing (spec points (f), (g)) and the P2 option string;
- A12's `person_only` confirmation flag and `CONFIRMATION_PERSON_ONLY` (spec point (g));
- A35's disclosure set, statement rows, the `{{cooling_off_basis}}` token, the document type and, if needed, the pinned SPDS set (loop A35.1);
- A36's material fields and the `reflection_period` re-confirmation state;
- A37's placement 428 on `person_confirmation_required` and the `reconfirm_lapsed` guard row;
- A43's templates;
- A45's RE calendar row;
- A50's ONB-07 option strings and preview banner text, redrawn from this node's bank and A21's `retail_launch_label`.

## Design references

- **`design:onb ONB-06 card pattern`** in `<workspace>\design\RealX App.dc.html`: screen key `onb`, markup lines 305-360, `ONB` array lines 814-823, ONB-06 at line 819. ONB-07 reuses this card exactly:
  - sticky stepper column `minmax(220px,260px)`, with 22 px step dots in Geist Mono 500 11px and the green `#1F5E4B` done state;
  - white card, border `#E3DED4`, radius 14px, padding 36px, gap 24px, `max-width:720px`;
  - eyebrow "ONB-07 · Step n of N", Geist Mono 500 12px, green (the total comes from `stepsFor`, never the prototype's hard-coded 8);
  - h1 Newsreader 400 38px/1.1; sub 15px `#45484D`;
  - OptionCards: padding 16px 18px, radius 10px, border `#E3DED4` turning ink `#17191C` when selected, background `#FBFAF7` when selected, an 18px radio dot filled green; title Geist 500 15px; description 13px `#5E6166` with line height 1.45;
  - footer: "Back" outline `#D6D0C4` and the green pill primary (hover `#174838`).
- **The result list** uses the ONB-10 CheckList rows (lines 347-356: padding 14px 18px, dividers `#ECE8E0`, pills Geist Mono 500 11px, padding 4px 10px, radius 999px) with the TONE map: green `#E3EEE8`/`#1F5E4B`, amber `#F6ECD6`/`#7A4F00`, red `#F6E1DE`/`#A3312A`, grey `#ECE8E0`/`#4A4D52`.
- **`design-ext:ONB-07`** from [[A50 - Design extension A- investor and sponsor screens not in the prototype]] (design analysis §5: "OptionCards per question + green banner 'you're in the target market'. Same card as ONB-06"). The product-map mock copy is: bar "Is this right for you?"; sub "Retail offers from 2027 ask a few questions for each property's target market."; the three sample answers; the banner "PAD01: you're in the target market."; the button "Continue". The extension file approved under HG-REVIEW is the pixel reference. Where it and this note differ, the extension wins on layout and this note wins on behaviour.
- **The P1 preview state** uses the brick banner of the resale preview (line 482: `padding:14px 18px; background:#F5E4DB; color:#7A3A1E; radius 10px; 14px/1.5`). Its copy comes from A21's `retail_launch_label`, the same field as the home eyebrow "Wholesale investors · retail from 2027" (line 59) and the footer (line 516).
- **INV-03 cooling-off** uses A57's `reflection` status label "Cooling-off (P2)" in amber and its empty banner slot, in the buy stepper's timeline card style (dc lines 399-459). The revoke control uses the design's danger outline (red `#A3312A` border and text, pill radius). Money and times are in Geist Mono.
- **ONB-06's P2 fourth option and the ONB-10 "Target market (G05)" row** render through A56's existing components from API strings. No new component is drawn.
- **Fonts and tokens:** Newsreader for display, Geist for UI, Geist Mono for codes, amounts, dates and pills, all from A05's design-system package; no literal hex in `features/retail/` outside the tokens (lint).

## Requirements covered

- **Screen ONB-07 Target market check.** Done when:
  - the four standard questions render as the ONB-06 card with OptionCards;
  - submission is a personal passkey declaration through A32's route;
  - results show in or outside per live retail listing with the TMD link;
  - the `stop` and `flag` outcomes, the expired, cooldown, empty and P1 preview states all exist;
  - the step appears in A56's stepper only when the flag is on;
  - the INV-01 fix-it deep link returns the person to the buy step;
  - screenshots at 1280 px and 375 px match the extension and the ONB-06 tokens.
- **Journey J16 Retail purchase with a target market.** Done when every step can be driven at service level with the P2 scenario and, for the screens owned here, through Playwright component tests:
  - step 1, ONB-07 answers;
  - step 2, the retail listing shows the PDS and TMD;
  - step 3, G05 at the quote;
  - step 4, PDS and TMD acknowledged, with the statement for the listing's days;
  - step 5, the outside-TMD dealing with its 10-business-day item on ADM-26;
  - step 6, the order settles through A37 and A38.

  The catalogue's oracle hooks must hold: `reflection_period` only when the cooling-off parameter is set; an outside-TMD dealing creates exactly one report item with a 10-business-day clock. The browser journey end to end is A94's.
- **G05 Target market fit.** Done when:
  - every retail order is evaluated at quote, placement and allocation against the offering's current questionnaire version;
  - `stop` refuses and `flag` warns and records a dealing;
  - agent-placed flagged orders need the person;
  - expired answers and an overdue TMD review fail;
  - wholesale offerings are `not_applicable`;
  - the `TMD_FIT` claim mirrors the answer set on-chain.
- **G12 Disclosure acknowledged.** Done when:
  - the retail disclosure set is PDS (plus the current SPDS), TMD and fee schedule;
  - the retail statement matches the listing's `cooling_off_days` and funding mode;
  - an SPDS or new TMD mid-raise makes older acknowledgements stale and triggers re-confirmation;
  - no agent acknowledges a retail disclosure while `orders.agent_standing_consent` is `wholesale_only`.
- **G21 Licence perimeter.** Done when:
  - no scheme activates, no retail listing submits or goes live, and no retail order passes unless `offers.retail` is on, bound to this scheme's RE licence, and that licence holds the three retail activities;
  - a lapse suspends the scheme within one event and fails new retail orders by name, while holdings and the register are untouched.
- **M04 Offerings and deals.** Done when the retail listing is reviewed against the AU-R pack, with the TMD attached and the retail blockers visible on ADM-07; go-live needs the RE as a checker; and the retail fields are material. **M07 Onboarding review.** Done when the answer set, its expiry, the INV-R role, the ONB-10 target-market row and the flip watch exist. The wholesale half stays A32's. **M19 Reporting and filings.** Done when outside-TMD dealings feed A45's filing with the 10-business-day clock, need a reasoned human assessment, go to the issuer when significant, and the RE's own ASIC reporting sits on the calendar as the RE's.
- **SM-02 Wholesale certificate.** Done when this node adds no state, transition or guard:
  - a retail investor has no SM-02 row;
  - an expired wholesale certificate still blocks wholesale buying but not a retail purchase (G04 `not_applicable` for retail, G05 decides);
  - the P2 fourth ONB-06 option creates no classification.
- **SM-05 Order.** Done when:
  - retail orders take `reflection_period` only with days > 0 and the flag on, and wholesale orders never do;
  - revoke, the timer's end and early close behave as A37's table says;
  - late orders are refused so a cooling-off period is never cut short;
  - no money moves during reflection under `after_reflection`;
  - `revoked_in_reflection` refunds in full under `confirm`.
- **Founder requirement D04.** Done when every delegable operation works under a scoped token with parity to the screen, the answers and every checker, consent and activation act refuse agents before scope evaluation, and every agent action is audited on behalf of its human.

## Gauntlet loop for this node

**Specify.** The frozen spec is this note plus:
- the settled spec points (a) to (j), and the route table in §9;
- A03's SM-05 table, the new policy rows and the new matrix cells;
- A32's G05 evaluator as extended by (f), its tables and route contract;
- A19's `GateVerdict` with `person_confirmation_required`, and G11 with the retail fact;
- A12's confirmation contract with `person_only`, A60's confirmation screen and A101's sensitive-action notices;
- A98's TP-RE principal;
- A33's `tenant_approval_rule`;
- A54's eligibility filter, A76's ADM-26 rows and A104's and A100's catalogue and parity manifest;
- A35's disclosure-set, statement and acknowledgement contracts;
- A36's validate, precheck and material-field interfaces;
- A37's reflection transitions and cancel contract;
- A21's flag-binding and G21 interfaces;
- A45's `recordOutsideTmdDealing`;
- A31's claim interface;
- A13's calendar and the new clock policies;
- A19's `FactProvider` and gate sets;
- A56's `registerOnboardingStep` and `stepsFor` exports;
- A57's `reflection` slot;
- the A50 ONB-07 extension.

The node QA records `specVersion` and the hashes of those data files in the manifest. It then writes the hidden tests to `<workspace>\gauntlet\hidden\A90\`: Vitest suites over the module against the Docker Postgres with the sandbox rail and a fake A14 (recording requests and running effectors on demand), Playwright component tests for the retail feature, and the injectable clock.

The upstream loops of spec points (f) to (i) run and pass their owners' re-run hidden tests before this node's Build starts.

**Build (Grok 4.7 xhigh, split by the planner).** Four parallel sub-agents on disjoint sub-folders, then integration:
1. **Schemes and sub-funds:** the local transition table, binding watch, the A14 request builders and effectors, the A33 and A34 calls, the retail-terms validator and the `RegimeTermsProvider`, the pack draft.
2. **TMD:** the question bank, records and `tmd.publish`, fit refresh, answer-set expiry, the `TMD_FIT` claim through A31, INV-R assignment, the status contributor, the G05 and G11 fact providers, the review clock and stop-distribution.
3. **Cooling-off and outside-TMD:** `coolingOffFor` and the clock policy, the dealing consumer, significance rules, assessment and A45 wiring, notifications, the agent-confirmation sub-reason.
4. **Web and HTTP:** the zod schemas, routes with `x-realx-scope` and `requiresHuman`, the error catalogue and OpenAPI tags, the `features/retail/` components, the `/target-market` route in both modes, the CoolingOffBanner and RevokeOrderDialog in A57's slot, the P2 scenario fixture (the sample scheme run by YBP's hired RE "Meridian Fiduciary Services", the `PAD01` sub-fund listing with PDS v1 and TMD v0.3 from the product-map mocks, three retail personas: in market, outside with `flag`, outside with `stop`; a "both to sign" joint account, an SMSF with one authorised signatory, and a wholesale investor who chose "Answer later"; and the sponsor's retained-stake investor record).

Integration:
- the module's own Vitest suite (transition coverage, fast-check over answers and criteria, the reflection property, the clock vectors, the answer-leak scan);
- the J16 service-level run;
- screenshot tests;
- `npm run verify`;
- `app/evidence/A90.json`.

**Challenge (node QA, Opus 5.5 high).** The node QA runs the hidden tests, then works by hand against the running two-process stack with the P2 override:
- the node-specific challenges below;
- every route under every principal type and its agent (investor, joint, SMSF signatory, SP-OWN, SP-OPS, TP-RE, CCO, LEGL, CAPM, SUPP, and an AGNT of each);
- the flag toggled off and on around live orders;
- the design-fidelity comparison of ONB-07 and the cooling-off banner against the extension crops at two widths;
- a grep of logs, the outbox, `gate_evaluation` inputs, OpenAPI examples, SDK types and the evidence folder for planted answers and criteria;
- the `Date.now()` grep;
- the OpenAPI document diffed against the routes;
- a check that only the owned paths and the agreed data diffs changed.

The node QA writes `app/evidence/A90.qa.md`.

**Gate.** A pass unlocks A94 and A95 and publishes the hidden tests into `app/`. At most three fix loops; each loop re-runs the full hidden set and every failing challenge.

**Escalation trigger.** Escalate to the user on any of these:
- the same assumption fails three times;
- a fix would need a change to A37's SM-05 table, A32's evaluator, A35's acknowledgement rules, A21's binding rules or A45's filing contract beyond the loops already recorded in spec points (f) to (i) (a spec defect: reopen upstream);
- counsel answers that cooling-off must run after issue (a right to hand back issued units): that needs a new SM-05 path from `settled` through burn (A28), register removal (A18) and refund (A37), which `fund_at` cannot express;
- a wholesale order ever reaches `reflection_period`, a retail order passes G21 without a binding to the RE's licence, or an agent submits target-market answers (Critical, stops the milestone);
- a pass depends on a counsel answer not yet given (record it in HG-LEGAL and ask the user).

## Node-specific challenges (node QA)

- **The flag bound to the wrong licence (compliance, G21).**
  - Steps: SUPER binds `offers.retail` to the sample AFSL (refused by A21, `G21_BINDING_NOT_COVERED`); then to a second RE licence that holds the retail activities but is not this scheme's; CCO tries to activate the scheme.
  - Expected: `SCHEME_FLAG_BINDING_MISMATCH` naming both licence ids.
  - Then, with a correct binding and a live PAD01 raise, the RE licence expires under the clock. The scheme must be `suspended` within the event, new retail quotes and placements fail G21 naming the RE and the licence, and orders in reflection stay revocable. At allocation the excluded orders refund through A37. No register row or token changes.
  - Any retail order accepted after the lapse fails the node.
- **Cooling-off that cannot finish (money).**
  - Setup: PAD01 has `cooling_off_days = 14` and closes in 10 days.
  - The quote and the placement must refuse with `G11_RETAIL_ORDER_WINDOW_CLOSED` naming `last_order_at`, under both `fund_at` values (each with its own `last_order_at`). It must not accept an order whose cooling-off would be cut short by the scheduled close or expire unfunded.
  - A wholesale offering seeded with `cooling_off_days = 14` must never enter reflection.
  - A retail offering with 0 days must skip reflection and show the "no cooling-off" retail statement.
  - A revoke sent at the last second of the timer, racing the timer job on the other process, must leave exactly one transition: either `revoked` with no journal, or `awaiting_funds`.
  - Under `fund_at = confirm`, a revoke must refund `total_cents` to the cent within 5 business days.
- **Agent answers, agent outside the market (agent misuse, D04).**
  - Steps: an investor's agent calls `POST /v1/tmd/responses` with answers; the same token asks to be widened to the answers scope; the agent places a PAD01 order for a person whose result is `outside` with `flag`, carrying a valid mandate signing key within caps.
  - Expected: `AGNT_NOT_ALLOWED` before any scope check; `SCOPE_PENDING_COUNSEL` at issue; `428 CONFIRMATION_REQUIRED` with `outside_target_market` and a `person_only` confirmation. No order, dealing or filing row exists until the person confirms, and after confirmation there is exactly one order, one dealing and one filing.
  - Direct probe (API, not the screen): the agent then posts the same placement again with a correctly signed `RealX-Mandate-Signature` over the exact payload, within caps, bypassing the SDK and the UI. Expected: `CONFIRMATION_PERSON_ONLY`, and a database count of zero order, `outside_tmd_dealing`, filing and `tmd.outside_market_dealing` outbox rows for that payload before the person confirms. Any order row fails the node.
  - A CCO's agent posts an assessment draft (`not_significant`, `ISOLATED_SMALL`): 201 with a draft id, and the filing is unchanged. The same agent calls `.../assess`: `AGNT_NOT_ALLOWED`. The CCO then assesses with a fresh assertion, adopting the draft. A TP-RE's agent that tries to consent to a sub-fund gets `AGNT_NOT_ALLOWED`, and one that calls stop-distribution gets `AGNT_NOT_ALLOWED` (reason code `G22_SCOPE_NON_DELEGABLE`).
  - The MCP catalogue lists the draft tool and not `assess`, `re-consent`, `activate`, `reactivate` or `stop-distribution`.
- **Whose answers (joint and entity accounts, D04).**
  - Steps: on an "either to sign" joint account one holder answers; on a "both to sign" joint account the first holder answers and the second does not assert within 72 hours; then a second run where the second holder asserts over a different answers hash; on the SMSF a non-signatory member answers, then the authorised signatory answers.
  - Expected: the either-to-sign set takes effect with one `declaration_id`; the both-to-sign set stays `pending_cosign`, lapses at 72 hours with G05 still `G05_NO_TMD_ANSWERS`, and a mismatched hash never makes a set; the non-signatory gets `TMD_SIGNATORY_REQUIRED`; the signatory's set takes effect. In each case the answer set, INV-R and `TMD_FIT` attach to the account, not to the person.
- **The retained stake and the skipped wholesale buyer (G05, on-chain).**
  - Steps: PAD01 finalises with the sponsor's 1,000 bps retained stake; a wholesale investor who chose "Answer later" tries to buy PAD01.
  - Expected: the retained-stake mint succeeds with an `issuer_retained` `TMD_FIT` claim, which no order can use; the wholesale investor gets `G05_NO_TMD_ANSWERS` with the fix-it link, and buys after answering.
- **Gaming the questions (compliance, privacy).**
  - Steps: a person outside PAD01's market re-answers immediately; waits out the cooldown and flips one answer; then the node QA searches every response, error message, OpenAPI example, SDK type and `gate_evaluation` input readable by that person for the criteria.
  - Expected: `TMD_REANSWER_COOLDOWN`, then `tmd.answers.flip` raised once with the fact on the next order. The criteria must not appear anywhere readable.
  - A sponsor token asking for individual answers or `tmd_fit_result` rows must get 404. Only the four counts come back.
- **A new TMD mid-raise (G05, G12).**
  - Steps: while orders sit in reflection, awaiting funds and funded, a `tmd.publish` request for TMD v0.4 with stricter criteria is created without a linked material change; then CAPM opens `offer.material_change` naming v0.4 and a new SPDS, and the linked `tmd.publish` and the material change are approved in that order.
  - Expected: the unlinked request is refused `TMD_MATERIAL_CHANGE_REQUIRED`. After both approvals A36 opens its 5-business-day re-confirmation for every order in `reflection_period`, `awaiting_funds`, `funded` and `submitted`. Every open order's PDS acknowledgement becomes `DISCLOSURE_VERSION_STALE`. Fit results refresh once per investor. A reflection order not re-confirmed by the deadline ends `revoked` with cause `reconfirm_lapsed` (no journal under `after_reflection`; a full refund under `confirm`). An order that no longer fits is excluded at allocation and refunded by A37's ordinary path. No investor is treated as in market under a questionnaire version the offering no longer references.
  - A TMD record whose data hash differs from its PDF version must be refused with `TMD_RECORD_DOC_MISMATCH`.
- **The outside-TMD clock (compliance, M19).**
  - Steps: one flagged order is placed at 16:59 on the Friday before the King's Birthday holiday (Monday 14 June 2027); the event is replayed three times; nobody assesses the dealing.
  - Expected: exactly one dealing and one filing; a due date 10 business days later on A13's NSW calendar, skipping the holiday; one Sev2 escalation on breach; the filing still `draft`. Nothing auto-closes.
  - An order revoked in reflection keeps its dealing with `order_state = revoked` in the facts.
  - An assessment without a reason code gets `OTD_REASON_REQUIRED`. A second assessment gets `OTD_ALREADY_ASSESSED`.
- **Sub-fund segregation of duties (maker-checker).**
  - Steps: an SP-OWN requests a PAD01 sub-fund. The scheme's TP-RE consents. Then the same CCO who drafted the `scheme.register` request tries to check it, a TP-RE tries to be both the consenting party and the maker of `tmd.publish`, and the sponsor owner tries to consent in place of the RE.
  - Expected: `APR_MAKER_IS_CHECKER`, A14's refusal and `FORBIDDEN` in turn.
  - A sub-fund whose trustee is not the RE, or whose property already backs a P1 trust, must be refused (`SUBFUND_TRUSTEE_MUST_BE_RE`, `SUBFUND_PROPERTY_TAKEN`).
  - A request made by SP-OPS (or by an SP-OWN's agent) and not yet approved by a human SP-OWN must refuse RE consent with `SUBFUND_OWNER_APPROVAL_REQUIRED`; the SP-OWN whose agent made it cannot approve it.
- **Design fidelity and phase leakage.**
  - Setup: flag off, `PHASE=P1`.
  - Expected: `/target-market` shows only the brick preview banner and calls no P2 route. A56's stepper has no ONB-07 and ONB-06 shows three options. No `au_mis_retail` listing appears on PUB-03 or in the SDK's public list.
  - With the flag on, the ONB-07 card, OptionCards, pills and banner are compared at 1280 px and 375 px with the A50 extension and the ONB-06 tokens (fonts, colours, radii, spacing, the inert Continue until every question is answered).
  - Any hard-coded "2027", legal sentence or hex value in `features/retail/` fails.

## Pass criteria

- **Hidden tests.** The hidden tests in `<workspace>\gauntlet\hidden\A90\` pass 100 % on three consecutive runs with different seeds. This includes a fast-check property over at least 10,000 random (regime, flag and phase, answer set or null, criteria, action, expiry, review state, principal type) tuples: G05's verdict, reason codes, precedence and `person_confirmation_required` agree with a reference implementation of spec point (f), and no tuple yields `pass` for an outside, expired, overdue or phase-off case. It also includes the declaration vectors for individual, either-to-sign joint, both-to-sign joint (co-signed, lapsed and mismatched hash), SMSF, company and trust accounts.
- **Reflection property.** In 10,000 random placements across both regimes, random `cooling_off_days` from 0 to 30, both `fund_at` values and the flag on and off, `reflection_period` is entered exactly when the regime is retail, the days are above 0 and the flag is on. No journal exists for any order revoked under `after_reflection`. Every `confirm`-mode revoke refunds `total_cents` exactly.
- **Transitions and approvals.**
  - Every transition of the scheme and sub-fund-request tables has an allowed test, and every unlisted pair is refused.
  - The four A14 action codes create requests with the policy codes named here, and their effectors apply exactly one change each.
  - A generated matrix of every route by every principal type and its AGNT gives allowed, draft-only or refused. It is derived from the §9 route table and A03's data (including the new cells), not from the running routes, and the running stack agrees with it, with A11's SoD rules and with the `requiresHuman` list, with zero exceptions.
  - The direct signed-payload probe of spec point (g) leaves zero order, dealing, filing and outbox rows in 100 of 100 runs.
- **Perimeter.** 100 % of scheme activations, retail submits, go-lives and orders attempted without an `offers.retail` binding to the scheme's RE licence are refused with the named code. A licence lapse suspends the scheme within one event in 50 of 50 timed runs on two processes.
- **Clocks.** Under the injectable clock, the outside-TMD due date equals A13's `businessDaysAfter(awareness, 10, NSW)` in every vector, including public holidays and the October daylight-saving change. The cooling-off `ends_at` shown on INV-03 equals A37's timer to the second. The answer-set expiry, TMD review warning and overdue jobs each fire once and are no-ops when re-run.
- **Privacy.** A scan of logs, the outbox, audit and `gate_evaluation` rows, OpenAPI examples, SDK types, error messages and the evidence folder finds zero occurrences of planted answers outside the person's own answer-set read, and zero occurrences of listing criteria outside the scheme module's tables. Sponsor and partner reads return counts only.
- **Design.** ONB-07 (all seven states) and the cooling-off banner and revoke dialog pass the screenshot comparison at 1280 px and 375 px against the A50 extension crops and the ONB-06 card tokens within the tolerance set in [[App Gauntlet]]. The P1 snapshot equals the preview banner.
- **J16.** J16 runs at service level with the P2 scenario three times: answers, retail quote with G05, acknowledgement with the retail statement, reflection, funding, allocation, register entry, mint with `TMD_FIT` present (including the sponsor's retained stake under its `issuer_retained` claim), a wholesale "Answer later" buyer refused `G05_NO_TMD_ANSWERS` and then buying after answering, and one outside-TMD dealing with its filing and clock. A10's chain verifier passes afterwards, and the number of `scheme.*`, `subfund.*`, `tmd.*`, `retail.*` and `outside_tmd.*` events equals the number of actions taken.
- **OpenAPI, SDK and MCP.** Every route, error code and schema is in the OpenAPI document. `packages/sdk` compiles without manual edits. The MCP catalogue lists the delegable tools and none of the `requiresHuman` routes. Every A90 route appears in A100's parity manifest with its screen action and delegation value, and A100's suite, re-run over the manifest, is green for those rows.
- **Hygiene.** Line coverage is at least 90 % on the module and 85 % on `features/retail/`. The module boundary lint passes. `npm run verify` is green. Only the owned paths and the agreed data diffs changed. `app/evidence/A90.json` and `A90.qa.md` exist and match disk.

## Human gates touched

None are crossed. The node runs with the P2 override in `local|test`:
- a sample scheme, ARSN and RE licence that carry no legal meaning;
- the sandbox bank rail and registrar;
- the local chain;
- A11's software authenticator;
- the injectable clock.

`offers.retail` stays off and unbound in every non-local profile.

**HG-LEGAL** (recorded, not decided here; each is a parameter or policy row followed by a re-run of this node's hidden tests):
- engaging the hired RE;
- registering the scheme with ASIC, and its constitution and compliance plan;
- whether one registered scheme with a sub-fund per property works as proposed;
- PDS and TMD content;
- **CQ-06:** retail cooling-off, meaning none or 14 days, how it is counted, whether it runs before funding or after issue (after issue is an escalation, not a row), whether it applies to an illiquid scheme, whether an early close may end it, and whether a re-confirmation after a material change restarts it (`cooling_off_days`, `fund_at`, `retail.cooling_off_count_basis`, `retail.reconfirm_restarts_cooling_off`);
- whose declaration the target-market answers are for joint and entity accounts, and the co-sign window (`tmd.cosign_window_hours`);
- **CQ-01:** whether target-market answers must be personal, and whether an agent may confirm an outside-market order (`tmd.agent_outside_market`);
- **CQ-02:** retail disclosure acknowledgement under standing consent (`orders.agent_standing_consent`);
- DDO distributor duties: what counts as a significant dealing (`ddo.significance`), when the 10-business-day clock starts, and what the report to the issuer must contain;
- whether a wholesale client buying into a retail sub-fund must answer the questions;
- answer validity and the re-answer cooldown;
- whether the RE must be a checker on every retail go-live;
- whether to hide or flag, per TMD.

**HG-LAUNCH (via A94):**
- Binding `offers.retail` to the RE licence in a real environment is the founder's P2 launch step, after the licence and relief records exist.
- A94 documents it.
- This node passes on software evidence only.

**HG-CAPITAL and HG-VENDOR (later):**
- Real retail client money in each sub-fund's trust account.
- The contracted registry for the sub-fund registers.

**HG-PENTEST (later):**
- The answer privacy, the criteria confidentiality, the outside-market confirmation and the cooling-off revoke race are in scope of the external penetration test before the P2 launch.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
