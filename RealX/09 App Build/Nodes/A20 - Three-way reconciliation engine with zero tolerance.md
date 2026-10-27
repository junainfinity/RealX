---
title: "A20 - Three-way reconciliation engine with zero tolerance"
created: 2026-09-23
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am1]
node: A20
milestone: AM1
lane: "Register"
oracle: yes
human_gate: none
---

# A20 - Three-way reconciliation engine with zero tolerance

**The engine that proves, every night and after every privileged action, that the trustee's unit register (the legal record), the ERC-3643 token ledger and the client-money bank accounts agree to the unit and to the cent. It is built from pure comparison functions over frozen snapshot objects and a frozen set of auxiliary inputs, implementing rules R1-R18. Each `ReconRun` can be reproduced from its keys: register version, chain block, bank statement ids and the hash of the auxiliary inputs. A grouping step turns rule findings into `ReconBreak`s, each with a type, delta, owner and proposed fix, raised through the alerts service. A Sev1 break halts the vehicle (G10 resale halt; mints, burns and payouts refused) until a clean re-run closes it.**

**Build status:** not started. **Milestone:** AM1 (Money, register, approvals and agent-access core). **Lane:** Register. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 12-16 agent-hours (build, challenge and verify loops).

**Depends on:** [[A18 - Unit register (legal record) and registry sandbox adapter]], [[A16 - Client-money ledger (double entry)]], [[A17 - Bank rail sandbox adapter (NPP-PayID-PayTo) and receipt matching]], [[A15 - Custody sandbox adapter and policy engine P1-P10]], [[A19 - Rules engine- gates G01-G22, rule packs, threshold monitors, simulator]], [[A13 - Alerts and SLA-clocks service]], [[A06 - Seed data and scenario simulator]]
**Unlocks:** [[A28 - Token operations service (mint, burn, freeze, pause, forced transfer, recovery)]], [[A46 - Job scheduler, integrations health and webhooks]], [[A54 - Public site- home, investments, property page, sponsor profile, fees]], [[A55 - Public site- how it works, returns calculator, transparency, help and documents]], [[A66 - Console- command center, alerts centre, approvals inbox, audit log]], [[A73 - Console- rule packs and threshold monitors, register and reconciliation]], [[A91 - P2- resale board market module (SM-09)]]

(A17, A15 and A06 were added on 24 Sep 2026 after the independent review. None of them depends on A20, so no cycle is created. The graph data, `app-graph.json` and [[App Development Graph]], must carry the same three edges.)

**Oracle rule:** The node QA writes hidden oracle tests from this spec before the build starts ([[App Gauntlet]] §2). They go to `<workspace>\gauntlet\hidden\A20\` and the builder never reads that folder. The oracle feeds synthetic register, chain, bank, ledger and auxiliary inputs with planted drifts into the pure comparison functions and into the run service. Examples: 9,998 register units against 10,000 tokens; an on-chain holder who is not on the register; a bank balance one cent away from the ledger; a frozen amount that disagrees. The oracle asserts the exact grouped break set, the exact halt effects, and that a break closes only on a clean re-run.

This note is not legal advice. The register is treated as the legal record because the founder decisions and the research pack say so (Corporations Act s169 as the pack reads it). Whether a constitution may ever designate the chain as the register is a counsel question, and it is off by default (`chain_is_register_clause` is null on every vehicle).

## Build brief

**Scope and placement**

- Three owned areas:
  - `packages/domain/src/recon/`: pure TypeScript with no I/O, no database and no clock. Holds the snapshot and auxiliary-input types, the rule functions, the break-set grouping builder, the cause/fix table and the result hash.
  - `services/api/src/modules/recon/`: the run service, the break lifecycle, the halt guard, the read models and routes, the Drizzle schema and migration (in the module folder), the chain-snapshot source registry and the auxiliary-provider registry.
  - `services/worker/src/jobs/recon/`: the nightly cycle, the targeted-run consumer of outbox events and the break-SLA watcher.
- Module pattern from A09: one public `index.ts`, boundary lint, zod schemas that feed the OpenAPI document, and jobs registered with pg-boss. The read contracts go into the OpenAPI document in the first commit, so `packages/sdk` and the operation annotations the MCP server is generated from carry them before the engine passes.
- Time comes only from A09's injectable clock. `Date.now()` and argument-less `new Date()` are forbidden in the owned paths, and `npm run verify` greps for them.
- Money is integer cents and units are whole integers, both from `packages/domain` (A02). The recon package contains no floating-point arithmetic and no tolerance parameter of any kind. A configuration key named `tolerance`, `epsilon` or `rounding_allowance` anywhere under the owned paths fails lint.
- The engine runs as the service account `svc-recon`. This is an SVCA row in A03's principal data: no login, no keys, no custody role and a cap of zero on every money or supply action. Every audit event it writes names that principal as the actor, which is what ADM-31 shows as "Recon job".

**The snapshots (inputs of every run)**

- **Register side.** A `RegisterSnapshot` built from an A18 `RegisterVersion` and its entries. It uses A18's field names:
  - `version_id`, `vehicle_id`, `as_of`, `source` (A18's enum: `internal` from the 00:00 snapshot job, `registry` from the 02:10 registrar extract, `certification`), `state`, `certified_by` (nullable; a TP-RE or TP-REG user id on a certified version), `file_sha256`, `total_units`, `holder_count`;
  - `entries[]`, each `{investor_id, wallet_address, units, frozen_units, lots[] {lot_id, units, acquired_at, price_cents, lock_in_until}}`.
  - **Which version is the compared side.** R2, R3, R3b and R7 compare against the **registrar extract**: the `source: registry` version for the cycle date, or a certified version when a run names one explicitly. The registrar's file is the legal record, so it is always on the compared side. RealX's own `source: internal` copy for the same `as_of` is a second register input, used only by rule R3a (register divergence).
  - A18 guarantees that `total_units` equals the sum of `entries[].units` and that each entry's lots sum to its units. A20 re-checks both. A mismatch opens a `register_internal` Sev1 break; the header is never trusted.
- **Chain side.** A `ChainSnapshot` comes through the `ChainSnapshotSource` interface defined in this node. `getSnapshot({tokenAddress, atBlock?, sinceBlock?})` returns:
  - `{block_number, chain_id, token_address, total_supply, paused, roles {owner, agents[], identity_registry, compliance, agent_gateway}, holders[] {address, balance, frozen_amount, frozen}, ir_members[] {address, onchain_id, country}, claims[] {onchain_id, topic, issuer, valid_until, revoked}, compliance_params_sha256, document_anchors[] {doc_type, sha256}}`;
  - plus `events[]` from `sinceBlock` (the previous clean run's block) to `block_number`. Each event is `{kind (mint | burn | transfer | forced_transfer | freeze | partial_freeze | unfreeze | recovery), tx_hash, block_number, log_index, from, to, amount, approval_id | null, data_hash | null}`.
  - This node ships an **in-memory stub source** (driven by fixtures, deterministic, scriptable by tests and by the seed) and a registry, `registerChainSnapshotSource(source)`. [[A27 - Chain adapter, deployment manifests and indexer]] registers the real indexer source later without touching this node. With the stub registered, an `atBlock` request returns the fixture keyed by that block, or `source_unavailable`.
- **Bank side.** A `BankSnapshot` per client-money bank account is built from A17's daily statement: `getStatement({accountRef, date})` returns `{statementId, accountRef, opening_balance_cents, lines[], closing_balance_cents, file_sha256, cut_at}`. The client-money accounts are the wallets account, offer escrow and distributions. The RealX operating account is never part of a recon snapshot.
- **Ledger side.** A `LedgerSnapshot` comes from A16 `snapshot({tenant_id, statement_id})`, taken for every tenant that holds ledger accounts in that bank account. It contains the header `{as_of_seq, as_of_time, statement_id, accounts_hash, client_money_total_cents}` and one row per account `{account_id, account_type, owner_ref, is_client_money, balance_cents}`.
  - A row with `is_client_money = false` (A16's `operating` and `revenue`) is excluded. If such an account turns up in a client-money comparison, that is itself a `cash` break.
  - The run stores `accounts_hash`, so a replay can prove the ledger side did not move.
- **Account-to-vehicle mapping (from A16's `owner_ref`, computed in the pure package).**
  - Each ledger account maps to the bank account whose `bank_mirror` it balances against.
  - Each bank account maps to a set of vehicles, `account_vehicles(account_ref)`. That set is the vehicles owning any ledger account in it: `offer_escrow` through offering to vehicle; `distribution_payable` through run to vehicle; `withholding_payable` and `rounding` by vehicle.
  - `investor_cash`, `suspense_unmatched`, `payout_clearing`, `fee_payable_realx` and `float_interest_payable` map to the platform, not to a vehicle.
  - The mapping is stored on the run as data, so the halt guard and the read models use the same answer.
- **Auxiliary inputs (`AuxInputs`, frozen type).** Every rule input that is not in the snapshots above arrives through a named provider interface registered with `registerAuxProvider(name, provider)`. AM1 ships a stub for each provider, fed from the A06 seed and scriptable by tests; the named later node registers the real provider without editing this node. The providers:

| Provider | Content | Used by | AM1 stub, real provider |
|---|---|---|---|
| `vehicles` | vehicle id, tenant, token address, offering id, lifecycle state (SM-03 and SM-04), lock-in mechanism (`partial_freeze` or `compliance_module`) | every run, and the cycle's vehicle list | A06 seed; [[A36 - Offerings- offer terms, listing review, raise monitor (SM-04)]] and A34 |
| `roles` | approved role set per token; app token state (`paused`) | R1 | A19 approved set; A28 token state |
| `allocations` | locked allocation files `{offering_id, allocation_id, total_units, lines_sha256}`; approved burns `{approval_id, units}` | R2, R7 | seed; A37 and A28 |
| `in_flight` | register instructions acked, and custody ops not yet `confirmed`, with approval id, kind, lines and `acked_at` | R2, R3, R17 | A18 instructions and A15 ops (both real in AM1) |
| `identity` | per investor: wallet, identity state, KYC expiry | R4 | seed; A19 report, fed by A30 |
| `claims` | required claim topics of the vehicle's rule pack; off-chain claim records | R5 | A19 pack; seed claims, then A31 |
| `compliance_rule` | effective `params_sha256` per token | R6 | A19 (real in AM1) |
| `freeze_orders` | active freeze and partial-freeze orders per wallet and units | R3b | seed; A28 |
| `distributions` | executed runs in the window: approved net, withholding total, per-holder credit lines, payout file lines | R8 | seed; A40 |
| `trades` | P2 resale trades with chain tx and registry ack refs | R11 | empty; A91 |
| `valuation` | latest approved valuation and validity window | R12 | A19 G20 data |
| `snapshots` | distribution and vote snapshot files `{kind, register_version_id, block, file_sha256}` | R13 | seed; A40 and A93 |
| `documents` | latest effective document version per type and its hash | R14 | seed; A35 |
| `funded_orders` | Σ funded orders per open offering (cents and application units) | R10 escrow leg | seed; A37 |
| `audit_verification` | A10's latest daily verification result and time | R18 | A10 (real in AM1) |
| `external_conditions` | open conditions behind external breaks (below), per break id | external breaks | fixture; A28 |

- **Acquisition.** Snapshot and auxiliary acquisition is the only impure step. The run service:
  - resolves the register version (the cycle's registrar extract, a certified version, or an explicit id), plus the internal version for the same `as_of`;
  - resolves the chain block (the latest finalised block, or an explicit number);
  - resolves the bank statements (the latest cut per mapped account, or explicit ids);
  - calls each auxiliary provider once, with the run's captured `as_of` instant;
  - hands the resulting immutable objects to the pure functions.
- If any source or provider cannot deliver, the run completes as `incomplete` and names what is missing. An incomplete run is never clean.

**ReconRun (reproducible from its keys)**

- The `recon_run` row:
  - `id` (`RUN-<yyyymmdd>-<6 base32>`), `cycle_id` (nullable), `scope` (`vehicle` or `platform`), `vehicle_id` (null for platform runs);
  - `trigger` (`nightly`, `targeted`, `preflight`, `manual`, `replay`) and `trigger_refs[]` (custody op ids, instruction ids, approval ids, statement ids, actor);
  - the keys: `register_version_id` (A18 `version_id` of the compared version), `internal_version_id`, `block_number`, `statement_ids[]` (sorted), `ledger_accounts_hash`, `aux_inputs_sha256`, and `aux_snapshot` (the canonical auxiliary inputs as JSON; investor ids and wallet addresses only, no names);
  - `rule_set_version`, `state` (`queued`, `snapshotting`, `comparing`, `completed`, `incomplete`, `failed`) and `missing_source` (nullable);
  - `result` (`clean`, `breaks`, `not_issued`) and `matched` (boolean; see the summary formula);
  - `result_sha256`, `rule_results_json` (one entry per rule: `pass`, `break`, `not_applicable`, `skipped_stub`, `in_flight`), `break_ids[]`, `account_vehicle_map_json`, `started_at`, `finished_at`, `duration_ms`.
- **Reproducibility is a contract.** `compare(inputs: {register, registerInternal, chain, bank[], ledger[], aux}, ruleSetVersion)` run twice on the same inputs yields byte-identical `rule_results_json` and the same `result_sha256`.
- `POST /v1/recon/runs/{id}/replay` re-fetches the register versions, block and statements by their stored keys and re-uses the stored `aux_snapshot`, after checking that it still hashes to `aux_inputs_sha256`. It must reproduce the stored hash. If it cannot, it raises a `replay_mismatch` Sev1 break: a source that changed history is itself a finding.
- A replay writes a new run with `trigger = replay` and a link to the original. It never overwrites the original run's results.
- **Not issued yet.** A vehicle whose token is not deployed, or whose allocation has not reached an acked register issue, gets `result = not_issued`. The run reports register total 0, chain supply 0, `escrow_application_units` and the escrow leg. `matched` is true when every leg the vehicle has agrees:
  - register 0 = chain 0;
  - the ledger `offer_escrow` for its open offering = Σ funded orders;
  - no open `cash` break on an account mapped to it.
- Marine Parade (in review, no offer) has all legs at zero, so it is `not_issued` and matched. ADM-17 then renders "6,400 (escrow) · token supply 0" and the transparency page shows the row rather than hiding it (product-map QA items 13 and 14: the seed carries all eight trusts, and the register column reads 0 before issue).
- **Units in flight.** "Register first" means an acked issue instruction precedes the mint. When the register exceeds the chain only by units of acked instructions whose paired custody op is not yet `confirmed`, and the ack is less than 24 h old, those units are reported as `in_flight` with the approval id. This is not a break, and `assertClear(vehicleId, 'mint')` for that approval id passes. Past 24 h, the same units become part of the vehicle's `qty` break.

**Rules (pure functions in `packages/domain/src/recon/rules/`)**

| Rule | Scope | Check (all comparisons exact) | Finding type | Severity |
|---|---|---|---|---|
| R1 roles | vehicle | `roles.owner`, `roles.agents`, `identity_registry`, `compliance` and `agent_gateway` equal the approved role set; `paused` equals the app's token state | `role_drift` | Sev1 |
| R2 supply | vehicle | chain `total_supply` = registrar `total_units` (less `in_flight` units) = Σ locked allocations − Σ approved burns | `qty` | Sev1 |
| R3 holder | vehicle | Per registrar entry: chain balance of `wallet_address` = `units` (less `in_flight`). An on-chain holder with balance > 0 whose address is on no entry is `unknown_holder`. | `qty`, `unknown_holder` | Sev1 |
| R3a register divergence | vehicle | Per holder: internal version units, `frozen_units` and lots = registrar extract units, `frozen_units` and lots at the same `as_of` | `register_divergence` | Sev1 |
| R3b freeze | vehicle | Per holder: registrar `frozen_units` = Σ active freeze and partial-freeze orders. Chain `frozen_amount` = that sum, plus the units in lots with `lock_in_until > as_of` when the vehicle's lock-in mechanism is `partial_freeze`. Chain `frozen` flag = holder frozen by order. | `freeze_drift` | Sev1 |
| R4 identity registry | vehicle | `ir_members` = {wallets of investors whose identity state is active and KYC unexpired}. Extra members are `stale_registration`; missing members with a balance are `missing_registration`. | `missing_registration`, `stale_registration` | Sev2 |
| R5 claims | vehicle | For each required claim topic: a claim is present and unexpired if and only if the off-chain record is verified and unexpired | `claim_drift` | Sev2 |
| R6 rule hash | vehicle | `compliance_params_sha256` = the effective `params_sha256` | `rule_drift` | Sev1 |
| R7 allocation | vehicle | Σ mint events per offering (from `events[]` and prior clean runs) = the locked allocation total; every mint, burn and forced-transfer event carries exactly one approval id known to the allocation or burn inputs | `qty`; an event with no known approval is `unknown_execution` | Sev1 |
| R8 distribution | vehicle | For each executed distribution run in the window: Σ ledger credits = Σ approved net; Σ withholding = the run's tax total; bank payouts (when any) match the file | `payout_mismatch` | Sev1 |
| R9 stablecoin | — | **Stub** (P3): returns `skipped_stub` with the rule version, never `pass` | — | — |
| R10 client money, account leg | platform | Per client-money bank account: `closing_balance_cents` = the `bank_mirror` balance = Σ client-money liability accounts mapped to it, at the statement cut. The rounding account is included on the ledger side, so recon sees zero. | `cash` (subject: the account) | Sev1 (any non-zero delta, one cent included) |
| R10 client money, escrow leg | vehicle | Per open offering: ledger `offer_escrow` balance = Σ funded orders | `cash` (subject: the offering) | Sev1 |
| R11 trade (P2) | vehicle | A resale trade is `registered` only when both the chain transfer and the registry ack exist; one without the other is pending | `pending_registration` | Sev2, Sev1 past the 24 h SLA |
| R12 valuation | vehicle | The latest approved valuation is inside its validity window | `valuation_stale` | Sev2 (G20 handles the resale halt; this node only records the finding) |
| R13 snapshot | vehicle | Recomputing a distribution or vote snapshot from its register version and the chain holders at its block reproduces the stored file hash | `snapshot_drift` | Sev1 |
| R14 documents | vehicle | Each anchored document hash equals the latest effective document version of that type | `doc_drift` | Sev2 |
| R15 vote anchors | — | **Stub** (P2): `skipped_stub` | — | — |
| R16 omnibus sub-ledgers | — | **Stub** (P3): `skipped_stub` | — | — |
| R17 registry instruction | vehicle | Every holding-change event in `events[]` since the previous clean run has an acked registry instruction with the same approval id. A pending instruction older than the 24 h SLA escalates. | `pending_ack` | Sev2, Sev1 past SLA |
| R18 audit chain | platform | A10's latest daily verification result is `ok` and no older than 26 h | `audit_chain_broken` | Sev1 |

- Every rule function has the signature `(inputs) -> RuleResult`, where:
  - `RuleResult = {rule, status, findings: Finding[]}`;
  - `Finding = {rule, type, severity, subject_ref, delta_units?, delta_cents?, values {registry?, internal?, chain?, bank?, ledger?}, in_flight_units?}`.
- `likely_cause` and `proposed_fix` are attached to a break after grouping, from a fixed table per type. Examples:
  - `qty` with chain > register: "mint confirmed before the registry ack" / "wait for ack or raise a register adjustment (6E)";
  - `cash` positive: "unmatched receipt not yet posted" / "match the bank line in Treasury";
  - `unknown_holder`: "transfer to an unregistered wallet or a stale register extract" / "freeze the wallet (SGL+R) and request a fresh extract through the 4E re-sync".
- The table is data, so REGO can refine the wording without a code change.
- Stubbed rules are visible. The run's `rule_results_json` lists R9, R15 and R16 as `skipped_stub`, and the read model shows them as "not in this phase". If a phase flag (A08) turns P2 or P3 on without a real implementation, the run becomes `incomplete` rather than silently clean.

**Break-set grouping (pure, in `packages/domain/src/recon/group.ts`)**

One real-world drift must open exactly one break. The rules above can each see the same drift, so the builder turns findings into breaks by these rules, applied in order:

1. **Unit breaks per vehicle.** All `qty` findings of R2, R3 and R7 for one vehicle fold into **one** `qty` break per vehicle, with `subject_ref` = the vehicle.
   - The break carries holder lines `{wallet, investor_id, registry, internal, chain, delta}`, plus a `residual` line for any supply delta the holder lines and unknown holders do not explain.
   - `delta_units` is the signed supply delta, register minus chain. For Gympie Road: registry 9,998, chain 10,000, `delta_units = −2`, one holder line.
   - R3a findings for the same vehicle fold into that break as `internal` values, and the break's `rules[]` lists R3a.
   - When a vehicle has R3a findings but registry = chain on every line (only RealX's copy differs), the single break is typed `register_divergence` instead of `qty`.
2. **Unknown holders.** Each unknown on-chain wallet is its own `unknown_holder` break (`subject_ref` = the wallet). Its balance is subtracted from the vehicle's supply delta before rule 1 decides whether a residual remains. Any R4 finding on the same wallet (`stale_registration`, `missing_registration`) is attached to the break as a line and opens no break of its own. An R7 `unknown_execution` finding whose transfer target is that wallet is attached too.
3. **Freeze.** All `freeze_drift` findings of a vehicle fold into one `freeze_drift` break per vehicle, with holder lines. A frozen-amount delta that R3 also reports as a balance delta on the same wallet stays in the `qty` break and is not repeated.
4. **Cash.** One `cash` break per client-money bank account (platform run), and one per offering for the escrow leg (vehicle run). A cash finding on the escrow account whose whole delta is explained by one offering's escrow-leg finding is kept on the offering break only.
5. **Everything else.** R1, R4 (wallets not already attached), R5, R6, R8, R11-R14, R17 and R18 findings group by `(vehicle or platform, type, subject_ref)`.
6. The `dedup_key` of a break is `scope + vehicle_or_account + type + subject_ref` after grouping. The break carries the list of contributing rules.

The grouping rules are part of the frozen spec. The oracle's expected break sets are stated after grouping.

**Break types outside the rule table**

| Type | Raised by | Severity | Owner | Halt effect | Closes when |
|---|---|---|---|---|---|
| `register_internal` | the register self-check (header ≠ Σ entries, or lots ≠ units) | Sev1 | REGO | as Sev1 | a completed run on a version that passes the self-check |
| `source_unavailable` | an `incomplete` run (subject: the missing source) | Sev2; escalates to Sev1 at 24 h | REGO (register or chain), TRSY (bank or ledger), ENGO (auxiliary provider) | refuses every pre-flight for the vehicle (or, for a bank source, for the vehicles mapped to the account); as Sev1 after escalation | the next completed (not incomplete) run over the vehicle or account |
| `replay_mismatch` | a replay whose hash differs | Sev1 | ENGO, with SECO informed | as Sev1 | a completed replay of a new run over the current keys that reproduces its own hash, plus a clean run; a mismatch on history is also logged as a security finding for SECO |
| `unknown_execution` | R7, or `raiseExternalBreak` from A28 | Sev1 | REGO + CCO | as Sev1 | a clean run that no longer produces the finding, and the raising module's `external_conditions` entry for it being resolved |

**Breaks and their lifecycle**

- The `recon_break` row:
  - `id` (`BRK-<yyyymmdd>-<6 base32>`), `scope`, `vehicle_id` (null for platform), `account_ref` (for cash), `run_id` (the first run that saw it), `last_seen_run_id`;
  - `rules[]`, `type`, `severity` (1 or 2), `subject_ref`, `delta_units`, `delta_cents`, `lines_json`, `values_json`;
  - `likely_cause`, `proposed_fix`, `owner_role` (REGO by default; TRSY for `cash`; SECO for `audit_chain_broken`; as in the table above for the extra types);
  - `origin` (`rule` or `external`), `raised_by` (for external breaks), `state`, `alert_id`, `sla_clock_id`, `fix_approval_id`;
  - `opened_at`, `closed_at`, `closed_by_run_id`, `dedup_key`, `previous_break_id`.
- **Local lifecycle `BREAK`** (not one of SM-00..SM-13):
  - transitions `open -> investigating -> fix_pending -> closed`, plus `open -> closed` and `investigating -> closed`, and `open -> escalated -> investigating`;
  - only the system closes a break, and only when a later completed run over the same scope, on current keys, no longer produces a break with the same `dedup_key` (and, for external breaks, the condition is resolved);
  - a human or an agent cannot close, suppress, snooze or delete a break. `POST .../close` does not exist, and the alert type has `closes_by = system`;
  - a break seen again by a later run updates `last_seen_run_id` and does not open a duplicate;
  - a break whose subject drifts again after closing opens a new break with a new id and `previous_break_id`.
- **Severity escalation.**
  - Every break opens a 24 h resolution clock through A13 (`kind: custom`, policy `recon.break.resolve`, elapsed).
  - On breach, the break moves to `escalated` and a Sev2 becomes Sev1 (`pending_ack`, `pending_registration`, `missing_registration`, `stale_registration`, `claim_drift`, `doc_drift`, `valuation_stale` and `source_unavailable` all upgrade). The alert is upgraded in place through A13, and the Sev1 halt effects below apply from that moment.
  - ADM-17's "Oldest break 4h (SLA 24h)" is `max(now − opened_at)` over open breaks against this policy.
- **Alerts.** Exactly one alert per break is emitted through A13 `emit()` when the break opens:
  - `chain.unknown_holder` for `unknown_holder` (Sev1, REGO + CCO);
  - `money.client_money_delta` for `cash` (TRSY + CFO, target ADM-18, the break's severity, `dedup_key` = the break's key). This node is the only emitter of that alert: A16 leaves the bank-versus-ledger delta to R10, and A39 only consumes the result;
  - otherwise `recon.break`, with the break's severity, `subject_ref` = the vehicle, `dedup_key` = the break's key and `target_screen` ADM-17.
  - `sponsor_visible` is true for `qty`, `unknown_holder`, `freeze_drift`, `pending_ack` and `pending_registration` (SPN-08 shows a banner) and false for the rest.
  - When the break closes, the run service calls `resolveBySystem()` with the closing run id as the reason.
- **Fix path.** Wired by later nodes; the contract is fixed here.
  - `POST /v1/recon/breaks/{id}/propose-fix` records a proposed register adjustment or corrective chain call and moves the break to `fix_pending`.
  - A14 turns the payload into a `6E` request (`register.adjust`: REGO -> CCO + CFO, plus the trustee on the wholesale track), or A28 turns it into a V2 custody op. Either way the payload carries `recon_break_id`.
  - The approval id is stored on the break. Approval or execution does not close the break; only the re-run does.
- **Registry re-sync (runbook R4).** This node has **no** re-sync endpoint and never asks A18 for an extract.
  - The re-sync is A18's `POST /register/{vehicleId}/resync`, a `4E` request under action code `register.resync` (REGO maker, head of operations checker, raised through A14). An agent token may only draft it as maker.
  - When the approved re-sync stores a fresh registrar version, A18 emits `register.version.created`, and this node runs the targeted run for that vehicle (see Triggers).
  - The break panel links to the A18 flow. This node adds no single-person path to the registrar.

**Halt effects (G10 and the refusals)**

- The module keeps `recon_halt` rows as its detail table: `id`, `scope`, `vehicle_id` or `account_ref`, `cause = recon_break`, `break_ids[]`, `a19_halt_id`, `set_at`, `cleared_at`, `cleared_by_run_id`.
- It exposes `assertClear(vehicleId | null, action, opts?: {correctiveFor?: breakId, approvalId?})`, `isHalted(vehicleId)` and `preflight({vehicleId, purpose})`.
- `assertClear` either throws `ReconNotClear {vehicleId, action, breakIds, sinceAt}` or returns the last clean run id.
- **Blocking matrix** (Y = refused):

| Action | Vehicle Sev1 open (any type) | Vehicle Sev2 open | Open `cash` or `source_unavailable` (bank) on an account mapped to the vehicle |
|---|---|---|---|
| `mint`, `burn`, `forced_transfer`, `recovery`, `unlock` | Y (except a corrective op whose approval carries `recon_break_id` of an open break on that vehicle, passed as `correctiveFor`; and a mint whose units are `in_flight` for its approval id) | — | — |
| `allocation`, `escrow_release`, `distribution_approval`, `distribution_execution` | Y | Y | Y |
| `board_open` (P2) | Y | Y | — |
| `payout`, `board_trade` (P2) | Y | — | Y for `payout` |

- **Platform level.** `assertClear(null, 'payout')` is the call A39 makes before releasing withdrawals. It refuses while any platform-scoped Sev1 break is open: `cash` on any client-money account, `audit_chain_broken`, or `source_unavailable` on a bank statement escalated to Sev1. `null` is accepted only with `payout`; any other action with `null` is a programming error (`RECON_VEHICLE_REQUIRED`).
- **Pre-flight.** `preflight({vehicleId, purpose})` runs a targeted run and returns `{clear, runId, breaks[]}`. `clear` is false when the run is incomplete or when the blocking matrix refuses the purpose.
- **G10 severity: a reconciled engineering difference.** The product map's G10 wording, ADM-17's purpose and its mock ("BREAK Sev2 ... Resale halted") say any break halts resale. This node follows the architecture brief §7.3 instead: only an open Sev1 break halts resale for the vehicle (G10 halt, `board_trade` refused). A Sev2 break refuses `board_open` and every pre-flight purpose, and it escalates to Sev1 at 24 h, so resale can continue for at most 24 h with a Sev2 break open. `valuation_stale` halts resale on its own through G20, and R17 `pending_ack` past SLA is Sev1.
  - This is recorded as a proposed, reversible engineering decision. It must be added to `08 Build Plan\Decisions.md` and to the [[Screen Coverage Matrix]] as a reconciled difference. The founder may reverse it by adding `board_trade` to the Sev2 column, a data change in the blocking matrix.
- **G10 wiring (one contract, the same in A19, A20 and A48).**
  - When the first Sev1 break opens on a vehicle, in the same database transaction, this node calls A19's exported `openHalt({vehicle_id, cause: 'recon_break', criteria: 'a completed run over the vehicle with zero open Sev1 breaks', source_ref: recon_halt.id})` and stores the returned `a19_halt_id`. A19's G10 evaluator then reports `halted` with cause `recon_break`, and every gate consumer (resale board A91, sell-screen preview, public property page) sees it without asking this module.
  - When the clean run closes the last Sev1 break, it calls `closeHalt({halt_id: a19_halt_id, cleared_by: svc-recon, reason: run_id})` in the same transaction.
  - This node does not call `registerHaltSource`.
  - When A48 lands, A19's `openHalt`/`closeHalt` forward to A48's formal `Halt` record with the same signature, so this node does not change.
  - A contract test imports A19's real `index.ts` exports (not a fake) and asserts their names and parameter types.
- A halt is never cleared by hand. Each set and clear writes one audit event and one outbox event (see the consumer contract).

**Consumer contract (published here; the consuming nodes align to it)**

| Item | Contract |
|---|---|
| `assertClear` actions | `mint`, `burn`, `forced_transfer`, `recovery`, `unlock`, `allocation`, `escrow_release`, `distribution_approval`, `distribution_execution`, `payout`, `board_open`, `board_trade`, refused per the blocking matrix |
| `preflight` purposes | `mint`, `burn`, `forced_transfer`, `allocation`, `escrow_release`, `distribution_approval`, `distribution_execution`, `board_open` |
| Error | `ReconNotClear {vehicleId, action, breakIds, sinceAt, severity}` from the module. HTTP `preflight` returns 200 `{clear: false}`. Each consumer maps it to its own stable code: A14 `APR_RECON_BREAK_OPEN`, A28 `TOP_BREAK_OPEN`, A38 `SET_RECON_NOT_CLEAR`, A39 `ESC_RECON_NOT_CLEAR` (hold reason `recon_not_clear`), A40 `DST_RECON_BREAK_OPEN` (vehicle breaks) and `DST_CLIENT_MONEY_DELTA` (a `cash` break on a mapped account). |
| Sev2 blocking set | `allocation`, `escrow_release`, `distribution_approval`, `distribution_execution`, `board_open`. A14's precheck and A40's funding check must call `preflight` (not only test for open Sev1 breaks) for these purposes. |
| Outbox events (A09 outbox; all carry `run_id` or `break_id`, `vehicle_id` or `account_ref`, `occurred_at`) | `recon.run.completed {run_id, scope, vehicle_id, trigger, trigger_refs[], result, matched, register_version_id, block_number, statement_ids[], aux_inputs_sha256, result_sha256, open_break_ids[]}`; `recon.run.incomplete {run_id, vehicle_id, missing_source}`; `recon.break.opened {break_id, type, severity, subject_ref, delta_units, delta_cents}`; `recon.break.escalated {break_id, severity}`; `recon.break.closed {break_id, closed_by_run_id}`; `recon.halt.set {halt_id, a19_halt_id, vehicle_id, break_ids[]}`; `recon.halt.cleared {halt_id, a19_halt_id, vehicle_id, cleared_by_run_id}` |
| External breaks | `raiseExternalBreak({vehicleId, type: 'unknown_execution' \| 'role_drift', severity: 1, subject_ref, values, raised_by, evidence_ref})` returns the break id (deduplicated on the key). The raiser supplies the resolution state through the `external_conditions` provider. Used by A28 when an indexer event matches no op. |
| Custody write-back | see Triggers: A15's status interface and A14's internal transition endpoint, never a private stub |

**Triggers and jobs**

- **Nightly (`recon.nightly`), labelled "02:00" as the product map's M26 job.** The register side it needs is A18's registrar extract, fetched at 02:10. So the cycle is created at 02:00 `Australia/Sydney` (a `recon_cycle` row), and each vehicle's run starts when A18 emits `register.version.created` with `source: registry` for that vehicle and date. A sweep at 03:00 runs every vehicle still waiting; one with no registrar version for the date completes as `incomplete` with `missing_source: register`. The platform run (R10 account leg and R18) starts when A17 has published the 00:00 statements for every client-money account, and no later than the 03:00 sweep.
- The cycle covers every vehicle in the `vehicles` provider, pre-issue ones included (they return `not_issued`), in parallel batches of 10, plus one platform run.
- The schedule is a `recon_schedule` row (cron, zone, sweep time, batch size, enabled), registered with the pg-boss cron here and listed by A46 as emitted by this module. The run time is a setting, changeable without a code change.
- The cycle is idempotent: a second start of the same cycle finds the cycle's runs and does nothing, and a partial cycle resumes with the vehicles that have no completed run for the cycle.
- **Daily evidence.** Each cycle's per-scope results and hashes (no PII) live in the `recon_cycle` and `recon_run` tables and are served by `GET /v1/recon/cycles/{date}`. The running app writes no files. `app/evidence/` belongs to the gauntlet and is not written by this module.
- **Targeted runs (`recon.targeted`)** consume these outbox events, using the producers' real names:
  - A18: `register.changed` (emitted with the instruction ack), `register.instruction.rejected`, `register.instruction.pending_sla` and `register.version.created`;
  - A17: `statement.ready`, as published on the outbox after ingest (a platform run for the account);
  - A15: `custody.op.confirmed`, which A28 carries on for token ops.
  - The consumer runs one targeted run for the vehicle named in the event within 60 s. It is keyed so that ten events for the same vehicle inside a minute produce one run with ten `trigger_refs`.
  - A pass criterion checks these subscriptions against the outbox catalogues exported by A18, A17 and A15.
- **Custody op write-back (SM-00 `confirmed -> reconciled`).**
  - When a targeted run triggered by `custody.op.confirmed` completes clean for the op's vehicle, and the op's holding change is visible in all three sources (registrar entry, chain event with the op's approval id, and, for money-moving ops, the ledger), the run service does two things:
    - moves the `CustodyOp` to `reconciled` through A15's exported status interface;
    - posts `reconciled` for the op's approval id to A14's `POST /v1/internal/approvals/{id}/transition` as `svc-recon`, with the run id as evidence.
  - A14 then allows `closed`, and refuses `closed` without that event (`APR_NOT_RECONCILED`).
  - If the run finds a break on the vehicle, the op stays `confirmed`, the break ids are attached to it through A15's interface, and nothing is posted to A14.
- **`preflight({vehicleId, purpose})`** is a synchronous targeted run with a hard budget of 5 s locally for a vehicle of 10,000 holders. The stub source answers from memory; the indexer source of A27 must meet the same budget, or the pre-flight returns `incomplete`, which is a refusal. A pre-flight opens breaks and halts like any other run, so a pre-flight that finds a drift leaves the vehicle halted.
- **`recon.break-watch`** runs every minute. It moves breaks whose 24 h clock breached (A13 reports this through the clock's `on_breach` callback) to `escalated`, upgrades their severity and applies the Sev1 halt. It is idempotent.
- **`recon.stub-source-seed`** (local only) loads the stub chain source and the auxiliary stubs from the A06 seed, so the eight design trusts reconcile as the design shows:
  - the seeded cycle's platform and vehicle runs have `started_at` 23 Sep 2026 06:00 AEST and `finished_at` 06:02;
  - Gympie Road Retail has one `qty` Sev1 break of −2 units (registrar 9,998, internal copy 9,998, chain 10,000), `opened_at` 06:02;
  - Kent Street, Ann Street and Unley Road are `not_issued` with escrow application units 6,400, 2,800 and 9,100. Unley Road is seeded with its allocation locked and no register issue submitted yet, so register 0 and chain 0;
  - Marine Parade is `not_issued` with every leg at zero;
  - Marion Street, Bay Road and Hawthorn Grove are clean.

**HTTP API (all under `/v1`, cursor pagination, idempotency keys on writes, in the OpenAPI document from the first commit)**

- **`GET /v1/recon/summary`** returns `{trusts_total, trusts_matched, trusts_not_issued, trusts_breaking, trusts_incomplete, breaks_open, breaks_open_units, breaks_open_cents, oldest_break_age_ms, sla_ms, last_cycle {started_at, finished_at, state}}`. The formulas, over the vehicles of the last cycle:
  - `trusts_breaking` = vehicles with an open vehicle-scoped break, or mapped to an account with an open `cash` break;
  - `trusts_incomplete` = vehicles whose last cycle run is `incomplete` or missing;
  - `trusts_matched` = `trusts_total − trusts_breaking − trusts_incomplete`, so `not_issued` vehicles whose legs agree count as matched;
  - `breaks_open_units` = Σ |`delta_units`| over open `qty` and `unknown_holder` breaks.
  - With the seed: 8 total, 1 breaking, 0 incomplete, so 7 matched; 4 not issued; 1 break of 2 units.
  - Design strings and the fields they read: "Run 06:00" and ADM-01's "reconciled at 06:00 AEST" read `last_cycle.started_at`; "4h" reads `oldest_break_age_ms` at A06's pinned `AS_OF`.
- **`GET /v1/recon/vehicles`** returns one row per vehicle:
  - `vehicle_id`, `name`, `register_units` (registrar extract), `chain_supply`, `escrow_application_units`, `bank_delta_cents`, `client_money_result` (`matched`, `break`, `not_applicable`), `last_run_at` (the run's `started_at`), `result`, `matched`, `break_summary`, `halted`, `open_breaks[]`. "Break · 2 units" is composed by the UI from `open_breaks[0].delta_units`.
  - Filters: `result`, `halted`, `tenant_id`. Sponsors see their own vehicles only, through row-level security.
  - The public transparency read (PUB-08) is `GET /v1/public/recon`: the same rows minus internal ids, plus `break_opened_at` (the page shows a red result with the date opened).
- **Other endpoints:**
  - `GET /v1/recon/vehicles/{id}`: the last run, its per-rule results, open breaks, in-flight units, halt status;
  - `GET /v1/recon/runs?vehicle=&trigger=&since=` and `GET /v1/recon/runs/{id}` (with `rule_results_json` and all keys);
  - `POST /v1/recon/runs` `{vehicle_id | 'platform', register_version_id?, block_number?, statement_ids?}`: a manual run by REGO or CCO, rate-limited to 10 per vehicle per hour for tokens;
  - `POST /v1/recon/runs/{id}/replay`;
  - `POST /v1/recon/preflight` `{vehicle_id, purpose}` (internal and staff);
  - `GET /v1/recon/breaks?state=&severity=&vehicle=&type=` and `GET /v1/recon/breaks/{id}`;
  - `POST /v1/recon/breaks/{id}/investigate` (assign to self; reason optional);
  - `POST /v1/recon/breaks/{id}/propose-fix` (reason and payload mandatory);
  - `GET /v1/recon/halts` and `GET /v1/recon/cycles/{date}`;
  - `GET /v1/public/recon` and `GET /v1/public/recon/{vehicle}`, which feeds the property page's proof band `{register_units, chain_supply, matched, checked_at, client_money_result, client_money_checked_at}`. `checked_at` is the last clean run's `started_at`.
  - There is no re-sync route and no close route.
- **Permissions: scopes are A03 matrix cells.** Each route declares `requiredScope` as an A03 action id in module M11 (scope id = action id):
  - `M11.recon.read`, `M11.recon.run`, `M11.recon.investigate`, `M11.recon.propose_fix` (the draft action of the `register.adjust` maker-checker row);
  - A18's `register.resync` draft action is used from the break panel;
  - any of these ids missing from A03's `actions.ts` is added in A03's data, not invented here.
  - Token scopes are checked by A12 through A03's `validateTokenScopes`.
- **Cells:**
  - REGO, CCO, CFO, SECO and CEO: full read;
  - REGO: `run`, `investigate`, `propose_fix`; CCO: `run`, `investigate`; TRSY: `investigate` and `propose_fix` on `cash` breaks only;
  - TP-RE: read, constrained to `own-vehicles` (the vehicles it is trustee or RE of). TP-REG is an adapter principal with `signsIn: false` in A03 and has no cells; its register acts arrive through A18's adapter;
  - TP-AUD and TP-OBS: read;
  - SP-OWN, SP-FIN and SP-IR: read of their tenant's vehicle rows and open sponsor-visible breaks;
  - investors and visitors: the public reads only.
- **Agent parity (D04, G22).**
  - Every staff and partner endpoint is available to that person's token with the same M11 cells, audited as the token on behalf of the person. A TP-RE token has read-only parity.
  - `propose-fix` from a token creates the proposal, but A14 refuses the resulting `6E` approval to any token.
  - A token can never close a break, because nobody can.
- **Empty and error states:**
  - no vehicles issued yet: `trusts_total` with every row `not_issued` and `breaks_open: 0`;
  - `GET /v1/recon/vehicles/{id}` for a vehicle with no run yet: `result: null, last_run_at: null, halted: false`;
  - a manual run while another run for the vehicle is in flight: 409 `run_in_progress` with its id;
  - a run that cannot get a source: 200 with `state: incomplete` and `missing_source`;
  - `preflight` for a halted vehicle: 200 `{clear: false}`, never an exception at the HTTP layer; consumers map it per the contract table;
  - an unknown vehicle: 404 for everyone, including other tenants (no existence leak);
  - a tolerance or rounding parameter passed to any endpoint: 400 schema error.

**Audit**

- One `AuditEvent` through A10 per:
  - run completion (`recon.run.completed` or `recon.run.incomplete`, with all keys and the result hash);
  - break transition (`recon.break.opened`, `recon.break.investigating`, `recon.break.fix_proposed`, `recon.break.escalated`, `recon.break.closed`);
  - halt change (`recon.halt.set`, `recon.halt.cleared`);
  - custody write-back (`recon.custody.reconciled`).
- Repeated sightings of an open break write none.
- The actor is `svc-recon` for jobs, and the human or token (with `on_behalf_of`) for manual actions.
- The design's ADM-31 row "06:02 Recon job Detected break Gympie Road Retail Alert" is the seed's `recon.break.opened` event at the break's `opened_at`.

## Owned paths (other nodes do not edit these)

- <workspace>\app\services\api\src\modules\recon\
- <workspace>\app\packages\domain\src\recon\
- <workspace>\app\services\worker\src\jobs\recon\

## Design references

- **`design:adRecon grid`** in `<workspace>\design\RealX App.dc.html`: screen key `adRecon`, product-map ADM-17, admin dark shell; the screen data is the `adRecon:` definition at line 885 of the same file.
  - Subline: "Trustee register, token supply and client-money ledger compared daily. The register is the legal record."
  - KpiGrid: "Trusts matched 7 of 8 (Run 06:00)", "Breaks open 1 (2 units)", "Oldest break 4h (SLA 24h)".
  - DataTable Trust / Register / Token supply / Client money / Last run / Status, with seven rows: six green "Matched", and "Gympie Road Retail 9,998 / 10,000 / Matched / 06:00 / Break · 2 units" in alert red `#A3312A`. Open offers show "6,400 (escrow)" with token supply 0.
  - This node's `GET /v1/recon/summary` and `GET /v1/recon/vehicles` are the read models for those tiles and columns. The seed reproduces the numbers through the summary formulas above. The eighth trust (Marine Parade) is present as `not_issued`, and the escrow figure moves to `escrow_application_units`, so the register column of an open offer reads 0 (product-map QA items 13 and 14 resolved in data).
  - The screen itself is drawn by [[A73 - Console- rule packs and threshold monitors, register and reconciliation]]. The break panel (likely cause, proposed fix, holder lines, approvals, resale-halt flag, unknown-holders strip) is not in the prototype; [[A51 - Design extension B- operator console screens not in the prototype]] specifies it from the `ReconBreak` fields fixed here.
- **`design:adCmd register-break alert`** (screen key `adCmd`, ADM-01, line 880 of the same file):
  - The alert row "Register break · Gympie Road Retail / Registry file shows 9,998 units; token supply 10,000 / High / 06:02" deep-links to `adRecon`. It also covers the "Open alerts 3 (1 high)" tile and the subline "all figures reconciled at 06:00 AEST".
  - The row is the `recon.break` alert this node emits through A13, at the break's `opened_at` (06:02). Its `title` and `detail` are composed from the break's values (`Registry file shows {registry} units; token supply {chain}`).
  - The subline reads `summary.last_cycle.started_at` (06:00).
  - Rendered by [[A66 - Console- command center, alerts centre, approvals inbox, audit log]].
- **`design:prop proof band`** (screen key `prop`, PUB-05 property page, investor and visitor roles). Anchor: the text "Register and chain agree" at line 277 of `RealX App.dc.html`.
  - The dark card "Register and chain agree" in Newsreader 22, with the Geist Mono subline "Trustee register {N} units = {N} tokens on-chain · checked 23 Sep 2026 06:00 AEST" in `#B9B4AA`.
  - The MATCHED pill (`#E3EEE8` on `#1F5E4B`).
  - The proof row "Client-money reconciliation · Trust bank ledger matches distributions paid · 23 Sep 2026".
  - Those strings come from `GET /v1/public/recon/{vehicle}` (`register_units`, `chain_supply`, `matched`, `checked_at` = the last clean run's `started_at`, `client_money_result`).
  - When `matched` is false, the band shows the break state in alert red with the date opened. This is a product map PUB-08 rule that the prototype does not draw; [[A50 - Design extension A- investor and sponsor screens not in the prototype]] specifies it.
  - Rendered by [[A54 - Public site- home, investments, property page, sponsor profile, fees]], and as the transparency table by [[A55 - Public site- how it works, returns calculator, transparency, help and documents]].
- Tokens: cream `#F5F2EC`, ink `#17191C`, green `#1F5E4B`, brick `#9A4524`, alert red `#A3312A`, with Geist Mono for units, hashes, ids and times.
- **Schedule label.** The product map fixes the nightly job at 02:00 AEST; the prototype's mock shows "Run 06:00" and "reconciled at 06:00 AEST".
  - Engineering follows the product map: the cycle is created at 02:00, and vehicle runs follow the 02:10 registrar extract.
  - Every "06:00" string in the design is rendered from a run's real `started_at`. The seed's pinned cycle is stamped 23 Sep 2026 06:00 AEST (start) and 06:02 (finish and break), so the design rows, the A06 seed and the A13 seed match exactly.
  - The coverage matrix records this as a reconciled difference, not a defect.

## Requirements covered

- **Product-map screens:** none owned. These read this module:
  - ADM-17 (A73);
  - the ADM-01 and ADM-31 rows (A66);
  - the PUB-05 proof band and PUB-08 transparency (A54, A55);
  - SPN-08's "Last reconciled" subline and break banner ([[A64 - Sponsor console- register, distributions, property reporting, valuations, announcements]]);
  - POR-02's "on-chain balance and register match" line and its "we're checking this with the trustee" mismatch state ([[A47 - Portfolio, holdings and transactions read models with statements]]);
  - ADM-18's `money.client_money_delta` alert and "Reconciled" status (A39).
  - "Done" means every field, count, pill, date and deep link those screens need exists in the API and the SDK, with the seed reproducing the design values, so the UI nodes add no logic.
- **Journeys:** none owned. These steps pass through this node:
  - J04 steps 10 and 12 (register first, then reconcile supply = register = allocations);
  - J07 step 6 and J09 step 9 (register updated after settlement or recovery, and confirmed by recon);
  - J17 steps 1-2, J19 step 2 and J20 step 4 (the 02:00 reconciliation review);
  - J01 step 8 (a visitor checks reconciliation).
  - "Done" means each of those steps has a run, break or read call it can assert on, exercised in this node's integration test with fixture events standing in for the later modules.
- **Gates:**
  - **G10** (a break halts resale for the trust): "done" means a Sev1 break opens the A19 halt through `openHalt` in the same transaction that opens the break; A19's G10 evaluation reports it; a clean re-run closes it through `closeHalt`; nothing else clears it. The Sev1-only rule is the reconciled difference recorded above.
  - **G13** (register adjustments are approved): "done" means the only fix path is `propose-fix`, which produces a payload for a `6E` approval and never changes a snapshot, a run result or a break state by itself. The module has no write that alters register, chain or ledger data, and no path to the registrar outside A18's 4E re-sync.
- **Modules:**
  - **M11** Register and reconciliation: "done" means the D05 behaviours of the requirements catalogue are implemented and tested here: zero tolerance; register entry precedes minting (in-flight units); token delivery not complete until reconciliation passes; a break opens an alert, halts resale, shows red on PUB-08 and a banner on SPN-08; a registry ack is required before any on-chain move (R17); runbook R4 goes through A18's 4E re-sync. So are the non-functional requirements N03 (zero-tolerance three-way recon), N12 (100 trusts x 10,000 holders inside the window) and N13 (idempotent, replayable jobs).
  - **M26** jobs: "done" means the nightly cycle, the 03:00 sweep, the targeted consumer and the break watcher are pg-boss jobs with schedules that A46 lists, idempotent under re-run, and visible in the integrations-and-jobs read model.
- **State machines:**
  - **SM-00** (privileged action): "done" means:
    - a custody op's `confirmed -> reconciled` transition is driven only by this module's clean targeted run, through A15's status interface and A14's internal transition endpoint (real modules, both dependencies);
    - A14's request reaches `reconciled` and then `closed` only after that clean run;
    - an op `confirmed` with a break stays `confirmed`, carries the break ids, and A14 refuses `closed` with `APR_NOT_RECONCILED`;
    - the invariant "a confirmed transaction the register does not reflect is a break, not a success" has a named test.
  - **SM-07** (holding): "done" means each transition has one named fixture and the rule that checks it on the next targeted run:
    - `[*] -> eligible_empty` (R4);
    - `eligible_empty -> active` (R3, R7, R17);
    - `active -> locked` and `locked -> active` at the lock-in date (R3b, including the lapse on the day);
    - `active -> partially_frozen`, `partially_frozen -> active` and `partially_frozen -> frozen` (R3b);
    - `active -> frozen` and `frozen -> active` (R3b);
    - `frozen -> recovered_out` and `active -> recovered_out` (R3, R7 with a forced-transfer or recovery approval id, R17);
    - `active -> eligible_empty` (R3);
    - `eligible_empty -> deregistered` (R4);
    - plus the negatives: a holding on-chain without a register entry is `unknown_holder`, and a frozen amount off by one unit is `freeze_drift`.

## Gauntlet loop for this node

- **Specify.** The frozen spec is:
  - this note, including the grouping rules and the blocking matrix;
  - the R1-R18 table of the architecture brief §7, with this note's R3a and R3b additions;
  - the D05 section and N03/N12/N13 of the requirements catalogue;
  - the `adRecon`, `adCmd` and `prop` regions above;
  - the snapshot and `AuxInputs` types as written here;
  - the real interfaces of A18 (versions, events), A17 (`getStatement`, `statement.ready`), A16 (`snapshot`), A15 (status interface, `custody.op.confirmed`), A14 (internal transition endpoint), A19 (`openHalt`, `closeHalt`) and A13.
  - The node QA then writes the hidden oracle to `<workspace>\gauntlet\hidden\A20\`: synthetic input sets with planted drifts, the expected grouped break sets (type, severity, subject, delta, lines), the expected halt effects per action, and the closing-only-on-clean-re-run sequences.
  - Any change to a rule's meaning, a severity, a grouping rule or the blocking matrix after the freeze is a spec defect, not a fix.
- **Build (Grok 4.7 xhigh, split by the planner).** Four parallel sub-agents on disjoint files, then one integration pass:
  1. `packages/domain/src/recon/`: snapshot and aux types with zod schemas, the rule functions with the stubs, the grouping builder with dedup keys, the account-to-vehicle mapping, the cause/fix table, the canonical result hash. Property tests with fast-check: a clean input set yields no breaks, and any single perturbation yields exactly one break after grouping.
  2. The API module: Drizzle schema and migration; the run service with snapshot and aux acquisition; the `ChainSnapshotSource` and aux-provider registries with their stubs; the `BREAK` transition table; `assertClear`, `preflight` and the A19 `openHalt`/`closeHalt` calls; `raiseExternalBreak`; A13 emit/resolve and clock calls; the A15 and A14 write-back; routes, zod schemas, permission checks against A03 cells; OpenAPI generation and an SDK compile check.
  3. The worker jobs: the cycle with its registry-extract trigger, 03:00 sweep, batching and resume; the targeted consumer with per-vehicle coalescing; the break watcher; the local seed loader. All run on the injectable clock.
  4. Fixtures: a scenario kit that produces input sets from the seed's eight trusts and can plant every break type through A18's sandbox registrar drift option, the stub chain source, A17's sandbox statements and the aux stubs, plus the outbox emitters standing in for later producers, using the real event names.
  - Integration: an end-to-end test runs a clean cycle over the seed, plants the Gympie Road drift, and sees the one break, the alert, the A19 halt and the audit rows; it then fixes the drift in the fixture, re-runs and sees the close. A 100 x 10,000 synthetic cycle is timed on the build machine.
- **Challenge (node QA, Opus 5.5 high).** Runs:
  - the module tests, then the hidden oracle against the pure functions and the run service;
  - the guard for every cell of the blocking matrix;
  - reproducibility (three runs on the same keys, byte-identical results);
  - the coalescing of targeted triggers and the escalation at 24 h under the injectable clock;
  - permission tests for every staff role, TP-RE, TP-AUD, TP-OBS, SP-*, an investor and a token per role;
  - audit-event counts through A10's verifier;
  - the tolerance lint and the `Date.now()` grep;
  - OpenAPI and SDK presence of every endpoint;
  - a design-fidelity read of the seed against the three regions;
  - a PII scan of `values_json`, `lines_json`, `aux_snapshot`, `likely_cause`, alert titles and the cycle read (the fixture plants an investor name, and the node QA expects it to be absent).
- **Gate.** A pass unlocks the seven dependents. Max three fix loops.
- **Escalation trigger.** Reopen the spec and ask the user when any of these happens:
  - a third failure on the same assumption (for example whether a Sev2 break should block a pre-flight, whether an incomplete run may count as clean, or how escrow applications appear in the register column);
  - any fix that needs a change outside the owned paths (A18's version shape or events, A17's statement or outbox event, A16's `snapshot`, A19's `openHalt`/`closeHalt`, A15's status interface, A13's clock callback or alert catalogue);
  - any finding that the design's rows cannot be reproduced without a tolerance.

## Node-specific challenges (node QA)

- **Two units, three tokens and one cent, planted in one night (money).**
  - The setup:
    - the registry sandbox's drift option makes Gympie Road's registrar extract show 9,998 units, with the internal copy also at 9,998 and 10,000 tokens on-chain;
    - a wallet `0x9e…a2` holding 3 Bay Road tokens is on no register entry but is on the identity registry (chain supply 10,003);
    - the wallets client-money statement sits one cent above the ledger.
  - After grouping, the cycle must open exactly three breaks:
    - `qty` Sev1 on Gympie Road (subject the vehicle, `delta_units −2`, one holder line, values 9,998 / 10,000);
    - `unknown_holder` Sev1 on Bay Road (subject the wallet, 3 units, with the R4 `stale_registration` finding attached as a line and no separate `qty` or R4 break, because the wallet explains the whole supply delta);
    - `cash` Sev1 on the wallets account in the platform run (delta +1 cent).
  - Expected halts:
    - Gympie Road and Bay Road are refused for every action in the Sev1 column;
    - the wallets account maps to no vehicle, so the cash break refuses `assertClear(null, 'payout')` and leaves the vehicles' pre-flights alone;
    - a variant planting the cent on the offer-escrow account must instead refuse `allocation`, `escrow_release`, `distribution_approval`, `distribution_execution` and `payout` for exactly the vehicles mapped to it (Kent Street, Ann Street and Unley Road).
  - Exactly one `money.client_money_delta` alert and no `recon.break` alert are emitted for the cash break.
  - Fails: a fourth break, a missed one, a Sev2 where a Sev1 is expected, eight cash breaks, or a `cash` result of "matched within tolerance".
- **Closing by hand, by agent, by approval or by expiry (compliance, G13).** With the Gympie Road break open, the node QA tries every route to make it go away without a clean re-run:
  - an HTTP call to a `close` or `resolve` path;
  - resolving the `recon.break` alert in A13;
  - approving and executing the `6E` fix proposal without a re-run;
  - letting the 24 h clock breach;
  - restarting the worker;
  - re-running with a stale register version that pre-dates the drift.
  - The break must stay `open` or `escalated` throughout and the A19 halt must stay open. Only a completed run over the current version, block, statements and aux inputs that produces no break with the same dedup key may close it. A re-run with an explicitly older `register_version_id` must complete but must not close the break: a clean result on old keys is not a clean state.
- **No single-person path to the registrar (compliance, maker-checker).** REGO alone, and a REGO agent token, try to obtain a fresh registrar extract through every A20 route and SDK method. None exists. Through A18's re-sync, the only result is a `4E` request in `submitted`, and no `register.version.created` fires until the head of operations approves. A token cannot approve it.
- **Mint that lands between approval and signing (money, SM-00).**
  - A mint for Unley Road is prechecked clean (its register issue acked, units `in_flight`). Then the fixture drops a `qty` drift on another holder into the registrar extract before the custody quorum signs. The consumer's second `assertClear` before broadcast must refuse with `ReconNotClear`.
  - If the fixture instead confirms the chain transaction first, the targeted run must leave the custody op `confirmed` with break ids attached through A15, and A14's request must refuse `closed` with `APR_NOT_RECONCILED`.
  - After the drift is fixed and a clean run completes, the op becomes `reconciled` and the request can close.
  - Fails: an implementation that only checks at precheck, that marks an op reconciled while a Sev1 break is open on its vehicle, or that treats in-flight units older than 24 h as clean.
- **The eighth trust and the escrow that looks like units (design fidelity).** With the standard seed:
  - `GET /v1/recon/summary` must return `trusts_total: 8, trusts_matched: 7, trusts_not_issued: 4, trusts_breaking: 1, trusts_incomplete: 0, breaks_open: 1, breaks_open_units: 2`, and `oldest_break_age_ms` consistent with an 06:02 `opened_at` at A06's pinned `AS_OF`;
  - the Kent Street row must report `register_units: 0`, `escrow_application_units: 6400`, `chain_supply: 0`, `result: not_issued`, `matched: true`;
  - the proof-band read for Bay Road must return `10000 = 10000`, `matched: true`, `checked_at` = 06:00 (the clean run's `started_at`).
  - Fails: a seed that puts 6,400 on the register before issue, hides Marine Parade, stamps the run at 02:00 while the design shows 06:00 without the recorded difference, or reads "06:00" from `finished_at`.
- **Reproducibility under a source that rewrites history (security).**
  - After a clean run, the stub chain source is re-seeded so that block N now shows a different balance for one holder. `POST /v1/recon/runs/{id}/replay` must raise a `replay_mismatch` Sev1 break (owner ENGO) and must never overwrite the original run's results.
  - A second case alters the stored `aux_snapshot` in a test copy; the replay must detect it through `aux_inputs_sha256`.
  - Separately, a token scoped to `M11.recon.read` only must be refused on `POST /v1/recon/runs`, and a REGO token must be rate-limited on manual runs at the eleventh call in an hour, with 429 and a `Retry-After`.
- **A run that cannot see one leg (compliance).** The registry sandbox is told to withhold the extract.
  - The 03:00 sweep must complete the vehicle's run as `incomplete` with `missing_source: register`.
  - It must open a `source_unavailable` Sev2 break that escalates to Sev1 at 24 h, and refuse every pre-flight for the vehicle meanwhile.
  - It must leave the previous clean-run id untouched: the proof band keeps showing the last real check with its date, not "matched" as of now.
  - The next completed run closes the break.
  - Fails: an implementation that treats two agreeing legs as a pass, or updates `checked_at` on an incomplete run.

## Pass criteria

- **Oracle.** The hidden oracle in `<workspace>\gauntlet\hidden\A20\` agrees with the pure functions and the grouping builder on 100 % of its cases, three runs in a row, with byte-identical `rule_results_json` and `result_sha256` on every run. The cases:
  - at least 40 planted-drift input sets across every break type of R1-R8, R3a, R3b, R10-R14, R17, R18 and the extra types;
  - at least 20 clean sets, including edge cases: zero holders, one holder with all units, frozen amounts, a lot whose lock-in lapses on the day, an open offer with escrow only, a vehicle with a burn, units in flight.
- **fast-check property tests.**
  - For 10,000 generated clean input sets: no break.
  - For 10,000 single perturbations: exactly one break after grouping, with the type the perturbation maps to. The perturbations: one unit moved to one holder's chain balance with supply adjusted, which gives `qty`; one unit on the supply header only, `qty` with a residual line; one cent on one statement, `cash`; one extra on-chain holder, `unknown_holder`; one changed role address, `role_drift`; one changed params hash, `rule_drift`; one unit of frozen amount, `freeze_drift`; one unit on the internal copy only, `register_divergence`.
  - The same property is run against the stub chain source through the run service, not only the pure functions.
- **Halt guard.**
  - A generated matrix of {no break, Sev2 open, Sev1 open, Sev2 escalated to Sev1, cash break on a mapped account, cash break on an unmapped account, closed} x the twelve actions, at vehicle and at platform level, returns exactly the refusals of the blocking matrix, including the `correctiveFor` and in-flight exemptions.
  - A19's G10 evaluation for a halted vehicle reports `halted` with cause `recon_break`, and reports clear one transaction after the closing run.
  - A contract test against A19's real `openHalt`/`closeHalt` exports passes.
  - Every set and clear leaves exactly one audit event and one outbox event.
- **Break lifecycle.**
  - Every allowed `BREAK` transition passes, and every generated forbidden pair returns 409.
  - No route, job or SDK method closes a break (and the OpenAPI document has no such operation for the MCP server to expose).
  - The only closing path is the run service on a clean run over the current keys; a re-run on older keys never closes.
  - Dedup produces one break for repeated sightings across at least 30 consecutive runs.
  - `raiseExternalBreak` dedups and closes only as the table says.
- **Jobs under the injectable clock.**
  - The cycle over the seed completes with one run per vehicle (8) plus one platform run. Vehicle runs start on the registrar's `register.version.created`.
  - The cycle is a no-op when started twice for the same cycle, and resumes correctly after a simulated crash at vehicle 4 of 8.
  - Ten targeted events for one vehicle within 60 s produce one run with ten trigger refs.
  - The 24 h escalation fires within one watcher tick and upgrades the A13 alert in place.
  - The `svc-recon` service account has zero caps and no custody role.
  - No file is written under `app/`.
- **Subscriptions.** Every event name the targeted consumer subscribes to exists in the outbox catalogue exported by A18 (`register.changed`, `register.instruction.rejected`, `register.instruction.pending_sla`, `register.version.created`), A17 (`statement.ready`) and A15 (`custody.op.confirmed`). A name missing from a producer's catalogue fails the build.
- **Performance on the build machine.**
  - A targeted run for a vehicle of 10,000 holders completes in under 5 s against the stub source.
  - A synthetic nightly cycle of 100 vehicles x 10,000 holders completes in under 10 minutes.
  - `GET /v1/recon/vehicles` pages in under 200 ms with 1,000 vehicles seeded.
- **Read models and design fidelity.** With the standard seed:
  - `summary` returns the figures of the design challenge above;
  - the vehicle rows reproduce the seven `adRecon` rows, plus Marine Parade `not_issued`;
  - the A13 alert produced by the seed matches the `adCmd` row text and its 06:02 time;
  - `GET /v1/public/recon/{bay-road}` returns the proof-band strings;
  - the ADM-31 audit row "Recon job Detected break Gympie Road Retail" exists as `recon.break.opened` by `svc-recon` at 06:02.
- **Permissions and agent parity.**
  - Matrix-generated tests cover every staff role, SP-OWN, SP-FIN, SP-IR, TP-RE (own vehicles, and 404 for others), TP-AUD, TP-OBS, an investor and a visitor for every endpoint.
  - Every `requiredScope` is an A03 action id, and a token's scopes pass A03's `validateTokenScopes`.
  - A token with M11 cells behaves exactly as its human on every case, is audited with `on_behalf_of`, and is rate-limited on manual runs.
  - The public endpoints carry no internal ids, wallet-to-person links or PII.
- **Hygiene.**
  - Every endpoint is in the generated OpenAPI document with an `operationId` and the tags the MCP server is generated from, and `packages/sdk` compiles with typed clients. [[A104 - MCP server generated from the OpenAPI document]] and [[A100 - Agent-parity suite- every UI action has an API and MCP equivalent under the same gates]] check the MCP catalogue from that document.
  - `npm run verify` finds no `Date.now()`, no argument-less `new Date()`, no floating-point money or unit arithmetic, and no `tolerance`/`epsilon`/`rounding_allowance` symbol in the owned paths.
  - A10's chain verifies after the full suite, with the audit-event count equal to the runs, transitions, halts and write-backs performed.
  - Line coverage of the three owned paths is at least 90 %, and every rule function and the grouping builder are at 100 % branch coverage.

## Human gates touched

- None directly. The node runs on the sandbox registrar (A18), the sandbox bank statements (A17 into A16), the sandbox custody adapter (A15), the in-memory chain source, the aux stubs and the injectable clock.
- Later gates re-run this node's oracle rather than change it:
  - **HG-ACCOUNTS:** a real registrar extract format and a real bank statement feed must pass the A18 and A17 adapter conformance suites; after that, the A20 oracle inputs are replayed through the live sandboxes.
  - **HG-LEGAL:** counsel's view on the register-of-record clause and on the guardian model changes data on the vehicle row, not the comparison rules.
  - **HG-LAUNCH:** the transparency page's "audited metrics only" rule needs the auditor role (a standing human role) to certify the published totals before launch.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
