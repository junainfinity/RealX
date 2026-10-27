---
title: "A56 - Onboarding flow ONB-01 to ONB-10 with entity branch"
created: 2026-09-24
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am5]
node: A56
milestone: AM5
lane: "Investor app"
oracle: no
human_gate: none
---

# A56 - Onboarding flow ONB-01 to ONB-10 with entity branch

**The investor's first hour on RealX as one screen family, built in the prototype's layout: a sticky vertical stepper beside a white card. The steps are:**
- **ONB-01:** account, residency facts, 18+ and consent.
- **ONB-02:** investor type. The choice now branches. A joint account adds a "Second person" step. An SMSF, company or trust adds ONB-04 after the person's own identity check.
- **Invitation landing:** a join step for every invited person (the second joint holder, and each trustee, director, signatory, owner or controller of an entity). It takes their own 18+, residency and consent before any identity data.
- **ONB-03:** identity, with the liveness DropBox. A referred case, a PEP hit and a sanctions hit all show the same "we are reviewing" view with an expected time.
- **ONB-04:** ABN lookup, the people checklist with an Invite action per row, and the deed upload.
- **ONB-05:** TFN, with the consequence of not giving one spelled out.
- **ONB-06:** the three wholesale tests, the certificate upload and the review states.
- **ONB-08:** the passkey wallet. "Linked to your verified identity" appears only after `registerIdentity`. Recovery shows as on.
- **ONB-09:** a bank account in the investor's own name, and the PayID.
- **ONB-10:** the checklist of A30's status rows, with a next action for each row.

**Every step renders from the API's status and never from client state. The portfolio stays unreachable until `ready_to_invest` is true, and no screen shows anything that could reveal a screening hit, a freeze or a restriction.**

**Build status:** not started. **Milestone:** AM5 (Investor app and preview). **Lane:** Investor app. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 10-14 agent-hours (build, challenge and verify loops).

**Depends on:** [[A52 - Web app shells, routing and passkey sign-in (public, investor, sponsor)]], [[A30 - Investor onboarding, KYC cases and KYB (SM-01)]], [[A32 - Classification- wholesale certificates and target-market hooks (SM-02)]], [[A29 - Wallet service- smart-account factory, bundler sandbox, guardians and recovery requests]], [[A17 - Bank rail sandbox adapter (NPP-PayID-PayTo) and receipt matching]], [[A41 - Tax engine- profiles, withholding, AMMA and CRS-FATCA]], [[A50 - Design extension A- investor and sponsor screens not in the prototype]], [[A06 - Seed data and scenario simulator]], [[A35 - Documents, disclosure library, e-sign and acknowledgements (G12)]], [[A36 - Offerings- offer terms, listing review, raise monitor (SM-04)]]
**Unlocks:** [[A61 - Docker Desktop preview of the investor app (AM5 exit)]], [[A90 - P2- registered scheme, sub-funds, PDS and TMD, target market (G05), retail cooling-off]]

The graph edges A06 → A56, A35 → A56 and A36 → A56 were added to this note on 24 Sep 2026. The graph data must carry the same edges. A06 provides the seed and the Alex James Morgan persona that the pass criteria use. A35 provides the current terms and privacy-policy versions for ONB-01 and the join step. A36 provides the open-offer read for the ONB-06 ticket path. All three are in earlier milestones (AM1, AM3), so the edges create no cycle.

**Oracle rule:** this is not an oracle node (`oracle: no`). No money, supply or holdings move on these screens. Eligibility is decided behind them, though, so the node QA still writes hidden tests from this frozen spec to `<workspace>\gauntlet\hidden\A56\` before the build starts. The hidden set covers:
- the step-table oracle for every investor sub-type and every invited person;
- the route-guard probes, including the exemption probes;
- the tipping-off DOM, network and timing diff over every onboarding step;
- the browser-side PII scan;
- the token-level screenshot comparison against the prototype's `onb` views and A50's extension;
- the agent-parity manifest check.

The builder never reads that folder. The tests are published into `app/` only after the gate passes ([[App Gauntlet]] §2).

This note is not legal advice. Some onboarding declarations may have to be made by the person themselves. The candidates are 18+, the DVS and credit-reporting-agency identity consents, the wholesale declaration, the tax self-certification and the entity signatory declaration. These are counsel questions, carried as data in A03's `delegable.ts`. Until counsel answers, a `"counsel"` value behaves as non-delegable (A03, A12). This node renders whatever the data says and blocks nothing silently.

## Build brief

This node is UI only. It is made of two folders in `apps/web`:
- the `onboarding` route folder inside A52's `(investor)` route group;
- the `features/onboarding` folder, which holds the components, step table, status-row presentation table, copy, guard, action manifest and tests.

The screens contain no business rule. Every gate, state transition, threshold, date and error below is decided by the API modules and reaches the screen through `packages/sdk`:
- A30: investors, joint accounts, KYC, KYB and status.
- A32: classification.
- A29: wallets.
- A17: payments.
- A41: tax.

Because of this, a person on these screens, their agent through the API and the MCP server all see one behaviour (D04, A100).

A52 provides the shell and passkey sign-in and sign-up. Sign-up is A11's email verification followed by passkey registration, which is the first half of ONB-01. A52 also provides `getSession()`, the SDK client factory, `declareScreen()` for route metadata and the guard registry (`registerGuard(name, { check, redirectTo, exempt })`). The exact export names are taken from A52's evidence at the Specify step and recorded in the manifest.

Times come from the API's responses (`expected_by`, `valid_until`), never from the browser clock. `formatDateAest` is used for display formatting only.

**1. Routes, steps, the stepper and the guard (`app/(investor)/onboarding/`)**

- **Routes.** Each route is a Next.js page that declares its product-map screen id:

  | Route | Screen |
  |---|---|
  | `/onboarding` | server redirect to the current step |
  | `/onboarding/join` | invitation landing, declared as ONB-01 (invited variant) |
  | `/onboarding/account` | ONB-01 |
  | `/onboarding/type` | ONB-02 |
  | `/onboarding/second-person` | the joint "Second person" step, declared as ONB-02 (joint continuation) |
  | `/onboarding/identity` | ONB-03 |
  | `/onboarding/entity` | ONB-04 |
  | `/onboarding/tax` | ONB-05 |
  | `/onboarding/wholesale` | ONB-06 |
  | `/onboarding/wallet` | ONB-08 |
  | `/onboarding/bank` | ONB-09 |
  | `/onboarding/status` | ONB-10 |

  ONB-07 is not created here (see the slot below). The design's ONB codes equal the product map's, so no reconciliation row is needed in `packages/domain/screens.ts` beyond the declarations.
- **Layout.** `layout.tsx` is a server component. It reads `GET /v1/investors/me` and `GET /v1/investors/me/status`, computes the step list with the pure function `stepsFor(investor, statusRows, phaseFlags)` in `features/onboarding/steps.ts`, and renders the prototype's grid (`minmax(220px,260px) minmax(0,1fr)`, gap 40px). The grid holds:
  - the sticky `VerticalStepper`: "Open an account" eyebrow; a done step has a filled green dot with a tick; the current step has a green outline dot on a white row; a future step has a `#B9B4AA` dot and muted text;
  - the white card: 1px `#E3DED4` border, radius 14px, padding 36px, max-width 720px;
  - the `StepHeader`: "ONB-03 · Step 3 of N" in Geist Mono 12 green, a Newsreader 38 title and a 15px subtitle.

  N is the length of the computed step list, not the prototype's hard-coded 8.
- **The step table.** `stepsFor` is a table, not a chain of ifs, and it is what the hidden oracle tests. The rows follow the product map's ONB-02 state ("Entity types go to entity verification (ONB-04) after the individual checks"), journey J03, design analysis §6.2 and A50's extension:

  | Who | Steps | Count |
  |---|---|---|
  | Individual, wholesale, sophisticated or foreign investor | ONB-01, 02, 03, 05, 06, 08, 09, 10 | 8 |
  | First holder of a joint account | ONB-01, 02, 03, Second person, 05, 06, 08, 09, 10 | 9 |
  | Invited second joint holder | join, ONB-03, 05, 10 | 4 |
  | SMSF, company or trust (the person doing the onboarding) | ONB-01, 02, 03, 04, 05, 06, 08, 09, 10 | 9 |
  | Invited trustee, director, signatory, owner or controller | join, ONB-03, 10 | 3 |

  Notes on the table:
  - For an entity, the person verifies themselves first, then ONB-04 lists the other people. J03 step 3 is the second trustee's own ONB-03 through the invited-person row.
  - The second joint holder also gets ONB-08's add-key step when spec point (b) settles that A29 accepts a second owner key and `signing_rule = both`.
  - The second joint holder has no ONB-06. A32's policy row `joint_account_rule = per_investor_record` makes a joint account one classification. Which holder's test qualifies the account is a counsel question (below). If counsel answers "both", the change is an A32 data row plus a new row here.
  - The previous proposal (ONB-04 before ONB-03, with "J03, the product map wins" as the reason) is withdrawn. The product map, J03, design analysis §6.2 and A50 all give the order above.
- **The ONB-07 slot.** `steps.ts` exports `registerOnboardingStep({ id, route, after, when })`. A90 calls it from its own folder (`features/retail`) at boot, with `id: 'ONB-07'`, `route: '/target-market'` (the page lives under A90's owned path `app/(investor)/target-market/`), `after: 'ONB-06'` and `when: phaseFlags['offers.retail']`. This node ships the slot and a test that the slot is empty in P1. A90 never edits a file under this node's paths.
- **Step navigation is server-driven.**
  - A step is `done`, `current`, `available` or `locked`, taken from the status rows.
  - The stepper renders done and available steps as links, and locked steps as disabled buttons. The prototype's "jump to any step" is a prototype convenience, not product.
  - A direct request to a locked step's URL redirects (307) to the current step.
  - Back is a link to the previous available step. On the first step it has `opacity: .4` and no link, as the design draws it.
  - The primary button carries the step's `cta`: "Continue", or "Go to portfolio" on ONB-10.
- **The guard.** `features/onboarding/guard.ts` exports `onboardingComplete(status)`. It is true only when `status.ready_to_invest` is true, a field that A30's composer computes from every contributor being `done` or `not_applicable`. The guard is registered with A52's registry with `redirectTo: '/onboarding/status'` and this exemption list:
  - `/onboarding/*`;
  - `/support` and every ACC-04 support and complaints route (G18: a person stuck in review must be able to complain under the internal dispute resolution (IDR) access that RG 271 expects);
  - `/recover` and the ACC-02 recovery routes;
  - the close-account and privacy parts of `/account` (ACC-01);
  - `/account/agents` (ACC-05). A12 and A60 decide which scopes a not-ready person may grant; the likely answer is the read scopes only, which lets their agent follow onboarding progress.

  The public site, `(auth)` and `/sign-out` are never guarded. Every other `(investor)` route (portfolio, invest, wallet, activity, statements, the rest of account) redirects to `/onboarding/status` while the guard is false.

  **Guard placement (A52 follow-up loop, raised before Build).** In the Next.js App Router, a layout and its page render in parallel. A layout redirect does not stop the page's server fetch, and a shared layout is not re-rendered on soft navigation. So a layout-only guard cannot meet the pass criterion. The guard therefore runs in two places:
  - in A52's `middleware.ts`, from the session plus a short-cached (at most 5 s) status call, so a direct URL, a history navigation and an RSC fetch are all redirected before any render;
  - in a `requireGuards()` (or `guardedPage()`) wrapper that every non-exempt `(investor)` page awaits before it fetches data.

  Both are A52's files. A56 registers only the check and the exemptions. If A52's frozen spec still says "the `(investor)` layout runs all registered guards", that is corrected in the A52 follow-up loop before this node's Build.

  The UI guard is never the only barrier. For an account that is not ready, the API itself refuses every order, deposit credit, transfer and withdrawal through A19's G01, G03 and G04 hooks. A portfolio read for such an account holds nothing.

  A person who is `whitelisted` but still has a wholesale review open is not ready. The user may relax this to a read-only portfolio later; that is recorded as a founder question, not decided here.
- **Phone width** (design analysis §6.3 decision). Below 768px:
  - The stepper becomes a horizontally scrollable strip above the card. It uses the same component with `orientation="horizontal"` if A05 ships it; otherwise a documented variant in this feature folder, built from the same tokens.
  - The card keeps a 16px gutter and there is no horizontal page scroll.
  - The DropBox and the field grid stack.

  Every screen passes axe (zero critical or serious findings) and can be completed by keyboard alone.

**2. The steps, one by one (what each writes, reads, shows and refuses)**

- **Join (invitation landing)** (`/join`; undesigned, a composition of catalogued components). An invitation link goes through A52's sign-up (email verification, then login passkey) and returns to `/onboarding/join?invite={opaque invitation id}`. The id is not PII.
  - **What it shows:** who invited the person, masked by A04 as "A. Morgan", and the role: "Joint holder of a joint account" or "Trustee of Kwan Family Superannuation Fund". Both come from the invitation read on A11's invitation path.
  - **What it takes:** the same residency block as ONB-01 (below), "I am 18 or over", and the terms and privacy consent at A35's current versions. The consent is the collection notice, shown before any identity data is asked for.
  - **Buttons:** "Accept and continue" and "Decline".
    - For a joint invitation, Accept calls `POST /v1/investors/{id}/joint/accept` and Decline calls `/joint/decline`. Both are `requiresHuman`.
    - For an entity invitation, the accept route is spec point (d). A30 names no person-accept route for KYB invitees today. The proposed route is `POST /v1/kyb/entities/{id}/people/{personId}/accept`. It creates or reuses the person's `person_identity` without opening an investor account (A30 §1).
    - Both accept routes must take the same person fields as `POST /v1/investors`: `country_of_residence`, `citizenships[]`, `visa_status`, the ordinary-residence answers, `tax_residency_countries[]`, the 18+ declaration and both consent versions. Where A30's contract lacks the route or the fields, the gap is requested as an A30 follow-up loop before Build and never implemented in the app.
  - **Errors and outcomes:**
    - `JOINT_INVITE_EXPIRED` and a withdrawn invitation render "This invitation is no longer valid. Ask {masked name} to send a new one."
    - `INVESTOR_UNDERAGE` renders the terminal card with the support link.
    - A person who already has an approved case inside `kyc_valid_until` skips ONB-03. A30 reuses the case, and the step is shown as `done`.
- **ONB-01 Account** (`/account`; design line 815). Shown after A52's sign-up card has verified the email and registered the login passkey.
  - **Fields**, in the `auto-fit minmax(220px,1fr)` grid:
    - Email (read-only, the verified address).
    - Mobile.
    - "Country where you usually live" (`country_of_residence`; a searchable country select, default Australia).
    - "Are you an Australian citizen?" (`citizenships[]`). When the answer is no, a `visa_status` select appears (permanent visa, temporary visa, no visa) and so do A30's ordinary-residence questions ("In the last 12 months, have you been in Australia for 200 days or more?" and "Is your stay in Australia limited in time?").
    - "Country of tax residence" (the design's label; `tax_residency_countries[]`, handed to A41 and not used for the foreign-person test).
  - The residency questions and their wording are rendered from A30's data-driven question set, never hard-coded, and A30 computes `foreign_person` from the answers (A30 §1, foreign-person rule). The design's single "Country of tax residence" field is kept for tax. The residency fields added beside it are design deviation DF-A56-1, logged with the reason: FIRB foreign-person status depends on citizenship, visa and ordinary residence, not on tax residence. Whether these inputs are right is an HG-LEGAL counsel question (below).
  - **Acknowledgements:** two `AcknowledgeCard`s that the prototype lacks and the product map requires: "I am 18 or over", and "I accept the RealX terms (version {n}) and privacy policy (version {m})". The version ids come from A35's current `terms` and `privacy_policy` documents.
  - The design's Password field is a prototype artefact (A11 has no passwords) and is not built.
  - Nothing is written yet. The values travel to ONB-02 in React state. A page refresh means re-entering them. No PII is ever placed in `localStorage`, `sessionStorage`, cookies or the URL.
  - When A30's computation would make the person a foreign person, or the answers are non-Australian, an inline `Banner` shows the counsel-owned copy row "Some offers are not open to foreign persons; each listing shows whether you may join (FIRB checks)". The banner is triggered by the answers, and the authoritative flag is A30's `foreign_person`, shown on ONB-10.
  - An unticked 18+ card disables Continue. The API's `INVESTOR_UNDERAGE` renders a terminal card with the support link.
- **ONB-02 Investor type** (`/type`; line 816).
  - **Options:** three `OptionCard`s with the design's exact copy: "Individual · Investing in your own name.", "Joint · Two people. Both are verified, and you choose whether either or both must approve.", and "SMSF, company or trust · Every trustee, director and 25%+ owner is verified."
  - **Design bug 17 is fixed here: the choice branches.**
    - Joint reveals a two-option `signing_rule` ("Either of us can approve", "Both of us must approve"). A30 sets the rule at creation.
    - The entity card reveals a sub-type select (`smsf`, `company`, `trust`). `minor_guardian` is absent in P1 and P2.
  - **Continue** calls `POST /v1/investors` once, under an `Idempotency-Key` generated per form instance. The body carries the ONB-01 residency facts, the 18+ declaration, both consent versions, mobile, `account_type` and, for joint, `signing_rule`. If A30's create body lacks `signing_rule`, that is an A30 follow-up request.
  - `INVESTOR_EXISTS` (a refresh after success) is not an error: the layout re-reads status and moves on.
  - The type cannot be changed on this screen once the row exists. The copy says "Contact support to change the account type" (A44's path).
- **Second person** (`/second-person`; joint first holder only; A50 draws the joint "Second person" step, and A50's file is the visual reference).
  - **Fields:** the second person's email, and the chosen signing rule shown read-only.
  - "Send invitation" calls `POST /v1/investors/{id}/joint/invite`. This route is delegable and withdraws any pending invitation.
  - A `CheckRow` shows the invitation state from `GET /v1/investors/me`: `pending` is "Invited" (amber) with a "Resend" link (the same route), `accepted` is "Accepted" (green), and `declined` or `expired` is "Declined" or "Expired" (grey) with "Invite again".
  - Continue is always available. The joint account's readiness is A30's `members` row on ONB-10, not this step.
- **ONB-03 Identity** (`/identity`; line 817; the person must be present).
  - **Fields:**
    - Full legal name, Date of birth and Residential address.
    - The identity-document fields, rendered from A22's document schema as exposed through A30's case-create contract, not hard-coded. For a driver licence: licence number, card number and issuing state (DVS needs the card number). For a passport: passport number and country.
    - Two consent `AcknowledgeCard`s: the DVS check, and the credit-reporting-agency identity check with the words "no credit check is made".
  - **Liveness:** the `DropBox` "Liveness check · Look at your camera · takes about 10 seconds". It opens the camera through `getUserMedia`, captures a short clip or still, and uploads it through A08's presigned upload. If the camera is refused, it falls back to a file picker with a plain explanation.
  - **Creating the case:** before the case is created, the page asks for a fresh passkey assertion (A11 `requireFreshAssertion`, purpose `identity_verification`, 5 minutes). This is how the session proves `passkey_present` to A22 through A30. The page then calls `POST /v1/kyc/cases` and `POST /v1/kyc/cases/{id}/documents`.
  - **Result views.** Only A30's investor-facing codes drive the result view:
    - `verified` moves to the next step with the row green.
    - `reviewing` shows "We are reviewing your details" with `expected_by`, and a Continue that leads on to the next step.
    - `action_needed` lists the coded RFI items in plain English ("A recent utility bill", "A second identity document"), with an upload box that calls `rfi-reply`.
    - `expired` offers "Start again".
    - `could_not_verify` shows "We could not verify your identity" with the support path.
  - **Personas.** There is no code for a hit, a restriction or a freeze. A `PEP` or `SANCTION` persona therefore renders the `reviewing` view byte-for-byte like a `REFER` persona, and the page has no branch that could differ. A `FRAUD` persona ends in A30's terminal `auto_rejected` and renders whatever code A30 maps it to; `could_not_verify` is expected, and A30's contract test is the authority. Only `PEP` and `SANCTION` must equal `REFER`.
  - **Other errors:**
    - `KYC_CASE_OPEN` re-renders the open case.
    - `KYC_CONSENT_REQUIRED` highlights the unticked consent.
    - `FRESH_ASSERTION_REQUIRED`, `FRESH_ASSERTION_STALE` and `FRESH_ASSERTION_MISMATCH` re-run the passkey prompt once, then show "Please confirm with your passkey again".
    - A 429 from A11's rate limit says "Too many attempts, try again in a few minutes".
- **ONB-04 Entity and owners** (`/entity`; undesigned, drawn by A50 as `design-ext:ONB-04`). Reached after the person's own ONB-03.
  - **Lookup:** an ABN or ACN `TextField` with a "Look up" outline button, which calls `POST /v1/kyb/entities`. The result card shows the registered name, type and registry status. A cancelled or mismatched entity shows the registry reason.
  - **People:** the `CheckList` is pre-filled from `resolveUbo` (owners at 25 % or more). The person can add rows (trustee, director, signatory, owner with ownership per cent, controller with `control_basis` from A30's counsel-owned list) and remove rows before submission.
  - **Pills.** Each row carries a pill from `GET /v1/kyb/entities/{id}`, in A50's vocabulary:

    | State | Pill |
    |---|---|
    | `verified` | Verified (green) |
    | `verifying` | Verifying (amber) |
    | `invited` | Invited (amber) |
    | a pre-filled row not yet saved | Not started (grey) |
    | `failed` | "Could not verify" (red; a composition A50 did not draw, logged as DF-A56-2) |

    A failed row never shows the other person's reason or case detail.
  - **Invite and Resend.** Every row whose pill is Not started has an "Invite" row button. It calls `POST /v1/kyb/entities/{id}/people` for that person, and A30 invites them to their own join step and ONB-03 through A11. An Invited row has "Resend". A30 names no resend route for KYB invitees, so Resend is spec point (d) as well: it is requested from A30, and until the route exists it is hidden.
  - Ownership over 100 % is refused inline with the API's `KYB_OWNERSHIP_EXCEEDS_100`.
  - **Deed:** the deed `DropBox` accepts PDF only, magic-byte checked server-side. A renamed executable shows "This file is not a PDF".
  - **Submit.** "Submit for review" calls `POST /v1/kyb/entities/{id}/submit`. A30's precondition is the deed, a signatory, the controller rule, and every listed person in `verifying` or better.
    - The page never computes that precondition. The button is disabled, with A50's tooltip naming who is outstanding, exactly when the entity read returns a non-empty `submit_blockers` list. That field is requested from A30 as a follow-up and must be computed by the same function as the submit check.
    - Until the field exists, the button stays enabled and A30's refusals render inline: `KYB_DEED_REQUIRED`, `KYB_SIGNATORY_REQUIRED`, `KYB_CONTROLLER_REQUIRED`, and `KYB_OWNERS_PENDING` naming the people.
    - This replaces A50's "disabled while any person is not Verified". A30 lets the entity be submitted while people are still verifying, and G03 is enforced at whitelisting and on ONB-10, not at submit. A50's extension must be changed to match (A50 follow-up loop).
  - **After submission:**
    - The banner reads "The fund can invest once {name} finishes verification", naming the pending people masked as "A. Kwan" per A04's mask.
    - The person continues to ONB-05 while the others verify.
    - The people list is read-only. Any change attempt is refused by A30 with `KYB_SUBMITTED_LOCKED`, rendered as "The people list is locked while we review it. Contact support if it has changed." The list reopens only when a KYCA's RFI `OWNERSHIP_CHANGE` puts the entity case into `rfi_pending`.
- **ONB-05 Tax** (`/tax`; line 818; subtitle "Without a TFN we must withhold tax from distributions at the top rate.").
  - **Fields:**
    - Tax file number: a write-only input with the placeholder mask "••• ••• •••". After saving, the field shows "On file", because A41 never returns a TFN, even masked. The design's "••• ••• 482" is the placeholder format, not a value.
    - "US person for FATCA?": a yes or no `OptionCard` pair; the indicia questions appear on yes.
    - "Other tax residencies": repeatable, a country plus a TIN or a reason the TIN is unavailable.
  - **No TFN.** "I don't have a TFN" reveals the exemption-code select, or the consequence `AcknowledgeCard` "I understand tax will be withheld from my distributions at the top rate until I provide a TFN", which sets `no_tfn_consequence_acknowledged_at`. A missing TFN never blocks Continue (product map D08).
  - **Entity branch:** shows the fund's ABN (public; the API runs the mod-89 check), the CRS entity type and, for a passive NFE, the controlling persons from the KYB list with their residencies.
  - **Joint:** a joint second holder sees only their own party's fields.
  - **Continue** calls `PUT /v1/me/tax-profile`, then `POST /v1/me/tax-profile/self-certification` under a passkey step-up.
  - `TAX_TFN_INVALID_FORMAT` renders as "Check the number". The copy says "format accepted", never "verified".
- **ONB-06 Wholesale status** (`/wholesale`; line 819; subtitle "Until the retail launch, RealX offers are open to wholesale investors only."). This step is shown to the account's first holder, or to an entity's verified signatory (`CLS_SIGNATORY_REQUIRED` otherwise).
  - **Options.** The three `OptionCard`s are rendered from the option strings that `GET /v1/classification/me` returns. They equal the design's copy verbatim, and a test asserts it.
  - **Accountant's certificate.** This option reveals:
    - the `DropBox` "Drop your certificate here · PDF or photo · signed within the last 6 months" (the "6 months" is derived from A32's `max_certificate_age_days_at_upload`);
    - the accountant name, professional body and number;
    - the certificate date.

    The page computes no expiry. Before the decision it shows only the option string's "Valid for 2 years". After approval it shows A32's `valid_until`.
  - **"Invest A$500,000 or more in one offer"** lists the open offers from A36's `GET /v1/public/offerings?status=open` and requires one. With no open offer, the card is disabled with "No offer is open right now". The ticket is per offer, never a standing claim. `CLS_OFFER_NOT_OPEN` renders "This offer is no longer open".
  - **"Sophisticated or professional investor"** takes the assessment document.
  - **Routes.**
    - `POST /v1/classification` creates the request and `POST /v1/classification/{id}/evidence` attaches the evidence. A32 marks both delegable to a scoped agent, unless A03's action split has not landed, in which case A32 keeps them `requiresHuman`. The manifest copies whatever A03 says.
    - The investor's declaration is `POST /v1/classification/{id}/declaration`. It is `requiresHuman` and needs a fresh passkey assertion over the evidence set.
  - **Errors:**
    - `CLS_CERTIFICATE_TOO_OLD` names the cut-off date.
    - `CLS_CERTIFICATE_DATE_FUTURE` renders "The certificate date can't be in the future".
    - `CLS_EVIDENCE_INCOMPLETE` names the missing items.
    - `CLS_DECLARATION_STALE` asks the person to declare again.
    - `CLS_IDENTITY_FIRST` sends the person back to Identity.
    - `CLS_ALREADY_OPEN` re-renders the open review.
  - **State views** (from `classificationFor`'s shape):
    - `requested`: the form.
    - `evidence_submitted` and `under_review`: "We check certificates within 1 business day", with the review kind. On the ticket path the review may sit in `awaiting_cco` (the CCO's `4E` approval); it shows the same text.
    - `rfi`: the reason code's plain-English text, an upload box that re-submits through `/evidence`, and a fresh declaration.
    - `valid`: the basis and "Valid to {valid_until}".
    - `rejected`: the reason's plain-English text and "Try another test".

    Renewal is never started from onboarding.
- **ONB-08 Wallet** (`/wallet`; line 820; subtitle "Your tokens live in a wallet secured by a passkey on your device. No seed phrase to lose.").
  - **Creating the wallet.** The `DropBox` "Create passkey wallet · Uses Face ID, Touch ID or Windows Hello" runs A29's WebAuthn wallet-key registration (purpose `wallet`, a separate credential from the login passkey) and then `POST /v1/wallets`. The call is idempotent per investor: a second click or a retry after a dropped connection returns the same wallet.
  - **The rows.** The `CheckList` then renders three rows from a typed projection of `GET /v1/wallets/me`. The projection picks only `address`, `registry_status`, `guardian_status` and `recovery_disclosure_version`.
    - "Wallet address · {truncateAddress(address)}", with `Created` (green) as soon as the counterfactual address exists.
    - "Linked to your verified identity · Required to receive tokens". The pill maps A29's `registry_status` vocabulary:

      | `registry_status` | Pill |
      |---|---|
      | `registered` | Linked (green) |
      | `registering` | Linking (amber) |
      | `unregistered`, `stale`, `deregistered` | "Pending identity verification" (grey), one identical view |

      The three grey statuses share one view so that no restriction can show through.
    - "Recovery · RealX recovery service", with `On` (green) when `guardian_status` is `realx_only` or `active`. The default RealX guardian satisfies J02 step 9 ("sets recovery").
  - **Polling.** The page polls the wallet route every 10 s while the wallet is not `registered`, and never invents the green state.
  - **What is never shown.** The page never renders `frozen`, `frozen_amount`, `frozen_reason`, `kyt_score` or any hold field, and the projection drops them before they reach a component. A wallet frozen after a sanctions match looks exactly like a referred person's unregistered wallet.
  - **Below the list:**
    - a `Banner` with the recovery text from `recovery_disclosure_version` ("RealX holds a recovery guardian key; it can never move your tokens on its own");
    - an optional outline action, "Add a second device (recommended)", which opens A29's add-key ceremony.

    Guardians beyond the default RealX guardian are POR-06's job (A59) and are not required to continue.
  - **Spec point (b).** For a joint account with `signing_rule = both`, the second holder's passkey is added as an additional owner key through the same add-key route. If A29 does not accept a second principal's credential, the wallet stays single-owner and joint approvals run off-chain through A12's co-sign path. A29's evidence decides, and this note records the outcome.
- **ONB-09 Bank and deposit** (`/bank`; line 821; subtitle "Withdrawals only go to an account in your name.").
  - **Withdrawal account.** Fields: BSB (six digits, formatted `062-000`), Account number and Account name.
    - Continue calls A17's `POST /v1/payments/nominated-account { bsb, account_number, account_name }` under a passkey step-up. The route is `requiresHuman`, because changing the withdrawal account is non-delegable under D04. Spec point (c) is settled: A17 names this route.
    - After save, the account shows A17's mask verbatim (`••4471`). The design's `••••4471` is DF-A56-3.
    - A17's `422 NOMINATED_NAME_MISMATCH` is shown plainly: "The account name does not match your verified name". It is not an AML matter.
    - `FRESH_ASSERTION_*` re-prompts once.
  - **Deposit details**, all from `GET /v1/payments/payids/me`:
    - "Your PayID for deposits" is read-only, with a copy `ValueChip` and the caption "Arrives in under a minute over NPP".
    - The deposit BSB and account number from the same response. If A17's response lacks them, that is an A17 follow-up loop.
    - The client-money statement "Held on trust at an Australian bank", which is a counsel-owned copy row.

    `409 PAYID_NOT_ISSUED` renders "Your PayID appears once your identity is verified". A17 returns this response identically for an unverified person and a restricted one. The page shows no hold, restriction or reason field, so a restricted person sees exactly what a referred person sees.
  - **Deposit rule copy:** "Deposits are matched to your name. Money sent from an account in someone else's name is held for review and may be returned." This matches A17's `payer_name_mismatch` hold and its TRSY → CFO decision.
  - **PayTo.** The "Set up PayTo" outline action is rendered only when A08's `payments.payto` flag is on. A17 returns `501 not_in_phase` while it is off, so it is hidden in P1.
- **ONB-10 Status** (`/status`; line 822).
  - **Title.** "You're verified" with the design's subtitle "You can now invest in any wholesale offer on RealX." only when `ready_to_invest` is true. Otherwise "Almost there".
  - **Rows.** The `CheckList` renders A30's eight status rows (A30 §8 mapping table). Their display order is fixed in `features/onboarding/status-rows.ts`, a presentation table with no rule in it. The design's five rows come first in the design's order:
    1. `identity`: "Identity verified (G01)"
    2. `sanctions`: "Sanctions and PEP (G02)"
    3. `tax`: "Tax details (G16)"
    4. `wholesale`: "Wholesale status (G04)"
    5. `residency`: "Residency (G06)"
    6. `members`: "Second holder" for joint, "Entity and owners (G03)" for entity
    7. `wallet`: "Token wallet"
    8. `bank`: "Bank account"

    An individual sees seven rows, because `members` is `not_applicable` and omitted. Joint and entity accounts see eight.
  - **Details.** The detail string is A30's or the contributor's `detail` code through `copy.ts`. Examples: "DVS match, 2 sources · liveness pass", "No matches", "TFN provided" or "No TFN · tax withheld at the top rate", "Accountant's certificate", "Australian resident" or "Foreign person · FIRB checks apply per offer", "2 of 3 people verified", "0x7a3f…c91e · linked", "CBA ••4471 · PayID ready".
  - **Pills per state:**

    | State | Pill |
    |---|---|
    | `done` | The row's design pill: identity "Pass", sanctions "Clear", tax "Pass", wholesale "Valid to {valid_until}" (the date from the API), residency "Pass", members "Verified", wallet "Linked", bank "Ready" |
    | `reviewing` | "Reviewing" (amber), with "Expected by {expected_by}" appended to the detail |
    | `action_needed` | "Action needed" (red), with the coded reason in plain English |
    | `not_started` | "Not started" (grey) |
    | `not_applicable` | Row omitted, except the sanctions row |
  - **The sanctions (G02) row.** A30 returns it `not_applicable` until `identity` is `done`, whatever the screening found. The page renders that state as the constant grey pill "After identity", with the detail "Checked when your identity is verified". It shows "No matches" / "Clear" only when A30 returns `done` with `NO_MATCHES`. The rendering depends only on A30's row, which depends only on the identity state, so a `REFER`, `PEP` and `SANCTION` persona render the same G02 row.
  - **Next action.** Every non-green row carries a `Button.link` "Continue" to its `next_action` route.
  - **When `ready_to_invest` is true,** the card also shows:
    - the class line from `GET /v1/classification/me` (basis and `valid_until`, for example "Wholesale · Accountant's certificate · valid to 30 Jun 2028") and the allowed products from that response's `rights` field;
    - a "See open offers" link to the investments list;
    - the primary CTA "Go to portfolio", whose target is `/portfolio`.

    Holder-cap headroom is per trust, and no investor-level API exposes it. It is shown per offer on the buy step (A57). If the founder wants it on ONB-10, an A19 investor-level read is needed first (recorded as an open item).
  - **Toast.** "Welcome to RealX, {first name}" fires once, on the first arrival at the ready state. A43's welcome notification is separate.
  - Rows, pills and lines come only from API fields. The page contains no rule that could turn a row green.

**3. Agent parity, audit and the action manifest (D04, G22)**

- **The action manifest.** `features/onboarding/actions.manifest.ts` lists every button and form on the eleven routes: the join step, the Second person step, and ONB-01 to ONB-10 without ONB-07. Each entry has its route, method, scope cell and `delegable: true | false | "counsel"`, copied from A03's `delegable.ts`, never decided here. The manifest's test meaning is fixed:

  | Value | What the route must do for an investor's agent token |
  |---|---|
  | `false` | The route is marked `requiresHuman` by A11 and refuses the token with `AGNT_NOT_ALLOWED` before any scope check. |
  | `"counsel"` | Refused with A12's counsel-pending code. Where the route is also `requiresHuman`, it is refused with `AGNT_NOT_ALLOWED` first. |
  | `true` | Succeeds within the token's scope and limits. |

  The counsel-pending code is spelled `SCOPE_PENDING_COUNSEL` in A12 and A30, and `SCOPE_PENDING_COUNSEL` in A03 and A32. The spelling is reconciled in A03 and A12 before Specify, and `copy.ts` and the tests carry the recorded one.
- **What the manifest lists today** (from A30 §7, A32, A41 and A17):
  - Non-delegable: `POST /v1/investors`, `/joint/accept`, `/joint/decline`, the KYB person-accept route, `POST /v1/kyc/cases`, its documents and RFI replies, `POST /v1/classification/{id}/declaration`, `POST /v1/wallets` and add-key, and `POST /v1/payments/nominated-account`.
  - Delegable within scope: `/joint/invite`, `POST /v1/kyb/entities`, `/kyb/entities/{id}/people`, `POST /v1/classification`, `/evidence`, `PUT /v1/me/tax-profile`, and every read (`/investors/me`, `/investors/me/status`, `/kyc/cases/me`, `/kyb/entities/{id}`, `/classification/me`, `/me/tax-profile`, `/wallets/me`, `/payments/payids/me`, `/public/offerings`).
  - `"counsel"`: `POST /v1/kyb/entities/{id}/submit` (the signatory declaration) and `POST /v1/me/tax-profile/self-certification` (the FATCA/CRS self-certification; A03's CQ-10).

  An investor's agent can therefore follow the person's onboarding progress and prepare the delegable parts. It never performs a personal act.
- **The tax self-certification conflict (escalated).** A03 and A12 say the self-certification is `"counsel"` and behaves as non-delegable until counsel answers. A41's default (`tax.self_cert.person_only` off) lets an agent sign it under the holder's standing consent. The upstream nodes must record one answer before this node's Specify step. The expected fix is that A41's default honours A03's `"counsel"` value, so the default becomes person-only and the refusal becomes the counsel-pending code. Until then, this node's hidden tests and challenge expect the counsel-pending refusal. The screen signs the self-certification with a passkey because a person is present.
- **The node's own manifest test** asserts all of these:
  - every route named exists in the OpenAPI document;
  - every `false` entry maps to a route A11 marks `requiresHuman`;
  - every `"counsel"` entry is refused with the counsel-pending code (or `AGNT_NOT_ALLOWED` where the route is also `requiresHuman`);
  - every route the screens call (captured by MSW during the component tests) is in the manifest;
  - `copy.ts` has an entry for every error code in the OpenAPI error enum of every manifest route.

  A100's parity suite consumes the manifest later (AM8). It is not a pass criterion here.
- **Audit.** Every write the screens make carries A09's idempotency key and lands in the person's activity through A10 as an act of the person themselves (`channel: ui`).

**4. Empty, error and resumption states**

- **Resumption.** Closing the browser at any point and signing in again lands on the current step, computed from status. A created KYC case shows its `reviewing` view, a created wallet shows its address, and a pending invitation lands the invitee on the join step. Nothing about progress lives in the browser. A network failure after a write is retried with the same idempotency key and never creates a second investor, case, wallet, invitation acceptance or classification.
- **API errors.** Every stable code named above has a copy entry in `features/onboarding/copy.ts`, including:
  - A30: `KYB_OWNERS_PENDING`, `KYB_SUBMITTED_LOCKED`, `KYB_CONTROLLER_REQUIRED`, `JOINT_BOTH_REQUIRED` and `JOINT_INVITE_EXPIRED`;
  - A32: `CLS_OFFER_NOT_OPEN`, `CLS_SIGNATORY_REQUIRED`, `CLS_EVIDENCE_INCOMPLETE`, `CLS_DECLARATION_STALE` and `CLS_CERTIFICATE_DATE_FUTURE`;
  - A17: `PAYID_NOT_ISSUED` and `NOMINATED_NAME_MISMATCH`;
  - A11: `FRESH_ASSERTION_*` and `AGNT_NOT_ALLOWED`;
  - A12: the counsel-pending code.

  An unknown code or a bare 5xx renders the generic card "Something went wrong on our side · reference {request id}" with the support link, never the raw message. A 403 `FORBIDDEN` or a lost session sends the person to A52's sign-in and back to the same step.
- **Uploads.** Type and size are checked in the browser before upload (PDF, JPG, PNG; 20 MB) and again by the API. Progress and a retry are shown. A failed presigned upload leaves the step unchanged.
- **Empty states:**
  - An entity with no owners from the registry lookup shows the empty checklist with "Add the people who control this entity".
  - With no open offer, the ticket option is disabled.
  - A status with `next_action: start_identity` and no case renders ONB-03 fresh.

## Owned paths (other nodes do not edit these)

- <workspace>\app\apps\web\app\(investor)\onboarding\
- <workspace>\app\apps\web\src\features\onboarding\

Inside the feature folder, by convention:
- `steps.ts`: the step table, `stepsFor` and `registerOnboardingStep`.
- `status-rows.ts`: the ONB-10 display order and pill vocabulary.
- `guard.ts`: `onboardingComplete`, the exemption list and the registration with A52's registry.
- `actions.manifest.ts`.
- `copy.ts`: every code in plain English.
- `components/`: StepFrame, StepCard, LivenessCapture, ResidencyBlock, InvitationCard, PeopleChecklist, UploadBox, StatusRow and the form components, all composed from `@realx/ui`.
- `hooks/`: status polling, the step-up helper over A52's session, and the typed wallet projection.
- `test/`: Vitest and React Testing Library, with MSW handlers generated from the OpenAPI document.
- `e2e/`: Playwright specs and screenshot fixtures, picked up by A52's web e2e config glob `src/features/**/e2e/*.spec.ts`. If A52's config lacks the glob, it is requested as A52's follow-up loop, never edited here.

The route folder holds only `layout.tsx`, `page.tsx`, the eleven step pages and their `loading.tsx` and `error.tsx`. No component and no rule lives there.

## Design references

- **`design:onb`** in `<workspace>\design\RealX App.dc.html`: the `onb` screen template at lines 306-361.
  - The grid `minmax(220px,260px) minmax(0,1fr)`, gap 40px.
  - The sticky stepper column: the eyebrow "Open an account" in Geist Mono 12 uppercase `#5E6166`; step buttons with `padding:10px 12px`, radius 8px, Geist 14/500; 22px dots with a 1px border.
  - The white card: `#fff`, 1px `#E3DED4`, radius 14px, padding 36px, gap 24px, max-width 720px.
  - The header block: the green mono code line, a Newsreader 38/1.1 title and a `#45484D` 15px subtitle.
  - The field grid: `repeat(auto-fit,minmax(220px,1fr))`, gap 14px, with 1px `#D6D0C4` inputs on `#FBFAF7`, radius 8px, padding `12px 14px`, and a green focus outline.
  - The option cards at lines 331-336: selected means an ink border, a `#FBFAF7` background and a green 18px dot.
  - The dashed `#B9B4AA` DropBox at lines 340-343: Newsreader 22 title, mono 12 caption, `#FBFAF7` background, radius 12px, padding 28px.
  - The check list at lines 346-353: `1fr auto` rows, padding `14px 18px`, `#ECE8E0` dividers, mono 11 pills.
  - The footer at lines 355-358: Back as outlineSoft with `opacity: {{ onbBackOp }}` = .4 on the first step; the primary green CTA with hover `#174838`.

  The `ONB` data at lines 815-822 gives every title, subtitle, field label, placeholder, option string, box caption, check row, pill and CTA quoted in the build brief. `onbCur`, `onbSteps`, `onbNext` and `onbBack` at lines 823-827 are the prototype's step logic, which `stepsFor` and the server-driven navigation replace. The "Step n of 8" string at line 318 becomes "Step n of N".
- **Components and tokens.** `design:VerticalStepper`, `design:OptionCard` and `design:DropBox`, plus `StepHeader`, `TextField`, `CheckList`/`CheckRow`, `StatusPill`, `AcknowledgeCard`, `Banner`, `ValueChip`, `Button` and `Toast`, all from A05's `@realx/ui` (design analysis §2 items 18-23, 27, 43, 46).
  - Tokens: cream `#F5F2EC`, ink `#17191C`, green `#1F5E4B`, brick `#9A4524`, red `#A3312A`, amber `#7A4F00` on `#F6ECD6`, grey `#4A4D52` on `#ECE8E0`, line `#E3DED4`, soft line `#ECE8E0`, input `#FBFAF7`, muted `#B9B4AA`.
  - Fonts: Newsreader, Geist, Geist Mono.
  - Helpers: `truncateAddress` (first 6 + last 4) for the wallet address, and `formatDateAest` for dates.
- **`design:bug 17 entity and joint steps`** (design analysis §6.4 item 17 and §6.2). The prototype stores `invType` and never uses it, and ONB-04 and the joint second-person step are absent. This node builds the branch in the order §6.2 gives (the entity step after the individual checks), and the ONB-10 rows the branch needs.
- **`design-ext:ONB-04`, the joint "Second person" step and the 9-step steppers** from [[A50 - Design extension A- investor and sponsor screens not in the prototype]]. The entity step is drawn as step 4 of 9 after ONB-03, with an ABN lookup TextField and the outline "Look up" button, the entity result card, the people CheckList with Verified (green), Invited (amber) and Not started (grey) pills and an "Invite" row button, the deed DropBox and the G03 note.

  This node adds the Verifying (amber) and "Could not verify" (red) pills (DF-A56-2) and takes the submit button's disabled state from A30's `submit_blockers`, not from the pills (see ONB-04; A50 follow-up).

  The remaining undesigned states are compositions of catalogued components under design analysis §5's rule (same tokens, same blocks, no new visual language):
  - the join step;
  - the residency block and consent cards on ONB-01;
  - the consent cards and the reviewing, action-needed, expired and could-not-verify views of ONB-03;
  - the accountant fields and the under-review, rfi and rejected cards of ONB-06;
  - the pending-link pills, recovery row and banner of ONB-08;
  - the before-PayID state and deposit details of ONB-09;
  - the amber, red and grey rows with a "Continue" link, the class line and the open-offers link on ONB-10.

  Where A50 drew any of them, A50's file is the reference, and this node's screenshot comparison uses it.
- **Product-map mocks** (content only; the design wins on visuals):
  - ONB-06: the accountant fields ("Chan & Partners · CPA 1234567", certificate date) and the banner "We check certificates within 1 business day".
  - ONB-09: the PayID and BSB/account deposit card, and "held on trust at an Australian bank".
  - ONB-10: the next-action pattern and "You can invest" with a link to open offers.
  - ONB-04: the "each person invited to verify" rows.
- **Design deviations logged** (DF findings, shown to the founder at the AM5 review):
  - DF-A56-1: the residency fields added on ONB-01.
  - DF-A56-2: the Verifying and "Could not verify" pills on ONB-04.
  - DF-A56-3: A17's `••4471` mask in place of the design's `••••4471`.
  - The Password field is removed from ONB-01.

## Requirements covered

- **Product-map screens:**
  - **ONB-01 Create account.** Done when the screen collects the residency facts (country where the person usually lives, citizenship, and visa and ordinary residence for non-citizens), the tax-residence country, the 18+ declaration, both consent versions and mobile after A52's email-and-passkey sign-up; shows the FIRB banner for a likely foreign person; never shows a password field; and the invited variant (the join step) collects the same facts before the invitee's ONB-03.
  - **ONB-02 Investor type.** Done when the three options branch (joint signing rule and the Second person step; entity sub-type and ONB-04 after ONB-03), `POST /v1/investors` is called once per person, `INVESTOR_EXISTS` resumes rather than errors, and the joint invitation is sent, re-sent and shown in its state.
  - **ONB-03 Verify identity.** Done when the identity fields from A22's schema, both consents and the liveness capture create a case under a fresh passkey assertion; each of the five investor-facing codes has its view; and a `PEP` or `SANCTION` persona cannot be told apart from a `REFER` persona.
  - **ONB-04 Entity and owners.** Done when ABN or ACN lookup, the people checklist with ownership and a per-row Invite action, the deed upload and submission work against A30's KYB routes; the post-submission lock renders A30's answer; and the entity cannot pass ONB-10 until every person is verified.
  - **ONB-05 Tax details.** Done when TFN, FATCA, other residencies and the no-TFN consequence are captured through A41; the TFN is write-only; the entity branch captures ABN, CRS type and controlling persons; and a missing TFN never blocks.
  - **ONB-06 Wholesale certificate.** Done when the three tests render from A32's option strings; no expiry is computed on the page; the ticket path binds to an open offer from A36; the declaration is passkey-signed; and the `under_review` (including `awaiting_cco`), `rfi` and `rejected` states render.
  - **ONB-08 Set up wallet.** Done when the wallet passkey ceremony and `POST /v1/wallets` create one wallet per investor; the address shows as `Created` at once; "Linked" appears only for `registry_status = registered`; recovery shows as on; and no frozen or hold field is rendered.
  - **ONB-09 Link bank and deposit.** Done when the nominated account in the investor's own name is captured through A17's route under a step-up; a name mismatch is explained plainly; the PayID and deposit details appear only when A17 returns them; `PAYID_NOT_ISSUED` renders the one waiting text; and PayTo is hidden while its flag is off.
  - **ONB-10 Verification status.** Done when every row, pill, detail and line comes from the API; each non-green row links to its next action; the class line, allowed products and open-offers link render when ready; and "Go to portfolio" exists only when `ready_to_invest` is true.
- **Journeys:**
  - **J02 Sign up and verify (individual, wholesale).** Done when the seeded clean persona walks steps 2, 3, 4, 6, 7, 9, 11 and 12 on these screens in one Playwright run against the compose stack. A30's, A32's and A31's fixtures perform steps 5, 8 and 10. The run ends on ONB-10 with the class line and allowed products shown and a "Go to portfolio" link whose target is `/portfolio`, followed through the guard to a 200. The portfolio page's own content belongs to A58 and is not asserted here.
  - **J03 Onboard an SMSF.** Done when:
    - steps 1, 2, 5, 6 and 8 run on these screens;
    - step 3 runs as the invited second trustee's own join step and ONB-03;
    - ONB-10 shows the entity pending until step 4 and then green;
    - on the ticket path, the wholesale row stays amber through step 7 (A32's review in `awaiting_cco`) until A32's `4E` effector fires, then turns green within one status poll.
- **Gates** (all evaluated by the API; this node proves the screens cannot get around them):
  - **G01 Identity verified.** Done when no guarded `(investor)` route is reachable before `ready_to_invest` is true, and the wallet never shows "Linked" before registration.
  - **G02 Sanctions and PEP clear.** Done when no onboarding step, poll response, PayID response or notification carries state that reveals a hit, a restriction or a freeze, and the G02 row renders only A30's row.
  - **G03 Entity and owners verified.** Done when an entity's or joint account's ONB-10 `members` row stays amber with the pending count until every person's case is approved, and the portfolio stays unreachable.
  - **G04 Wholesale status valid.** Done when ONB-06 submits only through A32, the expiry shown is A32's `valid_until`, and an under-review, rfi or rejected classification keeps the portfolio unreachable.
  - **G06 Residency and FIRB.** Done when residency facts are captured, on ONB-01 or the join step, for 100 % of the people who reach ONB-03, invitees included; foreign persons may continue with the FIRB banner; and the ONB-10 row reflects A30's `foreign_person`.
  - **G16 Tax details.** Done when ONB-05 writes a tax-profile version and self-certification, the no-TFN consequence is acknowledged before Continue, and the ONB-10 row shows the withholding consequence rather than a block.
- **Modules:**
  - **M06 Investors.** Done when every investor sub-type of D04 (individual, joint, SMSF, company, trust, wholesale, sophisticated, foreign) can create its record and accounts from these screens using A30's routes unchanged, and every invited person can accept and verify.
  - **M07 Onboarding review.** Done when every case, RFI reply, entity submission and classification these screens create appears in A30's `GET /v1/kyc/queue` (except restricted cases, which appear only in `GET /v1/kyc/restricted`) and A32's review queue, with the seeded details (case INV-00912's referral, the 30 Jun 2028 certificate). Every decision made through those APIs by A30's and A32's fixtures must show on ONB-03, ONB-06 and ONB-10 within one status poll. The ADM-10 and ADM-11 screens themselves are A68's (AM7).
  - **M08 Wallets and custody.** Done when ONB-08 creates the passkey smart account through A29 only, renders every `registry_status` value through its pill map, and offers the second-device path without requiring it.
- **State machines:**
  - **SM-01 Onboarding case.** Done when each investor-facing code A30 derives from the 15 states (`verified`, `reviewing`, `action_needed`, `expired`, `could_not_verify`) has exactly one view, the RFI reply re-enters the machine through the API, and no screen transition exists that the API did not report.
  - **SM-02 Wholesale certificate.** Done when `requested`, `evidence_submitted`, `under_review`, `rfi`, `valid` and `rejected` each render from `classificationFor`'s shape, and renewal is never started from onboarding. The `expiring` warning and the expired text "you can hold and sell but not buy" are A59's ACC-01 job and are not drawn here.
- **Founder requirement D04.** Done when:
  - the action manifest lists every UI action with its route and delegability;
  - the node's manifest test proves the `false`, `"counsel"` and `true` meanings against the OpenAPI document and A11's `requiresHuman` flags;
  - every non-delegable act is a passkey-present act on these screens;
  - every read the screens use is callable by the person's agent within scope.

  A100 consumes the manifest later.

## Gauntlet loop for this node

- **Specify.** The frozen spec is this note plus the settled spec points:
  - (a) the entity step order, now the product-map order, confirmed with the user;
  - (b) the joint second-holder wallet keys, from A29's evidence;
  - (c) the nominated-account route, settled by A17;
  - (d) the KYB person-accept route, the KYB Resend route, the accept routes' person fields and `submit_blockers`, from A30.

  **Follow-up loops that must close before this node's Build.** Each is a change to that node's spec, not made here:
  - A52: guards in `middleware.ts` plus a `requireGuards()` wrapper awaited by every guarded page; the route-table row for ONB-07 moved to `/target-market`; the e2e glob.
  - A30: spec point (d); `signing_rule` on the create body.
  - A41 against A03 and A12: the self-certification's delegability.
  - A03 and A12: the counsel-pending code spelling.
  - A17: the deposit BSB and account on `payids/me`.
  - A50: the ONB-04 submit state and the Verifying and failed pills; the position of the joint Second person step, if A50 drew it elsewhere.

  **Inputs the node QA records.** The node QA records `specVersion` and the hashes of these contracts in the manifest:
  - A52's exports: guard registry, session, SDK factory, `declareScreen` and the e2e glob.
  - A30's eight status rows, investor-facing codes and invitation routes.
  - A32's option strings, states and error codes.
  - A29's `GET /v1/wallets/me` fields and `registry_status` vocabulary.
  - A17's PayID and nominated-account contracts.
  - A41's tax-profile shape.
  - A22's identity-document schema and persona markers.
  - A35's terms and privacy versions.
  - A36's open-offer read.
  - A50's extension file.
  - A05's component and token tables.

  **Hidden tests.** The node QA then writes the hidden tests to `<workspace>\gauntlet\hidden\A56\`:
  - a Vitest table for `stepsFor` over every sub-type, invitee and flag state;
  - Playwright probes for the route guard and the exemptions;
  - the tipping-off diff over every step;
  - the browser PII scan;
  - the token-level screenshot comparison per step and width;
  - the manifest check.
- **Build (Grok 4.7 xhigh, split by the planner).** Four parallel sub-agents on disjoint files, then integration:
  1. **Frame and navigation:** `layout.tsx`, the redirecting `page.tsx`, `steps.ts` with `registerOnboardingStep`, `guard.ts` with its exemptions and registration, the StepFrame, StepCard and stepper composition at both widths, `loading.tsx` and `error.tsx`.
  2. **People:** the join step with InvitationCard and ResidencyBlock; ONB-01; ONB-02 with the joint and entity branches; the Second person step; ONB-03 with LivenessCapture, the step-up helper and the five result views; ONB-04 with PeopleChecklist, Invite and UploadBox.
  3. **Tax, wholesale and money:** ONB-05; ONB-06 with every state view; ONB-08 with the wallet ceremony, the typed projection and status polling; ONB-09 with the nominated account, PayID and deposit details.
  4. **Status, copy, manifest and tests:** ONB-10 with `status-rows.ts`, `copy.ts` and `actions.manifest.ts`; the MSW handlers from the OpenAPI document; the component tests; the Playwright specs for J02, J03, joint (both holders) and each A22 persona; the screenshot fixtures.

  Integration then runs, in order:
  - the feature's own Vitest suite;
  - the Playwright journeys on the compose stack with A06's seed;
  - the A30, A32, A29, A17 and A41 contract tests, reused unchanged;
  - axe over every step;
  - `npm run verify`.

  It ends by writing `app/evidence/A56.json`, with screenshots at 1280 and 375 for every step and state.
- **Challenge (node QA, Opus 5.5 high).** The node QA runs the hidden tests, then works by hand against the compose stack:
  - The clean, `REFER`, `REJECT`, `FRAUD`, `PEP`, `SANCTION` and `SLOW` personas through every onboarding step, with a DOM, network and timing diff between the `SANCTION`, `PEP` and `REFER` cases.
  - Every guarded `(investor)` URL typed directly, reached by Back and Forward, and fetched as an RSC payload while `ready_to_invest` is false; every exempt route reached in the same state.
  - The stepper's locked steps requested by URL.
  - An SMSF with two 50 % owners and one verified.
  - A joint account with the second holder not yet accepted, then declined, then re-invited.
  - The wallet created before the case is approved (the pill must not be green), and the poll after A31's fixture registers it.
  - A nominated account in another name.
  - A PayID request before whitelisting.
  - Double clicks and dropped connections on every write.
  - The TFN, licence and card numbers and the DOB searched for in URLs, `localStorage`, `sessionStorage`, cookies, RSC payloads, the Next.js server log, the API log and `evidence/`.
  - An investor token replaying each write and read in the manifest, with the outcome checked against the manifest's meaning.
  - The screenshot comparison per step at both widths against the prototype and A50's extension.
  - Axe and a keyboard-only run.
  - The manifest diffed against the routes and A11's `requiresHuman` list.
  - A check that only the owned paths changed.

  The node QA writes `app/evidence/A56.qa.md`.
- **Gate.** A pass unlocks A61 and A90 and publishes the hidden tests into `app/`. There are at most three fix loops. Each loop re-runs the full hidden set and every failing challenge.
- **Escalation triggers:**
  - The same assumption fails three times (for example the entity step order or the guard's placement).
  - A fix would need a change in A30's status codes or routes, A32's option strings, A29's registry statuses, A17's contracts, A41's profile shape or self-certification default, A11's `requiresHuman` list, A12's refusal codes or A05's tokens. That is a spec defect upstream, reopened there.
  - Any finding that a screen, a poll response or a notification reveals a hit, a restriction or a freeze, or that the portfolio was reached with a non-green row. This is Critical and stops the milestone.
  - A design deviation beyond the ones this note names needs the founder's decision. It is recorded as a DF finding and asked.

## Node-specific challenges (node QA)

- **Skip to the portfolio (compliance, G01, G04).** The node QA uses the `REFER` persona in `reviewing` and, separately, a clean persona whose certificate is `under_review`. With each, the node QA:
  - types `/portfolio`, `/invest/kst01/buy` and `/wallet`;
  - uses Back after an earlier visit;
  - soft-navigates between two guarded siblings;
  - fetches the RSC payload for those pages directly;
  - flips the ONB-10 CTA's disabled attribute in devtools;
  - edits the step index in a crafted request.

  Every attempt must be a server-side redirect to `/onboarding/status` (from middleware or `requireGuards()`) with no page data in the response, and the status must still show the amber rows. Then `/support`, `/recover`, the close-account and privacy parts of `/account` and `/account/agents` must each load in the same state. One rendered portfolio byte fails the node, and so does one blocked exempt route.
- **Tipping-off through the screen (compliance, security).** Three personas start in the same minute: `REFER`, `PEP` and `SANCTION`. Each walks every onboarding step it can reach. The node QA diffs, for all three, over a scripted hour:
  - the rendered DOM of every step;
  - every network response, including ONB-08's wallet poll responses and ONB-09's PayID response;
  - the poll cadence, the copy and the expected time;
  - the notification inbox.

  Midway, A30's fixture confirms the sanctions match: A31 and A28 freeze the wallet and A17 marks the investor restricted. The three personas must still render identically. Any difference after id substitution fails the node. So does any code path in the feature folder that reads or branches on a hit, score, restriction, `frozen`, `freeze`, `restricted`, `kyt` or a hold reason.
- **The entity that invests early (money-adjacent, G03).** The node QA uses the Kwan Family SMSF with two 50 % trustees, one of them verified.
  1. The node QA submits the entity and opens ONB-10. The row must be amber: "1 of 2 people verified".
  2. The node QA tries to remove the unverified trustee through the UI after submission. The screen must show A30's `KYB_SUBMITTED_LOCKED` answer and the unchanged list from the API, not a locally edited list.
  3. The node QA approves the second person through A30's fixture and expects one poll to turn the row green.

  A green row with an unverified owner, or a portfolio reached before it, fails the node.
- **Invited people get in properly (compliance, G06, privacy).** The node QA opens the second joint holder's invitation and an SMSF trustee's invitation. Each invitee must:
  - see who invited them and in what role;
  - accept the terms and privacy consent before any identity field is shown;
  - declare 18+ and answer the residency facts.

  The node QA then checks that each invitee's `person_identity` carries all three. An expired invitation must show the "no longer valid" card. A decline must turn the first holder's `members` row to "Action needed" without the other person's reason.
- **PII in the seams (security).** The node QA uses the synthetic TFN 123 456 782, the licence "NSW 1234 5678" with its card number, and the seeded DOB. The node QA searches for these values and their masked forms in every URL (including `/onboarding/join?invite=…`), browser storage key, cookie, RSC payload, `error.tsx` message, toast, Next.js server log line, API log line, screenshot file name and evidence JSON. The saved ONB-05 must show "On file" and never the digits. One occurrence fails the node.
- **The green tick that came early (design fidelity, G01).**
  - The node QA creates the wallet while the KYC case is still `manual_review`. The second ONB-08 row must read "Pending identity verification" in grey. The address row must read `Created` in green, with the first-6-plus-last-4 truncation. The Linked pill must appear only after A31's fixture fires `identity.registered`. Setting `registry_status` to `stale` and to `deregistered` must render the same grey view.
  - Then the node QA runs the token-level comparison of every step at 1280 and 375 against the prototype's `onb` views and A50's extension: fonts, colours, radii, paddings, the Back opacity on the first step and the dynamic "Step n of N". It must pass with no raw hex or font string in the feature folder.

  A hard-coded green pill, a static "of 8", or a token drift fails the node.
- **Agent parity by manifest (D04, G22).** An investor token holding every investor scope replays each write in `actions.manifest.ts` and each read:
  - `false` entries must return `AGNT_NOT_ALLOWED` before any scope check, with one G22 gate evaluation each.
  - `"counsel"` entries (the entity submit and the tax self-certification) must return the counsel-pending code, or `AGNT_NOT_ALLOWED` where the route is also `requiresHuman`.
  - `true` entries (joint invite, KYB lookup and people, classification create and evidence, the tax-profile write, every read) must succeed within scope.
  - The manifest's non-delegable set must equal A11's `requiresHuman` flags for those routes.

  If the upstream answer on the self-certification (A41 against A03 and A12) has changed the expectation, the node QA uses the recorded answer. A missing manifest entry, or a route the screens call that the manifest does not list, fails the node.
- **Resume, retry, repeat (correctness).** The node QA:
  - closes the tab after `POST /v1/kyc/cases` returns and before the page renders;
  - double-clicks Continue on ONB-02;
  - double-clicks Accept on the join step;
  - drops the connection after `POST /v1/wallets` and retries;
  - refreshes ONB-06 mid-upload.

  The database must hold one investor, one case, one joint membership, one wallet and at most one open classification, and each reopened session must land on the correct step with the correct view. A duplicate row or a lost step fails the node.

## Pass criteria

- **Hidden tests.** The hidden tests in `<workspace>\gauntlet\hidden\A56\` pass 100 % on three consecutive runs against the compose stack with A06's baseline seed. This includes the `stepsFor` table over all nine rows (individual, foreign, joint first holder, joint second holder, SMSF, company, trust, invited entity person, and a person who reuses an approved case) with and without the `offers.retail` flag. The table must give the step lists this note specifies, and the ONB-07 slot must be empty in P1 and filled only through `registerOnboardingStep`.
- **Route guard.** At least 50 generated attempts are made (direct URL, history navigation, soft navigation between guarded siblings, RSC fetch, crafted state) to reach a guarded `(investor)` route while `ready_to_invest` is false. 100 % of them end in a server-side 307 to `/onboarding/status` with no page data. 100 % of the exempt routes load in the same state. Once `ready_to_invest` is true, 100 % of attempts reach the target. That is shown on a test-only target page registered under the guard (or on A52's placeholder page), not on A58's content.
- **Journeys.** Playwright runs of the following each pass three times in a row, with the documented view for every state:
  - J02 (clean persona to ONB-10 and the followed "Go to portfolio" link);
  - J03 (SMSF, with the invited trustee's join step and the `awaiting_cco` step, to a green ONB-10);
  - a joint account with both holders, including the join step;
  - the `REFER`, `REJECT`, `FRAUD`, `PEP`, `SANCTION` and `SLOW` personas.

  Every investor-facing status code of A30, every classification state of A32 listed under SM-02 and every `registry_status` value of A29 (`unregistered`, `registering`, `registered`, `stale`, `deregistered`) is rendered at least once and screenshotted into `app/evidence/A56/`.
- **Tipping-off.** For the `SANCTION`, `PEP` and `REFER` personas, all of these are identical after id substitution over a scripted hour, before and after the sanctions freeze:
  - the DOM of every onboarding step;
  - every response body, including the wallet poll and the PayID response;
  - the poll timings;
  - the notifications.

  A static analysis finds no read of, or branch keyed on, a hit, score, restriction, freeze, `frozen*`, `kyt*` or hold reason in the feature folder.
- **Design fidelity.** The token-level comparison (font family, size and weight, colours, radii, and paddings and gaps within 2px) passes for:
  - the designed steps at 1280 and 375 against the prototype's `onb` views;
  - ONB-04, the Second person step and the 9-step steppers against A50's extension.

  The ESLint rule finds no raw hex, font string or px radius in the feature folder. "Step n of N" is computed. The Back button has `opacity .4` and no link on the first step.
- **PII.** The scan over URLs, browser storage, cookies, RSC payloads, both server logs, screenshots and evidence finds zero occurrences of the seeded TFN, licence and card numbers, DOB or address in 100 of 100 randomised runs. The TFN field is write-only and shows "On file" after save.
- **Agent parity.** `actions.manifest.ts` lists every button and form on the eleven routes, and every listed route exists in the OpenAPI document. The `false` entries equal A11's `requiresHuman` set for those routes. Replayed under an investor token, every `false` write is refused with `AGNT_NOT_ALLOWED`, every `"counsel"` write gets the counsel-pending refusal (or `AGNT_NOT_ALLOWED` where the route is also `requiresHuman`), and every `true` action and read succeeds. `copy.ts` covers the OpenAPI error enum of every manifest route.
- **Idempotency and resumption.** Under the retry, double-click and reload probes, the database holds exactly one investor, one case, one joint membership, one wallet and at most one open classification per persona, and every reopened session lands on the current step.
- **Accessibility.** Axe reports zero critical or serious findings on every step and state. The whole individual flow and the join step complete by keyboard alone. The phone-width layout has no horizontal page scroll at 375px.
- **Contracts and seed values.** The A30, A32, A29, A17 and A41 contract tests run unchanged and green. The seed persona Alex James Morgan reproduces the design's values:
  - ONB-08: `0x7a3f…c91e`.
  - ONB-09: `062-000`, A17's `••4471` and `alex.morgan.841@pay.realx.au`.
  - ONB-10: the design's five rows first, in the design's order, green, with the design's detail strings and pills including "Valid to 30 Jun 2028"; then the wallet and bank rows, green; no `members` row.
- **Housekeeping.** Line coverage of the feature folder is at least 90 %. `npm run verify` is green. Only the owned paths changed. `app/evidence/A56.json` and `A56.qa.md` exist and match disk.

## Human gates touched

- **None crossed.** The flow runs on A22's identity and sanctions sandboxes (persona markers), A08's storage sandbox, A11's software authenticator, A29's local chain and bundler sandbox, A17's sandbox bank and the injectable clock. No DVS query, vendor SDK, real bank account, real PayID, production key or mainnet is touched.
- **HG-LEGAL (counsel questions recorded, not decided here).** Counsel's answers change data and copy rows, and this node's hidden tests are re-run after each change. The questions:
  - Which onboarding declarations must be made personally, and so stay `"counsel"` in A03's `delegable.ts`: 18+; the DVS and credit-reporting-agency consents and their wording, including "no credit check is made"; the wholesale declaration; the FATCA/CRS self-certification (CQ-10, with the A41 default conflict escalated); the entity signatory declaration.
  - Whether "country where you usually live", citizenship, visa status and the ordinary-residence answers are the right inputs for the FIRB foreign-person test, and whether tax residence should play any part in it.
  - For a joint account, whose wholesale test qualifies the account (one holder, both, or the combined assets and income). A32's `joint_account_rule = per_investor_record` is the default until then.
  - Whether a person may see a read-only portfolio before every check is green. This node blocks; it is a founder decision.
  - The FIRB banner wording for foreign persons.
  - The recovery-guardian disclosure text shown on ONB-08 (`recovery_disclosure_version`).
  - The client-money statement "Held on trust at an Australian bank" on ONB-09.
- **HG-VENDOR (later).** The production liveness capture will use the contracted identity vendor's web SDK behind the same `LivenessCapture` component boundary. The sandbox capture is a camera still or file upload. The vendor integration must pass A22's conformance suite and this node's Playwright personas unchanged before it is selected.
- **HG-PENTEST (later).** The onboarding surface is in scope of the external penetration test before go-live: session step-ups, invitation links, uploads, the route guard in middleware and pages, and browser-side PII handling. This node passes on software evidence and records the gate as open.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
