---
title: "A11 - Authentication, sessions, RBAC and segregation of duties"
created: 2026-09-23
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am1]
node: A11
milestone: AM1
lane: "Compliance core"
oracle: yes
human_gate: none
---

# A11 - Authentication, sessions, RBAC and segregation of duties

**The identity spine of RealX: who a caller is (human, partner, agent or service account), how they proved it (passkey, hardware-attested passkey, fresh assertion), which tenant they are acting in, what A03's permission matrix lets them do, and which of A03's segregation-of-duties rules stop them. It is enforced once in the API, so every UI node and every agent surface inherits the same answer.**

**Build status:** not started. **Milestone:** AM1 (Money, register, approvals and agent-access core). **Lane:** Compliance core. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 12-16 agent-hours (build, challenge and verify loops). About ten subsystems sit here; if the orchestrator prefers smaller nodes, the clean split is A11a (principals, passkeys, email verification, sessions, tenant context, recovery) and A11b (RBAC consumption, SoD calls, delegation, break-glass, fresh assertion), with A11b depending on A11a.

**Depends on:** [[A09 - API and worker skeleton with OpenAPI-generated SDK]], [[A10 - Append-only hash-chained audit log]]. A03, A04 and A08 arrive through A09. A13 and A22 are not dependencies: both depend on A11 in the graph (A13 needs real sessions, A22 needs A11's fresh passkey assertions), so an edge the other way would be a cycle. A11 instead exposes two ports (in the list of what this node uses, below): the **break-glass clock port**, which A13 registers its `openClock`, `businessDaysAfter` and `emit` into after A13 passes, and the **liveness port**, which A22 registers its identity adapter into after A22 passes. A11's own tests and its hidden oracle use recorded fakes of both ports that follow A13's and A22's frozen contracts; the real wiring is proven at A13's, A22's and A23's gates (graph note, 24 Sep 2026).
**Unlocks:** [[A12 - API tokens, scopes, agent principals and mandates for every user type (G22)]], [[A14 - Approvals engine and maker-checker (SM-00)]], [[A30 - Investor onboarding, KYC cases and KYB (SM-01)]], [[A33 - Sponsor tenancy, entities, licences of sponsors and admission pipeline]], [[A52 - Web app shells, routing and passkey sign-in (public, investor, sponsor)]], [[A53 - Operator console shell, hardware-key sign-in and approval action pattern]], [[A67 - Console- users, roles, access reviews and break-glass (ADM-37)]], [[A98 - Partner principals- trustee-RE, valuer, property manager, auditor, regulator observer and licensee users]], [[A103 - Runbook drills R1-R9 executed on the stack]], [[A106 - IAM admin API for ADM-37]]

**Oracle rule:** The node QA writes hidden oracle tests from this spec to `<workspace>\gauntlet\hidden\A11\` before the build starts ([[App Gauntlet]] §2). The builder never reads that folder. At minimum, the hidden set covers:
- one generated test per (roleId, actionId) pair in A03's exported matrix;
- no self-grant by any route;
- SUPER never holding a money or signer role;
- no AGNT or SVCA principal ever holding an approve, ratify or sign cell, and no AGNT holding a non-delegable or counsel-pending cell;
- investor and sponsor agents keeping their delegable money and invest cells (D04 parity);
- delegation in both directions;
- fresh-assertion replay, staleness and payload drift;
- session device binding and CSRF;
- the dormancy boundary;
- tenant isolation under "view as tenant", with one probe per RLS variable;
- A03's ten SoD rules, called through `checkSod`, at grant time and at request time.

This node is API and data only. It draws no screens. Other nodes build the screens on this API:
- A52 (public, investor and sponsor) and A53 (console) draw and wire the sign-in, passkey and recovery cards.
- [[A67 - Console- users, roles, access reviews and break-glass (ADM-37)]] draws the users, roles, access-review and break-glass screens.
- [[A98 - Partner principals- trustee-RE, valuer, property manager, auditor, regulator observer and licensee users]] builds the partner rows and portal.
- [[A12 - API tokens, scopes, agent principals and mandates for every user type (G22)]] builds API tokens, scopes and mandates.

A11 gives all of them one principal model, one session model, one permission decision (over A03's data), one delegation model and one "fresh assertion" primitive. This plan is not legal advice. Every sentence about what counsel or a regulator requires is a counsel question.

## Build brief

**0. What A11 consumes and what it owns (no second copy of A03's rules)**

- **From A03, through `@realx/domain`, read-only.** `src/permissions/roles.ts` (51 roles = the 52 ids in `requirements-lists.json` minus AGT-U; VIS and INV-W, INV-S, INV-E, INV-J, INV-R, INV-F, INV-M are roles), `principals.ts`, `actions.ts` (the ten action kinds read, draft, submit, approve, ratify, sign, execute, export, configure and unmask, plus `delegable`, `family` and `sensitive` on each action), `matrix.ts` (cells `(roleId, actionId) -> {allow, constraint?}`), `delegable.ts`, `scopes.ts` and `sod.ts` with `checkSod`. A11 never edits these files. If A11 needs a missing cell, action or SoD row, that is an A03 follow-up loop. **A03.1 (SoD pack rows)** is requested at Specify for the research-pack rows A03 does not yet carry:
  - #5 KYCA on their own or a relative's case;
  - #8 VALO with RISK;
  - #16 PART commissions;
  - #17-#19 soft dealing and related-party rules;
  - #20 LEGL forced-transfer liaison;
  - "the platform CCO, not a sponsor owner, approves any tenant offering", if A03's rule 9 does not already express it.
  A11 checks nothing that is not in A03's data.
- **From A09:**
  - `defineRoute` with `meta.requiredScope`, emitted as `x-realx-scope`. A11's middleware reads it as the A03 actionId, since A03 sets scopeId = actionId.
  - The request context and principal shape.
  - `withTx`, `registerModule`, `registerErrors` and `registerJob`.
  - The hook slots `principal`, `kill-switch` and `rate-limit`.
  - The injectable clock. Every "now" in this node is `ctx.clock.now()`.
- **From A04:** the core tables `principal`, `account`, `passkey`, `session`, `staff_user`, `partner_user`, `role`, `role_assignment`, `access_review`, `sod_rule` and `break_glass_session`, plus the RLS policies and `registerModuleSchema`.
- **From A08:** the adapter registry, `packages/config`, and the `Clock` and `SeededRandom` interfaces.
- **From A10:** `audit.write` inside the caller's transaction.
- **Break-glass clock port (A13 registers it; not a dependency):** `registerBreakGlassClock({ openClock, businessDaysAfter, emit })`. Until A13 registers, and in A11's own tests and hidden oracle, a recorded fake with A13's frozen signatures answers; a missing registration at boot in the composed stack is a startup error, not a silent no-op.
- **Liveness port (A22 registers it; not a dependency):** `registerLivenessVerifier({ startVerification, getResult })` with `sessionAssurance: 'passkey_present'` and the liveness result. Until A22 registers, and in A11's own tests and hidden oracle, a recorded fake replays A22's frozen sandbox scenarios (the legal-name markers below).

**1. Principal model (the one table every other node keys on)**

- **One `principal` row per actor.** `principal_type` (A03's `principals.ts`) is one of `HUMAN`, `PARTNER`, `AGNT` or `SVCA`, with a `principal_kind` under it:
  - **HUMAN / `investor`.** Holds one of the investor roles INV-W, INV-S, INV-E, INV-J, INV-R, INV-F or INV-M on the investor record. These are A03 roles with their own cells.
  - **HUMAN / `sponsor_user`.** SP-OWN, SP-OPS, SP-FIN or SP-IR, always with a `tenant_id`.
  - **HUMAN / `staff`.** The 26 RealX staff roles, in the platform tenant.
  - **PARTNER.** Only the six partner roles A03 marks `signsIn: true`: TP-RE, TP-VAL, TP-PM, TP-AUD, TP-OBS and TP-LIC. A98 builds their rows and portal. A11 defines the type, the scope shape ("assigned vehicles / engagement / time box") and the rule that TP-AUD and TP-OBS are read-only and time-boxed, which is A03's `time-boxed` constraint. TP-CUS, TP-REG, TP-KYC, TP-BANK and TP-SIGN have `signsIn: false` and no cells. They are adapter counterparties reached through A08's sandbox adapters, or SVCA integration principals, and never interactive sign-ins.
  - **AGNT.** An agent acting on behalf of exactly one HUMAN or PARTNER principal. `on_behalf_of_principal_id` is not null and A12 fills the mandate. It covers both the staff agent (role id AGNT) and a user's own agent (AGT-U). Both share every cap in this node and differ only in whose cells they inherit.
  - **SVCA.** A named service account (eligibility job, recon job, indexer, notification sender, paymaster policy signer) holding only A03's fixed execute cells with policy `AUTO`. It has no interactive sign-in.
  - VIS is the anonymous caller. It is not a principal row. Unauthenticated requests reach only routes whose `requiredScope` is `public` or a VIS cell.
- **Principal ids are prefixed and stable** (`PRN-H-…`, `PRN-P-…`, `PRN-A-…`, `PRN-S-…`) so audit rows and evidence are readable. One human may hold an investor account and a sponsor-user membership. These are two principals linked by `person_id`, never one principal with two hats, so tenant context and RBAC never mix them.
- **Effective permission set (the one function every check uses).** `effectiveCells(principal, at)`:
  - **HUMAN or PARTNER:** the union of A03 cells over the principal's active role assignments at `at`, plus the cells of any active delegation to them (§6).
  - **AGNT:** the effective cells of the principal it acts for ∩ the mandate scopes from the registered `MandateProvider` (below), keeping only cells whose action has `delegable: true`. That drops every approve, ratify and sign cell, every `false` cell and every `"counsel"` cell. Execute, submit, draft, read and export cells that are delegable stay. So an investor agent with an invest-trade mandate can place an order, and one with a money mandate can request a withdrawal, within A12's caps, exactly as its person could on screen.
  - **SVCA:** its fixed A03 cells.
- **Hard structural rules (database constraint plus domain check).** Violating writes fail with `SOD_HARD_CONFLICT` and A03's rule id:
  - No AGNT or SVCA principal is ever given a role assignment or delegation that carries an approve, ratify or sign cell (A03 rule 2).
  - For segregation of duties an AGNT counts as its human, which A03's `checkSod` already models.
  - SUPER never holds TRSY, FINA, CFO, TOKO or any custody-signer role (A03 rule 3).
  - FINA/CFO and VALO/MKTO are exclusive (A03 rule 5).
- **`MandateProvider` port (fails closed).** `registerMandateProvider({ scopesFor(agentPrincipalId, at): Promise<ActionId[] | null> })`. With no provider registered, or a `null` result, an AGNT's effective set is empty, so every agent call is refused with `forbidden` and reason `MANDATE_ABSENT`. The visible suite registers a fixture provider. A12 registers the real one. A11 never reads mandate tables.
- **`GET /v1/iam/me`** returns the principal, its type, kind and tenant, its effective roles, and its effective cells as A03 actionIds with constraint, family and delegable flag. For an AGNT it also returns the human it acts for. A12 uses this as the ceiling for token scopes. The UI nodes use it to hide what the API would refuse.

**2. Passkeys (WebAuthn) for investors, sponsor users, partners and staff**

- **Ceremonies.** Registration and authentication run through `@simplewebauthn/server` (13.x line or latest patch) with resident (discoverable) credentials and `userVerification: required`.
- **Relying-party id per surface.** `web` covers public, investor, sponsor and partner. `console` covers staff and is a different origin, so the two never share credentials or cookies.
- **Endpoints:**
  - `POST /v1/auth/passkeys/register/options`
  - `POST /v1/auth/passkeys/register/verify`
  - `POST /v1/auth/passkeys/login/options`
  - `POST /v1/auth/passkeys/login/verify`
  - `GET /v1/auth/passkeys`
  - `DELETE /v1/auth/passkeys/{id}`. The last login passkey cannot be deleted. Deleting any passkey revokes all sessions except the current one and writes `auth.passkey.removed`.
- **Login and wallet credentials are separate.** `passkey` rows carry `purpose`, either `login` or `wallet`. Login passkeys are separate credentials from the wallet passkeys that sign UserOps (A29 creates and stores those). A `wallet` credential presented to a login endpoint is refused with `PASSKEY_WRONG_PURPOSE`, and vice versa.
- **Cloned authenticators.** A counter regression refuses the login, locks the credential and writes `auth.passkey.counter_regression`.
- **Staff passkeys require hardware attestation:**
  - The register-verify step accepts only attestation statements whose AAGUID is on the allow-list `STAFF_AUTHENTICATOR_AAGUIDS` (§11). It rejects `none` attestation for staff, and it stores the AAGUID and attestation format on the row.
  - The sandbox authenticator used by tests carries an allow-listed AAGUID so the rule is exercised locally.
  - The production list is a SECO decision recorded at go-live (see human gates).
- **Sign-up for investors and sponsor users:**
  1. `POST /v1/auth/signup` with an email creates an `email_verification` row (random 32-byte token, single use, 24 h expiry, at most 3 resends per hour).
  2. `POST /v1/auth/verify-email` creates the `account` and `principal` in state `email_verified`.
  3. Passkey registration moves the account to `active` and sets `last_successful_login_at` to that moment.
  No account exists before the email is verified. No password field exists anywhere. Email is an identifier, never a login.
- **No account enumeration.** `POST /v1/auth/signup` and `POST /v1/auth/recovery/start` always return the same `202` body (`{"status":"accepted"}`) for known and unknown emails, and do the same work on both paths so the timing matches. The email content differs: a known address on sign-up gets "you already have an account, sign in or recover". Failed logins never say whether the account exists.
- **Invitations.** Sponsor users and partners are created by invitation (A33, A98). Staff are invited by a role grant (A67). The invited principal is in state `invited` and accepts through the same verify-email and passkey path; staff accept on the console origin with a hardware-attested passkey. An invitation not accepted within `AUTH_INVITE_TTL` (default 7 days; A67's staff re-enrolment invitation is 24 h) moves the account to `closed` with reason `invitation_expired` and writes `iam.account.invitation_expired`. Dormancy (§3) never applies to an `invited` account, because it has never been active.
- **Email is a port, not A43's adapter.** A43 (AM3) owns `packages/adapters/email` and depends on A14, which depends on A11, so A11 cannot depend on it. A11 therefore defines an `EmailSender` port in `modules/auth/ports/email.ts`: `send({to, template, vars, idempotency_key})`. It ships a local-inbox implementation that writes to its own `auth_mail_capture` table, which tests read through `GET /__test/auth/inbox?to=` (local and test run modes only, absent from OpenAPI). `setEmailSender()` is exported from `index.ts`. When A43 lands, its integration step registers A43's email adapter behind this port and drops the capture table from use. A43's spec must say so.

**3. Sessions**

- **Server-side sessions only.** Sessions live in Postgres (the `session` row). No JWT is used for browser sessions.
- **Cookies.** All are `__Host-` prefixed (`__Host-rx_sid` for the session id, `__Host-rx_dev` for the device secret, `__Host-rx_csrf` for the CSRF token). All are `Secure` and `Path=/` with no `Domain`. The session and device cookies are also `HttpOnly`. `SameSite` is `Lax` on the web origin and `Strict` on the console origin.
- **CSRF.** Every cookie-authenticated request that changes state (any method other than GET, HEAD or OPTIONS) must pass both checks, or it is refused `403 CSRF_REJECTED` and `auth.csrf_rejected` is written:
  - the `Origin` header equals the surface's configured origin, or, if `Origin` is absent, `Sec-Fetch-Site` is `same-origin`;
  - the `X-CSRF-Token` header equals the `__Host-rx_csrf` cookie (double submit).
  Requests authenticated by an A12 bearer token carry no cookies and are exempt. A request carrying both a cookie and a bearer token is A12's refusal.
- **Device binding.** At login the server issues the device-secret cookie. Each request must present a session whose stored `device_hash = sha256(device_secret || user_agent_family)` matches. Otherwise the session is revoked and `auth.session.device_mismatch` is written with `DEVICE_MISMATCH`.
- **Timeouts** (defaults, from §11):
  - Idle: 30 min for investors, sponsor users and partners; 15 min for staff.
  - Absolute: 12 h for investors, sponsor users and partners; 8 h for staff.
  A request after a timeout returns `401 SESSION_EXPIRED`. The UI nodes use that code to show the sign-in card with the return path.
- **Dormancy (30 days, privileged accounts).** It applies to staff, SP-OWN, SP-FIN and partner principals in state `active`:
  - **Rule.** An account is dormant when `clock.now() - last_successful_login_at >= 30 × 24 h`. `last_successful_login_at` is set at activation (first passkey registration), at every successful passkey login, and at unlock (below).
  - **Check at sign-in.** The rule is checked inside `login/verify` before a session is created. A dormant result moves the account to `dormant`, refuses with `ACCOUNT_DORMANT` and writes `iam.account.dormant`.
  - **Daily job.** A job (§12) applies the same rule to every account and writes `iam.account.dormant` for accounts not already dormant. The event is written exactly once per dormancy, whichever path finds it first.
  - **Boundary.** Measured from the last sign-in at T0, a sign-in at T0 + 30 d - 1 min succeeds and resets the clock, and a first attempt at exactly T0 + 30 d is refused.
  - **Unlock (4E, through the pending-approval pattern of §6).**
    - Staff: maker SECO; checker CCO or a second SECO.
    - Sponsor users: maker their SP-OWN (not the dormant person), or SECO; checker SECO or CCO.
    - Partners: maker PART or SECO; checker SECO or CCO.
    The unlocked person and their agents are excluded from both sides. Unlock moves the account `dormant -> active`, sets `last_successful_login_at` to the unlock time and writes `iam.account.unlocked`. If the person does not sign in within `AUTH_UNLOCK_SIGNIN_WINDOW` (default 72 h, as A67 states), the account returns to `dormant`.
- **Listing and revoking sessions.** `GET /v1/auth/sessions` lists the principal's sessions (device label, created, last seen, current flag). `DELETE /v1/auth/sessions/{id}` revokes one; `DELETE /v1/auth/sessions` revokes all others.
- **Forced revocation.** Every session of a principal is revoked, and `auth.session.revoked` is written with the reason, on any of: a role grant, revoke or expiry; a delegation starting or ending; a passkey removal; recovery completion; a dormancy lock; account closure; a kill switch (A12).
- **Rate limits** apply to every `/v1/auth/*` endpoint, per IP and per account (defaults 10 attempts per 10 min per account for login, 5 per hour per IP for signup and recovery start). Exceeding a limit returns `429 RATE_LIMITED` and writes `auth.rate_limited`.

**4. Tenant context, RLS variables and "view as tenant"**

- **One tenant per request.** Every authenticated request resolves exactly one tenant. A request that reaches a handler without a tenant context is a 500 caught by a test, never a silent cross-tenant read.
- **The RLS variables.** A11 fills the request context with every row-level-security variable A04's policies read, using A04's names. A09's `withTx` must apply them with `SET LOCAL`; today it sets only the first three, and the rest need an **A09.x follow-up loop** so that `withTx` applies every `app.*` key in `ctx.db_settings`, falling back to the deny value. An empty string is the deny value everywhere.

| Variable | investor HUMAN | sponsor_user HUMAN | staff HUMAN | PARTNER | AGNT | SVCA |
|---|---|---|---|---|---|---|
| `app.tenant_id` | platform tenant | their sponsor tenant | platform tenant | platform tenant | same as the principal it acts for | the tenant the job names explicitly, never inherited |
| `app.principal_id` | own id | own id | own id | own id | the agent's own id | own id |
| `app.principal_kind` | `HUMAN` | `HUMAN` | `HUMAN` | `PARTNER` | `AGNT` | `SVCA` |
| `app.platform_access` | `false` | `false` | `true` | `false` | same as its principal | `true` only for platform jobs A04 lists, else `false` |
| `app.view_as_tenant` | empty | empty | the tenant of the active grant, else empty | empty | always empty (agents never view as tenant) | empty |
| `app.investor_id` | their investor id | empty | empty | empty | the investor id of its principal, if any | empty |
| `app.vehicle_scope` | empty | empty | empty | comma list of assigned vehicle and engagement ids | same as its principal | empty |
| `app.principal_role` | comma list of effective role ids | same | same | same | the role ids of its principal | `SVCA` |
| `app.sandbox_of` | empty (browser sessions are live) | empty | empty | empty | set by A12 or A99 for sandbox tokens only | the partition the job runs for, else empty |

- **Staff "view as tenant".** `POST /v1/iam/view-as-tenant` with `{tenant_id, reason}` creates a `view_as_tenant_grant` bound to the current session (TTL 60 min, extendable once). It sets `app.view_as_tenant` for later requests, which narrows the staff session to that tenant. It writes A04's event name `tenant.viewed_as` with the reason. Every request under the grant is audited with `tenant_id` = the viewed tenant and `view_as_grant_id` in the event detail; A67's `acting_tenant_id` filter reads that pair.
  - Ending the grant, session expiry or logout writes `tenant.viewed_as_ended`.
  - Only staff holding A03's view-as cell (the M27 or M02 read action A03 names for it) may start a grant. AGNT principals never may.
  - A request under an expired grant returns `VIEW_AS_NOT_ALLOWED`.
- **Cross-sponsor data.** Investors' shared KYC records cross tenant boundaries only through the `tenant_consent` rule (A30). Nothing in A11 grants a sponsor sight of another sponsor's rows.

**5. RBAC middleware over A03's matrix**

- **Every route declares a scope.** Each route declares `meta.requiredScope`: an A03 actionId, or `public`. A09 already fails registration on a missing or unknown id. On each request the middleware:
  1. resolves `effectiveCells(principal, now)`;
  2. checks that the cell for `requiredScope` exists and that its constraint (own-records, own-tenant, own-vehicles, time-boxed) holds for the request's subject;
  3. refuses with `403 forbidden` in A09's envelope, with `details {actionId, role_set}` and `reason_codes` (`NO_CELL`, `CONSTRAINT_FAILED`, `MANDATE_ABSENT`, `NOT_DELEGABLE`, `COUNSEL_PENDING`).
  Deny by default.
- **Human-only routes come from A03's data.** `requiresHuman` is derived from A03's `delegable.ts`, not declared by hand: a route whose action has `delegable: false` or `"counsel"` refuses every AGNT with `403 AGNT_NOT_ALLOWED`, whatever the mandate says. This check runs after A12's token verification, kill-switch and rate-limit steps and before any scope or G22 evaluation, in the order A12 §5 'Order of agent refusals' fixes. The refusal is a G22 decision: the envelope carries `gate: G22` and the G22 cause in `reason_codes` (`G22_SCOPE_NON_DELEGABLE`, `G22_SCOPE_PENDING_COUNSEL` or `G22_AGENT_NEVER_CHECKER`, naming the cell), and A12's middleware writes the same three rows as for every token request (one G22 `gate_evaluation`, one `api_token_activity` row with error code `AGNT_NOT_ALLOWED`, one `agent.access` audit event with `token_id` and `on_behalf_of_id`), so the refusal appears in the person's activity. `G22_SCOPE_NON_DELEGABLE` and `G22_SCOPE_PENDING_COUNSEL` are never the error code of a `requiresHuman` route. A03's D04 list covers:
  - identity and liveness verification;
  - creating the first passkey or wallet;
  - creating, widening or rotating API tokens;
  - changing the withdrawal bank account;
  - account recovery and guardian changes;
  - closing the account;
  - every approve, ratify and sign action;
  - the counsel-pending declarations (wholesale-status truthfulness, target-market answers, the sponsor declaration, FATCA/CRS self-certification).
  Counsel's answers change A03's data, not this node.
- **Generated matrix tests.**
  - The generator reads A03's exported JSON (`npm run domain:export`, the hash recorded in `app/evidence/A03.json`). It mounts one probe route per actionId in the test harness, declaring that actionId as `requiredScope`, with a subject fixture that satisfies the cell's constraint.
  - It emits one Vitest case per (roleId, actionId) pair over A03's 51 roles × every actionId. Each case calls the probe as a principal holding exactly that role and expects 2xx where A03 has the cell and 403 where it does not. VIS is tested as the anonymous caller.
  - A second pass calls each allowed probe with a subject that breaks the constraint and expects 403 `CONSTRAINT_FAILED`.
  - The hidden oracle runs the same generator against the node QA's own copy of A03's export. A builder who changes the data to make tests pass fails the oracle. The real business routes are checked later by A100.
- **`eligibleCheckers(request, at)`** is exported for A14. Given a draft approval, it returns the principals who may act as checker. It excludes:
  - every member of `makerIdentitySet` (§6);
  - every AGNT and SVCA;
  - anyone for whom `checkSod` returns a hard conflict with the maker on that action;
  - anyone whose checker role was granted after the request was drafted (a grant counts only for requests drafted after it).

**6. Segregation of duties, role grants and delegation**

- **A11 calls A03's `checkSod`; it does not re-implement it.** A11 calls it at three points:
  - **Grant time:** a role assignment, delegation or SoD exception is checked against the principal's current roles.
  - **Request time:** every request whose action kind is submit, execute, approve, ratify or sign is checked with the current role set and the request's maker identity set. A role granted after a request was drafted, an agent acting for the maker, or a delegation therefore cannot create a checker who is really the maker.
  - **Custody signatures:** A15 calls through A14 with the same function.
  A refusal writes `iam.sod.violation` with the rule id and the principals involved. Rule wording, hard or soft type and reason codes come from A03's data, including rows A03.1 adds.
- **Pending-approval pattern.** Used for every 4E act this node owns, before A14 exists:
  - The route creates the row in state `pending_approval` and returns `202` with the pending id.
  - An internal activate hook, exported from `index.ts`, is what A14's effector calls with `{approval_id, decided_by}`.
  - Each hook re-checks for itself that `decided_by` is a HUMAN holding a checker role for that act, is not in the maker identity set, is not the subject of the act, and is not an AGNT or SVCA. Otherwise it refuses with `SOD_SELF_GRANT` or `APR_CHECKER_MUST_BE_HUMAN`.
  - Until A14 exists, tests call the hooks directly with fixture decisions.
  - The hooks are `activateGrant` (role grants), `activateDelegation` (delegations), `activateSodException` (SoD exceptions), `activateUnlock` (dormancy unlock, §3) and `confirmRecovery` (login-recovery confirmation, §9).
- **Role grants.**
  - Route: `POST /v1/iam/roles/grant`. For staff, SUPER is maker and SECO is checker (4E); for sponsor users, SP-OWN is maker and a second SP-OWN or SECO is checker.
  - On submit: hard conflicts with existing roles refuse with `SOD_HARD_CONFLICT`. Self-grant refuses with `SOD_SELF_GRANT`: the maker is the grantee, or names themselves as checker, or acts through an AGNT whose human is the grantee. Soft conflicts need an active `sod_exception` (time-boxed, at most 90 days, approved by CCO + SECO through `activateSodException`, reported monthly) and refuse with `SOD_EXCEPTION_REQUIRED` without one.
  - A `role_assignment` has `expires_at`.
  - Events: `iam.role.granted` on activation, `iam.role.revoked` on revoke, and `iam.role.expired` when the job ends an expired assignment. The revoke route itself is the IAM admin API node A106's (A67's working id A11.1).
- **Delegation (one person to another).**
  - **Row.** `delegation {id DLG-…, delegator_principal_id, delegate_principal_id, action_ids[] (a subset of the delegator's own cells, never an approve, ratify or sign cell, never a non-delegable or counsel-pending cell), valid_from, valid_to (at most AUTH_DELEGATION_MAX_DAYS, default 30), reason, state (pending_approval | active | expired | revoked | rejected), approval_id}`.
  - **Who.** Only HUMAN staff and sponsor users delegate, and only to another HUMAN of the same tenant. Investors use agents (A12) or the joint-account rules instead.
  - **How.** `POST /v1/iam/delegations` is a 4E grant through `activateDelegation`: for staff the maker is the delegator and the checker is SECO; for sponsor users the maker is the delegator and the checker is an SP-OWN who is neither party. `checkSod` runs on the delegate's resulting cell set.
  - **Reading and revoking.** `GET /v1/iam/delegations` lists them. `DELETE /v1/iam/delegations/{id}` revokes (either party, SECO, or the tenant's SP-OWN).
  - **Automatic revocation.** A delegation is revoked when either party has a role change, and it expires at `valid_to` (job, §12).
  - **Events:** `iam.delegation.requested`, `iam.delegation.activated`, `iam.delegation.revoked`, `iam.delegation.expired`.
- **`makerIdentitySet(principalId, at)`** is exported for `eligibleCheckers` and A14. It returns:
  - the principal, or its human if it is an AGNT;
  - every human who delegated to, or received a delegation from, that human, in either direction, where the delegation was active at `at`;
  - every AGNT whose `on_behalf_of` is any of those humans.
  A14 calls it at submit and at every decision and takes the union, which matches A14's "active at either moment".
- **Review set.** `GET /v1/iam/roles/review-set` returns every active assignment and delegation with its last-used date, for A67's quarterly recertification. A12 adds tokens beside it.

**7. Break-glass sessions**

- **Request.** `POST /v1/iam/break-glass/request` with runbook id R1-R9, reason and `requested_duration`. It may be made by CCO, SECO, or an ENGO who is currently on call.
- **On call.** "On call" is an `on_call` attribute on an ENGO `role_assignment`, with `on_call_from` and `on_call_to`. It is granted through the same 4E grant path (SUPER maker, SECO checker) and is valid only inside its window. A48's HG-OPS roster supplies the real names later; A06 seeds one on-call ENGO persona.
- **Activation.** The session activates only when 2 of the 3 sealed-credential holders (CCO, SECO, CEO) each submit a fresh assertion (§8, purpose `breakglass_activate`) over the request hash (`POST /v1/iam/break-glass/activate`). The two must be different people. The requester may be one of them. On activation:
  - `expires_at = activated_at + min(4 h, requested_duration)`;
  - `recorded = true` is flagged (the UI nodes show the banner and flag; the recording itself is a production concern outside this build);
  - these events are written: `iam.breakglass.activated`, `iam.breakglass.board_notice` (the board notice required by A03 rule 10, a payload the notification node later delivers), and A13's alert `iam.break_glass_active` via `emit`;
  - the break-glass clock port's `openClock({kind: 'breakglass_review', dueAt: businessDaysAfter(activated_at, 5, calendar)})` opens the 5-business-day review (A13's implementation once registered; the recorded fake at this node's gate).
- **Elevation sets.** These are owned here, in `modules/auth/break-glass/elevations.ts`: one entry per runbook R1-R9 listing the extra A03 role ids granted while the session is active. A48's `runbooks.ts` and A103's drills reference these entries by runbook id and do not copy them. Elevation never:
  - grants an approve cell on the requester's own requests;
  - creates a signer role;
  - lifts A03 rules 1, 2 or 3;
  - applies to an AGNT.
- **Ending.** Expiry, or `POST /v1/iam/break-glass/close`, revokes the elevation and writes `iam.breakglass.closed`. The expiry job (§12) runs every minute of injected clock time, and the middleware also checks `expires_at` on each request, so the elevation stops at `expires_at` even with no traffic and no job run.

**8. Fresh-assertion primitive (used by A12, A14, A53, A67 and every passkey-confirmed action)**

- **Canonical payload.** The canonical form is RFC 8785 JSON Canonicalization Scheme (sorted keys, no whitespace, UTF-8), with strings NFC-normalised, money as integer cents and units as whole integers (no floats). This is the same rule A14 uses for `payload_canonical`.
  - `canonicalPayloadSha256(obj)` returns `payload_sha256`, the lower-case hex SHA-256 of that form. It is exported from `index.ts`, and every caller hashes through it.
  - The bulk payload hash is `sha256` of the lower-case hex `payload_sha256` values, sorted ascending and joined with `\n`, as A53 and A14's bulk-decide expect.
- **One exported verifier.** `verifyAndConsumeFreshAssertion({principalId, payload_sha256, purpose, maxAgeMinutes, assertion?})` returns `{assertionId}` or throws a stable code. It has two challenge modes:
  - **Nonce mode** (A11, A12 and A67 routes). `POST /v1/auth/fresh-assertion/options` with `{payload_sha256, purpose}` stores a nonce row and returns the challenge `sha256(server_nonce || payload_sha256 || purpose)`. `POST /v1/auth/fresh-assertion/verify` verifies the assertion and stores a `fresh_assertion` row `{principal, credential id, aaguid, payload_sha256, purpose, issued_at, consumed_at null}`. The route middleware `requireFreshAssertion({maxAgeMinutes, purpose})` calls the verifier over the request's own canonical payload hash and consumes the row atomically. Defaults: approvals 5 min; token creation and scope widening 5 min; bank-account change 10 min.
  - **Inline mode** (A14 `decide`, `ratify`, `veto` and `bulk-decide`). The raw assertion is passed in `assertion`, and its challenge must equal `payload_sha256` exactly, as A14 specifies. The function verifies it and records the `fresh_assertion` row as consumed in the same call. Replay is refused because one (principal, purpose, payload_sha256) can be consumed only once and the authenticator counter must not regress. Staleness in this mode comes from A14's request TTL. A14 must use this function; it has no verifier of its own.
- **Refusals.** `FRESH_ASSERTION_REQUIRED` (none, or already consumed), `FRESH_ASSERTION_STALE` (older than `maxAgeMinutes`), `FRESH_ASSERTION_MISMATCH` (different payload hash or purpose).
- **Rules for every principal type:**
  - **Staff:** the credential must be hardware-attested, and its stored AAGUID is re-checked against the current `STAFF_AUTHENTICATOR_AAGUIDS` list at assertion time, not only at registration. If an AAGUID is removed from the list, that key can no longer approve (`AAGUID_NOT_ALLOWED`).
  - **Investors, sponsor users and PARTNER principals:** any active login passkey.
  - **AGNT and SVCA:** always refused (`AGNT_NOT_ALLOWED`). Agents have no passkeys.
- The `assertionId` is what A14 stores on each approval decision and A12 stores as `created_with_assertion_id`.

**9. Lost-login-passkey recovery (separate from wallet recovery SM-13)**

- **Start.** `POST /v1/auth/recovery/start` with the verified email always returns the same `202` (§2). For a known account, it sends a magic link (single use, 30 min). Following the link opens a `login_recovery_request` in state `link_used`.
- **Re-verification:**
  - **Investors** re-verify through the liveness port (A22's identity adapter once A22 registers; the recorded fake at this node's gate): `startVerification` with `sessionAssurance: 'passkey_present'` and a liveness capture, then `getResult`. A22's sandbox scenarios are chosen by the legal-name marker: no marker gives `pass`, ` REFER` gives `refer`, ` REJECT` and ` FRAUD` give `fail`, and ` SLOW` stays pending until the clock advances. Only `pass` with `liveness.performed = true` moves the request to `reverified`. `refer` holds it for staff review. `fail` moves it to `cancelled` with an audit row.
  - **Sponsor users and partners** re-verify either by liveness as above, or by confirmation (4E through `confirmRecovery`, §6). For sponsor users an SP-OWN of the same tenant who is not the recovering person confirms; for partners, PART; SECO checks in both cases. The recovering person and their agents are excluded.
  - **Staff** never use this flow. SECO re-enrols them through A67.
- **Cooling.** A cooling period follows: default 24 h for investors, sponsor users and partners, configurable per principal kind (§11). A notice goes to every registered channel with a one-click cancel that works from any signed-in device. A request during cooling returns `RECOVERY_COOLING`.
- **Completion.** At `cooling_elapsed` the person registers a new login passkey. Completion:
  - revokes every session and every existing login passkey (the person may re-add devices);
  - publishes `auth.recovery.completed` once, with `{principal_id, request_id, completed_at}`, so A12 can revoke that principal's API tokens (A12's challenge checks the tokens);
  - writes the audit trail.
  Wallet passkeys and guardians are untouched; that is SM-13 in A29.

**10. Events, errors, empty states**

- **Every event goes through A10**, with `actor_id`, `on_behalf_of_id` for AGNT actors, `tenant_id`, `view_as_grant_id` when viewing as a tenant, `break_glass_id` when elevated, `session_id` and the request id.
- **The event list lives in `modules/auth/events.ts`** so the node QA can diff it against the hidden list. Consumers named in brackets.
  - Sign-in and credentials: `auth.signup.requested`, `auth.email.verified`, `auth.passkey.registered`, `auth.passkey.removed`, `auth.passkey.counter_regression`, `auth.login.succeeded`, `auth.login.failed`.
  - Sessions and request protection: `auth.session.revoked` (with reason), `auth.session.device_mismatch`, `auth.csrf_rejected`, `auth.rate_limited`.
  - Recovery: `auth.recovery.started`, `auth.recovery.reverified`, `auth.recovery.cancelled`, `auth.recovery.completed` (A12, A29, A67).
  - Accounts: `iam.account.dormant`, `iam.account.unlocked`, `iam.account.invitation_expired`, `iam.account.closed` `{principal_id, closed_at, reason}` (A12, A29, A44).
  - Roles and delegations: `iam.role.granted` `{assignment_id, principal_id, role_id, approval_id, expires_at}`, `iam.role.revoked` `{assignment_id, principal_id, role_id, reason_code}` (A12, A14, A44), `iam.role.expired` (same payload as revoked), `iam.delegation.requested`, `iam.delegation.activated`, `iam.delegation.revoked`, `iam.delegation.expired` `{delegation_id, delegator, delegate, action_ids, valid_to}` (A14).
  - Segregation of duties: `iam.sod.violation`, `iam.sod_exception.activated`.
  - View as tenant: `tenant.viewed_as`, `tenant.viewed_as_ended`.
  - Break-glass: `iam.breakglass.requested`, `iam.breakglass.activated`, `iam.breakglass.board_notice`, `iam.breakglass.closed`.
  - Fresh assertions: `auth.fresh_assertion.consumed`.
  Login failures, refusals and rate limits are audited as much as successes.
- **Error codes** are registered through A09's `registerErrors()` with their HTTP status and listed in the OpenAPI document. A11 uses A09's core `forbidden` for matrix refusals, with A11's reason codes in `reason_codes`. Its own codes are:
  - Sign-up and passkeys: `EMAIL_NOT_VERIFIED`, `VERIFICATION_EXPIRED`, `PASSKEY_WRONG_PURPOSE`, `ATTESTATION_REQUIRED`, `AAGUID_NOT_ALLOWED`.
  - Sessions and accounts: `SESSION_EXPIRED`, `DEVICE_MISMATCH`, `CSRF_REJECTED`, `ACCOUNT_DORMANT`, `ACCOUNT_INVITATION_EXPIRED`, `RATE_LIMITED`.
  - Authority and SoD: `AGNT_NOT_ALLOWED`, `SOD_HARD_CONFLICT`, `SOD_SELF_GRANT`, `SOD_EXCEPTION_REQUIRED`, `DELEGATION_NOT_ALLOWED`.
  - Fresh assertions: `FRESH_ASSERTION_REQUIRED`, `FRESH_ASSERTION_STALE`, `FRESH_ASSERTION_MISMATCH`.
  - Break-glass and view-as: `BREAK_GLASS_QUORUM`, `BREAK_GLASS_EXPIRED`, `VIEW_AS_NOT_ALLOWED`.
  - Recovery: `RECOVERY_COOLING`.
  These stay upper-case, as A09 allows for module codes (A10's `AUDIT_*`) and as A12, A14, A53, A100, A101 and A104 already consume them.
- **Empty states are explicit.** A principal with no passkeys, no sessions, no roles, no delegations or no assignments to review, and a tenant with no users, all return `200` with empty arrays and a `hint` field the UI can render.
- **Sandbox only.** The liveness port's recorded fake of A22's identity sandbox for re-verification, A11's local-inbox `EmailSender` for verification and magic links, and a software WebAuthn authenticator with configurable AAGUID for tests. No vendor calls and no secrets in the repo.

**11. Schema and configuration**

- **Tables A11 registers itself** through A04's `registerModuleSchema({node: 'A11', tables, migrationsDir})`. Drizzle schema is in `modules/auth/schema.ts`, and numbered migrations are under `modules/auth/migrations/`. Every table has `tenant_id`, RLS, enum state columns and `sandbox_of` as A04's conventions require. The tables are:
  - `email_verification`
  - `fresh_assertion`, holding both nonce and consumed-assertion rows
  - `view_as_tenant_grant`
  - `login_recovery_request`
  - `sod_exception`
  - `delegation`
  - `auth_mail_capture`, the local inbox
- **A04.x follow-up loop (requested at Specify; A11 does not edit `packages/db`):**
  - **Constraint trigger on `role_assignment`.** It refuses at insert and update any assignment that breaks A03 rules 2, 3 or 5 (AGNT or SVCA with an approve, ratify or sign cell; SUPER with TRSY, FINA, CFO, TOKO or a custody-signer role; FINA with CFO; VALO with MKTO). It uses role sets generated from A03's export, so the direct-SQL challenge refuses at the database.
  - **Columns on `session`, if A04 lacks them:** `device_hash`, `idle_expires_at`, `absolute_expires_at`, `break_glass_id`, `view_as_grant_id`, `csrf_hash`.
  - **Columns on `passkey`:** `purpose`, `aaguid`, `attestation_fmt`, `sign_count`, `locked_at`.
  - **Columns on `account`:** the state enum `invited | email_verified | active | dormant | locked | closed`, `last_successful_login_at`, `dormant_since`, `unlocked_at`, `invitation_expires_at`.
  - **Columns on `principal`:** `person_id`, `on_behalf_of_principal_id`.
  - **Column on `role_assignment`:** the `on_call` window.
- **Environment keys (A08.x follow-up loop adding them to `packages/config`'s `envSchema`, which is the only reader of `process.env`):**
  - `AUTH_RP_ID_WEB`, default `localhost`;
  - `AUTH_RP_ORIGIN_WEB`, default `http://localhost:3000`;
  - `AUTH_RP_ID_CONSOLE`, default `console.localhost`;
  - `AUTH_RP_ORIGIN_CONSOLE`, default `http://console.localhost:3001`;
  - `STAFF_AUTHENTICATOR_AAGUIDS`, a comma list whose default is the sandbox authenticator's AAGUID.
- **Module-local tunables.** All other tunables are a zod object in `modules/auth/config.ts` with the defaults above. It is pure constants, reads no environment, and tests can override it by injection. The tunables are the idle and absolute timeouts per kind, dormancy days, `AUTH_UNLOCK_SIGNIN_WINDOW`, `AUTH_INVITE_TTL`, cooling period per kind, the view-as TTL, `AUTH_DELEGATION_MAX_DAYS`, fresh-assertion maximum ages and auth rate limits.

**12. Worker jobs**

These are registered through A09's `registerJob`, in `services/worker/src/jobs/auth/`, and run against the injected clock:
- `auth.dormancy.sweep`: daily. Applies §3's rule and returns unlocked accounts to `dormant` when their sign-in window lapses.
- `auth.invitation.expire`: hourly.
- `auth.role.expire`: every 5 min. Ends expired role assignments and SoD exceptions; `iam.role.expired`.
- `auth.delegation.expire`: every 5 min.
- `auth.breakglass.expire`: every minute.
- `auth.viewas.expire`: every minute.
- `auth.recovery.cooling`: every minute. Moves `cooling -> cooling_elapsed` and expires stale links.
- `auth.verification.cleanup`: daily. Removes expired email verifications and unconsumed fresh-assertion nonces older than 24 h.

## Owned paths (other nodes do not edit these)

- `<workspace>\app\services\api\src\modules\auth\`
- `<workspace>\app\services\worker\src\jobs\auth\` (the generated graph reads this section, so `Data\app-graph.json` carries it)

Inside `modules/auth`, by convention:
- `principals/`, `passkeys/`, `sessions/`, `tenancy-context/`;
- `rbac/` (middleware and the generated matrix tests);
- `sod/` (calls to A03's `checkSod`, grants and exceptions), `delegation/`;
- `break-glass/` (including `elevations.ts`), `fresh-assertion/`, `recovery/`;
- `ports/email.ts`, `config.ts`, `schema.ts`, `migrations/`, `events.ts`;
- `index.ts`, the only import surface for other modules. It exports `effectiveCells`, `eligibleCheckers`, `makerIdentitySet`, `verifyAndConsumeFreshAssertion`, `requireFreshAssertion`, `canonicalPayloadSha256`, `registerMandateProvider`, `setEmailSender` and the five activate hooks.

A11 never edits `packages/domain` (A03), `packages/db` (A04), `packages/config` (A08) or `services/api/src/core` (A09). Needs there go to the A03.1, A04.x, A08.x and A09.x follow-up loops named above.

## Design references

- **No sign-in screen to copy.** The design has none: `login` in `<workspace>\design\RealX App.dc.html` is a one-click switch to Alex Morgan (design-analysis §6.4 bug 16, also flagged by the product-map QA). Do not reproduce it. This node's API replaces it.
- **Where the cards come from.** A52 (public, investor and sponsor) and A53 (console) draw the visible sign-in, passkey and recovery cards as centred onboarding cards. They use the design's box pattern (dashed `#B9B4AA` border, Newsreader 22 title, Geist Mono caption) already used in ONB-03 (liveness), ONB-06 (certificate upload) and ONB-08 (passkey wallet creation).
- **The design's ONB-01 Password field is dropped.** ONB-01 "Create your account" in the prototype (`RealX App.dc.html` line 815) has a Password field. It is replaced by passkey sign-up: email, then verify, then create passkey. No password field exists anywhere in RealX.
- **Tokens the dependent screens must use.** Cream `#F5F2EC`, ink `#17191C`, green `#1F5E4B`, brick `#9A4524`, alert red `#A3312A`. Fonts: Newsreader (display), Geist (text), Geist Mono (codes such as principal ids, session ids and approval ids).
- **ACC-01 and ACC-02 are not in the prototype.** ACC-01 Profile and security (a settings list with a Security row counting passkeys) and ACC-02 Recover account (a stepper) are extension layouts drawn by [[A50 - Design extension A- investor and sponsor screens not in the prototype]]; the design-analysis proposals are its input. ACC-01 is the investor view of `GET /v1/auth/passkeys` and `GET /v1/auth/sessions`. ACC-02's guardian stepper is wallet recovery SM-13 (A29), not this node. The login-recovery flow of §9 reuses A50's stepper with the steps Verify email / Re-verify / Wait / New passkey and no guardian step.
- **Product map wins on the approval step.** "Approve with hardware key" (design-analysis §6.2) is the fresh assertion of §8 in inline mode, rendered as a passkey confirmation modal in the design's visual style. A53 owns the modal.
- **Undrawn screens this node's data feeds** (product-map QA §4 item 5): users, roles, delegations, SoD exceptions, access reviews, break-glass, and sign-in for all three surfaces. The investor and sponsor ones are drawn in [[A50 - Design extension A- investor and sponsor screens not in the prototype]]; the console ones in [[A51 - Design extension B- operator console screens not in the prototype]] and built in A67 on the DataTable + detail pattern.

## Requirements covered

- **Product-map screens:** none owned. Done means every screen that signs in, changes passkeys, lists sessions, grants roles or delegations, or approves can be built on this API without a second permission check anywhere else. That covers ONB-01, ACC-01, SPN-14, ADM-30, ADM-37 and the sign-in cards.
- **Journeys:** none owned. Done means J01 step 2 (account created with passkey, email verification sent) and every "approves with hardware key" step in J05, J06, J09, J13, J15 and J20 can call this node's endpoints as specified.
- **Gates: G13 Maker-checker.** Done means:
  - `eligibleCheckers` never returns a member of the maker identity set, an AGNT or an SVCA;
  - `makerIdentitySet` covers agents and delegations in both directions;
  - every approval decision can be bound to a hardware-attested fresh assertion over the canonical payload hash;
  - a checker whose eligibility came from a role granted after the request was drafted is excluded.
  A14 builds the approvals engine on top; A11 makes the rule unbypassable.
- **Gates: G22 Agent authority (the identity half).** Done means an AGNT's effective set is its principal's cells ∩ mandate, keeping only delegable ones, and fails closed without a mandate provider. A12 owns the evaluator.
- **Modules: M25 Access, approvals and audit** (the access part: users, roles, SoD, delegation, break-glass, AI-agent principals). Done means all of the following exist as enforced behaviour with generated tests, and the approvals inbox and audit screens (A14, A10, A67) need nothing more from identity:
  - A03's 51 roles and matrix;
  - A03's ten SoD rules;
  - delegation;
  - break-glass 2-of-3 within 4 h with board notice and a 5-business-day review;
  - the 30-day dormancy lock;
  - the quarterly review set.
- **State machines:** none owned. This node adds three small internal machines, none of them one of the 14 product-map machines, each with exhaustive transition tests:
  - the `Account` lifecycle (`invited | email_verified | active | dormant | locked | closed`);
  - the `LoginRecoveryRequest` lifecycle (`started | link_used | reverified | cooling | cooling_elapsed | completed | cancelled | expired`);
  - the `Delegation` lifecycle (`pending_approval | active | expired | revoked | rejected`).
- **Founder requirement D04 (agent access).** Done means the following exist, so that A12 can issue tokens whose scopes never exceed the person's own cells and whose sensitive actions demand a passkey step-up, while an investor's or sponsor's agent keeps every delegable action its person has:
  - the AGNT principal type;
  - "agent counts as its human" through `checkSod`;
  - the never-checker, never-ratifier and never-signer caps;
  - `requiresHuman` derived from `delegable.ts`;
  - the fail-closed `MandateProvider`;
  - the fresh-assertion primitive.

## Gauntlet loop for this node

- **Specify.** This note is frozen at the start of the loop. The design refs attached are:
  - A03's export hash (the matrix, `delegable.ts` and `sod.ts`) and the A03.1 request;
  - the A04.x, A08.x and A09.x follow-up requests;
  - the RLS variable table (§4);
  - `elevations.ts` R1-R9;
  - the event list.
  A change to any of these after freezing reopens the spec.
- **Oracle.** Before the build, the node QA writes the hidden tests to `<workspace>\gauntlet\hidden\A11\`. They include:
  - **Matrix:** the generator against the node QA's own copy of A03's export.
  - **SoD:** allow and deny pairs for all ten A03 rules through the routes; self-grant attempts by every route; SUPER-holds-money attempts through route, seed script and direct SQL.
  - **Agents:** AGNT-as-checker and AGNT-as-signer attempts; an investor agent with an invest-trade mandate placing an order probe and one with a money mandate requesting a withdrawal probe within caps (both 2xx); the same agents refused on a non-delegable probe; a staff agent refused on approve and sign probes; an agent with no mandate provider refused.
  - **Delegation:** the delegator drafts and the delegate tries to check; the delegate drafts and the delegator tries to check; a delegation active when the request was drafted but expired at the decision (still excluded).
  - **Fresh assertions:** replay, staleness and payload-drift cases in both modes; a de-listed AAGUID.
  - **Sessions and CSRF:** session device mismatch and timeout cases; a cross-site POST with a valid session cookie.
  - **Enumeration:** identical sign-up and recovery-start responses for known and unknown emails.
  - **Dormancy:** at T0 + 30 d - 1 min (succeeds) and at T0 + 30 d (refused, one event).
  - **Break-glass:** with 1 of 3; with two assertions from the same holder; with 3 of 3; past 4 h with no traffic.
  - **Tenancy:** one view-as-tenant RLS probe per variable in §4.
  - **Recovery:** the activate hooks refusing self-confirmation and agents; wallet passkey used for login.
- **Build (Grok 4.7 xhigh, split by the planner).** Suggested shape: four parallel sub-agents on disjoint sub-folders, then one integration pass.
  - (a) Passkeys, email verification and the `EmailSender` port, invitations and login recovery.
  - (b) Sessions, cookies and CSRF, tenant context and RLS variables, view-as-tenant, and the worker jobs.
  - (c) Principals, `effectiveCells`, the `MandateProvider` port, RBAC middleware and the generated matrix tests.
  - (d) SoD calls, grants, exceptions, delegation, `makerIdentitySet`, `eligibleCheckers`, break-glass and the fresh-assertion primitive.
  The integration pass wires `index.ts`, the OpenAPI document, the audit events and module-schema registration. It runs the visible suite and the RLS tests against Docker Postgres and the sandbox authenticator. The builder writes `app/evidence/A11.json` with A03's export hash as consumed, the event list, coverage and test counts.
- **Challenge (node QA, Opus 5.5 high).** The node QA:
  - runs the hidden oracle and checks that the consumed A03 hash equals the frozen one;
  - attempts the node-specific challenges below by hand against the running API;
  - checks that every audit event in the spec appears in the A10 hash chain for one full scripted run;
  - checks that the OpenAPI document lists every endpoint and error code above, all under `/v1`;
  - confirms no vendor call, no password field, no JWT browser session and no secrets;
  - confirms that only the owned paths changed, plus the follow-up loops' own diffs, which are made by their owners.
  The node QA writes `app/evidence/A11.qa.md`.
- **Gate.** Pass unlocks the nine dependents. Fail returns findings by severity. The builder gets at most 3 fix loops on the same spec.
- **Escalate** to the user when:
  - three loops fail on the same assumption;
  - the builder needs A03's matrix, SoD data or delegable list changed to pass (that is an A03 spec question, not a code fix);
  - any finding shows an AGNT, SVCA or SUPER principal reaching an approve, ratify, sign or money-role cell by any route. This is Critical and stops the milestone until resolved.

## Node-specific challenges (node QA)

- **Self-grant by a side door.** Three attempts, each of which must refuse with `SOD_SELF_GRANT` and write `iam.sod.violation`:
  - SECO submits a grant of SECO to a second account they also control;
  - SUPER names themselves as the checker of their own grant;
  - a grant is made through an AGNT principal whose human is the grantee.
  Then `activateGrant` is called with `decided_by` = the grantee, which must also refuse.
- **Agent becomes a checker, and agent parity holds.** Four cases:
  - An AGNT acting for the CFO calls the approve probe on a distribution run that the CFO's own agent drafted as maker. It must fail on A03 rule 1 (the agent counts as its human), and the audit row must name the human.
  - A role assignment tries to give an AGNT an approve cell on M12. It must fail at insert on A03 rule 2, through both the domain check and the database trigger.
  - An investor's agent with an invest-trade mandate places an order probe within its caps and gets 2xx.
  - The same investor's agent calls the bank-account-change probe and gets `AGNT_NOT_ALLOWED`.
- **Money role on SUPER.** Grant CFO, then TRSY, then a custody-signer role to the SUPER account, through each of three paths: the grant endpoint, the seed script, and direct SQL under the API role. Every path must refuse (A03 rule 3), the SQL path through the A04.x trigger.
- **Delegation as a side door.** Three cases:
  - A CFO delegates a draft cell to FINA A. FINA A drafts a request, and the CFO tries to check it. Refused.
  - The delegation is revoked, and the CFO tries again. Still refused, because the delegation was active at draft time.
  - A delegation naming an approve cell refuses at submit with `DELEGATION_NOT_ALLOWED`.
- **Fresh-assertion replay and drift.** Three cases:
  - Obtain a nonce-mode assertion over payload hash H. Then submit an action whose canonical payload hashes to H', one cent different. Then submit the original twice. Then obtain a new assertion, wait N+1 minutes and submit once. Expected: `FRESH_ASSERTION_MISMATCH`; success then `FRESH_ASSERTION_REQUIRED`; `FRESH_ASSERTION_STALE`.
  - In inline mode, submit the same assertion to A14's decide probe twice. The second is refused.
  - Remove the staff credential's AAGUID from `STAFF_AUTHENTICATOR_AAGUIDS` and assert again with it. Refused with `AAGUID_NOT_ALLOWED`, although registration had succeeded.
- **Tenant leak under "view as tenant".** Three steps:
  - While a staff session views tenant A, request a list endpoint with a filter naming tenant B.
  - Run a worker job in the same process and check it has no inherited `app.tenant_id` or `app.view_as_tenant`.
  - Let the grant expire mid-pagination.
  No row of tenant B may appear, the job must run with its own explicit context, and page two after expiry must return `VIEW_AS_NOT_ALLOWED`. Then, for each RLS variable in §4, a probe sets the principal up so that a wrong value for that variable alone would leak a row, and no row leaks.
- **Session hijack, CSRF and dormancy edge.** Five cases:
  - Copy the session cookie to a client with a different device secret. Refused with `DEVICE_MISMATCH`.
  - Replay the original device secret with a different user-agent family. Refused with `DEVICE_MISMATCH`.
  - Post a state-changing request from a foreign `Origin` with a valid cookie and no CSRF header. Refused with `CSRF_REJECTED`.
  - For a staff account whose last sign-in was at T0, sign in at T0 + 30 d - 1 min. It succeeds and resets the clock.
  - On a second staff account with no sign-in since T0, try at exactly T0 + 30 d. Refused with `ACCOUNT_DORMANT`. The daily job then writes no second `iam.account.dormant`.
- **Recovery shortcuts.** Use the magic link twice. Skip re-verification by calling the new-passkey endpoint directly. Cancel from another signed-in device during cooling. Complete recovery and check three things:
  - every old session and every old login passkey is gone;
  - `auth.recovery.completed` was published exactly once with the principal id;
  - wallet passkeys and guardians are untouched.
  The token-revocation half of this check runs in A12's challenges.
- **Break-glass abuse.** Each of these is refused:
  - activating with the requester alone;
  - activating with two assertions from the same holder;
  - extending past 4 h;
  - using the elevated session to grant the requester a signer role;
  - using the elevated session to approve the requester's own pending request;
  - a request by an ENGO outside their on-call window.
  The elevation must vanish at the 4 h mark with no request. Activation must have produced the board-notice event and an open review clock due in 5 business days through the break-glass clock port (the recorded fake here; A13's real clock is proven at A13's and A23's gates).
- **Design fidelity (for the dependents, checked here as an API contract).** A52's sign-in card must be able to render every error code above as a human sentence in the design's style. The challenge fails if any refusal returns a bare 4xx without a stable code, or if any success response for a passkey ceremony omits a field the centred card needs (device label, purpose, created date).

## Pass criteria

- **Generated matrix tests:** 100% of the (roleId, actionId) pairs over A03's exported 51 roles and every actionId pass in the visible suite and in the hidden oracle. A11 has no matrix of its own, so the consumed A03 export hash equals the frozen hash.
- **Hidden-oracle agreement: 100%.** Every hidden case passes three runs in a row on the Docker stack. The cases cover:
  - the ten SoD rules (allow and deny);
  - self-grant and delegation;
  - SUPER never holding a money or signer role;
  - no AGNT or SVCA with an approve, ratify or sign cell;
  - D04 agent parity for delegable cells;
  - the fresh assertion in both modes;
  - sessions and CSRF;
  - dormancy;
  - view-as-tenant;
  - break-glass;
  - recovery.
- **Exhaustive transition tests** for the `Account`, `LoginRecoveryRequest` and `Delegation` lifecycles: every allowed transition succeeds, and every forbidden one refuses with a stable code.
- **Property tests** (fast-check, 10,000 runs) over random sequences of grant, revoke, delegate, expire and agent-mandate operations. They show that:
  - no AGNT or SVCA ever holds an approve, ratify or sign cell;
  - no AGNT holds a non-delegable or counsel-pending cell;
  - SUPER never holds a money or signer role;
  - an AGNT's effective set is always a subset of its principal's delegable cells;
  - no maker ever appears in their own `eligibleCheckers` result.
- **RLS tests:**
  - for every table with `tenant_id`, a sponsor principal reads zero rows of another tenant, with and without a staff view-as-tenant grant active in a neighbouring session;
  - one probe per RLS variable in §4 passes;
  - a context-less request reads nothing.
- **Audit:** every event named in §10 appears exactly once per triggering action in one scripted end-to-end run, including the role, delegation and account-closed events. `on_behalf_of_id` is filled for the AGNT cases, and the A10 chain verifies afterwards.
- **API surface and hygiene:**
  - The OpenAPI document covers every endpoint and error code above, all under `/v1`.
  - `packages/sdk` regenerates without manual edits.
  - `npm run verify` passes.
  - Line coverage of `modules/auth` and `jobs/auth` is at or above 90%.
  - There is no password field, no JWT browser session, no vendor call and no secret in the repo.
- **Evidence:** the manifest `app/evidence/A11.json` exists, matches disk, and records A03's consumed export hash, the event list hash, `oracleVersion` and `hiddenCasesHash`.

## Human gates touched

- **No human gate is crossed by this node.** Everything runs on the sandbox authenticator, the recorded fakes of A13's clock and A22's identity sandbox behind A11's two ports, A11's local-inbox email port and Docker Postgres.
- **HG-LEGAL (counsel questions recorded, not decided here):**
  - whether liveness re-verification through the identity provider is an acceptable proof of identity for login recovery, and what cooling period counsel expects;
  - which acts are legally personal and must stay non-delegable beyond the D04 defaults (tracked in A03's `"counsel"` flags);
  - whether break-glass review and board-notice wording meets the licence conditions;
  - whether "view as tenant" needs sponsor notice under the platform agreement.
- **HG-PROD / HG-KEYS (at go-live, not now):**
  - SECO fixes the production hardware-key AAGUID allow-list;
  - the sealed break-glass credentials are created and split among CCO, SECO and CEO in a recorded ceremony;
  - the production relying-party ids and origins are set;
  - the on-call ENGO roster is named (A48 HG-OPS).
  Until then the sandbox values are used and the console shows the sandbox environment badge (ADM-32).
- **HG-PENTEST (later milestone):** the session, CSRF, passkey, recovery, delegation and RBAC surfaces of this node are in scope of the external penetration test before launch. This node's pass is software evidence only.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
