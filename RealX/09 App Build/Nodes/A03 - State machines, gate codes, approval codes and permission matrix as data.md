---
title: "A03 - State machines, gate codes, approval codes and permission matrix as data"
created: 2026-09-23
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am0]
node: A03
milestone: AM0
lane: "Domain core"
oracle: yes
human_gate: none
---

# A03 - State machines, gate codes, approval codes and permission matrix as data

**Every rule about who may do what, in which state, under which approval code, lives here as pure TypeScript data: the 14 state machines SM-00 to SM-13 (181 catalogue states and 219 catalogue transitions, 183 states and 227 transitions once the three listed extensions are added) behind one engine that refuses anything not in the table, the gate codes G01 to G22, the six approval policy codes, the 51 roles plus five principal kinds, the role x module x action matrix from which both RBAC and API-token scopes are derived, and the ten segregation-of-duties rules.**

**Build status:** not started. **Milestone:** AM0 (Foundations). **Lane:** Domain core. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 8-12 agent-hours (build, challenge and verify loops).

**Depends on:** [[A02 - Domain ids, money and unit math]]
**Unlocks:** [[A04 - Database schema, migrations and row-level security]], [[A12 - API tokens, scopes, agent principals and mandates for every user type (G22)]], [[A14 - Approvals engine and maker-checker (SM-00)]], [[A19 - Rules engine- gates G01-G22, rule packs, threshold monitors, simulator]]

**Oracle rule:** The node QA writes hidden oracle tests from this spec to `<workspace>\gauntlet\hidden\A03\` before the build starts ([[App Gauntlet]] §2). The builder never reads that folder. The oracle scope is exhaustive transition tests over (state, triggerId), forbidden-transition tests, guard boundary vectors, matrix invariants, scope-subset property tests and SoD vectors.

This node is not legal advice. Where a rule below depends on counsel (which declarations a person must make in person, the DAF Act timing behind G21, the retail cooling-off text behind G05) the data carries a `"counsel"` marker and the question is logged under HG-LEGAL rather than decided here.

## Build brief

Why this node exists: the OddX QA trail (BQA-08, BQA-16) found state machines with missing transitions and RBAC that existed only in the UI. RealX puts every such rule in one data package, with tests generated from the data, so the API, the console, the token service and the MCP server all refuse the same things for the same reasons. Nothing in `src/` of this package does I/O, reads the clock or imports anything outside `packages/domain` (A02 supplies ids, money and unit math). The package has **no runtime dependency**: schemas are plain `as const` data (enum arrays and JSON-Schema objects written as data), so A02's `dependencies: {}` stays empty and no zod or other library is needed at runtime.

**0. Frozen fixtures (what the oracle and the builder both read).** Hashes are SHA-256 of the raw bytes on 24 Sep 2026; a different hash at build time is a blocking spec defect, never something the builder reconciles.
- `<workspace>\RealX\06 Evidence\Tools\product_map\data_states.py` `b5da57f9b236cad65ff4be70bac52179e3ee2053d26b92db2afac2f7f01c4e4a`; `data_admin.py` (MAKER_CHECKER, SOD, SCREENS actions, KEYS) `37e2cd561b24cd198d62f627c7c2100a253ecf64446f3602f06699d0525cb54a`; `data_roles.py` `4bc10be99044fe0203d274ac99691e7b0eb50a6c685869b64449cf6287de5d4d`; `data_investor.py` `29f8b09df037ba6edff37a2ef564b9701501361861dd1a57cdf0ba2f32e553ea`; `data_sponsor.py` `c75242b73ff3e5faca8493b6dfeeac86f3d29ec9102d6aeff63b492eb51b9ef8`; `data_core.py` (gate names) `5d8a9d9fae7036c24f6cf0ef2001c8d7c3e7141f7a1c64a256ec5a1107d2a224`.
- A00's charter outputs, which are the id and D04 sources (the session-scratchpad copy of `requirements-lists.json` is never read): `<workspace>\app\docs\charter\ids.json` (52 role ids, 22 gates, 27 modules, 14 machines; its own recorded ID-list hash must match A00's) and `<workspace>\app\docs\charter\agent-access.json` (`nonDelegable`, `counselFlagged`, `principalTypes`, and the frozen action-id vocabulary in `scopeFamilies`). A03 records the SHA-256 of both files as read in `app/evidence/A03.json` `fixtureHashes`.
- The design `<workspace>\design\RealX App.dc.html` `8ef1e73c23a3644aac5935cd41bddfbb1719c546b98e296d16be74b8d4589b03` for the label strings in §2.6.
- The action-source mapping written at the Specify step (see §6): `<workspace>\RealX\09 App Build\Data\fixtures\A03-action-sources.json`.

**1. Package layout (all pure `as const` data).**
- `src/state-machines/`: `engine.ts`, one file per machine `sm-00-privileged-action.ts` to `sm-13-wallet-recovery.ts`, `guards.ts` (guard ids and their pure implementations over `ctx`), `labels.ts` (design projections, §2.6), `extensions.ts` (the three additive entries of §2.5, each with its source), `mermaid.ts` (renders a machine back to Mermaid), `index.ts`.
- `src/codes/`: `gates.ts` (G01-G22), `approval-policies.ts` (4E, 4E+Q, 6E, 6E+Q, SGL+R, AUTO), `vaults.ts` (V1-V7), `maker-checker-baseline.ts` (the 49 rows of `data_admin.py` MAKER_CHECKER verbatim), `maker-checker-resolution.ts` (the normalised table of §4.1), `seeds.ts` (the only file that holds durations, counts and quorums, §4.2), `reason-codes.ts`, `index.ts`.
- `src/permissions/`: `roles.ts` (51), `principals.ts` (five kinds), `modules.ts` (M01-M27), `actions.ts`, `action-sources.ts` (encodes the frozen mapping), `matrix.ts`, `delegable.ts`, `scopes.ts`, `sod.ts`, `a11-view.ts` (kinds to A11 letters, and the `nonDelegable` view), `index.ts`.
- `scripts/domain-export/` (outside `src/`, allowed to write files): `npm run domain:export` writes `app/docs/domain/*.json` and `app/docs/domain/*.mmd` deterministically; verify diff-tests them against the fixtures.
- Exports: A02's `package.json` already maps `"./*": "./src/*/index.ts"`, so the three folders are reachable as `@realx/domain/state-machines`, `@realx/domain/codes` and `@realx/domain/permissions` with no edit to A02's files; A03 does not touch `src/index.ts`. **Needs from A02 (spec change raised, not made here):** the `domain:export` script, a `realx` verify step that runs A03's tests and the export diff, and coverage thresholds for A03's folders, all in A02's root files.

**2. State machines: tables, not code.**

**2.1 Table shape.** Every machine from `data_states.py` becomes `{ id, name, initial, states[], terminal[], transitions[] }`. A transition is `{ triggerId, from, to, label, guards[], actors[], humanOnly, policyRowRef?, gates[], phase, emits }`:
- **`triggerId` is canonical and is `"<from>__<to>"`** (two underscores). It is unique within a machine because no (from, to) pair appears twice in the catalogue or the extensions. `label` is the catalogue's text after ` : ` (or `""`), kept for display and for the Mermaid round trip only; the engine, the audit names and the oracle never key on it.
- Why the label cannot be the key: 66 of the 219 catalogue transitions have no label, and 7 (from, label) pairs lead to two targets: SM-01 `manual_review` "KYCA" (to `approved` and `rejected`); SM-08 `executing` "" (to `executed`, `partially_failed`); SM-08 `partially_failed` "" (to `retrying`, `manual_payout`); SM-09 `open` "" (to `partially_filled`, `filled`); SM-09 `settling` "" (to `settled`, `settlement_failed`); SM-10 `tallying` "" (to `quorum_failed`, `failed`); SM-11 `post_filing_monitoring` "" (to `exit_relationship`, `case_closed`).
- `emits` is the audit action name `sm.<machineId>.<triggerId>`, for example `sm.SM-05.allocated__settled`.
- `initial` is the target of the `[*] -->` line, never a transition row; `terminal` is every state with no outgoing transition after extensions (the oracle derives the same set).
- Catalogue counts, states / transitions: SM-00 14 / 14; SM-01 15 / 21; SM-02 9 / 11; SM-03 20 / 25; SM-04 14 / 16; SM-05 13 / 15; SM-06 12 / 14; SM-07 7 / 12; SM-08 15 / 18; SM-09 13 / 16; SM-10 13 / 13; SM-11 13 / 16; SM-12 9 / 11; SM-13 14 / 17; total 181 / 219 (233 Mermaid edges with the 14 initial entries). After extensions: SM-00 15 / 19, SM-04 15 / 19, total **183 states and 227 transitions**.

**2.2 Engine.** `transition(machineId, from, to, ctx)` returns `{ ok: true, next, triggerId, emits }` or `{ ok: false, reason, detail? }`. `fire(machineId, from, triggerId, ctx)` is the same call addressed by trigger id. It never throws, never mutates its arguments and never reads anything outside its arguments. Refusal precedence is fixed, first match wins:
1. `SM_UNKNOWN_MACHINE`;
2. `SM_UNKNOWN_STATE` (`from` or `to` is not a state of the machine);
3. `SM_UNKNOWN_TRIGGER` (`fire` only: the trigger id is not a transition of the machine);
4. `SM_TERMINAL` (`from` is terminal, whatever the target);
5. `SM_FORBIDDEN` (the pair is not in the table, including every self-pair; for `fire`, the trigger id's `from` is not the current state);
6. `SM_PHASE_OFF` (`ctx.phase` is earlier than the transition's phase);
7. `SM_ACTOR_NOT_ALLOWED` (actor rule below);
8. `SM_GUARD_FAILED` with `detail` naming the first failing guard in binding order (and `missing:<field>` when a fact or param is absent or of the wrong type: guards fail closed).

`can(machineId, from, to, ctx)` and `next(machineId, from)` are exposed for the UI so screens show only reachable actions.

**2.3 ctx shape.**
```
ctx = {
  now: string                      // ISO-8601 UTC instant supplied by the caller; the package never reads a clock
  phase: 'P1' | 'P2' | 'P3'
  actor: { principalId: string, identityId: string,   // identityId = the human for AGNT and AGTU, else principalId
           kind: 'HUMAN' | 'PARTNER' | 'AGNT' | 'AGTU' | 'SVCA', roleIds: string[] }
  params: Record<string, number>   // keys listed in seeds.ts; values come from A14 policy rows and A19 rule packs
  facts: Record<string, unknown>   // the per-guard fields in §2.4
}
```
Actor rule: the actor is allowed when its `roleIds` intersect the transition's `actors`, or `actors` contains `SVCA` and `kind` is `SVCA`, or `actors` contains `self` and `actor.identityId === facts.subjectIdentityId`; for SM-00 the actor tokens `row.maker`, `row.checker` and `row.vetoer` resolve through the §4.1 row named by `facts.policyRowId`. An AGNT or AGTU actor resolves to its human's roles but is refused on every transition with `humanOnly: true` (checker decisions, vetoes, SGL+R acts, custody-bound steps). Whether a person's agent may take a delegable action at all is decided upstream by `scopes.ts` (A12) and G22 (A19), not by the engine. Decisions that another person records (signoffs) are separate matrix actions; a transition that depends on them checks the recorded signoffs as facts and is fired by SVCA or by the last signer.

**2.4 Guard table (normative).** Each guard is a pure function of `ctx`. Durations compare instants as integer milliseconds (`Date.parse` of the given strings, never `Date.now`); a duration of N hours from instant T passes when `now >= T + N h`. Business-day deadlines are computed by the caller (A13 owns calendars) and passed as instants. The happy ctx for a transition is the ctx in which every bound guard's facts take the pass value below, `phase` is the transition's phase and the actor is SVCA or the first listed role; the acceptance sweep uses it.

| Guard | Facts and params (types) | Passes when | Bound to (machine: triggerId) |
|---|---|---|---|
| `precheck_passed` | `simulationOk`, `canTransferOk`: boolean | both true | SM-00 `submitted__prechecked` |
| `reject_reason_valid` | `rejectReason`: `'precheck_failed' \| 'checker_rejected' \| null` | non-null; `'checker_rejected'` only when the actor resolves to `row.checker` | SM-00 `prechecked__rejected` |
| `checkers_complete` | `policyRowId`: string; `checkerApprovals`: `{identityId, roleId}[]`; `makerIdentityId`: string; `sodViolations`: string[] | each checker role set of the row is covered by a distinct identity, none equal to the maker identity, and `sodViolations` is empty; a row with zero checker sets (SGL+R, AUTO) passes | SM-00 `prechecked__approved` |
| `timelocked` / `not_timelocked` | `timelockHours`: integer ≥ 0 (from the row's timelock key) | `> 0` / `=== 0` | SM-00 `approved__queued` / `approved__queued_for_custody` |
| `ttl_elapsed` | `ttlStartAt`: instant (submission); param `approval.ttlHours` | `now >= ttlStartAt + ttlHours` | SM-00 `approved__expired`, `prechecked__expired` |
| `timelock_elapsed_or_waived` | `queuedAt`: instant; `timelockHours`: integer; `courtOrderId`: string \| null; `waiverSignoffs`: `{identityId, roleId}[]` | `now >= queuedAt + timelockHours`, or `courtOrderId` non-null and signoffs by CCO and CEO from two distinct identities | SM-00 `queued__queued_for_custody` |
| `veto_recorded` | `veto`: `{identityId, roleId} \| null`; `makerIdentityId` | non-null, role in the row's `vetoerRoles`, identity not the maker | SM-00 `queued__cancelled` |
| `custody_quorum_met` | `custodySignatures`: integer; `custodyQuorum`: integer (vault seed); `approvalIdBound`: boolean; `initiatorCosigned`: boolean | signatures ≥ quorum, `approvalIdBound`, not `initiatorCosigned` | SM-00 `queued_for_custody__signed`; SM-13 `timelock__executing` |
| `ack_received` | `chainConfirmed`, `registryAck`: boolean | either true | SM-00 `broadcast__confirmed` |
| `broadcast_failed` | `failure`: `'revert' \| 'registry_reject' \| null` | non-null | SM-00 `broadcast__failed` |
| `recon_clean` | `reconRunAt`: instant \| null; `reconBreaks`: integer | run recorded and `reconBreaks === 0` | SM-00 `confirmed__reconciled` |
| `attestation_expired` | `attestationExpiresAt`: instant | `now >= attestationExpiresAt` | SM-00 `queued_for_custody__expired` |
| `provider_pass_low_risk` / `provider_refer` / `fraud_signal` | `providerResult`: `'pass' \| 'refer' \| 'hit' \| 'fraud'`; `riskRating`: `'low' \| 'medium' \| 'high'` | pass and low / refer or hit, or pass and not low / fraud | SM-01 `provider_checks__auto_approved` / `__manual_review` / `__auto_rejected` |
| `rfi_expired` | `rfiSentAt`: instant; param `onboarding.rfiExpiryDays` | `now >= rfiSentAt + days x 24 h` | SM-01 `rfi_pending__expired` |
| `pep_or_high_risk` / `not_pep_or_high_risk` | `pep`: boolean; `riskRating` | `pep` or high / negation | SM-01 `manual_review__edd_review` / `manual_review__approved` |
| `signoffs(sets, ordered)` | `signoffs`: `{identityId, roleId, at}[]` | for each role set in the argument, a signoff by a distinct identity holding a role in that set; when `ordered`, each set's signoff is not earlier than the previous set's | SM-01 `edd_review__approved` `[[MLRO],[CCO,CEO]]` ordered; SM-01 `suspended__whitelisted` `[[MLRO],[CCO]]` ordered; SM-03 `ic_review__ic_approved` `[[CFO,CCO,CEO],[CFO,CCO,CEO]]` unordered, distinct roles; SM-04 `compliance_review__approved` `[[CCO],[LEGL],[TP-RE]]` unordered; SM-07 `frozen__active` `[[MLRO],[CCO]]` ordered; SM-08 `approved__funding` `[[TRSY],[CFO]]` ordered; SM-09 `settlement_failed__busted` `[[MKTO],[CCO]]` ordered; SM-11 `triage__closed_false_positive` `[[KYCA],[MLRO]]` ordered; SM-12 `redress_proposed__response_issued` `[[CMPH],[CCO]]` ordered; SM-13 `compliance_approval__checker_approval` `[[CCO]]`; SM-13 `checker_approval__old_wallet_frozen` `[[CFO],[TP-RE]]` unordered |
| `qa_sample_cleared` | `qaOutcome`: `'not_sampled' \| 'passed' \| 'failed' \| 'pending'` | not_sampled or passed | SM-01 `auto_approved__approved` |
| `identity_registered` | `identityRegistered`: boolean | true | SM-01 `approved__whitelisted` |
| `review_due` | `nextReviewAt`: instant | `now >= nextReviewAt` | SM-01 `whitelisted__refresh_due` |
| `refresh_completed` / `refresh_overdue` | `refreshCompletedAt`: instant \| null; `refreshDeadlineAt`: instant | completed non-null / `now >= deadline` and not completed | SM-01 `refresh_due__whitelisted` / `refresh_due__suspended` |
| `evidence_complete` / `evidence_incomplete` | `evidenceComplete`: boolean | true / false | SM-02 `under_review__valid`; SM-13 `legal_review__compliance_approval` / `legal_review__rejected` |
| `wholesale_path_cleared` | `wholesalePath`: `'certificate' \| 'offer_amount'`; `approvalRequestState`: SM-00 state | certificate, or offer_amount with `approvalRequestState === 'closed'` (row 19b) | SM-02 `under_review__valid` |
| `certificate_expiring_window` | `expiresAt`: instant; param `wholesale.certificateExpiringDays` | `expiresAt - days <= now < expiresAt` | SM-02 `valid__expiring` |
| `renewal_accepted` | `renewalAccepted`: boolean | true | SM-02 `expiring__valid` |
| `certificate_expired` | `expiresAt`: instant (A32 sets it from param `wholesale.certificateValidityMonths`) | `now >= expiresAt` | SM-02 `expiring__expired` |
| `screening_passed` / `screening_failed` | `screeningPassed`: boolean | true / false | SM-03 `screening__due_diligence` / `screening__rejected` |
| `linked_approval_closed` | `approvalRequestState`: SM-00 state | `=== 'closed'` | SM-03 `title_verification__title_verified` (row 5); SM-04 `finalising__minted` (row 9), `failed__refunding` (row 10), `live__reconfirming` (row 8); SM-05 `refund_pending__refunded` (row 12); SM-06 `deploy_requested__deployed` (row 21), `paused__active` (row 26), `active__upgrading` (row 29; the timelock is served in SM-00 `queued`), `active__migrating` and `active__redeeming` (no catalogue row, see escalations); SM-07 `active__locked` (row 32) |
| `linked_approval_approved` | `approvalRequestState` | one of approved, queued, queued_for_custody, signed, broadcast, confirmed, reconciled, closed | SM-08 `in_review__approved` (row 35) |
| `title_clear` / `title_defect` | `titleDefect`: boolean | false / true | SM-03 `title_verification__title_verified` / `title_verification__rejected` |
| `vehicle_linked` | `vehicleId`: string \| null | non-null | SM-03 `title_verified__spv_formed` |
| `offering_state_in(list)` | `offeringState`: SM-04 state | in the list | SM-03 `spv_formed__listed` [live]; `listed__funded` [finalising, minted]; `listed__withdrawn` [failed, refunding, refunded, cancelled]; `funded__operating` [minted] |
| `distress_signal` / `distress_cleared` | `arrearsBreach`, `ratesOverdue`: boolean (A19 thresholds decide them) | either true / both false | SM-03 `operating__distressed` / `distressed__operating` |
| `court_order_recorded` / `court_order_discharged` / `court_directed_sale` | `courtOrderId`: string \| null; `courtOrderDischargedAt`: instant \| null; `courtDirectedSale`: boolean | non-null / non-null / true | SM-03 `distressed__legal_hold` / `legal_hold__operating` / `legal_hold__exit_approved` |
| `exit_resolution_in(list)` | `exitResolutionState`: `'filed' \| 'failed' \| 'quorum_failed' \| 'certified'` (from A93 in P2, or a recorded trustee resolution in P1) | in the list | SM-03 `operating__exit_proposed` [filed]; `exit_proposed__operating` [failed, quorum_failed]; `exit_proposed__exit_approved` [certified] |
| `waterfall_paid` | `waterfallPaid`: boolean | true | SM-03 `sold__wound_down` |
| `sponsor_owner_approved` | `sponsorOwnerApproval`: `{identityId, roleId} \| null`; `preparerIdentityId` | non-null, role SP-OWN, identity not the preparer | SM-04 `draft__compliance_review`; SM-08 `calculated__in_review` |
| `start_reached` | `startAt`: instant | `now >= startAt` | SM-04 `scheduled__live` |
| `soft_cap_reached` | `raisedCents`, `softCapCents`: integer | raised ≥ soft cap | SM-04 `live__soft_cap_reached`, `closed__finalising` |
| `end_or_hard_cap` | `endAt`; `raisedCents`, `hardCapCents` | `now >= endAt` or raised ≥ hard cap | SM-04 `soft_cap_reached__closed` |
| `end_passed_soft_cap_missed` | `endAt`; `raisedCents`, `softCapCents` | `now >= endAt` and raised < soft cap | SM-04 `live__closed` |
| `allocation_locked` | `allocationLockedAt`: instant \| null | non-null | SM-04 `closed__finalising`; SM-05 `funded__allocated` |
| `soft_cap_missed_or_reverted` | `raisedCents`, `softCapCents`; `mintReverted`: boolean | raised < soft cap or `mintReverted` | SM-04 `closed__failed` |
| `refunds_complete` | `refundsOutstanding`: integer | `=== 0` | SM-04 `refunding__refunded` |
| `reconfirm_window_closed` | `reconfirmDeadlineAt`: instant (caller computes from param `offering.reconfirmBusinessDays`) | `now >= reconfirmDeadlineAt` | SM-04 `reconfirming__live` |
| `offer_withdrawn` | `offerWithdrawn`: boolean | true | SM-04 `reconfirming__refunding` |
| `eligibility_passed` / `eligibility_failed` | `eligibilityFailures`: gate id[] | empty / non-empty | SM-05 `eligibility_check__awaiting_funds` and `__reflection_period` / `eligibility_check__rejected` |
| `retail_regime_p2` / `not_retail_regime_p2` | `ctx.phase`; `offerRegime`: `'wholesale' \| 'retail'` | phase P2 or P3 and regime retail / negation | SM-05 `eligibility_check__reflection_period` / `eligibility_check__awaiting_funds` (so a retail P2 order can never skip the reflection period) |
| `reflection_open` / `reflection_ended` | `reflectionEndsAt`: instant | `now < ` / `now >= ` | SM-05 `reflection_period__revoked` / `reflection_period__awaiting_funds` |
| `payment_matched` | `matchedCents`, `orderTotalCents`: integer | equal | SM-05 `awaiting_funds__funded` |
| `funds_not_received` | `fundsDueAt`: instant; `matchedCents`, `orderTotalCents` | `now >= fundsDueAt` and matched < total | SM-05 `awaiting_funds__expired` |
| `units_allocated` | `allocatedUnits`: integer | `> 0` | SM-05 `funded__allocated` |
| `offer_failed_or_zero_allocation` | `offeringState`; `allocationLockedAt`; `allocatedUnits` | offering in failed, refunding or refunded, or allocation locked with 0 units | SM-05 `funded__refund_pending` |
| `register_entry_exists` | `registerEntryUnits`, `allocatedUnits`: integer | equal and `> 0` | SM-05 `allocated__settled` |
| `token_delivery_exists` | `tokenDeliveredUnits`, `allocatedUnits`: integer | equal and `> 0` | SM-05 `allocated__settled` |
| `partial_allocation` | `allocatedUnits`, `orderedUnits`: integer | `0 < allocated < ordered` | SM-05 `allocated__refund_pending` |
| `already_paid` | `matchedCents` | `> 0` | SM-05 `revoked__refund_pending` |
| `tests_and_audit_attached` | `testReportHash`, `auditReportHash`: string \| null | both non-null | SM-06 `sandbox_deployed__deploy_requested` |
| `modules_configured` | `registryConfigured`, `complianceConfigured`, `agentsAppointed`: boolean | all true | SM-06 `deployed__configured` |
| `supply_positive` / `supply_zero` | `totalSupplyUnits`: integer | `> 0` / `=== 0` | SM-06 `configured__issuing` / `redeeming__retired` |
| `transfers_enabled` / `upgrade_verified` / `holders_reissued` | `transfersEnabled`, `implementationVerified`: boolean; `holdersPendingReissue`: integer | true / true / `=== 0` | SM-06 `issuing__active` / `upgrading__active` / `migrating__retired` |
| `balance_positive` / `balance_zero` | `balanceUnits`: integer | `> 0` / `=== 0` | SM-07 `eligible_empty__active` / `active__eligible_empty` |
| `lock_in_future` / `lock_in_passed` | `lockInUntil`: instant | `now < ` / `now >= ` | SM-07 `active__locked` / `locked__active` |
| `partial_freeze_recorded` / `frozen_units_zero` | `frozenUnits`, `balanceUnits`: integer | `0 < frozen < balance` / `frozen === 0` | SM-07 `active__partially_frozen` / `partially_frozen__active` |
| `freeze_authorised` | `policyVariant`: `'SGL+R' \| '4E+Q'`; `approvalRequestState` | SGL+R (the actor rule already requires a protective act cell holder) or 4E+Q with the request `closed` | SM-07 `active__frozen`, `partially_frozen__frozen` |
| `recovery_case_executing` | `recoveryCaseState`: SM-13 state | executing or confirmed | SM-07 `frozen__recovered_out`, `active__recovered_out` |
| `identity_deleted` | `identityDeleted`: boolean | true | SM-07 `eligible_empty__deregistered` |
| `income_locked` / `snapshot_locked` | `incomeLockedAt`, `snapshotLockedAt`: instant \| null | non-null | SM-08 `draft__calculating` (both); SM-10 `discussion__voting` (`snapshot_locked`) |
| `calculation_complete` / `exceptions_handled` | `calculationComplete`: boolean; `openExceptions`: integer | true / `=== 0` | SM-08 `calculating__calculated` / `calculated__in_review` |
| `funding_received` | `fundedCents`, `approvedTotalCents`: integer | equal | SM-08 `funding__funded` |
| `file_hash_matches` | `paymentFileHash`, `approvedFileHash`: string \| null | both non-null and equal | SM-08 `funded__executing` |
| `all_payments_sent` / `some_payments_failed` | `paymentsFailed`, `paymentsPending`: integer | both 0 / failed > 0 and pending 0 | SM-08 `executing__executed`, `retrying__executed`, `manual_payout__executed` / `executing__partially_failed` |
| `paid_equals_approved` / `paid_not_equal_approved` | `paidCents`, `approvedTotalCents`: integer | equal / not equal (no tolerance) | SM-08 `reconciling__closed` / `reconciling__partially_failed` |
| `resale_checks_passed` / `resale_checks_failed` | `resaleChecks`: `{band, lockIn, cap, eligibility}` booleans | all true / any false | SM-09 `validating__open` / `validating__rejected` |
| `halt_active` / `halt_inactive` | `instrumentHalted`: boolean | true / false | SM-09 `open__suspended` / `validating__open`, `suspended__open` |
| `halt_too_long` | `haltedAt`: instant; param `resale.maxHaltHours` (no seed; the rule pack must supply it, else the guard fails closed) | `now >= haltedAt + hours` | SM-09 `suspended__cancelled` |
| `partial_fill` / `fully_filled` | `filledUnits`, `orderUnits`: integer | `0 < filled < order` / equal | SM-09 `open__partially_filled` / `partially_filled__filled`, `open__filled` |
| `both_legs_ready` / `both_legs_settled` / `leg_failed` | `cashLegReady`, `tokenLegReady`, `cashLegSettled`, `tokenLegSettled`, `legFailed`: boolean | as named | SM-09 `filled__settling` / `settling__settled` / `settling__settlement_failed` |
| `retry_below_max` / `retries_exhausted` | `retryCount`: integer (retries already made); param `resale.settlementRetryMax` | `retryCount < max` / `retryCount >= max` | SM-09 `settlement_failed__settling` / `settlement_failed__busted` |
| `registry_ack` | `registryAck`: boolean | true | SM-09 `settled__registered` |
| `notice_published` / `silence_not_defaulted_to_manager` / `resale_halted` | `noticePublishedAt`: instant \| null; `silenceDefaultsToManager`, `resaleHalted`: boolean | non-null / false / true | SM-10 `filed__discussion` (first two) / `discussion__voting` |
| `voting_closed` | `votingClosesAt`: instant | `now >= votingClosesAt` | SM-10 `voting__tallying` |
| `quorum_met` / `quorum_not_met` | `votesCastUnits`, `quorumUnits`: integer | cast ≥ quorum / cast < quorum | SM-10 `tallying__passed`, `tallying__failed` / `tallying__quorum_failed` |
| `threshold_met` / `threshold_not_met` | `forUnits`, `votesCastUnits`: integer; `thresholdBps`: integer (from the rule pack) | `forUnits x 10000 >= thresholdBps x votesCastUnits` / negation | SM-10 `tallying__passed` / `tallying__failed` |
| `certifier_not_proposer` | `certifierIdentityId`, `proposerIdentityId`; `sodViolations` | identities differ and no violation | SM-10 `passed__certified` |
| `tipping_off_checked` | `tippingOffCheckedAt`: instant \| null | non-null | SM-11 `investigating__rfi_customer` |
| `smr_reference_recorded` / `rationale_documented` | `smrSubmissionRef`: string \| null; `noFileRationale`: string | non-null / non-empty | SM-11 `smr_decision__smr_filed` / `smr_decision__no_file` |
| `no_redress` / `complainant_accepted` / `afca_reference_recorded` / `external_outcome_implemented` / `new_information` | `redressProposed`, `complainantAccepted`, `externalOutcomeImplemented`, `newInformation`: boolean; `afcaReference`: string \| null | false / true / non-null / true / true | SM-12 `investigating__response_issued` / `response_issued__resolved` / `response_issued__escalated_external` / `escalated_external__resolved` / `resolved__reopened` |
| `case_type_in(list)` | `caseType`: `'lost_wallet' \| 'estate' \| 'court_order' \| 'error'` | in the list | SM-13 `triage__identity_reverification` [lost_wallet, estate]; `triage__legal_review` [court_order, error] |
| `identity_reverified` / `identity_reverification_failed` | `reverificationResult`: `'pass' \| 'fail' \| null` | pass / fail | SM-13 `identity_reverification__legal_review` / `identity_reverification__rejected` |
| `court_order_waiver` | `courtOrderId`; `waiverSignoffs` | order non-null and CCO and CEO signoffs from two distinct identities | SM-13 `checker_approval__executing` |
| `old_wallet_frozen_and_notice_sent` | `oldWalletFrozen`: boolean; `noticeSentAt`: instant \| null | true and non-null | SM-13 `old_wallet_frozen__timelock` |
| `timelock_elapsed` | `noticeSentAt`; `timelockHours`: integer (row 27 or 28 timelock key) | `now >= noticeSentAt + timelockHours` | SM-13 `timelock__executing` |
| `holder_objected` / `no_objection` | `objectionAt`: instant \| null; `noticeSentAt`; `timelockHours` | objection recorded before `noticeSentAt + timelockHours` / `objectionAt` null | SM-13 `timelock__disputed` / `timelock__executing` |
| `chain_confirmed` / `register_updated` | `chainConfirmed`, `registerUpdated`: boolean | true | SM-13 `executing__confirmed` / `confirmed__register_updated` |

Transitions not listed above carry no guard; their actors are in §2.7. The SMR clock (SM-11, 24 hours or 3 business days) and the complaint clocks (SM-12) are **A13 monitors, never guards**: a late SMR filing or a late acknowledgement is still accepted by the engine and is reported by A13 as a breach.

**2.5 Extensions (the only additions beyond the catalogue; the oracle checks `extensions.ts` equals exactly this list).**
- **(a) SM-00 veto window** (source: architecture brief §6.2 step 10; A14). New state `queued` (not terminal). `approved__queued`: guard `timelocked`, actors SVCA. `queued__queued_for_custody`: guard `timelock_elapsed_or_waived`, actors SVCA. `queued__cancelled` (label "veto"): guard `veto_recorded`, actors `row.vetoer`, `humanOnly`. Timelocks come from the row's timelock key (forced transfer and recovery 48 h, upgrade 72 h as seeds).
- **(b) SM-04 re-confirmation** (source: catalogue D06 and maker-checker row 8). New state `reconfirming` (not terminal). `live__reconfirming`: guard `linked_approval_closed`, actors SVCA. `reconfirming__live`: guard `reconfirm_window_closed`, actors SVCA (orders not re-confirmed move to SM-05 refund through A37). `reconfirming__refunding`: guard `offer_withdrawn`, actors SVCA.
- **(c) SM-00 stale and stuck requests** (source: A14's spec gaps; adopted here as a proposed, reversible engineering decision the founder confirms at HG-REVIEW; declining it is a one-entry data change plus the matching oracle file). `prechecked__expired` (label "TTL 72h"): guard `ttl_elapsed`, actors SVCA. `queued_for_custody__expired` (label "attestation expiry"): guard `attestation_expired`, actors SVCA. A14's third proposed edge, `prechecked -> rejected : checker rejects`, is **not** a new edge because the catalogue already has the pair `prechecked__rejected` ("precheck fails"); it is served by that edge through the `reject_reason_valid` guard, so a pair still identifies exactly one transition.
- Anything else the builder thinks is missing is a spec defect to escalate, not a transition to add.

**2.6 Labels (design projections).** `labels.ts` is a table of rows `(designKey, exactText, tone or style) -> (machine, state | event | derived fact)`; the strings are the prototype's, compared character for character:

| Design key (screen) | Exact text | Tone or style | Projection |
|---|---|---|---|
| `buy` step 3 `orderTimeline` (design code "INV-02 · Buy") | "Order placed" | done dot `#1F5E4B` / pending `#B9B4AA` | SM-05 event: entered `submitted` |
| same | "Funds held in escrow" | same | SM-05 state `funded` reached |
| same | "Offer reaches soft cap" | same | **SM-04** state `soft_cap_reached` reached for the order's offering |
| same | "Units issued by trustee" | same | derived fact: `register_entry_exists` true while the order is `allocated` (A18 register), not a state |
| same | "Tokens delivered to your wallet" | same | SM-05 state `settled` |
| `activity` table status (design code "INV-03") | "In escrow" | amber | SM-05 `funded` |
| same | "Allotted" | grey | SM-05 `settled` |
| `spDist` `distChain` (SPN-09) | "Rent roll uploaded" | DONE / YOUR TURN / WAITING by position | derived fact: `income_locked` for the period |
| same | "Distribution proposed" | same | SM-08 `calculated` reached |
| same | "Sponsor owner approves" | same | derived fact: `sponsor_owner_approved` (the SP-OWN signoff checked on `calculated__in_review`); SM-08 has no state for it and none is added |
| same | "RealX approval" | same | SM-08 `approved` reached. The prototype's sub-line "Treasury + compliance" is mock `who` text, not a label; the actual checkers are FINA then CFO (row 35). The mismatch is logged for A64 and A87 design fidelity, not changed here |
| same | "Paid to wallets" | same | SM-08 `executed` reached |
| `spRegister` status column (SPN-08) | "Active" / "Frozen" | green / red | SM-07 `active` / `frozen` |
| `adApprovals` (ADM-30) decision toast | "approved · sent to signing" | toast | SM-00 `prechecked__approved` then `approved__queued_for_custody` (or `approved__queued` when timelocked) |
| same | "rejected · maker notified" | toast | SM-00 `rejected` |
| same list rows | "High" / "Medium" | red / amber | **not a state label**: these are risk pills; the list shows only requests awaiting a decision (`prechecked`) |

A test fails if a row's text or tone differs from the prototype, if a projected state or trigger id does not exist, or if a derived fact is not a guard id in §2.4.

**2.7 Actors per transition (normative; `self` is the subject person).** Every transition not listed is SVCA only.
- SM-00: `draft__submitted` row.maker; `draft__cancelled` row.maker; `submitted__rejected` row.checker, humanOnly; `prechecked__approved` row.checker, humanOnly; `prechecked__rejected` SVCA or row.checker (humanOnly when `checker_rejected`); `queued__cancelled` row.vetoer, humanOnly.
- SM-01: `started__docs_submitted`, `rfi_pending__docs_submitted` self; `manual_review__rfi_pending`, `manual_review__edd_review` KYCA; `manual_review__approved`, `manual_review__rejected` KYCA, humanOnly; `edd_review__rejected` MLRO, humanOnly; `auto_approved__approved` KYCA or SVCA; `refresh_due__whitelisted` KYCA or SVCA; `whitelisted__suspended` MLRO, CCO, humanOnly (SGL+R act, §4.3); `suspended__offboarded` MLRO, CCO, humanOnly.
- SM-02: `requested__evidence_submitted`, `rfi__evidence_submitted`, `expired__requested` self; `under_review__rfi` CLSR; `under_review__valid`, `under_review__rejected`, `expiring__valid` CLSR, humanOnly; `valid__revoked` CLSR, CCO, humanOnly.
- SM-03: `sourced__screening`, `screening__*` PART, INVM; `due_diligence__ic_review` INVM; `ic_review__rejected` CFO, CCO, CEO, humanOnly; `ic_approved__title_verification`, `title_verification__*` LEGL; `title_verified__spv_formed` PART, LEGL; `operating__distressed`, `distressed__operating` PROP, ASTM, SVCA; `distressed__legal_hold`, `legal_hold__*` LEGL; `operating__exit_proposed`, `exit_approved__under_sale`, `under_sale__sold` ASTM, LEGL.
- SM-04: `draft__compliance_review` SP-OWN, CAPM; `compliance_review__draft` CCO, LEGL, humanOnly; `compliance_review__cancelled` CAPM, SP-OWN; `approved__scheduled`, `closed__finalising` CAPM; `live__suspended`, `suspended__live` CCO, humanOnly.
- SM-05: `submitted__cancelled`, `reflection_period__revoked` self.
- SM-06: `designed__sandbox_deployed` ENGO; `sandbox_deployed__deploy_requested` ENGO, TOKO; `deployed__configured` TOKO; `active__paused` CCO, SECO, MLRO (V3 holders), humanOnly (SGL+R act).
- SM-07: `active__frozen`, `partially_frozen__frozen` MLRO, CCO (humanOnly, SGL+R variant) or SVCA (4E+Q variant).
- SM-08: `draft__calculating`, `draft__cancelled` FINA, SP-FIN; `calculated__in_review` FINA or SVCA; `in_review__draft` CFO, humanOnly; `partially_failed__retrying`, `partially_failed__manual_payout` FINA.
- SM-09 (all phase P2): `open__cancelled` self, MKTO; `open__suspended` SVCA, MKTO.
- SM-10 (all phase P2): `draft__filed`, `filed__withdrawn`, `filed__discussion` LEGL, ASTM; `passed__certified` TP-RE (humanOnly) or SVCA with a recorded scrutineer attestation.
- SM-11: `alert_new__triage`, `triage__case_open` KYCA; `case_open__investigating`, `investigating__rfi_customer`, `rfi_customer__investigating`, `account_restricted__investigating`, `investigating__smr_decision`, `post_filing_monitoring__*`, `no_file__case_closed`, `exit_relationship__case_closed` MLRO; `investigating__account_restricted` MLRO, CCO, humanOnly (SGL+R act); `smr_decision__smr_filed`, `smr_decision__no_file` MLRO, humanOnly.
- SM-12: every transition CMPH or SVCA.
- SM-13: `requested__triage` SUPP; `triage__*` SUPP, KYCA; `identity_reverification__*` KYCA or SVCA; `legal_review__*`, `disputed__legal_review` LEGL; `timelock__disputed` self or SVCA.
- Phases: SM-09 and SM-10 transitions are P2; SM-05 `eligibility_check__reflection_period`, `reflection_period__*` and `revoked__refund_pending` are P2; every other transition is P1.

**2.8 Rights helpers** derived from state, used by A19 and the screens: `SM-01.canTransact(state)` true only in `whitelisted`; `SM-02.rights(state)` returns `{ buy, sell, hold }` so `expired` allows sell and hold but not buy; `SM-07.transferable(state)` false in `locked` and `frozen`, and in `partially_frozen` for the frozen part; `SM-06.forcedTransferAllowed(state, overrideApproved)` false in `paused` unless an approved override exists; `SM-03.resaleHalted(state)` true in `distressed` and `legal_hold`.

**2.9 Mermaid round trip.** `mermaid.ts` renders each catalogue table (extensions excluded) back to `stateDiagram-v2`, labels included; verify normalises whitespace only and compares with `data_states.py`, so the data cannot drift without a visible diff and an entry in `extensions.ts`.

**3. Gate codes G01-G22 as data.** `{ id, name, phase, enforcedAt (screen and module ids), onChain (true for G01, G02 wallet part, G06, G07, G08, G10 pause), counselQuestion (true for G06 fractional-unit treatment, G05 cooling-off text, G21 DAF timing), fixItMessageKey, sensitiveForAgents }`. Phases follow the architecture brief §11: P1 for G01-G04, G06-G08, G11-G14, G16-G21; P2 for G05, G09, G10; P3 for G15; G22 "Agent authority" carries phase `all` and `appliesToEveryAgentCall: true`, meaning A19 evaluates it on every agent-initiated action in every phase before any other gate. Thresholds, amounts and dates live in A19's rule packs (architecture brief risk 17). The gate list is exactly the 22 ids and names in `ids.json` (names cross-checked with `data_core.py`).

**4. Approval policy codes, vaults, the maker-checker baseline and its resolution.**
- `approval-policies.ts`: `{ code, minCheckerSets (0 for SGL+R and AUTO, 1 for 4E and 4E+Q, 2 for 6E and 6E+Q), checkersFromDifferentFunctions (6E, 6E+Q), custodyQuorum (+Q), actFirst (SGL+R), ratifiersRequired (1 for SGL+R), humanCheckerRequired (all but AUTO), autoBySvcaOnly (AUTO), ttlKey: 'approval.ttlHours', ratifyWithinKey: 'approval.ratifyWithinMinutes', reconcileAfter: true }`. Durations are keys into `seeds.ts`, never literals.
- `vaults.ts`: V1 owner (never moves value; timelock keys `timelock.ownerMinHours` / `timelock.ownerMaxHours`), V2 token agent (mint, burn, forced transfer, recovery, partial freeze, unpause; destination identity-registered; approval ID required), V3 pauser (restrict-only: pause, freeze, halt), V4 identity agent (registerIdentity, updateCountry, deleteIdentity, claims; auto within policy), V7 paymaster (auto within a daily cap key), V5 and V6 reserved for P3; V8 recorded as "not needed" with no code. Each vault has `functionClasses[]`, `quorumKey` (seed), **`signerRoles[]`**, `initiatorRoles[]` and `externalSigners`: V1 signerRoles CEO, CFO, CCO, SECO plus `externalSigners: 1` (the catalogue's "an independent", which is not a role and has no cell); V2 initiatorRoles TOKO, signerRoles CFO, CCO, TP-CUS (research pack `05-admin-panels/02-rbac-and-approvals.md` V2 row); V3 signerRoles CCO, SECO, MLRO (the on-call pauser is a rota drawn from these roles); V4 and V7 signerRoles SVCA.
- `maker-checker-baseline.ts` carries the 49 MAKER_CHECKER rows verbatim (action, maker text, checker text, code text, note). It is catalogue copy and is excluded from the hygiene grep (§4.2).

**4.1 Resolution table (`maker-checker-resolution.ts`, normative).** One entry per row or variant: `{ rowId, variant?, actionId, makerRoles[], checkerRoleSets[][], ratifierRoles[], vetoerRoles[], policyCode (exactly one of the six, or null for 19a), vault?, custodyFunction?, timelockKey?, evidence? }`. A14 maps these fields one to one onto `approval_policies` (`checker_role_sets`, `ratifier_roles`, `policy_code`); thresholds that choose a stricter row stay in A14. Non-role text maps as follows: "Custody quorum" = the V2 signer roles CFO or CCO as one checker set plus the V2 quorum; "Ratification" = `ratifierRoles` (the ratifier may not be the actor's identity); "Second approver" = the other of the two maker roles; "senior manager" = CCO or CEO; "board" = CEO as the board's delegate plus the board resolution hash as evidence; "RE board delegate" = TP-RE; "Scrutineer" = an external attestation recorded as evidence (the trustee variant holds the cell); "Admin quorum 3-of-5" and "owner quorum" = V1; "pauser" = the V3 signer roles; "independent" (row 6) is not a role: see escalations. `vetoerRoles` for every timelocked row is the row's checker roles plus CCO.

| Row | Maker roles | Checker role sets | Ratifiers | Code | Vault; function; timelock key |
|---|---|---|---|---|---|
| 1 Approve sponsor KYB | KYCA | [CCO] | | 4E | |
| 2 Create sponsor tenant | PART, SUPER | [CCO] | | 4E | |
| 3 Map licence to feature flag | SUPER | [CCO] | | 4E | |
| 4 IC approval of a property | INVM | [CFO,CCO,CEO] x2, distinct roles | | 6E | |
| 5 Record title verification | LEGL | [CCO] | | 4E | |
| 6 Publish valuation | VALO | [CFO], [CCO] interim (escalated) + independent member attestation as evidence | | 6E | |
| 7 Approve listing for go-live | CAPM | [CCO] (never CAPM or INVM, SoD 9) | | 4E+Q | V1; compliance and token module configuration |
| 8 Extend window or material change | CAPM | [CCO] | | 4E | |
| 9 Finalise offer and mint | CAPM | [CFO], [CCO] | | 6E+Q | V2; batchMint |
| 10 Revert and refund a failed raise | CAPM | [CFO] | | 4E+Q | V2; burn |
| 11 Deliver tokens | TOKO | [CFO,CCO] | | 4E+Q | V2; mint, batchMint |
| 12 Issue refund | FINA | [CFO] | | 4E | |
| 13 Change investor residency | KYCA | [CCO] | | 4E | V4; updateCountry |
| 14 Suspend account | MLRO, CCO | none | [MLRO,CCO] other than the actor | SGL+R | V3; setAddressFrozen (optional) |
| 15 Approve PEP or high-risk investor | KYCA | [MLRO], [CCO,CEO] | | 6E | |
| 16 Confirm true sanctions match | MLRO | none | [CCO] | SGL+R | V3; freeze |
| 17 Grant on-chain eligibility | SVCA | none (exempt from the checker invariant) | | AUTO | V4; registerIdentity |
| 18 Revoke on-chain eligibility | MLRO, CCO | [MLRO,CCO] the other one | | 4E | V4; deleteIdentity |
| 19a Approve wholesale status, certificate path | CLSR | none (exempt: the catalogue sets no checker) | | null | V4; claim with expiry |
| 19b Approve wholesale status, offer-amount path | CLSR | [CCO] | | 4E | V4; claim with expiry |
| 20 Change custody signer policy | SECO | [CCO], [CFO] | | 6E+Q | V1 |
| 21 Deploy token | TOKO | [CCO], [SECO] | | 6E+Q | V1; factory deploy |
| 22a Mint from a locked allocation file | TOKO | [CFO] | | 4E+Q | V2; mint |
| 22b Mint not from a locked file | TOKO | [CFO], [CCO] | | 6E+Q (normalised: two checkers need a 6E code) | V2; mint |
| 23 Burn | TOKO | [CFO] | | 4E+Q | V2; burn |
| 24a Freeze wallet, sanctions | MLRO, CCO | none | [MLRO,CCO] other than the actor | SGL+R | V3; setAddressFrozen |
| 24b Freeze wallet, other grounds | CCO | [LEGL] | | 4E+Q | V3; setAddressFrozen |
| 25 Pause token | CCO, SECO, MLRO | none | [CCO,SECO] other than the actor | SGL+R | V3; pause |
| 26 Unpause token | CCO | [SECO], [CFO] | | 6E+Q | V2; unpause |
| 27 Forced transfer | LEGL | [CCO], [CFO], [TP-RE] | | 6E+Q | V2; forcedTransfer; `timelock.forcedTransferHours` |
| 28 Recover lost wallet | KYCA (SUPP is intake only, no maker cell) | [CCO], [CFO], [TP-RE] (as SM-13) | | 6E+Q | V2; recoveryAddress; `timelock.recoveryHours` |
| 29 Upgrade token contract | ENGO | [SECO], [CCO] | | 6E+Q | V1; `timelock.upgradeHours` |
| 30 Change rule parameter | CCO | [LEGL] | | 4E+Q | V1; compliance module |
| 31 Publish rule-pack version | CCO | [LEGL], [CFO] interim (escalated: the catalogue names one checker under 6E) | | 6E | |
| 32 Set lock-up dates | REGO | [CCO] | | 4E+Q | V2; partial freeze with release date |
| 33 Register adjustment | REGO | [CCO], [CFO], [TP-RE] | | 6E | |
| 34 Certify register | REGO | [TP-RE] | | 4E | |
| 35 Approve distribution run | FINA | [CFO] | | 4E | |
| 36 Zero or reduced distribution | ASTM | [CFO], [CEO] + board resolution evidence | | 6E | |
| 37 Release escrow | FINA | [CFO], [CCO] | | 6E | |
| 38 Release withdrawal above threshold | TRSY | [CFO] | | 4E | |
| 39 Set resale band or floor | MKTO | [CCO], [CFO] | | 6E | |
| 40a Halt resale | MKTO, SVCA (auto on trigger) | none | [CCO] | SGL+R | |
| 40b Resume resale | MKTO | [CCO] | | 4E | |
| 41 Publish notice and open vote | LEGL | [CCO] | | 4E | |
| 42 Certify vote result | LEGL | [TP-RE] (or scrutineer attestation evidence) | | 4E | |
| 43 Publish disclosure or supplement | LEGL | [CCO] | | 4E | |
| 44 Publish public KPIs | IRCM | [CFO], [CCO] | | 6E | |
| 45 Change approval policy | SECO | [CCO], [CFO], [CEO] | | 6E | |
| 46 Grant a role | SUPER | [SECO] | | 4E | |
| 47 Toggle feature flag | SUPER | [CCO] | | 4E | |
| 48 Global inflow freeze | CCO, TP-RE | none | [CCO,CEO] other than the actor | SGL+R | |
| 49 Change fee schedule | CFO | [CEO] | | 4E | |

**4.2 One source for numbers.** `codes/seeds.ts` is the only A03 file that holds durations, counts and quorums, each as a named key: `approval.ttlHours` 72, `approval.ratifyWithinMinutes` 120, `timelock.forcedTransferHours` 48, `timelock.recoveryHours` 48, `timelock.upgradeHours` 72, `timelock.ownerMinHours` 24, `timelock.ownerMaxHours` 72, `onboarding.rfiExpiryDays` 30, `wholesale.certificateExpiringDays` 30, `wholesale.certificateValidityMonths` 24, `resale.settlementRetryMax` 3, `offering.reconfirmBusinessDays` 5, `breakGlass.activateOf3` 2, `breakGlass.maxHours` 4, `breakGlass.reviewBusinessDays` 5, vault quorums (V1 3-of-5, V2 2-of-3, V3 1-of-N). Seeds are read only by A06's seeding and by tests; guards read `ctx.params`, and a test asserts `guards.ts` and `engine.ts` do not import `seeds.ts`. At runtime **A14's versioned policy rows and A19's rule packs win**; A03 holds seeds only, so there is one source of truth.

**4.3 SGL+R is act-first, never a blocking guard.** A transition that carries an SGL+R act (SM-01 `whitelisted__suspended`, SM-06 `active__paused`, SM-07 `active__frozen` and `partially_frozen__frozen` in the SGL+R variant, SM-11 `investigating__account_restricted`, SM-09 `open__suspended` via row 40a) fires immediately for an actor holding the row's protective `act` cell. Ratification is tracked on the SM-00 request by A14 with an A13 clock (`approval.ratifyWithinMinutes`); a lapse triggers A14's sweep and an AUTO revert raised through the restorative row's own policy (for example row 26 for unpause). No guard in this package waits on a future fact.

**5. Roles and principal kinds (canonical for every consumer).**
- `roles.ts` is the 51-role list = the 52 ids of `ids.json` **less AGT-U**, in five groups: Investors 8 (VIS, INV-W, INV-S, INV-E, INV-J, INV-R, INV-F, INV-M), Sponsors 4 (SP-OWN, SP-OPS, SP-FIN, SP-IR), Licensed partners and providers 11 (TP-LIC, TP-RE, TP-CUS, TP-REG, TP-VAL, TP-PM, TP-AUD, TP-OBS, TP-KYC, TP-BANK, TP-SIGN), RealX staff 26 (CEO, SUPER, SECO, CCO, MLRO, KYCA, CLSR, PART, INVM, ASTM, VALO, CAPM, TOKO, REGO, FINA, CFO, TRSY, MKTO, TAXO, LEGL, IRCM, MKTG, SUPP, CMPH, PROP, ENGO), System 2 (AGNT, SVCA). Each role: `{ id, name, group, phase, principalKind, signsIn, tenantScoped }`. VIS **is** a role (the signed-out principal with public read cells and account creation). AGT-U is excluded because it is a principal kind bound to a named person, not a role with cells. A test asserts that every id in `ids.json` is one of the 51 or `AGT-U`, and `roles.ts` exports `excludedRoleIds: ['AGT-U']` so consumers never re-derive it.
- `principals.ts` defines **five** kinds, matching A00's `agent-access.json` `principalTypes`: HUMAN (investors, sponsors, staff); PARTNER (the six who sign in: TP-RE, TP-VAL, TP-PM, TP-AUD, TP-OBS, TP-LIC, each with its own cells; TP-CUS, TP-REG, TP-KYC, TP-BANK and TP-SIGN are adapters with `signsIn: false` and no cells); **AGNT**, the RealX staff AI agent (maker-only within its human's staff cells; never checker, ratifier, vetoer, act-first actor or custody signer); **AGTU**, a person's own agent acting through an API token (D04), bounded by the token's scopes and limits and by the person's live cells; when a staff person's token is used, the call is AGTU and AGNT's maker-only rule also applies; SVCA (execute-only cells: V4 identity registration after an approved case, V7 gas sponsorship, reconciliation and snapshot jobs, certificate expiry sweep, resale settlement in P2). AGNT and AGTU are never merged. Both agent kinds carry `identityId` = their human.
- `a11-view.ts` exports `kindToLetter` for A11's letters (read and unmask to `V`; draft and submit to `C`; configure to `E`; approve, ratify and veto to `A`; execute, sign and act to `O`; export to `X`) and `nonDelegable` (every action whose `delegable` is `false` or `"counsel"`), both importable from `@realx/domain/permissions`.

**6. Modules, actions and the matrix.**
- `modules.ts` is M01-M27 (M15 Lending and liquidity present with zero cells and `phase: P3`).
- **Action id grammar**: `<PREFIX>.<object>.<verb>`, where PREFIX is the owning module id (M01-M27) for console and maker-checker actions, or the product-map screen family (PUB, ONB, INV, POR, ACC, SPN and the other investor and sponsor families) for investor and sponsor screen actions; object and verb are lowerCamel; examples `M05.allocation.lock`, `M09.mint.approve`, `INV.order.place`, `ACC.token.create`. Every action also carries `vocabId`, the A00 `agent-access.json` action-vocabulary id (for example `order.place`, `token.create`), required for any action a person takes on their own account screens and null for console-only actions, so the D04 cross-checks join on it.
- **Module of every action**: `module` is always one M-module, the one that owns the record the action creates or changes (for reads, the module that owns the record read); investor and sponsor actions also carry `surface` (`public`, `investor`, `sponsor`, `partner`, `console`, `api`). For example `INV.order.place` is M05, `ACC.token.create` is M26, `SPN.listing.submit` is M04, `SPN.distribution.approve` is M12.
- **Sources and dedup**: the source references are exactly 433: the 49 MAKER_CHECKER rows x 2 (the maker action and the decision action) = 98, the 126 admin screen actions of `data_admin.py`, the 54 investor screen actions of `data_investor.py` (40 screens; PUB-06 and INV-07 have none), the 22 sponsor screen actions of `data_sponsor.py` and the 133 `acts` of `data_roles.py` (all 52 roles, including the D04 token-management acts appended by the file). `data_investor.py` has no `acts` list and is not read for one. Every reference maps to exactly one action id, or to `null` with a reason `off-platform` (for example TP-LIC "Appoint RealX and set the CAR scope") or `adapter` (TP-KYC, TP-BANK, TP-SIGN, TP-CUS, TP-REG acts). Two references map to one action only when they are the same verb on the same object by the same side (staff, investor, sponsor, partner), for example row 35's decision and ADM-19's approve action; every merge lists both references and a one-line reason. The mapping, with its exact distinct-action count in the header, is frozen at the Specify step in `A03-action-sources.json` (§0); `action-sources.ts` encodes it and the export reports the same count.
- An action is `{ id, vocabId, module, surface, screen?, kind (read | draft | submit | approve | ratify | veto | sign | execute | act | export | configure | unmask), policyRowRef?, gates[], phase, chain (off | on | both), family (read, invest-trade, money, documents, governance, sponsor-listing, sponsor-distribution, sponsor-reporting, access, staff-<module>), sensitive, delegable (true | false | "counsel", required, no default) }`. `sensitive` is true for the money, invest-trade and governance families, unmask, export, and every `access` action.
- `matrix.ts` is the set of cells `(roleId, actionId) -> { allow: true, constraint?: own-records | own-tenant | own-vehicles | time-boxed, protective?: true }`.
- **Invariants tested from the data, with their carve-outs listed so invariant and completeness tests agree:**
  1. Every §4.1 entry resolves to at least one maker cell, and to one checker cell per checker set, with checker roles different from maker roles; exempt: AUTO row 17 and row 19a (no checker), and the SGL+R rows 14, 16, 24a, 25, 40a, 48, which instead resolve to an `act` cell (protective) per maker role and a `ratify` cell per ratifier role.
  2. Every approve, ratify, veto, sign or act cell belongs to a HUMAN or PARTNER role, never AGNT or SVCA; SVCA cells are kind execute with `policyCode: AUTO`.
  3. **Own-token carve-out**: every signed-in principal (all HUMAN roles except VIS, and the six PARTNER roles) holds the own-records token-management cells (list, create, change, revoke its own tokens, see their activity) on ACC-05, SPN-16 or ADM-35 as `data_roles.py` grants them; these cells are exempt from invariants 4 to 6.
  4. VIS holds read cells on PUB screens plus exactly one non-read cell, `ONB.account.create`.
  5. SUPP holds no cell that moves money or changes a holding, the register, an identity, an investor's record or any M20 content. Carve-outs: SUPP may create and update support tickets (M23 ticket records), perform SM-13 `requested__triage` intake (row 28 intake, no maker cell), and hold the protective `act` cell "Revoke any token" on ADM-35 (with SECO).
  6. TP-OBS and TP-AUD hold read and export cells only, all `time-boxed`; TP-LIC holds read cells on M21 and M23 only (its "Appoint RealX" act is off-platform).
  7. Every action belongs to exactly one module; M15 has no cell.
  8. CAPM and INVM hold no approve cell on row 7; ENGO holds no approve cell on row 29.

**7. Delegable flags (founder decision D04).** Every action carries an explicit `delegable` value; the schema has no default and a test fails on any action without one. Kinds approve, ratify, veto, sign and act are always `false` (a staff agent can draft and submit but never check, veto, act first or sign; a sponsor agent can prepare a listing or distribution but never give the sponsor-owner approval). In addition, `false` for:
- identity and liveness verification and re-verification (ONB-03, ACC-02 step, the investor side of ADM-13 re-verify);
- creating the first passkey or wallet (ONB-01, ONB-08 "Create wallet");
- **any change to authority**: creating, widening, extending the expiry of, or rotating an API token (ACC-05 "Create token" and "Change scopes or limits", SPN-16 "Create token", ADM-35 "Create my token with write scopes"); granting or widening an on-chain session key (ACC-05 "Allow on-chain actions"); adding or removing a passkey or device (POR-06 "Add device");
- agent policy and kill switches: SPN-16 "Set sponsor agent policy", ADM-35 "Change platform agent policy", ADM-35 "Suspend all agent access" and every per-user, per-tenant, per-owner-type or platform kill-switch activation or lift (who may lift which of the five tiers is A00's by-engager rule in `agent-access.json`: staff-engaged, guard-engaged, owner-type and platform tiers 4E by two humans; a person-engaged user tier by that person with a fresh passkey; an SP-OWN-engaged tenant tier by an SP-OWN with a fresh passkey; a token hold by its owner with a fresh passkey or by a security officer who did not place it; never an agent); revoking someone else's token (SPN-16 "Revoke a team member's token", ADM-35 "Revoke any token"). Revoking one's own token stays `true` so an agent can switch itself off;
- changing the withdrawal bank account (ONB-09, ACC-01), setting up a PayTo mandate (ONB-09 "Set up PayTo"), and changing contact channels (email, phone, notification address on ACC-01), because those channels carry the sensitive-action notices;
- account recovery and guardian changes (ACC-02, POR-06 "Start recovery", ONB-08 "Set up recovery") and closing the account (ACC-01).
ACC-01 "Update details" is therefore split into `ACC.profile.update` (other fields, `true`), `ACC.contact.change`, `ACC.bank.change` and `ACC.account.close` (all `false`).
Flagged `"counsel"` and treated as `false` until [[Decisions]] records counsel's answer: the wholesale-status truthfulness declaration (ONB-06) and target-market answers (ONB-07, P2), both under A00's CQ-01; the sponsor declaration at submission (SPN-06) and the other sponsor declarations under A00's CQ-09; and the FATCA/CRS self-certification (ONB-05), the residency and TIN declaration and the no-TFN acknowledgement under A00's CQ-10. `"counsel"` is a data value, so flipping it later is a one-line change with a decision id.
**Cross-check with A00**: a test asserts that the set of `vocabId`s whose action is `false` or `"counsel"` is a superset of `agent-access.json` `nonDelegable` and `counselFlagged` (matched by vocabulary id, never by name), and that no `true` action's `vocabId` appears in either list.

**8. Scopes are matrix cells.** `scopes.ts`: `scopeId === actionId`; `scopesFor(owner)` returns the union of the owner's live cells with `delegable: true`, carrying each cell's constraint and family; `familiesFor(owner)` groups them for the ACC-05, SPN-16 and ADM-35 pickers so a picker never offers a right the viewer lacks. **`validateTokenScopes(owner, requested[])`** (A12's argument order) returns one refusal per rejected id, with the first matching code in this order: `SCOPE_UNKNOWN` (not an action id); `SCOPE_AGENT_NEVER_CHECKER` (kind approve, ratify, veto, sign or act); `SCOPE_NON_DELEGABLE`; `SCOPE_PENDING_COUNSEL`; `SCOPE_EXCEEDS_OWNER` (no such cell in the owner's rows); `SCOPE_PHASE_OFF`. It is monotone, so shrinking the owner's roles can only shrink the valid set; A12 re-runs it on every role change and every call. Non-delegable actions have no scope id at all.

**9. Segregation of duties as data.** `sod.ts` holds the ten rules of `data_admin.py` SOD. **Every rule compares identity sets**: an AGNT or AGTU principal's identity is its human (agent-of-X is X), with no exceptions. `checkSod(subject)` takes `{ kind: 'role_assignment' | 'approval_decision' | 'ratification' | 'veto' | 'custody_signature' | 'certification' | 'case_access' | 'break_glass', actor, request? }` and returns the violated rule ids. Rules: (1) `maker_ne_checker` on the same request; (2) `agent_never_checker_or_signer`: an AGNT or AGTU may draft and submit within its human's cells but can never check, ratify, veto, act first or sign; (3) `role_exclusive` SUPER with TRSY, FINA, CFO, TOKO and every role in any vault's `signerRoles` or `initiatorRoles` (derived from `vaults.ts`, not listed by hand); (4) `initiator_ne_cosigner` for custody operations; (5) `role_exclusive` FINA with CFO, VALO with MKTO; (6) `needs_checkers` register adjustment by REGO needs CCO and CFO; (7) `cannot_certify_own_proposal` for ASTM on votes; (8) `acl_hidden` M20 case content from SUPP and IRCM; (9) `no_self_grant`, plus `no_self_approve` for ENGO upgrades and the offering team's compliance review; (10) `break_glass`: holders CCO, SECO, CEO; activation needs the seed `breakGlass.activateOf3` distinct holders; duration and review deadline by seed keys; recorded; board notice. A14 and A11 call `checkSod`; they do not re-implement it.

**10. Shared vocabularies.** `reason-codes.ts` is the single list of refusal codes above (`SM_*`, `SCOPE_*`), each with a plain-English message key for the screens' fix-it text (design: any failed check disables the button and says how to fix it). A12 and A14 import these codes rather than defining their own; A14 may wrap `SM_FORBIDDEN` in its API error but must carry the engine code in the payload. Audit names for transitions are `sm.<machineId>.<triggerId>`; A10 records them and A14 emits them.

**11. Escalations raised by this spec (catalogue defects, not patched here).** (i) Row 6's "independent" valuation-committee member and V1's "an independent" signer are not roles: the founder names the principal via A00; interim mapping as in §4.1. (ii) Row 31 names one checker under 6E. (iii) No MAKER_CHECKER row covers SM-06 `active__migrating` or `active__redeeming`; A14 routes them under row 21's policy until the catalogue adds one. (iv) SM-05 `allocated__refund_pending` has no path back to `settled` for the allocated part of a partial allocation; A37 must settle the allocated units before moving the order or the catalogue must add an edge.

## Owned paths (other nodes do not edit these)

- <workspace>\app\packages\domain\src\state-machines\
- <workspace>\app\packages\domain\src\permissions\
- <workspace>\app\packages\domain\src\codes\
- <workspace>\app\packages\domain\scripts\domain-export\
- <workspace>\app\docs\domain\

(A02 owns the package root files and `src\money\`, `src\ids\`, `src\screens\`, `src\index.ts`; this node edits none of them and relies on A02's `"./*"` subpath export. The two new paths above must also be added to this node's owned paths in the graph data.)

## Design references

DESIGN = `<workspace>\design\RealX App.dc.html` run by `support.js`; EXT-A = [[A50 - Design extension A- investor and sponsor screens not in the prototype]]; EXT-B = [[A51 - Design extension B- operator console screens not in the prototype]].

- DESIGN `buy` step 2 `buyGates`: exactly G01, G04, G06, G07, G11 with Pass pills and the G07 line naming the cap; the gate ids and fix-it message keys come from `gates.ts`.
- DESIGN `buy` step 3 `orderTimeline` and `activity` status column: projections through `labels.ts` exactly as in §2.6 ("Tokens delivered to your wallet", not "Tokens delivered"; "Offer reaches soft cap" is SM-04; "Units issued by trustee" is a derived fact).
- DESIGN `adApprovals` (ADM-30): the inline note "Maker-checker (G13): you are not the maker, so you can approve. Approval ID is bound to the signing request." is SoD rule 1 plus SM-00; the decision toasts "approved · sent to signing" and "rejected · maker notified" are SM-00 projections; the list's "High" and "Medium" pills are risk, not states.
- DESIGN `adListing` (ADM-07) `tasks` rows G03, G21, G19, G20, G06, G17 with "Pass", "Set" and "2 flags": gate ids and `enforcedAt` from `gates.ts`.
- DESIGN `spDist` (SPN-09) `distChain` and `spRegister` (SPN-08) "Active" and "Frozen": projections of SM-08 and SM-07 per §2.6, with "Sponsor owner approves" as the derived fact `sponsor_owner_approved`.
- DESIGN `spNew` step 4 "Sponsor owner approval (G13)" "Required": an approve cell held by SP-OWN, never by the SP-OPS who prepared it and never by an agent.
- DESIGN tone map `TONE` (green, amber, red, grey, brick): `labels.ts` records the tone the prototype gives each projected label so screen nodes never invent tones.
- ACC-05, SPN-16 and ADM-35 are undesigned: EXT-A (scope picker limited to the viewer's rights, limits and expiry in plain English) and EXT-B (every token platform-wide, G22 evaluation log) consume `familiesFor`, `scopesFor` and `delegable.ts`; this node ships no screen.
- Visual tokens are not this node's concern beyond fix-it message keys, label text and tones.

## Requirements covered

- Product-map screens: none owned. The screens that show these states and codes (INV-02/INV-03 in design codes, ADM-30, ADM-07, SPN-08, SPN-09, ACC-05, SPN-16, ADM-35) are built by their own nodes on this data.
- Journeys: none owned; every journey's state assertions (for example J04 "settled only when both register entry and token balance exist", J06 "paid = approved", J09 "refused before 48 h unless CCO + CEO waiver") are guards in this package.
- Gates: **G13** done means the six policy codes, V1-V7 with signer roles, the 49-row baseline and its resolution table, SoD rule 1 and SM-00 exist as data, and `checkSod` refuses a checker whose identity equals the maker's (directly or through an agent) before A14 sees a request. **G22** done means the explicit delegable values, `scopesFor`, `validateTokenScopes`, the AGNT and AGTU principal kinds, the `sensitive` flags and SoD rule 2 exist as data, G22 is registered with phase `all` and `appliesToEveryAgentCall`, no non-delegable action has a scope id, and the A00 superset check passes; A19 and A12 consume this without adding rules of their own.
- Modules: none owned; M01-M27 are the matrix columns and every module except M15 has at least one cell.
- State machines, what done means for each (catalogue states / transitions): **SM-00** 14 / 14 plus extensions (a) and (c), 15 / 19 in total; `approved -> expired` and `prechecked -> expired` at the TTL key; a confirmed transaction that does not reconcile **stays in `confirmed`** with a reconciliation break raised by A20, because `confirmed__reconciled` requires `recon_clean` and there is no `confirmed -> failed` edge. **SM-01** 15 / 21; `canTransact` only in `whitelisted`; `rfi_pending -> expired` by `rfi_expired`; `auto_rejected` on a fraud signal. **SM-02** 9 / 11; `expiring` inside the expiring window; `expired` keeps sell and hold, blocks buy; `revoked` on misrepresentation. **SM-03** 20 / 25; `ic_review -> ic_approved` needs 2 of CFO, CCO, CEO; `title_verification -> rejected` on a title defect; `distressed` and `legal_hold` halt resale. **SM-04** 14 / 16 plus extension (b), 15 / 19; `closed -> failed` when the soft cap is missed; `finalising -> minted` only when the row 9 request is closed. **SM-05** 13 / 15; `reflection_period` unreachable in P1 or for wholesale, and unavoidable for retail in P2; `allocated -> settled` needs both register entry and token delivery; `refund_pending -> refunded` only when the row 12 request is closed. **SM-06** 12 / 14; `active -> paused` fires at once for a V3 holder and is ratified afterwards through SM-00; `paused -> active` through row 26; `upgrading` after the row 29 timelock; only `retired` is terminal. **SM-07** 7 / 12; `frozen -> active` MLRO then CCO; `locked -> active` only when the lock-in date has passed; `eligible_empty -> deregistered` on deleteIdentity. **SM-08** 15 / 18; `funded -> executing` only on a file-hash match; `reconciling -> closed` only when paid equals approved to the cent; `draft -> cancelled` is the only cancel. **SM-09** 13 / 16 (P2); `validating -> rejected` for band, lock-in, cap or eligibility; `settlement_failed -> busted` only when retries are exhausted and MKTO then CCO signed off; `settled -> registered` on the registry ack. **SM-10** 13 / 13 (P2); `filed -> discussion` blocked when silence defaults to the manager; `passed -> certified` never by the proposer's identity. **SM-11** 13 / 16; states carry the MLRO and CCO ACL; `investigating -> account_restricted` fires at once and is ratified afterwards; `smr_decision -> smr_filed` is never blocked by the clock, which A13 monitors. **SM-12** 9 / 11; the acknowledgement SLA is an A13 monitor; `escalated_external` shows AFCA; `resolved -> reopened` on new information. **SM-13** 14 / 17; `checker_approval -> executing` only with a court-order waiver by CCO and CEO; `timelock -> executing` only after the timelock, with no objection and custody quorum; `timelock -> disputed` when the holder objects.

## Gauntlet loop for this node

- **Specify.** This note is frozen with the fixtures of §0 and their hashes. Before the oracle is written: the action-source mapping `A03-action-sources.json` is produced and frozen (433 references, exact distinct count in its header); extension (c) is confirmed or declined at HG-REVIEW (default: adopted). Reason-code names, guard ids, trigger ids, fact names, seed keys and export file names above are part of the spec so the oracle can address them.
- **Oracle (node QA, Opus 5.5 high, before the build).** Hidden Vitest files in `<workspace>\gauntlet\hidden\A03\`: (a) for each machine, the allowed set generated independently from `data_states.py` plus §2.5, and a sweep over every ordered (from, to) pair of its states, 2,527 pairs in total including self-pairs, expecting `SM_TERMINAL` when `from` is terminal, `SM_FORBIDDEN` for any other pair outside the table, and success with the happy ctx for the 227 table transitions; the same sweep through `fire()` over (state, triggerId), plus unknown machine, state and trigger vectors checking the §2.2 precedence; (b) guard vectors on both sides of each boundary in §2.4 (47:59 vs 48:00 with the seed, one cent short, 2 vs 3 retries, wholesale vs retail in P1 and P2, missing fact fails closed); (c) SGL+R vector: pause at t0 with no ratification succeeds, and suspend, freeze and restrict likewise; (d) matrix invariants and carve-outs of §6, and completeness against the 433 references of the frozen mapping; (e) fast-check property tests over random owners and scope sets: `validateTokenScopes` never accepts a scope outside `scopesFor(owner)`, never accepts a non-delegable, counsel-pending or checker-kind action, and returns the first code by §8 precedence; (f) the A00 superset check of §7; (g) SoD vectors including an agent of the maker as checker (rule 1), an agent of SUPER drafting "grant role to SUPER" (rule 9), an agent of ASTM certifying ASTM's vote (rule 7), an agent of the initiator co-signing (rule 4), and SUPER plus a V2 signer role (rule 3); (h) the label table of §2.6 against the prototype; (i) a mutation probe that appends one transition to a copied table and expects the Mermaid diff test to fail.
- **Build (Grok 4.7 xhigh, split by the planner).** Five parallel sub-agents on disjoint files, then one integration pass: (1) `engine.ts`, `guards.ts`, `mermaid.ts` and SM-00 to SM-06; (2) SM-07 to SM-13, `labels.ts`, `extensions.ts`; (3) `codes/` (gates, approval policies, vaults, baseline, resolution, seeds, reason codes); (4) `roles.ts`, `principals.ts`, `modules.ts`, `actions.ts`, `action-sources.ts`, `matrix.ts`; (5) `delegable.ts`, `scopes.ts`, `sod.ts`, `a11-view.ts`. Integration: indexes, schemas as data, `scripts/domain-export`, the diff tests, the builder's own tests (which must not read the hidden folder), and the evidence manifest `app/evidence/A03.json` with counts, fixture hashes and hashes of the exported JSON.
- **Challenge (the node QA).** Run the hidden oracles and the mutation probe; read `delegable.ts` line by line against D04, `agent-access.json` and the counsel list; run the hygiene grep of the pass criteria; check imports for anything outside `packages/domain`; check the export is byte-identical over two runs; check `matrix.ts` has no cell for M15 and no approve, ratify, veto, act or sign cell for AGNT or SVCA; spot-check ten transitions against the catalogue notes (for example SM-06 "forced transfers are blocked while a token is paused unless an override is approved"); write `app/evidence/A03.qa.md`.
- **Gate.** Pass unlocks A04, A12, A14 and A19. Hidden cases are published into the repo after the pass.
- **Fix loop.** At most three; each loop re-runs the whole oracle, not only the failed files.
- **Escalate** when a needed transition, guard, role or action is missing from the catalogue itself (spec defect: reopen the A00 charter rather than patch the table; see §11), when a D04 delegable decision cannot be settled without counsel, when a fixture hash differs, or when the third loop still fails the same assumption (stop rule).

## Node-specific challenges (node QA)

- Money: SM-08 `reconciling__closed` accepted with paid one cent short of approved; SM-05 `allocated__settled` accepted with a register entry and no token delivery (or the reverse); SM-05 `awaiting_funds__funded` accepted on a partial match.
- Compliance (maker-checker): an approval whose checker is an AGNT or AGTU of the maker passes `checkSod`; a 4E policy lists an agent among eligible checkers; the SP-OPS who prepared a listing holds the "Sponsor owner approval" cell; a §4.1 entry with a free-text role that is not in `roles.ts`.
- Act-first: an SGL+R pause, freeze, suspend or restrict refused because no ratification exists yet, or any guard reading a ratification fact.
- Security (scopes): a TOKO's token requests `M09.mint.approve`, an investor's token requests `ACC.token.create`, `ACC.bank.change`, `ACC.contact.change` or the session-key grant, or a sponsor accountant's token requests `M12.run.approve`; any of these accepted, or refused with a code other than the §8 precedence gives, fails the node. Also: the owner loses a role and a previously valid scope is still returned by `scopesFor`; an action without an explicit `delegable` value compiles.
- Compliance (timelocks and waivers): SM-13 `timelock__executing` at 47 h 59 min with the seed, or `checker_approval__executing` with a waiver signed by the CCO alone; SM-06 forced transfer allowed from `paused` without an override; a guard that reads a duration literal instead of `ctx.params`.
- Phase leakage: SM-05 `reflection_period` reachable for a wholesale order or in P1, or skippable for a retail order in P2; G05 or G09 evaluated as anything but "not applicable" in P1; SM-09 or SM-10 transitions firing in P1.
- Design fidelity: a §2.6 row whose text or tone differs from the prototype, or whose projection names a state, trigger or guard that does not exist.
- Data drift: a transition or cell added outside the catalogue and §2.5 passes `npm run verify` because the Mermaid or JSON diff normalises too much; the initial `[*]` entry stored as a transition row; the refusal precedence differs from §2.2.

## Pass criteria

- Hidden oracle: 227 of 227 transitions (219 catalogue plus 8 extension edges) accepted with their happy ctx; every machine starts in its catalogue initial state and no `[*]` entry is fireable; all 2,527 ordered pairs give the expected result class, including `SM_TERMINAL` from terminal states and `SM_FORBIDDEN` for the rest; the §2.2 precedence holds for unknown machine, state and trigger; all guard boundary vectors agree; the SGL+R act-first vectors pass; zero disagreements after the final loop.
- Mermaid rendered from the catalogue tables equals `data_states.py` for all 14 machines after whitespace normalisation (233 edges including the 14 initial entries); the mutation probe fails verify.
- Counts in the exported JSON: 14 machines, 183 states, 219 catalogue and 8 extension transitions, every transition with a unique `triggerId` of the form `<from>__<to>`, 22 gates (G22 with phase `all`), 6 policy codes, 7 vault codes each with signer roles, 49 baseline rows and 53 resolution entries (49 rows with rows 19, 22, 24 and 40 split into two variants), 51 roles, 5 principal kinds, 27 modules (M15 with zero cells), 433 source references all mapped, and a distinct action count equal to the frozen mapping's header.
- Property tests: 10,000 random (owner, scope set) pairs and `validateTokenScopes` never accepts a scope outside `scopesFor(owner)`, never accepts a non-delegable, counsel-pending or checker-kind action, and every refusal carries the §8 code; zero approve, ratify, veto, act or sign cells for AGNT or SVCA; the A00 superset check passes.
- SoD: all ten rules have passing vectors, including agent-of-X cases for rules 1, 4, 7 and 9.
- Purity and hygiene: no import outside `packages/domain`; no `Date.now`, `new Date()` without an argument, or I/O under `src/`; **hygiene grep** over `src/state-machines`, `src/permissions` and `src/codes`, excluding `codes/seeds.ts`, `codes/maker-checker-baseline.ts` and the `label` string fields of the 14 machine files (verbatim catalogue text, guarded by the Mermaid round trip), finds no match for `A\$\s?\d`, `\d+(\.\d+)?\s?%`, `\b\d+\s?(h|hours?|min|minutes?|days?|business days|months?|years?)\b`, `\b(19|20)\d\d-\d\d-\d\d\b`, `\b\d{1,2} (Jan|Feb|Mar|Apr|May|Jun|Jul|Aug|Sep|Oct|Nov|Dec)[a-z]* \d{4}\b`, or the vendor names `Fireblocks|Tokeny|T-REX|Monoova|Zepto|Equifax|Safe\b`; 100 percent line coverage on `engine.ts`, `guards.ts`, `matrix.ts`, `scopes.ts` and `sod.ts`, at least 95 percent on A03's folders; `npm run domain:export` byte-identical on two consecutive runs.
- Labels: every §2.6 row matches the prototype text and tone exactly.

## Human gates touched

- None block this node. HG-LEGAL (counsel) receives questions from it: which personal declarations must stay non-delegable to a person's agent (wholesale-status truthfulness and target-market answers under CQ-01; the sponsor declarations under CQ-09; the FATCA/CRS self-certification, the residency and TIN declaration and the no-TFN acknowledgement under CQ-10). Until answered they are `"counsel"` in `delegable.ts` and behave as non-delegable; the answer is recorded in [[Decisions]] and changes data only.
- HG-REVIEW: the founder confirms or declines extension (c) and names the "independent" principal of §11 (i); both default as stated and change data only.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
