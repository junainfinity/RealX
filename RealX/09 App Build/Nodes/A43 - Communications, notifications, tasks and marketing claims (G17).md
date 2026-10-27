---
title: "A43 - Communications, notifications, tasks and marketing claims (G17)"
created: 2026-09-23
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am3]
node: A43
milestone: AM3
lane: "Services"
oracle: no
human_gate: none
---

# A43 - Communications, notifications, tasks and marketing claims (G17)

**The one module that talks to people. It holds a notification catalogue by type and channel, with per-person preferences and "mark all read". It has email and SMS sandbox adapters that capture every message into a local inbox for end-to-end tests, and versioned templates. Its worker jobs turn frozen producer events and due dates into the design's sponsor "Needs you" tasks and into investor, partner and staff notices. It keeps watch lists behind "Notify me", and marketing consent with a working unsubscribe for campaigns. It runs the RG 234 claims lint and the approval workflow behind every sponsor announcement, significant-event notice, campaign, piece of site copy, terms version and fee notice. Nothing reaches an investor or the public site without an approval record, and nothing muted can hide an agent acting in someone's name.**

**Build status:** not started. **Milestone:** AM3 (Business services). **Lane:** Services. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 10-16 agent-hours (build, challenge and verify loops). The graph still records 6-10; see the graph note below.

**Depends on:** [[A14 - Approvals engine and maker-checker (SM-00)]], [[A35 - Documents, disclosure library, e-sign and acknowledgements (G12)]]
**Unlocks:** [[A59 - AUD wallet, token wallet and account screens]], [[A62 - Sponsor console- apply, dashboard, team and approvals, billing|A62 - Sponsor console: apply, dashboard, team and approvals, billing]], [[A63 - Sponsor console- new listing wizard SPN-03 to SPN-06 and offer monitor SPN-07|A63 - Sponsor console: new listing wizard SPN-03 to SPN-06 and offer monitor SPN-07]], [[A64 - Sponsor console- register, distributions, property reporting, valuations, announcements|A64 - Sponsor console: register, distributions, property reporting, valuations, announcements]], [[A75 - Console- AML cases, complaints register, marketing compliance, disclosure library|A75 - Console: AML cases, complaints register, marketing compliance, disclosure library]], [[A101 - Agent guard- anomaly detection, sensitive-action notices and tiered kill switches|A101 - Agent guard: anomaly detection, sensitive-action notices and tiered kill switches]]

**Graph note for the plan reviewer.** A43's ancestors are A00-A04, A08-A14 and A35. This node never calls a module that is not an ancestor, and no producer calls A43. Everything else reaches it as outbox events, whose consumer contracts are frozen in this note (see "Event contracts consumed"), or through injected resolvers with fixtures. Two graph edges are still recommended. Neither creates a cycle. (1) A28 → A43 and A29 → A43: both are AM2 nodes, and this node implements their notifier ports in `comms/bindings/`. (2) A43's estimate in the graph should read 10-16 instead of 6-10. Adding the AM3 producers (A20, A32, A33, A34, A36, A37, A39, A40, A48) as dependencies is optional. The consumer contracts make the build independent of their order.

**Oracle rule:** this is not an oracle node (`oracle: no`), because no money, supply or holdings move here. The node QA still writes hidden tests from this frozen spec into `<workspace>\gauntlet\hidden\A43\` before reading any code. The claims lint takes adversarial input and the agent-notice path is a security control, so these vectors are written blind. The builder never reads that folder, and the tests are published into `app/` only after the gate passes ([[App Gauntlet]] §2). There are four hidden suites:
- claims-lint vectors: obfuscated phrases, negation and disclaimer vectors, yields with no basis or date, clean copy that must not be flagged, overlap and offset vectors, and pathological input under a time budget;
- notice-routing vectors: every catalogue type against every preference combination, mandatory types under a muted profile, agent notices with a token as the caller, campaigns against every consent state;
- tasks-from-events vectors under the injectable clock;
- consumer-contract vectors: every consumed event with its required fields present, missing and extra.

This note is not legal advice. The following are rows of versioned data with a `set_with_counsel` marker (HG-LEGAL), so counsel's answers change rows, not code: the phrase lists, the negation list, the yield-basis rule, the 30-day notice period, which announcements count as material, and the sender block for campaigns. RG 234 is the reference the rules cite. Whether a phrase is misleading in context is a human decision recorded on ADM-28. This module never issues that verdict on its own.

## Build brief

**Scope and placement**

- Module `comms` in the API (`services/api/src/modules/comms/`) follows A09's module pattern: public `index.ts` only, zod schemas feeding the OpenAPI document, a Drizzle schema and migration registered through A04's per-module registry, and error codes through `registerErrors()`. Jobs go in `services/worker/src/jobs/notify/`. There are two adapters in A08's shape (`interface.ts`, `sandbox/`, `conformance/`), picked by `ADAPTER_EMAIL` and `ADAPTER_SMS`; `sandbox` is the only value allowed while `RUN_MODE=local`.
- No screens are built here. ACC-03 and the ACC-01 contact flag belong to A59, the SPN-02 "Needs you" list to A62, the SPN-06 marketing copy check to A63, SPN-12 to A64, ADM-28 and ADM-24 to A75, and the public site copy and announcements to A54 and A55. Every read model those screens need is in the OpenAPI document in the first commit (A09 rule), so `packages/sdk` and the MCP catalogue (A104) see it before the engine passes.
- The only source of time is A09's injectable clock. `Date.now()` and argument-less `new Date()` are forbidden in the owned paths, and `npm run verify` greps for them. **Day arithmetic is calendar days in `Australia/Sydney` civil time.** That covers "due in N days", T-3, T-7, T-1, the 30-day notice period and the effective-day notice. Adding N days keeps the local wall-clock time; a time that falls in the DST gap moves forward to the first valid instant. It is never `86,400 × N` seconds and never business days. A13's `businessDaysAfter` is not used by this module.
- Text that people wrote is untrusted input everywhere in this module: announcement bodies, listing copy, template variables, agent names, watch labels. It is normalised, length-capped and rendered through a strict escaper. It is never interpolated into a template as HTML, a log line, an MCP tool description or a regular expression.

**Event contracts consumed (producer events only; no producer calls A43)**

- Every routed notice and every task opens and closes only from an outbox event on A09's outbox, or from this module's own transitions. No other module calls A43 to open or close a task.
- The consumer schema for each event lives in `comms/contracts/consumed/<producer>.ts`. It is a zod object listing only the fields A43 reads, with unknown fields passed through. `notify.dispatch` registers one handler per event name and is idempotent on `event.id`. An event missing a required field goes to A09's dead-letter queue with `COMMS_EVENT_CONTRACT` and raises A13's `job.dead_letter`; nothing is half-delivered.
- The schemas are packaged as `comms/contracts/consumed/*.contract.test.ts` so each producer can run them against its real outbox schema:
  - For a producer that passed before A43 (every AM1 and AM2 ancestor, and any AM3 producer already through its gate), A43's Specify step diffs the producer's frozen event schema against the consumer schema. A missing field is a spec defect reopened upstream.
  - For a producer that passes after A43, the producer's gate re-runs A43's packaged contract test. This must be added to that producer's spec; see the follow-up list.
- Fields common to every event: `event.id`, `occurred_at`, `tenant_id` and `vehicle_id` where they apply. Also `actor {principal_id, on_behalf_of_id?, token_id?}`, which is how an agent's action reaches the "your agent … did X" wording.

| Producer | Event | Fields A43 reads | Routed to |
|---|---|---|---|
| A12 (ancestor) | `agent.token.pending` | `token_id`, `owner_id`, `tenant_id`, `name`, `prefix` | `token.owner_approval_needed` to the tenant's SP-OWNs |
| A13 (ancestor) | `alert.page`, `alert.raised` (`mode: page`, `notify`) | `alert_id`, `type_code`, `severity`, `owner_role`, `title_params` | `alert.page` to on-call members of the owner role (SMS and email) |
| A13 | `alert.digest` | `date`, `role`, `content_ref` | `alert.digest` to the role's members |
| A13 | `investor.certificate_expiring` | `investor_id`, `certificate_id`, `expires_on`, `days_left` (60 or 30) | `certificate.expiring` |
| A14 (ancestor) | `approval.requested`, `approval.approved`, `approval.rejected`, `approval.expired`, `approval.cancelled` | `approval_id`, `action_code`, `maker {principal_id, on_behalf_of_id?}`, `checker_roles[]`, `tenant_id`, `object_ref`, `summary` (display fields), `expires_at` | the `approval.sponsor_decision` task (opened when `checker_roles` holds a sponsor role, closed on the other four); `approval.decided` to the maker's human; `comms.*` effectors |
| A14 | the TTL pre-warning 12 h before `expires_at` | `approval_id`, `checker_roles[]` | `approval.expiring` (requested as A14.1 if A14 does not emit it) |
| A35 (ancestor) | `document.published` | `document_id`, `version`, `vehicle_id`, `kind`, `audience` | `document.published` to holders |
| A28 (AM2) | `notice.requested` (see "Notices other modules need confirmed") | `request_id`, `recipient_id`, `type_code` (`transfer.notice`), `object_ref`, `vars` (earliest execution time) | `transfer.notice` |
| A29 (AM2) | `notice.requested`, or its `recovery.started` / `recovery.vetoable` events | `request_id`, `recipient_id`, `recovery_id`, `veto_until` | `recovery.started`, `recovery.vetoable` |
| A20 | `recon.halt.set`, `recon.halt.cleared` | `halt_id`, `vehicle_id` (never `break_ids` or deltas in any rendered channel) | `recon.hold` to holders |
| A32 | `wholesale.certificate.expired` | `investor_id`, `certificate_id` | `certificate.expired` |
| A33 | `billing.invoice.issued`, `billing.invoice.overdue`, `billing.invoice.paid`, `billing.invoice.voided` | `invoice_id`, `tenant_id`, `due_on`, `amount_cents` | `invoice.issued`, `invoice.overdue` to SP-OWN and SP-FIN; the `invoice.due` task |
| A34 | `property.attestation.opened` | `attestation_id`, `property_id`, `vehicle_id`, `tenant_id`, `period_label`, `deadline_at` | the `attestation.due` task |
| A34 | `property.attestation.attested`, `property.attestation.overdue` | `attestation_id`, `vehicle_id` | closes the task; `property.flag` to holders on overdue |
| A34 | `property.state.changed` (to `distressed` or `legal_hold`) | `property_id`, `vehicle_id`, `to` | `property.flag` to holders (generic wording) |
| A34 | `valuation.due` | `property_id`, `vehicle_id`, `tenant_id`, `next_valuation_due`, `warn_days` (21, A34's data) | the `valuation.due` task |
| A34 | `valuation.submitted`, `valuation.published` | `valuation_id`, `property_id`, `vehicle_id`, `valuation_date` | closes the task; `valuation.published` to holders |
| A36 | `offering.submitted`, `offering.returned`, `offering.approved`, `offering.live`, `offering.extended`, `offering.closed` | `offering_id`, `vehicle_id`, `tenant_id`, `marketing_copy [{item_id, body_sha256}]`, `review_comments_ref`, `g17_counts`, `go_live_approval_id`, `opens_at`, `closes_at`, public listing attributes (`asset_class`, `state`, `property_type`, `min_investment_cents`) | the `review.changes_requested` task; `listing.approved`, `listing.live`; `offer.opening`, `watch.new_listing`, `offer.closing_soon` schedule, `offer.closed`; the `listing_copy` lock and unlock |
| A37 | `orders.placed`, `orders.funded`, `orders.allocated`, `orders.refunded` | `order_id`, `investor_id`, `offering_id`, `requested_units`, `allocated_units`, `amount_cents` | `order.escrowed`, `order.allotted`, `order.scaled_back` (allocated < requested), `order.refunded`; the offer-participant projection behind `offer.closed` |
| A39 | `treasury.withdrawal.requested`, `treasury.withdrawal.settled`, `treasury.withdrawal.held` | `withdrawal_id`, `investor_id`, `amount_cents` (no reason field exists in the contract) | `withdrawal.requested`, `withdrawal.paid`, `withdrawal.under_review` |
| A40 | `distribution.line.paid` | `run_id`, `line_id`, `investor_id`, `net_cents`, `per_unit_display`, `withholding_cents`, `withholding_reason` | `income.paid`; `income.withheld_no_tfn` when the reason is `no_tfn` |
| A40 | `distribution.run.state_changed` (to `executed`) | `run_id`, `vehicle_id`, `tenant_id` | `distribution.paid` to SP-OWN and SP-FIN |
| A48 | `halt.opened`, `halt.cleared` | `halt_id`, `vehicle_id`, `public_reason_code` | `halt.opened`, `halt.closed` to holders |
| A93 (P2) | `vote.opened`, `vote.closing`, `vote.result` | `vote_id`, `vehicle_id`, `closes_at` | the P2 vote types (refused at emit while the P2 flag is off) |

- **Required from producers (follow-ups, not edited here).** A34 must emit `property.attestation.opened`, `valuation.due` and `valuation.submitted` as outbox events. Today it runs the attestation job, raises `valuation.due` as an A13 Sev3 alert, and says it "notifies the sponsor through A43's interface". A33 must emit `billing.invoice.issued` and `billing.invoice.voided`. A39 must emit `treasury.withdrawal.settled` and `treasury.withdrawal.held`. A32 must emit `wholesale.certificate.expired`. A36 must carry `marketing_copy [{item_id, body_sha256}]` and `g17_counts` in the listed events and in the `listing.go_live` payload. A14 must emit `approval.requested` and the TTL pre-warning if it does not already. Every AM3 producer must re-run A43's packaged consumer contract at its own gate. Until an event exists, A43's fixture emitter stands in for it, and the routing is tested against the contract.

**Recipients: resolvers, not reads of other modules**

- **The person:** `recipient_id` in the event or `actor.on_behalf_of_id`, always a human principal.
- **Role members:** members of a staff role, or a tenant's users by sponsor role, come from A11's role assignments (an ancestor), filtered by tenant under RLS.
- **Holders of a vehicle:** an injected `HolderResolver.holdersOf(vehicle_id, as_of) -> investor_id[]`. The legal record is A18's certified register. The real binding is registered by A47, whose read model derives from that register. Until A47 lands, a seeded fixture resolver is used, and its use is written into `app/evidence/A43.json`. Accounts that A11 marks closed are removed after resolution, so a closed account never receives a notice.
- **Partner assignees:** an injected `PartnerAssigneeResolver.engaged(vehicle_id, role: TP-VAL | TP-PM)` for the valuer or property manager engaged on a vehicle. A98 registers it; a fixture is used until then.
- **Watchers:** from this module's own `watch` table (below).
- **Offer participants:** a local projection `comms_offer_participant(offering_id, investor_id)` built from A37's `orders.placed`, used for `offer.closed`.

**Notification catalogue as data**

- The `notification_type` table is seeded from `catalogue.ts`. Its columns are:
  - `code`;
  - `audience`: `investor`, `sponsor_user`, `partner` or `staff`;
  - `category`: `money`, `offer`, `action`, `watch`, `document`, `legal`, `security`, `agent`, `governance`, `platform` or `marketing`;
  - `tone` for the ACC-03 pills: `income` green, `offer` grey, `action` amber, `watch` amber, `security` red;
  - `default_channels[]`, chosen from `in_app`, `email` and `sms` (webhooks belong to A46 and subscribe to the same events);
  - `mandatory`: cannot be muted on any channel the person has verified;
  - `out_of_band_required`: at least one of email or SMS must be attempted whatever the preferences;
  - `requires_marketing_consent`;
  - `template_code`;
  - `target_screen`: a product-map id that ACC-03 deep-links through;
  - `phase`: `P1`, `P2` or `P3`; a P2 type is seeded but refused at emit while A08's phase flag is off;
  - `sponsor_visible`;
  - `dedup_window`;
  - `source_events[]`: the routing table above, validated at boot.
- **Investor types** (P1 unless marked):
  - Offers: `offer.opening` (to `offer_open` watchers of the offering and `sponsor_listing` watchers of its tenant, deduplicated per person and offering), `watch.new_listing` (to `listing_match` and `sponsor_listing` watchers), `offer.closing_soon` (T-3 calendar days before `closes_at`, to the offering's watchers who have not ordered), `offer.closed`.
  - Orders: `order.escrowed`, `order.allotted`, `order.scaled_back`, `order.refunded`.
  - Income: `income.paid` (J06 step 7, "Distribution paid", with the amount, the trust and the per-unit figure), `income.withheld_no_tfn`.
  - Certificates: `certificate.expiring` (T-60 and T-30, from A13's event, not recomputed here), `certificate.expired`.
  - Property and documents: `property.flag` (generic wording), `valuation.published`, `document.published`, `announcement.published`, `announcement.withdrawn`.
  - Legal notices: `terms.new_version` (legal, mandatory), `fee.change` (legal, mandatory).
  - Holds and withdrawals: `recon.hold` ("we are checking this with the trustee", never the delta or the cause), `withdrawal.requested`, `withdrawal.paid`, `withdrawal.under_review` (generic; the template's allowed variables contain no AML field, so no AML reason can be rendered).
  - Security: `transfer.notice` (legal, mandatory), `recovery.started`, `recovery.vetoable` (security, mandatory), `contact.bounced` (security, mandatory; see "Delivery jobs").
  - Halts and votes: `halt.opened`, `halt.closed`, and in P2 `vote.opened`, `vote.closing`, `vote.result`.
  - Marketing: `campaign.message` (category `marketing`, not mandatory, `requires_marketing_consent: true`).
- **Sponsor-user types:** `task.assigned`, `task.due_soon`, `task.overdue`, `review.changes_requested`, `listing.approved`, `listing.live`, `distribution.paid`, `announcement.returned`, `announcement.published`, `invoice.issued`, `invoice.overdue`, `token.owner_approval_needed`.
- **Staff and partner types:** `alert.page` (a Sev1 page from A13, sent by SMS and email to the on-call members of the owner role; the pager is a sandbox), `alert.digest` (A13's 07:00 digest per role, delivered by `notify.dispatch`), `approval.decided` (A14's "maker notified" on approve and reject), `approval.expiring`, `task.assigned`, `task.due_soon`, `task.overdue` (partner users).
- **Agent notice types.** Ten are seeded here so that [[A101 - Agent guard- anomaly detection, sensitive-action notices and tiered kill switches|A101]] emits these strings and nothing else: `agent.token.created`, `agent.token.widened`, `agent.first_use_new_ip`, `agent.money_action` (carries the amount, the cap and the remaining daily allowance), `agent.mandate_near_cap`, `agent.refusal_streak`, `agent.anomaly_hold`, `agent.kill_switch` (per token, user, tenant or platform), `agent.daily_summary` and `agent.sensitive_action`.
  - All ten are `mandatory` and `out_of_band_required`.
  - **A101 decides whether to emit a notice** (its `NoticePolicy`, its daily-summary job, its dedup keys). **A43 only routes what is emitted.** A43 has no job that produces `agent.daily_summary`; the type and its template are seeded here, and A101's `agent-guard.daily-summary` job emits it.
  - A12 emits `agent.token.created` itself on its interim path until A101 takes it over.
- Emitting an unknown type code is refused. A catalogue row whose `template_code` or `source_events[]` does not resolve is a startup error: the registry is validated when the API and worker boot, and a test asserts that every type the module itself emits (including `announcement.withdrawn` and `contact.bounced`) is seeded.

**Preferences, feed and badge**

- `notification_preference` rows are kept per person, per type and per channel, with `enabled`. Absent rows mean the catalogue default.
- `PUT /v1/me/notifications/preferences` accepts a full or partial map.
  - A change that would mute a `mandatory` type on a verified channel returns 422 `COMMS_TYPE_MANDATORY` naming the type.
  - A change to an `out_of_band_required` type that would leave zero out-of-band channels returns 422 `COMMS_OOB_REQUIRED`.
  - The contact channels themselves (email, mobile) are read from A11's verified contact records. Changing them belongs to ACC-01 and needs a passkey. So a token can change preferences but never where notices go.
- The `notification` row holds:
  - `id` (`NTF-<yyyymmdd>-<6 base32>`);
  - `recipient_id`: a human principal, never a token or an agent;
  - `tenant_id`, `type_code`, `category`, `tone`, `title`, `body_text`, `object_ref`, `target_screen`, `template_version`;
  - `emitted_by`: the module, the actor, `on_behalf_of_id`, and `token_id` when an agent's action caused it;
  - `read_at`, `dedup_key`, `created_at`.
- `notification_delivery` rows are kept per channel: `channel`, `state` (`queued`, `sent`, `delivered`, `failed`, `bounced`, `suppressed`), `suppress_reason` (`muted`, `no_marketing_consent`, `unsubscribed`), `provider_ref`, `attempts`, `last_error`, `sent_at`. A `suppressed` row records a muted channel, so an auditor can see the person chose silence. Mandatory types never suppress.
- Feed routes:
  - `GET /v1/me/notifications?unread=&type=&since=` (cursor pagination, newest first) returns the ACC-03 rows: tone pill, title, detail, mono time and target.
  - `GET /v1/me/notifications/badge` returns `{unread}`.
  - `POST /v1/me/notifications/{id}/read` and `POST /v1/me/notifications/read-all` set `read_at` and clear the badge.
  - The empty feed is `{items: [], next: null}`, and the app shows "You're up to date".
- **Token callers.**
  - **Agent notices are hidden from token reads.** When the caller authenticates with a token, `category = agent` rows are left out of the list, the badge count and `read-all`. So an agent cannot learn what the guard reported.
  - A direct `read` on an agent row's id still returns 403 `COMMS_AGENT_CANNOT_ACK_NOTICE`.
  - `security` rows are listed to a token, but a token cannot mark them read. `read-all` under a token skips them and returns `{marked, skipped_protected}`, where `skipped_protected` counts only visible `security` rows. A direct `read` on one returns 403 `COMMS_AGENT_CANNOT_ACK_NOTICE`.
  - Agent and security rows clear only when the person is present with a session or a passkey, and the session badge always counts them.
- Every notification a person receives about an agent's action reads "your agent <token name> (<prefix>) did X on your behalf" and links to ACC-05, SPN-16 or ADM-35. The recipient is always the human behind `on_behalf_of_id`, never the token's own channels (a token has none). A delivery never goes to an address supplied in the triggering request.

**Watch lists ("Notify me")**

- The `watch` table holds `id` (`WCH-...`), `person_id` (the human principal), `kind` and `target`, plus `created_by {principal_id, token_id?}`, `created_at` and `deleted_at`. The kinds and targets are:
  - `offer_open`: an `offering_id`;
  - `listing_match`: criteria `{asset_class?, state?, property_type?, max_min_investment_cents?}`, each field optional and from a closed enum or an integer;
  - `sponsor_listing`: a tenant id of a sponsor with a public profile.
- Routes:
  - `POST /v1/me/watches {kind, target}` is idempotent per (person, kind, canonical target). A repeat returns the existing watch with 200; a new one returns 201.
  - `DELETE /v1/me/watches/{id}` is idempotent: deleting an already-deleted watch returns 204.
  - `GET /v1/me/watches` lists the person's watches.
  - All three are reachable by a token (`watches:write`, `notifications:read`). They are audited as "agent <token> on behalf of <person>" when a token acts. Another person's watch id returns 404.
- The design's `prop` Coming soon button ("We'll email you when this offer opens") creates an `offer_open` watch. `offer.opening` routes to those watchers when A36 emits `offering.live`, and `watch.new_listing` routes when `offering.approved` matches a criteria watch.

**Marketing consent and unsubscribe (Spam Act 2003, APP 7)**

- The `marketing_consent` record holds `person_id`, `channel` (`email`, `sms`), `state` (`granted`, `withdrawn`), `source` (the screen or `unsubscribe_link`), `consent_text_version`, `captured_at` and `captured_by`.
- Routes:
  - `GET /v1/me/marketing-consent` reads the record.
  - `PUT /v1/me/marketing-consent {channel, granted}` changes it. **Granting consent is `x-realx-requires-human: true`**: the person must be present, so an agent cannot opt its human in. **Withdrawing is delegable.**
  - `GET` and `POST /v1/public/unsubscribe/{token}` work without sign-in, using a single-purpose signed token (HMAC through A08's KMS sandbox; person, channel and campaign; no expiry).
  - Every campaign email carries that link and the `List-Unsubscribe` and `List-Unsubscribe-Post` one-click headers. Every campaign SMS carries "Reply STOP to opt out", and the SMS sandbox treats an inbound `STOP` as a withdrawal.
  - A withdrawal takes effect at once, well inside the 5 business days the Act allows, and is audited.
- `campaign.message` is delivered only to recipients whose consent for that channel is `granted` at send time. Everyone else gets a `suppressed` delivery with `suppress_reason` of `no_marketing_consent` or `unsubscribed`, and no message.
- The campaign templates carry a sender block: RealX's legal name, ABN, CAR number, the AFSL holder it acts for, and a contact address. The values come from A52's footer copy as data, and a template test fails if any of them is missing.
- Transactional, legal, security and agent notices are not treated as commercial electronic messages. Whether any of them (for example `watch.new_listing` or `offer.opening`) counts as commercial is a counsel question, recorded under Human gates. The per-type `requires_marketing_consent` flag is the switch counsel's answer flips.

**Adapters: email and SMS with a captured inbox**

- **Email interface** (`packages/adapters/email/interface.ts`):
  - `send({to, template_code, template_version, vars, idempotency_key, tenant_id, headers?}) -> {provider_ref, state}`;
  - `status(provider_ref)`;
  - `handleWebhook(payload, signature)` for bounce and complaint events;
  - `listInbox({to, since})` (sandbox only).
- **SMS interface** (`packages/adapters/sms/interface.ts`): the same verbs for E.164 numbers, with a 320-character body limit and an inbound hook for `STOP`.
- **Captured inbox.** Both sandboxes write every message to `sandbox_message`: `channel`, `to`, `subject`, `text`, `html_sha256`, `template_code`, `vars_sha256`, `headers`, `state`, `scenario`. It is exposed through `GET /v1/_sandbox/inbox?to=&channel=` (local and test modes only; absent from the OpenAPI document elsewhere) and through the ADM-32 sandbox panel. That is what Playwright and the node QA read to prove a notice arrived.
- **Deterministic scenarios by address:**
  - `+bounce@` bounces;
  - `+delay@` is delivered after 90 s of clock time;
  - `+fail@` fails after 5 attempts;
  - `+complaint@` raises a complaint;
  - SMS numbers ending `00` bounce and numbers ending `99` fail.
- A conformance suite in `conformance/` runs against the sandbox now and against a vendor client later (Postmark or SES, Twilio, under HG-VENDOR). The swap changes the client only.
- **Templates** live in `comms/templates/<code>/<version>/` as `subject.txt`, `body.txt` and `body.html`.
  - The engine is a strict Mustache-style one: variables are escaped by default, with no triple-stash, no partials from data and never any user-supplied template text.
  - Each template has an allow-listed variable set, declared in `catalogue.ts`.
  - The HTML follows the design's look: a cream `#F5F2EC` panel on white, ink `#17191C` text, a green `#1F5E4B` button, system-font fallbacks, and the CAR/AFSL footer from A52's footer copy.
  - A plain-text alternative is always rendered.
  - Content limits: first name only; no TFN, wallet address, bank account, AML wording or full document text in any channel; and no amount in an SMS beyond "a distribution was paid".
  - A template test renders every template with every allowed variable set, diff-snapshots the output, and fails on any unescaped `<` or `{{`.
- **Announcement starters** are platform-authored, versioned body skeletons in `comms/starters/<kind>/<version>/body.md`: "Monthly update" for `property_update` and "Significant event" for `significant_event`. They are the SPN-12 editor's templates and are served by `GET /v1/comms/starters?kind=`. They fill only allow-listed variables (property name, period label). A test lints every starter under the current ruleset with zero flags. A starter is a starting text, not an approval: an item created from one is linted and approved like any other.

**Delivery jobs** (`services/worker/src/jobs/notify/`, five jobs, all on the injectable clock)

- **`notify.dispatch`** consumes the events in the contract table and the module's own events.
  - It resolves recipients through the resolvers above, then applies preferences, the mandatory rules and marketing consent.
  - It writes the notification rows and their delivery rows in one transaction.
  - It is idempotent on `event.id` and on each type's `dedup_key`.
  - It also handles `notice.requested` (below).
- **`notify.deliver`** sends each queued delivery through the adapter.
  - It retries after 30 s, 2 min, 10 min, 1 h and 6 h, then marks the delivery `failed`.
  - It marks `bounced` on a webhook.
  - On a hard bounce of a verified channel, it records `contact_delivery_status(person, channel, bounced)` and raises a mandatory `contact.bounced` security notification on the other channel and in-app. The status is readable through `GET /v1/me/notifications/contact-status` for the ACC-01 flag "We couldn't deliver to this address". Investors get no task; tasks are for sponsor and partner users only.
- **`notify.schedules`** handles dated notices:
  - `offer.closing_soon` at T-3 calendar days before `closes_at` (re-planned on `offering.extended`);
  - terms and fee notices on publication, a T-7 reminder, and a notice on the effective day. This is the only notice schedule for terms and fees.
- **`notify.tasks_sweep`** runs every 5 minutes (see "Sponsor and partner tasks").
- **`notify.publish_scheduled`** moves `scheduled` items whose `scheduled_for` has passed to `published`, one transaction per item. It does nothing to an item that has been withdrawn.
- Running any job twice over the same interval changes nothing.

**Notices other modules need confirmed (AM2 producers)**

- The outbox pair for modules that cannot call A43:
  - A producer writes `notice.requested {request_id, recipient_id, type_code, object_ref, vars, requested_by_module}`.
  - `notify.dispatch` creates the notification.
  - When every `out_of_band_required` channel with a verified contact reaches `sent`, or has `bounced` with the fallback channel `sent`, the module writes `notice.confirmed {request_id, notification_id, confirmed_at, channels[]}`.
  - If no channel can be confirmed within the type's window, it writes `notice.failed {request_id, reason}`.
  - Each `request_id` is processed exactly once.
- `comms/bindings/` implements A28's `HolderNotifier` port (`notify`, `status`) and A29's notifier port on top of the same path. They are registered at boot, replacing those modules' sandbox notifiers. A28's packaged port tests (`tokenops/ports/*.contract.test.ts`) and A29's equivalents are re-run against these bindings at A43's gate. This is the swap that A28 §7 and A29 require. Its graph edges are in the graph note.
- For downstream modules (A101, A44 and later), the in-process interface is `comms.notify({recipient_id | role | audience, type_code, object_ref, vars})`. It returns the notification id and its queued deliveries synchronously. `comms.noticeConfirmed(notification_id)` applies the same rule as `notice.confirmed`.

**Sponsor and partner "Needs you" tasks, from events only**

- The `task` row holds:
  - `id` (`TSK-...`), `tenant_id`, `audience` (`sponsor_user`, `partner`);
  - `assignee_role`: SP-OWN, SP-FIN, SP-OPS or SP-IR; TP-VAL or TP-PM for A98's partner users;
  - `assignee_id` (optional), `kind`, `title`, `detail`, `due_at`;
  - `tone`: `green` "Your approval", `amber` "Due in N days", `brick` "Changes requested", `red` when overdue;
  - `target_screen`, `object_ref`;
  - `state`: `open`, `done` or `cancelled`;
  - `overdue`: a flag, not a state;
  - `source_event_id`, `close_event_id`, `dedup_key`.
- Kinds, with the event that opens each and the event that closes it:
  - `approval.sponsor_decision`
    - Opens on A14's `approval.requested` when `checker_roles` holds a sponsor role.
    - Closes on `approval.approved`, `approval.rejected`, `approval.expired` or `approval.cancelled`.
    - Title and target come from a per-action-code data map. Seeded: A40's sponsor sign-off code maps to "Approve {period} distribution · {vehicle}" and SPN-09; `listing.submit` to "Approve listing submission" and SPN-06; A12's sponsor token approval to "Approve team member's API token" and SPN-16.
  - `attestation.due`
    - Opens on A34's `property.attestation.opened`, due at `deadline_at`. It goes to the sponsor role, and to the engaged TP-PM when A98's role table lets them supply the report.
    - Closes on `property.attestation.attested`.
  - `review.changes_requested`
    - Opens on A36's `offering.returned`, carrying the reviewer's comments reference and the G17 counts.
    - Closes on `offering.submitted` for the same offering.
  - `valuation.due`
    - Opens on A34's `valuation.due`, due at `next_valuation_due`. The 21-day warning is A34's `warn_days` data, not a constant here. It goes to the sponsor role and the engaged TP-VAL.
    - Closes on `valuation.submitted`.
  - `announcement.returned`
    - Opens on this module's `return` transition.
    - Closes when the item is next submitted or is withdrawn.
  - `invoice.due`
    - Opens on A33's `billing.invoice.issued`.
    - Closes on `billing.invoice.paid` or `billing.invoice.voided`.
- A task is a read model that points at the screen where the action happens. There is no "complete" endpoint and no `openTask` or `closeTask` interface. A cancelled source cancels the task, and an unknown close event for a task that does not exist is a no-op.
- `notify.tasks_sweep` recomputes `tone` and `detail` from `due_at` in calendar days: "Due in 7 days" becomes "Due in 6 days" at the next local midnight. It also:
  - sets `overdue` at the deadline;
  - writes one `task.overdue` outbox event per task (the consequence "late reports pause new offers from this sponsor" belongs to A34 and A64; this module only reports);
  - emits `task.due_soon` notifications at T-7 and T-1 to the assignees.
- The standard seed reproduces the design's three `spDash` rows exactly:
  - "Approve September distribution · Your approval" (green, `spDist`, SPN-09);
  - "Attest August property report · Due in 7 days" (amber, `spReport`, SPN-10);
  - "Respond to RealX review · Changes requested" (brick, `spNew`, SPN-06);
  - plus the KPI "Reporting 1 due (August rent roll · 30 Sep)", derived from the open `attestation.due` task.
  - The seed raises these through the fixture emitter using the contract events, never by inserting task rows.
- `GET /v1/tasks?role=&state=&tenant=` is RLS-scoped. SP-OWN sees the tenant's tasks, SP-FIN only its own role's, and a partner only its own. `GET /v1/tasks/summary` returns open, overdue and due within 7 days. The empty list renders "Nothing needs you" in the design-extension's TaskRow empty state. Tokens read tasks within their scope. A task never carries anything a token could act on without the target screen's own API and gates.

**Announcements, marketing items and the RG 234 claims lint (G17)**

- The `marketing_item` row holds:
  - `id` (`MKT-...`);
  - `tenant_id`: the platform tenant for RealX's own copy;
  - `kind`: `property_update` (routine), `significant_event` (disclosure-relevant), `campaign`, `site_copy`, `listing_copy`, `terms_version` or `fee_notice`;
  - `audience`: `holders_of_vehicle`, `all_investors`, `public` or `tenant_users`;
  - `vehicle_id` (required for holder audiences), `offering_id` (for `listing_copy`), `copy_key` (for `site_copy`, for example `home.hero`);
  - `title`, and `body_md`: a markdown subset of paragraphs, emphasis, lists, and links to realx.au or A35 document ids only; at most 20,000 characters (413 `COMMS_BODY_TOO_LONG`); no raw HTML (`COMMS_LINK_NOT_ALLOWED` for any other link);
  - `attachment_doc_ids[]` (A35), `linked_doc_versions[]` (the IM version or versions a `listing_copy` report covers), `terms_doc {document_id, version, document_sha256}` (for `terms_version`), `parent_approval_id` (for `fee_notice`);
  - `material`: set by the sponsor; RealX may raise it but never lower it;
  - `claims_report_id`, `claims_ruleset_version`, `body_sha256`, `approval_id`, `scheduled_for`, `effective_from` (for terms and fees), `published_at`, `withdrawn_at`, `withdraw_reason`, `state`;
  - `authors[]`: every human principal who created or edited the item. An agent's edit adds its human (`on_behalf_of_id`); each edit also records the token id;
  - `submitted_by`, `starter_code`.
- **Editing.** An item can be edited (`PATCH`) only in `draft`. An edit in `checked` or `changes_requested` returns the item to `draft` and invalidates its report. An edit in `submitted` or any later state returns 409 `MKT_ILLEGAL_TRANSITION`. So an approved text can never change under its approval.
- **Local lifecycle `MKT`.** It is documented in the module and is not one of SM-00..13. The full transition table:

| From | To | By |
|---|---|---|
| `draft` | `checked` | `check` |
| `checked`, `changes_requested` | `draft` | an edit by an author |
| `checked` | `submitted` | `submit` |
| `submitted` | `draft` | `recall` by the submitter, before review starts |
| `submitted` | `in_review` | the A14 request is opened: at `submit` for every kind except a sponsor tenant's `significant_event`, which IRCM opens with `request-approval` |
| `in_review` | `approved` | A43's A14 effector on approval |
| `in_review` | `changes_requested` | `return` by CCO or LEGL, which cancels the open A14 request |
| `in_review` | `rejected` | A14 rejection |
| `in_review` | `draft` | A14 expiry, or a cancellation other than `return` |
| `approved` | `published` | the effector, when `scheduled_for` is empty or already passed |
| `approved` | `scheduled` | the effector, when `scheduled_for` is in the future |
| `scheduled` | `published` | `notify.publish_scheduled` |
| `approved`, `scheduled` | `withdrawn` | the maker's human or CCO |
| `published` | `withdrawn` | CCO, with a reason; the published text is kept and shown as withdrawn, and a corrective item may be required |

  Every other pair returns 409 `MKT_ILLEGAL_TRANSITION`.
- **Listing copy has its own lifecycle, and A43 never publishes it.** `listing_copy` uses the sub-lifecycle `draft -> checked -> locked`, `locked -> draft` and `locked -> approved`:
  - `locked` is set when A36's `offering.submitted` names the item with a matching `body_sha256`.
  - `locked -> draft` happens on `offering.returned`.
  - `locked -> approved` happens on `offering.approved`, recording A36's `go_live_approval_id` for display after the payload hash is checked against `body_sha256`.
  - The item is approved by A36's `listing.go_live`. Its payload must include each copy item's `body_sha256` and `claims_report_id`. `publish` and `submit` on a `listing_copy` item return 409 `COMMS_KIND_NOT_PUBLISHABLE_HERE`.
- **The `check` transition** runs `checkClaims` on the exact `body_sha256` and stores the report.
- **The `submit` transition** refuses if:
  - the stored report is for another hash or an older ruleset version: `COMMS_REPORT_STALE`;
  - any `block` flag exists: `gate_failed` with `gate: G17`, and `reason_codes` = the rule codes;
  - the notice period is short: `COMMS_NOTICE_PERIOD_SHORT`;
  - a sponsor rule below is broken.
- **Review flags do not block submission; they block approval.** A43 registers a precheck through A14's `registerPrecheck` on every `comms.publish_*` action code. It re-lints the exact `body_sha256` against the current ruleset, and it fails while any `block` flag or any unresolved `review` flag exists (`gate_failed` G17), or if the hash differs from the request payload. Only CCO or LEGL resolve a review flag, with a reason; the sponsor cannot.
- **Approval rows** are data in A14's policy table, seeded here. All use existing A14 policy codes and no `AUTO` condition.
  - `comms.publish_routine` (`property_update`, not `material`)
    - Policy `4E`, checker set `[[IRCM]]`, with a hardware-key assertion through A53's approval action pattern.
    - The maker is the sponsor submitter; a sponsor agent may be the maker, and A14's SoD counts it as its human.
    - This is the product map's "Publish routine property update (IRCM)": IRCM decides it as a person, and the routine update still carries a human approval record.
  - `comms.publish_announcement` (`significant_event`)
    - `4E`, IRCM maker, CCO checker; `6E` adding LEGL when `material`.
    - For platform items, the IRCM submitter is the maker. For a sponsor's item, IRCM opens the request with `request-approval` after reading it.
  - `comms.publish_campaign`: `4E`, MKTG maker, CCO checker.
  - `comms.publish_site_copy`: `4E`, MKTG or IRCM maker, CCO checker.
  - `comms.publish_terms_version`
    - `4E`, LEGL maker, CCO checker.
    - The payload hashes `body_sha256`, `terms_doc.document_sha256` and `effective_from`.
    - The rule is `effective_from ≥ approval instant + notice_days` (30, `set_with_counsel`, calendar days in `Australia/Sydney` as defined above), checked at submit and again in the precheck at decision time; otherwise 422 `COMMS_NOTICE_PERIOD_SHORT`.
    - One approval covers the notice and the document version. On approval, A43's effector calls A35's publish interface for `terms_doc` with this `approval_id` as its authority (A35.1 must accept it), so there are never two linked approvals.
  - `comms.publish_fee_notice`
    - `4E`, IRCM or LEGL maker, CCO checker.
    - `parent_approval_id` must name an approved fee-schedule change: M27's code or A33's `tenant.fee_schedule.set`, whose `effective_from` must equal the notice's.
    - The same notice period applies.
  - `comms.ruleset.change`: `4E`, CCO maker, LEGL checker.
  - `listing_copy` has no `comms.*` row. It is approved by A36's `listing.go_live`, where A43 registers a precheck through `registerPrecheck('listing.go_live', …)`. It fails while any copy item or linked IM version named in the payload has an open G17 flag, or a `body_sha256` that differs from the copy item's.
  - **Public KPIs are not A43's.** A45 owns `metrics.publish` (6E, IRCM → CFO + CCO) and `GET /v1/metrics/public`. A43 registers a precheck on `metrics.publish` that runs `checkClaims` with context `public_metric` over the rendered strings in the request (block severity).
- **Sponsor rules.**
  - SP-IR, SP-OPS and SP-OWN may draft and check.
  - SP-IR and SP-OWN may submit. J19 step 5 is SP-IR writing and submitting the monthly update.
  - Where `material`, the submitter's human principal must not be in `authors[]` (`COMMS_SECOND_HUMAN_REQUIRED`). Drafter and submitter are compared by human principal, so SP-OWN cannot submit a material item that SP-OWN's own agent drafted.
  - A sponsor agent may draft, check and submit routine items, and is refused on material items with `COMMS_MATERIAL_NEEDS_HUMAN`.
  - A staff agent (IRCM's or MKTG's) may draft, submit and open requests as maker. A14 refuses it as a checker.
- **Nothing publishes without an approval.** Publication reads A14. It refuses with 403 `COMMS_NO_APPROVAL` unless `approval_id` is `approved` or later and the approved `payload_sha256` covers the current `body_sha256`. Public and investor read routes serve only `published` and `withdrawn` items; any other state is 404 there, never 403.

**The claims lint, precisely**

- `checkClaims(fields[], ruleset, context) -> ClaimsReport` in `comms/claims/` is a pure function with no I/O. `fields[]` is `{field, text, page?}`. `context.kind` is an item kind, or `im_text` or `public_metric`.
  - A35's `getDocumentText(doc_id, version)` supplies IM pages as fields `{field: 'im', page}`. That is how the design's "guaranteed income on IM p.3" is produced.
  - `claims_ruleset` is a versioned table (`version`, `effective_from`, `set_with_counsel`, `rules[]`), never edited in place. Each report stores the version it used.
- Each rule has:
  - `code`;
  - `severity_by_kind`: a map from kind to `block`, `review` or `info`, with a `default`;
  - `phrases[]` or one bounded structural check;
  - `negatable` (default true);
  - `rule_text`: the sentence the sponsor sees, citing RG 234;
  - `suggested_wording`;
  - `applies_to` kinds.
- **Seeded severities.** The guarantee and comparison phrases below default to `block`, and are `review` for `listing_copy` and `im_text`. A listing is submitted with review flags, as the design shows, and they block go-live through A36's precheck. `CLAIM_YIELD_NO_BASIS` follows the same map. Review phrases are `review` everywhere.
- **Seeded phrases.**
  - `CLAIM_GUARANTEE`: "guaranteed", "guaranteed income", "guaranteed return", "capital guaranteed".
  - `CLAIM_CAPITAL_PROTECTED`: "capital protected".
  - `CLAIM_NO_RISK`: "risk free", "no risk" (for "no risk", the negator is part of the phrase, so it is not negatable).
  - `CLAIM_MARKET_COMPARISON`: "beats the market", "outperforms the market".
  - `CLAIM_REGULATOR_ENDORSEMENT`: "government backed", "ASIC approved", "ASIC endorsed".
  - Review phrases in `CLAIM_SAFETY_WORDING`: "secure", "safe", "safe as houses", "stable income", "passive income", "set and forget", "like a term deposit", "as safe as a bank deposit", "backed by real estate". The last is a review phrase because literal matching cannot tell a safety claim from a description of the asset. A reviewer decides.
- **Normalisation**, with an index map back to the original:
  1. NFKC.
  2. Case folding.
  3. A confusables map for common Latin-lookalike homoglyphs (Cyrillic, Greek, fullwidth).
  4. Removal of zero-width characters (U+200B-U+200D, U+2060, U+FEFF) and soft hyphens (U+00AD).
  5. Hyphens, dashes and non-breaking hyphens replaced by a space.
  6. **Spacing collapsed only inside a run of three or more single-letter tokens.** "G U A R A N T E E D" becomes "guaranteed", but "a big deal" and ordinary words are untouched.
- **Matching.**
  - Phrases match on word boundaries of the normalised text, so "safeguard", "secured", "unsafe" and "insecure" never match "safe" or "secure".
  - Between a phrase's words, the text may contain whitespace, or nothing at all (so "riskfree" matches "risk free").
  - Phrase lists compile to a literal-alternation matcher: no user-controlled regex and no nested quantifiers. A test proves that 20,000 characters of pathological input lint in under one second.
  - Nothing is ever decoded. A base64 string is not text.
- **Negation and disclaimers.** These are checked before a phrase flags. A `negatable` phrase does not flag when:
  - a negator ("not", "no", "never", "cannot", "can't", "isn't", "aren't", "wasn't", "nor", "neither", "without") appears within the 3 tokens before the phrase's first token, in the same sentence; and
  - no anti-negator ("not only", "not just", "no doubt", "no less than", "never been more") and no contrast word ("but", "yet", "however") stands between the negator and the phrase.
  - A disclaimer allow-list of whole sentences is also exempt: "returns are not guaranteed", "your capital is not guaranteed", "this investment is not capital protected", "it is not risk free", "unlike a term deposit", "not a bank deposit", "not approved or endorsed by ASIC".
  - Test vectors frozen here:

| Text | Expected |
|---|---|
| "Returns are not guaranteed." | no flag |
| "This is not capital protected." | no flag |
| "It is not risk-free." | no flag |
| "Unlike a term deposit, your capital is at risk." | no flag |
| "Not only safe but guaranteed." | two flags ("safe" review, "guaranteed" block) |
| "Income is guaranteed, not variable." | one flag |
| "There is no risk." | one flag (`CLAIM_NO_RISK`) |
| "We safeguard documents with secured storage." | no flag |

- **Overlaps.** The longest match wins, and overlapping or nested matches give one flag. "guaranteed income" gives one `CLAIM_GUARANTEE` flag for the whole phrase, not two. When two rules flag the same span, the higher severity wins, and then the earlier rule in the ruleset order. For example, `CLAIM_YIELD_NO_BASIS` suppresses `CLAIM_FORECAST_NO_DISCLAIMER` on the same percentage.
- **Offsets.** `offset` and `length` are UTF-16 code units in the original field text as stored, not the normalised text. A span covers the original characters from the first matched character to the last, including any zero-width or hyphen characters inside it. For IM pages, the offset is within that page's text.
- **Structural rules.**
  - **`CLAIM_YIELD_NO_BASIS`.**
    - Trigger: in any sentence, a percentage token (a number followed by "%", "per cent" or "percent") that appears together with a trigger word: "yield(s)", "return(s)", "income", "distribution(s)", "target(s/ed/ing)", "forecast(s/ed)", "aim(s/ed)".
    - The sentence must also contain both a basis token ("net", "gross", "p.a.", "per annum", "annualised", "before fees", "after fees", "pre-tax", "after tax") and a date or period ("as at <date>", a date such as "30 Jun 2026" or "30/06/2026", a month and year, "FY24", "FY24-FY26", a year range, "for the year to", "over the N years to").
    - The flag spans the percentage token.
    - Sentences split on `.`, `!` or `?` followed by whitespace and then an uppercase letter or a digit, or at the end of the text. They never split after "p.a.", "e.g.", "i.e.", "etc.", "approx.", "Pty.", "Ltd.", "No.", "vs.", "Mr.", "Ms.", "Dr.", "St." or a month abbreviation, and never inside a decimal.
  - **`CLAIM_FORECAST_NO_DISCLAIMER`**: "will", "projected" or "expected" in the same sentence as a figure, without "illustrative only, not a forecast"; `review`.
  - **`CLAIM_PAST_PERFORMANCE`**: a returns history without the past-performance sentence; `review`.
  - **`CLAIM_EXTERNAL_LINK`**: a link outside realx.au in listing copy or IM text; `review`. Body links are refused outright in items.
- **The report** has the shape the design shows:
  - `flags[] {flag_id, rule_code, severity, phrase (original text of the span), location {field, offset, length, page?}, rule_text, suggestion, resolved {by, reason, at}?}`;
  - `counts {block, review, info}`;
  - `summary` and `tone`.
- **The summary string for every combination of counts** (singular with a count of 1: "1 phrase"):

| Counts | Summary | Tone |
|---|---|---|
| none | "No issues found" | green |
| info only | "No issues found · {i} note(s)" | green |
| review only | "{r} phrase(s) flagged for review" | amber |
| block only | "{b} phrase(s) blocked" | red |
| block and review | "{b} blocked · {r} flagged for review" | red |

  Resolved review flags are not counted. The Marine Parade seed gives **two review flags**, so the summary is "2 phrases flagged for review", in amber.
- `POST /v1/comms/claims/check` runs a dry run for SPN-06, SPN-12 and the ADM-28 tester without creating an item. It is rate-limited per principal by A09's hook.
- Every stored report writes a `claims.report.updated {item_id, offering_id?, report_id, counts, summary}` outbox event, which A36's review read model and A19's G17 fact can consume.

**Routes**

All writes accept an `Idempotency-Key` (A09's hook) and carry `x-realx-scope`. Human-only routes carry `x-realx-requires-human: true`, so A12 refuses a token before the scope check.

| Method and path | Who | Scope | Agent | Errors |
|---|---|---|---|---|
| `POST /v1/comms/items` | IRCM, MKTG, LEGL (terms, fees); SP-IR, SP-OPS, SP-OWN (tenant kinds) | `comms:draft` | delegable | `COMMS_BODY_TOO_LONG`, `COMMS_LINK_NOT_ALLOWED`, `COMMS_KIND_NOT_ALLOWED` (403) |
| `PATCH /v1/comms/items/{id}` | the item's roles | `comms:draft` | delegable | `MKT_ILLEGAL_TRANSITION`, as above |
| `POST /v1/comms/items/{id}/check` | the item's roles | `comms:check` | delegable | `MKT_ILLEGAL_TRANSITION` |
| `POST /v1/comms/items/{id}/submit` | SP-IR, SP-OWN; IRCM, MKTG, LEGL per kind | `comms:submit` | delegable, except sponsor agents on material items | `COMMS_REPORT_STALE`, `gate_failed` G17, `COMMS_SECOND_HUMAN_REQUIRED`, `COMMS_MATERIAL_NEEDS_HUMAN`, `COMMS_NOTICE_PERIOD_SHORT`, `COMMS_KIND_NOT_PUBLISHABLE_HERE`, `MKT_ILLEGAL_TRANSITION` |
| `POST /v1/comms/items/{id}/recall` | the submitter | `comms:submit` | delegable | `MKT_ILLEGAL_TRANSITION` |
| `POST /v1/comms/items/{id}/request-approval` | IRCM (a sponsor's `significant_event`) | `comms:review` | delegable, as maker only | `MKT_ILLEGAL_TRANSITION` |
| `POST /v1/comms/items/{id}/return` | CCO, LEGL | `comms:review` | human only | `MKT_ILLEGAL_TRANSITION`, `COMMS_COMMENT_REQUIRED` |
| `POST /v1/comms/items/{id}/flags/{flagId}/resolve` | CCO, LEGL | `comms:review` | human only | `COMMS_FLAG_NOT_RESOLVABLE` (block flags), `COMMS_REASON_REQUIRED` |
| `POST /v1/comms/items/{id}/materiality` | IRCM, CCO, LEGL | `comms:review` | human only | `COMMS_MATERIALITY_CANNOT_LOWER` |
| `POST /v1/comms/items/{id}/withdraw` | the maker's human, or CCO before publication; CCO after publication | `comms:withdraw` | delegable for the maker before publication; human only after | `MKT_ILLEGAL_TRANSITION`, `COMMS_REASON_REQUIRED` |
| `POST /v1/comms/rulesets` | CCO | `comms:ruleset` | delegable, as maker only | opens A14 `comms.ruleset.change` |
| `POST /v1/comms/claims/check` | any comms role | `comms:check` | delegable | rate limit |
| `PUT /v1/me/notifications/preferences` | the person | `notifications:prefs` | delegable | `COMMS_TYPE_MANDATORY`, `COMMS_OOB_REQUIRED` |
| `POST /v1/me/notifications/{id}/read`, `/read-all` | the person | `notifications:ack` | delegable, except agent and security rows | `COMMS_AGENT_CANNOT_ACK_NOTICE` |
| `POST /v1/me/watches`, `DELETE /v1/me/watches/{id}` | the person | `watches:write` | delegable | `COMMS_WATCH_TARGET_NOT_FOUND` (404) |
| `PUT /v1/me/marketing-consent` | the person | `notifications:prefs` | grant is human only; withdraw is delegable | — |
| `GET`, `POST /v1/public/unsubscribe/{token}` | anyone holding the link | none | n/a | `COMMS_UNSUBSCRIBE_TOKEN_INVALID` |

- When a raise of materiality lands on an open `4E` request, that request is cancelled and a new one is opened under `6E`.
- **Reads:**
  - `GET /v1/comms/items?state=&kind=&tenant=` is the ADM-28 queue (tenant, kind, material, flag counts, age, maker) and the SPN-12 list.
  - `GET /v1/comms/items/{id}` returns the item, its report with offsets, the A14 approval state, the audit refs and **`allowed_actions[]`**: the actions this caller may take now, computed server-side from the state, the role, the second-human rule and whether the caller is a token. So A64 and A75 decide no button in the client.
  - `GET /v1/comms/items/summary` returns counts by state for the badges (`in_review` for ADM-28).
  - Also: `GET /v1/comms/rulesets/current`, `GET /v1/comms/rulesets` (history), `GET /v1/comms/templates`, `GET /v1/comms/starters`.
  - Public: `GET /v1/public/announcements?vehicle=&kind=` (published `significant_event`, public `property_update` and public `fee_notice` for the property page, PUB-06 and PUB-09), `GET /v1/vehicles/{id}/announcements` (holders), `GET /v1/public/site-copy/{copy_key}` (the published `site_copy` for A54 and A55).
  - The person's own: `GET /v1/me/notifications`, `/badge`, `/contact-status`, `GET /v1/me/watches`, `GET /v1/me/marketing-consent`, `GET /v1/tasks`, `/v1/tasks/summary`.
  - In local mode only: `POST /v1/_test/comms/emit` raises any type or consumed event, so e2e and the seed can drive every route without the producer.
- **Publication effects**, in one transaction plus outbox events:
  - `published_at` is set.
  - `announcement.published` goes to the audience through `notify.dispatch`.
  - A `campaign` is sent as `campaign.message` to consenting recipients in its audience only.
  - `site_copy` replaces the served copy for its `copy_key` atomically.
  - `terms_version` and `fee_notice` send `terms.new_version` or `fee.change` to every open account, with `effective_from` in the body, and plan the T-7 and effective-day notices.
  - Withdrawal sends `announcement.withdrawn` to the same audience, or restores the previous site copy for a `site_copy` key.

**Permissions, agents and audit**

- Actions go in the domain matrix under M21: `comms:read`, `comms:draft`, `comms:check`, `comms:submit`, `comms:review`, `comms:withdraw`, `comms:ruleset`, `notifications:read`, `notifications:ack`, `notifications:prefs`, `watches:write`, `tasks:read`. Approvals use A14's `decide`.
  - IRCM and MKTG draft and submit platform items.
  - CCO and LEGL review, return, resolve review flags and withdraw.
  - SP-IR, SP-OPS and SP-OWN act within their tenant under RLS.
  - TP-LIC reads its licensee's tenant items and rulesets only.
  - Investors have no `comms:*` scope.
- Every route carries `x-realx-scope`, so A12's `agentAuth` mirrors the person's rights on tokens. A104 lists the tools for reads, draft, check, submit, recall, request-approval, maker withdraw, watches, preferences and consent withdrawal. It lists none of the human-only routes and none of the approval tools.
- Audit goes through A10's `audit.record` inside the same transaction, exactly one event for each of:
  - an `MKT` transition (`comms.item.<transition>`, with `approval_id` where one exists);
  - a flag resolution, a return, a materiality raise and a ruleset change;
  - a watch created or deleted, and a consent change;
  - a refused preference change on a `mandatory` type (still logged as `gate.refused`);
  - each delivery of a `legal`, `security` or `agent` notice (`notice.sent` with `notification_id`, channel and `provider_ref`), because A28's `TOP_NOTICE_NOT_SENT`, A29's veto window and D04's notice clause rely on that record.
  - When a token acts, the event reads "agent <token> on behalf of <person>". Ordinary notifications are their own record and write no audit rows, because of volume. `notification.created` and `notification.delivered` outbox events exist for A46's webhooks.
- **Errors** are registered with A09, as stable strings in the OpenAPI document:
  - `COMMS_TYPE_MANDATORY`, `COMMS_OOB_REQUIRED`, `COMMS_AGENT_CANNOT_ACK_NOTICE`;
  - `COMMS_REPORT_STALE`, `COMMS_NO_APPROVAL`, `COMMS_NOTICE_PERIOD_SHORT`, `COMMS_SECOND_HUMAN_REQUIRED`, `COMMS_MATERIAL_NEEDS_HUMAN`;
  - `COMMS_BODY_TOO_LONG` (413), `COMMS_LINK_NOT_ALLOWED`, `COMMS_TEMPLATE_VAR_NOT_ALLOWED`;
  - `COMMS_KIND_NOT_ALLOWED`, `COMMS_KIND_NOT_PUBLISHABLE_HERE`, `COMMS_FLAG_NOT_RESOLVABLE`, `COMMS_MATERIALITY_CANNOT_LOWER`, `COMMS_REASON_REQUIRED`, `COMMS_COMMENT_REQUIRED`;
  - `COMMS_WATCH_TARGET_NOT_FOUND`, `COMMS_UNSUBSCRIBE_TOKEN_INVALID`, `COMMS_EVENT_CONTRACT`;
  - the `CLAIM_*` rule codes, surfaced under `gate_failed` with `gate: G17`;
  - `MKT_ILLEGAL_TRANSITION` (409).
  - Unknown ids are 404. A tenant user asking for another tenant's item or task gets 404, never 403.

## Owned paths (other nodes do not edit these)

- <workspace>\app\services\api\src\modules\comms\
- <workspace>\app\packages\adapters\email\
- <workspace>\app\packages\adapters\sms\
- <workspace>\app\services\worker\src\jobs\notify\

## Design references

- **`design:adListing G17 flagged phrases`** in `<workspace>\design\RealX App.dc.html` (screen key `adListing`, product-map ADM-07, admin dark shell).
  - The "Gates" TaskRow reads "Marketing claims (G17) · “guaranteed income” on IM p.3 · “beats the market” on listing copy · 2 flags" (amber pill). Its click toast is "Changes requested from Harbour & Co".
  - Together they fix the report shape this module returns: two flags, each with the exact phrase in typographic quotes, a location that names the document page or the field, and a count in the pill.
  - `adApprovals` APR-2290 "Marketing claims (G17) 2 flags" (amber) is the same report, read through A14's checks list on `listing.go_live`.
  - Both flags have `review` severity (the listing-copy and IM severity map). The seed reproduces both phrases, their locations and the count.
- **`design:spNew marketing copy check`** (screen key `spNew`, step 4 "Documents & submit", product-map SPN-06, sponsor light shell).
  - The CheckRow reads "Marketing copy check (G17) · 2 phrases flagged for review" (amber `Review` pill). It is followed by "Sponsor owner approval (G13) · Mia Chen · Required" (grey) and the CTA "Submit to RealX".
  - The summary string, the amber tone for review flags and the position before the G13 row are this module's `summary`, `tone` and `counts`.
  - Review flags do not stop submission and are cleared before go-live, which is the design's state. Listing copy never carries `block` severity, so A63's "block submission while a block flag exists" rule does not arise for listings.
- **`design:spDash Needs you tasks`** (screen key `spDash`, product-map SPN-02).
  - The TaskRows sit under the h2 "Needs you", in the sponsor variant grid `1fr auto`: title and description, pill at the right, hover border ink `#17191C`.
  - "Approve September distribution · Bay Road Apartment": pill "Your approval", green tone `#E3EEE8` / `#1F5E4B`, click → `spDist`.
  - "Attest August property report": pill "Due in 7 days", amber `#F6ECD6` / `#7A4F00`, → `spReport`.
  - "Respond to RealX review · Marine Parade Apartment": pill "Changes requested", brick `#F5E4DB` / `#9A4524`, → `spNew`.
  - The KPI tile reads "Reporting · 1 due · August rent roll · 30 Sep".
  - Rows, tones, targets and the tile are read models of the `task` table. The seed reproduces them exactly. The overdue red tone (`#F6E1DE` / `#A3312A`) is added by this node's rule.
- **Investor-side copy the notifications inherit:**
  - the `prop` Coming soon toast "We'll email you when this offer opens" (an `offer_open` watch);
  - the aside line "Offer closes {date}" (`offer.closing_soon`);
  - the `activity` and `wallet` rows "Distribution · Paid" (`income.paid` wording, amount format `en-AU` to 2 dp);
  - the `buy` step 3 timeline captions (`order.escrowed`, `order.allotted`, `order.refunded`).
- **Product-map SPN-12 mock:** the banner "Claims check: no issues found." (green) is the clean `summary`. The components "Editor with templates (monthly update, significant event)" are the two starters.
- **Undesigned screens:**
  - ACC-03 Notifications (TaskRows with tone pills: income green, offer grey, action amber, watch amber; a preferences CheckList; the marketing-consent toggles) and SPN-12 Announcements (TextField; TextArea 140 px; a claims-check banner with the flagged phrase highlighted and the rule beside it; Save draft and Submit for approval) are drawn in [[A50 - Design extension A- investor and sponsor screens not in the prototype|A50 - Design extension A: investor and sponsor screens not in the prototype]].
  - ADM-28 Marketing compliance is drawn in [[A51 - Design extension B- operator console screens not in the prototype|A51 - Design extension B: operator console screens not in the prototype]]: a split DataTable and detail with the flagged phrase, an RG 234 red-tint banner `#F6E1DE`, Return and Suggest wording, the claims library, templates and terms versions.
  - This node fixes the field names, tones, flag shape, summary strings and action verbs, so those files draw them and A59, A64 and A75 render them without translation. Email templates use the same tokens and the CAR/AFSL footer copy from the design's `Footer`.

## Requirements covered

- **Product-map screens:** none owned. ACC-03 and the ACC-01 contact flag (A59), SPN-02 tasks (A62), the SPN-06 claims row (A63), SPN-12 (A64), ADM-28 and the ADM-24 publication notices (A75), the ADM-07 G17 row, and the public announcements and site copy (A54, A55) consume this module. "Done" for those screens means every row, pill, count, flag, preference and allowed action they show is a field of this API, so they render without adding logic.
- **Journeys:** none owned. These steps run through `notify.dispatch`, the `MKT` lifecycle and the seeded approval rows:
  - J06 step 7 ("Distribution paid" notification);
  - J12 step 10 (listing page and claims approved, through A36's `listing.go_live` with A43's precheck);
  - J14 step 6 (holder disclosure: IRCM opens the request and CCO decides);
  - J19 step 5 (SP-IR writes the monthly update from the starter, checks it and submits it);
  - J21 step 3 (notice to holders on a halt or wind-down).
  - "Done" means each step can be driven at service level with the seeded personas, and the notice is visible in the sandbox inbox.
- **Gate G17, Marketing claims approved.** Done when:
  - every item is linted on its exact body hash against a versioned ruleset;
  - a `block` flag refuses submission with `gate_failed` and the rule codes;
  - an unresolved `review` flag refuses approval through A43's A14 precheck;
  - listing copy and IM text are linted by the same function, and A36's `listing.go_live` is refused by A43's precheck while a flag is open;
  - no announcement, campaign, site copy, terms version or fee notice is served or sent without an `approval_id` in state `approved` or later that covers its hash.
- **Gate G13, Maker-checker.** Done when:
  - the seeded rows route routine updates through IRCM as a human checker, announcements through IRCM → CCO (+ LEGL when material), campaigns and site copy through MKTG or IRCM → CCO, terms and fee notices through LEGL or IRCM → CCO with the notice period enforced, and rulesets through CCO → LEGL;
  - the sponsor second-human rule holds for material items, counting an agent as its human;
  - agents are refused as checkers by A14, and as submitters of sponsor material items and callers of the human-only routes here.
- **Module M21, Communications and IR.** Done when these hold:
  - the D14 behaviours of the requirements catalogue: notifications by type and channel with preferences and "mark all read"; sponsor monthly updates and significant-event notices; platform notices and campaigns; the claims library and approval workflow under RG 234; terms versions with 30-day notice;
  - N13 (idempotent, replayable jobs) and N17 (fee and terms changes carry an effective date and 30-day notice);
  - D04's notice clause (agent notices go to the person, never to the agent; they cannot be muted by the agent or read through its token);
  - Spam Act consent and unsubscribe for campaigns.
- **State machines:** none of SM-00..SM-13. The `MKT` lifecycle (including the `listing_copy` sub-lifecycle), the notification delivery states and the task states are local to this module, and their transition tables are tested exhaustively. "Done" means every allowed transition works and every other pair is refused.

## Gauntlet loop for this node

- **Specify.** The frozen spec is this note plus:
  - the D14 and D17 rows of the requirements catalogue;
  - the three design regions above;
  - A14's policy-row format, its existing policy codes, `registerPrecheck`, effector registration and the request lifecycle events;
  - A13's alert, digest and certificate event contracts;
  - A35's document-text and publish interfaces;
  - A12's outbox event names;
  - A11's role-assignment and verified-contact reads;
  - A28's `HolderNotifier` port and A29's notifier port with their packaged port tests;
  - **the "Event contracts consumed" table**. For every producer that has already passed its gate, its frozen event schema is diffed against A43's consumer schema, and any missing field is reopened upstream before the hidden tests are written.
  - The node QA writes the four hidden suites to `<workspace>\gauntlet\hidden\A43\` first: Vitest over the pure lint and routing functions, plus the module's service layer against the Docker Postgres from A01, with the injectable clock, the fixture resolvers and the sandbox inbox.
  - Any change after the freeze to the ruleset semantics, the matcher rules, the mandatory-type rule, the consumer contracts or the approval routing is a spec defect, not a fix.
- **Build (Grok 4.7 xhigh, split by the planner).** Four parallel sub-agents work on disjoint files, then one integration pass.
  1. The pure parts: `catalogue.ts`; the `MKT`, listing-copy, delivery and task transition tables; the normaliser with its index map and the confusables map; the phrase matcher with negation, overlap and offset rules; the sentence splitter and structural rules; the summary table; the template engine and its escaper. Each has unit and property tests.
  2. The adapters: email and SMS interfaces, sandboxes with scenarios, inbound `STOP` and the captured inbox, conformance suites, the local inbox route and the ADM-32 panel hooks.
  3. The API:
     - the Drizzle schema and migration (types, preferences, notifications, deliveries, contact status, watches, marketing consent, tasks, offer participants, marketing items, claims reports, rulesets, sandbox messages);
     - repositories under RLS, and permission checks against the domain matrix;
     - every route in the routes table, with zod schemas and `allowed_actions`;
     - the A14 policy-row seed, the prechecks (`comms.publish_*`, `listing.go_live`, `metrics.publish`) and the effectors;
     - the module interface, the A28 and A29 port bindings, the unsubscribe tokens;
     - OpenAPI generation and an SDK compile check.
  4. The worker: the five `notify.*` jobs on the injectable clock, the consumer schemas and handlers for every consumed event, the `notice.requested` pair, the resolvers with fixtures, and the seed that reproduces the `spDash` tasks and the `adListing` flags through the fixture emitter.
  - **Integration:**
    - a fixture emitter raises every consumed event and every catalogue type for one principal of every audience;
    - a scenario drafts, checks, submits, approves through A14's software authenticator, publishes and withdraws one item of every publishable kind, and locks and approves one `listing_copy` through a fixture `offering.*` sequence;
    - an end-to-end run advances the clock through T-7, T-1, overdue, a scheduled publication and a 30-day terms notice;
    - then `npm run verify`, and `app/evidence/A43.json` naming the resolver bindings in use.
- **Challenge (node QA, Opus 5.5 high).** The node QA runs the hidden suites, then:
  - the lint against 300 obfuscated variants of the seeded phrases, and 100 clean sentences built from the negation, disclaimer and word-boundary rules of this note, with zero false positives allowed on the clean set;
  - a 20,000-character pathological body under the 1 s budget;
  - every catalogue type against a fully muted profile: mandatory types still deliver, the others are `suppressed`;
  - every write route under a token of every family (investor, sponsor, staff, partner) against its human, including every human-only route;
  - notifications list, badge and `read-all` under a token with agent and security notices present;
  - `publish` with no approval, with an approval for a different `body_sha256`, and with an expired approval;
  - terms versions at one minute before and exactly at the earliest allowed instant;
  - a material item submitted by its own drafter, by the drafter's human after an agent drafted it, and by a sponsor agent;
  - a P2 type emitted with the phase flag off;
  - the sandbox `+bounce@` path and its `contact.bounced` notice;
  - a campaign to a mixed-consent audience, and the unsubscribe link;
  - every consumed event with a missing required field;
  - a job re-run over the same interval;
  - tenant crossing on every read route;
  - the `Date.now()` grep;
  - the OpenAPI document diffed against the routes table;
  - the seed against the three design regions;
  - PII probes: a TFN, a bank account and an AML case reference planted in template vars must be refused or absent from every rendered channel;
  - only the owned paths changed.
- **Gate.** A pass unlocks the six dependents. There are at most three fix loops, and each re-runs the full hidden suites and the failing challenges.
- **Escalation trigger.** Any of these reopens the spec, and the user is asked:
  - the same assumption fails three times (for example what counts as a "yield" figure, or where a negation window ends);
  - a fix would need a change outside the owned paths: A14's policy table format, policy codes or `AUTO` conditions; A13's event names; A35's text or publish interface; A12's event names; a producer's event schema; the domain permission matrix;
  - the catalogue's maker-checker rows contradict the design's flags or the seeded phrase list in a way this note does not already resolve.

## Node-specific challenges (node QA)

- **Obfuscated guarantee.** A `property_update` body (block severity) contains:
  - "G U A R A N T E E D  income";
  - "guarantеed" with a Cyrillic е;
  - "capital‑protected" with a non-breaking hyphen;
  - "risk\u200Bfree" with a zero-width space;
  - a base64-looking token that decodes to "beats the market" only if decoded, which the lint must not do.

  Expected:
  - The first four each give exactly one `block` flag with the right rule code. The first is one `CLAIM_GUARANTEE` flag for "guaranteed income", not two. Each flag's offset and length cover exactly the original UTF-16 characters of the span.
  - The fifth must not flag.

  A lint that misses any of the four, splits the first into two flags, flags the fifth, returns normalised-text offsets, or takes more than one second on the 20,000-character variant fails the node.
- **The yield with no basis.** A `listing_copy` item's `headline` field is exactly: `7.1% yield. 7.1% net yield p.a. as at 30 Jun 2026. Yields have averaged 7.1% (gross, FY24-FY26). We target 7.1%.`

  Expected:
  - Exactly two `CLAIM_YIELD_NO_BASIS` flags, with `review` severity for this kind: at offset 0, length 4, and at offset 107, length 4.
  - The second and third sentences pass: the "p.a." must not split the sentence.
  - No `CLAIM_FORECAST_NO_DISCLAIMER` flag on the same span.
  - `summary` is "2 phrases flagged for review" and `tone` is amber.

  The same text as a `property_update` gives "2 phrases blocked". Flagging the compliant sentences, passing the bare figure, or a third flag fails.
- **Disclaimers are not claims.** Clean copy is linted as a `significant_event`: "Returns are not guaranteed. This investment is not capital protected. It is not risk-free. Unlike a term deposit, your capital is at risk. We safeguard records with secured storage." It must give zero flags and "No issues found". "Not only safe but guaranteed." must give two flags. Any flag on the clean text fails.
- **The agent that tidies up after itself.** An investor's token creates a second, wider token, and A12 and A101 emit `agent.token.widened` through A43. The token then immediately:
  - calls `GET /v1/me/notifications` and `/badge`;
  - calls `read-all`;
  - calls `read` on the agent notice's id;
  - calls `PUT preferences` to mute the `agent` category on email.

  Expected:
  - The notice is delivered by email and SMS regardless.
  - The token's list and badge do not include the agent notice.
  - `read-all` returns `skipped_protected: 0` and leaves it untouched.
  - The direct `read` returns 403 `COMMS_AGENT_CANNOT_ACK_NOTICE`.
  - The preference change returns `COMMS_TYPE_MANDATORY`.
  - When the person signs in with a passkey, the session badge still counts the unread agent notice.

  A token that sees, silences or clears its own notice fails the node.
- **Publishing on a stale approval.**
  - IRCM drafts a significant-event notice, and CCO and LEGL approve it in A14. The maker then tries to edit one word: 409 `MKT_ILLEGAL_TRANSITION`.
  - A test fixture then changes the stored body directly in the database and calls publication with the same `approval_id`: 403 `COMMS_NO_APPROVAL`.
  - Separately, a routine property update is checked clean, then edited to add "guaranteed". It returns to `draft`. `submit` without a new check gives `MKT_ILLEGAL_TRANSITION`. After the check, `submit` gives `gate_failed` G17 `CLAIM_GUARANTEE`.
  - A clean routine update submitted by SP-IR must wait for IRCM's human decision; no path approves it without a person.
  - Nothing may reach the public route or any inbox, and no `comms.item.published` audit event may exist for the refused cases.
- **Second human, even through an agent.** SP-OWN's agent token drafts a material `significant_event`. SP-OWN then submits it with a session: `COMMS_SECOND_HUMAN_REQUIRED`. SP-IR then submits it and succeeds. A sponsor agent that submits a material item gets `COMMS_MATERIAL_NEEDS_HUMAN`.
- **Terms notice arithmetic.**
  - A terms version is approved on Friday 2 October 2026 at 16:00 AEST. With `effective_from` 1 November 2026 00:00 AEDT (29 days 7 hours later, across the DST change on 4 October) it must be refused `COMMS_NOTICE_PERIOD_SHORT`. The earliest allowed instant is 1 November 2026 16:00 AEDT: one minute before is refused, and that instant passes.
  - With `effective_from` 2 November 2026 00:00 AEDT it must pass. It must send `terms.new_version` to every open account by email (mandatory) on publication, a reminder at T-7 (26 October) and a notice on the effective day, each exactly once under the injectable clock, and never to a closed account.
  - A version that passes inside 30 civil days, uses business days, or notifies twice fails.
- **Consent and unsubscribe.** MKTG's campaign is approved by CCO and sent to an audience of three investors: one who granted email consent, one who never did, and one who unsubscribed through the link without signing in.
  - Exactly one message lands in the sandbox inbox. It carries the sender block and the one-click unsubscribe headers.
  - The other two recipients get `suppressed` deliveries, with `no_marketing_consent` and `unsubscribed`.
  - An investor's token trying to grant consent is refused as human-only.
  - A campaign message to a non-consenting person fails the node.
- **Design fidelity of the seed.** With the standard seed:
  - `GET /v1/tasks?tenant=harbour` for Mia Chen must return exactly the three `spDash` rows, with tones green, amber and brick and targets SPN-09, SPN-10 and SPN-06.
  - `GET /v1/tasks/summary` must give the "Reporting 1 due" tile.
  - The Marine Parade listing's claims report must carry exactly two `review` flags: "guaranteed income" (IM, page 3) and "beats the market" (listing copy), with summary "2 phrases flagged for review". These are the flags behind `adListing` and APR-2290.
  - A seed that adds a fourth task, changes a tone, loses a page number or turns either flag into `block` fails.

## Pass criteria

- The hidden suites in `<workspace>\gauntlet\hidden\A43\` pass 100 % on three consecutive runs:
  - every lint vector: obfuscation, negation and disclaimers, word boundaries, overlaps, offsets in original UTF-16 units, yield basis, forecast, past performance, the clean set with zero false positives, and pathological input under 1 s;
  - every routing vector: each catalogue type × each preference state × each consent state × session versus token caller;
  - every task-from-event vector under the injectable clock;
  - every consumer-contract vector.
- `GET /v1/comms/rulesets/current` returns the seeded phrases with `severity_by_kind`, the negation and disclaimer lists, and the four structural rules (`CLAIM_YIELD_NO_BASIS`, `CLAIM_FORECAST_NO_DISCLAIMER`, `CLAIM_PAST_PERFORMANCE`, `CLAIM_EXTERNAL_LINK`), each with `rule_text` and `suggested_wording`.
- `checkClaims` on the seeded Marine Parade IM text and listing copy returns exactly the two design flags, with `review` severity and their locations. The summary table holds for every combination of counts. A ruleset change is refused without an A14 approval, and every report records the version it used.
- The `MKT`, listing-copy, delivery and task transition tables are tested exhaustively: every allowed transition passes, and every generated forbidden pair returns 409.
  - Editing is possible only in `draft`, and an edit in `checked` or `changes_requested` returns the item to `draft`.
  - Publication is refused with `COMMS_NO_APPROVAL` for every approval state other than `approved` or later, and for any `body_sha256` the approved payload does not cover.
  - Only `published` and `withdrawn` items are ever returned by the public and investor routes; a generated sweep over every state proves it.
  - `listing_copy` is never published by A43.
- Every seeded `comms.*` row uses an existing A14 policy code with no `AUTO` condition. `comms.publish_routine` is decided only by a human IRCM with a hardware-key assertion. A43's prechecks on `listing.go_live` and `metrics.publish` refuse on an open G17 flag. There is no `public_kpi_set` kind and no `comms.publish_public_kpis` row.
- **Notice guarantees:**
  - For every `mandatory` type, a fully muted profile still yields at least one `sent` delivery on a verified channel in the sandbox inbox.
  - For every `out_of_band_required` type, a `+bounce@` address yields a `bounced` row, a `sent` fallback on the other channel, one `contact.bounced` notification and the ACC-01 contact status.
  - A token caller never sees, counts or marks read an `agent` notification, never marks a `security` notification read, and never changes where a notice goes.
  - Every `legal`, `security` and `agent` delivery has exactly one `notice.sent` audit event, with `on_behalf_of_id` where an agent's action caused it.
  - Every `notice.requested` gets exactly one `notice.confirmed` or `notice.failed`.
  - A28's and A29's packaged port tests pass against A43's bindings.
- **Jobs under the injectable clock:**
  - `notify.tasks_sweep` moves "Due in 7 days" to "Due in 6 days" at local midnight and sets `overdue` at the deadline, with exactly one `task.overdue` event.
  - `notify.schedules` sends the T-3 offer notice, and the terms and fee notices on publication, at T-7 and on the effective day, exactly once each, all in calendar days.
  - `notify.publish_scheduled` publishes at `scheduled_for` and skips withdrawn items.
  - The module has no job producing `agent.daily_summary`.
  - Running every job twice over the same interval changes nothing, and a 30-day advance fires every schedule in order.
- **Watches and consent:**
  - `POST /v1/me/watches` is idempotent per (person, kind, target) and `DELETE` is idempotent. Both are reachable by a token and audited on behalf of the person.
  - `offer.opening` reaches `offer_open` and `sponsor_listing` watchers exactly once per person.
  - Campaigns reach only people with granted consent. The unsubscribe link works without sign-in and takes effect before the next send. Granting consent is refused to a token.
- **Adapters:**
  - The email and SMS conformance suites pass against the sandboxes.
  - Every message lands in `sandbox_message` and is readable through the local inbox route and the ADM-32 panel.
  - The scenario suffixes behave as specified, and an inbound SMS `STOP` withdraws consent.
  - `ADAPTER_EMAIL=postmark` (or any non-sandbox value) refuses to start in `RUN_MODE=local`.
- **Templates:**
  - Every template and starter renders for every allowed variable set, in text and HTML, with no unescaped `<` or `{{`.
  - Every starter lints clean under the current ruleset.
  - A disallowed variable is refused with `COMMS_TEMPLATE_VAR_NOT_ALLOWED`.
  - Campaign templates carry the complete sender block.
  - The PII probe (TFN, bank account, AML case id and a full wallet address planted in vars) is absent from every rendered channel.
- **Permissions:**
  - Tests generated from the domain matrix cover every route for IRCM, MKTG, CCO, LEGL, SP-IR, SP-OPS, SP-OWN, TP-LIC, an investor, and a token of each family.
  - Every human-only route refuses every token.
  - The sponsor second-human refusal, counted by human principal, and the agent material-item refusal fire on every material path.
  - A14 refuses a token on `decide` for every `comms.*` action code.
  - `allowed_actions` agrees with the actual outcome of each action for every state and role.
- **Surface:**
  - Every endpoint is in the generated OpenAPI document with `x-realx-scope`, and human-only routes carry `x-realx-requires-human`.
  - `packages/sdk` compiles with typed clients for them.
  - The MCP catalogue lists the delegable tools and none of the human-only or approval tools.
  - `npm run verify` finds no `Date.now()` or argument-less `new Date()` in the owned paths.
- **Seed and evidence:**
  - The standard seed reproduces the three `spDash` tasks, the "Reporting 1 due" tile and the two `adListing` review flags exactly, all raised through contract events.
  - After the full suite, A10's chain verification passes, and the `comms.item.*` and `notice.sent` event counts equal the transitions and protected deliveries performed.
  - Line coverage of the lint, routing and lifecycle code is at least 90 %.
  - The module boundary lint passes, and only the owned paths changed.
  - `app/evidence/A43.json` (naming the resolver bindings and the port bindings in use) and `A43.qa.md` exist and match disk.

## Human gates touched

- None directly. The node runs on sandbox channels, A14's software authenticator, the fixture resolvers and the injectable clock.
- Recorded for HG-LEGAL as counsel questions, each already a data row with a default:
  - the phrase lists, the negation and disclaimer lists, and the severity by kind (RG 234);
  - the yield-basis rule;
  - the 30-day notice period for terms and fees, and whether it runs in calendar days (the default);
  - which sponsor announcements count as material, and whether any need trustee sign-off;
  - whether a notice that an agent acted on someone's behalf may ever be delivered only in-app (the default here is always email or SMS);
  - under the Spam Act 2003 and APP 7: whether `watch.new_listing`, `offer.opening` or any other notice is a commercial electronic message that needs consent; the wording of the consent capture; the sender block; and whether the unsubscribe also covers SMS by `STOP` (the defaults are as specified here).
- If the founder prefers routine property updates to publish by an automatic rule instead of an IRCM decision, that is a founder decision plus a counsel question. It would also need an A14.1 loop adding an `auto_condition` value, as a prerequisite of a re-specified A43. It is not assumed here.
- Real email and SMS providers arrive under HG-VENDOR through the conformance suites. Until then ADM-32 shows both adapters as "sandbox".

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
