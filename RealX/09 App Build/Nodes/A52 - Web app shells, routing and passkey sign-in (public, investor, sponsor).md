---
title: "A52 - Web app shells, routing and passkey sign-in (public, investor, sponsor)"
created: 2026-09-24
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am4]
node: A52
milestone: AM4
lane: "Investor app"
oracle: no
human_gate: none
---

# A52 - Web app shells, routing and passkey sign-in (public, investor, sponsor)

**`apps/web` becomes a real Next.js application with an empty frame and no pages yet. It contains: the route groups `(public)`, `(investor)` and `(sponsor)/s/[tenant]`; the `PublicHeader` in its visitor and investor states, with a phone menu; the `SponsorSidebar`, which collapses below 1024 px; the footer with the CAR/AFSL disclosure read from the licence perimeter; the passkey sign-up, invitation acceptance, sign-in, sign-out, session-expired, login-recovery and recovery-cancel screens that replace the prototype's one-click login (design bug 16), drawn from design extension A (SGN-01 to SGN-04); sessions carried through `packages/sdk`; a route table in which every product-map screen declares its path, its owning node and folder, and the design screen it renders; the onboarding guard registry; the Toast, empty and error states every page reuses; and no role switcher: each user type arrives through its own sign-up or sign-in (D20). At phone width (375 px) every page has 16 px gutters and no horizontal scroll.**

**Build status:** not started. **Milestone:** AM4 (Design extension and shells). **Lane:** Investor app. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 8-14 agent-hours (build, challenge and verify loops).

**Depends on:** [[A05 - Design system package from the prototype tokens]], [[A11 - Authentication, sessions, RBAC and segregation of duties]], [[A09 - API and worker skeleton with OpenAPI-generated SDK]], [[A50 - Design extension A- investor and sponsor screens not in the prototype]], [[A06 - Seed data and scenario simulator]] (Alex Morgan, Mia Chen, the HCO and NLC tenants), [[A12 - API tokens, scopes, agent principals and mandates for every user type (G22)]] (the AGNT token in the D04 probes), [[A14 - Approvals engine and maker-checker (SM-00)]] (the 4E sponsor-owner confirmation in login recovery, and inline-mode fresh assertions), [[A21 - Licences, feature flags and licence perimeter (G21)]] (`/v1/perimeter/public` and the licence-bound flag state), [[A22 - Identity-DVS and sanctions sandbox adapters]] (the liveness sandbox in recovery), [[A23 - Whole-stack Docker boot (AM1 exit)]] (the compose stack the smoke and probes run on), [[A32 - Classification- wholesale certificates and target-market hooks (SM-02)]] (the classification role line; brings A30 transitively) and [[A33 - Sponsor tenancy, entities, licences of sponsors and admission pipeline]] (tenants, memberships, slugs, invitations). The graph data (`final-graph.json`) must carry the same eight added edges; this note does not edit it.
**Unlocks:** [[A54 - Public site- home, investments, property page, sponsor profile, fees]], [[A55 - Public site- how it works, returns calculator, transparency, help and documents]], [[A56 - Onboarding flow ONB-01 to ONB-10 with entity branch]], [[A57 - Buy flow- order, review and confirm, order status (INV-01 to INV-03)]], [[A58 - Portfolio, holding detail, transactions, distributions and statements]], [[A59 - AUD wallet, token wallet and account screens]], [[A60 - Investor and sponsor API tokens and agents screens (ACC-05, SPN-16)]], [[A62 - Sponsor console- apply, dashboard, team and approvals, billing]], [[A63 - Sponsor console- new listing wizard SPN-03 to SPN-06 and offer monitor SPN-07]], [[A64 - Sponsor console- register, distributions, property reporting, valuations, announcements]], [[A92 - P2- resale board screens (sell, buy on the board, market tracker, board control)]], [[A93 - P2- governance votes and meetings (SM-10) with screens]], [[A105 - Partner portal- trustee-RE, auditor, regulator observer and licensee views]]. The route table also fixes the rows that [[A90 - P2- registered scheme, sub-funds, PDS and TMD, target market (G05), retail cooling-off]], [[A95 - P3- Auto Invest and PayTo mandates]] and [[A97 - P3- minors, development and debt vehicles, licensed-market venue mode]] serve; they reach A52 transitively.

**Oracle rule:** this node has no hidden oracle (`oracle: no`) because no money, supply or holdings move here. The node QA still writes its Challenge material to `<workspace>\gauntlet\hidden\A52\` before the build starts. It contains its own copy of the frozen route table, the open-redirect corpus and the tenant-enumeration probe, the storage and bundle grep lists, 375 px and 1280 px reference crops of the prototype's header, footer, sponsor sidebar and toast, and crops of A50's SGN-01 to SGN-04 cards. The builder never reads that folder ([[App Gauntlet]] §2).

This node draws the frame, not the pictures. It holds no business rule. Eligibility, money, approvals and gate outcomes come from the API through the SDK, and every page inside the frame belongs to a dependent node. The shell has one compliance job: keep the licence perimeter visible and enforced at the route level (G21). That means the footer disclosure, the header role line, and route flags bound to licences, which hide P2 and P3 routes or show them as previews. It also owns the browser half of sign-in, which is the one surface where D04's "person must be present" acts are performed. This plan is not legal advice. The disclosure wording is a counsel question, recorded under Human gates.

## Build brief

**1. Application scaffold (`apps/web` root files, `app/layout.tsx`)**

- Stack: Next.js App Router on the 15.x line (or the line A01 pinned), React 19 and TypeScript. Every visual element comes from `@realx/ui` (A05), every API call from `@realx/sdk` (A09) and every screen code from `@realx/domain` (A02's `screens.ts`).
- The app's `eslint.config.js` adds these rules:
  - `fetch` is allowed only inside `src/shell/api/` and inside a feature's own `api.ts` that wraps the SDK.
  - No raw hex colours and no font strings; A05's rule is extended to this app.
  - No `Date.now()` and no zero-argument `new Date()` under `src/features/**`. Time comes from API responses or from the `now` field of `/v1/meta` (A57 and A56 rely on this).
  - No money arithmetic in the app. `formatAud` and `formatDateAest` from `@realx/ui` format integer cents and ISO instants from the API, and `src/shell/format.ts` re-exports the en-AU formatters.
- `app/layout.tsx` loads `@realx/ui/tokens.css` and the vendored fonts, with no Google Fonts request. A test greps the production build for `fonts.googleapis.com` and expects zero hits. The layout also:
  - sets `<html lang="en-AU">` and the viewport meta `width=device-width`;
  - paints the cream page background through the token;
  - mounts the providers from `src/shell/providers.tsx`: the SDK client, the session context, the feature-flag snapshot, and the Toast host.
- Security headers are set in `next.config.ts` and `middleware.ts` (both root files):
  - a strict Content-Security-Policy with a per-request nonce for Next's own scripts, and no `unsafe-inline` or `unsafe-eval` for scripts. A nonce forces dynamic rendering in the App Router, so the rendering rule for every group, `(public)` and `(public-info)` included, is: **pages render per request; only data is cached.** Public reads go through `src/shell/api/publicRead.ts` (`cachedPublicRead(op, params, { revalidate: 60 })`), which uses a credential-less SDK client and Next's data cache, so a cached entry can never contain a session's data. No page uses `export const revalidate`, `generateStaticParams` or full-route caching. A54's "60-second revalidation" is read as this 60-second data revalidation (A54's rendering-model note must say so; that edit is A54's);
  - `frame-ancestors 'none'`, `Referrer-Policy: strict-origin-when-cross-origin` and `X-Content-Type-Options: nosniff`;
  - a `Permissions-Policy` that allows `publickey-credentials-get` and `publickey-credentials-create` on the app's own origin only;
  - `Cache-Control: no-store` on every `(investor)`, `(sponsor)` and `(auth)` response, so the back button after sign-out never shows a cached private page.
- The app runs against the compose API on the `web` relying-party origin from `packages/config` (A11 §2: `web` serves the public, investor and sponsor surfaces; the operator console is a different origin and is not part of this app).
- `RUN_MODE` comes from A08 and has only two values in this build, `local` and `test`; adding any other value is an HG-PROD change. The only build-time switch is `REALX_TEST_HARNESS` (`on` or `off`, default `off`). It adds the `.dev.tsx` harness pages of §9, such as `/dev/fresh-assertion`, which exercise shell helpers for a person who has already signed in normally and never sign anyone in or change who is signed in. `npm run build:release` forces it `off` and runs the bundle greps of §9; with it `on`, the server refuses to start unless `RUN_MODE` is `local` or `test`. No build, in any `RUN_MODE`, contains a role or persona switcher, a persona list or an authenticator bridge; every sign-in calls the browser's own WebAuthn API (`navigator.credentials`).
- `middleware.ts` also applies the public rate limit that A55's route handlers rely on: 60 requests per minute per IP by default, from config. Excess requests get A09's envelope `429 RATE_LIMITED`. The store is an in-process token bucket, so the middleware is pinned to the Node.js runtime (`export const config = { runtime: 'nodejs', matcher }`), never Edge, where an in-process bucket would not be shared. That needs a Next.js release on the 15.x line at 15.5 or later (or the later line A01 pins); if the pinned line cannot run Node middleware, the limiter moves into a `withPublicRateLimit(handler)` wrapper in `src/shell/api/` that A55's route handlers call, and that change is escalated to A55. A unit test asserts the middleware config's `runtime` is `nodejs`. A distributed limiter is a production infrastructure item under HG-PROD. `playwright.config.ts` picks up `src/shell/**/e2e/*.spec.ts` and `src/features/**/e2e/*.spec.ts`, the glob A57 and later nodes depend on.

**2. Route groups and the route table (`src/shell/routes.ts`)**

- The route groups:

  | Group | Who can use it | Notes |
  | --- | --- | --- |
  | `app/(public)/` | anyone: visitors and signed-in people | the header changes state with the session |
  | `app/(investor)/` | a signed-in `HUMAN` principal; what it sees depends on its state (§4) | anonymous requests get a 307 to `/sign-in?return=…` |
  | `app/(sponsor)/` outside `[tenant]` (`/s/apply`, `/s`) | any signed-in `HUMAN` principal | anonymous or expired requests get a 307 to `/sign-in/sponsor?return=…` |
  | `app/(sponsor)/s/[tenant]/` | a principal with a membership of that tenant in `/iam/me`: a `sponsor_user`, or a `PARTNER` holding an SP-VAL or SP-PM membership of that tenant (A98) | anonymous or expired requests get the same 307 as above; a signed-in non-member gets the group's 404 |
  | `app/(auth)/` | anyone | the sign-in family (§5) |

  The public frame is exported as `PublicShell` so that another public-surface group can mount it unchanged. A55's `(public-info)` group does this. A `(partner)` group under `/p/[engagement]/` is declared in the route table as reserved for [[A105 - Partner portal- trustee-RE, auditor, regulator observer and licensee views]]. This node creates no page there.
- This node gives each route group four thin files: `layout.tsx`, `loading.tsx`, `error.tsx` and `not-found.tsx`. Each is a one-line re-export of a component in `src/shell/`. Dependent nodes add pages beside those files and never edit them (see Owned paths).
- `routes.ts` is the single source of truth for URLs. It has one row per product-map screen on the public, investor and sponsor surfaces, with this shape:
  ```ts
  { code: ScreenCode, path, mapRoute, aliases?: string[], group, owner: NodeId, folder, phase, flag?, whenOff?: 'preview' | 'hidden' }
  ```
  - `code` is typed by A02's `ScreenCode` union, so a typo is a compile error.
  - `owner` and `folder` name the node that owns the page and the owned folder the `page.tsx` sits in (relative to `apps/web/app/`). A build test asserts that `folder` lies inside one of `owner`'s owned paths as recorded in the graph data, and that the page found by the `defineScreen` walk sits in exactly that folder.
  - `phase`, the title, the prototype key, the design code and the owning node are not typed here. They are read from A02's `screenByCode` and `designCodeFor`, which carry the charter's reconciliation table: design `port` "INV-01" is POR-01; design `buy` "INV-02" is INV-01 and INV-02, with INV-03 as its third step; design `activity` "INV-03" is POR-04; design `spNew` "SPN-03-06" is SPN-03 to SPN-06.
  - `mapRoute` is the product map's own route; `aliases` are older paths that earlier drafts or dependent specs cited. The redirect rules:
    - **Static redirects are 308.** Wherever `mapRoute` differs from `path`, and for every alias, `next.config.ts` registers a permanent 308 to `path` (sub-paths included, for example `/account/api-tokens/:rest*`). Links into the product map's vocabulary (`/documents/tax`, `/signup/type`, `/verify/identity`) therefore still work. A58's request for the `/documents/tax` redirect is met this way.
    - **A `mapRoute` that is itself a shell page path gets no redirect.** ONB-01's product-map route is `/signup`, which is A52's own sign-up card (§5): that card is the first half of ONB-01 and satisfies the `mapRoute`, so `/signup` is never redirected to `/onboarding/account`.
    - **Session-dependent redirects are 307 with `Cache-Control: no-store`, never 308.** The product map's `/sponsor` and `/sponsor/*` routes need the signed-in person's tenant, so they are answered by A52's route handler `app/(sponsor)/sponsor/[[...rest]]/route.ts`: one membership gives a 307 to `/s/<slug>/…` (the mapped path); several give a 307 to the `/s` chooser with the target kept in `return`; none, or anonymous, gives a 307 to `/sign-in/sponsor?return=…`. `/sponsor/apply` is the one static exception (308 to `/s/apply`), because it needs no tenant. The `/s` chooser page likewise answers with a 307 and `no-store` when it redirects.
    - **Tests.** A table test asserts that no redirect source equals any row's `path`, any shell page path (§5) or any other redirect's target; that no redirect is chained; and that no session-dependent redirect is a 308.
- The paths below are frozen at Specify. Every row names its owning node and the owned folder its page sits in (folders are relative to `apps/web/app/`), and every cited path of every dependent spec (A54 to A64, A90, A92, A93, A95, A97, A105) was reconciled against this table on 24 Sep 2026. A path changes only if this spec is reopened.

  | Code | Path | Product-map route and aliases (redirect) | Phase and flag | Owner and folder |
  | --- | --- | --- | --- | --- |
  | PUB-01 | `/` | same | P1 | A54 · `(public)\` |
  | PUB-02 | `/how-it-works` | same | P1 | A55 · `(public-info)\how-it-works\` |
  | PUB-03 | `/investments` | same | P1 | A54 · `(public)\investments\` |
  | PUB-04 | `/investments/[code]` | same | P1 | A54 · `(public)\investments\[code]\` |
  | PUB-05 | `/sponsors/[slug]` | same | P1 | A54 · `(public)\sponsors\[slug]\` |
  | PUB-06 | `/fees` | same | P1 | A54 · `(public)\fees\` |
  | PUB-07 | `/calculator` | same | P1 | A55 · `(public-info)\calculator\` |
  | PUB-08 | `/transparency` | same | P1 | A55 · `(public-info)\transparency\` |
  | PUB-09 | `/help` | same | P1 | A55 · `(public-info)\help\` |
  | ONB-01 | `/onboarding/account` | `/signup` (no redirect: A52's sign-up card, §5) | P1 | A56 · `(investor)\onboarding\account\` |
  | ONB-02 | `/onboarding/type` | `/signup/type` (308) | P1 | A56 · `(investor)\onboarding\type\` |
  | ONB-03 | `/onboarding/identity` | `/verify/identity` (308) | P1 | A56 · `(investor)\onboarding\identity\` |
  | ONB-04 | `/onboarding/entity` | `/verify/entity` (308) | P1 | A56 · `(investor)\onboarding\entity\` |
  | ONB-05 | `/onboarding/tax` | `/verify/tax` (308) | P1 | A56 · `(investor)\onboarding\tax\` |
  | ONB-06 | `/onboarding/wholesale` | `/verify/wholesale` (308) | P1 | A56 · `(investor)\onboarding\wholesale\` |
  | ONB-07 | `/target-market` (`?flow=onboarding` renders inside A56's `OnboardingStepLayout`) | `/verify/target-market` (308); alias `/onboarding/target-market` (308 to `/target-market?flow=onboarding`) | P2 · `offers.retail` · `hidden` | A90 · `(investor)\target-market\` |
  | ONB-08 | `/onboarding/wallet` | `/wallet/setup` (308) | P1 | A56 · `(investor)\onboarding\wallet\` |
  | ONB-09 | `/onboarding/bank` | `/wallet/deposit` (308) | P1 | A56 · `(investor)\onboarding\bank\` |
  | ONB-10 | `/onboarding/status` | `/verify/status` (308) | P1 | A56 · `(investor)\onboarding\status\` |
  | INV-01 | `/invest/[code]/buy` | same | P1 | A57 · `(investor)\invest\[code]\buy\` |
  | INV-02 | `/invest/[code]/buy/review` | `/invest/[code]/review` (308) | P1 | A57 · `(investor)\invest\[code]\buy\review\` |
  | INV-03 | `/invest/orders/[id]` | `/orders/[id]` (308) | P1 | A57 · `(investor)\invest\orders\[id]\` |
  | INV-04 | `/auto-invest` | same | P3 · phase gate · `preview` (as A50 draws it) | A95 · `(investor)\auto-invest\` |
  | INV-05 | `/resale/[code]/sell` | same; alias `/invest/[code]/sell` (308) | P2 · `resale.board` · `preview` (the prototype's brick "Preview · the resale board opens with the retail launch" banner, line 482) | A92 · `(investor)\resale\[code]\sell\` |
  | INV-06 | `/resale/[code]/buy` | same; alias `/invest/[code]/board` (308) | P2 · `resale.board` · `preview` (A50 draws it with the PreviewBanner) | A92 · `(investor)\resale\[code]\buy\` |
  | INV-07 | `/investments/[code]/market` (public group) | same | P2 · `resale.board` · `preview` (A50 draws it with the PreviewBanner) | A92 · `(public)\investments\[code]\market\` (a carve-out from A54's `(public)\` that the graph data must record; A92's alternative, loop A54.1, is not used because A54 is built before A92's component exists) |
  | POR-01 | `/portfolio` | same | P1 | A58 · `(investor)\portfolio\` |
  | POR-02 | `/portfolio/[code]` | same | P1 | A58 · `(investor)\portfolio\[code]\` |
  | POR-03 | `/portfolio/distributions` | same | P1 | A58 · `(investor)\portfolio\distributions\` |
  | POR-04 | `/portfolio/transactions` (nav label "Activity") | same | P1 | A58 · `(investor)\portfolio\transactions\` |
  | POR-05 | `/wallet` | same | P1 | A59 · `(investor)\wallet\` |
  | POR-06 | `/wallet/tokens` | same | P1 | A59 · `(investor)\wallet\tokens\` |
  | POR-07 | `/portfolio/statements` | `/documents/tax` (308) | P1 | A58 · `(investor)\portfolio\statements\` |
  | GOV-01 | `/votes` | same | P2 · `governance.votes` · `hidden` | A93 · `(investor)\votes\` |
  | GOV-02 | `/votes/[id]` | same | P2 · `governance.votes` · `hidden` | A93 · `(investor)\votes\[id]\` |
  | ACC-01 | `/account` | same | P1 | A59 · `(investor)\account\` |
  | ACC-02 | `/account/recover` | `/recover` (308) | P1 | A59 · `(investor)\account\recover\` |
  | ACC-03 | `/account/notifications` | `/notifications` (308) | P1 | A59 · `(investor)\account\notifications\` |
  | ACC-04 | `/account/support` | `/support` (308) | P1 | A59 · `(investor)\account\support\` |
  | ACC-05 | `/account/agents` | alias `/account/api-tokens` and sub-paths (308) | P1 | A60 · `(investor)\account\agents\` (carved out of A59's `(investor)\account\`) |
  | SPN-01 | `/s/apply` | `/sponsor/apply` (308) | P1 | A62 · `(sponsor)\s\apply\` (the graph data must add this folder to A62) |
  | SPN-02 | `/s/[tenant]` | `/sponsor` (307, session) | P1 | A62 · `(sponsor)\s\[tenant]\(admin)\` |
  | SPN-03 to SPN-06 | `/s/[tenant]/listings/new/property`, `…/diligence`, `…/terms`, `…/submit` | `/sponsor/listings/new/<step>` (307, session) | P1 | A63 · `(sponsor)\s\[tenant]\listings\new\<step>\` |
  | SPN-07 | `/s/[tenant]/offers/[code]` | `/sponsor/offers/[code]` (307, session) | P1 | A63 · `(sponsor)\s\[tenant]\offers\` (the graph data must add this folder to A63; A63's loop A63.0) |
  | SPN-08 | `/s/[tenant]/operations/register/[code]` | `/sponsor/register/[code]` (307, session); alias `/s/[tenant]/register/[code]` (308) | P1 | A64 · `(sponsor)\s\[tenant]\operations\register\[code]\` |
  | SPN-09 | `/s/[tenant]/operations/distributions` | `/sponsor/distributions` (307, session); alias `/s/[tenant]/distributions` (308) | P1 | A64 · `(sponsor)\s\[tenant]\operations\distributions\` |
  | SPN-10 | `/s/[tenant]/operations/reporting` | `/sponsor/property-reporting` (307, session); alias `/s/[tenant]/property-reporting` (308) | P1 | A64 · `(sponsor)\s\[tenant]\operations\reporting\` |
  | SPN-11 | `/s/[tenant]/operations/valuations` | `/sponsor/valuations` (307, session); alias `/s/[tenant]/valuations` (308) | P1 | A64 · `(sponsor)\s\[tenant]\operations\valuations\` |
  | SPN-12 | `/s/[tenant]/operations/announcements` | `/sponsor/announcements` (307, session); alias `/s/[tenant]/announcements` (308) | P1 | A64 · `(sponsor)\s\[tenant]\operations\announcements\` |
  | SPN-13 | `/s/[tenant]/meetings` | `/sponsor/meetings` (307, session) | P2 · `governance.votes` · `preview` (A50 draws "Meetings (P2)" in the nav) | A93 · `(sponsor)\s\[tenant]\meetings\` (the graph data must add this folder to A93) |
  | SPN-14 | `/s/[tenant]/team` | `/sponsor/team` (307, session) | P1 | A62 · `(sponsor)\s\[tenant]\(admin)\team\` |
  | SPN-15 | `/s/[tenant]/billing` | `/sponsor/billing` (307, session) | P1 | A62 · `(sponsor)\s\[tenant]\(admin)\billing\` |
  | SPN-16 | `/s/[tenant]/agents` | `/sponsor/agents` (307, session) | P1 | A60 · `(sponsor)\s\[tenant]\agents\` |

  Shell and reserved paths (not product-map rows; each declares a shell screen, §5):

  | Path | Owner and folder | Notes |
  | --- | --- | --- |
  | `/sign-in`, `/sign-in/sponsor`, `/sign-in/partner`, `/signup`, `/signup/passkey`, `/verify-email`, `/sign-out`, `/signed-out`, `/session-expired`, `/recover-login`, `/recovery/cancel` | A52 · `(auth)\` | the sign-in family (§5) |
  | `/wallet-recovery/cancel` | A59 · `(auth)\wallet-recovery\` (a carve-out from A52's `(auth)\` that the graph data must record) | A59's wallet-recovery (SM-13) cancel-link page, public, inside the `(auth)` frame; A52 declares the row and the frame, A59 writes the page (A59's spec point (f)) |
  | `/s` | A52 · `(sponsor)\s\page.tsx` | the tenant chooser (one membership: 307 to it; several: a plain list; none: the group's 404 for a signed-in person) |
  | `/sponsor`, `/sponsor/*` | A52 · `(sponsor)\sponsor\[[...rest]]\route.ts` | the session-dependent 307 handler above |
  | `/p`, `/p/[engagement]/…` | A105 · `(partner)\` | reserved for the partner portal; A52 creates no page there |

  - A dependent path not in these tables is either a sub-path inside its owner's folder (for example A60's `/account/agents/new`, A62's `/s/[tenant]/billing/[invoice]`, A92's `/resale/[code]/sell/orders/[id]`), which the table's pattern for that row covers, or an extension page that must bring an A50 extension code and a row here at its own Specify (A97's `/p3/*` pages fall in this case).
  - **Changes this table asks of other notes (not made here).** A60 serves ACC-05 from `app/(investor)/account/agents/`, which is carved out of A59's `(investor)\account\` folder (A59 and A60 both carry the carve-out in their Owned paths, and the generated graph reads it from there), and cites ACC-05 at `/account/agents` (route settled on 24 Sep 2026 by the plan fixer: the product map's path, which A59 and A60 already used); A64's five rows now match its own paths; A92's loop A52.1 (INV-05 and INV-06 paths, the redirects, `preview` for INV-06 and INV-07) is met here, and INV-07's page folder moves from A92's loop A54.1 to the carve-out above; A90's spec point (c) is met; A59's point (a) is settled (ACC-05 is at `/account/agents`, in A60's carve-out) and its point (f) moves to the `/wallet-recovery/cancel` row. The graph data must add `(sponsor)\s\apply\` to A62, `(sponsor)\s\[tenant]\offers\` to A63, `(sponsor)\s\[tenant]\meetings\` to A93, `(public)\investments\[code]\market\` to A92 and `(auth)\wallet-recovery\` to A59.
  - **Reserved slug.** `apply` can never be a tenant slug, because `/s/apply` is a static segment. The routes test fails if A33's reserved-slug list lacks `apply` (a follow-up to A33 if it does).
- No path parameter or query value may carry personal data. Allowed values are:
  - property and vehicle short codes, such as `KST01` from A06;
  - sponsor and tenant slugs;
  - order, proposal and document ids;
  - the `return` and `intent` values of §5, and the `flow` value `onboarding` of ONB-07.
  
  Investor ids, emails, names, TFNs and wallet addresses never appear in a URL. A unit test over the table asserts that every parameter name is on the allow-list. The single-use link tokens of the email links (verification, invitation, recovery magic link, recovery cancel) are the one exception, on `/verify-email`, `/recover-login`, `/recovery/cancel` and `/wallet-recovery/cancel` only: the page accepts the token from the URL fragment (`#token=…`) or, if the upstream email carries it in the query, from `?token=`; it removes it from the address bar with `history.replaceState` before any other request, posts it once, and those pages send `Referrer-Policy: no-referrer`. The web-storage and URL scan (§9) ignores only that first navigation.
- Every `page.tsx` in the app exports:
  ```ts
  export const screen = defineScreen({ code, design })
  ```
  `defineScreen` comes from `src/shell/screen.ts`, which also exports `declareScreen` as an alias because A56 is written against that name. `design` is the prototype key (`port`, `buy`, `prop` and so on), `ext:<code>` for an A50 drawing, or `null`. It must agree with A02's row for that code.
  - A build-time test walks the `app/` tree and asserts that every page declares a screen, that the code exists in A02's `SCREENS` and in `routes.ts` with the same path, and that `design` agrees with A02.
  - The test then writes `apps/web/screens.generated.json`, which the [[Screen Coverage Matrix]] and the e2e fidelity step read.
  - After this node the only pages are the `(auth)` family and a placeholder `not-found` per group. The test therefore also lists, per group, the rows that have no page yet together with their owning node, and each dependent's gauntlet ticks its rows off.
- Helpers and route-level flag handling:
  - `href(code, params)` builds every URL from the table; components never hand-write a path.
  - `useFeature(flag)` reads the flag snapshot and `PHASE` from `/v1/meta` (A09). [[A21 - Licences, feature flags and licence perimeter (G21)]] binds those flags to licences. The shell requires `/v1/meta`'s snapshot to be A21's **effective** platform-scope state, that is A08's `isEnabled` with A21's database source registered (flag on AND a valid licence binding AND the phase reached), and not A08's static defaults. It also relies on A08.1, which A21 already raised: in `RUN_MODE=local` a `FLAG_<KEY>` env override may only force a licence-bound or phase flag off. If A09's `/v1/meta` does not compute the snapshot this way, that is a follow-up to A09 raised at Specify; the shell never reads staff-only `/v1/flags` and never substitutes a local source. A test sets the `FLAG_<KEY>` override for `resale.board` to `on` in `RUN_MODE=local` with no valid binding and asserts that `/v1/meta`, `useFeature('resale.board')` and the INV-05 route all read off.
  - Group layouts consult the table before any page code runs. A row whose flag is off, or whose phase is later than `PHASE`, renders one of two things. With `whenOff: 'preview'` the page gets `previewMode: true` and the shell puts A05's `PreviewBanner` above it; in preview mode the shell's SDK wrapper refuses every mutating call locally, and the API refuses it again under G21. With `whenOff: 'hidden'` the group's `not-found` renders.
  - A flag turns a route on only when A21 reports the licence binding as satisfied. The shell has no local override, and it adds no flag keys; the keys are A08's (`offers.wholesale`, `offers.retail`, `resale.board`, `governance.votes`, `payouts.stablecoin`, `platform.inflow_freeze`, `agents.traffic`).
- `src/shell/guards.ts` exports a guard registry, `registerGuard(name, { check, redirectTo, exempt })`. The default for every guard is "pass". A56 registers `onboarding` with `redirectTo: '/onboarding/status'` and its own exemption list. The `(investor)` layout runs all registered guards server-side before any page data is fetched. `(public)`, `(auth)` and `/sign-out` are never guarded.

**3. Shell components (`src/shell/`)**

- **`PublicHeader`** composes A05's component. The logo is Newsreader 500 26 with an italic green X and links to `/`; nav pills sit in the middle and the right slot has two states:
  - **Visitor.** A `Log in` outline button (to `/sign-in`) and a `Get started` primary button (to `/signup`).
  - **Signed-in investor.**
    - The person's name.
    - A role line in Geist Mono 11. It is built from `GET /iam/me` plus the classification read model ("Wholesale investor", "Sophisticated investor", "Verification in progress" while onboarding is incomplete), and is never a hard-coded string.
    - A 36 px green `Avatar` with initials.
    - An account menu with Profile and security, Recover account, Notifications, Support, API tokens and agents (ACC-01 to ACC-05) and `Sign out`.
  - **Nav sets.** Visitor: Investments and Fees, as the prototype (line 796). Investor: Portfolio, Investments, Wallet, Activity, Statements, plus Account, which A50 adds. The active pill is derived from the route table, never from string-matching the URL.
  - **Phone menu.** Below 768 px the nav collapses into a menu button (A05 `bp.phone`). The button opens a full-height cream sheet with the same links. The sheet is focus-trapped and closes on Escape or navigation. The header row keeps 16 px side padding and never scrolls sideways.
  - **Two memberships.** A person with both an investor account and a sponsor-user membership (two principals linked by `person_id`, A11 §1) sees an extra "Sponsor console" entry in the account menu, linking to `/s/[tenant]`. The two shells are never mixed on one page.
  - **Caching.** Pages render per request (§1: the nonce CSP rules out static pages), so the right slot is rendered from the session on each request; only public data reads are cached, for 60 seconds, through the credential-less `cachedPublicRead`. No name or role can therefore be cached into a shared response.
  - **Login-recovery banner.** On every signed-in investor and sponsor frame, when `/iam/me` (or the A11 read named at Specify) reports an open login recovery for the principal, a red `Banner` sits under the header: "A sign-in recovery was started for your account · Cancel". Cancel calls A11's signed-in cancel operation, and the banner then reads "Recovery cancelled". If A11 publishes no signed-in cancel operation or no read of the open request, that is a follow-up to A11 raised at Specify (A11 §9 promises a one-click cancel from any signed-in device, and this banner is that button on the web origin).
- **`SponsorSidebar`** composes A05's light sidebar:
  - **Contents.** The logo with the `SPONSOR` mono tag; the cream tenant card, showing the tenant's legal name and the signed-in user's name and role label from `/iam/me`; and nav buttons in A50's order: Dashboard, New listing, Offer monitor, Investor register, Distributions, Property reporting, Valuations, Announcements, Meetings (P2 grey pill), Team, Billing, API tokens. A route that is flag-off and `hidden` gets no nav entry.
  - **Role filtering.** Each nav entry is also shown only when the principal's effective cells in `/iam/me` include the read action of that row's screen, looked up through `@realx/domain` from the screen code (A02's screen row joined to A03's role × module × action matrix). If A02 carries no read action per screen, that is a follow-up to A02 raised at Specify, never a role list typed into the client. The result: SP-OPS and SP-IR see no Billing entry, and SP-VAL and SP-PM see only what their cells allow (A62, A64, A98). This is convenience, not control: the API still refuses, and a typed URL still reaches the page's role state. A component test renders the sidebar for each seed role (SP-OWN, SP-FIN, SP-OPS, SP-IR, SP-VAL, SP-PM) from A06's `/iam/me` fixtures and asserts the exact entry list.
  - **Widths.** At 1024 px and above it is a fixed 240 px column (`grid-template-columns: 240px minmax(0, 1fr)`). Below 1024 px it becomes an off-canvas drawer behind a menu button in a 56 px top bar that carries the logo and the tenant name.
  - **Sponsor `main`.** It keeps the design's `padding: 32px 40px 64px; gap: 26px; max-width: 1240px` and drops to 16 px side padding below 768 px.
- **`Footer`** renders the CAR/AFSL disclosure paragraph and the links Fees, Complaints and Privacy in the design's position (`padding: 28px 40px`, 12 px / 1.6, muted text, paragraph `max-width: 720px`).
  - **Text source.** The paragraph, the licensee name, the AFSL number and the phase label come from A21's `GET /v1/perimeter/public` (`disclosure_text`, `licensee_name`, `afsl_number`, `phase_label`, `wholesale_only`), which is cached for one minute. None of it is typed into the app.
  - **Offers unavailable.** When `car_active` is `false`, A21 returns an empty `disclosure_text` on purpose. The footer then renders its "offers unavailable" state: a neutral `Banner` reading "Offers are unavailable right now", no disclosure paragraph and no licensee numbers. This is a correct state, not a failure.
  - **Missing disclosure.** When `car_active` is `true` but `disclosure_text` is empty, or the endpoint cannot be read, the footer shows a red "Disclosure missing" `Banner`, and the fidelity test fails.
  - **Link targets.** The Complaints and Privacy targets are A55's document routes.
  - **Hero and aside copy.** The hero eyebrow "Wholesale investors · retail from 2027" and the property page's "Wholesale investors only" aside belong to A54. The footer and the header role line are the shell's share of G21.
- **`ToastHost` and `useToast()`** wrap A05's `Toast`: an ink pill, bottom 24 px, centred, 2.6 s, `aria-live="polite"`, one toast at a time, queue of at most three. The prototype uses toasts to fake side effects ("Opens the PDF", "Withdrawal requested"); those are not reproduced. Pages call `toast()` only after a successful SDK response.
- **Shared page states.**
  - **`ErrorState`** takes a `RealXApiError` and turns its code into a plain-English sentence from the table in `src/shell/errors.ts`.
    - Coverage: the code list is taken from `packages/sdk/openapi.json` at the Specify commit, never typed from memory. Every code is upper case, as A09 requires. It includes A09's core table (`VALIDATION_FAILED`, `INVALID_CURSOR`, `UNAUTHENTICATED`, `FORBIDDEN`, `GATE_FAILED`, `NOT_FOUND`, `CONFLICT`, `IDEMPOTENCY_IN_PROGRESS`, `IDEMPOTENCY_KEY_REUSED`, `AGENTS_PAUSED`, `RATE_LIMITED`, `INTERNAL`, `NOT_IMPLEMENTED`, `SERVICE_UNAVAILABLE`), A11's codes (`EMAIL_NOT_VERIFIED`, `VERIFICATION_EXPIRED`, `PASSKEY_WRONG_PURPOSE`, `ATTESTATION_REQUIRED`, `AAGUID_NOT_ALLOWED`, `SESSION_EXPIRED`, `DEVICE_MISMATCH`, `CSRF_REJECTED`, `ACCOUNT_DORMANT`, `ACCOUNT_INVITATION_EXPIRED`, `AGNT_NOT_ALLOWED`, the SoD and delegation codes, `FRESH_ASSERTION_REQUIRED`, `FRESH_ASSERTION_STALE`, `FRESH_ASSERTION_MISMATCH`, `RECOVERY_COOLING`, `VIEW_AS_NOT_ALLOWED`) and every module code registered at that commit, such as A12's `CONFIRMATION_REQUIRED` (428), which is the agent-confirmation flow and gets a plain sentence only.
    - Unknown codes: the page shows "Something went wrong" with the `request_id` in Geist Mono and nothing from `details`.
    - Completeness test: a unit test asserts that the table has a sentence for every code registered in `packages/sdk/openapi.json`.
    - Gate failures: a `GATE_FAILED` error renders A05's `Banner` with `gateLabel(gate)` and the `reason_codes` mapped through the same table. This is the design's inline explanatory note.
  - **`EmptyState`** is the design's dashed box, with a title, one line of copy and an optional action.
  - **`LoadingState`** uses skeleton blocks in `line.soft`. It has no spinner and causes no layout shift; the header and sidebar render at once from the session context.
- **`PageHeader`** prints the page's code the way the prototype's header does, so the fidelity check can read it. It has three parts:
  - an eyebrow carrying the product-map code, in Geist Mono 500 12 uppercase;
  - the h1, in Newsreader 400: 44 px on public and investor pages, 40 px on sponsor pages;
  - an optional subtitle.
- **`NoInvestorAccount`** is a page for a signed-in principal that belongs to a console, not the investor app, and opens an `(investor)` route (A58): a `sponsor_user`, or a `PARTNER`. It never shows an empty portfolio. For a principal with a sponsor membership it says "You're signed in to the sponsor console" and offers "Go to the sponsor console" and "Open an investor account" (to `/signup`); for any other partner it says "You're signed in to the partner portal" with a link to `/p`. It is never shown to a `HUMAN` principal with neither an investor record nor a membership (§4).

**4. Sessions and the SDK (`src/shell/api/`)**

- **Client factory.** There is one `RealXClient` factory:
  - On the server (server components and route handlers), it forwards the incoming session and device cookies and the request id.
  - In the browser, it uses `credentials: 'include'` and lets the API set cookies.
  - A bearer token sent to the web origin is ignored. AGNT principals use the API and MCP surfaces (A12, A104), never this app.
- **Browser storage.** No session id, device secret, API token or cookie value is ever written to `localStorage`, `sessionStorage`, a URL or a query string. `src/shell/storage.ts` exports `safeStorage`, a wrapper with built-in try/catch and a key allow-list; A57's quote-bounded review store uses it. The smoke (§9) scans web storage and the network log for session ids, device secrets, token prefixes and email addresses, and expects none.
- **Reading the session.**
  - `getSession()` in a server component calls `GET /iam/me` once per request (React `cache`). It returns `{ principal, personId, kind, tenant, roles, memberships, investorAccount, classificationLabel, accountState }`.
  - `useSession()` reads the same object from context in client components.
- **Group layouts.**
  - `(investor)` decides by principal state, in this order:
    1. anonymous or expired: 307 to `/sign-in?return=…`;
    2. a `sponsor_user` or `PARTNER` principal (a console principal, even if the same person also holds an investor principal linked by `person_id`): `NoInvestorAccount`;
    3. a `HUMAN` principal with no investor record and no membership (a new sign-up, or a sponsor applicant who has only A33's self-scope): every `(investor)` path except `/onboarding/account` gets a 307 to `/onboarding/account`, where A56's ONB-01 creates the investor record;
    4. a `HUMAN` principal with an investor record: the guard registry runs, then the frame renders.
  - `(sponsor)` outside `[tenant]`: `/s/apply` and `/s` are open to any signed-in `HUMAN` principal; anonymous or expired requests get a 307 to `/sign-in/sponsor?return=<path>`.
  - `(sponsor)/s/[tenant]`: every anonymous or expired request to any `/s/*` path, real slug or not, gets the same 307 to `/sign-in/sponsor?return=<path>`, so a sponsor whose session lapsed, or who follows an A43 email deep link, sees the sign-in card. The answer does not depend on the slug, so it reveals nothing. A signed-in principal without a membership of that tenant gets the group's `not-found`: always a 404, never a 403, with a byte-identical body whatever the slug, so tenant slugs cannot be enumerated. A33 already makes the API answer 404 for another tenant's id. Members in any state A33 reports (for example `pending_identity`, or a suspended tenant) get the frame; A62 renders those in-tenant states.
  - `middleware.ts` only checks whether the session cookie is present, to short-circuit obvious cases. Every authorisation decision is the API's (A11 §5), and the layouts re-check with `/iam/me`.
- **Account lifecycle.** The shell carries A11's `Account` lifecycle as a redirect rule, with one test per state:
  - `invited` (an invitation not yet accepted) goes to `/verify-email`, which says "Open the link in your invitation email to continue";
  - `email_verified` without a passkey goes to `/signup/passkey`;
  - `dormant` and `locked` render the matching sentence from the error table;
  - `closed` (including `invitation_expired`) clears the session context and lands on `/signed-out` with the sentence "This account is closed" (or the `ACCOUNT_INVITATION_EXPIRED` sentence), and no private page renders;
  - `active` proceeds.
- **Mutations and passkey step-up.**
  - Mutations from client components use the SDK's automatic `Idempotency-Key`.
  - **`withFreshAssertion({ operation, body, purpose, summarise })`** carries D04's person-present step-up for token creation and widening (A60), the bank-account change (A59) and sponsor-side approvals (A62, A63). It is the investor and sponsor counterpart of A53's "approve with hardware key" modal. Its contract follows A11 §8, which has two modes:
    - **Nonce mode** (A11, A12 and A67 routes guarded by `requireFreshAssertion`). The helper computes `payload_sha256` over `body` with `canonicalPayloadSha256` imported from `@realx/domain`: RFC 8785 canonical JSON with NFC strings, integer cents and integer units, the same function A11 uses on the server. It then renders the summary card from `summarise(body)`, the very object that was hashed, with the first 8 hex characters of the hash in Geist Mono. After the person confirms, it calls `POST /v1/auth/fresh-assertion/options` with `{payload_sha256, purpose}`, runs the WebAuthn prompt on the returned challenge, calls `POST /v1/auth/fresh-assertion/verify`, and only then sends `operation` with that same `body`. The API never supplies the summary text, so what the person reads is always what was hashed.
    - **Inline mode** (A14's `decide`, `ratify`, `veto` and `bulk-decide`). The helper takes the request's canonical payload and `payload_sha256` from A14's request read. It recomputes the hash with the same function and refuses locally (the `FRESH_ASSERTION_MISMATCH` sentence, no prompt) if the two differ. It renders the summary from that canonical payload, runs the prompt with the challenge equal to `payload_sha256`, and sends the assertion in the decision body. If A14's read returns only the hash, a follow-up to A14 for the canonical payload is raised at Specify, because the summary must be rendered from the hashed bytes.
    - **Shared canonicaliser (Specify prerequisite).** `canonicalPayloadSha256` lives in A11's API module today. A follow-up loop on A11 (id A11.3; A11.1 was the working id of the IAM admin API, now node A106, and A11.2 is A53's loop) moves that pure function into `packages/domain`, and A11 re-exports it, so the browser and the server run one implementation. A52 does not write its own copy. If A11 declines, this is escalated.
    - **Retry rule.** On a definitive `FRESH_ASSERTION_REQUIRED` or `FRESH_ASSERTION_STALE` refusal, the helper runs the ceremony once more and resends with a **new** `Idempotency-Key`. That is safe because a refusal means nothing executed, and it does not depend on whether A09 stores refusals (a follow-up to A09, shared with A60's spec point (g), asks A09 to confirm that freshness refusals are never stored as completed responses). Network-error retries inside the SDK keep the same key, as A09 requires. `FRESH_ASSERTION_MISMATCH` is never retried: it renders "What you confirmed has changed · review it again". A second refusal after the retry is shown, never looped.
    - **Not this helper.** A12's `428 CONFIRMATION_REQUIRED` is the agent-confirmation flow (a `confirmation_id` the human confirms in A60), not a human step-up; the helper never handles it, and if the web app receives it, `ErrorState` renders its sentence.
  - So every D04 non-delegable act performed in this app gets its passkey step-up by construction.
- **Session expiry.** A `401 SESSION_EXPIRED` sends the browser to `/session-expired?return=…`. A page can opt in with `preserveOnReauth`; its form state is then held in memory for the length of the re-sign-in round trip, so a buyer does not lose an order. Nothing survives a full reload unless the page itself uses `safeStorage`.

**5. Sign-in family (`app/(auth)/`)**

- **`/sign-in` (A50 SGN-01)** is the centred onboarding card: 36 px padding, radius 14, white on cream, with the logo and the "Sign in with your passkey" primary button.
  - While the browser prompts, a DropBox-style status panel shows "Waiting for Face ID, Touch ID or Windows Hello" (dashed border, Newsreader 22 title, Geist Mono caption).
  - On mount the page calls `POST /auth/passkeys/login/options` and starts a WebAuthn conditional-UI (autofill) request. The button starts a modal request.
  - `POST /auth/passkeys/login/verify` sets the session and device cookies (A11 §3). The page then goes to the validated `return` path, or to the principal's home (`homeFor(session)` in `src/shell/home.ts`, used by every landing in this section): `/portfolio` for a principal with an investor record (which the onboarding guard may redirect); `/onboarding/account` for a `HUMAN` with neither an investor record nor a membership; `/s/[tenant]` for a principal with one tenant membership (a `sponsor_user`, or a `PARTNER` with an SP-VAL or SP-PM membership), or `/s` with several; and the reserved `/p` for every other partner.
  - The error state reads "No passkey found on this device · Recover your account". A50 links it to ACC-02, but a lost login passkey is A11 §9, not wallet recovery SM-13. So the link goes to `/recover-login`, and the Specify step records this as a mapping note on A50.
  - Below the card: "New to RealX? Get started", linking to `/signup`.
  - There is no email field, no password field and no magic-link login. Email is only an identifier (A11 §2).
- **`/sign-in/sponsor` (SGN-02) and `/sign-in/partner` (SGN-03)** are the same card with the `SPONSOR` or `PARTNER` tag and a tenant or organisation line. They use the same ceremony, since the discoverable credential identifies the principal. A sponsor-user credential presented on the investor card still signs in and lands on the sponsor home; the variant only sets the frame. On `/sign-in/sponsor`, "New to RealX? Get started" links to `/signup?intent=sponsor-apply`. A105 fidelity-checks SGN-03 against its partner frame.
- **`return` validation.** The `return` parameter is accepted only when every one of these holds:
  - it is a relative path starting with a single `/`;
  - it contains no scheme, no `//`, no backslash and no encoded control or slash characters;
  - it is 512 characters or fewer;
  - it matches a row pattern in `routes.ts`.

  Anything else falls back to the principal's home.
- **Sign-in errors.**
  - A `wallet`-purpose credential produces `PASSKEY_WRONG_PURPOSE`, shown as "That passkey secures your token wallet. Use your sign-in passkey.", and no cookie is set.
  - `ACCOUNT_DORMANT`, `RATE_LIMITED`, `DEVICE_MISMATCH` and `ATTESTATION_REQUIRED` render from the error table.
  - A failed sign-in never reveals whether an email or an account exists.
- **`/signup` (the "Get started" target; the first half of ONB-01, as A56 expects).**
  - One card with an email `TextField`. It calls `POST /auth/signup` and then shows "Check your inbox" with a resend button, which obeys A11's limit of three per hour.
  - The verification link lands on `/verify-email`. That page calls `POST /auth/verify-email` and then runs the first passkey registration ceremony (`POST /auth/passkeys/register/options` and `/verify`, purpose `login`) on `/signup/passkey`.
  - **Invitations.** `/verify-email` also takes invitation tokens. Sponsor users (A33, for example an HCO invitee for J11's SPN-14 step) and partners, including valuers and property managers (A98), accept through A11's same verify-email then passkey path; the API decides which kind of token it is, and the page only posts it. If A11 publishes a separate operation for invitation acceptance, the page calls that one; the path is the same.
  - **Landing after the first passkey** is by principal, through `homeFor`: a `sponsor_user` or an SP-VAL or SP-PM partner goes to `/s/[tenant]`, where A62 renders the identity-check state A33 reports (`TENANT_USER_NOT_VERIFIED`) until the membership is verified; any other partner goes to `/p`; an investor sign-up goes to `/onboarding/account`, where A56's ONB-01 collects the rest, unless its `intent` says otherwise (below). An invitee is never sent into investor onboarding.
  - `intent` is accepted only from an allow-list: `qualify`, `watch` (with a `return` validated by the rule above), `buy:<code>` (code matching the short-code pattern) and `sponsor-apply`. The first three are carried through verification and handed to A56 unchanged; A54 uses them. `sponsor-apply` lands the new `HUMAN` principal on `/s/apply` instead of `/onboarding/account` (A62's applicant path); `/sign-in/sponsor`'s "Get started" link carries it.
  - Creating the first passkey is a D04 non-delegable act, and this ceremony can only happen with the person at the browser.
- **Sign-out.** `/sign-out` calls `DELETE /auth/sessions/{id}` for the current session only; "sign out everywhere" lives on ACC-01 in A59. It clears the session context and lands on `/signed-out`, which is SGN-04: a small centred card reading "You're signed out · Sign in again".
- **`/session-expired`** is the SGN-01 card with the `SESSION_EXPIRED` sentence, and it keeps the return path.
- **`/recover-login`** implements A11 §9 with A05's `HorizontalStepper` in four steps: Verify email, Re-verify, Wait, New passkey.
  1. **Verify email.** Calls `POST /auth/recovery/start` and always shows "If that email has an account, a link is on its way", whatever the outcome.
  2. **Re-verify.** The magic link lands here. An investor runs the identity adapter's liveness scenario (sandbox only). A sponsor user or partner sees either liveness or "Waiting for your sponsor owner to confirm", which is the 4E action through A14.
  3. **Wait.** Shows the cooling period from the API's `RECOVERY_COOLING` payload (default 24 h) as a Geist Mono countdown computed from the API's `now`, with a note that any signed-in device can cancel it from the banner on its frame (§3) and that the notice email carries a cancel link.
  4. **New passkey.** Runs the registration ceremony. On completion it signs the person in and toasts "Passkey replaced. Every other session has been signed out."

  Wallet recovery (ACC-02, SM-13) is a different flow, with its own guardian step, and A59 builds it.
- **`/recovery/cancel`** is the public landing of the cancel link in A11's recovery notice. It reads the single-use token under the link-token rule of §2, posts it once to A11's link-token cancel operation (named from `openapi.json` at Specify; if A11 publishes none, a follow-up to A11), and always shows the same neutral card, "If a recovery was in progress, it has been cancelled", whether the token was valid, used or unknown, so it never reveals an account or a requester. A reload never re-posts. The wallet-recovery cancel link is a different page, A59's `/wallet-recovery/cancel`, which uses the same `(auth)` frame (§2).
- **Layout and screen codes.** Every screen in this family is one column, at most 480 px wide, centred on cream, with the public header in its visitor state above it and the footer below. It fits 343 px of content at 375 px wide.
  - The `SGN-` codes are A50's extension-only codes, and the pages declare them as shell screens (`defineScreen({ code: 'SGN-01', shell: true })`), so they appear in `screens.generated.json` without counting against the 91.
  - Session-expired, the sign-up card, verify-email, login recovery, the recovery-cancel card and the `/s` tenant chooser are not drawn in A50. They declare shell codes from the small list in `src/shell/screen.ts` (for example `SHELL-SIGNUP`, `SHELL-RECOVERY-CANCEL`, `SHELL-TENANTS`) and are built from the SGN-01 card, the DropBox pattern and the ACC-02 stepper. The Specify step records that composition. If the founder wants them drawn, it goes back to A50 as a spec-defect back-edge.

**6. D04 parity for the shell's own actions**

- The shell exposes no business action, but it does perform session acts. `src/shell/actions.ts` lists each one with its API operation id and its agent status. A100's parity suite reads this list, so these acts are counted explicitly instead of silently missed.

  | Act | API operation | Agent status |
  | --- | --- | --- |
  | Sign up | `POST /auth/signup` | `human_only` (D04: first passkey) |
  | Verify email | `POST /auth/verify-email` | `human_only` (D04: first passkey) |
  | Register the first passkey | `POST /auth/passkeys/register/*` | `human_only` (D04: first passkey) |
  | Sign in | `POST /auth/passkeys/login/*` | `not_applicable`: agents authenticate with tokens (A12) |
  | Sign out | `DELETE /auth/sessions/{id}` | `not_applicable`: an agent's equivalent is token revocation, which A60 exposes |
  | Accept an invitation | A11's verify-email and passkey operations with an invitation token | `human_only` (D04: first passkey) |
  | Recover login | `POST /auth/recovery/*` | `human_only` (D04: account recovery) |
  | Cancel a login recovery (banner or link) | A11's cancel operations, named from `openapi.json` | `human_only` (D04: account recovery) |
  | Fresh assertion | `POST /auth/fresh-assertion/*` | `human_only` (D04: step-up) |

- An AGNT token calling any `human_only` route gets `403 AGNT_NOT_ALLOWED` from A11. The parity manifest cites that behaviour, and the shell's e2e test re-checks it through the SDK against the running A23 compose stack with a real AGNT token for Alex Morgan, seeded by A06 or issued through A12 in the test run (A99's per-user sandbox environment is not needed), never an SDK mock.

**7. Each user type arrives through its own front door (no role switcher, D20)**

- The prototype's black "view as" bar (dc lines 23-31) is a design-review device and is not built in any form, in any build or `RUN_MODE`. There is no `src/shell/dev/` folder, no persona list and no client-side role state. One browser session is one signed-in person.
- Each user type reaches its flow the way a real user does, and `homeFor` (§5) decides where they land:
  - a visitor opens `/` signed out and browses the public site;
  - a new investor follows "Get started", verifies the email and registers a passkey (§5), then continues in A56's onboarding;
  - a returning investor or sponsor user follows "Log in" (SGN-01 or SGN-02) and lands on `/portfolio` or `/s/[tenant]`; a sponsor invitee arrives from the invitation email; a sponsor applicant from `/s/apply`; a partner from SGN-03;
  - staff never enter through the web app: the console is a different origin with its own sign-in (A53).
- The only way between surfaces is the signed-in person's own memberships (the "Sponsor console" entry of §3 for a person who holds both). Nothing lets one person act as, or preview, another.
- **Automated runs sign in the same way.** Playwright opens the real sign-in page and answers the WebAuthn ceremony with a virtual authenticator attached to that person's browser context (CDP `WebAuthn.addVirtualAuthenticator` and `WebAuthn.addCredential`), loaded with that persona's sandbox passkey from A06's gitignored `packages/db/seed/.local/credentials.json`. The helper lives in `e2e/support/sign-in.ts` and is never imported by app code.
- The product-map code readout that the prototype bar carried moves to test tooling: every page declares its `screen` (§2), and the node QA reads each route's product-map and design codes from `screens.generated.json` during the fidelity walk.
- A build test asserts that the release bundle contains no `DevRoleSwitcher`, no role-switcher component, no persona name, no `/__test/` path and no sandbox-authenticator code, and that no query string such as `?dev=1` or `?as=` changes who is signed in.

**8. Phone width, accessibility, and the shell's own error and empty states**

- **Widths.**
  - At 375 px, every page of this node has `document.documentElement.scrollWidth === 375` and 16 px side gutters, with the header menu collapsed and the sponsor drawer off-canvas. Tables added by dependents scroll inside their card only.
  - At 1024 px and 1280 px, the sponsor sidebar is a visible 240 px column, and public `main` keeps `max-width: 1240px; padding: 36px 40px 64px`.
- **Keyboard and screen readers.**
  - "Skip to content" is the first focusable element.
  - The phone menu and the drawer trap focus and restore it on close.
  - The active nav pill carries `aria-current="page"`.
  - The sign-in button and the passkey panel can be reached with Tab and operated with Enter.
  - axe-core over the auth family and the three empty group frames reports zero critical or serious findings.
- **API unreachable.**
  - `(public)` pages still render the header and footer, with an `ErrorState` for the section that failed; the page never goes blank.
  - `(investor)` and `(sponsor)` layouts render `ErrorState` with a retry.
  - `503 SERVICE_UNAVAILABLE` during startup shows "RealX is starting" in local mode.
- **Unknown route.** The group's `not-found` keeps the public header or the sponsor frame and shows "Page not found" in Newsreader 44 with a link home. For a route that is in `routes.ts` but has no page yet, the same page adds a Geist Mono line naming the product-map code and its owning node, read from `screens.generated.json`, when the dev tools are compiled in.
- **Guard redirects.** When a guard redirects, the header still shows the person's name, so the state is visible, and a toast says why ("Finish verifying to continue").

**9. Tests and evidence**

- **Unit tests (Vitest and Testing Library)**, covering:
  - `routes.ts`: the parameter allow-list, agreement with A02, flags limited to A08's key list, every row's folder inside its owner's owned paths, and the redirect table test of §2 (no redirect source equals a row path, a shell page path or another redirect's target; no chains; no session-dependent 308; `/signup` has no redirect);
  - the `return` validator and the `intent` allow-list, `sponsor-apply` included;
  - `homeFor` for each principal state (investor, new `HUMAN`, one and several memberships, SP-VAL and SP-PM partners, other partners);
  - the `(investor)` access rule for each of its four states, and the `Account` lifecycle with one test each for `invited`, `email_verified`, `dormant`, `locked`, `closed` and `active`;
  - the error table's completeness; the guard registry; `safeStorage`; the sidebar entry list per seed role; the footer's offers-unavailable and disclosure-missing states; the middleware `runtime: 'nodejs'` config;
  - `withFreshAssertion` against a mocked SDK: the hash equals A11's shared test vectors; the summary is rendered from the hashed object; a `STALE` refusal gives exactly one retry with a new `Idempotency-Key`; a `MISMATCH` gives no retry; a second refusal is shown, not looped; `CONFIRMATION_REQUIRED` is not handled by the helper.
- **Build tests:** `defineScreen` coverage and `screens.generated.json`; the release-bundle greps (`fonts.googleapis.com`, `DevRoleSwitcher`, "Alex Morgan", "Mia Chen", `/__test/`, `/dev/`, sandbox-authenticator symbols); raw hex and font strings. Dev-only harness pages use the `.dev.tsx` extension, which `next.config.ts` adds to `pageExtensions` only when `REALX_TEST_HARNESS=on`, so they cannot exist in the release build.
- **Playwright smoke** (`src/shell/e2e/shell.spec.ts`), run at 375 px and 1280 px against A23's compose stack with the `web` service added. The G21, tenant-enumeration, AGNT and fresh-assertion steps call the real compose services, never SDK mocks. The script:
  1. A visitor lands on `/`.
  2. Signs up with a fresh email through the local inbox, verifies it, and registers the first passkey, landing on `/onboarding/account` (a placeholder 404 until A56).
  3. Signs out and lands on SGN-04.
  4. Signs in as Alex Morgan through "Log in"; the ceremony is answered by the virtual authenticator holding his sandbox passkey (§7).
  5. Checks the investor header's classification label from the API.
  6. Opens `/s/<HCO slug>` and gets the 404.
  7. Signs out; the back button shows no private page.
  8. Signs in as Mia Chen on SGN-02 and sees the sponsor frame with the tenant card and, at 375 px, the drawer.
  9. Walks login recovery: email, sandbox liveness, cooling advanced through A09's clock, new passkey.
  10. Starts a second login recovery for Alex Morgan, then cancels it from the banner of a second signed-in session; starts a third and cancels it through the notice's `/recovery/cancel` link in a cookie-less context. Each request ends `cancelled`, and a new-passkey registration attempt against it is refused.
  11. Accepts an HCO invitation: the test setup has Mia Chen invite a new SP-IR through A33's API; the invitee opens the link, lands on `/verify-email`, registers the first passkey and lands on `/s/<HCO slug>` (never `/onboarding/account`).
  12. As a visitor opens `/s/apply`, gets the 307 to `/sign-in/sponsor?return=/s/apply`, follows "Get started" (`intent=sponsor-apply`), verifies, registers a passkey and lands on `/s/apply` (A62's placeholder until A62 lands), not on `/onboarding/account`.
  13. Opens `/s/<HCO slug>/team` with no session and gets the 307 to `/sign-in/sponsor`; signs in as Mia Chen and returns there.
  14. On the dev-only harness page `/dev/fresh-assertion` (a `.dev.tsx` page), runs `withFreshAssertion` as Alex Morgan against A12's token-create operation: success; a stale assertion (A09's clock advanced past the 5-minute window between verify and send) leads to one retry and success; a body changed after the assertion gets `FRESH_ASSERTION_MISMATCH`, and no token is created.
- **Evidence:** screenshots in `app/evidence/A52/`, and the manifest `app/evidence/A52.json`, which records the route-table hash, the OpenAPI hash the error table was built from, the screen list, the pinned versions (Next.js, React, `@simplewebauthn/browser`, Playwright), the screenshot hashes, and the builder and verifier model versions.

## Owned paths (other nodes do not edit these)

- <workspace>\app\apps\web\ (root files only)
- <workspace>\app\apps\web\app\layout.tsx
- <workspace>\app\apps\web\app\(auth)\ (except `(auth)\wallet-recovery\`, which is A59's)
- <workspace>\app\apps\web\src\shell\
- <workspace>\app\apps\web\app\(public)\layout.tsx, loading.tsx, error.tsx, not-found.tsx
- <workspace>\app\apps\web\app\(investor)\layout.tsx, loading.tsx, error.tsx, not-found.tsx
- <workspace>\app\apps\web\app\(sponsor)\s\[tenant]\layout.tsx, loading.tsx, error.tsx, not-found.tsx
- <workspace>\app\apps\web\app\(sponsor)\layout.tsx
- <workspace>\app\apps\web\app\(sponsor)\s\page.tsx (the tenant chooser)
- <workspace>\app\apps\web\app\(sponsor)\sponsor\ (the session-dependent redirect handler)

**Root files.** `package.json`, `tsconfig.json`, `next.config.ts`, `middleware.ts`, `eslint.config.js`, `playwright.config.ts`, `Dockerfile`, `compose.web.yml`, `.env.example`, `README.md` and `screens.generated.json` (generated). The compose stack itself (`infra\`) belongs to A08 and A23.

**Shared generated file.** `apps\web\screens.generated.json` is this node's, but every later web UI node that adds or changes a page regenerates it by running the `defineScreen` build test and commits it under A01's sanctioned-regeneration rule (the test reproduces it byte for byte from that node's tree); a hand edit is out of path. A merge conflict in it is resolved by taking main's copy and re-running the test ([[Build Orchestration]] §6).

**Thin files, then frozen.** The four thin files in each of the three group folders, `(sponsor)\layout.tsx` and `(sponsor)\s\page.tsx` are listed above one by one, so they are A52's by ownership, not by exception. Each thin file is a one-line re-export from `src\shell\`. Dependents add `page.tsx` files and sub-folders beside them and never edit them. A dependent that needs something in the shell raises a numbered loop on A52, not a local patch.

**Graph data changes this note needs (recorded here and requested from the orchestrator; `final-graph.json` is not edited by this note).** A52's `owned_paths` gain the explicit thin-file, chooser and redirect-handler entries above and lose `(auth)\wallet-recovery\`. A54's `owned_paths` narrow to `app\(public)\` minus A52's four thin files and minus `(public)\investments\[code]\market\` (A54's note must say the same). A59 gains `(auth)\wallet-recovery\`, A62 gains `(sponsor)\s\apply\`, A63 gains `(sponsor)\s\[tenant]\offers\`, A92 gains `(public)\investments\[code]\market\`, and A93 gains `(sponsor)\s\[tenant]\meetings\`. A52's deps gain A06, A12, A14, A21, A22, A23, A32 and A33.

**Read, never written.** `packages\ui` (A05), `packages\sdk` (A09), `packages\domain` (A02's `screens.ts`) and the design-extension file (A50). A gap in any of them is a spec-defect back-edge raised at Specify.

## Design references

- **`PublicHeader`.** `<workspace>\design\RealX App.dc.html` lines 33-52:
  - header at line 35: `padding: 18px 32px`, gap 20, bottom border `#E3DED4`, cream background;
  - logo at line 36: Newsreader 500 26, italic green X;
  - nav pills at line 39: `8px 14px`, Geist 500 14, active background from the nav model;
  - visitor buttons at lines 44-45: `Log in` outline ink `9px 18px`, `Get started` green `10px 18px`;
  - investor name, the "Wholesale investor" role line in Geist Mono 11, and the 36 px green avatar at lines 48-49.

  Nav sets are at line 796. The tokens and type roles are in design analysis §1.1-§1.4 (component 3 in §2). A50 §2 adds "Account" to the investor nav.
- **`Footer`.** Lines 515-517: top border, `padding: 28px 40px`, gap 24, muted `#5E6166`, 12 px / 1.6. The CAR/AFSL paragraph at line 516 is placeholder copy, replaced by A21's `disclosure_text`. The links Fees, Complaints and Privacy are at line 517. Component 44.
- **`SponsorSidebar`.** Lines 522-537:
  - grid `240px minmax(0,1fr)` at line 523;
  - logo with the `SPONSOR` Geist Mono tag at line 525;
  - cream tenant card "Harbour & Co Property / Mia Chen · Sponsor owner" at line 526;
  - nav items `10px 12px`, radius 8, Geist 500 14 at line 528.

  The design's six nav items are at line 864; A50 §2 appends Valuations, Announcements, Meetings (P2), Team, Billing and API tokens. Component 36. The phone drawer and the 1024 px breakpoint are A05 §4 decisions (design analysis §6.3); the prototype does not have them.
- **`Toast`.** Line 735: fixed, bottom 24 px, centred, ink pill `14px 22px`, shadow `0 12px 30px rgba(0,0,0,.25)`, z-index 50. Component 43.
- **`PreviewBanner`.** The resale preview note at line 482 ("Preview · the resale board opens with the retail launch", brick tint `#F5E4DB` on `#7A3A1E`, radius 10) is the look for every `whenOff: 'preview'` route.
- **Prototype-only role switcher.** Lines 23-31: ink bar with `8px 20px` padding, "RealX prototype · view as" label, `#2A2D31` pill track, and the screen code at the right in Geist Mono 11. It is not built in any form (D20, §7); its screen-code readout moves to test tooling.
- **Design bug 16** (design analysis §6.4). In the prototype, `login` is a one-click switch to Alex Morgan, and no surface has a sign-in screen. This is not reproduced; §5 replaces it.
- **Sign-in family.** Design extension A ([[A50 - Design extension A- investor and sponsor screens not in the prototype]]) §2 provides SGN-01 Sign in, SGN-02 Sponsor sign-in, SGN-03 Partner sign-in and SGN-04 Signed out: the centred onboarding card (36 px padding, radius 14), the DropBox-style waiting status, the error state and the "Get started" link. The recovery stepper follows A50's ACC-02 drawing (HorizontalStepper with a DropBox-style status per step) with A11's login-recovery step labels.
- **Phone layout.** Design analysis §6.3 (public and investor surfaces at phone width with the header collapsed; consoles desktop-first with a collapsible sidebar below 1024 px; 16 px minimum gutter), and A05 §4 (`bp.phone 768`, `bp.console 1024` as tokens).
- **Screen codes.** A02's `packages/domain/screens.ts` carries the charter's design-to-canonical reconciliation, and `routes.ts` imports it rather than copying it.

## Requirements covered

- **Product-map screens.** None is owned here, but every PUB, ONB, INV, POR, GOV, ACC and SPN screen gets a row in `routes.ts` and a frame to render into. Done means:
  - The route table has all 56 public, investor and sponsor rows from A02's `SCREENS` (9 PUB, 10 ONB, 7 INV, 7 POR, 2 GOV, 5 ACC, 16 SPN), plus the reserved partner path.
  - Each row's `mapRoute` redirect works, and the design mapping comes from A02.
  - The `defineScreen` build test is enforced.
  - The first dependent to add a page in each group (A54 or A55, A56, A62) needs no change to any file this node owns.
  
  The SGN-01 to SGN-04 shell screens and the sign-up, session-expired and login-recovery cards are not product-map screens. They are the sign-in, passkey and 2FA screens that the product-map QA lists as undrawn, delivered here for the web origin; A53 delivers the console's.
- **Journeys.** None is owned. Done means the e2e smoke proves three things:
  - J01 step 1: a visitor lands on `/` and sees the header in its visitor state and the footer disclosure.
  - J02 as far as ONB-01's first half: sign-up, verified email, first passkey.
  - Every journey's "signed in as investor" or "signed in as sponsor owner" precondition is reachable through `/sign-in` with the seed personas, and J11's invited sponsor user reaches the sponsor frame through `/verify-email`.
- **Gates: G21 Licence perimeter.** Done means:
  - The footer disclosure, licensee name and AFSL number come from A21's `GET /v1/perimeter/public`, and the fidelity test fails when they are missing.
  - The header role line comes from the classification read model.
  - Every P2 and P3 route is bound to one of A08's licence-bound flags or to `PHASE`, and renders `PreviewBanner` (with mutations refused) or a 404 while off. There is no local override.
  - A unit test checks the route table's `phase` and `flag` columns against A02's phases and A08's flag keys, so a new route cannot be added without naming the licence or phase it needs.
- **Modules.** None owned. Done means: nothing.
- **State machines.** None owned. The shell carries A11's `Account` lifecycle only as the redirect rule of §4, with one test per state.
- **Founder requirement D04:**
  - The shell adds the "API tokens and agents" entry to the investor account menu (ACC-05, `/account/agents`, the product map's path, with a 308 from `/account/api-tokens`) and "API tokens" to the sponsor sidebar (SPN-16, `/s/[tenant]/agents`, also shown to valuers and property managers when their cells allow), and routes both.
  - It provides `withFreshAssertion` with a defined contract (§4), so creating or widening a token in A60, changing the bank account in A59 and approving on the sponsor side in A62 and A63 each require a passkey step-up over the exact payload the person read.
  - It performs the non-delegable "first passkey", "accept an invitation" and "account recovery" acts, recovery cancellation included, only with the person present.
  - It publishes its session acts in `actions.ts` for A100.
  - No agent traffic reaches this app; it is the browser surface for humans only.

## Gauntlet loop for this node

- **Specify.** This note is frozen together with:
  - the route table (the paths, owners and folders in §2, cross-checked against every dependent that cites a path: A54 to A64, A90, A92, A93, A95, A97 and A105), together with the graph-data changes listed under Owned paths, which the orchestrator records before the build starts;
  - the follow-ups this note raises: A11 (the shared canonicaliser in `packages/domain`, the recovery cancel operations and the open-recovery read), A09 (effective flags in `/v1/meta`; freshness refusals never stored), A14 (the canonical payload on the request read, if missing), A02 (the read action per screen, if missing) and A33 (the reserved slug `apply`, if missing);
  - the SGN region mapping in A50, including the ACC-02 versus `/recover-login` note;
  - the error table's code list, taken from `openapi.json` at that commit;
  - A08's flag keys and A21's `/v1/perimeter/public` schema;
  - the export names the dependents cite (`PublicShell`, `getSession`, `useSession`, `defineScreen` or `declareScreen`, `registerGuard`, `withFreshAssertion`, `safeStorage`, `href`, `useFeature`).

  The node QA records the SHA-256 of the dc file, A50's extension file, `openapi.json` and A02's `screens.ts` in the evidence header.
- **Oracle.** This is not an oracle node. The node QA first writes its Challenge material to `<workspace>\gauntlet\hidden\A52\`:
  - its own copy of the route table;
  - the open-redirect corpus: about 40 `return` values, including absolute URLs, protocol-relative, backslash, encoded slashes, `javascript:`, over-long paths and paths not in the table;
  - the tenant-enumeration script;
  - the release-bundle and web-storage grep lists;
  - the reference crops;
  - the axe and 375 px scripts.
- **Build (Grok 4.7 xhigh, split by the planner).** Suggested shape: four parallel sub-agents on disjoint folders, then one integration pass.
  - **(a) Scaffold and security:** root files, `app/layout.tsx`, providers, CSP with nonce and the per-request rendering rule with `cachedPublicRead`, headers, `middleware.ts` on the Node.js runtime (cookie short-circuit, public rate limit), the SDK factory, `getSession` and `useSession`, `withFreshAssertion`, `safeStorage`, the `REALX_DEV_TOOLS` and `RUN_MODE` guards, and the lint rules.
  - **(b) Shell components:** `PublicHeader` with the phone menu and per-session slot, `SponsorSidebar` with the drawer, `Footer` reading A21, `ToastHost`, the error, empty and loading states, `PageHeader`, `NoInvestorAccount`, the error table and its completeness test.
  - **(c) The `(auth)` family:** SGN-01 to SGN-04, sign-up with the `intent` allow-list, verify-email with invitations and first passkey, `homeFor`, session-expired, the recovery stepper, `/recovery/cancel` and the recovery banner, all against the compose API and the sandbox authenticator, plus `actions.ts`.
  - **(d) Routing:** `routes.ts` on A02's `ScreenCode` with owners and folders, the static 308 and session-dependent 307 redirects, the `/s` chooser and the `/sponsor` handler, `href`, `useFeature`, `defineScreen`, the guard registry, the screen build test and `screens.generated.json`, the thin files of each route group, the no-switcher build test, and the Playwright smoke.
  - **Integration:** A23's compose stack with the `web` service added; run the smoke at 375 px and 1280 px, capture screenshots, run `npm run verify` (typecheck, lint, unit, screen test, axe) and `npm run build:release` with the bundle greps, and write the manifest.
- **Challenge (node QA, Opus 5.5 high).** From a clean checkout, the node QA:
  - runs `npm run verify` three times;
  - diffs `routes.ts` against its copy and against the paths every dependent cites (A54 to A64, A90, A92, A93, A95, A97, A105), and checks each row's folder against its owner's owned paths in the graph data;
  - follows every static and session-dependent redirect, including `/signup`, `/account/api-tokens`, `/invest/KST01/sell` and `/sponsor/team` as a two-membership person, and checks status codes and caching headers;
  - cancels a login recovery from a second signed-in session and from the link;
  - accepts an HCO invitation and applies as a sponsor from a signed-out browser;
  - runs the open-redirect corpus against `/sign-in?return=` and `/signup?intent=watch&return=`;
  - runs the tenant-enumeration script;
  - builds the release bundle and greps it;
  - walks the header, footer, sidebar, toast and the SGN cards at 375 px and 1280 px against the prototype crops and A50's regions;
  - tries a wallet-purpose credential at sign-in;
  - expires a session mid-form;
  - signs out and presses back;
  - opens P2 routes with their flags off, and again with a flag on but the licence binding removed in A21's sandbox;
  - ends the CAR appointment in A21's sandbox (offers-unavailable footer) and plants an empty disclosure with `car_active: true` in a component fixture (disclosure-missing footer);
  - calls the `human_only` routes with a real AGNT token on the compose stack;
  - runs axe.

  It writes `app/evidence/A52.qa.md`.
- **Gate.** A pass unlocks the thirteen dependents. A fail returns findings by severity. There are at most 3 fix loops on the same spec, and each re-runs the whole Challenge.
- **Escalate.** Escalate to the user when:
  - three loops fail on the same assumption (for example, the sandbox authenticator cannot exercise WebAuthn conditional UI on the build machine, or Next.js's nonce-based CSP will not work with the pinned React version);
  - a fix needs A02, A05, A09, A11, A14, A21, A33 or A50 to change (a spec-defect back-edge), or a follow-up listed under Specify is declined;
  - the graph-data changes under Owned paths are not recorded before the build starts;
  - a dependent's cited path or export conflicts with the frozen table.

  Any of the following findings is Critical and stops the milestone: a private page reachable without a session, a `return` value that escapes the origin, a tenant frame shown to a principal of another tenant, a session id or token found in web storage or a URL, or, in any build, a role or persona switcher or any way to change who is signed in without a sign-in ceremony.

## Node-specific challenges (node QA)

- **Open redirect through `return` (security).**
  - *Test:* Sign in with `return=https://evil.example`, `//evil.example`, `/\evil.example`, `/%2F%2Fevil.example`, `/%5Cevil.example`, `javascript:alert(1)`, a 5,000-character path, and a relative path not in `routes.ts`. Repeat through `/signup?intent=watch&return=…` and `/session-expired?return=…`.
  - *Pass:* Every case lands on the principal's home. None reaches an external origin or an unknown route, and the URL bar after sign-in never contains the attacker's string.
- **Tenant enumeration and cross-tenant frame (security).**
  - *Test:* Signed in as Alex Morgan (investor only), open `/s/<HCO slug>`, `/s/<NLC slug>`, `/s/does-not-exist` and `/s/<HCO slug>/team`, then `/sponsor/team` (the product-map redirect). Signed in as Mia Chen (HCO owner), open `/s/<NLC slug>`. Then remove the session cookie and repeat.
  - *Pass:* Every signed-in case returns the same 404 body with no tenant name, and the median response-time difference between real and non-existent slugs is under 20 ms over 100 probes each. `/sponsor/team` for Alex Morgan (no membership) is a 307 to the sign-in or chooser path with `no-store`, never a 308. The sponsor frame with a tenant card appears only for Mia Chen on HCO's slug. Without the cookie, every `/s/*` path, real slug or not, returns the same 307 to `/sign-in/sponsor?return=<path>` with an identical body, and `(investor)` paths return a 307 to `/sign-in`.
- **A cached public page leaks a person (security, privacy).**
  - *Test:* Sign in as Alex Morgan and load `/investments` and `/`. In a second browser context with no cookies, load the same pages within the 60-second data-revalidation window.
  - *Pass:* The second context must not show "Alex Morgan", "AM" initials or "Wholesale investor". Every page response carries a fresh CSP nonce (two loads, two nonces), no page is served from a full-route cache, and `cachedPublicRead` is never called with session cookies.
- **Session expiry and the back button (security).**
  - *Test:* Advance A09's clock past the 30-minute idle timeout while a `preserveOnReauth` test form under `(investor)` holds values, then submit. Afterwards, sign out and press back three times.
  - *Pass:* The submit lands on `/session-expired` with the return path; after re-signing in with the sandbox authenticator, the form values are still there. After sign-out, no private page content renders from cache, and the network log shows `no-store` on every private response. A scan of `localStorage`, `sessionStorage` and every URL in the network log finds no session id, device secret, token prefix or email.
- **Wrong passkey and silent enumeration (security).**
  - *Test:* Present a `wallet`-purpose credential at sign-in. Then start login recovery and sign-up with one email that has no account and one that has.
  - *Pass:* The wrong credential shows the `PASSKEY_WRONG_PURPOSE` sentence and sets no cookie. For each flow, the two emails produce identical responses and identical screens, and the median response-time difference is under 20 ms over 100 probes each.
- **Agent misuse of the human-only acts (D04, agent misuse).**
  - *Test:* On the running compose stack, with a real AGNT token belonging to Alex Morgan (seeded by A06 or issued through A12), call `POST /auth/passkeys/register/options`, `POST /auth/recovery/start` and `POST /auth/fresh-assertion/options` through the SDK, and send the token as a bearer header to the web origin's `(investor)` pages.
  - *Pass:* The API calls return `403 AGNT_NOT_ALLOWED`. The web origin ignores the bearer header and treats the request as anonymous. `actions.ts` marks exactly these acts as `human_only`.
- **A role switcher or sign-in shortcut anywhere (security, D20).**
  - *Test:* Build with `npm run build:release` and with `npm run build` plus `REALX_TEST_HARNESS=on`, and grep both outputs. On the running local stack, signed in as Alex Morgan, add `?dev=1` and `?as=mia` to several URLs and set a `role` or `persona` key in `localStorage` and in a cookie. Then sign in as Mia Chen in the same browser context through "Log in".
  - *Pass:* Neither bundle contains `DevRoleSwitcher`, any role-switcher component, "Alex Morgan", "Mia Chen" or sandbox-authenticator code, and the release bundle has no `/__test/` path. No query string or stored value changes the principal `GET /iam/me` reports. Mia's sign-in is a real ceremony that writes a new `Session` row in Postgres and replaces Alex's session cookie in that context; no page ever shows both people.
- **Licence perimeter at the route level (compliance, G21).**
  - *Test:* On the running compose stack with `resale.board` and `governance.votes` off, open `/resale/KST01/sell`, `/resale/KST01/buy`, `/investments/KST01/market`, `/votes` and `/s/<HCO slug>/meetings` (as Mia Chen). Set the `FLAG_<KEY>` override for `resale.board` to `on` in `RUN_MODE=local` while A21's licence binding is unsatisfied. End the CAR appointment in A21's sandbox.
  - *Pass:* The sell, buy, market and meetings pages show `PreviewBanner`, and any mutating SDK call from them is refused by the shell and by the API. `/votes` returns the group's 404. With the override on and the binding unsatisfied, `/v1/meta` and `useFeature` read off and the routes stay in preview or 404. With the CAR ended, the footer shows the offers-unavailable state; the component fixture with `car_active: true` and an empty disclosure shows "Disclosure missing" and fails the fidelity test. A hard-coded disclosure, licensee name or AFSL number anywhere in `apps/web` is a finding.
- **Recovery takeover defence (security, D04).**
  - *Test:* Start a login recovery for Alex Morgan from a second browser. In Alex's signed-in session, check for the banner and press Cancel. Start another and open the notice's `/recovery/cancel` link with no cookies; reload it; open it with a made-up token. Then try to register a new passkey against each cancelled request.
  - *Pass:* The banner appears on the investor frame within one request of the start; both cancels end the request in `cancelled`; the link page shows the same neutral card for a valid, a reused and a made-up token and posts only once; no new passkey can be registered; the token leaves the address bar before any other request.
- **Invitee and applicant landing (security, journeys).**
  - *Test:* Accept an HCO SP-IR invitation, an SP-VAL invitation from A98's seed, and a TP-AUD partner invitation; then apply as a sponsor from a signed-out browser.
  - *Pass:* The SP-IR and SP-VAL invitees land on `/s/<HCO slug>`, the auditor on `/p`, the applicant on `/s/apply`; none of them reaches `/onboarding/account` or sees `NoInvestorAccount` on landing.
- **Design fidelity at two widths (design).**
  - *Test and pass at 1280 px:* the header measures `padding: 18px 32px`, the logo is Newsreader 500 26 with the italic green X, and the nav pills are `8px 14px`. The footer is `28px 40px` in `#5E6166` at 12 px. The sponsor sidebar is 240 px white, with the cream tenant card and A50's nav order.
  - *Test and pass at 375 px:* the header shows the menu button, and the sheet traps focus and closes on Escape. The sponsor drawer is off-canvas. `scrollWidth` equals 375 on every auth card and every group frame. The SGN cards match A50's radius 14 and 36 px padding.
  - *Finding:* any raw hex, `font-family` string or Google Fonts request in the app.
- **Error-sentence completeness (design and support).**
  - *Test:* the node QA registers a scratch error code in a sandbox module and re-emits `openapi.json`. Then it sends a `GATE_FAILED` response with `gate: 'G07'` and two reason codes, and a `500 INTERNAL`.
  - *Pass:* The completeness test fails and names the scratch code. The gate failure renders a `Banner` with "Holder cap" from `gateLabel` and two sentences, never raw codes. The 500 shows the `request_id` in Geist Mono and nothing from `details`.

## Pass criteria

- `npm run verify` is green from a clean checkout three runs in a row. It includes:
  - the `fetch`, `Date.now()`, raw hex and font lint rules;
  - the `defineScreen` build test;
  - the error-sentence completeness test;
  - the route-table parameter allow-list test;
  - the flag and phase test against A08 and A02;
  - the `return` and `intent` validator tests;
  - the guard-registry and `Account` lifecycle tests;
  - axe over the auth family and the three group frames, with zero critical or serious findings.
- `npm run build:release` and its bundle greps find zero hits for `fonts.googleapis.com`, `DevRoleSwitcher`, persona names, `/__test/` and sandbox-authenticator symbols, and the same greps over the harness build find no role switcher, persona name or sandbox-authenticator symbol.
- `routes.ts` has 56 rows plus the reserved partner path:
  - every `code` is typed by A02's `ScreenCode`, and every design mapping comes from A02;
  - every `mapRoute` that differs from `path`, other than a shell page path, and every alias returns a 308 to it (static) or a 307 with `no-store` (session-dependent `/sponsor/*`);
  - every P2 and P3 row carries an A08 flag or the phase gate;
  - every path cited by every dependent (A54 to A64, A90, A92, A93, A95, A97, A105) resolves to its row, and that row's folder lies in the citing owner's owned paths as recorded in the graph data;
  - no redirect source equals a row path, a shell page path or another redirect's target; `/signup` has no redirect; every session-dependent redirect is a 307 with `no-store`;
  - The node QA's copy diffs with zero differences.

  `screens.generated.json` lists the SGN and auth pages as present and every other row as "no page yet" with its owning node.
- Security probes:
  - Open-redirect corpus: 0 of about 40 cases leave the origin or reach a route outside the table.
  - Tenant enumeration: for a signed-in non-member, 100 random slugs plus the cross-tenant cases return byte-identical 404 bodies with a median response-time difference under 20 ms over 100 probes; for an anonymous request, every `/s/*` path returns the same 307 to `/sign-in/sponsor`; the sponsor frame renders only for a member of that tenant.
  - Back button after sign-out: 0 private responses served from cache, and every `(investor)`, `(sponsor)` and `(auth)` response carries `Cache-Control: no-store`.
  - Cached-public-page probe: 0 personal strings in the anonymous context, and a fresh CSP nonce on every page response.
  - Recovery cancel: cancelling from a second signed-in session and from the link both end the request `cancelled`, no new passkey can be registered against it, and the link page's response is identical for valid, reused and unknown tokens.
- The Playwright smoke passes at 375 px and 1280 px with every step of §9: sign-up to first passkey, sign-out to SGN-04, Alex Morgan sign-in with the investor header, a 404 on HCO's console, a clean back button, Mia Chen on SGN-02 with the sponsor frame and drawer, login recovery ending signed in with every earlier session revoked, both recovery cancels, the HCO invitee landing on the sponsor frame, the applicant landing on `/s/apply`, the signed-out sponsor deep link reaching the sign-in card, and the fresh-assertion harness (success, one retry after a stale assertion, and a refused mismatch). Screenshots of every step at both widths are in `app/evidence/A52/`, and their hashes are in the manifest.
- Fidelity: the header, footer, sponsor sidebar, toast and SGN-01 to SGN-04 match the prototype crops and A50's regions within 1 px at 1280 px on font family, size, weight, letter-spacing, colours, paddings, radii and borders. At 375 px, `scrollWidth === 375` on all of them, with 16 px gutters.
- Sessions:
  - The wrong-purpose passkey, dormant account, rate-limited and device-mismatch cases each render their sentence and set no cookie.
  - A mid-form session expiry survives one re-sign-in round trip.
  - The web-storage and URL scan of the whole smoke finds no session id, device secret, token prefix or email.
  - AGNT calls with a real token on the compose stack to every `human_only` route in `actions.ts` return `403 AGNT_NOT_ALLOWED`.
- G21:
  - On the running compose stack (not SDK mocks), with the flags off, INV-05, INV-06, INV-07, INV-04 and SPN-13 render `PreviewBanner` with their mutations refused, and the `hidden` P2 routes (ONB-07, GOV-01, GOV-02) return 404.
  - With the flag on but the licence binding absent, those routes stay off.
  - With the flag on but the licence binding absent, `/v1/meta` and `useFeature` also read off, including with the `FLAG_<KEY>` env override set to `on` in `RUN_MODE=local`.
  - The footer disclosure and licence numbers are read from A21; `car_active: false` renders the offers-unavailable state, and the missing-disclosure case (`car_active: true`, empty text) fails the fidelity test.
- Sponsor sidebar: the component test's entry list matches the expected set for each of SP-OWN, SP-FIN, SP-OPS, SP-IR, SP-VAL and SP-PM.
- `withFreshAssertion`: the unit tests of §9 pass, and the harness steps of the smoke create exactly one token and refuse the mismatched body.
- Coverage of `src/shell` is 85 percent or higher. The evidence manifest `app/evidence/A52.json` exists, matches disk, and records the route-table hash, the OpenAPI hash, the pinned versions, the screenshot hashes, and the builder and verifier model versions. The node QA's `A52.qa.md` records no open Critical or High finding.

## Human gates touched

- **No human gate is crossed.** Everything runs locally:
  - A23's compose stack (Postgres, API, worker) with the `web` service;
  - A11's sandbox software authenticator;
  - A22's sandbox identity adapter for the recovery liveness step;
  - the local email inbox for verification and magic links.

  No vendor, no real money, no keys with value, no mainnet.
- **HG-LEGAL (recorded, not decided here).** These are counsel's to decide:
  - the exact wording of the CAR/AFSL disclosure in the footer and of the "wholesale investors only" statements, and the licensee name and numbers (this node renders whatever A21 serves and treats the prototype paragraph as placeholder copy);
  - whether the sign-up, sign-in and recovery screens need a legal notice, for example the guardian or quasi-custodial disclosure flagged in architecture brief §4.4;
  - whether accepting terms at sign-up must be a personal act. This is the D04 counsel question on personal declarations; A56 owns the acknowledgement itself.
- **HG-PENTEST (later milestone).** The CSP, cookie handling, `return` validation, tenant routing, the public rate limit and the auth family are all in scope of the external penetration test before launch. This node passes on software evidence and records that gate as open.
- **HG-PROD (at go-live, not now).** The human deployment sets the production relying-party id and origin for `web`, any `RUN_MODE` beyond `local` and `test`, the distributed rate-limit store, and the release build with `REALX_DEV_TOOLS=off`. Until then, the sandbox origin from `packages/config` is used.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
