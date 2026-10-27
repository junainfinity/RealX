---
title: "A04 - Database schema, migrations and row-level security"
created: 2026-09-23
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am0]
node: A04
milestone: AM0
lane: "Platform"
oracle: yes
human_gate: none
---

# A04 - Database schema, migrations and row-level security

**The one PostgreSQL 17 schema every RealX module writes to: Drizzle tables for every entity in the architecture brief §3 (domains A-E), plus the D04 agent-access tables, the audit table and the alert table. Every table has exactly one owner and a singular name. Every row has `tenant_id`. PII is encrypted with envelope keys from the kms adapter. The legal register is bitemporal, money is integer cents, and every state column is an enum taken from the 14 state machines. Every privileged row carries an approval id that the database checks is approved, for this object and used once, and every audit row carries "on behalf of". Row-level security is defined as eight named read classes plus a separate write class per table, so a person's session (or their agent's) can read its own legal and money rows but never write them directly. The database itself refuses to let an agent (AGNT or AGTU) or a service decide, approve or sign, refuses every agent write to a non-delegable table, and refuses a step-up assertion id that is fake, foreign, reused or for another purpose. The audit table cannot be changed, and migrations are reviewable files.**

**Build status:** not started. **Milestone:** AM0 (Foundations). **Lane:** Platform. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 16-24 agent-hours (build, challenge and verify loops). If the first build loop runs past 12 hours, the orchestrator may split the node at the fix-loop boundary into A04a (schema, conventions, migrations, registry, inventory) and A04b (roles, RLS classes, triggers, auth functions, PII codecs), with A04b depending on A04a (a graph change, raised through escalation below).

**Depends on:** [[A02 - Domain ids, money and unit math]], [[A03 - State machines, gate codes, approval codes and permission matrix as data]], [[A08 - Sandbox adapter framework, config and phase flags]]
**Unlocks:** [[A06 - Seed data and scenario simulator]], [[A09 - API and worker skeleton with OpenAPI-generated SDK]], [[A99 - Developer surface- API versioning, reference docs, per-user sandbox and SDK-MCP packaging]], [[A102 - Backups, restore drill, data retention and erasure]], [[A106 - IAM admin API for ADM-37]]

**Oracle rule:** The node QA writes hidden oracle tests from this spec to `<workspace>\gauntlet\hidden\A04\` before the build starts ([[App Gauntlet]] §2). The builder never reads that folder. This is an oracle node because the money, supply and holdings invariants that the oracle nodes build on, without re-implementing them, live here: the commit-time ledger balance trigger (A16 builds on it), `custody_op.approval_id` `UNIQUE NOT NULL` and the approval binding trigger (the database half of "one approval id binds both"), the bitemporal legal register and its as-of answer (A18), the human-only decider and signer triggers (G13 and G22), the fresh-assertion binding of every non-delegable act (G22), the write classes that keep sessions and agents off legal and money rows, and the tenant isolation of every row. The oracle scope is listed in the Oracle step of the Gauntlet loop below. This plan is not legal advice.

## Build brief

Why this node exists: the database is the only state shared by the API, the worker and the two apps. The OddX QA trail showed that anything enforced "by discipline" eventually breaks (tenancy filters in code, money as floats, audit rows edited by a job). Everything below is enforced by the schema, the roles, the policies and the triggers, and everything can be tested on the build machine against Postgres 17 in Docker from A01's compose file. No vendor is called. The only adapter used is A08's `kms` interface in its sandbox form.

**1. Package shape**

- `<workspace>\app\packages\db\` is the npm workspace `@realx/db`. It holds `package.json`, `drizzle.config.ts`, `tsconfig.json`, `vitest.config.ts`, `README.md` and these scripts:
  - `db:generate`: drizzle-kit emits SQL for tables, indexes and enums.
  - `db:migrate`: forward only, each file in its own transaction, checksum recorded in `__realx_migrations`.
  - `db:reset`: drop and recreate the local database, then migrate. Refused unless `RUN_MODE` is `local` or `test`.
  - `db:check`: see §6.
  - `db:inventory`: writes `<workspace>\app\docs\schema\inventory.md` and its machine-readable twin `<workspace>\app\docs\schema\ownership.json` from the live schema. They list every table with its domain, owner node (core or the registering node), shaping node, consuming nodes, read class, write class, insert-only flag, allowed update columns and every column. The node QA's ownership walk (Challenge) compares every node spec against `ownership.json`, not against prose.
  - `db:seed`: predeclared as `tsx seed/index.ts`. A06 owns `packages\db\seed\` and fills it; A06 never edits `package.json`.
- `src\` root files: `index.ts` exports the whole schema and the constant lists other nodes read:
  - `PII_PATHS`: A09's redaction list. `PII_FIELDS` is an alias.
  - `TABLE_OWNERS`: the ownership map in §2.
  - `RLS_CLASS`: table to `{ read, write }`, where `read` is one or more of the eight read classes and `write` is one of the write classes `S`, `P`, `N`, `A` or `5` (§4). One value per table; there is no table with a single combined class.
  - `TENANT_TABLES` and `PUBLIC_REFERENCE_TABLES`.
  - `PRIVILEGED_TABLES`: table, approval column, the states after which the approval must be set, the A14 action codes it may bind, the column that names its object (`objectRefColumn`, normally the row id), and `batch: true` only where one approval legitimately binds several rows of the table.
  - `APPROVAL_BINDABLE_STATES`: generated from A03's SM-00 export, every state reachable from `approved` that is not a failure or pending state. A test asserts it excludes `draft`, `submitted`, `prechecked`, `rejected`, `vetoed`, `expired` and `cancelled` (A03's names).
  - `INSERT_ONLY_TABLES`: table to its allowed update columns, usually an empty list.
  - `SOFT_DELETE_GUARDED`: money and legal tables with the draft states in which `deleted_at` may still be set (§3).
  - `RUNTIME_PURGEABLE`.
  - `HUMAN_ONLY_ACTOR_COLUMNS`.
  - `AGENT_KINDS`: generated from A03's `principals.ts`, the principal kinds that are agents: today `AGNT` (a RealX staff AI agent) and `AGTU` (a person's own agent acting through an API token, D04). Every CHECK, trigger and policy in this node that asks "is this an agent" tests membership of this set, never a literal `'AGNT'`. A test regenerates it from A03 and fails on drift.
  - `NON_DELEGABLE_TABLES` and `NON_DELEGABLE_COLUMNS`: the tables and columns no agent-kind actor may write (§3).
  - `ASSERTION_COLUMNS`: every column that holds a step-up assertion id, with the owner column it must match and the A11 or A12 purposes it accepts (§3).
  - `WRITE_SERVICES`: table to the named domain service accounts allowed to write it through `withDomainWrite` (§4), and module to the services it may use.
  - `DEFINER_GRANTS`: each SECURITY DEFINER function, its pinned `search_path` and the exact column grants `realx_definer` holds for it (§4).
  - `EXEMPT_TABLES`.
- `src\schema\` holds one file per domain area, plus `enums.ts` (generated from A03) and `_conventions.ts` (the shared column sets and custom column types). The domain files are `tenancy.ts`, `entities.ts`, `properties.ts`, `offerings.ts`, `documents.ts`, `investors.ts`, `identity.ts`, `wallets.ts`, `custody.ts`, `token.ts`, `register.ts`, `rules.ts`, `market.ts`, `recon.ts`, `orders.ts`, `ledger.ts`, `bank.ts`, `distributions.ts`, `tax.ts`, `fees.ts`, `approvals.ts`, `staff.ts`, `agent-access.ts`, `aml.ts`, `support.ts`, `governance.ts`, `comms.ts`, `reporting.ts`, `integrations.ts`, `flags.ts`, `audit.ts`, `alerts.ts`, `runtime.ts`, `retention.ts` and `platform.ts`.
- `src\client\` holds:
  - `createDb(role)`: one pool per login role, with connection strings from A08's validated config.
  - The context wrappers `withTenant(ctx, fn)`, `withPlatform(ctx, fn)`, `withSystem(svcaCtx, fn)`, `withPublic(ctx, fn)` and the nested `withDomainWrite(service, fn)`. They are described in §4.
  - The PII codecs `enc`, `dec`, `blindIndex` and `mask`, bound to the kms adapter.
  - The `cents`, `units` and `bp` column types, bound to A02's branded types. Using a `number` for money is a compile error.
  - Cursor pagination helpers keyed on `(created_at, id)`.
  - `asOf()` for the register (§3).
  - `dbTest()`: a per-test database cloned from a migrated template, so every test starts clean in under a second.
- `src\registry\` holds `registerModuleSchema` and its discovery code (§6).
- `scripts\` holds the `db:*` script sources. `test\` holds the generated suites (§7).
- `migrations\` holds numbered SQL files plus `meta\_journal.json`. These are the drizzle-kit output and the hand-written files for extensions (`pgcrypto`, `btree_gist`), roles, RLS policies, SECURITY DEFINER functions, triggers and constraint triggers. A migration file is immutable once committed. `db:migrate` refuses a file whose checksum has changed, and refuses to apply a file twice.

**2. Table ownership map (single source; every table appears once)**

Rules for the map:

- Table names are singular snake_case. A plural name anywhere in a node spec (`register_entries`, `orders`, `gate_evaluations`, `custody_ops`, `approval_requests`, `ledger_entries` and so on) refers to the singular table below. A plural never creates a second table.
- "Core" tables are created by this node's migrations. The "shaped by" node is the domain node whose spec fixes the columns. This node takes those column sets into the core table now and reconciles them with the invariants below. The builder reads each shaping spec at the Specify step.
- "Registry" tables are created by the named node through the module-schema registry (§6). They are wholly that node's own, and no core table may be altered by them.
- A column that a shaping node needs and does not find here is a spec defect on A04. It is added by a follow-up loop (A04.1, A04.2, ...) through a numbered core migration, never by the shaping node's own migration.

Read classes 1-8 and write classes `S`, `P`, `N`, `A` and `5` are defined in §4. In short: `S` the session's own rows, `P` platform staff or a named domain service only, `N` the human only (never an agent kind), `A` append under the read predicate with no update, `5` platform only.

| Tables (singular) | Owner | Shaped by | Read class | Write class |
|---|---|---|---|---|
| `tenant`, `tenant_user`, `tenant_guardrail` | core | A33 | 1 | P |
| `tenant_consent` | core | A30, A33 | 2 | S |
| `tenant_key` | core | A04 (kms) | 5, read only through `kms_get_wrapped_key` | 5 |
| `scheme`, `vehicle`, `entity` | core | A33, A36, A90 | 1 + 4 (+ 6 for public fields) | P |
| `licence` | core | A21 | 1 + 5 | P |
| `property`, `valuation`, `property_attestation`, `lease`, `income_period` | core | A34 | 1 + 4 (+ 6 for `property` public fields) | P |
| `offering`, `offering_version` | core | A36 | 1 + 4 + 6 | P |
| `document` | core | A35 | 1 + 4 + 6 (published only) | P |
| `document_acknowledgement` | core | A35, A37 (A37's `disclosure_acknowledgements` is this table) | 2 | A |
| `standing_consent` | core | A35, A12 | 2 | N |
| `principal`, `partner_user` | core | A11, A98 | 8 + 1 (sponsor tenant members) + 5 | P |
| `investor`, `investor_classification`, `kyc_case`, `beneficial_owner`, `tax_profile`, `account`, `wallet`, `wallet_key`, `guardian`, `identity_claim` | core | A30, A32, A41, A29, A31, A11 (`account` columns) | 3 | P (`wallet_key`, `guardian` and the `NON_DELEGABLE_COLUMNS` of `account` also refuse every agent-kind actor) |
| `recovery_request` | core | A29, A28 | 3 | P, and refuses every agent-kind actor |
| `passkey`, `session` | core | A11 | 8 | N |
| `fresh_assertion` (nonce rows and assertion rows) | core | A11 | 8 + 5 | N, insert-only apart from the consumption columns (§3) |
| `custody_account` | core | A15 | 5 | 5 |
| `service_account` | core | A09, A04 | 5 | 5 (migration only for the grant columns) |
| `token` | core | A27, A28 | 1 + 4 + 6 (address and supply) | P |
| `register_entry`, `register_lot` | core | A18 | 2 + 4 | P |
| `register_version`, `register_instruction`, `register_instruction_line` | core | A18 | 1 + 4 | P |
| `holding` | core | A27 | 2 + 4 | P |
| `rule_pack`, `compliance_rule` | core | A19 | 6 (published) + 5 | 5 |
| `gate_evaluation` | core, partitioned by month | A19, A12 | 2 | A |
| `threshold_monitor` | core | A19 | 1 | P |
| `halt` | core | A19, A48 | 1 + 6 | P |
| `transfer_request` | core | A28 | 2 | P |
| `instrument` | core (P2) | A91 | 1 + 6 | P |
| `resale_order`, `trade` | core (P2) | A91 | 2 | P |
| `recon_run`, `recon_break` | core | A20 | 1 | P |
| `order`, `allocation_line` | core | A37 | 2 | P |
| `allocation` | core | A37 | 1 | P |
| `ledger_account`, `ledger_entry` | core | A16 | 2 for rows with an investor, otherwise 1 (all ledger rows are in the platform tenant, so in practice staff and listed services only) | P |
| `ledger_journal`, `ledger_snapshot` | core | A16 | 1 (platform tenant) | P |
| `bank_transaction` | core | A17, A20 | 1 (platform tenant, so staff and listed services only) | P |
| `payment`, `withdrawal` | core | A17, A39 | 2 | P |
| `distribution_run` | core | A40 | 1 | P |
| `distribution_line`, `tax_component`, `amma_statement` | core | A40, A41 | 2 | P |
| `fee_schedule`, `fee_ledger`, `invoice` | core | A62, A74 | 1 | P |
| `approval_request`, `approval_policy` | core | A14 | 1 | P |
| `approval_decision` | core | A14 | 1 | P (plus the human-only trigger) |
| `custody_op` | core | A14 (columns), A15 (columns it needs, now folded in) | 1 | P |
| `custody_signature` | core | A15 | 1 | P (plus the human-only trigger) |
| `sod_rule`, `staff_user` | core | A11 | 5 | 5 |
| `role` | core | A11 | 6 | 5 |
| `role_assignment`, `access_review` | core | A11, A67 | 1 | P |
| `break_glass_session` | core | A11, A67 | 7 | P (plus the human-only trigger) |
| `aml_alert`, `aml_case` | core | A42 | 7 | P |
| `complaint`, `ticket` | core | A44 | 2 | P |
| `proposal` | core (P2) | A93 | 1 + 6 | P |
| `vote` | core (P2) | A93 | 2 | P |
| `marketing_item`, `announcement`, `claim`, `template` | core | A43, A64 | 1 + 6 (published only) | P |
| `regulatory_obligation`, `filing` | core | A45 | 1 | P |
| `notification` | core | A43 | 8 | S (read state only; the content columns are P) |
| `envelope` | core | A35 | 1 | P |
| `integration`, `job_run` | core | A46, A09 | 5 | 5 |
| `webhook_subscription` | core | A46 | 8 + 5 | S |
| `webhook_delivery` | core | A46 | 8 + 5 | 5 |
| `feature_flag` | core | A21, A08 | 6 (read) + 5 | 5 |
| `retention_policy`, `legal_hold` | core | A102, A10 | 5 | 5 |
| `erasure_request` | core | A102 | 2 + 5 | N |
| `idempotency_key` | core | A09 | 8 + runtime (§4) | S |
| `outbox`, `processed_event`, `job_schedule_fire`, `system_clock` | core | A09 | runtime (§4) | runtime (§4) |
| `audit_event` | core | A10 | its own policy (§4) | A, with its own `WITH CHECK` (§4) |
| `alert` | core | A13 | 1 | P |
| `api_token`, `agent_principal`, `agent_mandate` | core | A12 | 8 + 1 (sponsor tokens, for SP-OWN of the tenant) + 5 (ADM-35) | N |
| `api_token_activity` | core | A12 | 8 + 1 + 5 | A |
| `audit_partition`, `audit_anchor`, `audit_export` | registry | A10 | A10's declaration (`audit_partition` and `audit_anchor` insert-only) | A10's declaration |
| `email_verification`, `view_as_tenant_grant`, `login_recovery_request`, `sod_exception`, `delegation`, `auth_mail_capture` | registry | A11 | A11's declaration | A11's declaration |
| `agent_counter`, `mandate_signing_key`, `agent_confirmation`, `agent_rate_bucket`, `agent_switch` (every tier: platform, tenant, owner type, user), `agent_policy` | registry | A12 | 8 + 1 + 5 (A12's declaration) | A12's declaration; A101 writes the tenant, owner-type and user rows of `agent_switch` through A12's table |
| `partner_engagement` | registry | A98 | 1 | P |
| `alert_type`, `sla_policy` | registry | A13 | 6 | 5 |
| `alert_activity`, `escalation_path`, `alert_digest` | registry | A13 | 1 | P |
| `agent_anomaly`, `notice_policy` | registry | A101 | 8 + 1 + 5 (`agent_anomaly.source` and `signal_ref` read only through a class-7 view) | P |
| `order_state_history`, `order_quote`, `order_confirmation` | registry | A37 | 2 | P |
| `register_extract` | registry | A18 | 1 + 4 | P |
| `investor_override` | registry | A19 | 5 | 5 |
| `custody_signer`, `custody_policy_evaluation`, `custody_key_rotation` | registry | A15 | 5 | 5 |
| `ledger_reservation`, `ledger_journal_type`, `ledger_approval_draw`, `ledger_snapshot_line` (if A16 needs it) | registry | A16 | 2 / 6 / 1 / 1 | P |
| `system_echo` | registry | A09 | A09's declaration | A09's declaration |

**Final decisions on contested tables (24 Sep 2026; the other specs follow these, and any spec that says otherwise is flagged in §2's list below):**

- **D04 tables.** Core: `api_token`, `agent_principal`, `agent_mandate`, `api_token_activity`. Registered by A12 and wholly A12's: `agent_counter` (the daily usage counters; there is no `mandate_usage` table), `mandate_signing_key` (the mandate has no `signing_key` column), `agent_confirmation`, `agent_rate_bucket`, `agent_switch` for every tier including the platform tier, and `agent_policy`. A101 registers only `agent_anomaly` and `notice_policy`, and writes the tenant, owner-type and user rows of A12's `agent_switch`. The concurrency floor on the daily and monthly caps is A12's advisory lock over `agent_counter`, tested in A12.
- **`api_token` state.** The column is `state`, with A12's values `pending_cosign | pending_owner_approval | active | held | revoked | expired`. The API vocabulary `pending | active | suspended | revoked | expired` is A12's mapping and never a column.
- **Audit.** `audit_event` is core, with A10's column list folded in now (§3). `audit_partition`, `audit_anchor` and `audit_export` are A10's registered tables. There is no `audit_chain_head` table and no `chain_partition` column: a partition is defined by `sandbox_of` (null for `live`), as A10 defines it, and the head is read by `audit_next_link` directly from `audit_event` (§4).
- **Ledger tenancy.** Every ledger row (account, journal, entry, snapshot and A16's registered rows) carries the platform tenant, as A16 states. Rows that concern a sponsor vehicle carry `sponsor_tenant_id`, `vehicle_id` and `offering_id` as plain attributes; they never make a row readable by the sponsor. Sponsors read their own figures through A16's module, not the tables.
- **Assertions.** `fresh_assertion` is core (shaped by A11), not registered, so the core tables can hold a foreign key to it.

Reconciliations fixed now (the shaping nodes are flagged for a matching edit):

- **Register (A18).** A18's bitemporal columns replace this node's earlier draft: `effective` is a `tstzrange` holding A18's `valid_from` and `valid_to`, and `recorded` is a `tstzrange` holding `record_from` and `record_to`. The row also carries A18's `holder_display_name_enc`, `holder_type`, `classification`, `residency`, `units`, `frozen_units`, `wallet_address`, `legal_record_ref`, `source_instruction_id`, `register_version_id` and `approval_id`. Lots are rows in `register_lot`, with A18's lot columns plus `parent_lot_id` and `register_entry_id`. They are never a JSON column on the entry, so A18 drops `lots[]` and reads lots through a join. A lot is never edited: a change to a holding inserts a successor entry with its own lots, linked by `parent_lot_id`. `register_version` takes A18's columns. `register_instruction` stores `lines` as rows in `register_instruction_line`, not an array.
- **Orders (A37).** `order` takes A37's column list, with `lines[]` of an allocation held in `allocation_line`. `order.id` uses the `RX-ORD-` prefix from A02.
- **Ledger (A16).** Every ledger row is in the platform tenant, by `CHECK` against the platform tenant id. Investor cash accounts carry `investor_id`, which puts them in read class 2, so no investor can read another investor's account. A16's A04.1 list is folded into core now: `ledger_account.is_client_money` (derived from `account_type` by CHECK), `sponsor_tenant_id`, `vehicle_id`, `offering_id`, `frozen_approval_id`; on `ledger_journal` the origin columns (`origin_principal_kind`, `origin_principal_id`, `origin_token_id`, `origin_on_behalf_of_id`), `posting_seq` (gapless), `reversal_of` and `reversed_by` (set once), `UNIQUE (tenant_id, journal_type, external_ref)` and a partial unique index on `approval_id` for the journal types A16's catalogue marks single-use; on `ledger_entry` `balance_after_cents`, `line_ref`, `beneficiary_id` with an index on `(account_id, line_ref)`.
- **Rules (A19).** `gate_evaluation` is range-partitioned by month on `evaluated_at`, with primary key `(id, evaluated_at)`. The core migration creates the monthly partitions to December 2029 plus a `DEFAULT` partition, and a later numbered migration extends them. Every partition has RLS enabled and forced, and the app roles have no grant on a partition directly.
- **Approvals and custody (A14, A15).** `custody_op` carries A14's columns (`vault_code`, `target`, `selector`, `calldata_sha256`, `excluded_signers[]`, `consumed`) plus `approval_payload_sha256` (a copy of the bound request's `payload_sha256`, checked by the binding trigger in §3) and the columns A15's spec adds. A15 adds no columns by its own migration. `approval_decision` carries A14's columns including `fresh_assertion_id` (FK to `fresh_assertion`) and `bulk_batch_id`.
- **Runtime (A09).** The outbox table is named `outbox` and carries A09's column list. A09's and A85's `outbox_event` means this table. `system_clock`, `processed_event` and `job_schedule_fire` are core tables here, because this node's `realx_now()` reads `system_clock` (§3). A09 declares none of them. `idempotency_key` takes A09's full column list now (§3).
- **Tokens and mandates (A12).** A12's A04.1 list is folded into core now (§3): every `api_token`, `agent_mandate` and `api_token_activity` column A12 §7 names, under A12's names.
- **Audit (A10).** A10's A04.x items 1 to 8 and 10 are folded into core now (§3 and §4); item 9 is resolved by A10 declaring its two insert-only tables through the registry flag; item 11 is included as an optional CHECK.
- **Authentication (A11).** A11's requested columns on `session`, `passkey`, `account`, `principal` and `role_assignment` are folded into core now under A11's names, and the `role_assignment` constraint trigger for A03's SoD rules 2, 3 and 5 is built here from role sets generated from A03's export.

**3. Tables and conventions enforced by the schema, not by discipline**

- **Standard columns.** Every table carries `tenant_id`, `created_at`, `created_by` (a principal), `updated_at`, `updated_by`, `deleted_at` (soft delete) and `sandbox_of` (null for live rows; the owning principal id for A99's per-user sandbox partition).
  - Insert-only tables (below) carry `tenant_id`, `created_at`, `created_by` and `sandbox_of` only. They have no `updated_*` or `deleted_at`, because they are never updated or soft-deleted, apart from their listed columns.
  - The only full exemptions are `__realx_migrations`, `system_clock`, the `pgboss` schema, and the reference tables listed in `EXEMPT_TABLES`. The list is a constant the tests read, and adding to it needs a written reason in the migration.
  - `tenant_key` is no longer exempt from RLS (§4).
- **Hard delete.** A trigger refuses a hard `DELETE` for every role with three exceptions:
  - `realx_retention`: A102 grants it later. It is created now as `NOLOGIN` with no grants.
  - `realx_worker` on the `RUNTIME_PURGEABLE` tables `idempotency_key`, `processed_event`, `job_schedule_fire` and `fresh_assertion`. Each of these has `expires_at NOT NULL`, and the trigger allows the delete only when the row's `expires_at < realx_now()`, and for `fresh_assertion` only when the row is also unconsumed (`consumed_at IS NULL`); a consumed assertion is never purged, because a token, mandate or decision points at it. `realx_now()` returns `now()` plus the offset in `system_clock` when `RUN_MODE` is `local` or `test`, so A09's clock advance and the sweep agree. `realx_worker` holds `DELETE` on those four tables and nowhere else.
  - The sandbox audit purge A10 item 10 names: the NOLOGIN role `realx_audit_purger` (created now with no grants; A99 gives it its one function) may delete `audit_event` rows only when `current_user = 'realx_audit_purger'`, the transaction-local setting `realx.audit_purge` is `on`, and the row's `sandbox_of` equals the setting `realx.audit_purge_partition` (never a live row).
- **Soft delete of money and legal rows.** For each table in `SOFT_DELETE_GUARDED` (`order`, `allocation`, `allocation_line`, `payment`, `withdrawal`, `distribution_run`, `distribution_line`, `transfer_request`, `recovery_request`, `register_instruction`, `valuation`, `custody_op`, `approval_request`, `halt`, `tax_component`, `resale_order`, `trade`), a `BEFORE UPDATE` trigger refuses setting `deleted_at` unless the row's state is one of the table's declared draft states (for example `order` in `draft`, `approval_request` in `draft`); outside those it raises `<table>_soft_delete_refused`. Insert-only tables have no `deleted_at` at all. A withdrawal or payment past its first approval therefore cannot disappear from a list.
- **Money, units and percentages.** Money is `cents` (`bigint`, with a `CHECK` for non-negative where the domain says so). Units are `bigint` with `CHECK (units >= 0)`. Percentages are basis points (`integer`, with a `CHECK` between 0 and 10,000 where the domain says so). A verify test scans `information_schema.columns` and fails on any `numeric`, `real`, `double precision` or `money` column in any schema the migrations create.
- **State columns.** Every state column is a `pgEnum` generated by `enums.ts` from A03's `domain:export` JSON. That covers:
  - the 14 machines SM-00..SM-13, with every state A03's export lists, including its extensions
  - the gate codes G01-G22
  - the approval policy codes and the vault codes
  - the roles and the principal kinds (HUMAN, PARTNER, AGNT, AGTU, SVCA today), exactly as A03's `roles.ts` and `principals.ts` export them
  - the local lifecycles that A03 or the owning node exports in the engine format (api token, mandate, register instruction, audit result)
  - alert severities and the custody op states

  This spec deliberately states no counts. The counts in `enums.ts` and in every test are read from A03's export at generation time, so a change in A03 never leaves a stale number here. A drift test regenerates `enums.ts` from A03 and fails if the committed file or the database differs. A state added in A03 fails verify until a migration is generated. No state column is free text.
- **PII.** PII lives only in `_enc` columns (`bytea`): names, dates of birth, addresses, email, phone, TFN and ABN, document numbers, BSB and account numbers, PayID and bank references.
  - Envelope encryption uses one data-encryption key per tenant per purpose, from A08's `kms.generateDataKey`. The sandbox uses a local encrypted store with a dev master key that carries no value.
  - The wrapped key is in `tenant_key`. Every encrypted row has `enc_key_version`, so re-keying and per-tenant destruction never touch another tenant.
  - The tenant id is bound into the AAD, so a ciphertext cannot be decrypted under another tenant's key.
  - Lookups use HMAC blind-index columns keyed per tenant (`email_bidx`, `tfn_bidx`, `abn_bidx`, `phone_bidx`).
  - Nothing personal has a plain-text column, and no `_enc` column is ever indexed.
  - The register's `holder_display_name_enc` is encrypted under the sponsor tenant's key, because the sponsor and its trustee must be able to read their own register (SPN-08).
- **Masking.** `mask()` is the only path that renders another person's value, or a value on a staff or sponsor screen. The formats are a spec decision, fixed by snapshot tests against the prototype's strings:
  - TFN: `••• ••• 482`, the last three digits (ONB-05 shows exactly this).
  - Bank account: `••••4471` (ONB-06 account number) and `CBA ••4471` (institution plus last four, as the POR-05 ledger and the ADM-30 withdrawal row show).
  - Treasury accounts: `Bank A ••2201` (ADM-18).
  - BSB: shown unmasked, `062-000`, as the prototype does. The BSB is still stored encrypted.
  - Status-only renders: `TFN on file` and `TFN provided`, where the screen shows a status instead of a value.
  - Password: `••••••••••`.

  The owner's own contact fields (email, mobile, PayID `alex.morgan.841@pay.realx.au`) are shown unmasked to the owner alone, through `dec` under the owner's session, as the prototype's ONB review does. That read is never available to a sponsor, a partner or an agent without the owner's read scope.
- **Bitemporal legal register.** `register_entry` uses A18's `effective` and `recorded` ranges.
  - A `btree_gist` exclusion constraint forbids two open rows (`upper(recorded)` infinite or null) of the same `(vehicle_id, investor_id)` whose `effective` ranges overlap.
  - Rows are never edited. A correction inserts a successor and closes the old row's `recorded` upper bound, and a trigger refuses any other `UPDATE`.
  - `asOf(vehicle, validAt, knownAt)` answers "what did the register say on date X as known on date Y".
  - Escrow balances are never register entries (design bug 14): the funded orders of an open offer live in `order` and in the offer escrow `ledger_account`. A constraint trigger on `register_entry` refuses an entry whose source instruction is of kind `issue` and references an allocation, unless that allocation's offering is in the SM-04 state `finalising` or `minted` (A03 names; allocation is locked at `finalising`, and A18 and A38 write the register before the mint). Entries of kind `transfer`, `redeem` and `adjust`, and issues for other offerings of the same vehicle, are not affected by a second offering being open.
- **Privileged rows and approval ids.** These rows carry `approval_id` (an FK to `approval_request`): `token.deploy_approval_id`, `valuation`, `withdrawal`, `distribution_run`, `transfer_request`, `recovery_request`, `register_version` (the certification approval), `register_instruction` of kind adjust, redeem or issue (A18), `allocation`, `halt`, `feature_flag`, `approval_policy` versions, `tenant` admission, `role_assignment` and `legal_hold`.
  - `custody_op.approval_id` is `NOT NULL` and `UNIQUE`, so one approval can never bind two custody operations. This is the database half of "one approval id binds both".
  - `PRIVILEGED_TABLES` records, per table, the A03 state after which `approval_id` must be set, and a CHECK per table enforces it: a `withdrawal` in `approved` with a null approval id cannot exist.
  - **Binding trigger.** A constraint trigger `approval_binding_check` (deferred to commit) on every `PRIVILEGED_TABLES` member and on `custody_op` runs whenever the approval column is set or changed, and refuses with `approval_binding_invalid` unless all of these hold:
    - the `approval_request` exists and its `state` is in `APPROVAL_BINDABLE_STATES` (so a draft, pending, rejected, vetoed, expired or cancelled request cannot be cited);
    - its `action_code` is one of the action codes `PRIVILEGED_TABLES` lists for this table;
    - its `object_ref` equals `<table>:<value of objectRefColumn>` for this row (for `custody_op`, the `object_ref` of the business object the op executes, with `approval_request.custody_op_id` either null or this op's id), so an approval for another object or another table is refused;
    - its `tenant_id` equals the row's `tenant_id`, or the row is in the platform tenant and the request is platform-level;
    - for `custody_op`, `approval_payload_sha256` equals the request's `payload_sha256`. The database does not compare `calldata_sha256` with the payload hash, because A15 derives the calldata by encoding the canonical payload (`calldata_sha256 = sha256(encode(payload_canonical))`) and P1 in A15 recomputes and checks that match; the database binds the op to the exact approved payload version, and A15 binds the calldata to the payload.
  - **No reuse.** Every privileged table has a partial `UNIQUE (approval_id) WHERE approval_id IS NOT NULL`, except the tables marked `batch: true` in `PRIVILEGED_TABLES` (for example `allocation_line` rows of one allocation approval). Together with the object check, one approval cannot be cited by two withdrawals, by two tables, or by a withdrawal and a custody op for different objects.
- **Insert-only tables.** Each member of `INSERT_ONLY_TABLES` has a `BEFORE UPDATE OR DELETE` trigger, built from one generic function `realx_insert_only(allowed_columns)`. It refuses any `DELETE`, and any `UPDATE` that changes a column outside the allowed list, for every role including the table owner. A statement-level trigger blocks `TRUNCATE`. The members and their allowed columns:
  - `audit_event`, `gate_evaluation`, `api_token_activity`, `approval_decision`, `ledger_entry`, `ledger_snapshot`, `custody_signature`, `register_lot`, `register_instruction_line`, `document_acknowledgement`: none.
  - `ledger_journal`: `reversed_by`, settable once from null (A16's contra-journal link); every other column, including the amounts and the approval id, never changes.
  - `register_entry`: the upper bound of `recorded`, settable once from open.
  - `fresh_assertion`: `consumed_at`, `consumed_txid` and `consumed_by_ref`, each settable once from null (§3, step-up assertions).
  - `register_version`: `state`, `certified_by`, `certification_approval_id`, `superseded_by_id`. The trigger also enforces A18's forward-only transitions, and once `certified` only the move to `superseded` with `superseded_by_id` is allowed, so a certified version is otherwise immutable.
  - `bank_transaction`: `match_state`, `matched_ref`, `matched_at`, `matched_by`. These are the columns A17 and A20 update; the raw bank fields never change.
  - `outbox`: `published_at`.
  - `agent_mandate`: `state`, `superseded_by_id`, `closed_at`, used only to close a version.

  Registered tables may join the list through `registerModuleSchema({ insertOnly })`. A37's `order_state_history`, A10's `audit_partition` and `audit_anchor`, and A12's `agent_confirmation` (forward-only state columns) do this.
- **Ledger integrity.** A deferred constraint trigger checks at commit that:
  - every `ledger_journal` balances (sum of debits equals sum of credits, in integer cents)
  - no `ledger_entry` exists without a journal
  - every entry's `tenant_id` equals its journal's, and both equal the platform tenant (A16's tenancy model; a CHECK on each ledger table also pins `tenant_id` to the platform tenant id)

  A16 builds on this and does not re-implement it. The trigger function is SECURITY DEFINER (§4), so it sees every entry of the journal whatever session posts it.
- **Tenant guardrail floor.** `tenant_guardrail` carries `CHECK (kyc_required = true)`: no sponsor tenant can be configured to admit investors without KYC, whatever a later screen or migration tries.
- **Indexes and time.** Every foreign key has an index, and a verify test finds any FK without one. Every list surface's cursor key has a composite index on `(tenant_id, created_at, id)`. All timestamps are `timestamptz`. `Australia/Sydney` calendar days (for example A12's `agent_counter.sydney_day`) are computed in the client, never stored as a default.
- **Runtime tables** (A09's columns, taken exactly):
  - `idempotency_key`: `principal_kind`, `principal_id`, `token_id`, `operation_id`, `key`, `body_sha256`, `state` (`in_progress | completed`), `claim_token`, `locked_until`, `status`, `headers`, `body_enc` (`bytea`, through `enc`), `completed_at`, `expires_at`, with `UNIQUE (principal_id, operation_id, key)`.
  - `outbox`: `id` ULID, `type`, `occurred_at`, `aggregate_type`, `aggregate_id`, `tenant_id`, `actor_id`, `actor_kind`, `on_behalf_of_id`, `token_id`, `approval_id`, `request_id`, `schema_version`, `payload`, `published_at`.
  - `processed_event`: `event_id`, `handler`, `expires_at`.
  - `system_clock`: single row, `offset_ms`.
  - `job_schedule_fire`: unique `(job, due_at)`, `expires_at`.
  - `job_run`.
  - The `pgboss` schema, created by pg-boss under the worker role.

**Audit table (A10's contract, taken exactly; A10's A04.x items 1 to 8, 10 and 11 are folded in here):**

- `audit_event` columns:
  - chain and time: `seq` (bigserial), `id`, `ts` and `created_at` (both `timestamptz(3)`, millisecond precision), `tenant_id`, `sandbox_of` (null for the `live` partition; the partition is defined by this column, and there is no separate partition column)
  - actor: `actor_type` (a principal kind), `actor_id`, `actor_role`, `on_behalf_of_id`, `token_id`, `session_id`, `request_id`, `trace_id`
  - subject: `action`, `object_type`, `object_id` (A10 derives the API field `object_ref` as `<object_type>:<object_id>`; it is not stored), `subject_principal_id` (nullable FK to `principal`: the person the event is about), `vehicle_id` (nullable: the trust or sub-fund the event concerns, for partner scope)
  - visibility: `visibility` (enum `subject_visible | staff | need_to_know | security`, `NOT NULL`), `need_to_know_acl` (text[] of role codes; `CHECK (visibility <> 'need_to_know' OR (need_to_know_acl IS NOT NULL AND cardinality(need_to_know_acl) > 0))`)
  - links: `approval_id`, `custody_op_id`, `gate_evaluation_id` (nullable FK to `gate_evaluation`), `tx_hash`
  - hashes: `before_sha256`, `after_sha256`, `before_ref`, `after_ref`
  - outcome: `reason_code` (checked by A10's `record` against its closed registry) and `reason_params` (jsonb, nullable); no free-text reason column. `result` (a `pgEnum`: `ok | referred | alert | refused`), `gate_codes[]`, `rule_pack_version`, `sensitivity`, `canonical_version`
  - chain hashes: `prev_event_sha256` (`NOT NULL`, `CHECK (length = 64)`; the first row of a partition carries A10's genesis value `sha256_hex("realx-audit-genesis:" + partition_key)`), `event_sha256` (`CHECK (length = 64)`)

  There are no label columns (`actor_label` and `object_label` do not exist). Display labels are resolved at read time by A10, so no name ever enters the immutable table. `audit_event` has no `updated_at`, `updated_by` or `deleted_at`.
- `audit_event` constraints:
  - `UNIQUE (prev_event_sha256)` across the whole table: no fork in any chain, and no second genesis. Genesis values differ per partition key (A10 puts the sandbox generation in the key), so partitions never collide.
  - `UNIQUE (seq)`.
  - `CHECK (actor_type NOT IN (<AGENT_KINDS>) OR on_behalf_of_id IS NOT NULL)` and `CHECK (token_id IS NULL OR on_behalf_of_id IS NOT NULL)`: an event by an AGNT or an AGTU without a human behind it cannot be written. The list is generated from `AGENT_KINDS`.
  - Optional second net (A10 item 11): `CHECK (actor_type NOT IN (<AGENT_KINDS>) OR action NOT IN (<the human-only action codes A10 exports at that migration>))`, generated; A10's `record` refusal is the primary net.
  - Indexes: `(sandbox_of, seq)`, `(tenant_id, seq)`, `(actor_id, seq)`, `(on_behalf_of_id, seq)`, `(subject_principal_id, seq)`, `(token_id, seq)`, `(object_type, object_id, seq)`, `(approval_id)`, `(action, ts)`, `(vehicle_id, seq)`.
- `audit_next_link(partition_key)` is a SECURITY DEFINER function (§4). It takes `pg_advisory_xact_lock(hashtext('audit:' || partition_key))`, the same key A10's flush uses, and returns `(seq, event_sha256)` of the highest-`seq` row of that partition, read directly from `audit_event` as `realx_definer`, bypassing per-tenant RLS; it returns no row for an empty partition, and A10 then uses the genesis value from its `audit_partition` row. This gives one linear chain per partition however many tenants write to it. A10's flush calls this function instead of its own `ORDER BY seq DESC LIMIT 1` read, which under a tenant session would see only that tenant's rows and fork the chain.
- `audit_partition`, `audit_anchor` and `audit_export` are A10's registered tables, with A10's columns; this node gives them the registry, the insert-only flag and the convention tests, nothing else.
- Hash computation is A10's. This node gives it columns that cannot lie.

**Alerts.** `alert` holds the row A13 specifies: `ALT-` id, `type_code`, `severity`, `subject_ref`, `dedup_key`, `occurrences`, `ack_due_at`, `breached_at`, `snoozed_until`, and `emitted_by` with actor and on-behalf-of.

**Step-up assertions (the database half of the non-delegable acts).** `fresh_assertion` is core, shaped by A11 (A11's single table holding nonce rows and consumed-assertion rows):

- Columns: `id`, `row_kind` (`nonce | assertion`), `principal_id`, `purpose`, `payload_sha256`, `server_nonce` (nonce rows), `nonce_id` (assertion rows; `UNIQUE`, so a nonce is used once), `credential_id`, `aaguid`, `issued_at`, `expires_at`, `consumed_at`, `consumed_txid` (`xid8`), `consumed_by_ref` (`<table>:<id>` of the row that consumed it), plus the insert-only conventions. Insert-only apart from the three consumption columns, each set once from null.
- `ASSERTION_COLUMNS` lists every column that cites an assertion, with its owner column and accepted purposes (the purpose names are A11's and A12's exports, generated, never typed here):
  - `api_token.created_with_assertion_id` (owner `owner_principal_id`), `api_token.widened_with_assertion_id` (owner `owner_principal_id`), `api_token.cosigner_assertion_id` (owner `cosigner_principal_id`), `api_token.approver_assertion_id` (owner `approver_principal_id`)
  - `agent_mandate.assertion_id` (owner `on_behalf_of_id`, and the assertion's `payload_sha256` must equal the row's `mandate_sha256`)
  - `approval_decision.fresh_assertion_id` (owner `decider_id`; A14's bulk-decide exception: one assertion may be cited by every decision row sharing one `bulk_batch_id`, and by no other row)
  - `standing_consent.assertion_id`, `erasure_request.assertion_id` and the change assertion on `NON_DELEGABLE_COLUMNS` (owner: the person whose row it is)
  - registered tables add their own columns through `registerModuleSchema({ assertionColumns })` (A12's `mandate_signing_key` and `agent_confirmation`, A29's recovery and guardian rows).
- Each listed column is a FK to `fresh_assertion(id)`, and a constraint trigger `assertion_binding_check` refuses the write with `assertion_invalid` unless:
  - the assertion row is of kind `assertion`;
  - its `principal_id` equals the row's owner column for that assertion;
  - that principal's kind is HUMAN or PARTNER (never an agent kind or SVCA);
  - its `purpose` is one the column accepts;
  - it is consumed in this same transaction: the trigger sets `consumed_at`, `consumed_txid = pg_current_xact_id()` and `consumed_by_ref` when they are null, and refuses when they are already set by another row (reuse) or by another transaction;
  - it is not past `expires_at`.
- A random id fails the FK. A foreign principal's assertion, a reused one and a wrong-purpose one fail the trigger.

**D04 agent access** (single source for A12, A60, A77, A99, A101 and A104; the names are A12's, A12 §7, and its A04.1 list is folded in here):

- `api_token` columns:
  - identity: id `TOK-` (A02), `owner_principal_id` (a HUMAN or PARTNER principal; staff are HUMAN), `agent_principal_id`, `tenant_id`, `agent_name`
  - secret: `lookup` (12 base32 chars, unique, indexed), `prefix` (`CHECK (prefix ~ '^rx_(inv|spn|ptr|stf|sb)_$')`), `secret_hash` (Argon2id, unique, never the secret), `prev_secret_hash` and `prev_secret_valid_until` (the rotation grace window), `row_version` (A12's memo key and optimistic checks)
  - rights: `scopes` (an array of A03 matrix cell ids), `ip_allowlist`, `environment` (`live` or `sandbox`; `sandbox` forces `prefix = 'rx_sb_'` by CHECK), `expires_at`. Money, unit and rate limits live only on the mandate, so the database can CHECK them.
  - usage: `last_used_at`, `last_used_ip`
  - lifecycle: `state` (A12's `pending_cosign | pending_owner_approval | active | held | revoked | expired`), `pending_expires_at`, `hold_id`, `hold_class`, `revoked_at`, `revoked_by`, `revoke_reason`
  - second person: `cosigner_principal_id`, `cosigner_assertion_id` (the joint "both to sign" co-signer), `approver_principal_id`, `approver_assertion_id` (the sponsor or staff second approver), `approval_id` (A14's request, once A12.1 wires it)
  - assertions: `created_with_assertion_id NOT NULL`, `widened_with_assertion_id`
- `api_token` checks:
  - `CHECK (cosigner_principal_id IS NULL OR cosigner_principal_id <> owner_principal_id)` and the same for `approver_principal_id`.
  - `CHECK (NOT (state IN ('active','held') AND pending_expires_at IS NOT NULL AND cosigner_assertion_id IS NULL AND approver_assertion_id IS NULL AND approval_id IS NULL))`: a token that was pending cannot become active without its second person.
  - `CHECK (state NOT IN ('pending_cosign','pending_owner_approval') OR pending_expires_at IS NOT NULL)`.
  - The trigger `api_token_no_inplace_widening` refuses an `UPDATE` that widens the token in place, unless the same statement sets a new, different, non-null `widened_with_assertion_id` (which the assertion trigger then binds). Widening means a scope not in the old set, a later `expires_at`, or a looser `ip_allowlist` (entries added, or set to null). Narrowing needs no assertion.
  - The state column only moves along A12's transition table; the trigger refuses any other move with `api_token_illegal_transition`.
- `agent_principal` binds an agent `principal` row to `owner_principal_id NOT NULL`. A constraint trigger checks that the row's own principal kind is in `AGENT_KINDS` (A03: a person's token agent is `AGTU`, a RealX staff AI agent is `AGNT`) and that the owner's kind is HUMAN or PARTNER.
- `agent_mandate` columns: id `MND-`, `token_id`, `agent_principal_id`, `on_behalf_of_id NOT NULL`, `scopes`, `allowed_actions`, `per_action_cap_cents`, `daily_cap_cents`, `monthly_cap_cents`, `per_action_units`, `daily_units`, `monthly_units`, `rate_limit_reads_per_min`, `rate_limit_writes_per_min`, `allowed_vehicle_ids`, `allowed_property_ids`, `time_windows`, `expires_at`, `version`, `supersedes_id`, `superseded_by_id`, `session_key_policy` (jsonb, nullable, the on-chain delegated session-key limits A29 registers) and `session_key_policy_sha256`, `standing_consent_id` and `standing_consent_coverage` (jsonb list of document type, id and version), `mandate_sha256 NOT NULL`, `assertion_id NOT NULL`, `state` (`active | superseded | revoked | expired`).
  - A mandate is insert-only apart from closing a version. Every change, and so every widening, inserts a new version with its own assertion over its own `mandate_sha256`, and the old version stays readable.
  - CHECKs: `per_action_cap_cents <= daily_cap_cents <= monthly_cap_cents` and `per_action_units <= daily_units <= monthly_units`, all non-negative.
  - The signing key is A12's registered `mandate_signing_key`, not a column here.
- `api_token_activity` is insert-only: token, timestamp, operation id, outcome, `error_code`, `client_label` (`api | mcp | sdk`, from A104's `X-RealX-Client`), `gate_evaluation_id`, ip.
- Kill switches:
  - The platform-wide flag is the `feature_flag` row `agents.traffic` (default on, `requires_licence = false`), turned off only by A48's effector through A21.
  - A12 registers `agent_switch` for every tier (platform, tenant, owner type, user); A12's interim platform effector writes the platform row, and A101 writes the tenant, owner-type and user rows.
  - The counters, confirmations, rate buckets and policies are A12's registered tables (§2).

**Feature flags.** `feature_flag` carries `requires_licence boolean NOT NULL` and `licence_bound_to`, with `CHECK (NOT (state = 'on' AND requires_licence AND licence_bound_to IS NULL))`. A21 sets `requires_licence` for the perimeter flags. The phase flags and `agents.traffic` do not require a licence.

**Human-only deciders and signers (the database half of G13 and G22).**

- `HUMAN_ONLY_ACTOR_COLUMNS` names:
  - `approval_decision.decider_id`
  - `custody_signature.signer_principal_id`
  - `break_glass_session.activated_by`
  - `break_glass_session.approved_by`
- A constraint trigger on each of these tables refuses a row in any of these cases:
  - The actor's principal kind is not HUMAN or PARTNER. The one exception: an `approval_decision` with `credential_kind = 'auto'` whose decider is an SVCA and whose request's policy code is `AUTO`, or `SGL+R` with verdict `act` and a `trigger_ref`, exactly as A14's credential CHECK specifies.
  - `acting_token_id` or `on_behalf_of_id` is set on the row. Both columns exist, are filled from the request context by the write path, and must be null.
  - The session context says `app.actor_type` is in `AGENT_KINDS`, or `app.token_id` is set.
  - The actor equals the request's maker, or equals the maker's `on_behalf_of_id` or `owner_principal_id` when the maker is of an agent kind. Segregation of duties counts the agent as its human.
- A14's fuller maker-identity-set checks run in A14. This is the floor the database guarantees even for direct SQL.

**Non-delegable writes (the database half of D04's "person must be present").** A trigger `refuse_agent_write` (BEFORE INSERT, UPDATE and DELETE) on every table in `NON_DELEGABLE_TABLES` refuses the write with `agent_write_refused` whenever `app.actor_type` is in `AGENT_KINDS` or `app.token_id` is set, including when the write runs inside `withDomainWrite` (§4), because the actor variables are never replaced there:

- `NON_DELEGABLE_TABLES`: `passkey`, `session`, `fresh_assertion`, `kyc_case`, `identity_claim`, `api_token`, `agent_principal`, `agent_mandate`, `standing_consent`, `erasure_request`, `wallet_key`, `guardian`, `recovery_request`, plus registered tables that declare it (A12's `mandate_signing_key` and `agent_policy`, A11's `login_recovery_request` and `delegation`).
- Carve-outs, listed in the constant with a reason: none that let an agent create, widen or re-key its own access. A12's `api_token.last_used_at` and `last_used_ip` are written by A12's middleware in its own short transaction under `withSystem` as the `svc-agent-access` service account (an SVCA actor), never inside the agent's session.
- `NON_DELEGABLE_COLUMNS`: the withdrawal bank-account columns on `account` (the nominated BSB, account number and PayID `_enc` and `_bidx` columns, under the names A39 and A59 fix at their Specify step) and `account.state` moves to `closed`. A column-level trigger refuses an agent-kind change to any of them, and each change must cite a fresh assertion of the account owner (`ASSERTION_COLUMNS`).
- Identity and liveness verification, and the first passkey and wallet, are covered because `kyc_case`, `identity_claim`, `passkey` and `wallet_key` refuse agent-kind writes. Any further personal declaration counsel names (wholesale-status truthfulness, target-market answers) is added to these lists by a numbered migration after the answer, not guessed here.

**4. Roles, context and row-level security**

**Roles.**

- Four login roles:
  - `realx_migrator`: runs `db:migrate` and does DDL only by `SET ROLE realx_owner`.
  - `realx_app`: the API.
  - `realx_worker`: the worker, plus the `pgboss` schema.
  - `realx_readonly`: exports and reporting.
- Four `NOLOGIN` roles:
  - `realx_owner`: owns every table.
  - `realx_definer`: `BYPASSRLS`. It owns every SECURITY DEFINER function below (the pre-context lookups, `audit_next_link`, `kms_get_wrapped_key`, and every guard, derive and chain trigger function) and holds only the column-limited grants `DEFINER_GRANTS` lists for them.
  - `realx_retention`: no grants until A102.
  - `realx_audit_purger`: no grants until A99 gives it `developer_purge_partition` (A10 item 10).
- No login role has `BYPASSRLS` or `SUPERUSER`, and nothing connects as a superuser. A test reads `pg_roles` and checks exactly these eight roles and their flags.

**SECURITY DEFINER rules (why the guards work under RLS).** `FORCE ROW LEVEL SECURITY` binds `realx_owner`, so a trigger function that runs as the invoking session would see only that session's rows. Every function that must look across rows a session cannot see is therefore SECURITY DEFINER, owned by `realx_definer`:

- the pre-context lookups and `kms_get_wrapped_key`;
- `audit_next_link`;
- `derive_tenant_from_vehicle` (it reads `vehicle.tenant_id`, which an investor session cannot see);
- the ledger balance and same-tenant constraint trigger (it sums every entry of the journal, which an investor-originated posting cannot see);
- the register issue guard (it reads `register_instruction`, `allocation` and `offering`);
- the human-only decider and signer triggers, `approval_binding_check`, `assertion_binding_check`, `refuse_agent_write`, the `agent_principal` kind check and the `role_assignment` SoD trigger (they read `principal`, `approval_request`, `fresh_assertion` and `agent_principal`).

Each such function is declared with `SET search_path = pg_catalog, public` (pinned, so a caller cannot shadow a table or operator), schema-qualifies every object it touches, takes no dynamic SQL, and returns only the columns it needs. `DEFINER_GRANTS` lists, per function, the exact `SELECT` (and for `assertion_binding_check`, the `UPDATE` of the three consumption columns of `fresh_assertion`) that `realx_definer` holds, and nothing else. A verify test reads `pg_proc` and fails when any function in the constant is not owned by `realx_definer`, is not `prosecdef`, lacks the pinned `proconfig` search path, or when `realx_definer` holds a privilege the constant does not list.

`EXECUTE` grants: the three `auth_lookup_*` functions to `realx_app` only; `audit_next_link` to `realx_app` and `realx_worker` (worker jobs and event handlers write audit events through A10's `auditedJob` and `auditedHandler`); `kms_get_wrapped_key` to `realx_app` and `realx_worker`. Trigger functions have no `EXECUTE` grant to any login role.

**Context variables** (set only by the wrappers in `src\client\`, with `select set_config('app.<name>', $1, true)` inside the transaction; `SET` and `SET LOCAL` with string concatenation are never used):

- `app.tenant_id`, `app.principal_id` and `app.principal_kind` (A09's three names). `principal_kind` is the effective subject kind: `investor`, `sponsor_user`, `staff`, `partner`, `service` or `visitor`.
- `app.actor_type`: a principal kind from A03 (HUMAN, PARTNER, AGNT, AGTU or SVCA today; generated).
- `app.on_behalf_of_id`: set for agent sessions, and equal to the human.
- `app.token_id`: the `TOK-` id when the request presented an API token; null otherwise.
- `app.investor_id`: the investor id the session acts for, including an agent acting for an investor.
- `app.principal_role`: the role code or codes.
- `app.platform_access`.
- `app.runtime_access`: set only by `withSystem` for the runtime service accounts (below).
- `app.write_service`: set only by `withDomainWrite` (below).
- `app.view_as_tenant`: narrows a staff session to one tenant. A11 writes the `tenant.viewed_as` audit event.
- `app.vehicle_scope`: the partner's vehicles across every sponsor tenant it is engaged by, filled only by A98 from `partner_engagement` (plus A98's derived trustee vehicles), through `withTenant`.
- `app.sandbox_of`.

Every value is validated as an A02 id or an A03 code before it is set, and an invalid value raises `DB_CONTEXT_INVALID`. A lint rule fails `verify` on any `set_config`, `SET ` or `SET LOCAL` outside `packages\db\src\client\`. A09's `withTx` calls these wrappers and nothing else sets the variables (A09 and A98 are flagged to drop their own `SET LOCAL`).

**Wrappers.**

- `withTenant(ctx, fn)`: sponsor users, investors, partners and agents. It sets the subject's context, including `app.actor_type`, `app.on_behalf_of_id` and `app.token_id` for an agent.
- `withPlatform(ctx, fn)`: RealX staff sessions only. It sets `app.platform_access = true`.
- `withSystem(svcaCtx, fn)`: named `service_account` principals only.
  - It sets `actor_type = SVCA` and `principal_kind = service`.
  - It sets `app.platform_access = true` only when that service account's `platform_access_granted` column is true. The column is set by migration only, for the recon job, the alert sweeps, the bank-rail inbound handler, the webhook inbound handler and the V4 identity agent.
  - It sets `app.runtime_access = true` only when the account's `runtime_access_granted` column is true, set by migration only for A09's `svc-outbox-relay`, `svc-scheduler-tick` and `svc-idempotency-sweep` and for A11's assertion-cleanup job. Runtime access reaches the runtime tables (`outbox`, `processed_event`, `job_schedule_fire`, `job_run`, `system_clock`, `idempotency_key`, and the purge of unconsumed expired `fresh_assertion` rows) across tenants and nothing else.
  - Every `withSystem` transaction carries a `job_run` id or an inbound request id in `app.request_id`, so every row it writes is attributable.
- `withDomainWrite(service, fn)`: runs `fn` inside the current transaction with `app.write_service` set to a named domain service account (for example `svc-register`, `svc-ledger`, `svc-kyc`, `svc-orders`, `svc-approvals`, `svc-custody`), then restores the previous value. It changes nothing else: the actor, on-behalf-of, token, tenant and investor variables stay those of the person or agent whose request it is, so every read predicate, the human-only triggers, `refuse_agent_write` and the audit attribution still see who is really acting. It is the only way to satisfy write class `P` outside a platform session. The service must be listed for the target table in `WRITE_SERVICES`, and the calling module must be listed for that service; a lint rule fails `verify` on a call from any other module. Domain nodes that write `P` tables (A14, A15, A16, A17, A18, A19, A20, A27 to A42, A44, A48, A91, A93 and the registrants) call it through their module context; A09's module context may set it automatically from the module's declared service.
- `withPublic(ctx, fn)`: anonymous visitor routes. It sets `principal_kind = visitor` and nothing else, so only class 6 rows are readable.
- `platform_access` is granted to staff sessions and to the listed SVCA principals, and to nothing else. It is never used for logins.

**Pre-context lookups** (SECURITY DEFINER, owned by `realx_definer`, each returning only the listed columns):

- `auth_lookup_token(lookup)`: `id`, `secret_hash`, `prev_secret_hash`, `prev_secret_valid_until`, `state`, `owner_principal_id`, `agent_principal_id`, `tenant_id`, `environment`, `sandbox_of`, `expires_at`, `row_version`. Used by A12's `agentAuth` before any context exists.
- `auth_lookup_session(session_hash)`: `id`, `principal_id`, `tenant_id`, `expires_at`, `revoked_at`, `last_step_up_at`. Used by A11.
- `auth_lookup_passkey(credential_id)`: `id`, `principal_id`, `public_key`, `sign_count`, `state`. Used by A11 at login.
- `audit_next_link(partition_key)`: see §3.
- `kms_get_wrapped_key(tenant_id, purpose, version)`: returns the wrapped key only when `tenant_id = app.tenant_id` or `app.platform_access` is true. `tenant_key` has RLS enabled and forced, and no direct grant to any login role.

Inbound webhook and bank callbacks run under `withSystem` with their named integration service account, resolve the tenant from the payload, and then do the tenant's work inside `withTenant`.

**Missing context fails closed.** With no variables set, every policy evaluates to false, so a query returns zero rows and accepts zero writes. The client raises `DB_CONTEXT_MISSING` before any query when no wrapper is active.

**RLS: read classes and write classes.** Every table has `ENABLE` and `FORCE ROW LEVEL SECURITY`, except the few `EXEMPT_TABLES`. Its policies are generated from its `RLS_CLASS` entry: `SELECT` policies (`USING`) from the read class, and separate `INSERT` (`WITH CHECK`), `UPDATE` (`USING` and `WITH CHECK`) and `DELETE` policies from the write class. No policy is declared `FOR ALL`, so a read predicate never silently becomes a write predicate. Every class also applies the sandbox rule: a live session sees and writes only `sandbox_of IS NULL`, and a sandbox session only `sandbox_of = app.sandbox_of`.

A common rule applies to every class that tests `tenant_id`: **the platform tenant id never grants anything by equality.** A session whose `app.tenant_id` is the platform tenant (every investor and every partner) sees platform-tenant rows only through `app.platform_access` or through its own class 2, 3 or 8 predicate. This is what stops investor X from reading investor Y's rows.

Read classes:

| Class | Name | Read predicate |
|---|---|---|
| 1 | Tenant-owned | `tenant_id = app.tenant_id` for `sponsor_user` sessions of a sponsor tenant; for `app.platform_access`, all tenants, or `tenant_id = app.view_as_tenant` when it is set; nothing for `investor`, `partner` or `visitor` sessions. A partner reads tenant-owned rows only through class 4 where the table lists it; class 1 gives a partner nothing. |
| 2 | Investor-owned (spans tenants) | `investor_id = app.investor_id` for investor sessions (agents included), regardless of `tenant_id`; otherwise the class 1 predicate on the row's `tenant_id` (plus class 4 where listed). Outside the ledger, the row's `tenant_id` is the sponsor tenant of the row's vehicle, derived and enforced by the SECURITY DEFINER trigger `derive_tenant_from_vehicle` (a mismatch is refused), or the platform tenant when the row has no vehicle (a withdrawal). Ledger rows are always the platform tenant (§2). A sponsor therefore sees its own holders' register rows, orders and distribution lines, including `holder_display_name_enc` for SPN-08, and nothing else of the investor. |
| 3 | Investor master data | `investor_id = app.investor_id` for investor sessions; all rows for `app.platform_access` (support roles see only the `under_review` flag through a column-limited view, see class 7). Sponsors have no direct access: they read the view `investor_for_sponsor` (investor id, investor type, classification state and expiry, KYC state, residency; no `_enc` column). That view is owned by `realx_definer` and filtered to rows where `tenant_consent(investor, app.tenant_id)` is active. Shared KYC documents are released through A30's consent path, not the view. |
| 4 | Partner-scoped | For `partner` sessions only: `vehicle_id = ANY(app.vehicle_scope)`, whatever the row's `tenant_id`, so a trustee-RE or valuer engaged by two sponsors sees exactly its engaged vehicles in both tenants and nothing else. `app.tenant_id` plays no part. A table with no `vehicle_id` is invisible to partners unless its entry names class 4 with a declared predicate. |
| 5 | Platform-only | `app.platform_access` only. |
| 6 | Public reference | `is_public = true` for any session with a context, `visitor` included. Public routes select only the public columns, through the `public_property`, `public_offering` and `public_token` views. |
| 7 | Need-to-know | Class 5 plus `app.principal_role` in the table's role list (MLRO and CCO, plus the delegates A42 names), so outside those roles a row is invisible, not forbidden (tipping-off, N08). Support reads `investor.under_review` only. |
| 8 | Principal-owned | `owner_principal_id` (or `principal_id`) `= coalesce(app.on_behalf_of_id, app.principal_id)`; plus class 1 for a sponsor tenant's SP-OWN over tokens and agent rows of that tenant; plus class 5 (ADM-35). |

Write classes (each also requires the row's read predicate for the session, so no class can write a row it could not read):

| Class | Name | Write predicate |
|---|---|---|
| S | Subject | The read predicate. Used only for a person's own preferences and consents (`tenant_consent`, `notification` read state, `webhook_subscription`, `idempotency_key`). |
| P | Platform or domain service | `app.platform_access` (staff under their read class), or `app.write_service = ANY(WRITE_SERVICES[table])`. An investor, sponsor, partner or agent session writing directly, without `withDomainWrite`, is refused. Every legal, money, KYC, custody, approval and register table is `P`. |
| N | Non-delegable | `P` or the read predicate, and `refuse_agent_write` (§3) refuses every agent-kind actor. The row's assertion columns must bind a fresh assertion of the human (§3). |
| A | Append | `INSERT` under the read predicate (or `P`); no `UPDATE` or `DELETE` policy exists, and the insert-only trigger refuses them anyway. |
| 5 | Platform only | `app.platform_access` only (and migrations). |
| runtime | Runtime | `app.runtime_access` or `app.platform_access`; for `idempotency_key`, also class 8 for the request's own claim. |

**`audit_event` policy** (A10 item 7, one generated policy set with a test per clause; the insert-only trigger refuses every `UPDATE`, `DELETE` and `TRUNCATE`):

- `SELECT`, after the sandbox rule:
  - platform staff (`app.platform_access`): all tenants, or one under `app.view_as_tenant`; a `need_to_know` row only when a role of the session is in `need_to_know_acl`; a `security` row only for SECO and CCO.
  - sponsor users: `tenant_id = app.tenant_id` and `visibility IN ('subject_visible','staff')`.
  - partners: `vehicle_id = ANY(app.vehicle_scope)` and `visibility IN ('subject_visible','staff')`.
  - investors, their agents and every other person: rows where `actor_id`, `on_behalf_of_id` or `subject_principal_id` equals `coalesce(app.on_behalf_of_id, app.principal_id)`, and either `visibility = 'subject_visible'` or the person is the row's actor or on-behalf-of person.
  - never, for anyone: a `need_to_know` row to a session none of whose roles is in `need_to_know_acl`, or a `security` row outside SECO and CCO. The subject is included in this refusal, so an event about an AML alert, case, SMR or freeze whose subject is the investor is invisible to that investor, to the sponsor and to support, and visible to MLRO.
- `INSERT WITH CHECK` (works for investor, agent, sponsor, partner, staff and SVCA sessions alike): the sandbox rule; `actor_id = app.principal_id` and `actor_type = app.actor_type`; `on_behalf_of_id` equals `app.on_behalf_of_id` and `token_id` equals `app.token_id` (both null outside agent sessions); and `tenant_id = app.tenant_id`, or `tenant_id` is the tenant of the row's `vehicle_id` (read through a SECURITY DEFINER helper), or `app.platform_access` or `app.runtime_access` is set. An investor's or an agent's own transaction can therefore write its audit events, and cannot write one in someone else's name.

**5. Error and empty states**

- A missing context variable, an invalid context value, an unknown role, or a connection as a role outside the four login roles raises `DB_CONTEXT_MISSING` or `DB_CONTEXT_INVALID` from the client before any query runs.
- A policy refusal surfaces as zero rows, never as a 403 that leaks existence. A13's "404 not 403" rule starts here.
- `db:migrate` outcomes:
  - On an empty database it applies everything.
  - On a migrated database it applies zero files and exits 0.
  - On a changed file it exits 1, naming the file and both checksums.
  - On a missing extension it exits 1 before any DDL.
- On a fresh database with no seed, every list query returns an empty set under every role, and `db:inventory` still lists every table with zero rows.
- `enc()` without a tenant key creates the key through the kms adapter on first use. `dec()` with the wrong tenant or version returns `PII_KEY_MISMATCH`, never a garbled string.
- A hard `DELETE` outside the purge rule raises `row_delete_refused`. An update outside an insert-only table's allowed columns raises `<table>_immutable`. A soft delete of a guarded row past draft raises `<table>_soft_delete_refused`. An agent or service decider raises `decider_must_be_human`. An agent-kind write to a non-delegable table or column raises `agent_write_refused`. A missing, foreign, reused, expired or wrong-purpose assertion raises `assertion_invalid`. An approval that is not bindable, belongs to another object or action, is already used, or (for `custody_op`) has a different payload hash raises `approval_binding_invalid`. A direct write to a `P` table without `withDomainWrite` or platform access is refused by the policy (zero rows affected, and the client raises `DB_WRITE_REFUSED` when an insert returns no row).

**6. Module-schema registry and `db:check`**

**Registry.** `registerModuleSchema({ node, tables, rlsClass: { read, write }, writeServices, insertOnly, humanOnlyActorColumns, nonDelegable, assertionColumns, migrationsDir })` lets a service module declare additional Drizzle tables that are wholly its own.

- Discovery has exactly one root: `<workspace>\app\services\api\src\modules\<module>\schema.ts`, with numbered migrations in `...\<module>\migrations\`. A `schema.ts` or migrations folder anywhere else fails `db:check`.
- The runner applies core migrations first, then module migrations in `(node id, sequence)` order.
- The registry refuses a table whose name, or its singular or plural variant, already exists in core or in another registrant.
- A module migration may not contain `ALTER`, `DROP`, `CREATE TRIGGER`, `CREATE POLICY` or `GRANT` on any table it did not create.
- A module migration may not contain `DISABLE TRIGGER`, `session_replication_role`, `OWNER TO`, `BYPASSRLS` or `SET ROLE`.
- Core migrations may use these forms only on their own new objects, and never on a guarded table (an insert-only table, a table in `HUMAN_ONLY_ACTOR_COLUMNS`, `ledger_*`, `register_*`, `custody_op` or `tenant_key`).

**`db:check` fails when:**

- generating from the schema would produce a new migration
- a committed migration's checksum differs
- a registered schema is not covered by a migration
- a table appears in two owners, or under two names
- any of the forbidden forms above appears
- a table lacks a `RLS_CLASS` entry with both a read class and a write class, or has a `FOR ALL` policy
- a table of write class `P` has no `WRITE_SERVICES` entry, or a `withDomainWrite` call names a service its module is not listed for
- a SECURITY DEFINER function is missing from `DEFINER_GRANTS`, is not owned by `realx_definer`, or has no pinned `search_path`
- a guard trigger is missing or not enabled: a verify test reads `pg_trigger.tgenabled = 'O'` for every guard trigger

**Coverage.** `db:check`, `db:inventory` and every convention test run over the *whole* live schema. A registered table that lacks `tenant_id`, RLS, a class or an enum state column fails verify exactly as a core table would.

**7. Generated test suites (in `packages\db\test\`)**

- **RLS classes.** The suite is generated from `RLS_CLASS`. It seeds:
  - sponsor tenants A (Harbour & Co Property) and B (Northline Capital), both from the design's mock data
  - the platform tenant
  - two investors X and Y in the platform tenant, each also holding, ordering and receiving distributions in one vehicle of A and one of B
  - a partner engaged by one vehicle of A and one vehicle of B (a multi-sponsor valuer or trustee-RE)
  - an agent of X (kind `AGTU`, through a token) and a RealX staff agent (kind `AGNT`) of a staff maker
  - a sandbox partition for X
  - one row per table per relevant owner

  It then asserts, per table, that:
  - A sees only A and B sees only B, and neither can insert or update the other's rows
  - staff sees both, and view-as-A sees only A
  - X sees zero rows of Y in every class 2, 3 and 8 table, including platform-tenant rows such as ledger accounts, wallets and tax profiles, and an agent of X sees exactly what X sees
  - A sees X's class-2 rows in A's vehicle and none in B's vehicle, and no ledger row at all (ledger rows are platform-tenant)
  - A sees zero investor master rows without consent, and only the `investor_for_sponsor` columns with consent
  - the partner sees exactly the two engaged vehicles, one in A and one in B, and nothing else in either tenant; class 1 gives it nothing
  - the sandbox session sees only its partition
  - a visitor sees only class 6 public rows
  - a context-less session sees nothing and writes nothing
  - write classes: for every `P` table, an insert and an update of X's own row from X's session and from X's agent session are refused without `withDomainWrite`, and succeed with the listed service; for every table in `NON_DELEGABLE_TABLES` and every `NON_DELEGABLE_COLUMNS` column, a write by X's `AGTU` agent and by a staff `AGNT` is refused, including inside `withDomainWrite`, and the same write by the human with a valid assertion succeeds
- **Audit visibility.** An `audit_event` with `visibility = need_to_know`, `need_to_know_acl = {MLRO}` and `subject_principal_id = X` (an AML alert about X) is invisible to X, to X's agent, to sponsor A and to a SUPP staff session, and visible to MLRO; a `security` row is visible to SECO and CCO only; a `subject_visible` row about X is visible to X and X's agent and not to Y. An insert of X's own event succeeds inside X's session and inside X's agent session; an insert naming another actor, another on-behalf-of person or another token is refused.
- **Assertions.** For every `ASSERTION_COLUMNS` entry: a random id, another principal's assertion, an assertion already consumed by another row, an assertion from another transaction, an expired one and a wrong-purpose one are all refused; a fresh one of the right owner and purpose is accepted and marked consumed by this row.
- **Approval binding.** For every `PRIVILEGED_TABLES` member and `custody_op`: a rejected, a pending (`draft`, `submitted`, `prechecked`), an expired, a foreign-object, a wrong-action and an already-used approval are refused; for `custody_op`, a payload-hash mismatch is refused; the correct approval is accepted.
- **Definer functions under RLS.** Inside an investor session and inside that investor's agent session: an `order` insert through `withDomainWrite` whose tenant is derived from a sponsor vehicle the investor cannot read succeeds; a balanced journal posted through `withDomainWrite('svc-ledger')` commits and an unbalanced one fails; an audit write through `audit_next_link` links to the true head written by another tenant. `realx_worker` can call `audit_next_link`. Every function in `DEFINER_GRANTS` is owned by `realx_definer` with the pinned search path.
- **Agent kinds.** Every probe in these suites that uses an `AGNT` principal is generated a second time with an `AGTU` principal (and the set comes from `AGENT_KINDS`, so a new kind in A03 is probed automatically).
- **Pre-context functions.** Each auth lookup function returns only its columns, a direct `SELECT` on `api_token`, `session`, `passkey` or `tenant_key` with no context returns zero rows, and tenant A cannot obtain B's wrapped key through `kms_get_wrapped_key`.
- **Audit chain.** 1,000 alternating tenant-A and tenant-B `audit_event` writes through `audit_next_link` leave one linear `live` chain (every `prev_event_sha256` equals the previous `event_sha256`, with no gap and no fork), and a second genesis row in the same partition is refused by `UNIQUE (prev_event_sha256)`.
- **Other suites:** type scan, enum drift, FK index, insert-only (allowed and refused columns, including `register_lot`, `register_instruction_line`, `ledger_snapshot`, `ledger_journal` apart from a single `reversed_by` set, and `fresh_assertion` apart from its consumption columns), soft delete of guarded rows past draft, `tenant_guardrail` with `kyc_required = false`, hard delete (the purge allowed for an expired row as `realx_worker`, refused for an unexpired row, refused on any other table and under any other role), ledger balance, bitemporal, register issue guard (a second offering open on a vehicle that already has holders leaves transfers and adjustments working), privileged-state CHECKs, human-only deciders, token widening, trigger-enabled check, context leak, and ownership walk.

**Not in this node (owned elsewhere, but its tables are here):** seed data and the simulator (A06, which owns `packages\db\seed\`); audit hashing, the write path and the daily chain verification (A10); passkeys, sessions and the RBAC middleware (A11); token issuance, scopes and the G22 evaluator (A12); alert emission (A13); approvals behaviour (A14); custody behaviour (A15); posting (A16); register behaviour (A18); gate evaluation logic (A19); recon rules (A20); backups, the retention role's grants and erasure (A102). Those nodes read `@realx/db` and never edit its files. A table or column that a later node needs and does not find is a spec defect on this node. The orchestrator reopens A04 as a follow-up loop (A04.1, A04.2, ...) that adds a numbered core migration, or the node registers an additive table through the registry when the table is wholly its own. To keep that rare, the builder reads every node spec named in the ownership map before writing the schema, and records the consuming nodes per table in the inventory.

## Owned paths (other nodes do not edit these)

- <workspace>\app\packages\db\ (root files only: package.json, drizzle.config.ts, tsconfig.json, vitest.config.ts, README.md)
- <workspace>\app\packages\db\src\ (root files: index.ts and the exported constant lists)
- <workspace>\app\packages\db\src\schema\
- <workspace>\app\packages\db\src\client\
- <workspace>\app\packages\db\src\registry\
- <workspace>\app\packages\db\scripts\
- <workspace>\app\packages\db\test\
- <workspace>\app\packages\db\migrations\
- <workspace>\app\docs\schema\

## Design references

This node renders nothing, so no visual tokens apply. The prototype is used as evidence of which columns the screens need. All references are to `<workspace>\design\RealX App.dc.html` (read only, never edited) unless marked design-ext.

**The `data-dc-script` mock data model (design analysis §7), mapped to tables:**

| Design data | Tables |
|---|---|
| `SP` (sponsor facts, "Own AFSL" vs "Trustee: hired RE") | `tenant`, `entity`, `licence` |
| `PR` (price, val, y, g, lvr, status, raised, closes, keyFacts, details) | `property`, `vehicle`, `offering`, `valuation` |
| `HOLD` (qty, avg) | `register_entry`, `register_lot`, `holding` |
| Orders `RX-ORD-55120` and `RX-ORD-51877` (status In escrow, Allotted, Refunded) | `order`, `payment`, `ledger_account` |
| `APS` (APR ids, type, maker, risk, checks with gate refs) | `approval_request`, `approval_decision`, `gate_evaluation` |
| `OPS` (OP ids, appr, sigs in 3 segments, tx, status) | `custody_op`, `custody_signature` |
| Recon rows (register / token supply / client money / status "Break · 2 units") | `recon_run`, `recon_break` |
| Treasury accounts (••2201, purpose, "Fees (not client money)" Separate) | `ledger_account` |
| Audit rows (Time / Actor / Action / Object / Approval / Result; actors "Identity provider", "Recon job", "Custody quorum") | `audit_event`, including SVCA actors, with labels resolved at read time |
| KYC case INV-00912 | `kyc_case` |
| AML-118 | `aml_alert`, `aml_case` |
| Statements (AMMA, payment advice) | `amma_statement`, `document` |
| Alerts and tasks (High red / Medium amber / Review grey, target screen) | `alert` |

**Screens:**

- `adAudit` (ADM-31): columns and the "Append-only" subtitle.
- `adRecon` (ADM-17): columns and the escrow annotation (bug 14: escrow is not register).
- `adToken` (ADM-14): the waiting-for-signatures list.
- `adApprovals` (ADM-30): checks and "Approval ID is bound to the signing request".
- `adTreasury` (ADM-18): the account table, `Bank A ••2201`.
- `spRegister` (SPN-08): holder columns (holder, type, units, % of trust, wallet, status Active / Frozen).
- `wallet` (POR-05): the ledger table (date, type, property, detail, amount, status) and "To CBA ••4471".
- `onb` steps (ONB-01..10): the PII field list, TFN `••• ••• 482`, account `••••4471`, BSB `062-000`, and the PayID and email shown to the owner.

**design-ext (undesigned screens):**

- [[A50 - Design extension A- investor and sponsor screens not in the prototype]]: ACC-05 and SPN-16 (token name, prefix, scopes, limits and expiry in plain English, last used, activity log, revoke, sandbox section).
- [[A51 - Design extension B- operator console screens not in the prototype]]: ADM-35 (every token and mandate platform-wide, the G22 evaluation log, kill switches), ADM-37 (users, roles and access reviews) and ADM-39 (entities register).

These fix the columns of `api_token`, `agent_mandate`, `api_token_activity`, `gate_evaluation`, `role_assignment`, `access_review` and `entity`.

**Constants the design fixes** (design analysis §7 tail): 10,000 units per trust, holder cap 5%, price band 85-115%, withdrawal second approval at A$50,000, certificate validity 2 years, and the as-of date Wednesday 23 Sep 2026. They are parameters in rule packs and policies (A19, A14) and seed values (A06), never column defaults here (architecture brief risk 17).

## Requirements covered

- **Product-map screens:** none owned. Every one of the 91 canonical screens reads and writes through this schema. "Done" means every column a listed screen shows has a home table in the inventory. The builder records the screen ids per table, and the node QA spot-checks ADM-17, ADM-30, ADM-31, SPN-08, POR-05, ACC-05 and ADM-35.
- **Journeys:** none owned. "Done" means J04, J06, J09, J15 and J20 can be traced table by table in the inventory without a missing entity:
  - order → payment → ledger → allocation → register entry → custody op → recon run
  - distribution run → line → tax component → ledger journal
  - recovery request → transfer request → register
  - withdrawal → approval request → decision
  - approval → custody op → custody signature → audit event
- **Gates:** none evaluated here (A19 evaluates and records in `gate_evaluation`). "Done" means each named column and trigger below exists with the stated constraint and a test asserts it. The data support that must exist:
  - G13: `approval_id` on every privileged table; `custody_op.approval_id` unique and not null; `approval_decision` and `custody_signature` insert-only; the human-only decider and signer triggers, with segregation of duties counting an agent as its human.
  - G22: `api_token` with a non-null creation assertion that is bound to a fresh, unused assertion of the owner, and no in-place widening; `agent_principal` of an agent kind owned by a HUMAN or PARTNER; versioned `agent_mandate` whose assertion covers its `mandate_sha256`; `api_token_activity`; the registry for A12's `agent_confirmation`, `agent_counter` and `agent_switch`; `on_behalf_of_id` and `token_id` on `audit_event` and `gate_evaluation`; the never-checker triggers; `refuse_agent_write` on every non-delegable table and column.
  - G10: `halt`, `recon_break`.
  - G08: `register_lot.lock_in_until`.
  - G07: `threshold_monitor`.
  - G16: `tax_profile`, `tax_component`.
  - G19: `property_attestation`.
  - G20: `valuation.valid_until`.
  - G21: `licence`, `feature_flag.licence_bound_to` and `requires_licence`.
- **Modules:** storage for all 27 (M01-M27). "Done" means the two-tier tenancy (platform tenant over sponsor tenants) is expressed by the eight RLS classes, not by application filters. Three are named because no other node owns their data:
  - M25 access, approvals and audit: `audit_event` insert-only, `approval_*`, `role_*`, `access_review`, `break_glass_session`.
  - M26 integrations and API: `api_token`, `agent_*`, `integration`, `webhook_*`, `job_run`.
  - M27 settings and tenancy: `tenant`, `tenant_guardrail`, `feature_flag`, `fee_schedule`.
- **State machines:** SM-00..SM-13 as enum columns on `approval_request`, `kyc_case`, `investor_classification`, `property`, `offering`, `order`, `token`, `holding`, `distribution_run`, `resale_order` / `trade` / `instrument`, `proposal`, `aml_case`, `complaint`, `recovery_request` / `transfer_request`. "Done" means the enum values equal A03's state lists exactly (drift test) and no state column is free text.
- **Non-functional requirements from the catalogue:**
  - N01: `retention_policy`, `legal_hold`, seven-year columns.
  - N02: insert-only chained audit with on-behalf-of, one linear chain per partition, and visibility classes so an AML-subject event is never shown to its subject.
  - N04 and N05: balanced journals; one approval per custody op.
  - N07: PII encrypted and masked by codec.
  - N08: need-to-know class 7.
  - N13: `idempotency_key`, `outbox`, `processed_event`.
  - N15: no vendor call; kms sandbox only.
  - N19: multi-tenancy by RLS classes; per-sponsor tokens tenant-scoped.
  - N20: `tenant_key` rotation columns, with the key read only through `kms_get_wrapped_key`.

## Gauntlet loop for this node

**Specify.** The frozen inputs are:

- this note
- A02's id, money and unit types
- A03's exported state, gate, code and matrix JSON, including the local lifecycles
- A08's `kms` interface and config schema
- the column lists of the shaping nodes named in the ownership map (A09, A10, A11, A12, A13, A14, A15, A16, A18, A19, A37, A98, A101), as folded into §2 and §3 on 24 Sep 2026; where a shaping spec still says otherwise than the "final decisions" in §2, the decision in §2 wins and the conflict is recorded in `A04.qa.md` for the orchestrator

The builder confirms the Postgres 17 image, Drizzle ORM and drizzle-kit versions pinned by A01 (this line or its latest patch) and records them in the evidence manifest. The node QA records `specVersion` and the hashes of the inputs.

**Oracle (node QA, Opus 5.5 high, before the build).** Hidden Vitest and fast-check files go to `<workspace>\gauntlet\hidden\A04\`. `oracleVersion` and `hiddenCasesHash` are recorded in `app\evidence\A04.json`. The hidden files cover:

- (a) its own RLS read-class and write-class suite with its own row shapes: two investors in the platform tenant and in each sponsor vehicle, a partner engaged by one vehicle in each of two sponsor tenants, a sandbox session, a visitor, an `AGTU` agent of an investor and an `AGNT` staff agent; direct writes by the investor and its agent to every `P` and `N` table; the AML-subject audit event seen by the investor, the sponsor, support and MLRO
- (b) `UPDATE` of every non-allowed column, `DELETE` and `TRUNCATE` on every insert-only table (including `register_lot`, `register_instruction_line`, `ledger_snapshot`, `ledger_journal` and `fresh_assertion`) under all four login roles and as `realx_owner`, and a soft delete of an approved `withdrawal` and a posted `payment`
- (c) a chain-fork insert, a second genesis, and 1,000 interleaved two-tenant writes that must leave one linear chain
- (d) an AGNT event and an AGTU event without a human
- (e) an AGNT or AGTU agent, a token-bearing row or session, an SVCA outside `AUTO`, the maker and the maker's owner as decider or signer, through direct SQL
- (f) a register as-of oracle over 1,000 randomised corrections, computed independently in Python
- (g) 100 unbalanced and 100 cross-tenant journals, and 100 balanced ones
- (h) a privileged row in a post-approval state with a null approval id; two custody ops for one approval; two withdrawals citing one approval; a withdrawal citing a rejected, a pending, an expired or a foreign-object approval; a custody op bound to a pending approval or with a payload hash that differs from its approval's
- (i) an in-place token widening without an assertion, a mandate edit in place, a token with a null creation assertion, and tokens and mandates citing a random, a foreign-principal, a reused, an expired and a wrong-purpose assertion
- (j) the pre-context functions returning extra columns, or rows without their key
- (k) a `SET` (not transaction-local) leak probe over a pooled connection with 1,000 interleaved requests
- (l) a plaintext scan of a `pg_dump` for seeded PII and a token secret, and a cross-tenant decrypt

**Build (Grok 4.7 xhigh, split by the planner).** Suggested shape: five parallel sub-agents, one per domain group, then one integration agent.

- The five groups are: A tenancy and assets; B people and wallets; C token, register, rules, market and recon; D orders and money; E governance, administration, agent access, audit, alerts and runtime.
- Each group agent writes its schema files against `_conventions.ts`, the ownership map and a shared naming sheet (singular snake_case table names; `_enc`, `_bidx`, `_cents`, `_units` and `_bp` suffixes).
- The integration agent then:
  - generates `enums.ts` from A03 and wires the shared column sets
  - writes the hand-written migrations: extensions, the eight roles, the RLS class policies, the SECURITY DEFINER functions, triggers and constraint triggers
  - builds `src\client\` and `src\registry\`
  - runs `db:generate`, `db:reset`, `db:check` and `db:inventory`
  - writes the generated suites in §7
- Evidence goes to `<workspace>\app\evidence\A04.json`: the migration list with checksums, the table inventory with owner, shaping and consuming nodes, test counts, timings, pinned versions, `oracleVersion` and `hiddenCasesHash`.

**Challenge (node QA, Opus 5.5 high).** On a fresh Docker Postgres:

- `npm run verify`
- `db:reset` twice, then `db:migrate` again as a no-op
- `db:check` clean
- its hidden suites from `gauntlet\hidden\A04\`
- a walk of every node spec for table and column names against the generated `docs\schema\ownership.json`: every table name must resolve to exactly one entry, singular, with one owner (core or the named registrant), and every column a spec names on a core table must exist. A spec that still contradicts a "final decision" in §2 (for example a registered table claimed by two nodes, a `status` column on `api_token`, `audit_chain_head`, or a sponsor-tenant ledger row) is recorded as a finding against that spec, not as a defect of this node
- a read of every migration for anything vendor-specific, any production connection string, any threshold or date that belongs in a rule pack, and any forbidden form (§6)

Findings carry a severity per [[App Gauntlet]] §5 and are written to `app/evidence/A04.qa.md`.

**Gate, fix loop, escalation.** A pass unlocks A06, A09, A99, A102 and A106, and the hidden suites are then published into `app/`. At most three fix loops, each re-running the whole suite. Escalate (split into A04a and A04b, respec, or ask the user) when:

- the same finding fails a third time
- a fix needs A02, A03 or A08 to change (a spec defect back-edge)
- a shaping node's column list conflicts with an invariant here
- Drizzle or drizzle-kit cannot express a constraint on Postgres 17, and the hand-written SQL fallback is also refused by `db:check`

## Node-specific challenges (node QA)

- **Tenant and investor leak (security).**
  - A `realx_app` session for Harbour & Co inserts a `valuation` row with `tenant_id` set to Northline, reads Northline's `register_entry` rows through a join on `vehicle`, or reads them through a table left out of `RLS_CLASS`.
  - Investor X reads investor Y's `investor`, `kyc_case`, `tax_profile`, `wallet` or `ledger_account` row. All of these carry the platform tenant id.
  - A sponsor reads an investor master row without consent.
  - A partner reads a vehicle outside its scope, or reads any row of a tenant through class 1; or a partner engaged by one vehicle in each of two tenants fails to see both.
  - Investor X, or X's agent, inserts or updates its own `register_entry`, `holding`, `ledger_entry`, `distribution_line`, `kyc_case`, `wallet_key` or `guardian` row directly, without `withDomainWrite`.

  Any row visible or accepted fails the node.
- **Audit tampering (compliance).**
  - As `realx_worker`, `realx_migrator` and `realx_owner`: `UPDATE audit_event SET action = ...`, `DELETE`, `TRUNCATE`.
  - An insert whose `prev_event_sha256` equals an existing predecessor in the same partition, or a second genesis.
  - An AGNT or AGTU event without `on_behalf_of_id`.
  - An AML-subject `need_to_know` event about investor X read by X, X's agent, sponsor A or support.
  - A module migration containing `ALTER TABLE audit_event DISABLE TRIGGER`.

  All must be refused by the database or by `db:check`, not by application code.
- **Never checker (compliance, G13 and G22).**
  - An `approval_decision`, a `custody_signature` or a break-glass activation whose actor is an AGNT or AGTU principal, whose row carries a token id, whose session context is an agent kind or carries a token, whose actor is an SVCA on a non-`AUTO` request, or whose actor is the maker or the maker's human when the maker is an agent.

  Each must be refused through direct SQL.
- **Money and units (money).**
  - A migration that introduces a `numeric` column.
  - A `distribution_line` with fractional units.
  - A ledger journal whose entries do not balance at commit, or whose entries span two tenants.
  - A `register_entry` with negative units.

  The type scan, the CHECKs and the constraint trigger must catch each.
- **Register history (money).** Each of these is a failure:
  - two open `register_entry` rows for the same holder and vehicle with overlapping `effective`
  - an in-place `UPDATE` of units
  - an as-of query for "what was known on 1 Sep" that returns a correction recorded on 15 Sep
  - an issue entry for an allocation of an offering still `live` (escrow written as register)

  Also a failure: a legitimate transfer refused because a second offering on the same vehicle is open.
- **Secrets and PII (security).**
  - A plaintext token secret in any column, migration, fixture or log.
  - An `_enc` column decryptable with another tenant's data key.
  - Tenant A reading B's `tenant_key` row.
  - An `_enc` column with an index.
  - An email stored outside `email_enc` and `email_bidx`.
  - A label column on `audit_event`.
  - A `mask()` output that differs from the prototype strings listed in §3.
- **Context and pools (security).**
  - A request that never called a wrapper still sees rows (fail open).
  - A context value concatenated into SQL.
  - A `set_config` outside `src\client\`.
  - A non-transaction-local setting in one request observed by the next request on the same pooled connection.
  - A sandbox session reads a live row through any table missing `sandbox_of`.
  - A support-role session can see that an `aml_case` row exists.
  - A login lookup performed through `withPlatform`.
- **Agent access (security, G22).**
  - A token inserted with a null `created_with_assertion_id`.
  - An `UPDATE` that adds a scope, extends expiry or clears the IP allow-list without a new assertion.
  - An `agent_principal` owned by another agent (AGNT or AGTU) or by an SVCA, or whose own principal is not an agent kind.
  - A token or mandate citing a random, another person's, a reused or a wrong-purpose assertion id.
  - An agent session writing its human's `passkey`, `session`, `api_token`, `agent_mandate`, `guardian`, `wallet_key`, `recovery_request` or withdrawal bank-account columns, directly or through `withDomainWrite`.
  - A mandate edited in place.
- **Binding (compliance, G13 and G21).**
  - Two `custody_op` rows for one `approval_id`, or one bound to a pending approval or to an approval whose payload hash differs.
  - Two withdrawals citing one approval, or a withdrawal citing a rejected or another object's approval.
  - A `withdrawal` in state `approved` with `approval_id` null.
  - A `feature_flag` with `requires_licence = true` switched on with no `licence_bound_to`. `agents.traffic` and the phase flags, which do not require a licence, must switch freely.
  - A `tenant_guardrail` row with `kyc_required = false`.
- **Insert-only and purge (design fidelity of the contract).**
  - `bank_transaction.match_state` must update and its raw fields must not.
  - A certified `register_version` must refuse any change except supersession.
  - An expired `idempotency_key` must be purged by `realx_worker`, and an unexpired one refused.
  - A `DELETE` on any other table must be refused under every role.
- **Migrations and registry.** Each of these must fail:
  - a schema edit without a migration passes `db:check`
  - a migration file edited after commit still runs
  - `db:migrate` twice applies a file twice
  - a module registers `register_entries` or `orders` next to the core table
  - a module migration alters a core table
  - a registered table without `tenant_id`, RLS or a class passes verify
  - a state enum differing from A03's export passes the drift test

## Pass criteria

- **Build and migrate.** `npm run verify` is green repo-wide. `db:reset` succeeds twice from an empty Postgres 17 container, and a third `db:migrate` applies zero files. `db:check` reports no drift, and every committed migration checksum matches. A fresh migrate completes in under 30 seconds on the build machine.
- **Ownership.** Every table named in any node spec resolves to exactly one row of `TABLE_OWNERS`, with a singular name and one owner. `db:check` refuses a duplicate or a plural variant, and refuses a module migration that alters a core table. The shaping-node column lists are present in the core tables: the node QA's walk of `ownership.json` finds no missing column for A09 (the full `idempotency_key` list), A10 (items 1 to 8 and 10), A11 (its requested `session`, `passkey`, `account`, `principal` and `role_assignment` columns and `fresh_assertion`), A12 (its §7 A04.1 list), A13, A14, A15, A16 (its A04.1 ledger list), A18, A19 and A37. A conflict between a shaping spec and a final decision in §2 is reported against that spec.
- **Inventory.** `docs\schema\inventory.md` lists every entity of the architecture brief §3 (domains A-E) plus every agent-access, audit, alert, runtime and retention table named in §2. Each is listed with its owner, shaping and consuming nodes, RLS class and insert-only columns.
- **Conventions.**
  - Every table outside `EXEMPT_TABLES` has `tenant_id`, `sandbox_of`, a read class and a write class in `RLS_CLASS`, RLS enabled and forced, a `SELECT` policy and separate write policies, and no `FOR ALL` policy.
  - Every SECURITY DEFINER function is owned by `realx_definer`, has the pinned `search_path` and holds exactly its `DEFINER_GRANTS`.
  - Mutable tables have the audit columns and `deleted_at`. Insert-only tables have neither `updated_*` nor `deleted_at`.
  - There are zero `numeric`, `real`, `double precision` or `money` columns.
  - Every state column is a `pgEnum` equal to A03's export. The drift test passes, and fails in a scratch run when a state is added to A03.
  - Every `_enc` column is `bytea`, unindexed and listed in `PII_PATHS`.
  - Every FK has an index.
  - Every table in `PRIVILEGED_TABLES` has its approval CHECK.
  - Every guard trigger has `tgenabled = 'O'`.
- **RLS.** The generated class suite and the node QA's hidden suite both pass across every table:
  - zero cross-tenant rows visible, and zero cross-tenant writes accepted
  - zero rows of investor Y visible to investor X in every class 2, 3 and 8 table
  - zero investor master rows for a sponsor without consent
  - exactly the engaged vehicles, across both tenants, for a partner, and nothing through class 1
  - zero direct writes accepted from investor or agent sessions on `P` tables, and zero agent-kind writes on `N` tables and `NON_DELEGABLE_COLUMNS`
  - the AML-subject audit event visible to MLRO only among the probed sessions
  - staff sees both tenants, and view-as sees one
  - sandbox sees its partition only, a visitor sees class 6 only, and a context-less session sees nothing

  Also: exactly four login roles, none with `BYPASSRLS` or `SUPERUSER`; the four `NOLOGIN` roles cannot log in; the 1,000-request interleaved pool test never leaks context; each pre-context function returns only its listed columns.
- **Immutability and purge.**
  - 100% of `UPDATE` (outside allowed columns), `DELETE` and `TRUNCATE` attempts on every `INSERT_ONLY_TABLES` member, under all four login roles and `realx_owner`, raise.
  - The allowed-column updates succeed.
  - The chain-fork insert, the second genesis and the agent-without-human insert are refused.
  - 1,000 alternating two-tenant audit writes leave one linear chain.
  - `custody_op` refuses a second row for one approval, and `register_entry` refuses an in-place update.
  - The runtime purge deletes only expired rows (and for `fresh_assertion` only unconsumed ones), and only as `realx_worker`.
  - A soft delete of any `SOFT_DELETE_GUARDED` row past its draft states is refused, and `tenant_guardrail` refuses `kyc_required = false`.
- **Human-only.** Every decider and signer probe in the Challenge list, for both AGNT and AGTU, is refused through direct SQL, and a legitimate HUMAN, PARTNER and `AUTO` SVCA decision is accepted.
- **Binding.** Every assertion probe (random, foreign, reused, other-transaction, expired, wrong purpose) and every approval-binding probe (rejected, pending, expired, foreign object, wrong action, reused, payload-hash mismatch) is refused, and the legitimate case is accepted.
- **Money and register.** The ledger constraint trigger rejects 100 of 100 random unbalanced or cross-tenant journals and accepts 100 of 100 balanced ones. The bitemporal exclusion constraint rejects every overlapping pair in 1,000 fast-check sequences, and `asOf` agrees with the node QA's independent oracle in all 1,000. The register issue guard refuses escrow-as-register and leaves transfers working while a second offering is open.
- **PII.** A `pg_dump` of a seeded-and-encrypted database contains no seeded name, TFN, email, BSB, PayID or token secret in clear. A tenant A data key cannot decrypt a tenant B row. `mask()` snapshot strings equal the prototype strings listed in §3.
- **Evidence.** The manifest `<workspace>\app\evidence\A04.json` exists, matches disk, and names the pinned Postgres, Drizzle and drizzle-kit versions, `oracleVersion` and `hiddenCasesHash`. The node QA's `A04.qa.md` records no open Critical or High finding.

## Human gates touched

- None block this node. Two are recorded as open:
  - HG-KEYS: the envelope-encryption master key and the `tenant_key` wrapping move from A08's sandbox kms to a cloud KMS or HSM under the key ceremony. The local dev master key carries no value and is regenerated on reset.
  - HG-PENTEST: the RLS classes, the roles, the SECURITY DEFINER functions and the triggers are in scope for the external test before go-live.
- The HG-LEGAL questions about retention periods and erasure under the Privacy Act are carried by A102, not here. This node only gives them columns.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
