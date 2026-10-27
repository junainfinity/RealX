---
title: "A46 - Job scheduler, integrations health and webhooks"
created: 2026-09-24
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am3]
node: A46
milestone: AM3
lane: "Platform"
oracle: yes
human_gate: none
---

# A46 - Job scheduler, integrations health and webhooks

**The operational backbone that ADM-32 shows and every other module leans on: one catalogue of every scheduled and event-triggered job on the platform (register snapshot 00:00, reconciliation 02:00, sanctions screening 03:00, claim expiry, certificate and attestation jobs, payout execution as an event-triggered row, and every schedule other modules declare through A09's `registerJob`), all fired through one zone-aware, calendar-aware due-instant calculator on the injected clock with correct Australia/Sydney daylight-saving behaviour, as idempotent, replayable jobs; a run history and a dead-letter list with idempotent replay and a Sev3 alert; one `Integration` row per adapter with health, latency, environment (sandbox or live) and a versioned configuration whose changes go ENGO to SECO under maker-checker; and signed, replay-safe webhook subscriptions for investors, sponsors, partners and staff that carry their bound token's scopes, re-check scope and visibility before every attempt, and stop the moment that token is revoked.**

**Build status:** not started. **Milestone:** AM3 (Business services). **Lane:** Platform. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 10-16 agent-hours (build, challenge and verify loops). Raised from 4-8 by the 24 Sep 2026 plan review: now an oracle node, with the scheduler, integrations health and signed webhooks with visibility checks.

**Depends on:** [[A09 - API and worker skeleton with OpenAPI-generated SDK]], [[A20 - Three-way reconciliation engine with zero tolerance]], [[A27 - Chain adapter, deployment manifests and indexer]], [[A13 - Alerts and SLA-clocks service]], [[A12 - API tokens, scopes, agent principals and mandates for every user type (G22)]]
**Unlocks:** [[A66 - Console- command center, alerts centre, approvals inbox, audit log]], [[A76 - Console- regulatory calendar and data room, sponsor performance, integrations and jobs]], [[A99 - Developer surface- API versioning, reference docs, per-user sandbox and SDK-MCP packaging]]

**Oracle rule:** an oracle node (`oracle: yes`, decided on 24 Sep 2026 by the plan fixer under D04's rule that oracles go wherever money or authority is involved: the money dead-letter discard, the webhook signing and replay window, and the visibility checks that decide which of a person's events an agent's webhook may receive all pass through here; the founder may reverse it with a one-line frontmatter change and a regenerated graph). No money, supply or holdings move inside this node: the jobs it fires belong to the modules that own them and keep their own gates, and every two-person act here is decided by A14 (the approvals engine), not by this module. The node QA writes these vectors as hidden oracle tests to `<workspace>\gauntlet\hidden\A46\` before the build starts ([[App Gauntlet]] §2): the daylight-saving and calendar-rule cron vectors (at least 40 instants), the two-worker single-fire probe, the dead-letter replay counter, the webhook signature and replay-window vectors, the token narrowing, hold and revocation race, the visibility-loss race, the hostile-URL egress list with its flipping DNS fixture, and the configuration allow-list cases. The builder never reads that folder. The tests are published into `app/` after the gate, so A76, A99 and the AM8 journeys can re-run them as regression.

This note is not legal advice. Nothing here decides which vendor, chain or network RealX uses in production; every "live" value is a reserved word that no code path in this build can select (A08), and a configuration change that would reach a mainnet or a vendor endpoint is refused before it is even proposed.

## Build brief

**1. What this node reuses and what it adds (no duplication of A04, A08, A09, A12, A13, A20, A27)**

- Module `integrations` in the API (`services/api/src/modules/integrations/`) and the worker folder `services/worker/src/scheduler/`, following A09's module pattern: a public `index.ts` (the only import other modules may use, enforced by the boundary lint), zod schemas feeding the OpenAPI document from the first commit through `stubRoute` so `packages/sdk` types and the MCP tool catalogue exist before the engine passes, Drizzle schema and migration in the module's own folder registered with A04's per-module registry, jobs registered through A09's `registerJob`.
- Tables declared here, in the module's own schema: `job_definition`, `job_schedule_state`, `job_dead_letter`, `integration_health_sample`, `integration_config_version`, `webhook_secret`. Core tables reused, never redefined and never migrated here: A04's `integration`, `webhook_subscription`, `webhook_delivery` and `job_run` (A04's inventory lists them as core tables used by A46), and A04's `outbox` (the table A09 calls `outbox_event`; A04 fixes the name `outbox`), `service_account`, `processed_event` and `job_schedule_fire`. The columns this node needs on the four core tables are requested from A04 as loop A04.1 below; this node writes no migration for any core table.
- Time comes only from A09's injectable clock (the `Date.now()` lint of A09 applies to every file here); business-day and holiday arithmetic comes from A13's `businessDaysAfter` and `isBusinessDay`; alerts are raised only through A13's `emit()` with type codes that exist in its catalogue or that this module registers through A13's `registerAlertTypes`; audit rows only through A10's `audit.record` inside the same transaction as the state change; adapter health only through A08's `adapterHealth()` and the swap-point rows of `SWAP-POINTS.md`; chain and indexer status only through A27's `GET /v1/chain/status` read model; token state and scopes only through A12's `introspect()` and `validateTokenScopes()`; maker-checker only through the approvals module's public interface (A14, an AM1 node complete before AM3 starts; this node consumes its `submit` and `onApproved` calls and registers effectors through `registerEffector(action_code, …)` for the four action codes below, and never edits A14's policy table).
- Follow-up loops requested from upstream nodes before this node's Specify freezes, in the precedent of A12's A04.1. Nothing in them is implemented here; each is a data-and-hook change with a seed test in its own node.
  - **A09.1** (joins the A09.1 list A104 already requested; the `SchedulePolicy` hook in the tick and the `deadLetter: 'jobs.dead-letter'` option on every job are already part of A09's spec):
    1. `scheduler.tick` takes an injectable `DueInstantCalculator { next(cron, zone, calendarRule, calendarCode, after): Instant | null }` (default: A09's plain-cron arithmetic; this node installs `scheduler/cron.ts`). The tick becomes the ONLY cron path on the platform. `registerJob` gains the optional fields `zone` (default `Australia/Sydney`), `calendar_rule`, `calendar_code`, `critical`, `money`, `expected_duration_ms` and `grace_ms`. Every `cron` passed to `registerJob` is fired by the tick through the calculator and the policy. A lint rule forbids calling pg-boss `schedule()` or any other cron API outside A09's core.
    2. A read of the job registry, `listRegisteredJobs()` (name, owner module, cron or none for an event-triggered job, and the fields above), which this node's boot sync reads.
    3. A bootstrap slot `registerConfigOverlaySource(fn)`. After the database connects, A09's bootstrap calls the registered source (this node's), passes its result to A08's existing `loadConfig({ overlay })` (re-validated by the same schema, allow-lists included) and boots on the `.env` values alone if the source returns nothing.
    4. `/v1/meta` gains `config_versions[]` (`integration`, `version`, `values_sha256`, `approval_id`) from the applied overlay.
  - **A04.1**:
    1. `job_run` gains `trigger` (`schedule | event | manual | replay`), `due_at`, `on_behalf_of_id`, `replay_of`, `trace_id` and the outcome enum `ok | failed | skipped_paused | skipped_disabled | skipped_no_handler | cancelled`.
    2. `integration`, `webhook_subscription` and `webhook_delivery` take the column lists in §4 and §5 of this note.
    3. A read-only RLS probe, `canPrincipalSee({ principal_id, tenant_id, vehicle_id?, subject_principal_ids[] })`. It answers under the principal's own RLS context without loading any other module's aggregate, and the webhook visibility check (§5) uses it.
  - **A03.1** (with the matching A14 `approval_policy` seed rows). This covers the M26 actions and cells this node's routes need; the product map carries only "Switch RPC or network config" for ADM-32.
    - Action codes and policy rows: `integrations.config.change` (`4E`, ENGO to SECO; the product-map row); `jobs.schedule.pause` (`4E`, ENGO to SECO, critical rows only); `jobs.schedule.disable` (`4E`, ENGO to SECO, every row); `jobs.dead_letter.discard` (`4E`, ENGO to CFO, money rows only).
    - Cells for the roles in §6: `jobs:read`, `jobs:trigger`, `jobs:replay`, `jobs:pause`, `jobs:disable`, `jobs:discard`, `integrations:read`, `integrations:config_propose`, `integrations:sandbox` and `webhooks:read_all`. The cell `webhooks:manage_own` goes to every role that signs in: every investor sub-type, sponsor role, partner type and staff role.
    - Delegable flags in `delegable.ts`: every cell above is delegable as maker (the four `4E` acts are never delegable as checker, which A14 already enforces), and `webhooks:manage_own` is delegable in full (§5).
    - The three policy rows that the product map does not carry (`jobs.schedule.pause`, `jobs.schedule.disable`, `jobs.dead_letter.discard`) are founder-visible additions. They must be recorded in [[Decisions]] or the Product Map QA Report before the freeze. This node registers only the effectors for the four codes, never a policy row.
- The alert type `integration.degraded` is registered from this module's own folder through A13's `registerAlertTypes`, as A13's extension registry already names (Sev3, ENGO, ADM-32; Sev2 through the severity parameter when an adapter is `down` for more than 15 minutes; sub-code `config_hash_mismatch` for §4). No A13 loop is requested. The id "A13.1" belongs to A66's read-contract request, which A13 has already absorbed.
- Spec defects raised against other nodes. Their owners must change them; this node does not.
  - **A20, A31 and A42:** each says its nightly job is "registered with the pg-boss cron". Each must say `registerJob` with `cron` and `zone`, so the job is fired by the tick through A09.1.
  - **A20:** `recon_schedule.enabled` must become a read-only mirror of `job_definition.enabled` and `paused_until`. Its cron must become the value A20 passes to `registerJob`, so a schedule has one switch, not two.
  - **A20 and A18:** A20's nightly section says A18's extract is fetched at 02:10, but A18 fixes `register.fetch_extract` at 01:40.
  - **A13:** once A09.1 lands, `alerts.digest` should register `0 7 * * *` in `Australia/Sydney`, not an hourly UTC cron with a local-hour check.
  - **A12:** its nightly token-expiry sweep has no job name and no time.
  - **A51:** its ADM-32 extension draws "Run now" and "Pause job" through the hardware-key modal. See "Step-up" in §2.
- The worker jobs of this node run as the service account `svc-scheduler` (a row in A04's `service_account` table, principal kind `SVCA` in A09's context, with no login, no keys, no custody role and a cap of zero on every money or supply action; a handler that moves money does so under its own module's approval binding, never under this account's authority); a replayed or manually triggered job runs under the human or token that asked for it as `on_behalf_of`, with `svc-scheduler` as the executing actor, so ADM-31 shows both.

**2. The job catalogue (`job_definition`) and the due-instant calculator**

- One row per scheduled or event-triggered job on the platform. Rows come from two sources, merged at worker boot by an **upsert-only** sync that never deletes, disables or rewrites a row it did not write.
  - **A09's job registry** (`listRegisteredJobs()`, loop A09.1). This is how every owning module declares its job, cron, zone, calendar rule and flags, through its own `registerJob` call. No module edits this node's files to add a schedule; A101's and A102's jobs arrive this way.
  - **This node's `catalogue.ts`.** It holds this node's own jobs, plus an **expected** row for each job that an owner's spec declares, so ADM-32 shows the row before the owner is built.
  - When the owner's registration arrives, its values replace the expected row.
- Columns:
  - Identity and source: `name` (the pg-boss job name), `owner_module`, `registered_by` (`scheduler` for this node's own jobs, `expected` while only `catalogue.ts` knows the row, `module` once the owner's `registerJob` provided it), `trigger_kind` (`cron` or `event`), `description`.
  - Schedule: `cron` (null for event rows, or while an expected row's owner states no time), `zone` (default `Australia/Sydney`), `calendar_rule`, `calendar_code`.
  - Flags: `critical` (a pause needs a second person), `money` (the handler can move money, supply or holdings; mirrors A09's `x-realx-money`), `singleton`.
  - Timing: `expected_duration_ms`, `grace_ms` (how late a fire may run before it counts as missed).
  - Switches: `enabled`, `paused_until`, `pause_approval_id`.
  - `handler_registered`: true when the owning module's handler is present in the worker registry at boot.
- The expected rows are copied from each owner's spec, in `Australia/Sydney` unless stated. Where an owner states no time, `cron` stays null and ADM-32 shows "schedule not set" rather than an invented time, until the owner registers it:

  | Job | Owner (spec section) | Schedule | Critical | Money |
  |---|---|---|---|---|
  | `register.snapshot` | A18 (register versions) | `0 0 * * *` | yes | no |
  | `properties.attestation_open` | A34 (proof of property) | `30 0 1 * *` | no | no |
  | `bank.statement.ingest` | A17 (Jobs, M26) | `30 0 * * *` | no | no |
  | `register.fetch_extract` | A18 (register versions: 01:40, after the registrar's 01:30 extract) | `40 1 * * *` | yes | no |
  | `recon.nightly` | A20 (Nightly) | `0 2 * * *` | yes | no |
  | `claims.sweep-expired` | A31 (worker) | `0 2 * * *` | yes | no (revokes on-chain claims through V4; no money) |
  | `screening.nightly` | A42 (screening) | `0 3 * * *` | yes | no |
  | `classification.expiry_digest` | A32 (drift check) | `0 5 * * *` | no | no |
  | `properties.valuation_due` | A34 (valuations) | `0 5 * * *` | no | no |
  | `audit.verify` | A10 (jobs) | `30 5 * * *` | yes | no |
  | `ledger.integrity_nightly` | A16 (rebuild) | owner states "nightly", no time | yes | no |
  | `audit.evidence-ship` | A10 (jobs) | owner states "nightly", no time | no | no |
  | `properties.dd_expiry` | A34 (due diligence) | owner states "nightly", no time | no | no |
  | `properties.attestation_sweep` | A34 (overdue) | `0 * * * *` | no | no |
  | `alerts.digest` | A13 (worker jobs) | `0 * * * *` in `UTC`, as A13 registers it today (defect raised above) | no | no |
  | `distributions.funding_check` | A40 (fund) | event | n/a | yes |
  | `distributions.execute` | A40 (execute; fired by A14's `AUTO` row on the `funded` transition) | event | n/a | yes |
  | `integrations.health_probe`, `jobs.watchdog`, `webhooks.token_sweep` | this node | `* * * * *` | no | no |
  | `webhooks.retry` | this node | `*/30 * * * * *` (six-field, seconds; the calculator accepts it) | no | no |
  | `scheduler.prune` | this node | `0 5 * * *` | no | no |

- Jobs that no owner's spec declares are not seeded:
  - A12's nightly token-expiry sweep appears when A12 registers it; A12 names no job and no time.
  - A09's idempotency sweep is a platform runtime job and appears through the registry if A09 registers it with `registerJob`.
  - The quarterly access review is A67's screen workflow and has no scheduled job in A11 or A67.
  - The "critical" list is exactly the rows marked yes above, plus any row whose owner passes `critical: true`.
- Seed test (`catalogue.seed.test.ts`). For every expected row whose owner is built (its module appears in A09's module registry), `name`, `cron`, `zone`, `calendar_rule` and `trigger_kind` must equal the owner's `registerJob` entry exactly. A mismatch fails this node's regression suite and names the owner's spec section. A drift between this table and an owner is therefore a spec defect, never a silent `skipped_no_handler` or `handler_missing`.
- Firing rule. `scheduler/cron.ts` is the `DueInstantCalculator` installed into A09's tick (A09.1), so it decides when every cron on the platform fires, including module-registered ones.
  - It computes the next due instant of a cron in its zone from local civil time using the IANA rules, never by adding multiples of 86,400 seconds.
  - It uses Temporal's `compatible` disambiguation. A local time that does not exist on the daylight-saving start night (02:00 to 02:59 on the first Sunday of October) moves forward by the length of the gap: on 4 October 2026, 02:00 fires at 03:00 AEDT (16:00 UTC on Saturday 3 October) and 02:10 fires at 03:10 AEDT (16:10 UTC on 3 October).
  - A local time that occurs twice on the daylight-saving end night (02:00 to 02:59 on the first Sunday of April) fires once, at the earlier offset: on 4 April 2027, 02:00 fires at 02:00 AEDT (15:00 UTC on Saturday 3 April) and never again at 02:00 AEST.
  - The idempotency key of a fire is `(job_name, due_instant_utc)` through A09's `job_schedule_fire`, so a worker restart, two workers or a 30-day clock advance never fire one schedule twice for one instant.
- Calendar rules, evaluated inside the calculator so the policy never has to turn a skip into a fire. `calendar_code` defaults to `AU-NSW`.
  - `none` (the default).
  - `business_days`: a fire whose local date is not a business day produces no due instant at all. It is neither a skipped row nor moved to the next day.
  - `first_business_day_of_months(m1,…)`: the cron's day-of-month field is ignored, and the due instant is the cron's local time on the first business day of each listed month, found with A13's `isBusinessDay`.
  - Reference vectors: `first_business_day_of_months(1,4,7,10)` at 08:00 gives Monday 4 January 2027 08:00 AEDT (21:00 UTC on 3 January), because 1 January is a public holiday and 2 and 3 January are a weekend. `business_days` at 09:00 gives no fire on Monday 5 October 2026 (Labour Day in NSW).
  - A calculation beyond A13's last seeded calendar year is refused by A13 with 422. The row then shows `calendar_exhausted` and the watchdog raises `missed_schedule`, rather than the job firing on a guess.
- `SchedulePolicy` (A09's tick hook) answers `skip` with a reason when the row is disabled, paused or has no registered handler. A skip writes a `job_run` row with outcome `skipped_disabled`, `skipped_paused` or `skipped_no_handler` and raises no alert, so ADM-32 shows the gap instead of hiding it. The policy reads `job_definition` for every cron fired by the tick, so `enabled` and `paused_until` are the single on/off switch for every schedule on the platform, whoever registered it. A handler of a node not yet built when this node runs (A31, A32, A34, A40 and A42 are AM3 peers) is exactly the no-handler case: the expected row exists from day one and starts firing the day the owner registers.
- Pause and resume. `POST /v1/jobs/{name}/pause {duration_minutes, reason}` takes a duration of at most 1,440 minutes; a longer value is refused with 422 `pause_too_long` before anything is stored.
  - For an ordinary row the pause is an ENGO act that takes effect at once, with `paused_until = now + duration`.
  - For a row with `critical: true` the call creates an `ApprovalRequest` with action code `jobs.schedule.pause` under `4E` (ENGO to SECO) and returns 202 with the approval id. The row is not paused until A14 reports `approved`; then `paused_until = approved_at + duration`. A pause therefore never lasts longer than 24 hours from the moment it takes effect, and an approval that waits in the queue neither shortens nor extends it.
  - A pause expires by itself, so a forgotten pause cannot silently stop reconciliation for a second night. Resume is immediate for ENGO and audited.
  - Disabling a row (`POST /v1/jobs/{name}/disable`, `enabled = false`) is `jobs.schedule.disable` under `4E` (ENGO to SECO) for every row, and is the only way to stop a schedule for longer than a day. Re-enabling is immediate for ENGO.
- Step-up. A manual trigger, an ordinary pause, a resume and a replay need no fresh assertion on the server. The hardware-key assertion belongs only to the checker of the four `4E` paths, which A14 enforces. A51 and A76 must therefore draw "Run now" and the ordinary "Pause job" as a plain confirmation with a mandatory reason, and use the hardware-key modal only for the approval step of a critical pause, a disable, a money discard or a configuration change. No check then exists only in the UI.
- Manual trigger. `POST /v1/jobs/{name}/trigger {reason}` (ENGO, or a staff token with the cell as maker) sends the job:
  - For a cycle-based job it uses the idempotency key of the current cycle: a manual trigger of `recon.nightly` during a cycle that already ran is a no-op that returns the existing run id.
  - Otherwise it uses a fresh key.
  - An event-triggered row cannot be triggered by hand: 409 `job_not_triggerable`.
  - The run carries `trigger: manual`, the actor and `on_behalf_of`, and writes one audit event.
- `jobs.watchdog` (every minute, on the injected clock) raises A13's `job.dead_letter` with one of four sub-codes:
  - `missed_schedule`: a due fire has no `job_run` row (including a skipped row) after `grace_ms`, or the calculator reports `calendar_exhausted`.
  - `overrun`: a run exceeds three times `expected_duration_ms` and is still running.
  - `handler_missing`: raised once per day for a row whose owning module is built (its module name appears in A09's module registry) but whose handler is absent.
  - Every case dedups per `(job_name, sub_code)` and resolves by system when the condition clears.

**3. Job runs, outcomes and the dead-letter list**

- `job_run` (an A04 core table; columns through A04.1) is the run history: `job_name`, `job_id`, `attempt`, `trigger` (`schedule`, `event`, `manual`, `replay`), `due_at`, `started_at`, `finished_at`, `duration_ms`, `outcome` (`ok`, `failed`, `skipped_paused`, `skipped_disabled`, `skipped_no_handler`, `cancelled`), `error_code`, `error_summary` (a code and a first line, passed through A09's redaction; never a stack, never an argument, never a name), `actor_id`, `on_behalf_of_id`, `replay_of` and `trace_id`. A09 already writes the row for every run it executes; this node writes the skipped rows from its policy, and adds the read models and the `next_due_at` and `last_outcome` projections in `job_schedule_state`.
- Dead letters. pg-boss delivers a job that exhausted its `retryLimit` to the `jobs.dead-letter` queue (A09). This node's `dead-letter.collector` works that queue and writes one `job_dead_letter` row per `(job_name, payload_sha256)`, with:
  - `id` (`DLQ-<yyyymmdd>-<6 base32>`), the original job id, `attempts`, `first_failed_at`, `last_error_code` and `last_error_summary`;
  - `money` (from the catalogue) and `state` (`open`, `replayed`, `discarded`).
  - It raises A13's `job.dead_letter` (Sev3, ENGO, ADM-32) with sub-code `failed_final`, deduped per row and resolved by system when the row leaves `open`.
  - A second exhaustion of the same payload increments `occurrences`; it never creates a second row.
- Replay. `POST /v1/jobs/dead-letters/{id}/replay {reason}` (ENGO, or a staff token as maker) re-sends the original payload as a new job with `trigger: replay` and `replay_of` set, under the same pg-boss singleton key where the original had one.
  - Replay is safe only because N13 makes every handler idempotent on its own keys: A40's distribution lines on their line ids, a snapshot on its cycle, a screening run on its list version and date. This node relies on that contract and tests it with a fixture handler that counts successful side effects.
  - A replay never bypasses the owning module's gates. The replayed handler runs A20's `assertClear`, A14's approval binding and every G01-G22 evaluation exactly as a scheduled run does.
  - A row can be replayed at most three times. The fourth attempt is refused with 409 `replay_limit`, and the row stays `open` for a human decision.
- Discard. `POST /v1/jobs/dead-letters/{id}/discard {reason}`:
  - For a `money: false` row it is an ENGO act with a mandatory reason.
  - For a `money: true` row it creates an `ApprovalRequest` with action code `jobs.dead_letter.discard` under `4E` (ENGO to CFO), and the row is discarded only on `approved`.
  - A discarded money row keeps its payload hash and reason forever (retention with A10's audit event), and the owning module receives the outbox event `jobs.dead_letter.discarded` so it can surface the consequence (for example a distribution run left in `funded`).
- Lifecycle `open -> replayed | discarded` is a small local transition table in A03's format (not one of SM-00..SM-13); `replayed` returns to `open` if the replay itself dead-letters again (a new occurrence on the same row). Every transition writes exactly one audit event.

**4. Integrations: health, latency, environment and configuration versions**

- `integration` rows (an A04 core table; columns through A04.1). There is one per adapter manifest from A08: `identity`, `sanctions`, `bank`, `custody`, `registry`, `esign`, `email`, `sms`, `chain`, `bundler`, `paymaster`, `valuation-feed`, `property-data`, `stablecoin`, `kms` and `storage`.
  - Two derived rows, `chain-rpc` and `indexer`, are fed from A27's status read model (chain id, RPC host name only, head, cursor, lag seconds, manifest check result).
  - A third derived row, `mcp-server`, is owned by this node. The health probe calls the `/readyz` endpoint of A104's compose service `mcp`, at the URL in this module's config variable `MCP_HEALTH_URL` (defaulting to the address A104's compose fragment exposes), with the same 2 s timeout. The row shows `unknown` while the service is absent, so A51's connector list (which includes the MCP server) is complete from day one. A104 is an AM1 node and calls nothing in this module.
- Columns of an `integration` row:
  - `name`, `implementation`, `environment` (`sandbox` or `live`; `live` cannot be produced by any code path in this build, per A08), `status` (`ok`, `degraded`, `down`, `unknown`).
  - `latency_p50_ms` and `latency_p95_ms` over the last 15 minutes, `uptime_24h_bps` (basis points), `last_ok_at`, `last_check_at`, `last_error` (`code`, `at`, and a summary that has passed A09's redaction).
  - `config_version` (the applied `integration_config_version`).
  - `human_gate` and `candidates` (from `SWAP-POINTS.md`, so ADM-32 shows the Q12 swap point and its open gate next to each connector).
  - `sandbox_controls` (the `SandboxControl.listScenarios()` result when a control is registered, for the ADM-32 sandbox panel).
  - `credential_ref`: the product map's "provider integrations keep their service-account keys in ADM-32". It holds a `kms` key reference and its creation date only, never a value. Every reference in this build points at the sandbox master key, so HG-KEYS stays open.
- The product map's ADM-32 state "Indexer lag above 5 min: Sev3" is A27's `chain.stuck_or_lagging` alert (`indexer_lag_5m`). This node shows the `indexer` row as `degraded` at the same threshold and suppresses its own `integration.degraded` for that row while A27's alert is open, so one lag never raises two alerts. The other ADM-32 components belong to A12 and A104: "API keys by sponsor with scopes", "AI-agent tools (MCP) with read or write scope", and their actions ("Create API key with write scope" ENGO or PART to SECO, "Revoke API key", "Enable an agent write tool" SECO to CCO). This node's summary only links to them and adds no route for them.
- `integrations.health_probe` (every minute):
  - It calls A08's `adapterHealth()` (per-adapter 2 s timeout, never cached), A27's status and the `mcp` health endpoint, and writes one `integration_health_sample` per integration (`at`, `status`, `latency_ms`, `error_code`).
  - It recomputes the rolling columns.
  - It emits `integration.degraded` through A13 when a row is `degraded` or `down` on two consecutive probes. The alert dedups per integration, is upgraded to Sev2 through the parameter after 15 minutes `down`, and is resolved by system on the first `ok`.
  - A single slow adapter never delays the others: probes run in parallel with the timeout, and a probe that overruns the minute is skipped, not queued.
  - Samples older than 30 days are pruned by `scheduler.prune`, the module's one housekeeping job.
- Configuration versions. `integration_config_version` has the columns `integration`, `version` (v1, v2, ...), `values_json`, `values_sha256`, `proposed_by`, `proposed_at`, `approval_id`, `state` (`proposed | approved | rejected | applied | superseded`) and `applied_at`.
  - Values are the subset of A08's `packages/config` variables that belong to that integration and that A08 marks `overlayable`. For `chain-rpc` these are `CHAIN_RPC_URL` and `CHAIN_ID`, which A08's core section must mark overlayable (A08's own overlay test already overlays a chain id). For adapters they are `ADAPTER_<NAME>`, timeouts, poll intervals and the sandbox seed.
  - `POST /v1/integrations/{name}/config` (ENGO, or a staff token as maker) validates the proposal against A08's schema before anything is stored. It refuses with 422 `config_value_not_allowed`, naming the variable, a chain id outside `{31337, 84532}`, an RPC host outside A08's allow-list, or an adapter selector other than `sandbox`. It refuses with A08's `CONFIG_NOT_OVERLAYABLE` a variable A08 never allows in an overlay (`RUN_MODE`, `PHASE`, `TESTNET_OPT_IN`, every `secret`). A mainnet or vendor endpoint therefore cannot even reach a checker.
  - The product map's "switch RPC or network config, ENGO to SECO, config v+1" row is implemented as an `ApprovalRequest` with action code `integrations.config.change` under `4E`, ENGO to SECO. On `approved` the version becomes `approved`.
  - It becomes `applied` in one of two ways. (a) At the next start of the API and worker, through A09.1's `registerConfigOverlaySource`: this node's source reads the latest `approved` or `applied` version per integration, recomputes `values_sha256` over `values_json`, and hands only matching versions to A08's `loadConfig({ overlay })`. (b) At once, for an adapter that declares A08's `reconfigure: true`: on `approved`, this node publishes a Postgres NOTIFY and each API and worker process calls `reconfigure(values)` after the same hash check. `chain-rpc` has no reconfigure hook, so its versions apply only at restart.
  - If the hash does not match, the version is not applied and stays `approved`. One audit event is written, `integration.degraded` is raised with sub-code `config_hash_mismatch`, and the process runs on the previous applied version.
  - The applied version, its hash and its approval id are reported on `GET /v1/integrations/{name}` and in A09's `/v1/meta.config_versions[]` (A09.1). Every transition writes one audit event, and a rejected or superseded proposal stays readable.
- Sandbox panel. `POST /v1/integrations/{name}/sandbox/{event}` forwards to A08's `SandboxControl.trigger` (mounted by A09 only in `RUN_MODE=local|test`). This node adds the permission check (`integrations:sandbox`, ENGO and SECO), the audit event and the run record, so each button of the ADM-32 panel A76 draws ("receive a payment", "return a payment", "ack an instruction", "drift the register") is one call. Outside local and test modes the routes are absent from the OpenAPI document and answer 404.

**5. Webhook subscriptions: scoped like tokens, signed, replay-safe, revoked with the token**

- `webhook_subscription` (an A04 core table; columns through A04.1). Its columns:
  - `id` (`WHS-<yyyymmdd>-<6 base32>`), `owner_principal_id`, `owner_kind` (`HUMAN` or `PARTNER`, from A04's principal kinds; a staff member is a `HUMAN` principal with a staff role), `tenant_id`.
  - `token_id`: the API token the subscription is bound to. It is required for every subscription, whether created through the API, through MCP, or through the ACC-05, SPN-16 or ADM-35 screen, which pick one of the person's active tokens.
  - `created_via` (`ui`, `api`, `mcp`) and `created_by_token_id` (null when a person created it in the UI).
  - `url`, `event_types[]`, `description`.
  - `state` (`active`, `paused`, `revoked`, `failing`), `consecutive_failures`, `last_delivery_at`, `last_success_at`.
- A subscription's scopes are its token's scopes. At creation, every requested event type must map, through this node's `webhook_event_catalogue.ts`, to a scope cell the token holds. The check uses A12's `validateTokenScopes`. A type outside the token's scopes is refused with 403 `webhook_scope_exceeds_token`, naming the type and the cell; it is never dropped silently. Limits: 10 subscriptions per token, 25 event types per subscription, one endpoint URL per subscription.
- The event catalogue lists which outbox event types may leave the platform, what each carries, and which envelope fields decide who may see it.
  - Only types marked `webhook_eligible: true` can be subscribed to (the flag name A101 uses). Their payload is ids, codes, amounts in integer cents or units, timestamps and state names: never a name, email, address, TFN, bank detail, document content or free-text reason. Every other outbox type is never fanned out, including A101's `agent.guard.*` types, which A101 asks to be marked `webhook_eligible: false`.
  - Each catalogued type names its **visibility inputs**: the fields of A04's `outbox` envelope and payload (`tenant_id`, `vehicle_id` where the type has one, and the subject principal ids) that A04.1's `canPrincipalSee` needs.
  - Types are grouped by scope family, so the screens can show them the way they show token scopes:
    - investors: `investor.*`, `order.*`, `payment.*`, `distribution.*`, `holding.*`, `document.*`, and `governance.*` (P2);
    - sponsor users and partners: `listing.*`, `offer.*`, `register.*`, `attestation.*`, `valuation.*`, `sponsor.billing.*`;
    - staff: `approval.*`, `alert.*`, `recon.*`, `job.*`, `integration.*`, `agent.*` (except `agent.guard.*`);
    - anyone: `agent.confirmation.requested` and `agent.confirmation.resolved` from A12, and the built-in `ping`.
  - A test walks every catalogued type's zod schema and fails on any field name in A09's `PII_PATHS`, on any eligible type without visibility inputs, and on any `agent.guard.*` type marked eligible.
- Dispatch. `webhooks.dispatch` is an A09 event handler registered for every eligible type, and is idempotent on `event.id` through `processed_event`. For each active subscription of the type it runs three steps:
  - (a) It checks that the bound token is `active` and still holds the required cell, through A12's `introspect()`. A narrowed token narrows its webhooks at once. A token in A12's `held` state pauses them.
  - (b) It checks visibility with A04.1's `canPrincipalSee`, using the owner principal and the type's visibility inputs, never by loading another module's aggregate. An event about another tenant's vehicle or another investor's order is recorded as `skipped_not_visible` and never delivered.
  - (c) It writes one `webhook_delivery` row per `(subscription_id, event_id)` with a unique key, so a redelivered outbox event can never produce a second delivery row.
  - **Both checks (a) and (b) run again immediately before each attempt, including retries and replays, not only at fan-out.** A token that lost the cell produces `skipped_scope`. A person who lost sight of the aggregate between fan-out and attempt (role removed, tenant or vehicle scope withdrawn) produces `skipped_not_visible`.
  - A12's `agent.token.held` and `agent.token.released` and A101's `agent.guard.hold_placed` and `agent.guard.hold_cleared` are subscribed as a fast path that marks queued deliveries `skipped_paused` at once. The guarantee is still the `introspect()` re-check, which detects a held token whether or not an event arrived.
- `webhook_delivery` (an A04 core table; columns through A04.1). Delivery states:
  - `queued`, `delivered`, `retrying`, `dead`;
  - `cancelled` (token revoked or subscription revoked before send);
  - `skipped_scope` (the token no longer holds the cell);
  - `skipped_not_visible` (the RLS probe says the owner cannot read it);
  - `skipped_paused` (the subscription is paused, or the token is held);
  - `skipped_failing`.
  - Every skip is recorded with its reason and never sent. Delivery is at-least-once and unordered, with the event id in the headers so receivers dedupe.
- Wire format:
  - Body: JSON `{ id, type, occurred_at, api_version, data }`, at most 64 KB.
  - Headers: `X-RealX-Event-Id`; `X-RealX-Event-Type`; `X-RealX-Timestamp` (Unix seconds from the injected clock); `X-RealX-Signature: v1=<hex HMAC-SHA256 of "<timestamp>.<body>">` (two `v1=` values during a secret-rotation overlap); `X-RealX-Subscription`; and `Idempotency-Key`, which equals the event id.
  - Receivers are told in the docs (A99) to refuse a timestamp older than 5 minutes. This node's `POST /v1/webhooks/verify` helper and the SDK's `verifyWebhook(secret, headers, body)` implement exactly that window, so the replay rule is testable on both ends.
- Secrets. `webhook_secret` rows carry the prefix `whsec_` and 32 base32 characters from a CSPRNG, and are returned once, at creation and at rotation.
  - They are stored encrypted through A08's `kms` adapter. Signing needs the value, so it is never hashed. It is never logged and never in an audit or outbox event, and A09's redaction list gains the pattern `whsec_[A-Z2-7]{32}`.
  - Rotation issues a new secret with a 24-hour overlap, during which deliveries carry both signatures. An old secret past the overlap is deleted.
- Who may manage a subscription (D04 parity). Creating, widening (adding an event type, changing the URL), rotating the secret, pausing, resuming, narrowing and revoking are all ordinary acts of the owner under the cell `webhooks:manage_own`, and all are delegable to the owner's agent.
  - None of these acts is on D04's non-delegable list, and none gives an agent data it cannot already read through the API with the same token.
  - A person in the UI needs a signed-in session, and an agent needs a token; no passkey step-up is required of either.
  - An agent can create or widen a subscription only if it is bound to the token the agent is calling with, never another of its person's tokens (403 `webhook_token_mismatch`), and only within that token's scopes.
  - Every create, widen, URL change and secret rotation emits the outbox event `webhook.subscription.changed` (ids and codes only, with `created_via` and `token_id`). A43 turns it into a notice to the person, and A101 may classify it as a sensitive agent action, so the person sees an agent adding or moving an endpoint.
  - `webhooks:manage_own` is recorded as delegable in A03's `delegable.ts` through A03.1. If the founder or counsel later wants any of these acts person-only, that is a data change there, not here.
- Retries and failure:
  - `webhooks.retry` (every 30 seconds) redelivers failed rows at 1, 5, 15 and 60 minutes, then at 4, 8 and 24 hours (eight attempts in all), with jitter, on the injected clock.
  - A 2xx within 10 seconds is a success. Anything else is a failure, including a redirect, which is never followed; the status and a 200-character body excerpt are stored after redaction.
  - After eight attempts the delivery is `dead`, and A13's `job.dead_letter` is raised with sub-code `webhook_dead` (Sev4 through the severity parameter, owner ENGO, deduped per subscription). The person's own notice is the outbox event `webhook.subscription.failing`, which A43 turns into a notice.
  - A subscription with 50 consecutive failures, or 24 hours without a success, moves to `failing` and receives no new events until the owner resumes it. Deliveries during that window are recorded as `skipped_failing` and can be replayed.
- Egress guard. The webhook sender is the platform's only outbound HTTP to a user-supplied address, and the same rules hold in every run mode:
  - `https` only.
  - Before every attempt the host is resolved, and every resolved address is checked against the block list: loopback, link-local including `169.254.169.254`, private RFC 1918 and unique-local ranges, multicast, unspecified addresses, and the compose network's own service names `api`, `worker`, `postgres`, `hardhat`, `otel-collector` and `mcp`, whatever they resolve to. A blocked target fails the attempt with `egress_blocked`, and no request is sent.
  - DNS results are pinned for the attempt: the connection goes to the checked address, with SNI and `Host` set from the URL. A host that resolves differently between check and connect therefore cannot escape the guard.
  - Redirects are never followed (`redirect_refused`). Custom headers are not supported; the only credential a receiver gets is the signature.
  - There is exactly one exception: the fixture sink URL in this module's config variable `WEBHOOK_FIXTURE_SINK_URL`. It is unset by default and honoured only when `RUN_MODE` is `local` or `test`. It is matched after DNS pinning on the normalised URL: scheme, host, port and path prefix exactly. A path with dot segments, or with a percent-encoded `/`, `\` or `.`, is refused before matching. `http` is allowed for that one URL only.
- Replay and inspection:
  - `POST /v1/webhooks/{id}/replay {since, until}` (the owner; at most the last 7 days and 1,000 events per request) re-queues only deliveries in `delivered`, `dead` or `skipped_failing`, with the same event ids. It never re-queues `skipped_scope`, `skipped_not_visible`, `skipped_paused` or `cancelled`, and both checks (a) and (b) run again before each replayed attempt.
  - `GET /v1/webhooks/{id}/deliveries` (cursor-paginated) shows attempts, statuses, durations and the redacted response excerpt.
  - `POST /v1/webhooks/{id}/test` sends a `ping`.
  - Staff with `webhooks:read_all` (SECO, ENGO) see every subscription and delivery platform-wide, for ADM-35 and ADM-32. Investors, sponsor users and partners see only their own, through RLS.
- Revocation follows the token.
  - The event handler for A12's `agent.token.revoked` and `agent.mandate.revoked` moves every subscription bound to that token to `revoked`, in the same transaction that processes the event, and cancels its pending deliveries.
  - A12 publishes no expiry event (its token lifecycle reaches `expired` by time), so expiry is caught twice without one. First, the dispatcher's `introspect()` re-check answers `TOKEN_EXPIRED` for the next event and moves the subscription to `revoked` in that transaction. Second, this node's `webhooks.token_sweep` (every minute) revokes any subscription whose token's `expires_at` has passed.
  - Because the per-attempt re-check runs before any send, a revoked or expired token can never be followed by a delivery, even while the outbox event is still in flight. A subscription can never outlive its token, and there is no route that re-binds a subscription to another token: the person, or the agent with its own new token, creates a new one.
  - Tenant suspension (`tenancy.tenant.suspended`) and account closure do the same for every subscription of the tenant or person.
- Fixture sink. `packages/adapters` gains nothing. This node ships a local receiver as routes of its own API module: `POST /v1/sandbox/webhook-sink/{sink_id}` and a read route for tests. The routes are registered only in `RUN_MODE=local|test` and are absent from the OpenAPI document otherwise, like the sandbox panel, so no A09 change is needed.
  - e2e, the sim and the node QA's challenges set `WEBHOOK_FIXTURE_SINK_URL=http://api:4000/v1/sandbox/webhook-sink/` and point subscriptions at it.
  - The sink records every delivery, and can be told to answer 500 or 302, to time out, or to verify signatures.

**6. HTTP API (all under `/v1`, cursor pagination, idempotency keys on writes, in the OpenAPI document from the first commit)**

- Jobs (module scope M26; cells from A03.1):
  - Cells: `jobs:read` for ENGO, SECO and PART (the product map's ADM-32 roles) plus CCO, CEO, REGO, TRSY, TP-AUD and TP-OBS; `jobs:trigger`, `jobs:replay`, `jobs:pause` and `jobs:disable` for ENGO; `jobs:discard` for ENGO, with the `4E` path above.
  - Routes: `GET /v1/jobs` (catalogue rows with `next_due_at`, `last_run`, `last_outcome` and `state`), `GET /v1/jobs/{name}`, `GET /v1/jobs/{name}/runs?outcome=&since=`, `GET /v1/jobs/runs/{id}`, `POST /v1/jobs/{name}/trigger`, `POST /v1/jobs/{name}/pause`, `POST /v1/jobs/{name}/resume`, `POST /v1/jobs/{name}/disable`, `POST /v1/jobs/{name}/enable`, `GET /v1/jobs/dead-letters?state=&money=`, `GET /v1/jobs/dead-letters/{id}`, `POST /v1/jobs/dead-letters/{id}/replay`, `POST /v1/jobs/dead-letters/{id}/discard`.
- Integrations:
  - Cells: `integrations:read` for the same read set, plus SP-OWN for a tenant-scoped subset that shows only the `name`, `environment` and `status` of the adapters its journeys use; `integrations:config_propose` for ENGO; `integrations:sandbox` for ENGO and SECO.
  - Routes: `GET /v1/integrations`, `GET /v1/integrations/{name}`, `GET /v1/integrations/{name}/samples?since=`, `GET /v1/integrations/{name}/config` (versions), `POST /v1/integrations/{name}/config`, and `POST /v1/integrations/{name}/sandbox/{event}` (local and test only).
  - `GET /v1/integrations/summary` feeds the ADM-01 tile and the ADM-32 header. It returns the `ok`, `degraded` and `down` counts, the oldest `down` age, the applied config versions, the open human gates, and `banners[]`. `banners[]` carries the code `registry_sync_failed` while the `registry` row is `down` or the last `register.fetch_extract` run failed, so A76 can draw A51's red banner "No on-chain moves without a registry ack · runbook R4" from data.
- Webhooks:
  - Cells: `webhooks:manage_own` for every user type in the A03 matrix, including every investor sub-type, sponsor role, partner type and staff role; `webhooks:read_all` for SECO and ENGO.
  - Routes: `GET /v1/webhooks`, `POST /v1/webhooks`, `GET /v1/webhooks/{id}`, `PATCH /v1/webhooks/{id}` (widen or narrow), `POST /v1/webhooks/{id}/pause | resume | revoke`, `POST /v1/webhooks/{id}/rotate-secret`, `POST /v1/webhooks/{id}/test`, `POST /v1/webhooks/{id}/replay`, `GET /v1/webhooks/{id}/deliveries`, `GET /v1/webhooks/event-types` (the catalogue filtered to the caller's cells, grouped by family with plain-English labels), and `POST /v1/webhooks/verify`.
- Agent parity (D04, G22):
  - Every read and every maker act above is available to a token whose owner holds the cell, as maker, and is audited as the token acting on behalf of the person.
  - Four acts are approvals that A14 refuses to any token as checker: pausing a critical schedule, disabling a schedule, discarding a money dead letter, and applying a configuration change.
  - Every webhook act is delegable within the calling token's own scopes and bound to that token (§5).
  - No route here is a checker, ratifier or signer act, so nothing touches the never-checker rule beyond delegating to A14.
- Empty and error states:
  - An empty list is `{ items: [], next_cursor: null }`.
  - `GET /v1/jobs/{name}` for a row that has never run returns `last_run: null` with `next_due_at` computed (or `null` with `schedule_not_set` for an expected row whose owner states no time).
  - `GET /v1/integrations` with an empty adapter registry returns `[]` with `checked_at` (A08's contract).
  - A trigger of a paused job returns 409 `job_paused` with `paused_until`; a trigger of an event row returns 409 `job_not_triggerable`.
  - A pause of a critical row returns 202 with the pending approval id. A pause longer than 1,440 minutes returns 422 `pause_too_long`.
  - Replay of a dead letter not in `open` returns 409 `dead_letter_not_open`.
  - A webhook URL that fails the egress guard at creation returns 422 `egress_blocked`, naming the rule, not the resolved address.
  - An event type outside the token's scopes returns 403 `webhook_scope_exceeds_token`. An agent binding a subscription to a token other than its own returns 403 `webhook_token_mismatch`.
  - A subscription whose token was revoked returns 404 to everyone but staff (no existence leak), and `state: revoked` to staff.
  - An unknown job, integration or subscription is 404 for everyone, including other tenants.
- Audit (A10). One event is written for each of these: a catalogue change, pause, resume, disable, enable, manual trigger, replay, discard, configuration proposal or transition (including a hash mismatch), health status change (`ok` to `degraded` or `down` and back; individual samples write none), and a subscription creation, widening, narrowing, pause, resume, revoke, secret rotation or replay request. Individual deliveries and retries write none; they are `webhook_delivery` rows. Every event carries `actor_id`, `on_behalf_of_id` and `token_id` where a token acted, and `approval_id` where a `4E` path applied.

## Owned paths (other nodes do not edit these)

- <workspace>\app\services\api\src\modules\integrations\
- <workspace>\app\services\worker\src\scheduler\

Inside the API module, by convention:
- `jobs/`: catalogue read models, pause, disable, trigger, replay, discard.
- `health/`: integration rows, samples, summary.
- `config/`: configuration versions, the overlay source and the A14 wiring.
- `webhooks/`: subscriptions, secrets, event catalogue, verify, and the fixture sink routes.
- `schema.ts`, `events.ts`, `index.ts`.

Inside the worker folder:
- `catalogue.ts`: this node's own rows and the expected rows.
- `sync.ts`: the upsert-only boot sync from A09's registry.
- `cron.ts`: the `DueInstantCalculator`, with zone and calendar rules.
- `policy.ts`: the `SchedulePolicy`.
- `watchdog.ts`.
- `dead-letter/`: collector and replay.
- `health/`: the probe.
- `webhooks/`: dispatch, retry and the egress guard.

This node never edits A09's `src/core/` (the tick, `registerJob`, `job_schedule_fire`), A08's adapter core or config schema, A04's core tables, A13's catalogue files, A12's token tables, or A03's and A14's policy data. Gaps there are the follow-up loops A09.1, A04.1 and A03.1 above, or spec defects raised against those nodes.

## Design references

- No screen in `<workspace>\design\RealX App.dc.html` draws ADM-32. Its layout comes from `design-ext:ADM-32` in [[A51 - Design extension B- operator console screens not in the prototype]] (design analysis §5: "two DataTables, connectors with latency; jobs with last run"), and [[A76 - Console- regulatory calendar and data room, sponsor performance, integrations and jobs]] renders it.
- A51 already fixes the drawn columns:
  - connectors (Identity/DVS, sanctions, bank rail NPP/PayID/PayTo, custody, registry, chain RPC, email/SMS, MCP server) with Latency / Last ok / Sandbox pill (brick "sandbox" in every local run) / State;
  - jobs (reconciliation 02:00, register sync, screening refresh, expiry sweep, webhook delivery, anomaly baseline) with Last run / Duration / Next / State. These map to `recon.nightly`, `register.fetch_extract`, `screening.nightly`, `claims.sweep-expired`, `webhooks.retry` and A101's baseline job, which is present once A101 registers it;
  - "Run now" and "Pause job", which are a plain confirmation except for the approval step of a critical pause (see "Step-up" in §2);
  - the registry-sync red banner;
  - the states default, connector down, job failed and empty.
- This node's read models carry every one of those fields under the same meaning (`latency_p95_ms`, `last_ok_at`, `environment`, `status`; `last_run`, `duration_ms`, `next_due_at`, `state`; the trigger and pause routes; `banners[]`), plus the detail fields A76 shows in a row drawer, so A76 renders without translation:
  - Connector detail: Connector / Implementation / Environment / Status / Latency (p50 · p95) / Last check / Last error / Config v / Gate. Tone pills: `ok` green, `degraded` amber, `down` red (`#A3312A`), `unknown` grey, `sandbox` brick (the design's tone for previews) and `live` green.
  - Jobs table: Job / Owner / Schedule / Last run / Duration / Outcome / Next due / State. Tone pills: `ok` green, `failed` red, `skipped_*` grey, `paused` amber, and `schedule not set` grey.
  - A dead-letter list below it: ID / Job / First failed / Attempts / Money / State, with the row actions Replay and Discard. The approval step of a money discard uses the design's "Approve with hardware key" pattern from A14 and A51.
  - The sandbox panel, as OptionCards per registered scenario.
- Prototype regions that constrain the read models:
  - `adRecon` (ADM-17) shows a "Last run 06:00" column and the KPI subline "Run 06:00", and `adCmd` (ADM-01) shows the subtitle "all figures reconciled at 06:00 AEST". The seed's last `recon.nightly` run, recorded by A20 at 06:00 on 23 Sep 2026, must appear as that run in `GET /v1/jobs/recon.nightly` with outcome `ok`, while the schedule column shows 02:00 from the product map.
  - `adAudit` (ADM-31) has the row "23 Sep 06:02 · Recon job · Detected break · Gympie Road Retail", which fixes how a service-account actor is displayed. This node's `svc-scheduler` rows follow it: "Scheduler" as the actor label, and the human as on-behalf-of when one exists.
  - The `adCmd` KPI tiles are the shape of the ADM-32 header tiles: Connectors ok N of M, Jobs failed 24 h, Dead letters open, Webhook deliveries failing.
- Webhook sections on the undesigned screens:
  - `design-ext:ACC-05` and `design-ext:SPN-16` from [[A50 - Design extension A- investor and sponsor screens not in the prototype]] carry a "Webhooks" block under each token: URL, event families as the same scope-group chips the token picker uses, state pill, last delivery, and a copy-once secret card in the `wallet` screen's mono value chip style (Geist Mono 500 on cream `#F5F2EC`). A60 renders them.
  - `design-ext:ADM-35` (A51, rendered by A77) lists every subscription beside its token, with `created_via`.
  - The status vocabulary `active | paused | failing | revoked` maps onto the design's TONE map as green, amber, amber and red.
- Tokens: cream `#F5F2EC`, ink `#17191C`, green `#1F5E4B`, brick `#9A4524`, alert red `#A3312A`. Newsreader for the page title, Geist for text, and Geist Mono for job names, ids, latencies, hashes and cron strings. No pixels are produced here; "done" for design means the seed reproduces the rows above through the API exactly.

## Requirements covered

- Product-map screens: none owned. ADM-32 Integrations and jobs is rendered by [[A76 - Console- regulatory calendar and data room, sponsor performance, integrations and jobs]]; the webhook blocks of ACC-05 and SPN-16 by A60, and of ADM-35 by A77; the ADM-01 integrations tile by [[A66 - Console- command center, alerts centre, approvals inbox, audit log]]. "Done" means every column, pill, count and action those screens need exists in the API and the SDK, checked by a contract test the screen nodes reuse unchanged.
- Journeys: none owned. J20 (operator daily run) step 5 is "check integrations and jobs". "Done" means the operator can read the summary, the failed runs of the last 24 hours and the open dead letters in three calls, and that every other journey step marked "on" or "both" leaves a `job_run` or a delivery that this node can show.
- Gates:
  - **G13 Maker-checker.** Done when the four ADM-32 two-person acts exist and are enforced by A14, not by the UI: a configuration change, ENGO to SECO (`4E`, config v+1); pausing a critical schedule or disabling any schedule, ENGO to SECO (`4E`); and discarding a money dead letter, ENGO to CFO (`4E`). No route in this module lets a maker, a delegate or an agent of the maker approve any of them. A replay, a manual trigger and an ordinary pause are single-person maker acts, because the owning handler keeps its own gates.
  - **G22 Agent authority** (exercised, not owned). A webhook subscription can never carry an event type outside its token's scopes, is bound only to its own token, narrows with the token, pauses while the token is held, and stops with it.
- Modules: **M26 Integrations and API** is done when the product map's D18 rows are implemented here, on top of A08's adapter half, A09's API half, A12's token half and A104's MCP half. The rows are "connector health (identity provider, bank/NPP, unit registry, custody, chain RPC and indexer, e-sign)", "webhooks", "scheduled jobs (snapshots 00:00, reconciliation 02:00, sanctions re-screen 03:00, payouts, screening, property reporting reminders, reports)" and "switch RPC or network config (ENGO to SECO, config v+1)". Payouts appear as A40's event-triggered rows. The non-functional requirements tested here are N13 (idempotent, replayable jobs; webhook retries never double-apply), N12 (server-side filtering, paginated lists) and N06 (Australia/Sydney time with daylight saving).
- State machines: none of SM-00..SM-13. The dead-letter lifecycle and the subscription lifecycle are local tables in A03's format and are tested exhaustively.
- Founder requirement D04: done when a subscription can be created, widened, rotated and revoked by a fixture principal of every user type (seven investor sub-types, sponsor roles, partner types, staff roles) and by that person's agent through the API and MCP, bound to a token of that type, and when the event families offered to each equal the families of that token's scopes.

## Gauntlet loop for this node

- **Specify.** The frozen spec is this note plus:
  - A09's job, tick and outbox contracts, with loop A09.1 applied;
  - A04's core tables, with loop A04.1 applied;
  - A03's matrix and A14's policy seed, with loop A03.1 applied;
  - A08's `AdapterHealth` shape, overlay and `reconfigure` hooks, and `SWAP-POINTS.md`;
  - A13's catalogue and extension registry;
  - A12's `introspect` and `validateTokenScopes` signatures and token states;
  - A27's status read model and the design-extension column lists above.
  - The founder-visible policy rows must already be recorded (§1).
  - Any change to the daylight-saving rule, the calendar rules, the replay semantics or the egress guard after the freeze is a spec defect, not a fix.
- **Build (Grok 4.7 xhigh, split by the planner).** Four parallel sub-agents work on disjoint files, then one integration pass does the rest.
  - (a) The scheduler: `catalogue.ts` and the upsert-only sync; `cron.ts` with property tests around both daylight-saving nights and the calendar rules; the `SchedulePolicy`; the `job_schedule_state` projections; pause and disable with the A14 wiring; manual trigger; the watchdog; the catalogue seed test.
  - (b) Dead letters and integrations: the collector on the `jobs.dead-letter` queue; replay and discard, with the A14 path for money rows and a fixture money handler with a counting ledger fixture; the health probe (including the `mcp` endpoint), samples and rolling statistics; configuration versions with A08 schema validation, the overlay source, the reconfigure path and the A14 path; the sandbox forwarding.
  - (c) Webhooks: the event catalogue with its PII and visibility-input tests; subscriptions and secrets through the `kms` adapter; the dispatcher with the token and visibility re-checks before every attempt; signing, retries, the egress guard with pinned DNS and the single sink exception; replay; the fixture sink routes.
  - (d) The API routes and zod schemas; permission checks against the A03 matrix; OpenAPI entries and the SDK compile; fixtures for every user type and an agent token for each; the ADM-32 read models; and the screen contract tests A76, A60, A77 and A66 will reuse.
  - Integration pass:
    - boot the stack under the injected clock, advance through 3-4 October 2026 and 3-4 April 2027, and record every fire;
    - plant a failing fixture handler and a failing fixture money handler, and watch the dead letter, its alert, a replay and a discard;
    - stop one sandbox adapter and watch the row go `down`, the alert fire and then resolve;
    - subscribe every fixture principal to the sink, run the sim for a day and count deliveries;
    - write `app/evidence/A46.json`.
- **Challenge (node QA, Opus 5.5 high).** The node QA runs the module tests, then its own vectors (listed in the Oracle rule; the builder has never seen them), then these checks:
  - the daylight-saving and calendar instants below;
  - a 30-day clock advance with two workers running, where every schedule fires once per due instant;
  - the dead-letter replay against the counting fixture handler and the fixture money handler;
  - secret and PII greps for `whsec_` and every fixture PII string, over the logs, the database dump, the audit, outbox and delivery tables, and the evidence folder;
  - the egress guard against a list of hostile URLs, including a DNS name that flips to a private address;
  - the signature verifier against tampered bodies, stale timestamps and rotated secrets;
  - the token narrowing, hold and revocation cases, and the visibility-loss case;
  - permission tests for every role, with a token acting for each;
  - a review that the module imports A04, A08, A09, A10, A12, A13, A14, A20 and A27 only through their public `index.ts`;
  - a design-fidelity read of the seed rows against the column lists above.
  - The node QA writes `app/evidence/A46.qa.md` and, after the gate, files its vectors under `gauntlet\hidden\A46\`.
- **Gate.** Pass unlocks A66, A76 and A99. Max three fix loops; each loop re-runs the whole Challenge list.
- **Escalation trigger.** Reopen the spec and ask the user in any of these cases:
  - a third failure on the same assumption (for example, whether a missed critical fire should run late or be skipped, whether `applied` may happen without a restart, or whether an agent may create a webhook);
  - any fix that would need a change outside the owned paths beyond loops A09.1, A04.1 and A03.1 (A08's config schema, A12's introspection, A14's policy table, A27's status shape);
  - any finding that a job can run twice for one due instant, or that a delivery can leave the platform without a signature or after its owner lost the scope or the visibility.

## Node-specific challenges (node QA)

- **Daylight-saving nights and calendar rules (compliance).**
  - Advance the injected clock across Saturday 3 to Sunday 4 October 2026 (AEST to AEDT; 02:00 to 02:59 does not exist). Each of these must fire once, in this order, with one `job_schedule_fire` row each:
    - `register.snapshot` at 00:00 AEST (14:00 UTC on 3 October);
    - `register.fetch_extract` at 01:40 AEST (15:40 UTC);
    - `recon.nightly` and `claims.sweep-expired` at 03:00 AEDT (16:00 UTC);
    - a fixture row with cron `10 2 * * *` at 03:10 AEDT (16:10 UTC).
  - Advance across Saturday 3 to Sunday 4 April 2027 (AEDT to AEST; 02:00 to 02:59 occurs twice). `recon.nightly` and `claims.sweep-expired` must each fire exactly once, at the first 02:00 (02:00 AEDT, 15:00 UTC on 3 April), and `screening.nightly` once at 03:00 AEST (17:00 UTC).
  - The implementation fails if it adds multiples of 86,400 seconds, uses the server's zone, fires twice, or skips the DST-start night's reconciliation.
  - Run the same two nights with two worker processes: there must still be one fire per instant.
  - Calendar fixtures: a `first_business_day_of_months(1,4,7,10)` row at 08:00 must fire at Monday 4 January 2027 08:00 AEDT (21:00 UTC on 3 January) and never on 1 January. A `business_days` row at 09:00 must produce no fire and no row on Monday 5 October 2026.
  - Pause a module-registered schedule through the approved `4E` path across its fire. This is `recon.nightly`, as registered by A20's `registerJob` (A20 is a dependency). The fire must write exactly one `job_run` row with `skipped_paused` and must not send; this proves the pause governs schedules this node did not register.
- **Replaying a money dead letter (money).**
  - Setup: a fixture money handler, registered through `registerJob` with `money: true`, posts one entry per line to a counting ledger fixture. It is idempotent on its line ids, as N13 requires, and calls A20's `assertClear('payout')` before posting. A fixture fault makes it fail after its first line. It exhausts its retries and lands in the dead-letter list with `money: true` and one Sev3 alert.
  - Replay: after the fault is cleared, replay must complete with exactly one successful side effect per line (the counting fixture counts successful side effects only), and the `assertClear` spy must show a call on the replay. A second replay of the same row must be a no-op that returns the same result.
  - Discard: a discard by ENGO alone must be refused. ENGO proposing and ENGO approving through a delegated or agent route must be refused by A14. ENGO proposing and CFO approving with a hardware-key assertion must discard, leave the fixture run in its pre-execution state and emit `jobs.dead_letter.discarded`.
  - The same case against A40's real `distributions.execute` on the seed's Bay Road run is not claimed here; it belongs to A40's challenges or the AM8 journeys (A79, A81), which run after A40 exists.
- **A webhook that outlives its token (security, G22).** An investor's token with `read` and `order` scopes has a subscription to `order.*` and `payment.*`. Then, in turn:
  - The token is narrowed to `read`: a `payment.*` delivery must then be `skipped_scope`, not sent.
  - The token is placed on hold through A12's own `held` state, using a fixture that drives one of A12's hold triggers through A12's handler, as A12's tests do (A101 is not needed): deliveries must be `skipped_paused`, including one already queued before the hold.
  - The hold is released: deliveries resume.
  - The token is revoked: the subscription must be `revoked` in the same transaction and its queued deliveries cancelled. The sink must receive nothing after the revoke commits, measured across two worker processes fifty times.
  - The node fails if a subscription receives one more event after revocation.
  - Separately, the investor's agent creates a subscription bound to its own token, which must succeed and emit `webhook.subscription.changed`. It then tries to bind one to another token of the same person, which must be refused with `webhook_token_mismatch`.
- **Losing sight between fan-out and retry (security).**
  - A sponsor user subscribes to `register.*`. An event is fanned out and its first attempt fails at the sink. Before the retry, the user's role assignment in that tenant is removed. Separately, a partner user's vehicle scope is withdrawn before the retry of a `valuation.*` event.
  - The retry and an owner replay must both end `skipped_not_visible`, with zero deliveries at the sink.
  - A replay request must never re-queue a `skipped_scope`, `skipped_not_visible`, `skipped_paused` or `cancelled` row.
- **Hostile endpoints (security).**
  - With `WEBHOOK_FIXTURE_SINK_URL` set to the sink, create subscriptions pointing at: `http://api:4000/v1/jobs/recon.nightly/trigger`; `http://api:4000/v1/sandbox/webhook-sink/../../jobs/recon.nightly/trigger` and its percent-encoded variant; `http://169.254.169.254/latest/meta-data`; `https://10.0.0.5/`; `https://[::1]/`; a public hostname whose fixture DNS answer flips to `127.0.0.1` on the second lookup; and a public host that answers 302 to `http://postgres:5432`.
  - Every attempt must end `egress_blocked` or `redirect_refused`, with zero requests reaching the compose services; the sink and the API access log prove it.
  - The sink URL itself must succeed. With the fixture sink setting unset, the sink URL must be refused `egress_blocked` as well.
  - Signatures: tamper a delivery body by one byte, age a timestamp by six minutes, and sign with a rotated-out secret. The SDK's `verifyWebhook` must refuse all three and accept the untampered, in-window, current-secret delivery.
  - The `whsec_` value must appear nowhere except the creation and rotation responses.
- **Configuration change that reaches for mainnet (security, G13).**
  - ENGO proposes three versions of `chain-rpc` and the bank adapter:
    - v2: `CHAIN_ID=8453` and a public RPC host;
    - v3: `CHAIN_ID=84532` and an allow-listed Base Sepolia host;
    - v4: `ADAPTER_BANK=monoova`;
    - and a proposal that sets `RUN_MODE`.
  - v2 and v4 must be refused at proposal with `config_value_not_allowed`, naming the variable, and the `RUN_MODE` proposal with `CONFIG_NOT_OVERLAYABLE`. None of them may create an `ApprovalRequest`.
  - v3 must create a `4E` request that ENGO's own agent cannot approve and that SECO can approve. It must show `approved` but not `applied` until the processes restart (`chain-rpc` has no reconfigure hook). After the restart, `/v1/meta.config_versions[]` and `GET /v1/integrations/chain-rpc` must report v3 with its hash and approval id.
  - Editing `values_json` in SQL after approval must be caught by the hash check at apply time: v3 is not applied, the processes run on the previous version, and `integration.degraded` with sub-code `config_hash_mismatch` is raised once.
- **Pausing reconciliation quietly (compliance).**
  - ENGO asks to pause `recon.nightly` for 24 hours. The call must return 202 with a pending `jobs.schedule.pause` approval, and the row must not be paused: a 02:00 fire that falls before the approval still runs.
  - SECO approves. `paused_until` must equal the approval time plus 24 hours, and the next 02:00 fire must write one `skipped_paused` row.
  - A 25-hour request must be refused with `pause_too_long` before any approval is created.
  - A forgotten pause must expire by itself, so the next 02:00 fires.
  - The watchdog must raise `missed_schedule` if a fire is skipped without a `skipped_paused` row.
  - The node fails if a pause survives past `paused_until`, takes effect before approval, or skips a fire without a row.
- **Design fidelity of the read models.** With the standard seed:
  - `GET /v1/jobs` must show `recon.nightly` with schedule `0 2 * * *` in `Australia/Sydney`, and last run at 06:00 on 23 Sep 2026 with outcome `ok`. The audit actor label "Scheduler" must appear only on rows this node wrote. Every A51 job maps to the catalogue name listed in the design references.
  - `GET /v1/integrations` must return exactly one row per A08 manifest, plus `chain-rpc`, `indexer` and `mcp-server`. The `mcp-server` row is probed from A104's `mcp` service and shows `unknown` when the service is absent.
  - Every `environment` must equal `sandbox`, and every `human_gate` must be one of HG-VENDOR, HG-KEYS or HG-PROD.
  - Every A51 column (Latency, Last ok, Sandbox pill, State; Last run, Duration, Next, State) and the detail fields above must be present, with no extra or missing field.
  - Stopping the registry sandbox must put `registry_sync_failed` in `banners[]`.
  - The status strings must be exactly the vocabulary the tone map expects.
  - The contract test fails on any unknown status string, a `live` environment, or a connector without its swap-point gate.

## Pass criteria

- **Scheduling.**
  - Cron arithmetic agrees with the node QA's vector set on 100 percent of cases, three runs in a row. The set has at least 40 cases: both daylight-saving nights of 2026, 2027 and 2028 under the `compatible` rule, month and year boundaries, `business_days` fixtures across the seeded NSW holidays, and the `first_business_day_of_months` rule including 1 January 2027 → Monday 4 January 2027 08:00 AEDT.
  - A 30-day clock advance with two workers produces exactly one `job_schedule_fire` and one `job_run` per (job, due instant) for every enabled catalogue row with a handler, and one `skipped_no_handler` row per fire of a row without one.
  - Pausing or disabling a module-registered schedule produces `skipped_paused` or `skipped_disabled` rows and no fire.
  - The catalogue seed test matches every expected row against its owner's `registerJob` entry, for every owner already built.
  - The lint finds no pg-boss `schedule()` call outside A09's core.
- **Dead letters.**
  - A fixture handler that fails N times lands in `job_dead_letter` exactly once, with `attempts` equal to the retry limit plus one, and raises one `job.dead_letter` alert.
  - A two-line fixture handler commits line 1 and fails before line 2 until its fault is cleared. After that, three replays leave the counting fixture (successful side effects only) at 2, one per line, not 4. The fourth replay is refused.
  - Discard of a `money: false` row is immediate and audited. Discard of a `money: true` row exists only through an approved `4E` request whose checker is not the maker, the maker's delegate or any agent of the maker.
- **Integrations.**
  - Stopping a sandbox adapter shows `down` within two probes, raises `integration.degraded` once, upgrades it to Sev2 after 15 minutes on the injected clock, and resolves it by system on the first `ok`.
  - An indexer lag above 5 minutes raises only A27's `chain.stuck_or_lagging`, never a second `integration.degraded`.
  - A slow adapter (3 s) is `degraded` and never delays another row's sample by more than the timeout.
  - `uptime_24h_bps` and `latency_p95_ms` are recomputed from samples and match a reference computation.
  - The configuration path refuses every non-allow-listed or non-overlayable value at proposal, and applies an approved version only when its hash matches.
- **Webhooks.** Over a simulated day with one subscription per fixture user type (at least 20 subscriptions) and at least 2,000 catalogued events:
  - every delivery row is unique per (subscription, event);
  - every delivered body verifies with the SDK's `verifyWebhook`;
  - zero deliveries carry a field in `PII_PATHS` or any fixture PII string;
  - zero deliveries reach a receiver for an event the owner could not read under RLS at the moment of the attempt, whether the attempt was the first, a retry or a replay;
  - every failed attempt follows the retry schedule on the injected clock and ends `dead` after eight attempts, with one Sev4 alert per subscription;
  - every hostile URL in the egress list is blocked, with no request observed.
- **Token binding.**
  - Creating a subscription with an event type outside the token's cells is refused, naming the type and the cell.
  - Narrowing the token narrows deliveries on the next event, and a held token pauses them.
  - Revoking the token revokes its subscriptions in the same transaction. An expired token's subscriptions are revoked by the next dispatch or the one-minute sweep, whichever comes first. Both are measured fifty times each across two worker processes, with zero deliveries after the revoke commit or the `expires_at` instant.
  - An agent token can create, widen and rotate only subscriptions bound to itself and within its own scopes. Each such act emits one `webhook.subscription.changed` event, and binding to another token is refused with `webhook_token_mismatch`.
- **Permissions and parity.**
  - Tests generated from the A03 matrix cover every route above for every staff role, SP-OWN, SP-FIN, TP-AUD, TP-OBS, an investor of each sub-type, and a token acting for each.
  - A token behaves exactly as its human on every maker act and every webhook act, and is refused as checker on the four `4E` paths.
  - Every token-authenticated call leaves one G22 `gate_evaluation`, one activity row and one audit event with `on_behalf_of_id`, and A12's counters agree with the request log.
- **Audit and secrets.**
  - After the full suite, A10's chain verifier passes.
  - The number of `jobs.*`, `integration.*` and `webhook.*` audit events equals the number of catalogue changes, pauses, disables, triggers, replays, discards, configuration transitions, health status changes and subscription changes the suite performed. Deliveries and samples write none.
  - A scan of the logs, the database dump, and the audit, outbox, delivery and evidence stores finds zero `whsec_` values and zero fixture PII.
- **Contracts and hygiene.**
  - Every endpoint above is in the generated OpenAPI document with `x-realx-scope`, `x-realx-gates` (G13 on the pause, disable, discard and config routes; G22 on the webhook routes), `x-realx-mutation` and `x-realx-money` set.
  - `packages/sdk` compiles with typed clients, and the MCP tool catalogue lists the endpoints. No stub route remains in the module.
  - `npm run verify` finds no `Date.now()`, no argument-less `new Date()`, no deep import and no core-table migration in the owned paths.
  - Module line coverage is at least 90 percent.
  - The seed reproduces the ADM-32 rows and the 06:00 recon run exactly.
  - `app/evidence/A46.json` exists and matches disk, and the node QA's `A46.qa.md` records no open Critical or High finding.

## Human gates touched

- None crossed. Every adapter row reports `environment: sandbox`. The configuration path cannot propose a value outside A08's allow-list, so **HG-PROD** (a mainnet chain id or an RPC provider beyond the testnet) and **HG-VENDOR** (any vendor adapter selector) remain open, and are shown in the `human_gate` column on ADM-32 rather than crossed here. **HG-KEYS** is untouched: webhook secrets are sandbox-encrypted through the `kms` adapter's local master key and carry no value.
- **HG-PENTEST (later):** the webhook egress guard (including the fixture-sink exception, which must be unset in any deployed mode), the signature scheme, secret rotation, the replay window, agent-created subscriptions and the configuration-change path are in scope of the external penetration test before go-live. This node passes on the software evidence and records the gate as open.
- No real money, production keys, vendor accounts or mainnet are touched. Every fire, delivery and probe runs on the injected clock against sandbox adapters, the local chain and the fixture sink.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
