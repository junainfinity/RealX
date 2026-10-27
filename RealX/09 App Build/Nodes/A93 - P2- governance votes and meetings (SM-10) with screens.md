---
title: "A93 - P2: governance votes and meetings (SM-10) with screens"
created: 2026-09-24
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am9]
node: A93
milestone: AM9
lane: "P2"
oracle: yes
human_gate: none
---

# A93 - P2: governance votes and meetings (SM-10) with screens

**Unitholder votes run on the platform from the notice to the executed outcome. Proposals and statutory notices are filed as drafts; a sponsor sends its notice to RealX for approval, and the notice is published as `4E` (LEGL maker, CCO checker). The statutory notice period is measured from the moment the notice is actually published, not from filing. Publishing takes a record-date snapshot from the legal unit register (a registry or certified version, never the internal mirror), and it is refused whenever the proposal lets silence count for the manager. Each vote has a members' meeting record (format, chair, questions and minutes). A meeting halt on the resale board, run through the halts module, is linked to the vote, and voting cannot open until that halt is confirmed. Each ballot is signed by the holder (and, on a joint account that needs both, by the co-holder) and weighted by the holding at the snapshot, never by today's balance: by units for a sub-fund vote, or by the dollar value of the holding for a scheme-wide vote when counsel says so. The responsible entity and its associates are excluded from voting on a resolution in which they have an interest other than as members. An agent may cast a ballot only under a mandate whose scope names governance, and only once counsel allows delegated voting; the ballot code refuses agent ballots unless the counsel row says exactly `true`. Quorum and thresholds come from the trust deed as data (75 % at any time, 50 % at each 5-year anniversary for a liquidity vote), computed in integer arithmetic. Certification is by the trustee/RE or an appointed scrutineer, deciding in the partner portal, never by the proposer or the proposer's agents. Failed votes go into a cooldown. Execution is tracked, and a sale hands off to the wind-down pipeline, which reuses the distributions engine for the exit waterfall and token operations for the burn. The node also builds the four screens GOV-01, GOV-02, SPN-13 and ADM-23. The P1 reduced mode (off-platform notice and result records) is specified in §8, but its placement in a P1 node is still open (see §8).**

**Build status:** not started. **Milestone:** AM9 (P2 features, from about 8 April 2027). **Lane:** P2. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 10-14 agent-hours (build, challenge and verify loops), as in `final-graph.json`. The independent plan review found this too low for the scope (an API module with six jobs and eight maker-checker actions, signature and cosign handling, quarantine, interested-party exclusion, the meeting record and four screens). Its proposed split is A93a (module and jobs, 12-14 h) and A93b (the four screens, 8-12 h), with §8 moved to a P1 node. That split is a graph change and is not made in this note.

**Depends on:** [[A18 - Unit register (legal record) and registry sandbox adapter]], [[A91 - P2- resale board market module (SM-09)]], [[A14 - Approvals engine and maker-checker (SM-00)]], [[A40 - Distributions engine (SM-08)]], [[A52 - Web app shells, routing and passkey sign-in (public, investor, sponsor)]], [[A53 - Operator console shell, hardware-key sign-in and approval action pattern]], [[A50 - Design extension A- investor and sponsor screens not in the prototype]], [[A51 - Design extension B- operator console screens not in the prototype]]
**Interfaces used that are not yet direct edges in `final-graph.json` (the graph must add them):** [[A43 - Communications, notifications, tasks and marketing claims (G17)]] (notices, claims lint, blocked phrases, tasks), [[A98 - Partner principals- trustee-RE, valuer, property manager, auditor, regulator observer and licensee users]] (TP-RE and the `TRUSTEE_OF_VEHICLE` checker resolver), [[A105 - Partner portal- trustee-RE, auditor, regulator observer and licensee views]] (PTR-07, where the TP-RE decides), [[A101 - Agent guard- anomaly detection, sensitive-action notices and tiered kill switches]] (the agent-vote notice), [[A104 - MCP server generated from the OpenAPI document]] (MCP tools), [[A100 - Agent-parity suite- every UI action has an API and MCP equivalent under the same gates]] (parity rows), [[A06 - Seed data and scenario simulator]] (BAY01 fixtures), [[A38 - Finalise-and-mint and revert-and-refund pipelines]] (the first issue date), [[A33 - Sponsor tenancy, entities, licences of sponsors and admission pipeline]] (trustee entity and associates) and [[A46 - Job scheduler, integrations health and webhooks]] (webhooks).
**Unlocks:** [[A94 - P2- end-to-end journeys and launch update]]

**Oracle rule:** this is an oracle node (`oracle: yes`, set on 24 Sep 2026 by the plan fixer, as the plan review proposed). No money or supply moves inside it: the exit waterfall and the burn run in A40, A48 and A28 under their own oracles. But voting power and the legal outcome of a trust do move here, so the node QA writes hidden oracle tests from this frozen spec to `<workspace>\gauntlet\hidden\A93\` before the build starts. The hidden set holds:
- an SM-10 transition oracle;
- a tally oracle over random snapshots, ballots, rule sets, weight bases and exclusion sets;
- the silence and snapshot-weight probes;
- the notice-period-at-publish probes;
- the interested-party exclusion matrix;
- the proposer-cannot-certify matrix (by `person_id`) and the empty-certifier-set probe;
- the halt-linkage invariant, including the lift after a failed vote or a quorum miss;
- the agent-ballot matrix under both values of the counsel flag and with the row missing;
- the joint-account cosign sequences;
- the tipping-off diff of every holder- and sponsor-visible read;
- screen contract and fidelity checks for GOV-01, GOV-02, SPN-13 and ADM-23.

The builder never reads that folder, and the tests are published into `app/` only after the gate passes ([[App Gauntlet]] §2).

This note is not legal advice. The following are all rows of data marked "set with counsel": every threshold, quorum, notice period, anniversary window, cooldown, threshold basis, comparator, weight basis, interested-party rule, meeting format, rule on frozen or restricted holders and wording of the rule preview, and whether an agent may cast or cosign a holder's vote. Counsel's answers change rows, not code. The deed of each trust (the umbrella constitution and sub-fund terms in P2) is the source, and the BrickX figures are only the seeded defaults. Section numbers of the Corporations Act named below are the plan's reading, for counsel to confirm.

## Build brief

This node has two parts:
- **API.** Module `governance` in the Fastify monolith (`services/api/src/modules/governance/`). Its worker jobs are registered from the same folder through A09's job registry.
- **Screens.** GOV-01 and GOV-02 in the investor app, SPN-13 in the sponsor console of `apps/web`, and ADM-23 in the operator console. The TP-RE's certification decision is rendered by A105's PTR-07 (§2), not by this node.

Every rule is enforced in the API, never only in a screen. Every route carries `x-realx-scope` and is in the OpenAPI document from the first commit, so `packages/sdk` and the MCP tool catalogue (A104) see it before the module passes. Every mutating route requires an `Idempotency-Key` (A09's convention). Time comes only from A09's injectable clock. The module talks to other modules only through their `index.ts`:
- A18: register versions, version comparison and cap table as of a date;
- A40: the snapshot canonicaliser, imported directly from A40's `index.ts`;
- A48: halts, the P2 exit path, the wind-down pipeline and, through it, A40's exit waterfall and A34's property transitions;
- A34: `nav_per_unit_cents` of the valuation current at a record date (read only, for dollar-value weighting);
- A33: the vehicle's `trustee_entity_id` and the entity associate list (§1);
- A30: `screeningStatus` and verified signatories;
- A14: approval requests and effectors;
- A98: the `TRUSTEE_OF_VEHICLE` checker-set resolver;
- A35: notice, memo, minutes and result documents;
- A43: notices to holders, the claims lint and tasks;
- A46: webhook event registration;
- A12: agent authority and `delegable.ts` reads;
- A13: SLA clocks and alerts;
- A21: the `governance.votes` flag.

**0. Phase and flag**

- The whole P2 path is behind the flag `governance.votes` (A08 default off; A21 binds it only to a licence or relief record that holds `operate_registered_scheme`). With the flag off, or while `PHASE` is P1:
  - every SM-10 route refuses with `GOV_PHASE_P1_RECORDS_ONLY`;
  - A03's phase-leakage test (no SM-10 transition in P1) must pass unchanged;
  - only the reduced mode in §8 is reachable.
- The screens follow A52's route table: GOV-01 and GOV-02 use `whenOff: 'hidden'`, and SPN-13 uses `whenOff: 'preview'` with A05's `PreviewBanner`. ADM-23 carries the P2 `PreviewBanner` and shows the records tab of §8 live.

**1. Data and identifiers (tables registered through A04's module-schema registry; RLS by tenant and vehicle)**

- **`governance_ruleset`.** One row per vehicle (per sub-fund, and one for the umbrella scheme in P2), versioned, never edited in place. Each row is seeded from the deed with `set_with_counsel: true`. Columns:

  | Column | Content and seed |
  |---|---|
  | `resolution_types[]` | `ordinary` (seed 5,000 bp of votes cast, comparator `gt`), `special` (7,500 bp of votes cast, `gte`), `extraordinary` (5,000 bp of total units, `gte`) |
  | `liquidity_rule` | resolution kind `liquidity_sale`: 5,000 bp inside the anniversary window, 7,500 bp outside it |
  | `anniversary_window_days` | seed 90, centred on each 5-year anniversary of the vehicle's first issue date from A38 |
  | `quorum_bps` | seed 2,500 of the eligible weight at the snapshot |
  | `quorum_min_holders` | seed 2 |
  | `notice_min_days` | seed 21 calendar days, measured from publication (§2), counsel to confirm against the Corporations Act and the deed |
  | `extension_notice_min_days` | seed 7 calendar days between an extension's re-notice and the new `closes_at` |
  | `cooldown_days` | seed 180 |
  | `live_tally_visible` | seed `false` until counsel answers (§3 "Restricted holders" explains why); the design's "Votes for so far" meter then shows only the threshold marker and "Tally shown at close" |
  | `restricted_holder_rule` | seed `quarantine`, see §3 |
  | `interested_party_rule` | seed `exclude_re_and_associates` (Corporations Act s253E: the RE and its associates may not vote their interests on a resolution in which they have an interest other than as members). Per `resolution_kind`, the parties treated as interested: the RE entity (`trustee_entity_id`) and its associates for every kind; for `change_property_manager` also the incumbent and proposed manager entities and their associates; the sponsor entity where counsel treats it as an associate of the RE |
  | `excluded_in_denominators` | seed `exclude`: excluded weight is left out of the quorum and `total_units` denominators; the other value `include` exists for counsel |
  | `weight_basis` | seed `units` for a sub-fund vote and `dollar_value` for a scheme-wide vote (s253C and s253F: on a poll, one vote per dollar of the value of the member's interests), with the value source in §1 `governance_snapshot` |
  | `meeting_required` | seed `true` for a P2 registered scheme |
  | `meeting_formats_allowed[]` | seed `in_person`, `hybrid`; `virtual` only when the constitution allows it |
  | `minutes_due_months` | seed 1 |
  | `fallback_text` | the deed's anti-deadlock fallback, proposed, shown and never automated |
  | `silence_rule` | locked to `not_counted`; no other value exists in the enum |

  A vehicle without a ruleset refuses every proposal with `GOV_RULESET_MISSING`. A ruleset change is an A14 request `governance.ruleset.change` (`4E`, LEGL maker, CCO checker) and never alters a proposal already filed.
- **`governance_proposal`.** Id `GOV-<yyyymmdd>-<6 base32>` from A02.

  | Column | Content |
  |---|---|
  | `scope` | `sub_fund` (one vehicle) or `scheme` (every sub-fund of the P2 umbrella: RE replacement, constitution amendment and any other resolution counsel says is a vote of all members) |
  | `vehicle_id`, `tenant_id` | the trust (or the umbrella scheme for `scope = scheme`) and its sponsor tenant (the platform tenant for a scheme-wide vote) |
  | `vehicle_ids[]` | for `scope = scheme`, every sub-fund with units on issue at the record date, resolved at snapshot lock |
  | `origin` | `sponsor`, `realx`, `trustee` or `members_requisition` |
  | `proposer_principal_id` | the principal that created the draft |
  | `proposer_person_ids[]` | the proposer identity set, by person: the `person_id` of every principal or agent token (resolved to its human, A11's rule that an agent counts as its human) that created, edited or filed the draft; for `origin = sponsor` this includes every co-drafter of the tenant. Every principal and token of any of these persons is in the set, whatever role it signs in with |
  | `resolution_kind` | `liquidity_sale`, `sale_and_wind_up`, `change_property_manager`, `change_trustee`, `deed_amendment`, `borrowing_limit`, `other` |
  | `resolution_type` | from the ruleset |
  | `resolution_text` | plain text, at most 2,000 characters, run through A43's claims lint |
  | `notice_doc_id`, `memo_doc_id` | A35 documents of type `notice` and `explanatory_memo` |
  | `silence_default` | `none` or `manager` |
  | `record_date` | the record date |
  | `voting_opens_at`, `closes_at` | the voting window (Australia/Sydney); the poll closes at `closes_at` |
  | `halt_starts_at` | defaults to `voting_opens_at`; must be on or before it (§2) |
  | `excluded_entity_ids[]` | interested parties resolved at filing from `interested_party_rule`, A33's `trustee_entity_id` and A33's associate list, plus any entity LEGL adds with a reason and evidence document; frozen at publish and shown in the rule preview |
  | `scrutineer_principal_id` | the scrutineer appointed in the publish payload, or null (§2 "Certify") |
  | `ruleset_version_id`, `threshold_bps`, `threshold_basis`, `comparator`, `quorum_bps`, `weight_basis` | resolved at filing and frozen at publish |
  | `exit_track_opened` | `true` when filing a sale-kind proposal moved the property to `exit_proposed` (§2) |
  | `state` | SM-10 |
  | `submit_approval_id`, `publish_approval_id`, `certify_approval_id`, `execute_approval_id` | the A14 approval ids |
  | `published_at` | the effector time of the publish approval |
  | `snapshot_id` | the locked snapshot |
  | `result_sha256` | the result hash |
  | `cooldown_until` | the end of the cooldown |

  `silence_default` exists only so that a proposal imported from a sponsor's template can be refused by name: any value other than `none` blocks publishing.
- **`governance_meeting`.** One row per proposal when `meeting_required` is true:
  - `format` (`in_person`, `virtual` or `hybrid`, and only a value in `meeting_formats_allowed`, else `GOV_MEETING_FORMAT_NOT_ALLOWED`);
  - `venue_text`, `access_link` (shown only to holders eligible at the snapshot and to staff; never in events or notices other than the holder's own);
  - `starts_at`, which must lie inside `[voting_opens_at, closes_at)`; the poll closes at `closes_at`;
  - `chair_principal_id` (a LEGL or TP-RE person, named in the publish payload);
  - `minutes_doc_id` (A35 `kind = minutes`) and `minutes_due_at = starts_at + minutes_due_months`, tracked by an A13 clock opened when the meeting starts (Sev3 at 7 days before, Sev2 on breach). Counsel confirms the minute-book rule.
- **`governance_question`.** Insert-only question register: `proposal_id`, `holder_ref` (never shown to other holders), `text` (at most 1,000 characters), `asked_at`, `answer_text`, `answered_by`, `answered_at`. An answer passes A43's claims lint before it is visible to every eligible holder.
- **`governance_snapshot`.** One row per proposal, built with A40's exported canonicaliser so that A20's rule R13 recomputes vote snapshots exactly as it does distribution snapshots. Columns:
  - `register_version_ids[]`: per vehicle, the A18 `register_version` used. It must be one of these, and nothing else:
    - a `certified` version (A18's derived status) whose `as_of` is the close of the record date, or
    - a `source = registry` version whose `as_of` is the close of the record date and whose A18 `register_version_compare` row for that `as_of` has `result = match`.

    "The close of the record date" is 00:00 Australia/Sydney on `record_date + 1`, the instant A18's 00:00 `register.snapshot` job and the registry extract describe. An `internal`-source version never locks a vote snapshot on its own. If A40's "as_of equals the record date" wording turns out to mean a different instant, the difference is raised to A40's and A18's owners at Specify; this node does not pick one silently. In every case the three-way reconciliation for that date must have closed with zero breaks.
  - `holders[]`: `vehicle_id`, `holder_ref`, `investor_id`, `account_id`, `units`, `frozen_units`, `restricted` (from A30's `screeningStatus`, need-to-know), `vote_excluded` (the account's investor is, or is linked through A30 to, an entity in `excluded_entity_ids[]`), `nav_per_unit_cents` (only for `weight_basis = dollar_value`: A34's published valuation current at the record date for that vehicle), `weight` (`units` for the unit basis; `units * nav_per_unit_cents` in integer cents for the dollar basis).
  - `total_weight`, `excluded_weight`, `eligible_weight` (`total_weight - excluded_weight` when `excluded_in_denominators = exclude`, else `total_weight`), `holder_count`, `snapshot_sha256`.

  Locking is refused with `GOV_NO_REGISTER_VERSION` when no qualifying version exists yet, with `GOV_RECON_BREAK_OPEN` when the date's reconciliation has a break, and with `GOV_WEIGHT_PRICE_MISSING` when the dollar basis needs a valuation that A34 does not have for a sub-fund. For `scope = scheme`, every sub-fund must qualify before any row is locked. In each case the job retries every 15 minutes and raises a Sev2 through A13 after 2 hours.
- **`ballot`.** Insert-only; a new choice is a new row. Id `BLT-`. Columns:
  - `proposal_id`, `account_id`, `holder_ref`;
  - `choice`: `for`, `against` or `abstain`;
  - `weight_units` and `weight`: the sums of the account's non-excluded snapshot rows, copied from the snapshot and never from the request;
  - `cast_by_principal_id`, `on_behalf_of_id`, `token_id`;
  - `signature_kind`: `passkey` or `mandate_key`;
  - `assertion_id` or `mandate_signature`, `payload_sha256`;
  - `cosign_principal_id`, `cosign_signature_kind`, `cosign_assertion_id` or `cosign_mandate_signature`, `cosigned_at` (joint accounts only);
  - `supersedes_ballot_id`;
  - `status`: `counted`, `pending_cosign`, `superseded`, `quarantined` or `lapsed_cosign`;
  - `cast_at`.
- **`governance_tally`.**
  - `for_weight`, `against_weight`, `abstain_weight`, `turnout_weight`, `voters`, `eligible_weight`, `excluded_weight`;
  - `for_bps_of_cast`, `for_bps_of_eligible`, `against_bps_of_eligible`, `turnout_bps` (display only);
  - `quorum_met`, `threshold_met`, `ballot_merkle_root`, `result_sha256`, `computed_at`, `ruleset_version_id`.
- **`execution_step`.** Rows per certified proposal, each with `kind`, `state`, `evidence_sha256`, `approval_id` and `source_event_id`.
- **`governance_record`.** The reduced records mode, see §8.

**2. SM-10 lifecycle, verbatim from A03**

- The table is A03's: 13 states (`draft`, `filed`, `withdrawn`, `discussion`, `voting`, `tallying`, `quorum_failed`, `passed`, `failed`, `certified`, `executing`, `executed`, `cooldown`) and 13 transitions, with the guard `silence_not_defaulted_to_manager` on `filed -> discussion` and A11's SoD rule 7 `cannot_certify_own_proposal`. Every transition goes through `packages/domain` `transition()`; any other pair is refused with `SM10_ILLEGAL_TRANSITION`.
- **Create, edit, preview, file.**
  - `POST /v1/governance/proposals` creates a `draft`. Who may create:
    - SP-IR or SP-OWN, for their tenant's vehicles (`scope = sub_fund` only);
    - LEGL or ASTM, for any vehicle and for `scope = scheme`;
    - the TP-RE of the vehicle (A98's partner principal; the cell is an A03 data diff, see "Upstream amendments"). The TP-RE has no builder screen: its proposals are created through the API or entered by LEGL on ADM-23 with `origin = trustee` and the trustee's resolution document.
  - `PATCH` edits a draft only; after filing it returns `GOV_PROPOSAL_LOCKED`. `POST /v1/governance/proposals/{id}/copy` creates a new draft with the same text and documents, for a proposal that was withdrawn or refused for its dates.
  - `GET /v1/governance/proposals/{id}/rule-preview` returns the rule preview. It holds the scope, resolution type, threshold, basis, comparator, weight basis, quorum and notice minimum; the excluded parties by entity name and reason; whether `closes_at` falls inside an anniversary window, and the next anniversary; the cooldown state; the deed clause references; and one plain-English sentence, for example "Special resolution · 75 percent of votes cast needed · quorum 25 percent of units at the record date · the responsible entity and its associates cannot vote".
  - `POST /v1/governance/proposals/{id}/file` moves `draft -> filed`. It is SP-OWN for sponsor proposals, with a second SP-OWN when the tenant's SPN-14 rule asks for two, and LEGL or TP-RE otherwise. It checks:
    - the resolution text, notice document and memo document are present (`GOV_DOCS_REQUIRED`);
    - `voting_opens_at >= now + notice_min_days` (`GOV_NOTICE_PERIOD_TOO_SHORT`); this is only an early warning, and the binding check runs at publish;
    - `record_date <= voting_opens_at` and `closes_at > voting_opens_at`;
    - `halt_starts_at <= voting_opens_at` (`GOV_HALT_WINDOW_INVALID`);
    - the meeting, when required: format allowed and `starts_at` inside the window;
    - the vehicle is not in a cooldown for the same `resolution_kind` (`GOV_COOLDOWN_ACTIVE`, with `cooldown_until`), except for `members_requisition` and `change_trustee`, whose exemption is a counsel row.

    Filing resolves and freezes the ruleset version, threshold, quorum, weight basis and `excluded_entity_ids[]`. A `liquidity_sale` gets 5,000 bp only when `closes_at` lies inside an anniversary window. For the tenant's vehicles, a filed sponsor proposal appears in LEGL's ADM-23 queue and as a "Needs you" task through A43.
  - **Sale kinds open the exit track at filing (J08 step 1).** For `liquidity_sale` and `sale_and_wind_up`, the filing transaction calls A48's P2 exit propose path, which drives A34's `operating -> exit_proposed` with the proposal id and the filing audit event as evidence, and sets `exit_track_opened`. On `withdrawn` and on `cooldown`, the same transaction calls A48's P2 exit withdraw path (`exit_proposed -> operating`). This needs the A48 amendment listed under "Upstream amendments"; until it lands, the call is behind a fake in this node's tests and J08 step 1 is recorded as blocked upstream.
  - `POST /v1/governance/proposals/{id}/withdraw` moves `filed -> withdrawn`. It is available to the proposer or LEGL, with a reason code.
- **Issue notice from the sponsor (SPN-13 "Issue notice").** `POST /v1/governance/proposals/{id}/issue-notice`, by SP-OWN or SP-IR of the tenant (operation `governance.proposals.issue_notice`), on a `filed` sponsor proposal. It creates the A14 request `governance.notice.submit` (`4E`, the sponsor user as maker, LEGL as checker) over the same payload as publish below, runs the same prechecks (silence, notice period, halt window, certifier set), and returns its `APR-` id, which SPN-13 shows as "Sent to RealX for approval · APR-…". When LEGL approves, the effector drafts `governance.notice.publish` in LEGL's ADM-23 queue with the approved payload; a LEGL person then submits it as maker and CCO checks it. A LEGL rejection returns the proposal's queue item to the sponsor with the reason. RealX-, trustee- and requisition-origin proposals skip this step.
- **Publish: `filed -> discussion`.**
  - `POST /v1/governance/proposals/{id}/publish` creates an A14 request `governance.notice.publish` under `4E`, with LEGL as maker and CCO as checker, deciding with a hardware key in A53's pattern.
  - The payload covers:
    - the canonical proposal hash;
    - the notice and memo document hashes;
    - the ruleset version, record date, voting window, halt window and excluded entities;
    - the meeting row (format, venue or link hash, `starts_at`, chair);
    - `scrutineer_principal_id` or null;
    - the holder-notice template version.
  - Checks run twice, in A14's precheck when the request is created and again in the effector when it is applied:
    - the silence guard (`GOV_SILENCE_DEFAULTS_TO_MANAGER`);
    - **the notice period, measured from publication**: `voting_opens_at >= clock.now() + notice_min_days`, and the same for `governance_meeting.starts_at`, both on the injectable clock at that moment (`GOV_NOTICE_PERIOD_TOO_SHORT`). A proposal filed 22 days before `voting_opens_at` whose CCO decision comes 2 days later is refused at the effector. The remedy is withdraw and `copy` with new dates;
    - `halt_starts_at <= voting_opens_at` (`GOV_HALT_WINDOW_INVALID`);
    - the certifier set is not empty (`GOV_CERTIFIER_SET_EMPTY`, see "Certify");
    - a non-null `scrutineer_principal_id` names a principal A98 recognises as a scrutineer for the vehicle (`GOV_SCRUTINEER_NOT_AVAILABLE`; until A98 defines scrutineers, every non-null value gets this code).
  - The effector, in one transaction:
    - applies the transition and sets `published_at`;
    - publishes both documents through A35 under the same approval id (A35's publish row has the same policy, so one approval binds both);
    - schedules the snapshot lock for the record date;
    - emits `governance.notice.published`, which A43 turns into a notice to every current holder, and which A46 delivers as a holder-scoped webhook;
    - records the scheduled meeting halt with A48 through `POST /internal/halts/events` (`governance.meeting.scheduled`, carrying `max(halt_starts_at, published_at)` and the criterion "the vote closes and the result is certified or the proposal enters cooldown").
- **Snapshot.** Worker job `governance.lock_snapshot` runs every 15 minutes and is idempotent. It locks the snapshot once the qualifying register version (§1) and its clean reconciliation exist, marks `vote_excluded` rows, and computes weights. A second notice goes to the holders at the record date who were not holders at publish.
- **Open voting: `discussion -> voting`.**
  - Worker job `governance.open_voting` fires at `voting_opens_at`. It requires a locked snapshot (`GOV_SNAPSHOT_NOT_LOCKED`) and emits `governance.meeting.opened` to A48 and as a holder-scoped webhook through A46.
  - It applies the transition only after A48's `haltState(vehicleId)` shows an open halt with cause `meeting` for this proposal (`GOV_HALT_NOT_CONFIRMED` otherwise); for `scope = scheme`, every sub-fund must show it. Voting implies halted, so no ballot can be cast before that.
  - If the halt is not confirmed within 15 minutes, the job raises a Sev2 and keeps retrying. `closes_at` never moves by itself.
  - Only `POST /v1/governance/proposals/{id}/extend` moves it: an A14 request `governance.window.extend` (`4E`, LEGL maker, CCO checker) that re-notifies every eligible holder. It may only move `closes_at` later, and, in precheck and again in the effector, the new `closes_at` must be at least `extension_notice_min_days` after the effector time (`GOV_NOTICE_PERIOD_TOO_SHORT`).
- **Close and tally.**
  - Worker job `governance.cosign_lapse` runs at `closes_at` before the close: every `pending_cosign` ballot becomes `lapsed_cosign`.
  - Worker job `governance.close_voting` then moves `voting -> tallying` at `closes_at` (AUTO).
  - It runs the pure `tally()` from §4 and applies exactly one of `tallying -> passed`, `-> failed` or `-> quorum_failed` as `AUTO` on an exact `result_sha256` match. While a quarantined ballot is undisposed and `restricted_holder_rule` says "wait", the proposal stays `tallying` and every holder and sponsor sees the same "Result being certified" state.
  - `failed -> cooldown` and `quorum_failed -> cooldown` follow in the same job. They set `cooldown_until = closes_at + cooldown_days`, record the deed's fallback text on the proposal, emit `governance.cooldown.started` (which meets A48's meeting-halt criterion once A48 is amended) and close the exit track if it was opened.
  - `governance.result.final` is emitted on `certified` and on `cooldown`, to A48 and as a holder-scoped webhook through A46.
- **Certify: `passed -> certified`.**
  - `POST /v1/governance/proposals/{id}/certify` creates an A14 request `governance.result.certify` under `4E`:
    - maker: LEGL, or SP-OWN of the tenant (the SPN-13 "Certify result with trustee" button). The proposer may be the maker: certification is the checker's act, and the product map's rule is that the proposer cannot certify;
    - checker set: A98's `TRUSTEE_OF_VEHICLE` resolver for the vehicle (active TP-RE persons with a hardware key), plus `scrutineer_principal_id` when set, minus every principal and token of a person in `proposer_person_ids[]`, minus A14's maker identity set.
  - The checker decides in A105's PTR-07 "Approvals awaiting me". The request's `checker_view` carries `result_sha256`, the tally (for, against, abstain, voters, eligible and excluded weight, quorum and threshold result), `snapshot_sha256`, the ruleset version and the ballot Merkle root, and the certifier's assertion is over `result_sha256` together with the tally and the snapshot hash. ADM-23 shows the request's state; it has no checker control for the TP-RE.
  - An attempt by a person in the proposer identity set is refused with `GOV_PROPOSER_CANNOT_CERTIFY`, as well as A14's `APR_MAKER_IS_CHECKER` where it applies. A11's rule 7 refuses an ASTM proposer independently. When the resolved checker set is empty (for example a trustee-origin proposal whose only TP-RE person filed it and no scrutineer is appointed), publishing and certify requests are both refused with `GOV_CERTIFIER_SET_EMPTY`.
  - The effector applies the transition and emits `governance.result.certified`. A48 marks the meeting-halt criterion met; lifting the halt stays with A48 and A91 (MKTO with CCO, J08 step 7), never here.
- **Execute: `certified -> executing`.**
  - `POST /v1/governance/proposals/{id}/execute` creates an A14 request `governance.outcome.execute` under `4E` with ASTM as maker. The checker is CFO for `liquidity_sale` and `sale_and_wind_up`, and LEGL for every other kind.
  - The effector by kind:
    - **Sale.** Calls A48's P2 exit path, which drives A34's `exit_proposed -> exit_approved` (the property entered `exit_proposed` at filing) with the certification's approval id as evidence. A48's `WD_P2_VOTE_REQUIRED` is satisfied only by a `certified` proposal of a sale kind for that vehicle. The wind-down pipeline then runs the exit waterfall through A40 and the burn (which must equal supply) through A28. This node only mirrors A48's events into `execution_step` rows: `exit_approved`, `under_sale`, `sold`, `waterfall_paid`, `tokens_burned`, `register_closed` (from `winddown.step_completed`, `winddown.waterfall_requested`, `winddown.burn_requested` and `winddown.register_closed`).
    - **`change_property_manager`.** Opens an A34 property-operations change record.
    - **`deed_amendment`.** Opens an A35 document publish draft.
    - **Other kinds.** Open a checklist of evidence steps.
  - `executing -> executed` fires on A48's `vehicle.wound_down` for sale kinds. For other kinds it fires when the last step has evidence, recorded by ASTM with `POST /v1/governance/proposals/{id}/steps/{step}/evidence`.
- **Meeting procedure.** `PATCH /v1/governance/proposals/{id}/meeting` (LEGL, before publish) edits the meeting row. `POST /v1/governance/votes/{id}/questions` (an eligible holder or their mandated agent, until `starts_at`) adds a question; `POST /v1/governance/questions/{id}/answer` (LEGL or the chair) answers it. `POST /v1/governance/proposals/{id}/minutes` (LEGL) attaches the A35 minutes draft; A35 publishes it under its own policy and the A13 clock closes. When `meeting_required` is false for a vehicle, these routes return `GOV_MEETING_NOT_REQUIRED` and the screens hide the meeting row.
- **Spec gaps to settle at Specify, before the node QA writes the hidden tests.**
  - **(a)** SM-10 has no certifier step for `failed` and `quorum_failed`. Proposed: the TP-RE countersigns the tally as a record through the same `governance.result.certify` action with `outcome = failed`, but this does not gate `-> cooldown`, which stays `AUTO`. A48's criterion is met on `cooldown` entry.
  - **(b)** The design's percentages disagree:
    - the product-map mock (46 % For, 12 % Against, 61 % Turnout) reads only as percent of units;
    - A50's GOV-02 and SPN-13 mock (For 61 %, Turnout 58 %) reads only as percent of votes cast.

    Proposed: the API returns both bases, and the deed's `threshold_basis` decides which one the ThresholdMeter shows and which one the result uses. A50 corrects its mock caption to name the basis.
  - **(c)** A vote of the members to requisition a meeting (a statutory right) is modelled as `origin = members_requisition`, filed by LEGL with the requisition document. Counting the requisitioning holders is out of scope.
  - **(d)** A scheme-wide vote has no single sponsor tenant and no single property. Proposed: `tenant_id` is the platform tenant, SPN-13 lists it read-only for every tenant with a sub-fund in it, and sale kinds are not allowed at `scope = scheme` (`GOV_SCOPE_KIND_NOT_ALLOWED`).

  The user decides these points; if a point is declined, the verbatim table is implemented and the gap is recorded in `A93.qa.md`.

**3. Ballots (GOV-02 and the `votes.cast` operation)**

- **Cast.** `POST /v1/governance/proposals/{id}/ballots` takes `{account_id, choice}` and a mandatory `Idempotency-Key`.
  - The server computes the weight from the snapshot. A request that names a weight is refused with `GOV_WEIGHT_NOT_ACCEPTED`.
  - An account with zero weight at the snapshot gets `GOV_NOT_ELIGIBLE` (403), with `record_date` in `details` so GOV-02 can draw its not-eligible state.
  - An account whose snapshot rows are all `vote_excluded` gets `GOV_HOLDER_EXCLUDED` (403), with the rule-preview sentence in `details`; GOV-02 draws its excluded state.
  - Outside the window the call gets `GOV_VOTING_NOT_OPEN` or `GOV_VOTING_CLOSED`. The window is closed from `closes_at` inclusive, on the injectable clock.
  - While the meeting halt is not open the call gets `GOV_HALT_NOT_CONFIRMED`.
- **Signature.** Every ballot is signed over a canonical payload: proposal id, account id, choice, `weight_units`, `weight`, `snapshot_sha256`, idempotency key and a server nonce.
  - **A person** signs with a fresh passkey assertion (A11 primitive, purpose `ballot_sign`, at most 5 minutes old). A11's `FRESH_ASSERTION_*` codes apply.
  - **An agent** signs with the mandate's Ed25519 `signing_key` (A12). Without a key the call returns `428 CONFIRMATION_REQUIRED`, and the person confirms with a passkey on GOV-02 or ACC-05.
  - **Entity accounts:** only a verified signatory from A30 may sign (`GOV_SIGNATORY_REQUIRED`).
- **Joint accounts with `signing_rule = both`.**
  - The first holder's signed ballot is stored `pending_cosign`. A43 sends the other holder "Your co-holder voted on <trust> · your signature is needed by <close>", and the ballot's `my_ballot.cosign_state` is `needed` for the co-holder and `waiting` for the first holder.
  - `POST /v1/governance/ballots/{id}/cosign` (operation `governance.ballots.cosign`, mandatory `Idempotency-Key`) takes the co-holder's fresh passkey assertion (purpose `ballot_cosign`) or the co-holder's own mandate signature over the ballot's `payload_sha256`. The caller must resolve to the other joint holder: the first holder, the first holder's agent or anyone else gets `GOV_COSIGN_OTHER_HOLDER`. A ballot not in `pending_cosign` gets `GOV_COSIGN_NOT_PENDING`. A cosign at or after `closes_at` gets `GOV_VOTING_CLOSED`.
  - A valid cosign makes the ballot `counted`. If the first holder changes the vote while it is pending, the pending row is `superseded` and the co-holder must cosign the new one. At `closes_at`, `governance.cosign_lapse` marks every remaining `pending_cosign` row `lapsed_cosign`; it is not counted, and the holders' GOV-02 shows "Not counted: your co-holder didn't sign before the close".
- **Changing a vote.** A new signed ballot before `closes_at` inserts a row with `supersedes_ballot_id` and marks the previous one `superseded`. The tally counts the latest `counted` ballot per account. Nothing is ever updated in place.
- **Silence.** Holders who do not vote are not counted anywhere. No code path creates a ballot the holder or their mandated agent did not sign: no default ballot, no "manager votes undirected units" and no proxy-to-chair option. A test enumerates every insert into `ballot` and asserts that each one comes from the signed-cast route.
- **Restricted holders.** A holder whose A30 `screeningStatus` is `restricted` at cast time gets the same `201` response and the same GOV-02 voted state as anyone else (tipping-off rule, AML/CTF Act s123). The row is stored `quarantined` and left out of every decision until the MLRO disposes of it. The disposal is an A14 request `governance.ballot.release` (`4E`, MLRO maker, CCO checker) or `governance.ballot.exclude` (the same policy, with reason code).
  - **Nothing visible outside MLRO and CCO may differ.** Every holder-, sponsor-, LEGL-, TP-RE- and scrutineer-visible aggregate (GOV-02's tally, the SPN-13 and ADM-23 monitor, turnout by hour, voter count) includes a quarantined ballot's weight and choice on a display-only basis, exactly as if it were counted. Decisions (`tally()` for quorum, threshold and `result_sha256`) never include it until it is released.
  - Because the final result can still differ from the last displayed figure after an exclusion, `live_tally_visible` is seeded `false` until counsel answers: holders and sponsors then see turnout only, and the For and Against figures appear with the result.
  - If a proposal reaches `closes_at` with a quarantined ballot, the tally records `quarantined_weight` in the need-to-know section only, and the counsel row `restricted_holder_rule` decides whether the result waits. The seeded value is "wait"; every holder sees "Result being certified", which is the same state they see during any certification.
- **Frozen and locked units.** These vote. The weight is the legal holding at the record date; A40's snapshot pays on `units`, not `units - frozen_units`, for the same reason. Counsel confirms the default.
- **Agent-cast ballots: fail closed in this module.** Before any other check on an agent-cast or agent-cosigned ballot, `ballots/` reads A12's delegable lookup for `votes.cast` (and `governance.ballots.cosign`). Unless the value is exactly `true`, the call is refused with `G22_SCOPE_NON_DELEGABLE`: a missing row, `undefined`, `"counsel"` or `false` all refuse. This guard stands even if A03's row or A12's pending-counsel list is wrong.
- **Investor reads.**
  - `GET /v1/governance/votes` (GOV-01) returns three groups: `open`, `upcoming` and `results`. It covers only vehicles the caller's accounts hold now or held at the record date of a listed proposal (for a scheme-wide vote, any sub-fund). Each item carries the vehicle name and code, resolution, state pill code, `closes_at`, `my_ballot` and the result percentages. The empty state is `{open: [], upcoming: [], results: [], next: null}`.
  - `GET /v1/governance/votes/{id}` (GOV-02) returns:
    - the notice key-value rows and documents, and the meeting row (format, `starts_at`, venue or link for eligible holders, chair, minutes document once published);
    - the answered questions, and the caller's own questions;
    - `voting_power: {units, weight, weight_basis, bps_of_eligible, current_units, caption_code}`, where `caption_code = record_date_differs` when today's units differ, and `excluded` when the holder is excluded;
    - `my_ballot` with `signed_at`, `signature_kind`, `cast_by` (`you`, `agent <name>`, `co-holder`), `cosign_state` (`none`, `needed`, `waiting`, `lapsed`) and `can_change_until`;
    - the tally, following `live_tally_visible` and the display rule above;
    - `threshold_bps`, `basis`, `quorum_met`;
    - the halt notice sentence from A48's `haltNotice()`, never from screen copy;
    - `eligible`, `state`, `certified_by_role`, `fallback_text` and `cooldown_until` for failed votes.
- **Who sees ballots.**
  - An investor sees only their own ballots.
  - Sponsors see aggregates only.
  - LEGL, CCO, TP-RE and the scrutineer see the ballot list with masked holder refs.
  - `quarantined` rows are visible only to MLRO and CCO (A04's need-to-know RLS). Every other role sees the ballot as counted until release or exclusion, and every aggregate they see includes it (the display rule above).

**4. Tally, quorum and thresholds (pure functions in the module's `domain/` folder, integer arithmetic only)**

- Weights are integers: units for `weight_basis = units`, cents for `dollar_value`. The tally counts only `counted` ballots, each account at most once, and never a `vote_excluded` row:
  - `turnout = for + against + abstain`;
  - `denominator = eligible_weight` (§1);
  - **quorum** is met when `turnout * 10,000 >= quorum_bps * denominator` **and** `voters >= quorum_min_holders`;
  - **threshold**, basis `votes_cast`: `for * 10,000 >= threshold_bps * (for + against)`, with abstentions out of the denominator; `gt` replaces `>=` with `>`, and an empty denominator never passes;
  - **threshold**, basis `total_units`: `for * 10,000 >= threshold_bps * denominator`.
- The outcome is `quorum_failed` when quorum is not met, whatever the For count. It is `passed` when quorum and threshold are met, and `failed` otherwise.
- Display basis points use A02's half-even rounding and are never used in a decision.
- `ballot_merkle_root` is the root of the sorted hashes of counted ballots. `result_sha256 = sha256(canonical {proposal_id, snapshot_sha256, ruleset_version_id, threshold_bps, basis, comparator, quorum_bps, weight_basis, eligible_weight, excluded_weight, for, against, abstain, voters, merkle_root})`.
- Recomputing the tally from the snapshot and the ballot rows reproduces `result_sha256` byte for byte. The node QA checks this, and A20's rule R13 applies to the snapshot. Rule R15 (vote anchors) stays a stub. When A35's anchor option is on, the certified result document is anchored through A35's DocumentAnchor, which is the only on-chain footprint of a vote.
- `GET /v1/governance/proposals/{id}/monitor` serves ADM-23 and SPN-13. It returns the displayed tally so far (§3 display rule), turnout by hour, pending cosigns, the time to close, the quorum and threshold state, and, for MLRO and CCO only, `quarantined_weight`.

**5. Gates**

- **G10 Trading halt.** This node is the source of the meeting cause; A48 owns the `Halt` row and A19's G10 read. It guarantees four things:
  - voting never opens without a confirmed meeting halt (on every sub-fund for a scheme-wide vote);
  - the halt's reopening criterion names this proposal;
  - `governance.halt_invariant`, a worker job run every minute, raises a Sev1 through A13 if a proposal is in `voting` while its vehicle has no open meeting halt;
  - A48's lift is refused with `HALT_CRITERIA_NOT_MET` until `governance.result.certified` or `governance.cooldown.started` arrives, and is allowed after either (the second needs the A48 amendment below).

  A resale order during voting is refused by A91 through G10; this node's integration test asserts the refusal but does not implement it.
- **G13 Maker-checker.**
  - Eight A14 action codes: notice submit, publish, certify, execute, window extend, ruleset change, and ballot release and exclude.
  - The record publish in §8.
  - All are `4E` with human checkers deciding by assertion; A14 refuses tokens with `APR_CHECKER_MUST_BE_HUMAN`.
- **G22 Agent authority.** See §6.
- This node registers no evaluator with A19. It supplies the events and reads that G10 and G13 use.

**6. Agents and tokens (D04, G22): parity for every action this node exposes**

- Every route passes through A12's `agentAuth` and G22 evaluator unchanged, and is enumerated by A100's parity suite with its operation id and MCP tool (A104):

  | Operations | Who can use them through the API |
  |---|---|
  | `governance.votes.list`, `governance.votes.get` | investor read family |
  | `votes.cast` (A50's name; also used to change a vote), `governance.ballots.cosign` | investor governance family, subject to the counsel row |
  | `governance.questions.create` | investor governance family |
  | `governance.proposals.create / update / copy / preview / file / withdraw` | sponsor SP-IR and SP-OWN; staff LEGL and ASTM |
  | `governance.proposals.issue_notice` | sponsor SP-IR and SP-OWN (maker only) |
  | `governance.proposals.publish`, `governance.proposals.certify`, `governance.proposals.extend`, `governance.proposals.meeting.update`, `governance.proposals.minutes`, `governance.questions.answer` | makers only (LEGL; SP-OWN for certify) |
  | `governance.proposals.execute`, `governance.steps.evidence` | ASTM |
  | `governance.monitor.get` | all of the above |
  | `governance.records.*` | §8 |

  A100's rows must hold for these UI and API pairs, among others: SPN-13 "Issue notice" and `issue_notice` return the same `APR-` id shape; SPN-13 "Certify result with trustee" is visible and callable for exactly the SP-OWN principals the API accepts as maker, including the proposer; GOV-02 "Sign as co-holder" and `governance.ballots.cosign`.
- **Webhooks (A46).** Holder-scoped events `governance.notice.published`, `governance.meeting.opened` and `governance.result.final`, with ids and codes only, so an investor's agent learns of new notices, opened votes and results without polling. Each is delivered only to webhooks of principals whose accounts held the vehicle at publish or at the record date.
- **Agent-cast ballots.** The `votes.cast` cell must be `"counsel"` in A03's `delegable.ts` (the row is an upstream amendment, below). The question is whether a vote cast by the holder's agent is the holder's own signed vote or needs the proxy formalities of the constitution and the Corporations Act; A50 logged it under HG-LEGAL. Until counsel answers:
  - a token cannot be issued with that cell (`SCOPE_PENDING_COUNSEL`, A12);
  - a call reaching the route with it is refused `AGNT_NOT_ALLOWED` with reason code `G22_SCOPE_PENDING_COUNSEL` by A11's route flag (A12 §5 order of agent refusals); this module's fail-closed guard (§3) stands behind the flag and answers `G22_SCOPE_NON_DELEGABLE` only if a call ever passes the flag.

  When the row flips to `true`, which is a data change, an agent may cast only when all of these hold:
  - the mandate's scope names the governance family cell;
  - `allowed_vehicle_ids` includes the vehicle;
  - the mandate is live at `cast_at`;
  - the ballot is signed with the mandate key, or the person confirms it.

  Each agent ballot emits `agent.action.governance`, which A101 turns into the notice "Agent '<name>' voted <choice> on your behalf · change it until <close>" through A43, delivered to the person and never to the agent. GOV-02 shows the same line. The hidden tests run under both values of the flag and with the row missing.
- **Staff and partner agents.** A LEGL agent may draft proposals, run the rule preview and submit the publish, extend and certify requests as maker. It can never be the CCO checker. A TP-RE or scrutineer agent can never certify, because certification is a checker act (A14 refuses it before scope evaluation). An ASTM agent may submit `execute` and record evidence as maker.
- **Sponsor agents.** An SP-IR or SP-OWN agent may draft, file and issue the notice as maker. When the tenant's rule needs two people, the agent counts as its human (CQ-04), so the second SP-OWN must be another person. The agent's human joins `proposer_person_ids[]` when the agent drafts or files.
- **Non-delegable.** Nothing in this node is on D04's non-delegable list except that a co-holder's cosign can never be made by the first holder or the first holder's agent: `GOV_COSIGN_OTHER_HOLDER`. Each holder signs for themselves or through their own mandate.
- **Audit.** Every agent action is audited as "agent <token prefix> on behalf of <person>" and appears in the person's activity and on ADM-35.

**7. Screens**

- **GOV-01, `apps/web/app/(investor)/votes/page.tsx`** (exports `screen = 'GOV-01'` and `designKey = 'design-ext:GOV-01'`):
  - three TaskRow groups in this order: Open votes (amber "Open" pill and a "Vote" row button to GOV-02), Upcoming meetings (grey "Upcoming", with the meeting date), and Results history (green "Passed", grey "Failed" and "Quorum not met", each with the certified percentage);
  - the empty state "No votes for the trusts you hold · votes appear here when a sponsor issues a notice";
  - hidden in P1 per A52.
- **GOV-02, `votes/[id]/page.tsx`.** Top to bottom:
  1. The dark ProofBand-style halt banner, with its sentence from `haltNotice()`.
  2. The notice as a KeyValueList (Resolution, Type, Proposed by, Record date, Meeting as "<format> · 1 Oct 2026, 10 am Sydney time", Closes as "7 Oct 2026, 5 pm Sydney time") with DocumentRows for the notice, the memo and, once published, the minutes.
  3. A voting-power KpiTile ("120 tokens at the record date · 1.20 percent of 10,000"). A Geist Mono caption appears when today's holding differs.
  4. For / Against / Abstain FilterChips.
  5. The ThresholdMeter "Votes for so far" (or "Tally shown at close" with the marker only, while `live_tally_visible` is false), with a 2px ink marker at the threshold and the caption "<type> · <n> percent of <basis> needed · quorum reached | not yet reached".
  6. "Sign and submit vote", which opens the passkey DropBox "Sign your vote with Face ID, Touch ID or Windows Hello".
  7. Questions: the answered questions, and "Ask a question" (TextArea and outlineSoft button) until the meeting starts.

  States:
  - not voted;
  - voted: a green DecisionBanner "You voted For · signed <time> · you can change it until <close>" and a "Change vote" outlineSoft button;
  - waiting for co-holder (first holder): amber "Waiting for your co-holder's signature · needed by <close>";
  - co-holder signature needed (second holder): amber "Your co-holder voted <choice> · sign by <close> for it to count" and a "Sign as co-holder" primary button that opens the passkey DropBox;
  - co-holder signature lapsed: grey "Not counted: your co-holder didn't sign before the close";
  - result being certified;
  - closed-passed: meter frozen, pill, "Certified by the trustee" line, no button;
  - closed-failed: meter frozen, "Failed · cooldown until <date>" and the grey line "The trust deed's fallback applies: <fallback_text> (proposed)";
  - quorum not met: "Quorum not met · cooldown until <date>" and the same fallback line for a liquidity vote;
  - not eligible: "You didn't hold <trust> tokens at the record date, so you can't vote on this resolution", with no button;
  - excluded: "As an associate of the responsible entity, this holding can't vote on this resolution", with no button;
  - voted by an agent: shown only when the counsel row is `true`;
  - refused: A09 error codes mapped to plain sentences, never a bare 4xx.
- **SPN-13, `/s/[tenant]/meetings`.** A page file added beside A52's thin files that re-exports `SponsorMeetingsScreen` from `src/features/governance/`. It contains:
  - the tenant's proposals list (and scheme-wide votes read-only);
  - the draft builder, with resolution TextArea, dates, the halt schedule (validated on or before the voting start), the meeting row, a rule preview card, and the notice and memo DropBoxes linked to A35 drafts;
  - "File", then "Issue notice" (`issue_notice`), which ends in the banner "Sent to RealX for approval · APR-…";
  - KpiGrid (For, Against, Turnout, Closes), the ThresholdMeter and the dark halt banner;
  - a resolution KeyValueList;
  - "Certify result with trustee" as primary, disabled until the proposal is `passed`, and shown to every SP-OWN of the tenant including the proposer, because it submits only the maker side. The banner then reads "Sent to the trustee for certification · APR-…". A proposer is never offered a checker control anywhere;
  - states: draft, filed, sent to RealX, discussion, voting open, result being certified, certified, **failed** ("Failed · cooldown until <date> · deed fallback: <fallback_text> (proposed)", A50's deadlock-fallback Banner), **quorum not met** ("Quorum not met · cooldown until <date>", with the same fallback line for a liquidity vote), refused;
  - preview mode when the flag is off.
- **ADM-23, console route group `(governance)`, page `governance` (product-map route `/admin/governance`), `designKey = 'design-ext:ADM-23'`.**
  - Tabs: Proposals (DataTable with state pills), Calendar (notice, record date, meeting date and format, window and halt per vehicle), Builder (the proposal builder card with the rule preview KeyValueList, the meeting row, the excluded parties and the optional scrutineer), Ballot monitor (KpiGrid, ThresholdMeter, turnout by hour, pending cosigns, questions awaiting an answer, the MLRO- and CCO-only quarantine count), Execution tracker (Timeline: notice filed → discussion → voting → tallying → certified by the RE → executing → executed, plus the mirrored wind-down steps for sales; minutes due date), and Records (§8).
  - The actions "Publish notice", "Extend window", "Certify" (maker side), "Execute outcome", "Release or exclude ballot", "Answer question" and "Publish record" all go through A53's privileged-action pattern and `KeyConfirmModal` where they create an A14 request. No console code calls an approval decision directly (A53's lint rule). A sponsor's "Issue notice" request appears in LEGL's A66 approvals inbox and on the Proposals tab.
  - The states are default, voting open, result being certified, certified, **failed with cooldown and the deed fallback line** ("Failed · cooldown until <date> · deed fallback: <fallback_text> (proposed)"), **quorum not met** ("Quorum not met · cooldown until <date>"), empty ("No proposals yet") and refused.
- **Shared rules for all four screens.**
  - Every screen reads and writes only through `packages/sdk`, which A100's lint enforces.
  - Every amount and percentage comes from the API.
  - Every id (GOV-, APR-, BLT-) renders in Geist Mono.
  - The feature code lives in `src/features/governance/` of each app.
  - States not drawn in A50's and A51's frames (co-holder states, excluded, meeting row, questions, failed and quorum-not-met on ADM-23) use only existing primitives and are requested from A50 and A51 as frame additions (below).

**8. Reduced records mode (M17 "at launch"): placement open**

- **Placement.** The product map's M17 row says "At launch, a reduced version records notices and results", and the requirements catalogue marks M17 "P2 (reduced notices P1)". This node sits in AM9, after the P1 launch (A88, A89), so anything built here is not available at the P1 launch. The plan review's fix is to move this section into a P1 node before A88 (for example A70 or A75 for the console tab, with A64 for the sponsor side), or to record a founder decision that P1 launches without it. Both are graph or founder changes outside this note. Until one is made, this node keeps the contract below so it is not lost, and does not claim it exists at launch.
- `governance_record` rows record off-platform notices and results. Columns:
  - `kind`: `notice` or `result`;
  - `vehicle_id`, `meeting_date`, `resolution_text`;
  - `doc_id` (A35), `reported_for_bps`, `reported_against_bps`, `outcome`, `source` = `off_platform_trustee`.
- `POST /v1/governance/records` creates a draft. `POST /v1/governance/records/{id}/publish` is an A14 request `governance.record.publish` (`4E`, LEGL maker, CCO checker). `GET /v1/governance/records?vehicle=` lists them.
- Holders see a published record as its A35 document (kind `notice` or `result`) in the investor document vault and on the holding's documents list, through A35's own reads; no other module reads `governance_record`.
- No ballot, snapshot, halt or SM-10 transition exists in this mode. A test runs every SM-10 route under `PHASE=P1` and expects `GOV_PHASE_P1_RECORDS_ONLY`.

**9. Audit, events and errors**

- **Audit.** Exactly one hash-chained A10 audit event per transition, notice submission, ballot, supersession, cosign, lapse, quarantine disposal, snapshot lock, question, answer, minutes attachment, record publish and step evidence. The events are:
  - `governance.proposal.created`, `.filed`, `.withdrawn`, `.copied`;
  - `governance.notice.submitted`, `governance.notice.published`, `governance.snapshot.locked`;
  - `governance.meeting.opened`, `governance.voting.closed`, `governance.question.asked`, `.answered`, `governance.minutes.attached`;
  - `governance.result.passed`, `.failed`, `.quorum_failed`, `.certified`;
  - `governance.cooldown.started`, `governance.outcome.executing`, `.executed`;
  - `governance.ballot.cast`, `.superseded`, `.cosigned`, `.lapsed`, `.quarantined` (need-to-know), `.released`, `.excluded`;
  - `governance.record.published`.

  Each event carries `actor_id`, `on_behalf_of_id`, `token_id` and `approval_id` where they exist. Ballot events carry the choice only for the holder's own activity feed; audit readers other than LEGL, CCO and TP-RE see "ballot cast" without the choice.
- **Outbox (A09).** The same names, plus `governance.meeting.scheduled` and `governance.result.final` for A48 and `agent.action.governance` for A101. Events carry ids and codes, never names, choices or meeting links.
- **Error codes.** Stable strings in the OpenAPI document, returned with 403, 409, 422 or 428:
  - this node's: `GOV_PHASE_P1_RECORDS_ONLY`, `GOV_RULESET_MISSING`, `GOV_PROPOSAL_LOCKED`, `GOV_DOCS_REQUIRED`, `GOV_NOTICE_PERIOD_TOO_SHORT`, `GOV_HALT_WINDOW_INVALID`, `GOV_COOLDOWN_ACTIVE`, `GOV_SILENCE_DEFAULTS_TO_MANAGER`, `GOV_SCOPE_KIND_NOT_ALLOWED`, `GOV_MEETING_FORMAT_NOT_ALLOWED`, `GOV_MEETING_NOT_REQUIRED`, `GOV_NO_REGISTER_VERSION`, `GOV_RECON_BREAK_OPEN`, `GOV_WEIGHT_PRICE_MISSING`, `GOV_SNAPSHOT_NOT_LOCKED`, `GOV_HALT_NOT_CONFIRMED`, `GOV_NOT_ELIGIBLE`, `GOV_HOLDER_EXCLUDED`, `GOV_VOTING_NOT_OPEN`, `GOV_VOTING_CLOSED`, `GOV_WEIGHT_NOT_ACCEPTED`, `GOV_SIGNATORY_REQUIRED`, `GOV_COSIGN_OTHER_HOLDER`, `GOV_COSIGN_NOT_PENDING`, `GOV_CERTIFIER_SET_EMPTY`, `GOV_SCRUTINEER_NOT_AVAILABLE`, `GOV_PROPOSER_CANNOT_CERTIFY`, `GOV_NOT_PASSED`, `SM10_ILLEGAL_TRANSITION`;
  - reused: A11's `FRESH_ASSERTION_*`, A12's `G22_*` and `CONFIRMATION_REQUIRED`, A14's `APR_*`.
- **Empty and edge states.**
  - A proposal that left voting returns 200 with its terminal state, never 404.
  - A sponsor asking for another tenant's proposal gets 404.
  - An investor asking for a vote on a trust they never held gets 404.
  - A ballot or cosign after close gets 409 `GOV_VOTING_CLOSED`.
  - A publish on a non-`filed` proposal gets `SM10_ILLEGAL_TRANSITION`.

**10. Upstream amendments this node needs (applied by their owners as spec amendments before A93's Specify stage)**

This node does not edit these. Each is listed with its owner so the orchestrator can schedule it; a missing one is an escalation, not a local workaround.
- **A03** (data): the SM-10 table unchanged apart from the settled spec points; a `delegable.ts` row `votes.cast = "counsel"` (and `governance.ballots.cosign = "counsel"`) with a decision id, added to A03 §7's counsel list; the TP-RE cells for proposal create and file; the policy rows for the eight A14 action codes and `governance.record.publish`; the ruleset seed rows of §1.
- **A12**: `votes.cast` and `governance.ballots.cosign` in its `SCOPE_PENDING_COUNSEL` list.
- **A14**: the `checker_view` fields for `governance.result.certify` (§2).
- **A48**: the meeting-halt criterion met on `governance.result.certified` **or** `governance.cooldown.started` (today A48 seeds only "the result is certified", so a failed vote could never lift the halt), with a test that lifts the halt after a quorum miss; a P2 exit propose and withdraw path driven by a filed or withdrawn sale-kind proposal (J08 step 1); `vehicle.wound_down` confirmed as the completion event this node consumes.
- **A98**: a scrutineer (a partner kind or a TP-RE-like engagement per vehicle) that its resolver can return; until then `scrutineer_principal_id` is always null.
- **A105**: `governance.result.certify` in PTR-07's list of request types the trustee sees, with the `checker_view` above.
- **A33**: an associate (related-party) read per entity, used for `excluded_entity_ids[]`; until it exists, LEGL enters the excluded entities at filing with evidence.
- **A43**: the proxy-to-manager phrase row in the blocked list; the notice templates (notice, second notice, co-holder signature needed, extension re-notice, minutes available).
- **A101**: the `agent.action.governance` notice template.
- **A46**: registration of the three holder-scoped webhook events.
- **A100**: the parity rows named in §6.
- **A50 and A51**: frame additions for the states in §7 not yet drawn, and the caption fix in spec point (b).
- **A06**: fixtures for a joint "both" account, a restricted holder, an RE-associated holder and a two-sub-fund umbrella with different `nav_per_unit_cents`.
- **Graph (`final-graph.json`)**: the edges listed at the top, the SPN-13 page folder in `owned_paths`, and the plan review's proposals on the estimate split and the placement of §8 (its oracle proposal was applied on 24 Sep 2026).

## Owned paths (other nodes do not edit these)

- <workspace>\app\services\api\src\modules\governance\
- <workspace>\app\apps\web\app\(investor)\votes\
- <workspace>\app\apps\web\app\(sponsor)\s\[tenant]\meetings\ (the SPN-13 page folder, A52 route table)
- <workspace>\app\apps\web\src\features\governance\
- <workspace>\app\apps\console\app\(governance)\
- <workspace>\app\apps\console\src\features\governance\

A52's rule has dependent nodes add pages beside its frozen thin files, so this node also adds the SPN-13 page folder `<workspace>\app\apps\web\app\(sponsor)\s\[tenant]\meetings\`. It holds one `page.tsx` that re-exports from `src\features\governance\`, and it counts as owned for the "only owned paths changed" check. It is not yet in `owned_paths` in `final-graph.json`, so the graph must add it for the collision check to see it.

Inside the API module, by convention:
- `proposals/` (records, rule preview, filing, notice submission, meeting and questions, the A14 request builders and effectors);
- `snapshot/`;
- `ballots/` (cast, signature verification, cosign, supersession, quarantine, the fail-closed delegable guard);
- `domain/` (`tally.ts`, `thresholds.ts`, `weights.ts`, `result-hash.ts`, pure and property-tested);
- `execution/` (the effectors by kind and the A48 event mirror);
- `records/` (§8);
- `jobs/` (`governance.lock_snapshot`, `governance.open_voting`, `governance.cosign_lapse`, `governance.close_voting`, `governance.halt_invariant`, `governance.minutes_clock`);
- `events.ts`, `errors.ts`, `index.ts` (the only import surface for other modules).

The SM-10 table, the `delegable.ts` rows and the seeded ruleset rows are A03 and `packages/domain` data, applied by their owners as listed in §10.

## Design references

- **Halt banner, `design:ProofBand`.** `<workspace>\design\RealX App.dc.html` lines 274-278 (`tab.proof` on the property page): a dark band (`#17191C` background, `#F5F2EC` text, radius 12, padding `18px 20px`) with a Newsreader 22 title, a Geist Mono 12 subline in `#B9B4AA`, and a pill on the right. GOV-02's and SPN-13's "Resale halted for <trust> until the meeting closes" and ADM-23's halt banner reuse this shape exactly (design analysis §2 item 16 and §5 "Governance (P2)"). The pill reads "HALTED" in brick `#9A4524` on its tint instead of the green MATCHED.
- **ThresholdMeter.** Built from the design's RaiseBar:
  - line 188: 8px track `#ECE8E0`, fill `#1F5E4B`, 2px `#17191C` marker at the soft cap 75 %, Geist Mono 12 labels `#45484D`;
  - line 584: the 14px variant on `spOffer`.

  A50 specifies the block in `blocks.md`. The marker sits at the deed threshold (7,500 or 5,000 bp), and the label names the basis.
- **The G10 line on the sell preview.** Line 910 has `sellChecks`, including "Trading halt (G10) · No meeting in progress · Open". During voting, that row must turn to the halted verdict for the vehicle through A48's `haltNotice()` (A91 and A92 render it; this node's integration test checks the string).
- **Approvals.** `design:adApprovals` (lines 655-665, and APR-2286 at line 781) is the approval standard for "Publish notice", "Certify" and "Execute outcome": the copy "you are not the maker, so you can approve", Approve in green `#1F5E4B`, Reject with a `#A3312A` border, and the "Inbox clear." empty state. The console side goes through A53's `KeyConfirmModal`; the trustee side is A105's PTR-07 on the light shell.
- **Undesigned screens.**
  - `design-ext:GOV-01` and `design-ext:GOV-02` from [[A50 - Design extension A- investor and sponsor screens not in the prototype]] (the TaskRow groups, the KpiTile voting power with the record-date caption, the FilterChips, the ThresholdMeter, the passkey DropBox, and the voted, not-eligible and agent-voted states, with Bay Road Apartment `BAY01` of Harbour & Co and Alex Morgan's 120 tokens as the seed persona);
  - `design-ext:SPN-13` from the same file (KpiGrid, ThresholdMeter, halt banner, "Issue notice" ending "Sent to RealX for approval", "Certify result with trustee" disabled until close, the deadlock-fallback Banner for a failed liquidity vote);
  - `design-ext:ADM-23` from [[A51 - Design extension B- operator console screens not in the prototype]] (the "Markets (P2)" group with its brick "2027" pill, the proposal builder card with the rule preview, the ThresholdMeter, the halt banner, the execution Timeline, the PreviewBanner, and the states default, voting open, certified and empty).
- **Tokens.** Only A05's `tokens.json`: cream `#F5F2EC`, ink `#17191C`, green `#1F5E4B`, brick `#9A4524`, red `#A3312A`; Newsreader for the trust names and the resolution heading, Geist for body text, Geist Mono for ids, dates, units and percentages. The product map's coral BrickX look and its PAD01 mock are requirements, not design.

## Requirements covered

- **GOV-01 Votes and meetings.** Done when open, upcoming and result groups come from `GET /v1/governance/votes` for exactly the trusts held now or at a record date, with the design's pills and empty state, hidden in P1, and A50's fidelity comparison passes.
- **GOV-02 Vote.** Done when all of these hold:
  - the notice, meeting row, documents, snapshot voting power with its caption, For / Against / Abstain, ThresholdMeter, halt banner, questions and passkey signing exist;
  - every state in §7 is reachable with the seed, including both co-holder states, excluded, failed with cooldown and quorum not met;
  - a vote can be changed until close and not after;
  - the product-map "Cast vote" action (G10, chain `both`: signed off-chain, result hash anchorable) works identically from the UI, the API and MCP, subject to the counsel row.
- **SPN-13 Meetings and resolutions.** Done when a sponsor can draft, file and issue a notice that ends in a RealX approval with an `APR-` id, see the live tally (or turnout only, per the counsel row) and the halt banner, see the failed and quorum-not-met states with the deed fallback and cooldown, and submit certification to the trustee. The proposer can never be the certifier.
- **ADM-23 Meetings and ballots.** Done when the builder with its rule preview, the calendar with meetings, the ballot monitor, the execution tracker, the failed and quorum-not-met states and the records tab work, and "Publish notice and open vote", "Halt trading for the meeting" (automatic, linked), "Certify result" (maker here, checker in A105's PTR-07) and "Execute outcome" follow the product map's maker-checker rows through A53's pattern.
- **J08 Five-year liquidity vote and exit.** Done when:
  - steps 1-2 and 4-6 run at service level and through the screens with the seeded BAY01 scenario, step 1 including `operating -> exit_proposed` (after the A48 amendment) and step 6 decided by the TP-RE in PTR-07;
  - step 3 (meeting halt) and step 7 (resume) run through A48 and A91 with this node's events;
  - steps 8-10 (sale, waterfall, burn) are started by `governance.outcome.execute` and mirrored as execution steps;
  - both failure paths (fail, quorum miss → cooldown, exit track closed and halt lifted; silence never counts) are tested.

  A94 drives J08 end to end in Playwright.
- **G10 Trading halt.** Done when voting cannot open without a confirmed meeting halt, a board order during voting is refused, the halt cannot be lifted before certification or cooldown and can be lifted after either, and the halt-invariant job raises a Sev1 on any violation.
- **G13 Maker-checker.** Done when notice submit, publish, certify, execute, extend, ruleset change, ballot release and exclude, and record publish are `4E` requests with human checkers, and the proposer identity set (every principal and agent of each proposing person) is refused as certifier by name.
- **G22 Agent authority.** Done when:
  - every operation in §6 works under a scoped token with parity to the screens;
  - agent ballots and cosigns are refused while the counsel row is `"counsel"` or missing, and admitted only under a governance-scoped, vehicle-allowed, live mandate when it is `true`;
  - agents are never checkers or certifiers;
  - every agent action is audited on behalf of its human.
- **M17 Governance and voting.** Done when proposals, notices, record dates, meetings, ballots, certification with the trustee and execution tracking exist as this API. The launch-time reduced version is covered only once §8 is placed in a P1 node or a founder decision says P1 launches without it.
- **SM-10 Governance proposal.** Done when:
  - the 13 states are reachable only through A03's 13 transitions plus the settled spec points;
  - every other pair is refused with `SM10_ILLEGAL_TRANSITION`;
  - the invariants hold: publishing blocked when silence defaults to the manager, notice period met at publication, thresholds from the deed, interested parties excluded, proposer cannot certify, voting implies halted.

## Gauntlet loop for this node

- **Specify.** The frozen spec consists of:
  - this note, after spec points (a) to (d) are decided;
  - A03's SM-10 table, guard and SoD rule 7;
  - the `delegable.ts` rows for `votes.cast` and `governance.ballots.cosign`;
  - the seeded ruleset rows;
  - A14's policy rows for the eight action codes and the record publish, and the certify `checker_view`;
  - A48's internal events contract, `haltState`, `haltNotice()`, the amended meeting criterion and the P2 exit propose and withdraw path;
  - A18's `register_version`, certification and comparison reads, and A40's snapshot canonicaliser;
  - A98's `TRUSTEE_OF_VEHICLE` resolver and A105's PTR-07 contract;
  - A33's associate read and A34's valuation read;
  - A12's mandate-signature and confirmation contracts;
  - A50's and A51's `screens.json` rows for the four screens.

  Every upstream amendment in §10 must be applied first; one that is missing blocks Specify. The node QA records `specVersion` and those hashes in the manifest. It then writes the hidden tests to `<workspace>\gauntlet\hidden\A93\`: Vitest and fast-check suites over the service layer against the Docker Postgres from A01, with fakes for A48 (a halt that can be confirmed late or never), A14 (recording requests and running effectors on demand, at chosen clock times), A18 (versions of every source and status, with and without recon breaks), A98 (checker sets, including empty) and A34 (valuations present or missing), the injectable clock, and Playwright contract checks for the four screens.
- **Build (Grok 4.7 xhigh, split by the planner).** Four parallel sub-agents on disjoint folders, then integration:
  1. `domain/` and `snapshot/`: the tally, thresholds, weights and result hash with property tests; the snapshot lock through A18, A40, A33 and A34.
  2. `proposals/`, `execution/`, `records/` and `jobs/`: the SM-10 executor, filing and publish checks, notice submission, meeting and questions, A14 request builders and effectors, the A48 event mirror and the six jobs.
  3. `ballots/` and the HTTP layer: signature verification for passkey and mandate key; cosign; supersession; quarantine and the display rule; the fail-closed delegable guard; zod schemas; routes with `x-realx-scope` and `Idempotency-Key`; OpenAPI; webhooks registration; the error catalogue; fixtures (Alex Morgan and 185 other BAY01 holders from A06, a joint "both" account, an SMSF, a restricted holder, an RE-associated holder, a two-sub-fund umbrella, SP-OWN and SP-IR of HCO, LEGL, CCO, ASTM, CFO, MLRO, two TP-RE persons, and one token per relevant prefix).
  4. The four screens in `apps/web` and `apps/console` from A50's and A51's extension files, with the screen contract tests.

  Integration covers the module suite (state coverage matrix, tally property, PII and choice leak scan), the J08 service-level run to `executing` against A48's wind-down fake, `npm run verify` and `app/evidence/A93.json`.
- **Challenge (node QA, Opus 5.5 high).** The node QA runs the hidden tests, then works by hand against the running two-process stack:
  - every action by every role and by each role's agent token;
  - the design's BAY01 vote driven from notice to certification on screen, the TP-RE deciding in PTR-07;
  - a proposal filed 22 days before `voting_opens_at` whose CCO decision comes 2 days later (expect `GOV_NOTICE_PERIOD_TOO_SHORT` at the effector), and one decided with exactly 21 days left (expect success);
  - a liquidity vote closing one day inside and one day outside the anniversary window;
  - threshold boundaries at exactly 7,500 bp and 7,499 bp, and at exactly 5,000 bp under `gt` and `gte`;
  - a quorum miss with an overwhelming For, then an MKTO lift after cooldown entry (expect success);
  - a halt confirmed late, and a `halt_starts_at` after `voting_opens_at` (expect `GOV_HALT_WINDOW_INVALID`);
  - a board sell attempt during voting;
  - a lift attempt before certification;
  - a restricted holder's ballot diffed against an ordinary one;
  - a joint account cast, cosign, change while pending, and lapse at close;
  - an RE-associated holder casting on a sale resolution;
  - a snapshot attempt with only an `internal`-source version for the record date (expect no lock);
  - the `Date.now()` grep and the OpenAPI diff;
  - screenshot comparison of the four screens against A50's and A51's frames at 1440 px and 390 px, in light and dark themes;
  - a check that only the owned paths plus the agreed data diffs changed.

  It writes `app/evidence/A93.qa.md`.
- **Gate.** A pass unlocks A94 and publishes the hidden tests into `app/`. There are at most three fix loops, and each loop re-runs the full hidden set and the failing challenges.
- **Escalation trigger.** Any of these escalates:
  - the same assumption failing three times;
  - an upstream amendment in §10 that is missing or differs from what this note assumes, or a fix that would need a further change to A03's SM-10 table or `delegable.ts`, A14's policy rows, A48's halt contract or A18's version semantics (a spec defect, reopened upstream);
  - a finding that a ballot counted with a weight other than its snapshot weight, that silence counted for anyone, that an excluded party's ballot counted, that a proposer or any agent certified, that notice was shorter than the minimum at publication, or that voting opened without a halt (Critical, stops the milestone);
  - a refusal whose correctness depends on a counsel answer not yet given (recorded under HG-LEGAL, and the user is asked).

## Node-specific challenges (node QA)

- **Buy the vote after the record date (compliance).** Alex Morgan holds 120 BAY01 units at the 15 Sep record date and buys 20 more on 20 Sep through a seeded transfer. A second persona holds nothing at the record date and buys 500 units on 16 Sep. The node QA casts for both, through the UI and the API. Expected:
  - Alex's ballot weighs exactly 120 and GOV-02 shows the record-date caption;
  - the second persona gets `GOV_NOT_ELIGIBLE` and the not-eligible state;
  - a request body that names `weight_units: 620` gets `GOV_WEIGHT_NOT_ACCEPTED`;
  - a seller who sold all their units after the record date can still vote their 60.

  Any weight read from the live cap table, or from an `internal`-source register version alone, fails the node.
- **Short notice by delay (compliance).** A proposal is filed with `voting_opens_at` exactly 22 days ahead. The CCO decides 2 days later on the injectable clock. Expected: the effector refuses with `GOV_NOTICE_PERIOD_TOO_SHORT`, the proposal stays `filed`, no document is published and no notice is sent. The same run with the decision 1 day later succeeds. An extension whose new `closes_at` is 6 days after its effector time is refused. Any holder receiving a notice with less than `notice_min_days` to the voting start or the meeting fails the node.
- **Silence for the manager, by the back door (compliance).** The node QA tries four routes:
  - a sponsor draft with `silence_default = manager` imported through `PATCH`;
  - flipping the field to `manager` by direct SQL after filing, before the CCO decides;
  - a proposal whose resolution text reads "undirected votes will be cast by the manager";
  - a job replay that inserts ballots for non-voters.

  Expected:
  - the first two are refused at precheck and again at the effector with `GOV_SILENCE_DEFAULTS_TO_MANAGER`, and the payload hash no longer matches after the SQL edit (`APR_CHALLENGE_MISMATCH`);
  - the third is caught by the claims lint as a proxy-to-manager phrase added to A43's blocked list (§10);
  - the fourth is impossible because no insert path exists.

  Any unsigned ballot fails the node.
- **The interested party votes (compliance).** Harbour & Co's RE entity holds 400 BAY01 units through an associated company. On the sale resolution, that account casts For. Expected: `GOV_HOLDER_EXCLUDED`, the excluded state on GOV-02, `excluded_weight = 400` in the tally, and quorum and `total_units` denominators without the 400 under `excluded_in_denominators = exclude` (and with them under `include`). Named tally vectors: a result that passes only if the excluded weight is counted must fail; one that fails only because of the excluded weight in the denominator must pass under `exclude`. The rule preview names the excluded party.
- **The proposer certifies (G13, agent misuse).** HCO's SP-OWN files the BAY01 sale proposal. Then, in turn:
  - the same person, logged in as a TP-RE fixture that shares their `person_id`, tries to certify;
  - their agent token tries to certify;
  - an SP-IR of HCO who edited the draft tries to certify;
  - a TP-RE's own agent token tries to certify;
  - an ASTM who proposed a RealX-origin vote tries to certify.

  Expected: `GOV_PROPOSER_CANNOT_CERTIFY` for the first three (the first because the set is by `person_id`), `APR_CHECKER_MUST_BE_HUMAN` for the fourth, and A11's rule 7 refusal for the fifth. Only an unrelated TP-RE person or scrutineer, deciding in person in PTR-07 with an assertion over `result_sha256`, succeeds. Then a trustee-origin proposal filed by the vehicle's only TP-RE person, with no scrutineer: publishing is refused with `GOV_CERTIFIER_SET_EMPTY`.
- **Sponsor parity on SPN-13 (G22, parity).** HCO's SP-OWN clicks "Issue notice" and, separately, calls `issue_notice` with an SP-OWN token: both return an `APR-` id and a LEGL queue item. After the vote passes, the proposer SP-OWN sees "Certify result with trustee" enabled and gets an `APR-` id from both the button and the API; a non-SP-OWN sponsor user sees neither. Any case where the screen and the API disagree fails the node.
- **Voting without a halt, and lifting mid-vote (G10).**
  - The A48 fake never confirms the halt: `voting_opens_at` passes, the proposal stays `discussion`, no ballot is accepted (`GOV_HALT_NOT_CONFIRMED`), and a Sev2 is raised.
  - The fake confirms the halt; then an MKTO tries to lift it through A48 mid-vote, and it must be refused with `HALT_CRITERIA_NOT_MET`.
  - After a quorum miss and cooldown entry, the same lift succeeds and the exit track is closed (`exit_proposed -> operating`).
  - The node QA deletes the halt row by SQL: `governance.halt_invariant` must raise a Sev1 within one tick.
  - A resale ask on INV-05 for BAY01 during voting must be refused under G10, and the sell preview row must show the halted verdict.
- **Arithmetic at the edges (compliance).** A fast-check property over random snapshots (1 to 5,000 holders, 1 to 10,000,000 units, 0 to 5 % excluded, one or two sub-funds with `nav_per_unit_cents` from 1 to 10,000,000), random ballot sets and every rule combination compares `tally()` with a reference implementation that uses rational numbers. Named vectors:
  - 7,500 bp exactly: passes under `gte` and fails under `gt`;
  - For = 100 %, Against = 0, turnout 24.99 % of eligible weight: `quorum_failed`;
  - all abstain with quorum met: `failed` under `votes_cast`, never a division error;
  - one holder with 100 % and `quorum_min_holders = 2`: `quorum_failed`;
  - a liquidity vote closing at 23:59 Sydney on the last day of the window versus 00:00 the next day: 5,000 versus 7,500 bp, frozen at filing, not re-read at close;
  - a scheme-wide vote where the For side holds more units but less dollar value: `passed` under `units` and `failed` under `dollar_value`.
- **Agents voting before counsel says so (D04, agent misuse).**
  - With `votes.cast` still `"counsel"`: token issue with the governance cell is refused (`SCOPE_PENDING_COUNSEL`), and a forged request with a read-only token to the ballot route gets `AGNT_NOT_ALLOWED` with reason code `G22_SCOPE_PENDING_COUNSEL` (the route is `requiresHuman` while the cell is `"counsel"`; A12 §5).
  - With the `votes.cast` row deleted from the scratch `delegable.ts` and A12's pending list bypassed by a fake: the route flag no longer applies, and an agent ballot is still refused with `G22_SCOPE_NON_DELEGABLE` (as the code) by this module's guard, the step (5) case of A12 §5.
  - With the row flipped to `true` in a scratch run:
    - a mandate without BAY01 in `allowed_vehicle_ids` gets `G22_VEHICLE_NOT_ALLOWED`;
    - a mandate that expired one minute before `cast_at` gets `MANDATE_EXPIRED`;
    - an unsigned agent ballot gets `428 CONFIRMATION_REQUIRED`;
    - holder A's agent cosigning for holder B's joint ballot where A is the first signer gets `GOV_COSIGN_OTHER_HOLDER`;
    - a replayed mandate signature with a new idempotency key is a signature mismatch;
    - a valid agent ballot produces exactly one notice to the person, none to the agent, and the GOV-02 agent line.
- **Tipping-off through a ballot (compliance, security).** A holder with a confirmed sanctions match votes For at the same instant as an ordinary holder of the same size, under both values of `live_tally_visible`. The node QA diffs, for both holders:
  - status codes, bodies after id substitution and GOV-02 states;
  - notification events and webhooks;
  - the holder-visible tally, the SP-OWN monitor and the LEGL and TP-RE ballot lists;
  - latency.

  The holder-visible tally, the sponsor monitor and the LEGL and TP-RE views must be identical for the two cases; only MLRO and CCO may see `quarantined_weight`. At close the result waits (seeded rule) until the MLRO releases or excludes the ballot through `4E`, and every holder sees the same "Result being certified" state meanwhile.
- **Design fidelity.**
  - With the seed, GOV-01 must show BAY01 open, MRN01 upcoming and the two results with the extension's exact strings and pills.
  - GOV-02 must show the halt banner sentence from `haltNotice()`, 120 tokens at the record date, and the marker at the deed threshold with the basis named.
  - SPN-13 and ADM-23 must match A50's and A51's frames within 1px on font, size, colour, radius and padding, including the failed and quorum-not-met states.
  - The ProofBand-style banner must use `#17191C` and `#F5F2EC` only.

  Any new status string not in this note or the extension files, colour outside `tokens.json` or bare 4xx fails the node.

## Pass criteria

- **Hidden tests.** They pass 100 % in `<workspace>\gauntlet\hidden\A93\` on three consecutive runs with different seeds. This includes the tally property over at least 20,000 random (snapshot, ballots, ruleset, weight basis, exclusion set) tuples with zero disagreement against the rational reference, and at least 5,000 random ballot sequences (cast, change, cosign, lapse, quarantine, release) asserting that each account counts at most once, with its latest counted ballot and its snapshot weight.
- **State coverage.** A generated test:
  - exercises all 13 SM-10 states and every transition in A03's table plus the settled spec points at least once;
  - refuses 100 % of other pairs with `SM10_ILLEGAL_TRANSITION`, and the refused count equals the count derived from the table;
  - refuses every SM-10 route under `PHASE=P1` with `GOV_PHASE_P1_RECORDS_ONLY`.
- **Snapshot and result integrity.** In 100 random runs, recomputing the snapshot from the locked register versions reproduces `snapshot_sha256`, and recomputing the tally from ballot rows reproduces `result_sha256` byte for byte. No ballot's weight differs from its snapshot rows. No snapshot was locked from an `internal`-source version alone.
- **Notice, silence and certification.** Every published proposal had at least `notice_min_days` between `published_at` and both `voting_opens_at` and the meeting start. Every insert into `ballot` over the full run traces to a signed cast (zero unsigned rows). No `vote_excluded` weight entered any decision. For every principal in the fixtures and every agent of each, the certify matrix has zero cells that disagree with §2 and §6. Every `4E` action creates a request with the policy named here, and its effector applies exactly one transition.
- **Halt linkage.** Across the suite:
  - no proposal was ever in `voting` without an open meeting halt on its vehicle;
  - every meeting halt opened by this node was lifted only after `certified` or `cooldown`, and one lift after cooldown succeeded;
  - `governance.halt_invariant` raised exactly one Sev1 per planted violation;
  - A91's refusal of board orders during voting is asserted in the integration test.
- **Agent parity.** Every operation in §6 is present in the OpenAPI document, `packages/sdk` and the MCP catalogue, every mutating route requires `Idempotency-Key`, and A100's parity rows for GOV-01, GOV-02, SPN-13 and ADM-23 (including issue notice, certify maker visibility and cosign) are green (UI, API and MCP) under matching scopes. Every refusal names its scope, cap or counsel state. Every agent-made event carries `on_behalf_of_id` and `token_id`. The agent-ballot matrix passes under both values of the counsel row and with the row missing. The three webhook events are delivered only to holders of the vehicle.
- **Screens.**
  - The four screens render every state listed in §7 with the seed.
  - Screenshot comparison against A50's and A51's frames shows no difference above the fidelity threshold A50 sets (1px on layout, exact on token colours) at 1440 px and 390 px, in both themes.
  - Axe finds no serious or critical issue.
  - No screen imports anything but `packages/sdk` and `packages/ui`.
- **Audit and leaks.** After the full run, A10's chain verifier passes. The count of `governance.*` events equals the transitions, submissions, ballots, cosigns, lapses, dispositions, locks, questions, answers, minutes and evidence records performed. A scan of logs, the outbox, webhooks, alerts and the evidence folder finds no holder name, meeting link or ballot choice outside the holder's own feed and the LEGL, CCO and TP-RE reads.
- **Standing checks.**
  - Line coverage is at least 95 % on `domain/` and at least 90 % on the module.
  - The module boundary lint passes and `npm run verify` is green.
  - Only the owned paths, the SPN-13 page folder and the agreed data diffs changed.
  - `app/evidence/A93.json` and `A93.qa.md` exist and match disk.

## Human gates touched

- **None crossed.**
  - The node runs on A18's registry sandbox, A48's halts and A28's token operations on the local chain.
  - Signing uses A11's software authenticator and A12's mandate keys generated in tests.
  - Time comes from the injectable clock.
  - The `governance.votes` flag stays off and unbound in every launch profile (A21).
  - No real vote, meeting, trustee, licence or document is touched.
- **HG-LEGAL: counsel questions recorded here, not decided here.**
  - Whether a ballot cast or cosigned by a holder's agent under a mandate is the holder's own vote or needs proxy formalities. This is `votes.cast` in `delegable.ts`, raised by A50 and carried here; the default is non-delegable.
  - The quorum, the thresholds and their basis and comparator, the 5-year anniversary window, the notice period (and that it runs from the notice actually given), the extension re-notice period and the cooldown for each vehicle's deed and for the P2 umbrella constitution.
  - Interested parties (s253E): which entities count as the RE's associates, which resolution kinds give the RE or the sponsor an interest other than as members, and whether excluded weight stays in the quorum and threshold denominators.
  - Scheme-wide votes and weighting (s253C, s253F): which resolutions are votes of all members across sub-funds, whether a poll is weighted by units or by dollar value, and which price sets the value at the record date.
  - The members' meeting: whether a meeting is required for each resolution type, which formats the constitution allows, the chair, members' questions and the minute-book rule.
  - Whether frozen, locked or sanctions-restricted units vote, whether the result must wait for a quarantined ballot, and whether a display that includes a quarantined ballot is acceptable.
  - Whether a live tally may be shown to holders and sponsors during voting (seed: no).
  - Who may act as scrutineer.
  - Whether members' requisitions and trustee changes are exempt from the cooldown.
  - The wording of the rule preview and of the anti-deadlock fallback.
  - Whether electronic ballots with passkey signatures, and joint-account cosigns, satisfy the constitution's meeting and voting procedure.

  Counsel's answers change data in A03 and the ruleset rows. This node's hidden tests are then re-run.
- **HG-LEGAL: licence and relief (later).** The `governance.votes` flag binds only to a licence or ASIC relief record covering the registered scheme (A21). Binding it is a human act after counsel, recorded in A94's launch update.
- **HG-PENTEST (later).** Ballot signing, the cosign path, quarantine confidentiality and the certification SoD are in scope of the external penetration test before go-live. This node passes on software evidence and records the gate as open.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
