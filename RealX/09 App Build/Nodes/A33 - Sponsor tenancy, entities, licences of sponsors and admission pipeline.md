---
title: "A33 - Sponsor tenancy, entities, licences of sponsors and admission pipeline"
created: 2026-09-23
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am3]
node: A33
milestone: AM3
lane: "Services"
oracle: yes
human_gate: none
---

# A33 - Sponsor tenancy, entities, licences of sponsors and admission pipeline

**The sponsor as a tenant of the two-tier console. It covers the admission pipeline a sponsor passes before it can list (applied, KYB, background, fee agreement, approved, tenant created, with two people at every decision step). It covers the tenant record, with guardrails no sponsor can switch off (KYC required locked on, eligibility packs, holder-cap ceiling, fee caps, whitelisting mode). It covers sponsor users with roles SP-OWN, SP-OPS, SP-FIN and SP-IR behind passkeys and identity checks, and internal approval rules that can never drop below the platform minimums and never let a person or their agent approve their own submission; these rules are enforced on A14's real submit and decide path. It covers the legal entities and issuing vehicles (Trust now, SubFund in P2), with each sponsor's licence recorded as a verified row of A21's shared licence register rather than a second register. It covers sponsor billing with GST tax invoices and the 30-day listing pause, sponsor performance and the watchlist, and revenue-share agreements. It covers a public sponsor profile that shows only figures approved through the 6E publication. Finally, it covers suspension: a protective SGL+R act that revokes the tenant's sessions, tokens and partner engagements, and a reinstatement that restores exactly what the suspension changed. A route catalogue with a D04 column says, for every route, what a person's own agent may do on it.**

**Build status:** not started. **Milestone:** AM3 (Business services). **Lane:** Services. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 12-18 agent-hours (build, challenge and verify loops). The scope is large: four local state machines, admission with A21 and A30 integration, guardrails, members and rules wired into A14, billing jobs and GST, the performance snapshots, the watchlist, revenue share, the public contract, entities and vehicles, the suspension race, the seed, the route catalogue and contract tests for four screen nodes. If the first Build loop runs over 18 hours, the orchestrator may split the node into A33a (admission, entities, vehicles, tenants, users, rules, suspension) and A33b (billing, performance, watchlist, revenue share, public profile). That split is a graph change and is not made here.

**Depends on:** [[A11 - Authentication, sessions, RBAC and segregation of duties]], [[A14 - Approvals engine and maker-checker (SM-00)]], [[A21 - Licences, feature flags and licence perimeter (G21)]], [[A30 - Investor onboarding, KYC cases and KYB (SM-01)]], [[A12 - API tokens, scopes, agent principals and mandates for every user type (G22)]]
**Unlocks:** [[A34 - Properties, valuations and proof of property (SM-03)]], [[A60 - Investor and sponsor API tokens and agents screens (ACC-05, SPN-16)]], [[A62 - Sponsor console- apply, dashboard, team and approvals, billing]], [[A69 - Console- sponsor admission, licences, sponsors and guardrails, feature flags, entities register]], [[A76 - Console- regulatory calendar and data room, sponsor performance, integrations and jobs]], [[A98 - Partner principals- trustee-RE, valuer, property manager, auditor, regulator observer and licensee users]]

**Oracle rule:** The node QA writes hidden oracle tests from this frozen spec to `<workspace>\gauntlet\hidden\A33\` before the build starts ([[App Gauntlet]] §2). No money, supply or holdings move in this node. It is still an oracle node because it controls authority: who may approve inside a sponsor, whether a sponsor may be admitted and list at all, and a suspension that revokes a tenant's sessions and tokens. D04 asks for oracles wherever authority is involved. The oracle scope:
- the pipeline, tenant, vehicle and invoice transition vectors;
- admission blocked by an A21 licence that is not `active` or not `verified`, and a tenant row whose `approval_id` points at a foreign approval;
- guardrail-immutability probes through the API, the generic A14 draft path and SQL;
- self-approval, sibling-principal, drafting-agent, sibling-agent and `required_count` probes driven through A14's real `POST /v1/approvals/{id}/decide`;
- the one-owner fallbacks;
- suspension and reinstatement round trips over members in every state;
- a two-tenant isolation suite over every table this node registers;
- billing, GST and overdue boundaries under the injectable clock;
- the public-profile rule that only published figures appear;
- the tipping-off payload scan;
- the D04 route table against the OpenAPI flags.

The builder never reads that folder, and the tests are published into `app/` only after the gate passes.

This node is API and data only; it draws no screen. The screens it feeds belong to other nodes:
- SPN-01, SPN-02, SPN-14 and SPN-15 are A62.
- ADM-03, ADM-33 and ADM-39 are A69.
- ADM-29 is A76, which reads A45's sponsor-performance table built partly from this node's snapshots and watchlist.
- PUB-05 (the sponsor profile and the Sponsor tab) is A54, which reads this node's public contract.
- ACC-05 and SPN-16 are A60, on top of A12.

Everything below is enforced in the API, never only in a screen.

This plan is not legal advice. Every sentence about what a licence, a regulator or counsel requires is a counsel question, recorded under the human gates. Every threshold is a row of data marked "proposed" with a default, not a rule in code. The thresholds are the 10 percent retained stake, the 6-month reapply period, the 30-day billing pause, two late distributions in 6 months, the sprawl count, the 30-day fee-notice period and the GST treatment.

## Build brief

**1. What this node reuses and what it adds**

- **Principals, sessions, tenant context and the permission matrix come from A11.**
  - `sponsor_user` HUMAN principals always carry a `tenant_id`.
  - One person who holds an investor account and a sponsor membership is two principals linked by `person_id`.
  - `withTenant` and RLS from A04 scope every read.
  - Staff "view as tenant" is A11's grant. This node only reads `acting_tenant_id` for audit.
  - Sponsor users are created by invitation and accept through A11's verify-email then passkey path. There is no password anywhere.
- **Approvals come from A14.** Every privileged act below is an `ApprovalRequest` with an action code, a policy row this node proposes (seeded from the product map's maker-checker table), and precheck hooks registered through `registerPrecheck`. Sponsor-side checker sets are supplied at submit, and re-checked at decide, through the two A14 ports named in section 4. Nothing in this module approves anything itself.
- **KYB comes from A30:**
  - the call is `kyb.openCase({ entity, directors[], owners[] })`;
  - there is one `kyc_case` per natural person (G01, G02) and per entity (G03);
  - every owner at 25 percent or more is verified;
  - the events `kyb.case.approved`, `kyb.case.info_requested` and `kyb.case.rejected` drive the pipeline.
  This node invents no identity check. The identity and sanctions adapters stay behind A22's sandboxes, reached only through A30.
- **Licences come from A21.** A04 created the one shared `licence` table and A21 shapes it. That split is frozen in A21's spec:
  - A21 owns the lifecycle `state` (`draft`, `active`, `suspended`, `expired`, `revoked`), the single expiry sweep `perimeter.expiry_sweep`, the T-90/30/7 alerts, `evaluateG21`, `initTenantFlags` and the summary read `GET /v1/licences/summary?entity=`.
  - This node inserts sponsor-held rows (kinds `sponsor_afsl`, `trustee_licence`, `authorised_rep`, already in A21's kind union) with lifecycle `state = active`. It owns and writes only their `verification_state` (`declared`, `verified`, `unverified`).
  - This node never runs its own expiry transition and never defines a second licence table or state list.
- **Agent access comes from A12.**
  - Sponsor tokens are tenant-scoped, and `agent.token.create_sponsor` is A14's 4E row.
  - The sponsor agent policy card (SPN-16) is A12's `PUT /v1/agent-access/tenant-policy` (see section 10).
  - This node publishes `tenancy.tenant.suspended`, `tenancy.user.suspended` and `tenancy.user.removed`. A12 subscribes to them for token revocation and A11 for session revocation.
- **Audit, alerts and clocks.** The audit write path is A10's `audit.record(tx, event)`, with `on_behalf_of_id` on every agent-made event. Alerts and clocks are A13's, reached through A14's transitive dependency.
- **Tables from A04.** They come from `tenancy.ts`, `entities.ts` and `fees.ts`:
  - `tenant` (a `PRIVILEGED_TABLES` member whose approval column is A04's `approval_id`);
  - `tenant_user`, `tenant_guardrail` (versioned; `kyc_required` cannot be false, by CHECK), `tenant_consent`;
  - `scheme` (P2), `vehicle` (`register_of_record` fixed to `unit_register`), `entity`;
  - `fee_schedule`, `fee_ledger`, `invoice`.
  This node reads and writes the shared `licence` table only as A21's ownership split allows.
- **Tables this node registers**, through A04's `registerModuleSchema`:
  - `sponsor_application` (the pipeline);
  - `sponsor_application_check`: licence, background, references, retained stake and fee agreement, one row per check with evidence document ids, the checker, and for the licence check the declared licence claim plus the linked `licence_id`;
  - `sponsor_application_info_request`;
  - `tenant_invitation`;
  - `tenant_approval_rule` (versioned);
  - `tenant_listing_pause`;
  - `sponsor_metric_snapshot`;
  - `sponsor_watchlist`;
  - `revenue_share_agreement`;
  - `invoice_payment`.
  Every registered table carries `tenant_id`, RLS and A03 enums exactly as a core table does, or `db:check` fails.
- **Time.** Time comes only from A09's injectable clock. Calendar days are `Australia/Sydney`, and business days use A13's calendar. `Date.now()` in the owned paths fails `npm run verify`.

**2. Admission pipeline (SPN-01 in, ADM-03 through, a tenant out)**

- **The application machine.**
  - `sponsor_application` has id `SPA-`. Its states form a small local machine, recorded in A03's machine format as a proposed data file (not one of the 14 product-map machines, and not an entry in A03's closed `extensions.ts` list).
  - Main path: `draft -> applied -> kyb -> background -> fee_agreement -> approved -> tenant_created`.
  - `info_requested` is a side exit from `kyb`, `background` or `fee_agreement`. It returns to the state it left once the applicant has answered every open request.
  - `declined` is terminal and reachable from any state after `applied`. A reason code is mandatory, and `reapply_after = declined_at + 6 months` by default (data marked proposed).
  - `withdrawn` is taken by the applicant before `approved`.
  - Any other pair is refused with `APPLICATION_ILLEGAL_TRANSITION`. Every transition writes exactly one audit event.
- **The applicant.**
  - The applicant is a HUMAN principal created through A11's ordinary signup. Until a tenant exists it holds the proposed role `SP-APP` (sponsor applicant), with self-scope cells over its own application only (constraint `own-records`), and it belongs to the platform tenant for RLS.
  - The role, its cells and their `delegable` values are an explicit diff to A03 and A11 (section 11). Until that diff lands, this node's gate cannot pass.
  - The applicant creates the application with `POST /v1/sponsor-applications` and edits it with `PATCH` while `draft`, capturing the five SPN-01 sections:
    - **Company:** legal name, ABN or ACN as `_enc` plus a blind index, and registered address.
    - **Licence:** a declared licence claim of kind `sponsor_afsl`, `trustee_licence` or `authorised_rep`, with number, holder entity and expiry. It covers either the applicant's own AFSL or authorisation for its trusts, or the licensed third-party trustee it will use. It is stored on the application's `licence` check row with `verification_state = declared`. No `licence` row exists yet.
    - **People:** directors, beneficial owners at 25 percent or more, and key persons with fit-and-proper declarations.
    - **Track record:** years operating, trusts managed, assets managed, and past trusts with planned versus actual. All of it is sponsor-declared and never rendered as audited.
    - **Commitments:** a retained stake of at least 10 percent of every trust, monthly property reporting, and the RealX sponsor terms version. Each is accepted as a `standing_consent` row with the terms document id, version and sha256, plus the applicant's fresh passkey assertion (A11 primitive, purpose `sponsor_terms`).
  - `POST .../submit` moves `draft -> applied` only when every section validates and every commitment is accepted.
  - The submit declaration and the commitment and fee-agreement acceptances are on A03's `delegable.ts` `"counsel"` list until counsel answers. An agent call is therefore refused with `SCOPE_PENDING_COUNSEL` through A12.
- **`applied -> kyb`.**
  - PART calls `POST /v1/admin/sponsor-applications/{id}/kyb/open`. This opens the A30 KYB case set for the sponsor entity, its trustee entity if different, every director and every owner at 25 percent or more, and records the case ids on the application.
  - The pipeline moves to `background` only when A30 reports every case `approved`.
  - A rejection or a true sanctions match anywhere moves it to `declined` with reason `kyb_failed` or `sanctions`. Per A30, the applicant is told nothing about the hit. This is the tipping-off guard: the applicant sees "we could not complete your application" and a support route, never the reason class. The event rule is in section 9.
- **`background`.** PART records the checks that need human judgement. Each is a `sponsor_application_check` row with evidence document ids and a result.
  - **Licence check.** On `pass`, this node inserts, or links to an existing, `licence` row in A21's shared table. The row carries the declared kind, number, holder entity, `authorisations[]` including `issue_wholesale_units` when the evidence shows it, `expires_on`, lifecycle `state = active` as A21 prescribes, and `verification_state = verified`. On `fail`, the linked row, if one exists, gets `verification_state = unverified` and the application moves to `declined`. The check row stores the `licence_id`. Whether the authorisation is legally sufficient is a counsel question, not a rule here.
  - **Fit-and-proper and background:** director bankruptcies, the banned-person register and adverse media, from the sanctions adapter's adverse-media scenario through A30.
  - **References.**
  - **Track-record review.**
  - **Retained-stake commitment:** at least the guardrail default, 10 percent proposed.
  - Every check must be `pass` or `pass_with_note` before `background -> fee_agreement`. A `fail` on the licence or background check moves the application to `declined`.
- **`fee_agreement`.**
  - PART attaches the draft `fee_schedule` (section 6) and the sponsor terms and fee agreement document.
  - The applicant accepts with a fresh assertion, recorded as a `standing_consent` row with purpose `fee_agreement`.
  - E-signature through A35's envelope is wired when A35 lands. Until then the record is the document sha256 plus the assertion.
  - Acceptance sets `fee_agreement_accepted_at`. The state stays `fee_agreement` until the approval below.
  - This first schedule takes effect through the `tenant.create` decision. Every later change is `tenant.fee_schedule.set` (section 6).
- **`-> approved`.**
  - This is `ApprovalRequest` action `sponsor.kyb.approve`, policy `4E`, KYCA maker, CCO checker (the product map's "Approve sponsor KYB: KYCA to CCO, before a tenant can be created").
  - The precheck hook refuses with `APPLICATION_KYB_INCOMPLETE` when any KYB case is not `approved`, or when a director or owner was added after the last case approval (a new person reopens KYB).
  - It refuses with `APPLICATION_FEE_AGREEMENT_MISSING` when the fee agreement is missing.
  - It refuses with `APPLICATION_LICENCE_UNVERIFIED` when the licence check is missing or failed.
  - It calls A21's `evaluateG21({ activity: 'issue_wholesale_units', actorEntityId: <the trustee entity>, tenantId: null })`. A21's wrapper writes one G21 `GateEvaluation` row, and the hook stores its id in `precheck_result`. A `fail` is `APPLICATION_LICENCE_NOT_ACTIVE`, with A21's reason codes, which name the entity and the licence id.
  - The request's canonical payload is the application snapshot hash, so an edit after submission invalidates the checker's assertion (A14's `APR_CHALLENGE_MISMATCH`).
- **`-> tenant_created`.** This is `ApprovalRequest` action `tenant.create`, policy `4E`, PART or SUPER maker, CCO checker (the product map's "Create sponsor tenant: KYC locked on in production").
  - Its precheck repeats the `evaluateG21` call above and stores the new `GateEvaluation` id. A licence that expired, was suspended or was revoked between the two approvals refuses here.
  - Its effector, registered with A14's `registerEffector`, runs in one transaction:
    1. Insert `tenant`: `TEN-` id, `legal_name`, `short_name`, `slug`, an `abn` reference to the sponsor `entity`, `issuing_licence_id` (the A21 row verified above), `kyb_status = approved`, `plan`, `status = onboarding`, A04's `approval_id` (the `tenant.create` request) and `admitted_on`.
    2. Insert `tenant_guardrail` version 1 from the platform defaults (section 3).
    3. Copy the `fee_schedule` from the application.
    4. Create the sponsor `entity` rows as tenant rows (section 5).
    5. Call A21's `initTenantFlags({ tenantId, issuingLicenceId, approvalId })`. A21's spec flags this as needing the graph edge A21 -> A33.
    6. Create the first `tenant_user` as SP-OWN for the applicant, as a new `sponsor_user` principal linked by `person_id`. The applicant principal keeps read access to its own application and closes 30 days later.
    7. Issue the owner's invitation.
    8. Write `tenancy.tenant.created`.
  - **Database guard.** A04's `PRIVILEGED_TABLES` CHECK makes `tenant.approval_id` non-null. On its own that would accept any approved request id. This node's migration therefore adds a deferred constraint trigger on `tenant` (`INITIALLY DEFERRED`, so the effector's own transaction passes), which refuses the insert unless all three hold:
    - the referenced `approval_request` has `action_code = 'tenant.create'`;
    - it is in an SM-00 state at or after `approved` and not `rejected`, `expired`, `cancelled` or `failed`;
    - its canonical payload's `application_id` names a `sponsor_application` whose `sponsor.kyb.approve` request is `confirmed` or `closed`.
    A tenant without an approved KYB is then impossible by construction, which is J11's oracle hook. A hidden SQL test inserts a tenant carrying a foreign approved id (for example a `flag.toggle` request) and expects the refusal.
- **`onboarding -> active`.**
  - This is a system transition, audited, when all four hold:
    - the first SP-OWN has accepted the invitation and registered a login passkey;
    - guardrails v1 exist;
    - the fee schedule is attached;
    - the issuing licence is A21 `state = active` with `verification_state = verified` at the clock's instant.
  - It needs no second approval, because the CCO's `tenant.create` decision covered it.
  - `GET /v1/sponsor/tenant` returns `onboarding`, with the missing items, until then, so SPN-02 can show a checklist.
  - A hidden test ages the licence to expiry between the two approvals: the tenant never reaches `tenant_created`. Another suspends it after `tenant_created`: the tenant never reaches `active`.
- **Reapplication.** `POST /v1/sponsor-applications` by a person or entity (a blind-index match on ABN or ACN) with a `declined` application whose `reapply_after` has not passed is refused with `APPLICATION_REAPPLY_TOO_SOON`, naming the date. After that date, a new application starts at `draft` and links to the old one for the reviewer. A second open application for the same person or ABN is refused with `APPLICATION_ALREADY_OPEN`.
- **Sprawl signal.** When a tenant's count of vehicles exceeds the guardrail `max_vehicles` (default 20, proposed; the RealT lesson), the module raises A13 alert `sponsor.sprawl` Sev3 to PART on the vehicle creation that crosses the line. It is a signal, not a block, and it is raised once per crossing.

**3. Tenant record and guardrails (M27, ADM-33)**

- **The `tenant.status` machine (local).**
  - `onboarding -> active -> suspended -> active` (reinstated).
  - `active | suspended -> offboarded` (terminal). This is `ApprovalRequest` `tenant.offboard`, `4E`, PART maker, CCO checker. Its precheck refuses with `TENANT_HAS_LIVE_VEHICLES` while any vehicle is in a state other than `closed`.
  - Illegal pairs are refused with `TENANT_ILLEGAL_TRANSITION`.
- **`tenant_guardrail` is versioned and immutable.** Each version carries `version`, `effective_from`, `approval_id` and `supersedes_id`, plus these fields:
  - `kyc_required`: always true. The API refuses any payload that sets it false with `GUARDRAIL_LOCKED` before the database CHECK ever sees it.
  - `whitelisting_mode`: `auto_on_approved_kyc` (default) or `admin_confirm`.
  - `eligibility_packs[]`: the rule-pack codes the tenant's offerings may use; default the current `AU-W` pack. `AU-R` packs appear only when A21's `offers.retail` flag is on, checked at listing time by A36.
  - `holder_cap_ceiling_bp`: default 500. A listing's holder cap may be at most this. The platform maximum `holder_cap_platform_max_bp` in `packages/config` bounds the ceiling itself; a version above it is refused with `GUARDRAIL_ABOVE_PLATFORM_MAX`.
  - `fee_cap_investor_bp`: default 50, the design's 0.5 percent transaction fee.
  - `fee_cap_sponsor_mgmt_bp_pa`: default 90, the design's 0.5 to 0.9 percent band.
  - `allowed_jurisdictions[]`: Australian states and territories.
  - `rails[]`: `npp_payid` on; `payto` off until P3.
  - `max_vehicles`.
  - `retained_stake_min_bp`: default 1,000.
  - `branding`: display name, short name, logo document id.
  - `listing_visibility_mode`: mirrored from A21's platform setting and never wider than it.
- **Changing guardrails.** This is `ApprovalRequest` action `tenant.guardrails.set`, policy `4E`, CCO maker, LEGL checker. The effector inserts version n+1 and closes version n, which stays readable with its approval id.
  - A sponsor principal has no `E` cell on M27. `PUT /v1/admin/tenants/{id}/guardrails` from any sponsor user or sponsor agent is therefore `403 FORBIDDEN` from A11's matrix before this module runs. The route is also `requiresHuman` for staff (section 10).
  - `GET /v1/sponsor/guardrails` is read-only. It returns the current version with plain-English labels, for example "KYC required · Locked on", the wording ADM-33's mock uses.
- **Refusal before any approval exists.** `GUARDRAIL_LOCKED` and `GUARDRAIL_ABOVE_PLATFORM_MAX` are raised on this node's route before any `ApprovalRequest` is created. The same validators are registered as A14 precheck hooks on `tenant.guardrails.set`. A draft opened through A14's generic `POST /v1/approvals` can therefore exist, but its submit ends `rejected` with the same code, and it can never be approved. If A14 later offers a draft-validator port (section 11, spec point 3), the generic draft is refused as well.
- **Tenant-level pauses.**
  - `tenant_listing_pause` rows carry `reason` in {`billing_overdue`, `attestation_overdue`, `watchlist_hold`, `licence_expired`, `suspended`}, plus `detail` (for `licence_expired`, the A21 licence id and its state: `expired`, `suspended` or `revoked`), `set_by` (principal, job or event), `set_approval_id` (for `watchlist_hold`), `set_at`, `cleared_at` and `cleared_approval_id`.
  - `canSubmitListing(tenantId)` from `index.ts` returns `{ allowed, reasons[] }`. A36 (offerings) and A63 (the sponsor wizard) call it. A refusal is `TENANT_LISTINGS_PAUSED`, naming every open reason.
  - The billing reason is evaluated at read time as well as from the stored row (section 6).
  - Who sets and clears each reason:
    - `billing_overdue`: this node (section 6).
    - `licence_expired`: this node (section 5).
    - `suspended`: this node (section 8).
    - `attestation_overdue`: A34 (proof of property overdue 10 days or more, proposed), through `setListingPause` and `clearListingPause` on `index.ts`.
    - `watchlist_hold`: set by `ApprovalRequest` `tenant.listing_pause.set` (`4E`, PART maker, CCO checker) and cleared only by `tenant.listing_pause.clear` (`4E`, PART maker, CCO checker).
  - Pauses stop new listings only. They never touch approved distributions, live offers or investors' holdings.

**4. Sponsor users, invitations and internal approval rules (SPN-14)**

- **`tenant_user`.** Columns:
  - `principal_id`, `tenant_id`, `person_id`;
  - `role` in {`SP-OWN`, `SP-OPS`, `SP-FIN`, `SP-IR`}, plus A98's `SP-VAL` and `SP-PM` as enum values. There is one membership per person per tenant (`UNIQUE (tenant_id, person_id)`). A person who needs two roles gets the higher one. The design's "Can approve" column is derived from the role and the current rules, not stored.
  - `state` in {`invited`, `pending_identity`, `active`, `suspended`, `removed`};
  - `state_before_suspension` and `suspended_by_approval_id`, both set only by a tenant suspension (section 8);
  - `suspended_reason` (`individual` or `tenant`);
  - `identity_case_id` (A30), `invited_by`, `joined_at`.
  A11's 30-day dormancy applies to SP-OWN and SP-FIN.
- **Invitation.**
  - `POST /v1/sponsor/team/invitations` by an active SP-OWN, with an email and a role from the caller's `invitable_roles[]`, creates a `tenant_invitation`. The token is single-use (32 bytes) with a 7-day expiry and at most 3 resends; after expiry the invitation is refused with `INVITATION_EXPIRED`.
  - The invitation is sent through the email sandbox.
  - The invitee accepts through A11's verify-email then passkey path. That creates the `sponsor_user` principal and moves the membership to `pending_identity`.
  - A30 opens an individual identity case (G01, G02). The membership becomes `active` on `kyc.case.approved`.
  - Until `active`, the principal's effective cells are empty except reading its own membership. Anything else is refused with `TENANT_USER_NOT_VERIFIED`.
  - A98's valuer and property-manager members join through the same path. `invitable_roles[]` includes SP-VAL and SP-PM only as A98's `validatePartnerInvitation` allows.
- **Role changes, suspension, reinstatement and removal.** These are routes on `/v1/sponsor/team/{membershipId}` used by SP-OWN (section 10).
  - **Granting SP-OWN** is `ApprovalRequest` `tenant.user.grant_owner`, `4E`, SP-OWN maker. The checker is a different SP-OWN of the same tenant, or RealX PART when `tenant_owner_count = 1` (the same resolution rule A12 and A14 use for sponsor tokens).
  - **Other changes** are SP-OWN acts, audited, that need no second person: granting SP-OPS, SP-FIN or SP-IR; demoting; suspending (`suspended_reason = individual`); reinstating an individually suspended member (back to `active` only when the identity case is still `approved`, else to `pending_identity`); and removing.
  - The last active SP-OWN can never be demoted, suspended or removed (`TENANT_LAST_OWNER`).
  - Removal or suspension publishes `tenancy.user.removed` or `tenancy.user.suspended`. A11 turns these into session revocation, and A12 into token revocation, for that principal in the same event handling.
  - An agent can only draft these acts, and its human must confirm them (section 10).
- **Internal approval rules.**
  - `tenant_approval_rule` holds versioned rows per `action_code`, with `required_roles[]` and `required_count`. The action codes are `listing.submit`, `distribution.propose`, `valuation.submit`, `attestation.submit`, `announcement.publish`, `agent.token.create_sponsor` and `agent.tenant_policy.loosen`.
  - Platform minimums are data in `packages/domain`, in `sponsor-minimums.ts` under the key `sponsor.min_approvers`. The domain node's owner adds the file from this spec, and A36's refusal text cites the key.
    - `listing.submit` needs one SP-OWN (or a holder of `listing:approve`) who is not the submitter.
    - `distribution.propose` needs two people at the sponsor: SP-FIN prepares and SP-OWN approves, because RealX's FINA and CFO are the two on the RealX side.
    - `agent.token.create_sponsor` and `agent.tenant_policy.loosen` need an SP-OWN who is not the requester.
  - A rule version below a minimum is refused with `APPROVAL_RULE_BELOW_MINIMUM`, naming the minimum, whoever asks. It is refused on this node's route before any approval request exists, and as a precheck at submit on A14's generic path.
  - A version that lowers `required_count` or removes a role while staying at or above the minimum is `ApprovalRequest` `tenant.approval_rule.change`, `4E`, SP-OWN maker, RealX CCO checker (the product map's "needs RealX approval if it weakens").
  - A version that only tightens is `ApprovalRequest` `tenant.approval_rule.tighten`, `4E`, SP-OWN maker, a second SP-OWN checker, or RealX PART when `tenant_owner_count = 1`.
  - In-flight requests keep the rule version frozen into their `precheck_result` at submit.
- **The checker rule.** `evaluateSponsorApproval({ tenantId, actionCode, ruleVersion, submitter, makerIdentitySet, approver })` returns pass or a reason code. `eligibleSponsorCheckers(tenantId, actionCode, submitter)` lists who may approve. The rules, in order:
  1. The approver is a HUMAN. An AGNT, an AGTU or a token is refused with `SPONSOR_APPROVER_MUST_BE_HUMAN` before any other check; A14's `APR_CHECKER_MUST_BE_HUMAN` and A12's `G22_AGENT_NEVER_CHECKER` catch the same at decision time.
  2. The approver's membership is `active` and holds a required role.
  3. The approver is not the submitter, and not any principal sharing a `person_id` with any member of A14's maker identity set. This covers the investor principal, a partner principal and a membership in another tenant. The approver is also not the human behind any agent that drafted, edited or submitted, and no agent of the approver's own person drafted it. Any of these is refused with `SPONSOR_SELF_APPROVAL`.
  4. `required_count` distinct approvers must be reached before the request may leave the sponsor.
- **Where the rule is enforced on A14's real path.** This reopens A14 as an upstream spec point (section 11, points 1 and 2). A precheck hook alone cannot do it: A14 runs prechecks at submit, before any approver exists, and its decide step checks only `required_checkers[]`.
  - **At submit:** A14 calls the `CheckerSetResolver` this node registers through `registerCheckerSetResolver(actionCode, resolver)` for every tenant-scoped action code, after A11's `eligibleCheckers`. The resolver returns `required_count` sponsor checker sets, each holding the eligible sponsor humans under the rule above, and A14 appends them to `required_checkers[]`. The platform sets stay: RealX's CCO remains a required checker on every tenant offering (A11 SoD rule 9, A14 SoD #15), whatever the sponsor configures.
  - **At decide:** A14 calls the `DecisionGuard` registered through `registerDecisionGuard(actionCode, guard)` before it records the decision. The guard re-runs `evaluateSponsorApproval` with the decider and the frozen rule version, so a person who changed hats after submit, or an agent's human, is refused with `SPONSOR_SELF_APPROVAL` even when A11's `makerIdentitySet` would not catch it.
  - **Too few eligible humans.** When fewer distinct eligible humans exist than `required_count`, the resolver fails the submit:
    - `listing.submit` fails with A36's `OFR_SECOND_HUMAN_REQUIRED`, with `detail.minimum = sponsor.min_approvers.listing.submit`. This is the case of a tenant whose only SP-OWN is also the submitter, and there is no RealX fallback, as A36 specifies.
    - `distribution.propose`, `valuation.submit`, `attestation.submit` and `announcement.publish` fail with `SPONSOR_SECOND_HUMAN_REQUIRED`.
    - `tenant.user.grant_owner`, `tenant.approval_rule.tighten`, `agent.token.create_sponsor` and `agent.tenant_policy.loosen` do not fail. Where the A14 policy row carries `tenant_owner_count = 1`, the resolver returns the RealX PART set (SECO or PART for tokens, as A14's row says) in place of the second SP-OWN.
  - **Gate dependency.** Until A14 exposes the two ports, this node's gate fails on the pass criterion that drives these cases through `POST /v1/approvals/{id}/decide`. That is escalated as an upstream spec defect, never patched locally.

**5. Entities, licences and vehicles (M02, ADM-39)**

- **`entity`** (`ENT-` id). Columns:
  - `entity_type` in {`sponsor_company`, `trustee`, `custodian`, `responsible_entity`, `licensee`, `valuer_firm`, `property_manager_firm`, `audit_firm`, `regulator`}. `audit_firm` and `regulator` are added for A98's TP-AUD and TP-OBS.
  - `trustee_relationship` (for `trustee` only): `sponsor_group` (the sponsor's own trustee company) or `third_party_professional` (a hired professional trustee).
  - `legal_name`, `abn_enc` and `abn_bidx`, `acn_enc`, registered address `_enc`.
  - `kyb_required` (false only for `regulator`), `kyb_case_id`, `kyb_status`.
  - `panel_valuer` (boolean, `valuer_firm` only), `valuer_registration_number`, `valuer_registration_expires_on`, `panel_approval_id`.
  - `tenant_id`: the sponsor, for its own companies; the platform tenant, for shared entities such as a custodian, a professional trustee, the hired RE (P2), the licensee, a valuer firm, an audit firm or a regulator. A shared entity is readable across tenants only through A04's `is_public` reference policy, on the non-PII columns.
  - Creation is by PART or by the admission effector. Edits to the legal name or ABN reopen KYB through A30 (the `entities.entity.reverification_required` event).
  - Setting `panel_valuer = true` is `ApprovalRequest` `entities.panel_valuer.set`, `4E`, PART maker, CCO checker. Its precheck needs a current registration (A34 reads the flag, and `VAL_NOT_PANEL` is A34's code).
  - A registration passing its expiry sets `panel_valuer_current = false` on read. The flag stays set, but A34 treats the valuer as not current. What "panel valuer" must mean is A34's counsel question.
- **Sponsor licences.** They are rows in A21's shared `licence` table. This node does not define its own licence schema, states or sweep.
  - This node inserts sponsor rows at the licence check (section 2), links them from `sponsor_application_check.licence_id` and from `tenant.issuing_licence_id`, and writes only `verification_state`.
  - A21 owns every lifecycle transition, including expiry through its single `perimeter.expiry_sweep`.
  - This node subscribes to A21's `licence.expired`, `licence.suspended` and `licence.revoked` events for any licence that is a tenant's `issuing_licence_id` or a vehicle's trustee licence. On each it sets a `tenant_listing_pause` with reason `licence_expired` (detail: the licence id and the A21 state) and raises Sev2 `sponsor.licence_inactive` to CCO. When A21 reports `licence.reinstated`, or a newly verified active licence replaces it, the pause clears exactly once.
  - Whether existing vehicles may keep operating under an inactive sponsor licence is a counsel question logged under HG-LEGAL. This node does nothing to them.
  - Renewals: SP-OWN may upload a renewal document through A21's route, which creates A21's `draft` version for CCO review. Re-verification of a renewed row is a PART licence check on the tenant's record.
- **`vehicle`** (`VEH-` id). Columns:
  - `vehicle_type` in {`trust`, `sub_fund`}, `tenant_id`, `name`, `abn_enc`;
  - `trustee_entity_id`: required (`VEHICLE_TRUSTEE_REQUIRED`);
  - `custodian_entity_id`;
  - `issuing_licence_id`: the A21 licence under which units are issued;
  - `scheme_id`: P2 only. A `sub_fund` while `PHASE=P1` is refused with A08's `PHASE_NOT_ENABLED`;
  - `bank_account_ref`: the client-money trust account reference, filled by the treasury node, null until then;
  - `tax_class` (`amit`, `mit`, `other`), `deed_document_id`;
  - `register_of_record = unit_register` (A04 CHECK), `chain_is_register_clause` null.
  - `state` is a small local machine, `proposed -> formed -> active -> closing -> closed`:
    - `formed` needs a trustee, a custodian and a deed, plus a passing `evaluateG21({ activity: 'issue_wholesale_units', actorEntityId: trustee_entity_id, tenantId, vehicleId })`, which requires the issuing licence to be A21 `active` and `verified`. The G21 `GateEvaluation` id is stored on the vehicle.
    - `active` is set when the first offering of the vehicle finalises. This node consumes A38's `settlement.run.completed` event (carrying `vehicle_id` and `offering_id`) and calls its own idempotent `vehicleSetActive`. A38 must publish that event on the outbox (section 11).
    - `closing` and `closed` are driven by A48 (wind-down) through `vehicleSetState` and the register close.
  - A vehicle of a `suspended` tenant keeps its state; investors' holdings are unaffected.
  - One property per vehicle is A34's rule. This node exposes `vehicle.property_id` as a nullable slot that A34 fills once.
- **Read and create routes** are listed in section 10. All are cursor-paginated through A09's `listRoute`, and PII columns render through A04's `mask()` only.

**6. Billing (SPN-15, M27)**

- **`fee_schedule`** is per tenant and versioned, with `effective_from`. Lines:
  - `listing_fee_cents`, charged per listing on go-live;
  - `platform_fee_per_investor_month_cents`, per distinct active holder across the tenant's vehicles at month end (the product map's model);
  - `distribution_run_fee_cents`;
  - `trust_fee_bp_pa`: the design's "RealX platform fee 0.25 percent p.a. charged to the trust from net assets".
  - Each line carries `gst_treatment` in {`taxable`, `gst_free`, `input_taxed`, `out_of_scope`}, as data. The default is `taxable` (proposed).
  - Both fee bases exist as data. The product map's per-investor basis is on and the trust basis is off by default, because the two documents disagree and the founder has not chosen (design-analysis §6; recorded below as a founder question).
  - Changing a schedule is `ApprovalRequest` `tenant.fee_schedule.set`, `4E`, CFO maker, CEO checker (the product map's "Change fee schedule · CFO → CEO · Notice period", G13, G17). Its precheck refuses `effective_from < now + billing.fee_notice_days` (default 30, proposed, N17) with `FEE_SCHEDULE_NOTICE_TOO_SHORT`, and publishes `billing.fee_schedule.changed` for A43's `fee.change` notice. Whether PART should draft instead is a founder question.
- **The accrual job `billing.month_end`** runs on the first Sydney calendar day of each month. It is idempotent and replayable. It computes the previous month's `fee_ledger` lines per tenant from three inputs:
  - listing fees, from A36's `offering.live` events;
  - active holders at month end, from the `BillingInputProvider.activeHoldersAsOf(tenantId, monthEnd)` port;
  - distribution runs paid, from A40's `distribution.run.state_changed` events reaching the paid state.
  In this node's tests the three inputs come from registered fixtures. The real implementations are registered from their owners' paths: holders by A47, from A18's cap table (`GET /v1/register/vehicles/{vehicleId}/cap-table?as_of=`, distinct `holder_ref` across the tenant's vehicles), and runs by A40 (section 11).
- **Invoices.** The job issues one `invoice` per tenant and month:
  - `INVC-` id, with a per-tenant display number (`INV-0931` in the product map's mock). A02's `INV-` id prefix is reserved for investors, so it is not used for ids here.
  - `issued_on`, and `due_on = issued_on + 14 days`.
  - Line items carrying `amount_ex_gst_cents`, `gst_treatment` and `gst_cents`.
  - `subtotal_ex_gst_cents`, `gst_cents` and `total_cents`. GST is computed per invoice on the sum of `taxable` lines at `gst.rate_bp` (1,000, data) and rounded half-up to the cent. The per-invoice rounding rule is proposed and is an accountant question; per-line GST is shown as the allocated share.
  - Tax-invoice fields: the words "Tax invoice", the supplier legal name and ABN from `packages/config` `billing.supplier` (sandbox placeholders locally), the recipient's legal name and ABN (the tenant's own, unmasked on its own invoice), the issue date, and each line's description, GST treatment and GST amount.
  - All amounts are integer cents from A02's money math.
- **The invoice machine:**
  - `draft -> issued -> paid`;
  - `issued -> overdue`, by the job below or on read;
  - `overdue -> paid`;
  - `issued | overdue -> void`, which is `ApprovalRequest` `invoice.void`, `4E`, FINA maker, CFO checker;
  - anything else is refused with `INVOICE_ILLEGAL_TRANSITION`.
- **When an invoice becomes overdue.** An invoice becomes `overdue` at `pause_on`, which is 00:00:00 `Australia/Sydney` on the day after `due_on + billing.pause_after_days` (default 30, proposed). `INV-1002`, due 15 Oct, has `pause_on` at 00:00 on 15 Nov. Two paths apply it:
  - the job `billing.overdue_sweep`, daily at 00:00 `Australia/Sydney`, idempotent;
  - the read-time rule: `canSubmitListing` and every billing read treat an unpaid `issued` invoice past `pause_on` as overdue even if the sweep has not yet run.
  Both are tested.
- **Payment.** Sponsors pay by bank transfer into RealX's operating account, never a client-money account.
  - The payment instructions come from `packages/config` `billing.operating_account`. At config load, a check refuses any account reference that appears in A16's client-money account registry.
  - FINA records a payment with `POST /v1/admin/billing/invoices/{id}/record-payment`, giving the bank reference, amount and received date. That inserts an `invoice_payment` row and is audited.
  - When the sum of payments equals `total_cents`, the invoice moves to `paid`. A partial payment sets `paid_cents` and `partially_paid = true` without changing the state, and does not lift a pause.
  - Payments are matched from the operating-account bank feed through the same recording function once A17 lands.
- **The pause.**
  - When an invoice becomes `overdue`, the sweep or the first read that observes it (whichever commits first; a unique partial index makes it exactly once):
    - sets `tenant_listing_pause` with reason `billing_overdue`;
    - raises A13 alert `sponsor.billing_overdue` Sev3 to PART;
    - publishes `billing.invoice.overdue` (A43 turns it into the SP-OWN and SP-FIN notice).
  - When every overdue invoice of the tenant is `paid` or `void`, the pause clears in the same transaction that recorded the last payment or void, exactly once, and `billing.invoice.paid` is published.
  - The boundary is Sydney midnight from the injectable clock.
- **Sponsor reads.** `GET /v1/sponsor/billing`, the invoice detail and the payment-instructions read return the SPN-15 shape, including `pause_on` for every unpaid invoice (section 10). Sponsor users see their own tenant only. SP-OPS, SP-IR, SP-VAL and SP-PM have no `V` on billing (matrix).

**7. Sponsor performance, watchlist and the public profile (M24, ADM-29, PUB-05)**

- **`sponsor_metric_snapshot`** is taken per tenant per month by job `sponsor.metrics` (idempotent), and on demand for staff. Fields:
  - `vehicles_count`;
  - `capital_raised_cents`, only from finalised allocations and never from open offers or escrow (N16);
  - `yield_planned_bp` and `yield_paid_bp`, per vehicle and weighted;
  - `distributions_on_time_pct`: a run is late when paid more than 10 business days after the record date (A13's calendar);
  - `late_distributions_6m`, `distribution_missed` (a scheduled run not paid by its date);
  - `proof_of_property_score`: attestations submitted on time over attestations due, from A34;
  - `retained_stake_bp` per vehicle: the sponsor's own register holding over units, against `retained_stake_min_bp`;
  - `complaints_open`, `incidents`;
  - `per_vehicle[]`.
  Inputs arrive through registered `MetricProvider` interfaces, so that A34, A40, A47 (register holdings) and A44 (complaints) plug in without this node importing them. Fixtures stand in until they land. A missing provider shows on the snapshot as `source: missing`, never as zero.
  - Snapshots are internal, staff-only records. They are inputs to A45's governed metrics, and A45's `GET /v1/reporting/sponsor-performance` composes the ADM-29 table from them and from the watchlist. `sponsorSnapshotSeries(tenantId, range)` is exported for A45.
  - A snapshot is never a public figure.
- **Watchlist.**
  - A `sponsor_watchlist` row opens automatically when `late_distributions_6m >= 2` (proposed, the product map's rule), or when `retained_stake_bp < retained_stake_min_bp` for any active vehicle. The row carries `reason` and `opened_by = job`, and the A13 alert `sponsor.watchlist` Sev3 goes to PART.
  - PART or CCO may open or close a row by hand, with a reason; this is audited and needs no second person.
  - An agent may only draft a hand open or close, which its human must confirm (section 10).
  - A `watchlist_hold` listing pause is the separate `4E` in section 3.
  - A watchlist row alone blocks nothing. It is a signal on ADM-29 and, as a `flag`, on the sponsor's own scorecard.
- **Revenue share.**
  - `revenue_share_agreement` is per tenant, with `basis` in {`flat`, `pct_of_platform_fee`, `transaction_based`}, a rate in basis points or cents, `effective_from`, `effective_to`, `document_id`, and `legal_review_required = (basis = transaction_based)`.
  - Setting or changing one is `ApprovalRequest` `tenant.revenue_share.set`, `6E`, PART maker, CFO and CCO checkers. The policy row adds LEGL as a third checker when `legal_review_required` (the product map's "legal review if transaction-based"). A transaction-based agreement submitted without a LEGL checker set resolved is refused with `REVENUE_SHARE_LEGAL_REVIEW_REQUIRED`.
  - Revenue-share rows are staff-only; there is no sponsor `V` cell.
  - Whether transaction-based remuneration to a sponsor is permitted under the conflicted-remuneration rules is a counsel question, recorded below.
- **Public contract.** `GET /v1/public/sponsors` and `GET /v1/public/sponsors/{slug}` are routes marked `public`, with no principal. They serve three kinds of field, and the response labels each one.
  - **Facts of record,** served live:
    - `name`, `short_name`, `slug`, `status`;
    - `admitted_on`: the badge "Vetted by RealX · admitted Mar 2026" is `admitted_on` formatted `MMM yyyy`;
    - the fact `Trusts on RealX`: the count of this node's `active` vehicles;
    - the fact `Licence`: the string from A21's `GET /v1/licences/summary?entity=<the issuing trustee>`.
    The `Licence` string is `Own AFSL` when the trustee entity is `trustee_relationship = sponsor_group` holding a `sponsor_afsl` row. It is `Trustee: hired RE` when the trustee entity is `third_party_professional` (P1), or a `responsible_entity` (P2). Both are the design's strings. A21's summary must map these entity facts (section 11). Whether "RE" is accurate wording for a P1 professional trustee is a G17 claims question for A43's review and the founder, and the design string is kept until they answer.
  - **Published content,** served only from the latest approved publication:
    - `about`;
    - the sponsor-declared facts `Trusts managed`, `Assets managed` and `years_operating`, each with `source: sponsor_declared`;
    - the performance lines: `capital_raised_cents`, `distributions_on_time_pct`, `proof_of_property_score`, `retained_stake_bp`, `incidents`, `complaints_resolved`;
    - `trusts[]`, one row per trust with planned versus actual yield, forecast versus actual growth, and the retained stake.
    The publication is A45's `metric_publication`, created only through A45's `metrics.publish` (IRCM maker, CFO and CCO checkers, `6E`, with the G17 claims check as precheck). This is the product map's "Publish public KPIs and track record · Claims reviewed". It is one act, so this node defines no second publication table or action code. This node reads the publication through the `PublishedSponsorProfileProvider` port that A45 registers (section 11), and returns `as_of`, `publication_id` and `approval_id` with the published block. Until a publication exists, or while the provider is not registered, the block is `{ source: 'missing' }` and the public page shows "figures not yet published", never a snapshot value.
  - **Live flags,** the one stated exception, because PUB-05 requires that "a sponsor with an overdue proof item or a missed distribution shows it here":
    - `proof_overdue`: true while this node holds an open `attestation_overdue` pause for the tenant;
    - `distribution_missed`: true while the latest A40 provider value says a scheduled run is unpaid past its date.
    Both are booleans with `as_of`. No figure travels with them.
  - A `suspended` or `offboarded` tenant returns `status`, so the public page can show a neutral "not currently listing" state. A45's precheck already refuses publishing figures for a suspended tenant.
  - No user, invoice, guardrail, revenue-share, watchlist or snapshot field ever appears in a public response (schema test).
  - The string "tokenised value" appears nowhere in this module (N16 grep).
  - A hidden test creates a snapshot with values that differ from the latest publication. The public response must carry only the published values, and a snapshot with no publication must never appear.

**8. Suspension and reinstatement**

- **Suspending.** `POST /v1/admin/tenants/{id}/suspend` creates `ApprovalRequest` `tenant.suspend`, policy `SGL+R`, `protective = true`. The CCO acts with a hardware-key assertion and a reason code. Ratification by the CEO, SECO or LEGL is due within 2 hours; the ratifier roles are data on the policy row. The effector runs in the acting transaction and does the following:
  - `tenant.status -> suspended`.
  - For every `tenant_user` membership whose state is not `removed`: it records `state_before_suspension = <current state>` and `suspended_by_approval_id = <this approval>`, sets `suspended_reason = tenant`, and moves the state to `suspended`. This covers `invited`, `pending_identity`, `active` and already individually `suspended` members, and SP-VAL and SP-PM memberships.
  - Every open `tenant_invitation` is cancelled.
  - A `suspended` listing pause is set.
  - Pending sponsor-side approval requests of the tenant are cancelled through A14 (reason `tenant.suspended`).
  - `tenancy.tenant.suspended` is published, carrying `tenant_id`, `approval_id` and every affected `principal_id`.
  - A13 alert `sponsor.suspended` Sev2 goes to the CEO.
  - One audit event is written.
- **The consumers of `tenancy.tenant.suspended`, each named and each tested:**
  - A11 revokes every sponsor session.
  - A12 revokes every sponsor token and mandate.
  - A101 engages the tenant kill-switch tier.
  - A98 ends every SP-VAL and SP-PM engagement in the tenant.
  - A46 disables every webhook subscription of the tenant.
  - The owner of sponsor integration API keys (ADM-32) revokes them. No frozen spec owns those keys yet (section 11).
  - A36 pauses the tenant's offerings with `state_reason = tenant_suspended` and refuses new submissions, as A36 specifies.
  This node's contract test asserts that the A11 and A12 revocations land before the outbox event is acknowledged. Using registered fixture consumers, it also asserts that the event carries everything the other five need. Each consumer proves its own handler at its own gate.
- **What suspension does not touch.** This node does not touch approved distribution runs, investors' holdings or vehicles, because the sponsor's suspension must not become the investors' loss. Whether and how investors must be told is a counsel question. The node exposes `tenant.status`, the public contract shows the neutral state, and A43 owns any notice.
- **Irreversible by timer.** The row is `reversible = false`. A lapsed ratification raises A14's Sev2 `approval.ratification_missed`, and the tenant stays suspended until a human reinstates it. A suspension is never undone by a timer.
- **Reinstating.** `POST /v1/admin/tenants/{id}/reinstate` is `ApprovalRequest` `tenant.reinstate`, `4E`, CCO maker, CEO checker. It is restorative, so `SGL+R` is refused by A14's `APR_SGLR_NOT_PROTECTIVE`. The effector:
  - restores exactly `state_before_suspension`, and only on rows whose `suspended_by_approval_id` equals the suspension being reversed. It then clears both columns and sets `suspended_reason` back to `individual` on rows that were individually suspended before;
  - never moves an `invited` or `pending_identity` member to `active`. Such a member still needs its A30 identity case approved (G01), and cancelled invitations stay cancelled, so the owner re-invites;
  - leaves a member that SP-OWN had suspended for cause before the tenant suspension `suspended`;
  - requires active members to re-authenticate. Tokens are not restored: people issue new ones with fresh assertions;
  - clears the `suspended` pause;
  - publishes `tenancy.tenant.reinstated`.
  A101's tenant tier is released only through its own staff `4E`.
  Hidden tests take a `pending_identity` member, an `invited` member and an individually suspended member through a suspend and reinstate cycle and expect each back in exactly its prior state.

**9. Interfaces, events, errors, empty states**

- **`index.ts` exports:**
  - `canSubmitListing`, `setListingPause`, `clearListingPause`;
  - `evaluateSponsorApproval`, `eligibleSponsorCheckers`, and the registered `CheckerSetResolver` and `DecisionGuard`;
  - `tenantSummary(tenantId)`: the sidebar card, with display name, the caller's display name and role label, and status;
  - `registerMetricProvider`, `registerBillingInputProvider`, `registerDashboardTileProvider` (A36, A40, A43 and A47 register SPN-02 tiles), `registerTeamRowsProvider` (A98 registers `teamPartnerRows`) and `registerPublishedSponsorProfileProvider` (A45);
  - `sponsorSnapshotSeries`, `licencesForEntity` (a thin read over A21's rows plus this node's `verification_state`);
  - `vehicleSetActive(vehicleId, runId)`, which is idempotent and also driven by A38's event, and `vehicleSetState` (for A48);
  - the seed hook that A06 calls to create:
    - the design's three sponsors: Harbour & Co Property, admitted Mar 2026, whose own trustee company (`sponsor_group`) holds a `sponsor_afsl`; Northline Capital, Apr 2026, with the same structure; and Yarra Bend Partners, Jun 2026, whose trustee is a `trustee` entity with `trustee_relationship = third_party_professional` holding a `trustee_licence` row with `issue_wholesale_units`;
    - their sponsor licence rows in A21's table, all `active` and `verified`. A21 seeds no sponsor row;
    - their users: Mia Chen SP-OWN and Leo P. SP-FIN at Harbour & Co; at least one SP-OWN per tenant; every membership `active` with a passkey;
    - the product map's pipeline rows: Kestrel Lane Property `approved`, Saltbush Commercial `fee_agreement`, and Wattle & Stone Living `kyb` with 2 of 3 directors verified;
    - the SPN-15 invoices: `INV-0931` paid and `INV-1002` due 15 Oct, both with GST lines.
- **Outbox events (A09):**
  - `tenancy.application.submitted | advanced | info_requested | declined | approved | withdrawn`;
  - `tenancy.tenant.created | activated | suspended | reinstated | offboarded`;
  - `tenancy.guardrails.changed`;
  - `tenancy.user.invited | joined | verified | role_changed | suspended | reinstated | removed`;
  - `tenancy.approval_rule.changed`;
  - `tenancy.listing_pause.set | cleared`;
  - `billing.invoice.issued | paid | overdue | void`, `billing.fee_schedule.changed`;
  - `sponsor.watchlist.opened | closed`, `sponsor.metrics.snapshot`;
  - `tenancy.revenue_share.set`;
  - `entities.entity.created | updated | reverification_required`, `entities.panel_valuer.set`;
  - `entities.licence.verified | unverified`;
  - `entities.vehicle.created | state_changed`.
  Every event carries `tenant_id`, `actor_id` and `on_behalf_of_id`.
- **The tipping-off rule.** `tenancy.application.declined` goes out on two topics:
  - the staff topic, with the real `reason_code`;
  - the non-staff topic (A43's applicant notice and any sponsor-facing consumer), with `reason_class: withheld` whenever the reason is `kyb_failed` or `sanctions`.
  A hidden test captures the non-staff payload, and A43's rendered notice fixture, and finds no reason class.
- **Error codes** are stable strings in the OpenAPI document:
  - `APPLICATION_ILLEGAL_TRANSITION`, `APPLICATION_NOT_EDITABLE`, `APPLICATION_ALREADY_OPEN`, `APPLICATION_KYB_INCOMPLETE`, `APPLICATION_LICENCE_UNVERIFIED`, `APPLICATION_LICENCE_NOT_ACTIVE`, `APPLICATION_FEE_AGREEMENT_MISSING`, `APPLICATION_REAPPLY_TOO_SOON`;
  - `TENANT_ILLEGAL_TRANSITION`, `TENANT_SUSPENDED`, `TENANT_LISTINGS_PAUSED`, `TENANT_HAS_LIVE_VEHICLES`, `TENANT_LAST_OWNER`;
  - `TENANT_USER_NOT_VERIFIED`, `TENANT_USER_EXISTS`, `MEMBERSHIP_SUSPENDED_BY_TENANT`, `ROLE_NOT_INVITABLE`, `INVITATION_EXPIRED`, `INVITATION_RESEND_LIMIT`;
  - `GUARDRAIL_LOCKED`, `GUARDRAIL_ABOVE_PLATFORM_MAX`;
  - `APPROVAL_RULE_BELOW_MINIMUM`, `RULE_VERSION_CONFLICT`, `SPONSOR_SELF_APPROVAL`, `SPONSOR_APPROVER_MUST_BE_HUMAN`, `SPONSOR_SECOND_HUMAN_REQUIRED`;
  - `VEHICLE_TRUSTEE_REQUIRED`, `VEHICLE_ILLEGAL_TRANSITION`, `VEHICLE_LICENCE_NOT_ACTIVE`;
  - `INVOICE_ILLEGAL_TRANSITION`, `FEE_SCHEDULE_NOTICE_TOO_SHORT`, `PAYMENT_EXCEEDS_BALANCE`;
  - `REVENUE_SHARE_LEGAL_REVIEW_REQUIRED`.
  Codes from other nodes are reused, never duplicated: A08's `PHASE_NOT_ENABLED`; A11's `FORBIDDEN`; A12's `AGNT_NOT_ALLOWED`, `SCOPE_PENDING_COUNSEL`, `G22_AGENT_NEVER_CHECKER` and `428 CONFIRMATION_REQUIRED`; A14's `APR_*`; A36's `OFR_SECOND_HUMAN_REQUIRED`.
- **Empty and edge states:**
  - A staff pipeline with no applications returns `{ items: [], next_cursor: null }`.
  - A tenant with no vehicles, no invoices or no snapshot returns empty arrays and `hint` strings.
  - A sponsor user whose membership is `pending_identity` gets `TENANT_USER_NOT_VERIFIED` with the identity case state, so SPN-02 can show "verify your identity to continue".
  - A suspended tenant's members get `TENANT_SUSPENDED` on every sponsor route except `GET /v1/sponsor/tenant`.
  - A public request for an unknown slug is 404.
  - A sponsor request for another tenant's id is 404 (no existence leak).
- **Swap points: none of this node's own.**
  - KYB and adverse media reach the identity and sanctions sandboxes only through A30 and A22.
  - E-signature goes through A35 when it lands.
  - The operating-account bank feed goes through A17.
  - The supplier ABN and operating-account details are `packages/config` values, with sandbox placeholders locally.
  - No vendor call, no secret and no production selector exists in the owned paths.

**10. Route catalogue and D04 agent column**

Every route below is in the OpenAPI document with its `x-realx-scope` action id, its `requiresHuman` flag and its `x-realx-agent` mode. The action ids are proposed in A03's grammar (`<PREFIX>.<object>.<verb>`). A03's owner assigns the final ids and maps each to its product-map source. The file `tenancy/parity.json`, generated from this table, is what A100's parity suite and A104's catalogue read. Any route without a row, or any row whose flags differ from `delegable.ts`, fails `npm run verify`.

The agent modes:
- **read**: delegable read. An agent may call it within its token's scopes.
- **maker**: delegable. An agent may act or draft as maker within its human's cells. Checkers are always human.
- **confirm**: every agent call returns A12's `428 CONFIRMATION_REQUIRED` with a `confirmation_id`, whatever the mandate's caps or signing key, and nothing happens until the person confirms with a fresh passkey assertion. A12 must honour a per-action `confirm_always` value (section 11). If it cannot, these routes fall back to `requiresHuman`.
- **human**: `requiresHuman: true`. An agent is refused with `AGNT_NOT_ALLOWED`.
- **counsel**: the `delegable.ts` value is `"counsel"`. An agent is refused with `SCOPE_PENDING_COUNSEL` until counsel answers.

"Notice yes" means the action is `sensitive` in A03's data, so A101 sends the person a notice whenever an agent performs it, drafts it, or it is confirmed.

*SPN-01, applicant (role SP-APP, `own-records`):*

| Route | Request → response | Errors | Action id (family) | Agent · notice |
|---|---|---|---|---|
| `POST /v1/sponsor-applications` | `{legal_name, abn or acn}` → application read, `state: draft` | `APPLICATION_REAPPLY_TOO_SOON`, `APPLICATION_ALREADY_OPEN` | `SPN.application.create` (sponsor-listing) | maker · no |
| `GET /v1/sponsor-applications/me` | → `{id, state, display_tone, sections, people[] {masked name, role, kyb_state invited/verifying/verified}, info_requests[] {id, question, answered_at}, fee_agreement {lines[], document_id, sha256, accepted_at}, submittable, reapply_after, reply_sla_days, previous_id}`; never a decline reason class | 404 | `SPN.application.read` (read) | read · no |
| `PATCH /v1/sponsor-applications/{id}` | section fields; in `info_requested`, only the requested fields → read | `APPLICATION_NOT_EDITABLE`, validation codes | `SPN.application.update` (sponsor-listing) | maker · no |
| `POST /v1/sponsor-applications/{id}/commitments/{code}/accept` | fresh assertion (`sponsor_terms`) → `standing_consent_id` | `FRESH_ASSERTION_*` | `SPN.application.acceptCommitment` | counsel · yes |
| `POST /v1/sponsor-applications/{id}/submit` | → `state: applied` | `APPLICATION_ILLEGAL_TRANSITION`, validation codes | `SPN.application.submit` | counsel · yes |
| `POST /v1/sponsor-applications/{id}/info-requests/{rid}/answer` | `{text, document_ids[]}` → read (state returns once all answered) | `APPLICATION_ILLEGAL_TRANSITION` | `SPN.application.answerInfo` (sponsor-listing) | maker · yes |
| `POST /v1/sponsor-applications/{id}/withdraw` | `{reason?}` → `state: withdrawn` | `APPLICATION_ILLEGAL_TRANSITION` | `SPN.application.withdraw` | confirm · yes |
| `POST /v1/sponsor-applications/{id}/fee-agreement/accept` | fresh assertion (`fee_agreement`) → `standing_consent_id` | `APPLICATION_ILLEGAL_TRANSITION` | `SPN.application.acceptFeeAgreement` | counsel · yes |

*SPN-02 and SPN-14, sponsor users (`own-tenant`):*

| Route | Request → response | Errors | Action id (family) | Agent · notice |
|---|---|---|---|---|
| `GET /v1/sponsor/tenant` | → `{tenant_id, card {display_name, user_display_name, role_label}, status, display_tone, onboarding_checklist[], listing_pauses[] {reason, text, set_at, link}, counts {trusts_on_realx, offers_live, listings_in_review}}` | `TENANT_USER_NOT_VERIFIED` | `SPN.tenant.read` | read · no |
| `GET /v1/sponsor/dashboard` | → `tiles[] {key, label, value_formatted, detail, source: ok or missing}` for Raised in live offer, Investors, Next distribution and Reporting, from registered tile providers | `TENANT_SUSPENDED` | `SPN.dashboard.read` | read · no |
| `GET /v1/sponsor/guardrails` | → current version with labels | — | `SPN.guardrails.read` | read · no |
| `GET /v1/sponsor/team` | → `members[] {membership_id, display_name, role, role_label, passkey_count, identity_verified, can_approve_labels[], state, display_tone, actions[]}`, `invitations[] {id, email_masked, role, expires_at, resend_count, state, actions[]}`, `partners {source, rows[]}` from A98's `teamPartnerRows`, `invitable_roles[]` for the caller | — | `SPN.team.read` | read · no |
| `POST /v1/sponsor/team/invitations` | `{email, role}` → invitation | `ROLE_NOT_INVITABLE`, `TENANT_USER_EXISTS` | `SPN.team.invite` (access) | confirm · yes |
| `POST /v1/sponsor/team/invitations/{id}/resend` | → invitation | `INVITATION_EXPIRED`, `INVITATION_RESEND_LIMIT` | `SPN.team.resendInvite` (access) | maker · no |
| `POST /v1/sponsor/team/invitations/{id}/cancel` | → invitation `cancelled` | — | `SPN.team.cancelInvite` (access, protective) | maker · yes |
| `PATCH /v1/sponsor/team/{membershipId}` | `{role}` → membership; `role: SP-OWN` → `202 {approval_id}` (`tenant.user.grant_owner`) | `TENANT_LAST_OWNER`, `SOD_*` | `SPN.team.changeRole`; `SPN.team.grantOwner` | confirm · yes; grant owner: human · yes |
| `POST /v1/sponsor/team/{membershipId}/suspend` | `{reason}` → membership | `TENANT_LAST_OWNER` | `SPN.team.suspend` (access) | confirm · yes |
| `POST /v1/sponsor/team/{membershipId}/reinstate` | → membership | `MEMBERSHIP_SUSPENDED_BY_TENANT` | `SPN.team.reinstate` (access) | confirm · yes |
| `DELETE /v1/sponsor/team/{membershipId}` | → membership `removed` | `TENANT_LAST_OWNER` | `SPN.team.remove` (access) | confirm · yes |
| `GET /v1/sponsor/approval-rules` | → `rules[] {action_code, label, required_roles[], required_count, version, platform_minimum {roles, count, text}, pending {approval_id, kind}}`, `minimums_banner[]` | — | `SPN.approvalRule.read` | read · no |
| `PUT /v1/sponsor/approval-rules/{actionCode}` | `{required_roles[], required_count, expected_version}` → `202 {approval_id, kind: tighten or weaken}` | `APPROVAL_RULE_BELOW_MINIMUM`, `RULE_VERSION_CONFLICT` | `SPN.approvalRule.change` (governance) | human · yes |

*Sponsor entities and vehicles:*

| Route | Request → response | Errors | Action id (family) | Agent · notice |
|---|---|---|---|---|
| `GET /v1/sponsor/entities`, `GET /v1/sponsor/vehicles`, `GET /v1/sponsor/vehicles/{id}` | → lists and detail (masked PII) | 404 | `SPN.entity.read`, `SPN.vehicle.read` | read · no |
| `POST /v1/sponsor/vehicles`, `PATCH /v1/sponsor/vehicles/{id}` (while `proposed`) | `{name, trustee_entity_id, custodian_entity_id, tax_class, deed_document_id?}` → vehicle `proposed` | `VEHICLE_TRUSTEE_REQUIRED`, `PHASE_NOT_ENABLED` | `SPN.vehicle.create` (sponsor-listing) | maker · no |
| `POST /v1/sponsor/vehicles/{id}/form` | → vehicle `formed` with `gate_evaluation_id` | `VEHICLE_LICENCE_NOT_ACTIVE`, `VEHICLE_ILLEGAL_TRANSITION` | `SPN.vehicle.form` (sponsor-listing) | maker · yes |

*SPN-15, SP-OWN and SP-FIN:*

| Route | Request → response | Errors | Action id (family) | Agent · notice |
|---|---|---|---|---|
| `GET /v1/sponsor/billing` | → `{plan {name, lines[] {code, label, amount, gst_treatment, enabled}}, usage_this_month {investors, distribution_runs, listings}, current_invoice {id, total_cents, due_on}, invoices[] {id, display_number, period, subtotal_ex_gst_cents, gst_cents, total_cents, paid_cents, partially_paid, due_on, pause_on, state, status_text, display_tone}, next_cursor}` | — | `SPN.billing.read` | read · no |
| `GET /v1/sponsor/billing/invoices/{id}` | → lines, totals, `tax_invoice {supplier_legal_name, supplier_abn, recipient_legal_name, recipient_abn, issue_date}`, `payments[] {received_on, bank_reference_masked, amount_cents}` | 404 | `SPN.invoice.read` | read · no |
| `GET /v1/sponsor/billing/invoices/{id}/payment-instructions` | → `{method: bank_transfer, account_name, bsb, account_number, reference, amount_due_cents, sandbox, note}` from the operating account, never a client-money account. "Pay invoice" is this read: the sponsor pays outside the platform and FINA records it | 404 | `SPN.invoice.paymentInstructions` | read · no |

*ADM-03, ADM-33, ADM-29 and ADM-39, staff (A11 cells; staff agents follow the AI-agent principal rule):*

| Route | Request → response | Errors | Action id (family) | Agent · notice |
|---|---|---|---|---|
| `GET /v1/admin/sponsor-applications`, `GET /v1/admin/sponsor-applications/{id}` | → pipeline rows `{id, legal_name, state, display_tone, age, directors "3 · 2 verified", owners, trusts_planned, track_record, next_action}`; detail with sections, people and KYB case states, `checks[]`, `info_requests[]`, `fee_agreement`, `approvals[]`, `gate_evaluations[]`, `decline_reason` (staff only) | — | `M02.application.read` | read · no |
| `POST /v1/admin/sponsor-applications/{id}/kyb/open` | → case ids | `APPLICATION_ILLEGAL_TRANSITION` | `M02.application.openKyb` | confirm · yes |
| `POST /v1/admin/sponsor-applications/{id}/checks` | `{kind, result, note, evidence_document_ids[]}` → check (licence pass writes the A21 row and `verification_state`) | `APPLICATION_ILLEGAL_TRANSITION` | `M02.application.recordCheck` | confirm · yes |
| `POST /v1/admin/sponsor-applications/{id}/info-requests` | `{questions[]}` → `state: info_requested` | same | `M02.application.requestInfo` | confirm · yes |
| `POST /v1/admin/sponsor-applications/{id}/decline` | `{reason_code, note}` → `state: declined` | same | `M02.application.decline` | confirm · yes |
| `POST /v1/admin/sponsor-applications/{id}/fee-agreement` | `{fee_schedule lines, document_id}` → application | same | `M02.application.attachFeeAgreement` | maker · yes |
| `POST /v1/admin/sponsor-applications/{id}/approval` | → `202 {approval_id}` (`sponsor.kyb.approve`) | precheck codes | `M02.application.approve` (maker side) | maker · yes |
| `POST /v1/admin/sponsor-applications/{id}/tenant` | `{short_name, slug, plan}` → `202 {approval_id}` (`tenant.create`) | precheck codes | `M27.tenant.create` (maker side) | maker · yes |
| `GET /v1/admin/tenants`, `GET /v1/admin/tenants/{id}` | → rows `{id, name, status, display_tone, admitted_on, guardrail_version, open_pauses[], vehicles, members}`; detail with guardrail versions, pauses, fee-schedule versions, approval rules, status history | — | `M27.tenant.read` | read · no |
| `PUT /v1/admin/tenants/{id}/guardrails` | guardrail fields → `202 {approval_id}` | `GUARDRAIL_LOCKED`, `GUARDRAIL_ABOVE_PLATFORM_MAX` | `M27.guardrails.set` | human · yes |
| `POST /v1/admin/tenants/{id}/listing-pauses`, `.../listing-pauses/{pauseId}/clear` | `{reason: watchlist_hold, note}` → `202 {approval_id}` | — | `M27.listingPause.set`, `.clear` | maker · yes |
| `POST /v1/admin/tenants/{id}/suspend` | `{reason_code}` + hardware-key assertion → tenant `suspended` | `APR_*` | `M27.tenant.suspend` | human · yes |
| `POST /v1/admin/tenants/{id}/reinstate`, `.../offboard` | → `202 {approval_id}` | `TENANT_HAS_LIVE_VEHICLES` | `M27.tenant.reinstate`, `.offboard` | maker · yes |
| `PUT /v1/admin/tenants/{id}/fee-schedule` | `{lines[], effective_from}` → `202 {approval_id}` | `FEE_SCHEDULE_NOTICE_TOO_SHORT` | `M27.feeSchedule.set` | maker · yes |
| `GET /v1/admin/billing/invoices`, `GET /v1/admin/billing/invoices/{id}` | → invoices across tenants with payments | — | `M27.invoice.read` | read · no |
| `POST /v1/admin/billing/invoices/{id}/record-payment` | `{amount_cents, bank_reference, received_on}` → invoice (may clear a pause) | `PAYMENT_EXCEEDS_BALANCE`, `INVOICE_ILLEGAL_TRANSITION` | `M27.invoice.recordPayment` | confirm · yes |
| `POST /v1/admin/billing/invoices/{id}/void` | `{reason}` → `202 {approval_id}` | same | `M27.invoice.void` | maker · yes |
| `GET /v1/admin/sponsor-metrics`, `POST /v1/admin/sponsor-metrics/snapshots` | `?tenant=&month=` → snapshot series; `{tenant_id}` → on-demand snapshot | — | `M24.metrics.read`, `.snapshot` | read · no; maker · no |
| `GET /v1/admin/sponsor-watchlist`, `POST /v1/admin/sponsor-watchlist`, `.../{id}/close` | → rows; `{tenant_id, reason}` → row | — | `M24.watchlist.read`, `.open`, `.close` | read · no; confirm · yes |
| `GET /v1/admin/revenue-share`, `POST /v1/admin/revenue-share` | → agreements; agreement fields → `202 {approval_id}` | `REVENUE_SHARE_LEGAL_REVIEW_REQUIRED` | `M24.revenueShare.read`, `.set` | read · no; maker · yes |
| `GET /v1/admin/entities`, `GET /v1/admin/entities/{id}`, `GET /v1/admin/vehicles`, `GET /v1/admin/vehicles/{id}` | → register and detail (licences via A21, KYB, vehicles, partner flags) | — | `M02.entity.read`, `M02.vehicle.read` | read · no |
| `POST /v1/admin/entities` | entity fields → entity (opens KYB when `kyb_required`) | validation codes | `M02.entity.create` | maker · yes |
| `PATCH /v1/admin/entities/{id}` | fields; a change to legal name or ABN reopens KYB | — | `M02.entity.update` | confirm · yes |
| `POST /v1/admin/entities/{id}/panel-valuer` | `{registration_number, expires_on}` → `202 {approval_id}` | — | `M02.entity.setPanelValuer` | maker · yes |

*Public:* `GET /v1/public/sponsors` and `GET /v1/public/sponsors/{slug}` (section 7). No principal, and no agent column.

**Staff single-person acts** are `confirm` for staff agents. A staff agent acting as maker must never make a final decision that no second person checks. This covers recording a payment (which can lift a pause), opening KYB, recording a check, requesting information, declining, and opening or closing a watchlist row. The agent drafts, and its human confirms in person.

**Acts that change authority** are `human` for everyone's agent: guardrails, suspension, the owner grant and approval-rule changes. They sit with token creation and widening, which are already non-delegable.

**Checkers.** Every approval decision on these action codes is A14's `decide`, which refuses any agent (`G22_AGENT_NEVER_CHECKER`, `APR_CHECKER_MUST_BE_HUMAN`). Segregation of duties counts each agent as its human.

**The sponsor agent policy card (SPN-16)** is A12's. A12's frozen spec already defines it as `PUT /v1/agent-access/tenant-policy`: SP-OWN only, `requiresHuman`, and refused above the platform policy with `POLICY_ABOVE_PLATFORM`. This node does not define a second policy table. It contributes:
- the tenant approval-rule code `agent.tenant_policy.loosen`, with its platform minimum (a second SP-OWN, or PART when `tenant_owner_count = 1`) in `sponsor-minimums.ts`;
- the checker resolver above, so that a loosening routed through A14 gets its second person here.
Tightening takes effect at once. A12 must version the tenant policy and route loosening through A14 (section 11). A60's specify point (e) resolves to A12.

## Owned paths (other nodes do not edit these)

- <workspace>\app\services\api\src\modules\tenancy\
- <workspace>\app\services\api\src\modules\entities\

Inside `tenancy/`, by convention:
- `applications/`: the pipeline machine, checks, info requests, KYB event handlers and the A21 licence calls;
- `tenants/`: the status machine, guardrails, pauses, and the suspension, reinstatement and offboarding effectors;
- `users/`: memberships, invitations, approval rules, `evaluateSponsorApproval`, and the A14 resolver and guard;
- `billing/`: schedules, GST, the accrual and overdue jobs, invoices and payments;
- `performance/`: snapshots, watchlist and providers;
- `revenue-share/`;
- `public/`: the sponsor profile contract;
- `parity.json`, `jobs.ts`, `events.ts`, `schema.ts` and `migrations/` (registered through A04, including the `tenant` constraint trigger), `seed.ts`;
- `index.ts`, the only import surface.

Inside `entities/`: `entities/`, `licences/` (verification and A21 event handlers only), `vehicles/`, `schema.ts`, `events.ts`, `index.ts`.

These are data files proposed to `packages/domain` and applied by the domain node's owner in the same milestone:
- the pipeline, tenant, vehicle and invoice transition tables (A03's machine format, in a new file `tenancy-machines.ts`, not in `extensions.ts`);
- `sponsor-minimums.ts`;
- the proposed action ids and `delegable` values of section 10.

These are seed rows in A14's `approval_policies`, proposed by this node and reviewed there:
- `sponsor.kyb.approve` (4E, KYCA → CCO);
- `tenant.create` (4E, PART or SUPER → CCO);
- `tenant.guardrails.set` (4E, CCO → LEGL);
- `tenant.user.grant_owner` (4E, SP-OWN → second SP-OWN, or PART when `tenant_owner_count = 1`);
- `tenant.approval_rule.change` (4E, SP-OWN → CCO);
- `tenant.approval_rule.tighten` (4E, SP-OWN → second SP-OWN, or PART when `tenant_owner_count = 1`);
- `tenant.fee_schedule.set` (4E, CFO → CEO);
- `invoice.void` (4E, FINA → CFO);
- `tenant.listing_pause.set` (4E, PART → CCO);
- `tenant.listing_pause.clear` (4E, PART → CCO);
- `tenant.revenue_share.set` (6E, PART → CFO + CCO, plus LEGL when transaction-based);
- `tenant.suspend` (SGL+R, CCO, ratified by CEO, SECO or LEGL within 2 h);
- `tenant.reinstate` (4E, CCO → CEO);
- `tenant.offboard` (4E, PART → CCO);
- `entities.panel_valuer.set` (4E, PART → CCO).

That is fifteen rows.

## Design references

- **`design:spDash tenant card`** in `<workspace>\design\RealX App.dc.html` line 526 (sponsor shell `sp`, the sidebar card under the logo). It is a cream `#F5F2EC` card with radius 10px and padding 12px. The first line is Geist 500 "Harbour & Co Property"; the second is 12px `#5E6166` "Mia Chen · Sponsor owner". `tenantSummary()` returns exactly `display_name`, `user_display_name` and `role_label`, with the four labels "Sponsor owner", "Operator", "Fund accountant" and "Investor relations" (the SPN-14 mock's words), so A62 renders the card with no translation. The sidebar itself and the nav are A62's.
- **`design:prop Sponsor tab facts`**, lines 264-271 (public shell, `prop` screen, tab `sponsor`, shown as "PUB-05 · Sponsor"):
  - the sponsor name as a Newsreader 30px card title;
  - the about paragraph (`#2E3135`, line-height 1.6);
  - the "Vetted by RealX · {vetted}" badge: green tint `#E3EEE8` background, green `#1F5E4B` text, Geist 500 12px, pill radius;
  - the 2-column FactsGrid of four `{l, v}` cells: white cells on a `#E3DED4` 1px grid, label 12px `#5E6166`, value Geist Mono 500 16px.
  Mock data lines 746-748 (`SP.s1`, `s2`, `s3`) fix the labels "Trusts on RealX", "Trusts managed", "Assets managed" and "Licence", and the values "Own AFSL" and "Trustee: hired RE". The public contract in section 7 uses these strings verbatim, and the seed reproduces the three sponsors. The facts of record render live; the declared facts render from the published block.
- **Status vocabulary for the tone pills** (design-analysis §1.1 `TONE` map). Each status is returned as an enum value plus a `display_tone`, so no screen maps them itself:
  - amber: pipeline `applied`, `kyb`, `background`, `info_requested`; `onboarding`, `pending_identity`, `invited`; invoice `issued`;
  - brick: `fee_agreement` (the design's "in progress, needs a person" tone, as for "changes requested");
  - green: `approved`, `tenant_created`, tenant `active`, membership `active`, invoice `paid`;
  - red: `declined`, tenant `suspended`, membership `suspended`, invoice `overdue`;
  - grey: `withdrawn`, `offboarded`, `removed`, invoice `void`.
- **Undrawn screens this node feeds:**
  - From [[A50 - Design extension A- investor and sponsor screens not in the prototype]] (`design-ext:SPN-01`, `design-ext:SPN-14`, `design-ext:SPN-15`):
    - SPN-01: the listing-wizard pattern, with AcknowledgeCards for the three commitments;
    - SPN-14: DataTable Person / Role / 2FA / Can approve, "Invite user", and the platform-minimum banner "distributions need two people at the sponsor and two at RealX";
    - SPN-15: KpiGrid plus the invoices DataTable, and the "How to pay" card.
  - From [[A51 - Design extension B- operator console screens not in the prototype]]:
    - ADM-03: the ApprovalList and Detail pattern, with the pipeline table left and the KeyValueList and approvals chain right;
    - ADM-29: DataTable Sponsor / Trusts / Raised / Yield plan vs paid / On-time / Proof score / Keeps / Flag;
    - ADM-33: KeyValueList cards such as "KYC required · Locked on", and the tenant status card;
    - ADM-39: the entities register and detail, a DataTable plus detail.
  - The "2FA ✓" column of SPN-14 is `passkey_count > 0 && identity_verified` from this node, not a stored flag.
- **Superseded by this node's data:** the product-map mocks' coral palette and the BrickX-style tenant "Kestrel Lane" branding are seed names only. The visual system is the dc prototype's.

## Requirements covered

- **Product-map screens: none owned.** The consuming nodes are listed in the introduction; ADM-35 shows sponsor tokens revoked on suspension through A77 over A12. "Done" here means that every field, action, status string and error those screens need exists as a route in section 10, an event or an `index.ts` function, proven by a contract test that the screen nodes reuse unchanged. A62's specify points are settled here:
  - (b) `GET /v1/sponsor/dashboard`;
  - (c) `actions[]` and `invitable_roles[]` on `GET /v1/sponsor/team`;
  - (d) `pause_on` on every unpaid invoice.
- **Journeys: none owned.** "Done" means J11 (sponsor joins RealX) runs end to end at service level with the seeded personas:
  1. SPN-01 submit.
  2. PART opens KYB.
  3. KYCA cases approve through A30's sandbox scenarios.
  4. The licence check writes an A21 row that is `active` and `verified`.
  5. The CCO approves `sponsor.kyb.approve` with a passing G21 evaluation.
  6. PART and the CCO complete `tenant.create`, and A21's `initTenantFlags` runs.
  7. The owner accepts the invitation and the tenant reaches `active`.
  8. SP-OWN invites a team member who verifies identity.
  9. An approval rule below the minimum is refused.
  J11's three oracle hooks must hold: a tenant cannot exist without an approved KYB record (by constraint trigger); `kyc_required` is immutable; and sponsor users cannot approve their own submissions (through A14's decide). J19 step 6 (open the invoice and its payment instructions; FINA records the payment) and J20 step 9 (PART reviews sponsor performance) run against this node's billing and snapshot contracts. J22 (delegate to your own agent) runs against section 10's agent modes for a sponsor token.
- **Gates:**
  - **G02 Sanctions and PEP clear** and **G03 Entity and owners verified**: done when the pipeline cannot pass `kyb` while any director, owner at 25 percent or more or entity case is not `approved`, a later-added person reopens KYB, a true match declines silently (with a withheld reason class to non-staff consumers), and each outcome is a `gate_evaluation` row through A30.
  - **G13 Maker-checker**: done when every act in the fifteen proposed policy rows is an `ApprovalRequest` with the stated code, and A14's decide, with this node's resolver and guard, refuses the submitter, any principal of the same person, any drafting agent's human, any agent and any token. The `required_count`, last-owner, one-owner-fallback and platform-minimum rules must also hold.
  - **G19 Proof of property current**: done when the attestation-overdue pause set by A34 through `setListingPause` blocks `canSubmitListing`, the public profile shows `proof_overdue` live, and the pause clears exactly once when A34 clears it.
  - **G21 Licence perimeter**: done when no tenant is created, and no vehicle formed, unless `evaluateG21` passes on an A21 licence that is `active` and `verified`; a G21 `GateEvaluation` id is stored on every admission approval and every formed vehicle; an A21 expiry, suspension or revocation pauses listings and alerts the CCO; and the legal sufficiency of the authorisation is recorded as a counsel question rather than a rule.
  - **G17 Marketing claims**: done when no sponsor figure or declared fact reaches a public response except through an approved A45 publication.
  - **G22 Agent authority**: done when every route's agent mode in section 10 matches `delegable.ts` and the OpenAPI flags, and the parity file is generated from them.
- **Modules:**
  - **M02 Sponsors, trusts and entities**: done when the admission pipeline (ADM-03's table and detail), the entities register and detail (ADM-39), sponsor licence verification over A21's rows, and vehicles exist as data with their transitions tested.
  - **M24 Partners**: done when sponsor performance snapshots, the watchlist and revenue-share agreements exist with providers registered, and the published-only rule for public numbers is enforced by schema and test.
  - **M27 Settings and tenancy**: done when the tenant registry, versioned guardrails with `kyc_required` locked on, memberships, invitations, approval rules, billing with GST and suspension (ADM-33) exist, and no tenant, by any route, can turn off KYC.
- **State machines:** none of the 14 is owned. The four local machines (application, tenant, vehicle, invoice) are recorded in A03's format with exhaustive transition tests. The tenant `suspended` state is the object of A14's `SGL+R`, and the vehicle `closed` state is the end of A48's J21.
- **Founder requirement D04:** done when every route in section 10 carries its agent mode and notice flag. A sponsor agent must be able to read everything its owner can read, prepare an application, answer information requests, create and form vehicles, and draft invitations and member changes for its human to confirm. Every approval, guardrail, suspension, owner grant and rule change must refuse an agent or token before scope evaluation. Staff agents must confirm every single-person staff act through their human. `parity.json` must feed A100 and A104 unchanged.

## Gauntlet loop for this node

- **Specify.** This note is frozen together with:
  - the four local transition tables in `tenancy-machines.ts`, and `sponsor-minimums.ts`, as proposed to A03;
  - the SP-APP role and self-scope cells and the section 10 action ids and `delegable` values, as the A03 and A11 diff;
  - the module `schema.ts` migration list, including the `tenant` constraint trigger;
  - the fifteen approval policy rows proposed to A14;
  - A14's `registerCheckerSetResolver` and `registerDecisionGuard` ports;
  - A21's licence ownership split, `evaluateG21`, `initTenantFlags` and the summary read;
  - A30's KYB interface and event names;
  - A11's matrix rows for SP-OWN, SP-OPS, SP-FIN, SP-IR and SP-APP;
  - A12's confirmation mode;
  - the design regions above.

  If A14's two ports, the A03/A11 diff or the A21 edge are absent, the Specify step stops and the upstream nodes are reopened.

  The node QA records `specVersion`, the hashes of those data files, `oracleVersion` and `hiddenCasesHash` in the evidence manifest. It then writes the hidden tests to `<workspace>\gauntlet\hidden\A33\` before the build:
  - pipeline transition vectors: every allowed pair and every forbidden pair;
  - a KYB case moved to `referred` after `sponsor.kyb.approve` was submitted;
  - an A21 licence expired, suspended or revoked at each pipeline stage (no `tenant_created`, no `active`, no `formed`, a `licence_expired` pause after admission);
  - a tenant insert carrying a foreign approved approval id;
  - `kyc_required = false` through this node's route, through A14's generic draft and submit, and through direct SQL under the API role;
  - self-approval through the submitter, a principal sharing the submitter's `person_id`, the human behind a drafting agent, the submitter's agent token, a sibling agent of another owner and a staff token, all driven through `POST /v1/approvals/{id}/decide`;
  - `required_count` 2 and 3 with one, two and three eligible humans;
  - the one-owner cases for `listing.submit` (`OFR_SECOND_HUMAN_REQUIRED`) and `tenant.user.grant_owner` (PART);
  - the last-owner rule;
  - rules below, at and above the minimum, including a weakening approved by the CCO;
  - suspend and reinstate cycles over `invited`, `pending_identity`, `active` and individually `suspended` members;
  - billing and GST: `pause_on` on day 29, 30 and 31 after due; the sweep path versus the read-time path; partial payments; void; double clearing; GST rounding at half a cent;
  - watchlist windows at 6 months minus and plus one day;
  - suspension with an in-flight sponsor approval and an in-flight agent call on a second API process;
  - a two-tenant RLS suite over every registered table;
  - a public-profile schema scan, and the unpublished-snapshot case;
  - the non-staff decline payload scan;
  - the section 10 table against the OpenAPI flags;
  - the "tokenised value" grep.
- **Build (Grok 4.7 xhigh, split by the planner).** Suggested shape: four parallel sub-agents on disjoint sub-folders, then integration.
  - (a) `applications/` and `entities/`: the pipeline machine, checks, info requests, KYB handlers, the licence check over A21's table, the `sponsor.kyb.approve` and `tenant.create` prechecks with `evaluateG21`, the effectors with `initTenantFlags`, the `tenant` constraint trigger, entity and vehicle records with their machines, `panel_valuer`, and the A21 and A38 event handlers.
  - (b) `tenants/` and `users/`: the status machine, versioned guardrails, pauses, `canSubmitListing`, memberships, invitations through A11's path, approval rules with the minimums, `evaluateSponsorApproval`, the A14 resolver and decision guard, and the suspension, reinstatement and offboarding effectors with the event contract.
  - (c) `billing/` and `performance/`: schedules with GST, the month-end and overdue jobs, invoices and payments, the overdue pause, providers, snapshots, watchlist and revenue share.
  - (d) `public/`, `events.ts`, the section 10 routes and OpenAPI entries with the `x-realx-agent` flags, `parity.json`, the error catalogue, the seed hook, and the screen contract tests for A62, A69, A76 and A54.
  - Integration: register the module schemas and jobs; wire the A14 hooks, resolver and guard; run the visible suite against Docker Postgres with the injectable clock and A30's sandbox scenarios; regenerate A104's MCP catalogue; run `npm run verify`; write `app/evidence/A33.json` with the data-file hashes, event list, coverage and test counts.
- **Challenge (node QA, Opus 5.5 high).**
  - Runs the hidden tests.
  - Drives J11 by hand against the running two-process stack with the seeded personas.
  - Attempts the node-specific challenges below.
  - Counts one audit event per transition, decision and job effect over a scripted run, and verifies A10's chain afterwards.
  - Diffs the builder's transition tables, minimums, policy rows and `parity.json` against the frozen copies.
  - Checks that the module imports A03, A04, A09, A10, A11, A12, A14, A21 and A30 only through their public `index.ts`.
  - Confirms the public contract's field set and the seed's three sponsors against the design's mock data.
  - Confirms no vendor call, no secret and no `Date.now()`, and that only the owned paths (plus the agreed A03 and A14 data proposals) changed.
  - Writes `app/evidence/A33.qa.md`.
- **Gate, fix loop, escalation.**
  - Pass unlocks the six dependents and publishes the hidden cases into the repo.
  - At most three fix loops on the same spec, each re-running the full hidden set and the failing challenges.
  - Escalate to the user when the same assumption fails a third time.
  - Escalate as a spec defect upstream, reopened there, when a fix needs a change in A11's matrix or SoD data, A14's engine or ports, A21's licence split or summary mapping, A30's KYB interface, A12's confirmation mode or A04's `tenant_guardrail` CHECK.
  - Escalate as Critical, stopping the milestone, when any route lets a sponsor principal, any agent or any token change a guardrail, approve its own submission, suspend a tenant or reach a `tenant.create` decision.
  - Escalate when a founder question below (fee basis, pause days, reapply period, watchlist rule, GST treatment) turns out to block a test rather than parameterise it.

## Node-specific challenges (node QA)

- **KYC switched off by a side door (compliance, G03).**
  - A CCO calls `PUT /v1/admin/tenants/{id}/guardrails` with `kyc_required: false`.
  - The same payload is drafted through A14's generic `POST /v1/approvals` and submitted.
  - The same values are inserted as a guardrail version by direct SQL under the API role.
  - A tenant is created from an application whose fee agreement is signed but whose sole owner's case is `info_requested`.

  All four must fail: `GUARDRAIL_LOCKED` on this node's route with no approval request created; the generic request `rejected` at submit with `GUARDRAIL_LOCKED` and never approved; the A04 CHECK on the SQL path; and `APPLICATION_KYB_INCOMPLETE` at submit. Afterwards no tenant row and no guardrail version exists.
- **The owner approving themselves through a second hat (compliance, G13).**
  - The database must refuse a second `tenant_user` for Mia's `person_id` in Harbour & Co, whether as SP-OPS or as an A98 SP-VAL membership (`TENANT_USER_EXISTS`).
  - A tenant-scoped request whose maker is a principal sharing Mia's `person_id` must not be approvable by her SP-OWN principal. Two variants: her investor principal drafting through A14's generic route with a fixture tenant-scoped action, and a partner principal of hers in another tenant.
  - Tom (SP-OPS) submits a listing that Mia's agent token drafted. Mia may not approve it.
  - Mia's agent token tries to approve.
  - A sibling agent of the second owner tries to approve.
  - A RealX staff token with every M02 scope tries to approve.

  Every attempt goes through `POST /v1/approvals/{id}/decide` and must be refused: `SPONSOR_SELF_APPROVAL` for the person cases, and `SPONSOR_APPROVER_MUST_BE_HUMAN` or `G22_AGENT_NEVER_CHECKER` for the agents and the token, before scope evaluation. Only the second SP-OWN in person, with a fresh assertion, succeeds, and the audit rows name the human behind each attempt.
- **Weakening the minimum with RealX's help (compliance).**
  - SP-OWN submits an approval rule for `distribution.propose` with `required_count: 1`. It must be refused with `APPROVAL_RULE_BELOW_MINIMUM`, naming the two-person minimum, on this node's route before any request exists.
  - The same payload is drafted through A14's generic route and the RealX CCO tries to approve. The request must end `rejected` at submit with the same code, and the rule must be unchanged.
  - A rule from 3 down to 2 must create a `tenant.approval_rule.change` request and take effect only after the CCO's decision.
  - A rule from 2 up to 3 must create `tenant.approval_rule.tighten` and wait for a second SP-OWN.
- **One owner, one submitter (compliance).**
  - Northline has one SP-OWN, who submits a listing. It must be refused with `OFR_SECOND_HUMAN_REQUIRED`, with `sponsor.min_approvers` in the detail.
  - The same owner grants SP-OWN to a colleague. The request resolves RealX PART as checker, and only PART in person can decide it.
- **Suspension against an in-flight action (security, money).**
  - With the SP-FIN's agent midway through a mandated distribution proposal on API process 1, the CCO suspends the tenant on process 2, and the transactions race. Whichever commits second must see the other: the agent's next call is `TOKEN_REVOKED`, the pending sponsor approval is cancelled, the SP-FIN's session is gone on process 1, and an approved distribution run already in `funding` for that tenant continues untouched.
  - The ratification is then left to lapse. A14 raises Sev2, and the tenant must still be `suspended` an hour later. If any timer reinstates it, the node fails.
  - The reinstatement must bring back exactly the prior member states. The `pending_identity` member stays `pending_identity`, and the member suspended for cause stays suspended.
- **Billing boundary and the pause (money).**
  - Invoice `INV-1002`, due 15 Oct, with `pause_after_days = 30`, has `pause_on` at 00:00 on 15 Nov.
  - A payment recorded at 23:59:59 Sydney on 14 Nov must prevent the pause. One at 00:00:01 on 15 Nov must not: the pause is visible to `canSubmitListing` at 00:00:01 even with the sweep disabled.
  - A partial payment of all but one cent leaves `overdue` and the pause.
  - A `void` clears it.
  - Two invoices overdue and one paid leaves the pause.
  - Running the month-end job twice over the same month issues one invoice.
  - GST on taxable lines summing to an odd half-cent rounds half-up once per invoice.
  - The Sydney day, never the UTC day, decides.
- **Published numbers only (compliance, N16, G17).**
  - Seed an open offer with A$1.53m in escrow and a finalised allocation of A$2.0m for the same sponsor. The snapshot's `capital_raised_cents` must be A$2.0m.
  - The public profile must show no performance figure until an A45 `metrics.publish` request is approved. It then shows exactly the published values, with `publication_id` and `approval_id`, even after a newer snapshot differs.
  - "Assets managed A$212m" appears only inside the published block, with `source: sponsor_declared`.
  - A missing provider shows `source: missing`, not zero.
  - An open `attestation_overdue` pause shows `proof_overdue: true` live.
  - The strings "tokenised value" and any escrow figure must be absent from every public response body.
- **Tenant leak (security).**
  - Northline's SP-OWN lists team, invoices, vehicles, guardrails and the application of Harbour & Co by guessing ids, then through a `?tenant=` filter, then through a public slug.
  - A staff session viewing Harbour & Co as tenant asks for Northline's revenue share.
  - Every private read must be 404 or `VIEW_AS_NOT_ALLOWED`, the public read must show only the section 7 field set, and the RLS suite over every registered table must show zero cross-tenant rows.
- **Design fidelity of the data contract.** These must equal the frozen vocabulary exactly, so that A62, A69, A76 and A54 render the design without a mapping table: the tenant card, the four sponsor fact labels, the two licence strings (from A21's summary for the seeded structures), the "admitted MMM yyyy" badge text, the four role labels and every status string above. Any new enum value, any free-text status, or a public fact without a `source` fails the contract test.

## Pass criteria

- **Hidden-test agreement:** 100 percent of the node QA's tests in `<workspace>\gauntlet\hidden\A33\` pass on three consecutive runs on the Docker stack with different seeds. This includes the two-tenant RLS suite over every table this node registers, and the two-process suspension race repeated 50 times with zero misses.
- **Transition coverage:** a generated test exercises every state of the application, tenant, vehicle and invoice machines and every listed transition at least once, and refuses 100 percent of the non-listed pairs with the stable code. The count of forbidden pairs in the test equals the count derived from the tables.
- **Approval and SoD matrix through the real path:**
  - For every tenant-scoped action code and every principal fixture, a generated test drafts, submits and decides through A14's real `POST /v1/approvals`, `/submit` and `/decide` endpoints, never only through `evaluateSponsorApproval`. The fixtures are the four sponsor roles, a principal sharing a member's `person_id`, an AGTU per sponsor role, one token per prefix, PART, KYCA, CCO, LEGL, CFO, CEO, SECO and SUPER.
  - It asserts who may draft, submit, approve and ratify, the `required_count` distinct-approver rule, and the one-owner fallbacks.
  - Zero cells disagree with A11's `eligibleCheckers`, A14's policy rows, the resolver and `sponsor-minimums.ts`.
  - Every `kyc_required = false` attempt, through this node's route, A14's generic path or SQL, is refused.
- **J11** end to end at service level with the seeded personas produces a tenant in `active`. Its `approval_id` references a `tenant.create` request that passes the constraint trigger; that request's precheck cites approved KYB cases for every director and owner and a passing G21 `GateEvaluation`; and its issuing licence is A21 `active` and `verified`. The run leaves exactly one audit event per transition, decision and job effect (counted against the request and job log), and A10's chain verifies afterwards. A sponsor whose A21 licence expired before `tenant.create` never reaches `tenant_created`.
- **Billing and metrics under the injectable clock:**
  - the month-end and overdue jobs are idempotent (a second run changes nothing);
  - for 1,000 fast-check tenant-month fixtures, invoice totals equal the sum of fee-ledger lines plus GST to the cent, and GST equals the rounded taxable sum times the rate;
  - the overdue pause is set and cleared exactly once per invoice lifecycle on both the sweep and read-time paths;
  - the watchlist opens on the second late run inside 6 months and not on one outside.
- **Contracts:**
  - Every route in section 10, every event and every error code is in the OpenAPI document with its `x-realx-scope`, `requiresHuman` and `x-realx-agent` flags, and `parity.json` equals the table.
  - `packages/sdk` regenerates without manual edits.
  - A104's MCP catalogue, regenerated in the integration step, lists the sponsor-side and staff maker tools of section 10. It lists none of the `human` or `counsel` routes (guardrails, suspension, owner grant, approval-rule change, commitments, fee-agreement acceptance, application submit), and A14's decide stays agent-refused.
  - The public-profile schema test proves that no private field and no unpublished figure is reachable.
  - The seed reproduces the design's three sponsors, with their licence structures, and the product map's three pipeline rows exactly.
  - The screen contract tests for A62 (including the dashboard, `actions[]`, `invitable_roles[]` and `pause_on`), A69, A76 and A54 pass.
- **Hygiene:**
  - line coverage is at least 90 percent on both modules;
  - the `Date.now()`, vendor-selector and "tokenised value" greps are clean;
  - module-boundary lint passes;
  - `npm run verify` is green on a fresh clone on Node 24.21.0;
  - only the owned paths and the agreed A03 and A14 data proposals changed;
  - `app/evidence/A33.json` and `A33.qa.md` exist and match disk.

## Human gates touched

- **HG-LEGAL (counsel questions recorded, not decided; none blocks the node):**
  - whether a sponsor's AFSL authorisations, or a named licensed third-party trustee, suffice for it to issue units of its wholesale trusts through RealX as CAR, and what the admission licence check must verify;
  - whether transaction-based revenue share to a sponsor is permitted under the conflicted-remuneration rules, and what disclosure it needs;
  - whether investors must be told when a sponsor is suspended, and whether vehicles may keep operating under an inactive sponsor licence;
  - whether the sponsor application declaration, the three commitments and the fee-agreement acceptance must be made in person (they sit on A03's `"counsel"` list until answered; the application declaration is an entry beside SPN-06's under A00's CQ-09);
  - whether the retained-stake commitment is enforceable, and how it is measured;
  - what the sponsor terms must say.

  Each answer changes data (a check row, a policy row, a `delegable.ts` value), not code, and the node's hidden set is re-run after the change.
- **Founder questions (recorded as "proposed" data with defaults):**
  - the platform fee basis: per investor per month, as the product map says; 0.25 percent p.a. of net assets charged to the trust, as the design says; or both;
  - whether PART should draft fee-schedule changes (the product map says CFO → CEO);
  - the 30-day billing pause and the 30-day fee-notice period;
  - the 6-month reapply period;
  - the two-late-distributions-in-6-months watchlist rule;
  - the sprawl count;
  - the 10 percent retained stake;
  - whether the design string "Trustee: hired RE" stays for a P1 professional trustee (with A43's G17 review).

  The build ships every one as a configurable row, so the choice is a data change.
- **Accountant question (recorded with the founder questions):** the GST treatment of each fee line (listing fee, per-investor platform fee, distribution-run fee, and the trust fee charged to the trust rather than the sponsor, whose recipient is the trustee); per-invoice versus per-line rounding; and the tax-invoice wording. All are data in `packages/config` and on the fee lines. The defaults are `taxable` at 10 percent, per-invoice rounding, and the supplier ABN as a sandbox placeholder until the real one is configured.
- **HG-VENDOR: none crossed.** KYB, sanctions and adverse media run through A30 on A22's sandbox adapters with name-suffix scenarios. E-signature is a document hash plus a passkey assertion until A35. The operating-account feed is a recorded payment until A17. No production key, real money, mainnet or vendor account is touched.
- **HG-PENTEST (later):** invitation tokens, the applicant self-scope, the public sponsor contract, the confirmation path for agent-drafted member changes and the suspension path are in scope of the external penetration test before go-live. This node passes on software evidence and records the gate as open.

## Upstream and downstream spec points this node depends on

These are not edited here. Each is a change the named node must make, raised by the orchestrator:
1. **A14:** expose `registerCheckerSetResolver(actionCode, resolver)`, called at submit after A11's `eligibleCheckers` so that tenant checker sets are appended to `required_checkers[]`, and `registerDecisionGuard(actionCode, guard)`, called at decide before the decision is recorded. Add the fifteen policy rows proposed above.
2. **A11 and A03:** add the role `SP-APP`, a HUMAN in the platform tenant, with `own-records` cells `SPN.application.*`. Add the section 10 action ids with their `delegable` values (`"counsel"` for the application submit, the commitments and the fee-agreement acceptance). Add `sponsor-minimums.ts` and `tenancy-machines.ts`.
3. **A14 (optional):** a `registerDraftValidator(actionCode)` port, so that the generic `POST /v1/approvals` refuses module-owned payloads at draft. Without it, the refusal happens at submit, as tested above.
4. **A12:** honour a per-action `confirm_always` agent mode (`428 CONFIRMATION_REQUIRED` regardless of caps or signing key). Version the sponsor agent policy and route loosening through A14 as `agent.tenant_policy.loosen`, using this node's resolver. A60's point (e) resolves to A12.
5. **A21:** the summary read maps `trustee_relationship = third_party_professional` (P1) and `responsible_entity` (P2) to "Trustee: hired RE", and `sponsor_group` with `sponsor_afsl` to "Own AFSL". Publish `licence.expired`, `licence.suspended`, `licence.revoked` and `licence.reinstated` on the outbox. Add the graph edge A21 -> A33 that A21 already flags.
6. **A38:** publish `settlement.run.completed` on the outbox, carrying `vehicle_id` and `offering_id`.
7. **A45:** register `PublishedSponsorProfileProvider` over its approved `metric_publication` for tenant scope, including declared facts, per-trust rows and the `about` text. Read this node's snapshots through `sponsorSnapshotSeries`.
8. **A47, A40, A34 and A44:** register the `BillingInputProvider`, `MetricProvider` and `DashboardTileProvider` implementations from their own paths. A36 and A43 register the Raised and Reporting tiles.
9. **A98:** register `teamPartnerRows` through `registerTeamRowsProvider`.
10. **ADM-32 owner (unassigned):** the node that owns sponsor integration API keys revokes them on `tenancy.tenant.suspended`.
11. **Graph:** set `oracle: yes` for A33 in `app-graph.json`, and add the dependency A21 -> A33.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
