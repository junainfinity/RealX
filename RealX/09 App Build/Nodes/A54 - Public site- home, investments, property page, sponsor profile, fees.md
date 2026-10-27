---
title: "A54 - Public site: home, investments, property page, sponsor profile, fees"
created: 2026-09-24
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am5]
node: A54
milestone: AM5
lane: "Investor app"
oracle: no
human_gate: none
---

# A54 - Public site: home, investments, property page, sponsor profile, fees

**The five public pages a visitor or investor reads before any money moves, built as server-rendered pages in the `(public)` route group of `apps/web`. PUB-01 home has the hero, the how-it-works strip, open offers and an audited-numbers strip. PUB-03 investments has the product map's six filters and sorts, eligibility chips on every card, and "notify me" when nothing matches; the filtering, sorting and closest-match relaxation are done by A36's public API, so an agent gets the same view from the same parameters. PUB-04 property page has the seven tabs, a sticky aside with the soft-cap marker at the listing's own soft cap, a proof-of-property band from the last reconciliation run, and a buy button whose kind the server computes, naming the check that failed. PUB-05 sponsor profile works both as the property page's Sponsor tab and as its own route, with planned versus actual and overdue proof. PUB-06 fees shows every fee with its effective date, pending changes with their notice date, and float interest disclosed. Every figure, label, pill and computed value (discount, property value, counts, CTA kind, eligibility verdict) comes from an upstream read contract, and only approved copy is rendered. The pages add layout, not logic, and do no money arithmetic. Fidelity is checked region by region against the prototype.**

**Build status:** not started. **Milestone:** AM5 (Investor app and preview). **Lane:** Investor app. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 10-14 agent-hours (build, challenge and verify loops).

**Depends on:** [[A52 - Web app shells, routing and passkey sign-in (public, investor, sponsor)]], [[A36 - Offerings- offer terms, listing review, raise monitor (SM-04)]], [[A34 - Properties, valuations and proof of property (SM-03)]], [[A20 - Three-way reconciliation engine with zero tolerance]], [[A47 - Portfolio, holdings and transactions read models with statements]], [[A50 - Design extension A- investor and sponsor screens not in the prototype]] (PUB-05 standalone page and the drawn-screen supplements of this node's screens; the Sponsor tab is only partly drawn in the prototype, so PUB-05 counts as Partial from 24 Sep 2026)
**Unlocks:** [[A61 - Docker Desktop preview of the investor app (AM5 exit)]]

**Oracle rule:** this node has no hidden oracle (`oracle: no`), because no money, supply or holdings move here. It renders read contracts and makes two small writes (watch an offer or a search, follow a sponsor) that only create notification subscriptions. But the property page is where a failed compliance check is first named, so the node QA still writes its Challenge material to `<workspace>\gauntlet\hidden\A54\` before the build starts:
- a reference implementation of the filters, sorts and closest-match relaxation;
- the discount vectors (checked against A36's API field and the rendered line);
- the SM-03 × SM-04 × principal CTA matrix;
- the visibility matrix;
- the planted-literal copy patches;
- the 1280 px and 375 px reference crops of the prototype's `home`, `invest`, `prop` and `fees` regions.

The builder never reads that folder, and the tests are published into `app/` only after the gate passes ([[App Gauntlet]] §2).

**Graph change required (open; for the graph owner, not edited here):** this node uses interfaces and fixtures from six nodes that are not yet its ancestors in the node data or `final-graph.json`:
- A43 (AM3): published `site_copy`, announcements, fee notices and the `watch` endpoint;
- A06 (AM0): the seed, its invariants and the scenario runner the J01 journey uses;
- A23 (AM1): the whole-stack Docker boot the Playwright J01 run and the screenshots need;
- A32 (AM2): the expired-certificate fixture and the classification data behind G04;
- A42 (AM3): the AML-restriction fixture and the tipping-off rule behind the redacted CTA line;
- A104: the MCP tool names the parity map records.

The edges A43, A06, A23, A32, A42 and A104 → A54 must be added to the node data and to `final-graph.json`. All six sit in milestones before AM5, so no milestone order breaks. The scheduler does not model milestone priority, so a Specify-time check is not a substitute: **A54 is not schedulable until those six edges exist in the graph**, and the node QA refuses to open Specify if any of them is missing. **Depends on** above lists only the edges the graph holds today; it is updated when the graph owner adds the six.

This note is not legal advice. Every copy line that makes a claim ("vetted", "audited", "wholesale investors only", yields, float interest) is rendered from an approved record with its basis and date. Whether the site may show wholesale-only offers to the public at all is A21's `listing.visibility` setting and a counsel question. This node does not decide it.

## Build brief

**Scope and placement**

- **Owned paths.** There are two, both in `apps/web`: the pages under `app/(public)/`, and the feature package `src/features/public-core/`. The feature package holds the loaders over `packages/sdk`, the pure view-model functions and the page components that compose `packages/ui`.
- **The shell belongs to A52.** That covers `(public)/layout.tsx`, `PublicHeader`, `Footer`, session detection, the router conventions and `src/shell/format.ts` (which re-exports `formatAud` and `formatDateAest` from `@realx/ui`). The sign-in and "Get started" routes live in A52's separate `(auth)` group, not in `(public)`. This node does not edit any of them. A change the pages need in the shell is a spec defect routed to A52, never a local patch.
- **Components.** They come from [[A05 - Design system package from the prototype tokens]]: `FeaturedCard`, `PropertyCard`, `RaiseBar`, `Tabs`, `KpiGrid`, `FactsGrid`, `KeyValueList`, `DataTable`, `DocumentRow`, `ProofBand`, `ProofRow`, `FeeRow`, `FilterChip`, `StatusPill`, `BarChart`, `Breadcrumb`, `Banner`, `EmptyState`. This node adds no colour, font or radius of its own. `npm run verify` fails on any hex literal, `font:` shorthand or `border-radius` value in the owned path outside `packages/ui`.
- **Routes.** Product-map codes are canonical, and each page declares the design screen key it renders through A52's `defineScreen`:
  - `/` is PUB-01 (design `home`).
  - `/investments` is PUB-03 (design `invest`).
  - `/investments/[code]` is PUB-04 (design `prop`). The tab sits in the query string (`?tab=summary|distributions|capital|details|documents|sponsor|proof`), so every tab is linkable and the back button works. The prototype has no router (design analysis §0).
  - `/sponsors/[slug]` is PUB-05 (the design's Sponsor tab plus A50's standalone-page frames, `design-ext:PUB-05`).
  - `/fees` is PUB-06 (design `fees`).
  - `/how-it-works`, `/calculator`, `/transparency` and `/help` belong to [[A55 - Public site- how it works, returns calculator, transparency, help and documents]], in A55's own `(public-info)` route group that mounts A52's `PublicShell`. This node only links to them.
- **Rendering model.** The pages are Next.js App Router server components. Everything a visitor sees is rendered on the server from the public read contracts, with a 60-second revalidation to match the upstream caches (A21, A34, A36 and A47 cache their public reads for one minute).
  - Only two fragments depend on the session: the header slot (A52's) and the property page's call-to-action block. The CTA block is a dynamic server segment keyed on the passkey session. A signed-in investor's eligibility is therefore never computed in the browser and never cached across sessions.
  - **Web pages never accept API tokens.** The pages authenticate only through A52's passkey session cookie. A bearer token sent to a web route is ignored (the request is treated as a visitor's). An agent reaches every fact and action on these pages through the API and the MCP server (section 7), never through the web routes.
  - No business rule, threshold, cap, fee rate, date rule, status label, filter, sort or money arithmetic is written in this node. The pure functions in `public-core` map URL parameters to API parameters, map response fields to components, and format. Every value they show, including discounts, property value, counts and the CTA verdict, arrives computed in a response.
- **Read contracts consumed.** All must be in the OpenAPI document and `packages/sdk` before this node starts. Fields marked **(IR-n)** are not yet in the owning node's frozen contract; they are named interface requests below, and the node QA checks each one against the OpenAPI hash at Specify. A missing field is a spec defect routed to the owning node, never parsed or computed locally (for example, the state is never parsed out of "Paddington NSW 2021").
  - **A36:**
    - `GET /v1/public/offerings` (cards). Today: code, name, sub, type, sponsor short name, unit price, valuation and discount, net yield, growth, LVR, raised percent, `closes_at`, status label. Added by **IR-4**: `state` (SM-04), `regime`, `residency_rule`, `asset_class` (residential or commercial), `property_kind` (A34's taxonomy), `location_state` (state or territory code), `published_at`, `opens_at`, `supply_units`, `discount_bps` and `discount_tone` (`below` | `at` | `above`, with the "at valuation" band applied by A36 from A19's parameters), `yield_basis` and `yield_as_of`, `growth_basis` and `growth_as_of`, `proof_flag` (`ok` | `due` | `overdue` | `failed`, from A34), `cta_kind`, and the filter, sort, relaxation and count parameters in PUB-03.
    - `GET /v1/public/offerings/{code}`, which feeds the aside. Today: price, valuation, raise bar `{raised_units, total_units, soft_cap_units, soft_cap_bps, hard_cap_cents, raised_cents, closes_at, days_left}`, holder cap, lock-in, residency rule text, fee lines, reserve months, status label, CTA kind. Added by **IR-4**: every card field above, `regime`, `residency_rule` (the enum, not only its text), `per_investor_cap_bps`, `sponsor_mgmt_fee_bps_pa`, `platform_fee_bps_pa`, `transaction_fee_bps`, `fee_schedule_doc_id`, `payment_rule`, `sponsor_retained_stake_bps`, `state_reason` (a public category for `suspended` or a hold), `material_change_at` and `reconfirm_until` (for `reconfirming`), `checks_needed[] {gate_id, title}` (the gates an order on this listing needs, from A19's `order_eligibility` set and A03's gate titles, for the visitor's list), and `cta_kind` with SM-03 folded in (PUB-04, "Call to action").
  - **A34:**
    - `GET /v1/properties/{id}/valuation/current` (`null` with `reason: none_published`). It must carry `property_value_cents`, `nav_per_unit_cents`, `valuation_date` and `valid_until` (**IR-4** if the public shape lacks `property_value_cents`; A34's `Valuation` record already stores it).
    - `GET /v1/properties/{id}/valuations` (published history).
    - `GET /v1/properties/{id}/proof`: `register_chain {units, tokens, checked_at, result}` plus the five rows `{kind, title, detail, when, state}`.
    - **IR-4:** the public projection of the property master (address, configuration, land, year built, kind, photos with captions, description, gross rent, trust, trustee, custodian, debt), either as a public A34 route or folded by A36 into `GET /v1/public/offerings/{code}`, plus the public `lifecycle_state` category A36 needs to fold SM-03 into `cta_kind`. This node never calls the staff `GET /v1/properties/{id}`.
  - **A20:** `GET /v1/public/recon/{vehicle}` returns `{register_units, chain_supply, matched, checked_at}`.
  - **A47:**
    - `GET /v1/public/vehicles/{vehicleCode}/distributions`: `items[] {month, per_unit_cents, total_net_cents, paid_at, status}`, from closed runs only.
    - `GET /v1/public/transparency/{vehicleCode}`: the trust row `{result: matched | break | not_issued, escrow_application_units, break_opened_at, halted, halt}`, which is A20's public row unchanged plus A19's G10 halt, and `contracts[] {token_address, identity_registry, compliance, explorer_url}`.
    - `GET /v1/public/transparency`: `totals[] {metric, value_cents | value, as_of, attestation}`. These are A45's attested metrics (latest publication only); A47 omits any metric without an attestation. This is the audited-numbers strip's only source.
  - **A33:** `GET /v1/public/sponsors` and `GET /v1/public/sponsors/{slug}`: `name`, `short_name`, `about`, `admitted_on`, `facts[]` with `source`, the `Licence` fact, `status`, and the audited performance lines `capital_raised_cents`, `distributions_on_time_pct`, `proof_of_property_score`, `retained_stake_bp` per vehicle, `incidents`, `complaints_resolved`, each with `as_of`. Added by **IR-2** and **IR-6** (below).
  - **A35:** `GET /v1/documents/public?scope=platform|vehicle:<id>` and `POST /v1/documents/versions/{id}/open` (a presigned download; an open row only for a signed-in person, `open_recorded: false` for a visitor).
  - **A21:** `GET /v1/perimeter/public`: `{representative_name, licensee_name, afsl_number, car_active, phase_label, wholesale_only, retail_launch_label, disclosure_text, links[]}`, plus the public flag states for `resale.board` and `offers.retail`.
  - **A43:** `GET /v1/public/site-copy/{copy_key}` (published `site_copy`, including `GA-01`, the general-advice warning A55 also renders), `GET /v1/public/announcements?vehicle=&kind=` (published `significant_event` and public `fee_notice` items), and the watch routes `POST /v1/me/watches`, `DELETE /v1/me/watches/{id}` and `GET /v1/me/watches`. A43 has no public KPI kind; public KPIs are A45's, read here through A47.
  - **A19:** `POST /v1/rules/simulate` with the pre-trade eligibility kind of **IR-5** (`kind = order_eligibility`, `dry_run = true`) under the self shape. This is the one no-write evaluation the property page, the buy stepper's first screen and an agent's MCP tool all call, so all three give one answer.
- **Interface requests to settle at Specify.** Each is named so the reviewer sees it. None may be filled with a local substitute; if one is not answered by its owner, this node waits.
  1. **A43 watch endpoint: answered.** A43's spec now has `POST /v1/me/watches {kind: offer_open | listing_match | sponsor_listing, target}` (idempotent per person, kind and canonical target), `DELETE` and `GET`, token-reachable and audited. Two details remain for A43 and A33: `listing_match` accepts only `{asset_class?, state?, property_type?, max_min_investment_cents?}`, so PUB-03's watch carries only those filters (status, eligibility and sponsor are not part of a saved search, and the button caption says so from approved copy); and `sponsor_listing` targets a tenant id, so A33's public sponsor read must return the watch target for a slug (or A43 accepts the slug).
  2. **A33 track record (open).** Per-vehicle `yield_planned_bp` (with its IM date), `yield_paid_bp` and growth-versus-plan, each with `as_of`, in the public sponsor contract. A33's monthly snapshot computes them, but its public response lists only the platform-level lines. A33 also returns the `track_record_band` per vehicle (the band decision is A33's, not this page's). Founder default for A33's bands: "on track" at or above 95 percent of plan, "slightly under" from 80 to 95 percent, "under" below 80 percent, and "too early" with fewer than three paid months.
  3. **A36 featured flag (optional).** A curated `featured` flag, only if the founder wants one. Without it, the default rule in PUB-01 applies.
  4. **A36 and A34 public fields (open).** Every field marked IR-4 above, and on `GET /v1/public/offerings` the query parameters `investment`, `type`, `state`, `status`, `eligibility`, `sponsor`, `sort` (`featured`, `newest`, `below_valuation`, `lowest_price`, `net_income`, `lvr`, `growth`), `closest=true`, `cursor` and `limit`, returning `{items[], n, m, common_supply_units | null, relaxed_filter | null, facets {type[], state[], sponsor[]}, next}`. `n` (offerings) and `m` (sponsors) count the whole filtered set, not the page. With `closest=true` and no match, A36 relaxes one filter at a time in the fixed order Sponsor, Type, Location, Status, Eligibility, Investment and returns the first non-empty result with `relaxed_filter` named. Ties break by `code`. A36 also folds SM-03 into `cta_kind` with the precedence in PUB-04. A34 supplies A36 with the property projection, `property_value_cents` and `proof_flag` per vehicle.
  5. **A19 and A42 pre-trade eligibility (open).** A19 owns a gate-set row `order_eligibility` (the pre-trade check with no quantity and no acknowledgement yet): G22 (for a token), G01, G02, G04 (P1) or G05 (P2), G06, G07 headroom, G10, G14 and G21, with G03, G11, G12 and G16 returned as `deferred` (checked at the order, INV-01 to INV-03). The response carries the ordered first failure as an **investor-safe line** `{gate_label, reason_text, fix_it_href}`: A19, using A42's tipping-off rule, replaces any G02 (sanctions or PEP review) or G14 (AML) failure with the approved generic line "Buying is unavailable while we review your account" and no gate id. The page renders only that line.
  6. **A33 public counts and proof summary (open).** `Trusts on RealX` counts only publicly visible vehicles (a vehicle whose offering is hidden under A36's rules is not counted, so a hidden listing does not leak through a count), and the public sponsor read returns the proof summary `{ok, total, as_of, items[] {vehicle_name, row_title, state}}` over the sponsor's public vehicles and, per vehicle, `retained_stake_bp`, the committed `sponsor_retained_stake_bps` and a `stake_shortfall` boolean.
  7. **A06 and A20 seed (open).** (a) A06 adds a scenario with an approved or scheduled offering (for example `coming-soon`: APR-2290 passes and MPA01 moves to `approved` with an `opens_at`), used for every Coming soon assertion here; the base seed has none. (b) A20 and A06 pin that an offering before mint reports `result = not_issued` with `escrow_application_units` (A06 currently reconciles the live offers as `matched` with register 0 and chain 0).
  8. **A03 matrix cells (open).** The scopes these calls check must be A03 matrix cells (A03 §8: a scope id is an action id). Requested: the public reads stay A09 `public` routes (no scope; A47's `public:read` name must resolve to a cell or to that literal); the watch and follow calls use the cell behind A43's `watches:write` (proposed `ACC.watch.create` and `ACC.watch.delete`, delegable); the document open uses A35's cell for `POST /v1/documents/versions/{id}/open`. Whatever ids A03 publishes are the ones the parity map records; this node invents none.

**1. PUB-01 Home (design `home`).**

- **Hero.**
  - The eyebrow is A21's `retail_launch_label` (the design reads "Wholesale investors · retail from 2027"). The page never hard-codes the phase.
  - The h1 and paragraph are the published `site_copy` items `home.hero.title` and `home.hero.body`. The design strings are the seeded approved versions.
  - The primary button "Browse investments" goes to `/investments`. The outline button "Check if you qualify" goes to A52's sign-up route (ONB-01 with `intent=qualify`).
- **FeaturedCard.**
  - It shows the first `live` or `soft_cap_reached` offer in A36's public list order, which is publication order. In the seed that is Kent Street Terrace (the design's `feat = decorate(byId.p1)`). A curated flag replaces this rule only if the founder asks for one.
  - Its pill binds to the status label (design bug 12: the prototype hard-codes "Open offer"). Its 6 px RaiseBar shows `raised_units / supply_units`. Its mono line reads "{raised}% raised · closes {closes_at}" in `Australia/Sydney`.
  - When no offer is open, the slot shows the newest "Coming soon" offer with "opens {opens_at}". When there is none either, the slot is omitted and the hero spans one column. (The base seed has no Coming soon offering; this fallback is tested in A06's `coming-soon` scenario, IR-7.)
- **How-it-works strip.** The three cells (`01 Verify once`, `02 Buy tokens`, `03 Get paid monthly`) come from the `site_copy` items `home.how.1..3`. Each cell links to A55's `/how-it-works`, which reuses this cell for four steps.
- **Open offers.**
  - `GET /v1/public/offerings?status=open`, rendered as `PropertyCard` home variant (no sponsor pill, 4 px raise bar), in A36's list order so the seed reproduces the design's Kent Street, Ann Street, Unley Road sequence. The grid is capped at six, with the "All investments" link.
  - The product map's live-offers strip also names "coming soon" and "resale". When coming-soon offers exist, one mono line under the grid reads "and {n} coming soon →" and links to `/investments?status=coming_soon`. Resale entries appear only when A21 reports `resale.board` on (P2).
  - An empty open list renders a single cream `Banner`, "No offers open right now · see what's coming".
- **Audited-numbers strip.** The product map requires "audited numbers: capital raised, distributions paid, investors". The design analysis §6.2 (last bullet) decides it is a small strip in the `KpiGrid` style.
  - It shows A47's `GET /v1/public/transparency` `totals[]` for `capital_raised`, `distributions_paid` and `investors_active`, in that order (the product map's three). These are A45's attested metrics, latest publication only. Each tile is `{label, value, as_of}`: the label is the published `site_copy` item `home.kpi.<metric>`, the value is formatted from the response, and the mono caption reads "audited · as at {as_of}".
  - Every row carries an `attestation` reference. The page never sums anything itself and never shows a "tokenised value" figure (D17).
  - When no metric is published, the strip is absent, not zeroed. A row without an `attestation` (A47 already omits these; the page checks again) is dropped, not rendered.
- **Not built, with reasons.**
  - The product map's BrickX-style product tiles (Build My Own, Wholesale investors, SMSF and trusts, Become a sponsor) are superseded by the design. "Become a sponsor" (SPN-01) is reachable from A52's footer and from PUB-05.
  - The product-map banner "Wholesale investors only until our retail launch" is the hero eyebrow plus A52's footer disclosure, both from A21.

**2. PUB-03 Investments (design `invest`).**

- **One API call.** The page calls A36's `GET /v1/public/offerings` (IR-4) with the URL's filter and sort parameters passed through unchanged, and renders what comes back. Filtering, sorting, closest-match relaxation, counts and facets are A36's. The page holds no filter or sort logic, so an agent calling the same endpoint (or its A104 MCP tool) with the same parameters gets exactly the same items, counts and relaxed filter.
- **Header.** Eyebrow "PUB-03", h1 "Investments", then the count line "{n} property trusts from {m} sponsors. Prices are per token; each property has {supply} tokens."
  - `n` and `m` are the response's counts over the whole filtered set, never a count of the rendered page (design bug 11 hard-codes "3 sponsors").
  - `supply` is the response's `common_supply_units`. When it is `null` (mixed supplies), the sentence ends after "per token".
- **Filters.** All six product-map filters are present. The design draws Type and Sort only. The other rows use the same `FilterChip` rows with the 56 px group label. Chip options come from the response's `facets` and A21's flags, never from a list in the page.
  - **Investment:** All, Residential, Commercial (`asset_class`).
  - **Type:** A34's property kind taxonomy (House, Unit, Multiunit, Offices, Industrial, Retail, Build-to-rent in the seed), from `facets.type`.
  - **Location:** state or territory, from `facets.state` (A36's `location_state`, never parsed from the address text).
  - **Status:** Open, Coming soon, Funded. "Funded" is added because the design lists funded trusts; "Resale" appears only when `resale.board` is on.
  - **Eligibility:** Wholesale. "Retail" appears only when `offers.retail` is on.
  - **Sponsor:** short names from `facets.sponsor`.
- **Sorts.** The product map's six, plus the design's "Featured", each a value of A36's `sort` parameter:
  - Featured (`featured`): A36's list order.
  - Newest (`newest`): `published_at` descending.
  - Below valuation (`below_valuation`): A36's `discount_bps` descending.
  - Lowest token price (`lowest_price`).
  - Net income (`net_income`): net yield descending.
  - Loan-to-value ratio (`lvr`): ascending, debt-free first.
  - Historical growth (`growth`): descending.

  The design's chip names ("Lowest price", "Highest net income", "Lowest LVR", "Highest growth") are the labels. The product map's names are the sort keys. A36 breaks ties by `code`, so results are deterministic.
- **URL state.** Filters and sort live in the URL (`?investment=&type=&state=&status=&eligibility=&sponsor=&sort=&cursor=`), with the same names as A36's query parameters. A filtered view can be shared and the back button restores it.
  - `public-core/query.ts` only validates and passes the parameters through (an unknown value is dropped, not guessed). A36 enforces visibility and state, so no parameter can reveal a hidden offering.
  - **Pagination.** The page requests A36's default page size and shows a "Show more" link carrying `next` as `?cursor=`. The count line always uses the response's whole-set `n` and `m`.
- **Cards.** `PropertyCard` list variant, with:
  - the status pill (top-left, tone from A36's label table) and the white sponsor pill (top-right);
  - the name, and "{sub} · {type}" (A36's `sub`);
  - the price in mono 22, with the discount line from A36's `discount_bps` and `discount_tone`: green "{x}% below valuation" for `below`, brick "{x}% above valuation" for `above`, "At valuation" for `at`. The page does no subtraction or division and holds no band; the "at valuation" band is A36's (from A19's parameters).
  - the three metrics: Net yield, 5-yr growth, and LVR or "Debt free";
  - the product map's eligibility chips under the metrics, as grey `StatusPill`s: "Wholesale" (`regime = s708_wholesale`), "Residents only" (`residency_rule = residents_only`), "FIRB check" (`after_firb_check`), and "Retail" (`au_mis_retail`, P2);
  - the amber "Proof due" chip when A36's `proof_flag` is `due`, `overdue` or `failed` (red for `failed`).

  A yield always carries its basis and date from `yield_basis` and `yield_as_of`, for example "net yield · forecast from the IM dated {date}" or "paid · trailing 12 months to {date}". A card whose yield has no basis in the response renders the yield slot empty and is a spec defect to A36 (A43's `CLAIM_YIELD_NO_BASIS`), never a bare number.
- **No results.** When the response has no items, the page calls the same endpoint with `closest=true` and shows "No investments match", plus the closest matches A36 returns.
  - A line names the filter A36 relaxed (`relaxed_filter`), using the filter's label.
  - A "Notify me" outline button follows. For a signed-in investor, it calls A43's `POST /v1/me/watches` with `kind = listing_match` and the criteria A43 accepts (investment → `asset_class`, `state`, type → `property_type`; the other filters are not saved, and the approved caption `invest.notify_scope` says so), and flips to "We'll notify you". An agent does the same through the API or MCP tool.
  - For a visitor, it links to A52's sign-up route with `intent=watch` and `return=/investments?{query}` (A52's allow-list and its `return` validator). There is no anonymous e-mail capture in P1 (a counsel question, listed under Human gates).

**3. PUB-04 Property page (design `prop`).**

- **Head.**
  - The breadcrumb reads "Investments / {name}".
  - Left column pills: the status label with its tone, the property type in grey, and A21's `phase_label` in grey (the design's "Wholesale · P1").
  - Then the h1 name and the line "{sub} · {kind} · listed by {sponsor}", where the sponsor links to `?tab=sponsor`.
  - The 2fr/1fr gallery shows A34's photos (`object-fit: cover`, alt text from the captions). The "+ {n} photos" tile opens a simple lightbox. When the seed has no images, placeholders use the design's stripe style.
  - A "Share" text button copies the canonical URL. It is client-only and sends nothing.
- **Sticky aside.**
  - Token price and Token valuation in mono 30.
  - The discount line "{disc} · valued {MMM yyyy}": `{disc}` is A36's `discount_bps` and `discount_tone`, rendered as on the cards, and the date is A34's current `valuation_date`. A `null` valuation renders "Valuation pending" in `text.3` with no discount. A valuation past its `valid_until` adds "Valuation review due" beside the date.
  - The three metrics follow.
  - For `live` and `soft_cap_reached` offers:
    - the 8 px `RaiseBar`, with the soft-cap marker at `soft_cap_bps / 100` percent, not the design's fixed 75 percent;
    - the mono line "{raised}% of hard cap · soft cap {soft_cap_bps / 100}%";
    - the approved sentence `prop.escrow_note` ("Offer closes {closes_at}. Money waits in escrow; refunded automatically if the soft cap isn't reached.").
  - For a `suspended` offer, or any public state with an active hold, the amber `Banner` "Offer paused · {state_reason category}" replaces the bar.
  - For a `reconfirming` offer (A03's SM-04 extension (b): a material change opened a re-confirmation window), the raise bar stays, frozen, and the amber `Banner` from the approved `site_copy` item `prop.terms_changed` reads "Terms changed {material_change_at} · investors who applied are re-confirming until {reconfirm_until}". New applications wait; applicants re-confirm in their portfolio (A57, INV-03).
  - Under the CTA sits the note "{transaction_fee_bps / 100}% transaction fee. Read the information memorandum before investing." When A21 reports `wholesale_only`, it adds "Wholesale investors only."
- **Call to action, computed on the server, never in the browser.**
  - The button is rendered by the session-keyed segment from `{cta_kind, label, href, blocked_by?}`.
  - **Public visibility and `cta_kind` come only from A36's public read.** A36 folds the property's SM-03 state into `cta_kind` (IR-4) with this precedence, highest first: (1) not publicly visible → 404; (2) SM-03 `legal_hold`, `exit_proposed`, `exit_approved`, `under_sale`, `sold` or `wound_down` → `unavailable`; (3) a G10 halt on a funded trust → `unavailable` in place of `sell_resale`; (4) otherwise the SM-04 mapping below. The page applies no SM-03 or SM-04 rule of its own; the table under "Public states" is what the node QA checks A36's folding against.
  - **Visitor:** the kinds are:
    - `sign_up` "Sign up to invest" (open offer) goes to A52's sign-up route with `intent=buy:<code>`. Under it, the list "To invest in this listing you'll need" names the checks from A36's `checks_needed[]` (the gate titles, for example identity, wholesale status, residency and FIRB for this listing's `residency_rule`, holder cap), each with its approved `site_copy` explanation `checks.<gate_id>`. This is J01 step 4 for a visitor.
    - `notify_me` "Notify me" (coming soon).
    - `funded_no_resale` "Fully funded", disabled, while `resale.board` is off.
    - `sell_resale` "Sell tokens (resale · {year})" when the board is on (P2).
    - `unavailable` for `closed`, `finalising` and the SM-03 exit and hold states, with the status label as caption.
    - `did_not_proceed` for `failed`, `refunding` and `refunded`.
    - `paused` for `suspended`, a hold, and `reconfirming`.
  - **Signed-in investor on an open offer (public `cta_kind` is `sign_up`):** the segment calls A19's `POST /v1/rules/simulate` with `kind = order_eligibility` and `dry_run = true` (IR-5) for this person and offering, under the person's passkey session. A19 owns the gate set (G01, G02, G04 in P1 or G05 in P2, G06, G07 headroom, G10, G14, G21; G03, G11, G12 and G16 deferred to the order) and the order of failures. The page chooses no gates and filters none.
    - When the outcome passes, the button is `buy` "Buy tokens", going to `/invest/<code>/buy` (the buy flow, A57).
    - When it fails, the button is `blocked`: disabled in the design's `opacity: .45` state. Under it, a `GateRow`-style line renders A19's investor-safe first failure `{gate_label, reason_text, fix_it_href}` exactly as returned, with the link. Examples:
      - "Wholesale status (G04) · certificate expired 30 Jun 2026 · Renew in Profile";
      - "Residency and FIRB (G06) · this listing is residents only";
      - "Holder cap (G07) · you can buy up to {n} more tokens".
  - **AML, sanctions and PEP.** A G02 (sanctions or PEP review) or G14 (AML) failure is never explained. A19, applying A42's tipping-off rule, already returns the approved generic line "Buying is unavailable while we review your account" with no gate id (IR-5). The page has no mapping of its own; it renders what A19 returns and never receives the underlying reason.
  - **Agents.** A web page never runs as a token (see "Rendering model"). An agent gets the same answer by calling the same `POST /v1/rules/simulate {kind: order_eligibility}` with its token (or the A104 MCP tool generated from it) plus A36's public `cta_kind`. The token's verdict adds G22 at the head of the set, so a token whose mandate excludes the listing gets "Agent authority (G22) · this listing is outside the token's allowed trusts". The node QA asserts that the page's CTA for a person equals the API CTA for that person's token in every case except that added G22 line.
  - **Watching and selling.** The signed-in `notify_me` calls A43's `POST /v1/me/watches` with `kind = offer_open` and the offering id. A visitor's `notify_me` goes to sign-up with `intent=watch` and `return=/investments/{code}`. A holder's "Sell tokens" is P2 and shows only with the board flag.
- **Tabs,** in the design's order, each a server-rendered section chosen by `?tab=`:
  - **Summary:**
    - A34's description.
    - The six key facts:
      - Property value = A34's `property_value_cents` from the current published valuation, as served (never `nav_per_unit_cents × supply_units`, which is net assets, not property value, whenever the trust has debt; the design's formula is superseded by A06's valuation rule);
      - Gross rent;
      - Tokens = `supply_units`;
      - Cash reserve = "{reserve_months} months";
      - Next valuation = `valid_until` as `MMM yyyy`;
      - Holder cap = "{per_investor_cap_bps / 100}% per investor".
    - Under the facts, "Sponsor updates": A43's published significant-event notices for the vehicle, with title and date. A withdrawn notice stays listed with a grey "Withdrawn" pill, as A43 keeps it. Routine updates go to holders only and are not shown.
    - A link "Model returns" to A55's `/calculator?vehicle={code}`.
  - **Monthly distributions:**
    - The 12-bar `BarChart` of A47's paid `per_unit_cents` for the trailing twelve months, with the caption "Paid to wallet by the {payment_rule} · {reserve_months}-month cash reserve held".
    - Months without a paid run on a funded trust are empty labelled slots.
    - A trust that has not paid yet, such as an open offer, renders `EmptyState` "No distributions paid yet · the first run follows the first full month after units issue". The page draws no forecast bars. A forecast would need its own approved claim, and the design's bars on an open offer are mock data.
  - **Capital returns:** the six most recent published valuations from A34, as the light-green bars, with the caption "Independent valuation · net assets ÷ {supply_units} tokens". With fewer than two, it shows one bar and the line "First valuation {date}".
  - **Details:** `KeyValueList` rows:
    - Address, Configuration, Land, Year built;
    - Trust: "{name} Trust (unit trust)", or the P2 sub-fund name;
    - Trustee: "{entity} (own AFSL)" or "Professional RE (hired)";
    - Custodian;
    - Debt: "LVR {n}%" or "None";
    - Foreign investors, from `residency_rule`: "Not permitted (established residential)", "Permitted after FIRB checks", or "Permitted".
  - **Documents:**
    - A35's public vehicle documents as `DocumentRow` (title, "PDF · {pages} pages · v{n}, {date}"). Superseded versions sit under "Previous versions" with the grey "Superseded" pill and stay openable.
    - "Open ↗" downloads through A35. For a signed-in investor or a token, that writes A35's open row, which INV-02's G12 acknowledgement can later cite. For a visitor it writes nothing.
    - This page records no acknowledgement. The design's toast "G12 acknowledgement recorded" is a prototype shortcut; G12 lives in INV-02 (A57).
  - **Sponsor:** the two cards of PUB-05 (item 4) from A33's public read, with the eyebrow "PUB-05 · Sponsor" and the link "Full sponsor profile" to `/sponsors/{slug}`.
  - **Proof of property:**
    - The `ProofBand`, "Register and chain agree", reads "Trustee register {register_units} units = {chain_supply} tokens on-chain · checked {checked_at}" with the MATCHED pill. The data comes from A20's `recon/{vehicle}`.
    - Below it, A34's five `ProofRow`s, in A34's order with A34's strings.
    - Below those, the "View register" facts (a product-map action): the token contract address from A47's `contracts[]`, and "View on explorer ↗" only when `explorer_url` is not `null` (it is `null` on the local chain).
    - For an offering before mint, A47's `result = not_issued` gives "Applications {escrow_application_units} (escrow) · units issue at close" (design bug 14), with no MATCHED pill. This depends on IR-7 (b): A06 currently reconciles the live offers as `matched` with register 0 and chain 0, and the page does not reinterpret a `matched` result from the offering state.
- **States the prototype does not draw.** They are composed from A05 components and the TONE map. Every field comes from upstream.
  - **Reconciliation break.** `matched = false`, or A47's `result = break`, turns the band `red.tint`/`red`. It reads "Register and chain differ · break opened {break_opened_at} · we're checking this with the trustee", and the tab label gets a red dot.
  - **Halt.** When A47 reports `halted`, a dark `Banner` above the tabs reads "Resale halted · {cause} · expected to reopen {criteria}" (P2).
  - **Proof due or overdue.** A proof row in `due` or `overdue` renders amber with its `when`. It adds the amber chip "Proof due" to the tab label and to this listing's cards on PUB-01 and PUB-03.
  - **Proof failed.** A `failed` row renders red.
  - **Proof pending.** A `pending` row reads "before first period" in grey.
  - **Offer did not proceed.** A `failed`, `refunding` or `refunded` offer replaces the raise bar with "This offer did not proceed · refunds {in progress | completed}".
  - **Stale valuation.** Whether a stale valuation or a break halts resale is decided by A19 and A20. This page only shows their flags.
  - **Terms changed.** A `reconfirming` offer shows the "Terms changed" banner in the aside (above).
- **Public states (SM-03 × SM-04).** A36 decides visibility and `cta_kind`; this table is the expected result the node QA's matrix checks. "404" means no public row and no existence leak.

  | SM-03 property state | Public? | Offering states possible | `cta_kind` | Banners and extras |
  |---|---|---|---|---|
  | `sourced`, `screening`, `rejected`, `due_diligence`, `ic_review`, `ic_approved`, `title_verification`, `title_verified` | 404 | none public | none | none |
  | `spv_formed` | public only while the offering is `approved` or `scheduled`; otherwise 404 (`draft`, `compliance_review`, `cancelled`) | `approved`, `scheduled` | `notify_me` | "Coming soon · opens {opens_at}" |
  | `listed` | public | `live`, `soft_cap_reached` | `sign_up` (visitor); `buy` or `blocked` (signed in) | raise bar, escrow note |
  | `listed` | public | `suspended` or a hold | `paused` | "Offer paused" banner |
  | `listed` | public | `reconfirming` | `paused` | "Terms changed" banner |
  | `listed` | public | `closed`, `finalising` | `unavailable` | "Closed" or "Closed · allotting" |
  | `withdrawn` | public (applicants keep the page); 404 when the offering is `cancelled` | `failed`, `refunding`, `refunded` | `did_not_proceed` | "This offer did not proceed · refunds …" |
  | `funded` | public | `finalising`, `minted` | `unavailable` (finalising) or `funded_no_resale` / `sell_resale` (minted) | not-issued proof line until mint |
  | `operating` | public | `minted` | `funded_no_resale` / `sell_resale`; `unavailable` under a G10 halt | paid-distributions chart; halt banner if halted |
  | `distressed` | public | `minted` | as `operating` (A48 usually halts resale, so `unavailable`) | A34's public proof state in amber or red, never the internal cause |
  | `legal_hold` | public | `minted` | `unavailable` | red public proof state; halt banner |
  | `exit_proposed`, `exit_approved`, `under_sale` | public | `minted` | `unavailable` | status label as caption |
  | `sold`, `wound_down` | public (holders' records) | `minted` | `unavailable` | status label as caption |

**4. PUB-05 Sponsor profile (`/sponsors/[slug]` and the Sponsor tab).**

- **The design's two cards come first, unchanged.**
  - The sponsor card: the name in Newsreader 30, the about text, and the "Vetted by RealX · admitted {MMM yyyy}" badge from `admitted_on`.
  - The `FactsGrid`: Trusts on RealX, Trusts managed, Assets managed, Licence. Any fact with `source: sponsor_declared` carries the mono caption "sponsor-declared".
- **The product map's blocks follow, on the standalone page only,** in the design's language:
  - **Header line:** "{city} · {licence} · since {year}". When A33 returns it, this adds "A${x}m raised on RealX · audited · as at {as_of}" from `capital_raised_cents`. When it is absent, the figure is omitted, not zeroed.
  - **Planned vs actual:** a `DataTable` with one row per active vehicle, from IR-2. The columns are Trust, Yield plan (with its IM date), Paid (with `as_of`) and Growth vs plan. The last column is a `StatusPill` mapped from A33's `track_record_band`: green "on track", amber "slightly under", red "under", grey "too early". The page divides nothing and compares nothing: every percentage and every band is A33's.
  - **Skin in the game:** "keeps {retained_stake_bp / 100}% of {trust}" per vehicle, from A33's register-derived figure, beside "committed {sponsor_retained_stake_bps / 100}%", both from A33's public read (IR-6). A33's `stake_shortfall` flag renders the row amber; the page does not compare the two figures.
  - **Proof of property:** "Items current {ok} of {total}" from A33's proof summary (IR-6), with A33's `proof_of_property_score` and its `as_of`. Each item A33 lists (`due`, `overdue` or `failed`) renders as "{trust} · {row title} · {state}" with the amber or red chip. An overdue item therefore shows here as well as on the property page, as the product map requires, and an agent reads the same summary from the same endpoint.
  - **Incidents and disclosures:** the sponsor's published significant-event notices from A43, in date order. A33's `incidents` and `complaints_resolved` lines show with their `as_of`, followed by the approved line `sponsor.complaints_line`, which links to A55's complaints policy document.
  - **Follow sponsor:** an outline button. A signed-in investor calls A43's `POST /v1/me/watches` with `kind = sponsor_listing` and the sponsor's watch target (IR-1), and the button flips to "Following"; an agent makes the same call with its token. A visitor goes to A52's sign-up route with `intent=watch` and `return=/sponsors/{slug}` (A52's allow-list has no `follow:` intent; after sign-up the person lands on the profile and presses Follow).
  - **Apply link:** a small "List your trusts on RealX" link to SPN-01 (A50's placement).
- **Sponsor status.**
  - When A33 returns `status` `suspended` or `offboarded`, the page stays reachable for holders' sake. It shows A33's neutral "Not currently listing" state, with no Follow button and no open offers.
  - A slug that is not yet approved is a 404, with no existence leak.

**5. PUB-06 Fees (design `fees`).**

- **Header:** eyebrow "PUB-06", h1 "Fees", and the design's paragraph from `site_copy` `fees.intro`.
- **Platform fee list.** The `FeeRow` list is the current published platform fee schedule from A35 (`doc_type = fee_schedule_platform`). Each row is `{title, value, description}`, and the list carries the mono caption "effective from {effective_from} · v{n}". The design's six rows are the seeded version 1.
- **Pending changes.** Any published `fee_notice` from A43 whose `effective_from` is in the future renders above the list as an amber `Banner`: "Fee change from {effective_from} · {summary} · notice given {published_at}". This is the product map's 30-day notice. A43 enforces the notice period; this page only shows the dates.
- **Float interest.** The "Float interest" row's value and description are whatever the approved schedule says ("Disclosed" in the seed), and the row links to A55's `/transparency`. How interest on client money is treated is A16's counsel question. No wording is hard-coded here.
- **Fees per listing.** A second block lists every visible offering's three fee lines from A36, as `FeeRow`s with a link to that listing's `fee_schedule_doc_id`.
  - Both the design's "RealX platform fee 0.25% p.a." and the product map's per-investor sponsor platform fee are just rows, if and when the founder approves them (design analysis §6.2). Neither is written in copy.
  - Sponsor-side fees render only when A35 publishes them publicly. Otherwise the block says "Sponsor fees are set out in the sponsor agreement".

**6. Copy, claims and the perimeter (G17, G21).**

- **Approved copy only.** Every visible string in the owned path is one of three things:
  - a design string on the allow-list `public-core/copy.allowlist.json` (generated from the prototype, checked in, diffed by the node QA);
  - a `site_copy` item published by A43 with an approval record;
  - a formatter output over a response field.

  A lint in `npm run verify` extracts JSX text and template literals in the owned path and fails on anything else. New copy is added by publishing a `site_copy` item, never by editing a page. A withdrawn `site_copy` item leaves its slot empty; it does not fall back to the old text.
- **Perimeter strings.** The eyebrow, the "Wholesale investors only" note and the phase pill come from A21's `GET /v1/perimeter/public`. The footer disclosure is A52's.
- **General-advice warning.** The product map puts "the general-advice warning" on PUB-01, and these pages show yields, growth and buy buttons. PUB-01, PUB-03, PUB-04, PUB-05 and PUB-06 therefore render the published `site_copy` item `GA-01` (the same key A55 seeds and renders, set with counsel) as a 13 px `text.3` block above A52's footer, and PUB-04 also renders it under the aside's CTA note. If `GA-01` is not published, the slot is empty and every price and yield block on these pages renders the cream `Banner` "Offers unavailable right now" (fail closed, as for G21). If the founder prefers the warning in the shared public footer, that is an A52 change raised as a loop; until then it lives here.
- **Failing closed.** When A21 reports `car_active = false`, or A36's public read fails closed with `G21_VISIBILITY_HIDDEN`, three places render the cream `Banner` "Offers unavailable right now" with no prices: the home page's offers, the investments list and every property aside. The pages still render their non-offer content.
- **Visibility modes.**
  - `listing.visibility = signed_in` renders cards with no price, valuation, discount or raise bar, plus a "Sign in to see prices" button.
  - `verified_wholesale` also requires an active classification. A36 enforces it; this page only reflects it.

**7. Sessions, agents and parity (D04, G22).**

- **Reads.** Public reads need no principal. They are A09 `public` routes; a token calling them needs no scope beyond a valid token (A47 calls this `public:read`; the name must resolve to an A03 cell or to A09's literal, IR-8).
- **Writes.** The three person-bound calls (watch, follow, document open) go through the SDK under the person's passkey session from the pages, and under a bearer token from an agent. They carry idempotency keys, and they are the same endpoints. The scopes are A03 matrix cells, named in IR-8: the cell behind A43's `watches:write` (proposed `ACC.watch.create` and `ACC.watch.delete`) for watch and follow, and A35's cell for `POST /v1/documents/versions/{id}/open`. A missing cell is a spec defect to A03; this node names no scope of its own. An open made only by a token does not count towards a human G12 acknowledgement (A35's rule).
- **MCP tools.** The tools for these operations are the ones A104 generates from their operationIds. This node adds no tool by hand.
- **Parity map.** `app/evidence/A54.json` maps every fact, filter, sort, count, relaxation, summary, tab and action on PUB-01, PUB-03, PUB-04, PUB-05 and PUB-06 to its endpoint, its query parameters and its A104 MCP tool name. Each PUB-03 filter and sort maps to a parameter of A36's `GET /v1/public/offerings`; the PUB-05 proof summary and stake shortfall map to A33's public sponsor read; the CTA maps to A36's `cta_kind` plus A19's `order_eligibility` simulate. No row may map to a web URL. The later agent-parity suite (A100) consumes that map.
- **No token sessions on web pages.** Web pages never accept API tokens (see "Rendering model"). An agent's CTA is the API answer described under PUB-04 "Agents": the same `order_eligibility` evaluation for the token's person, with G22 at the head of the set. A token whose mandate excludes the listing therefore gets `blocked`, naming G22, from the API.
- **Audit.** Every watch, follow or document open made under a token is audited as "agent {prefix} on behalf of {person}" (A10's `on_behalf_of_id`) and appears in the person's activity. This node shows nothing more than the flipped button.

**8. Layout, accessibility, performance and errors.**

- **Phone width** (design analysis §6.3). At 375 px:
  - no horizontal page scroll, and 16 px gutters;
  - the property page's columns stack, and the aside's CTA repeats as a sticky bottom bar;
  - the tabs scroll horizontally and the filter rows wrap;
  - tables scroll inside their card.

  At 1240 px, the prototype's grid values apply exactly: `max-width: 1240px`, `padding: 36px 40px 64px`, gaps 48/32/20.
- **Accessibility.** WCAG AA on the real tokens (A05 measures contrast). Tabs and chips are keyboard-operable with `aria-selected`. The raise bar is a `progressbar` with `aria-valuenow`. The CTA's blocked reason sits in an `aria-live` region.
- **Upstream failures.** An upstream 5xx renders the page with the affected block replaced by the cream `Banner` "This section is temporarily unavailable" and a retry. It never renders a blank page.
- **Missing pages.** An unknown `code` or `slug`, including a hidden offering (A36 item 6), is a 404 inside the design's shell.
- **Empty data.** A `null` valuation, an empty distributions history, zero documents, zero proof rows and zero announcements each have the state named above.
- **Formatting.** Only A52's `src/shell/format.ts` formatters are used (they re-export `formatAud` and `formatDateAest` from `@realx/ui`). Times render in `Australia/Sydney`. Prices, fees and per-unit amounts render to the cent. The key-facts amounts Property value and Gross rent render in whole dollars as the design does (`$(…, 0)`), through `formatAud`'s whole-dollar option; if `@realx/ui` has no such option, that is a request to A05, not a local formatter. Units render with thousands separators and percentages to one decimal. The page does no money arithmetic.
- **Evidence.** `app/evidence/A54.json` lists:
  - the routes;
  - the upstream endpoints each block reads;
  - the allow-list hash;
  - the screenshot set: five pages at 1280 px and 375 px, plus every named state;
  - the parity map;
  - the answers to the eight interface requests (IR-1 to IR-8).

## Owned paths (other nodes do not edit these)

- <workspace>\app\apps\web\app\(public)\ **excluding** <workspace>\app\apps\web\app\(public)\investments\[code]\market\ (A92's INV-07 page, A52 route table) and A52's four thin files in `(public)\` (`layout.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx`)
- <workspace>\app\apps\web\src\features\public-core\

Inside `(public)\`, the files A52 created for the shell stay A52's: `layout.tsx` and the header and footer wiring. The sign-in and get-started routes are in A52's separate `(auth)` group, and the formatters are A52's `src/shell/format.ts`. This node adds `page.tsx`, `investments/`, `investments/[code]/`, `sponsors/[slug]/` and `fees/`. A55 owns `/how-it-works`, `/calculator`, `/transparency` and `/help` in its own `(public-info)` group, which mounts A52's `PublicShell`.

This node edits no API module, no `packages/ui` component and no seed row. A needed field, component state or seed value is a spec defect routed to the owning node: A36, A34, A20, A47, A33, A35, A21, A43, A19, A05, A06 or A52.

## Design references

DESIGN = `<workspace>\design\RealX App.dc.html`, run by `support.js` (read-only; the dc runtime is not ported). ANALYSIS = the design analysis (§1 tokens, §2 components, §3.1 screens, §4 acceptance items, §6 differences, §7 seed).

**`design:home`** (DESIGN lines 56-114, `sc-if s.home`):
- the hero grid `repeat(auto-fit, minmax(340px, 1fr))` with gap 48 px;
- the eyebrow in Geist Mono 500 12 px, `.08em`, uppercase green (line 59);
- the h1 in Newsreader 400 64 px/1.02, `-.025em`, with the green `<em>` (line 60);
- the paragraph at 18 px/1.5 in `#45484D`, max-width 520 px;
- the primary and outline-ink buttons (lines 63-64);
- the how-it-works strip (lines 79-87): 1 px `#E3DED4` grid, cream cells, numeral in Newsreader 32 green, title in Geist 500 16;
- "Open offers": the h2 with the "All investments" link (lines 88-92) and the card grid `repeat(auto-fill, minmax(280px, 1fr))` with gap 20 (lines 93-111).

Mock data: `howSteps` (line 812), and `feat`/`openProps` (line 897).

The audited-numbers strip is not drawn in the prototype and is not in A50's frozen list. It is composed as a `KpiGrid` in the style ANALYSIS §6.2 names, matching A50's PUB-08 audited-totals `KpiGrid` and its mono "Audited to …" caption.

**`design:FeaturedCard`** (DESIGN lines 67-77):
- a white card with radius 14 px and shadow `0 30px 60px -30px rgba(23,25,28,.25)`;
- a 260 px photo, with the pill at 14 px/14 px (`#E3EEE8`/`#1F5E4B`, Geist Mono 500 11 px uppercase `.05em`);
- body padding 22 px, name in Newsreader 26;
- the 6 px `RaiseBar` (track `#ECE8E0`);
- the mono 12 px line (line 75).

The hard-coded pill (line 69) binds to the status label (ANALYSIS §6.4 item 12).

**`design:PropertyCard`**, home variant at DESIGN lines 95-110 and list variant at lines 137-155. The list variant has the white sponsor pill top-right (line 141) and the discount colour `discC` (line 146).
- radius 12 px, hover border ink;
- a 150 px photo, with the status pill 10 px in from the top and left;
- body padding 18 px, name in Newsreader 21;
- the price in Geist Mono 500 22 with the 12 px discount line;
- the three-metric grid under a `#ECE8E0` rule;
- the 4 px raise bar, on the home variant only.

The eligibility chips and the "Proof due" chip are grey and amber `StatusPill`s under the metrics. They are not drawn in the prototype; they follow the TONE map.

**`design:invest`** (DESIGN lines 116-156):
- the header (lines 117-121): eyebrow "PUB-03", h1 in Newsreader 44/1.05, count line 15 px;
- the filter panel (lines 122-134): white, radius 12, padding 16, with the chip rows and 56 px group labels. `FilterChip` on is an ink background with white text; off is transparent with ink text and a `#D6D0C4` border, padding 7 px 14 px, Geist 500 13;
- the card grid (lines 136-155).

Mock data: `typeChips` and `sortChips` (lines 805-806). The four extra filter rows and the no-results block reuse these rows and `EmptyState`.

**`design:prop`** (DESIGN lines 158-289):
- **Head:**
  - breadcrumb (line 159);
  - pills (lines 162-166): status, type grey `#ECE8E0`/`#4A4D52`, "Wholesale · P1";
  - h1 in Newsreader 52/1.02, `-.02em` (line 167);
  - the "listed by" line (line 168);
  - the gallery `2fr 1fr`, rows `150px 150px`, gap 8, radius 10, with stripe placeholders (lines 169-173).
- **Aside** (lines 175-195): `position: sticky; top: 20px`, white, radius 14, padding 24, gap 18.
  - price and valuation in Geist Mono 500 30 (lines 177-178);
  - the discount line in Geist Mono 13 `discC` (line 180);
  - the metric row (lines 181-185);
  - the 8 px raise bar with the 2 px ink marker at `left: 75%` (line 188), replaced here by `soft_cap_bps`;
  - the lines at 189-190;
  - the CTA with 15 px padding (line 193), bound to `buyLabel`/`buyClick` (lines 837-838);
  - the 12 px note (line 194).
- **Tabs row** (lines 197-201): a 1 px bottom rule, the active tab with a 2 px ink underline at `margin-bottom: -1px`, `overflow-x: auto`. `TABS` is at line 807.
- **Tab bodies:**
  - Summary (lines 202-214);
  - Monthly distributions (lines 215-228: 200 px chart, green bars with radius `3px 3px 0 0`);
  - Capital returns (lines 229-242: `#D7E4DC` bars with a 2 px green top border);
  - Details (lines 243-249: rows `minmax(160px, 280px) 1fr`);
  - Documents (lines 250-260; `docs` at line 809);
  - Sponsor (lines 261-273: eyebrow line 264, name line 265, about line 266, badge line 267, facts lines 269-271; `SP` at lines 745-748);
  - Proof (lines 274-288: the `ProofBand` ink on cream at lines 276-279, with a Newsreader 22 title, a Geist Mono 12 `#B9B4AA` subline and the MATCHED pill; the `ProofRow` grid `24px 1fr auto` at lines 280-286; `proof` at line 810; `regUnits` at line 765).
- **State variants** (break, halt, paused, proof due, proof failed, pending, did-not-proceed, null valuation) are not drawn. They use `Banner`, `ProofBand` and `ProofRow` with the TONE amber `#F6ECD6`/`#7A4F00` and red `#F6E1DE`/`#A3312A`. They match A50's PUB-08 red break row "Break · opened 23 Sep 06:02" and its POR-02 ProofBand.

**`design:fees`** (DESIGN lines 291-304):
- the header: eyebrow "PUB-06", h1 in Newsreader 44, paragraph 15 px/1.55 (line 295);
- the white list, max-width 900 (line 297);
- the `FeeRow` grid `minmax(0,1.2fr) minmax(0,.6fr) minmax(0,1.4fr)`, gap 16, padding 16 px 20 px (lines 299-300): name in Geist 500, value in Geist Mono 500 15, description 13 px `#5E6166`.

Mock data: `fees` (line 811, six rows). The pending-change `Banner`, the effective-date caption and the second per-listing `FeeRow` list are compositions of the same components.

**PUB-05 standalone page (`design-ext:PUB-05`, A50 §3).** The design's Sponsor-tab cards come first, then the blocks A50 draws for the page: the planned-against-actual `DataTable` per trust (th in Geist Mono 500 11, uppercase `.05em`, `#5E6166`; the last column a `StatusPill`), skin in the game, the proof-of-property summary as `ProofRow`s, the incidents and complaints disclosure, the trust card grid and the outline "Follow sponsor" button with its signed-out and followed states. The page is compared with A50's frames, not composed freely; a component A50 did not draw is a finding raised on A50, not a local composition.

**Tokens (ANALYSIS §1).**
- Colours: cream `#F5F2EC`, ink `#17191C`, green `#1F5E4B` (hover `#174838`), brick `#9A4524` above valuation, `green.tint #E3EEE8`, `green.soft #D7E4DC`, `line #E3DED4`, `line.soft #ECE8E0`, `line.input #D6D0C4`, `text.2 #45484D`, `text.3 #5E6166`, `text.body #2E3135`, `muted.dark #B9B4AA`.
- Type: Newsreader for display, Geist for text, Geist Mono for figures, codes and captions.
- Radii: 14/12/10/8/999.
- Public main: `max-width 1240px; padding 36px 40px 64px; gap 32px`.

The BrickX-style coral look in the product-map document is not used anywhere.

**Seed invariants** this node must reproduce from the API (A06; ANALYSIS §6.4 item 20, §7):
- Kent Street Terrace: A$238.40 / A$246.10, "3.1% below valuation", 2.6% / 7.2% / 32%, 64% raised, closes 14 Oct 2026.
- Ann Street: "At valuation".
- Marion Street: "1.6% above valuation", in brick.
- Hawthorn Grove: "Debt free".
- Status mix in the base seed: three "Open offer" (Kent Street, Ann Street, Unley Road) and four "Fully funded". There is no "Coming soon" offering in the base seed. Marine Parade is in `compliance_review` and absent from every public page (ANALYSIS §6.4 item 19, A06 row 19, A36 item 6). Every Coming soon assertion (card, FeaturedCard fallback, coming-soon line, visitor and signed-in Notify me) runs in A06's `coming-soon` scenario (IR-7 (a)), where MPA01 becomes "Coming soon".
- Bay Road: five proof rows, the ProofBand "10,000 units = 10,000 tokens", and Property value "A$1,560,000" (A34's `property_value_cents` from A06's stored figure; A06 row 105 retires the design's "valuation × 10,000", which would read A$1,186,000 and is net assets at 25 percent LVR).
- Gympie Road: the break state (9,998 ≠ 10,000, opened 23 Sep 2026 06:02).
- The six fee rows.
- Harbour & Co: "admitted Mar 2026" and its four facts, with "Trusts on RealX" counting only publicly visible trusts (2 in the base seed: Kent Street and Bay Road; Marine Parade is hidden), per IR-6. The design's "3" counts the hidden listing and would leak its existence; this deviation is logged under HG-REVIEW.

## Requirements covered

**Product-map screens**
- **PUB-01 Home:** done when all of the following hold.
  - The hero, how-it-works strip, open-offers grid and audited-numbers strip render from A43, A21, A36 and A47, with no literal claim in the page, and the `GA-01` general-advice warning is shown.
  - The featured pill binds to the status label.
  - The strip is absent when no attested metric is published, and drops a row without an attestation.
  - The coming-soon line appears only when coming-soon offers exist (tested in A06's `coming-soon` scenario).
  - The page matches the prototype's `home` region token by token.
- **PUB-03 Investments:** done when all of the following hold.
  - All six product-map filters and six sorts, plus Featured, work from the URL through A36's query parameters with deterministic ties, and the same parameters on the API and the MCP tool return the same items, counts and relaxed filter.
  - The count line uses A36's whole-set `n`, `m` and `common_supply_units`, including when the result spans more than one page.
  - Every card carries its eligibility chips.
  - The no-results state shows the closest matches, names the relaxed filter and offers "Notify me".
  - The design's three-status gallery renders from the seed.
- **PUB-04 Property page:** done when all of the following hold.
  - All seven tabs render from A34, A47, A35, A33, A43 and A20.
  - The soft-cap marker sits at the listing's own soft cap.
  - The CTA kind comes from A36 and the named failed check from A19's `order_eligibility` verdict, on the server, for visitors and signed-in investors; the API gives a token the same answer plus G22.
  - A visitor on an open offer sees the list of checks an order needs.
  - Share and View register work.
  - Every state variant renders as specified.
- **PUB-05 Sponsor profile:** done when all of the following hold.
  - The tab and the standalone route share the two design cards.
  - The standalone route adds planned vs actual with the banded pill, skin in the game, the proof summary that lists every overdue item, and the incidents block.
  - Follow works for signed-in investors, and through the API and MCP tool for tokens.
- **PUB-06 Fees:** done when all of the following hold.
  - The schedule renders from A35's published `fee_schedule_platform` with its effective date and version.
  - A future fee notice shows as the banner with both dates.
  - Per-listing fee lines come from A36.
  - No fee rate exists in the page source.

**Journeys**
- **J01 Browse and decide.** This node owns five steps:
  - step 1: PUB-01, with the wholesale-only eyebrow from A21 and G17 copy;
  - step 3: PUB-03 filtered to commercial and NSW, sorted by net income, with chips on every card (G04, G06);
  - step 4: PUB-04 with proof and documents (G19, G20), where the invest panel names the checks needed: for the visitor, the "To invest in this listing you'll need" list from A36's `checks_needed[]`; for Alex Morgan, A19's verdict;
  - step 5: PUB-05 planned vs actual (G19);
  - step 6: PUB-06 (G17).

  Steps 2, 7, 8 and 9 are A55's and are linked from here.

  The failure line "the visitor is a foreign person: listings show which offers they may join" is covered by the "Residents only" and "FIRB check" chips and the Details tab's foreign-investors row.

  Done means the Playwright journey `J01` walks steps 1, 3, 4, 5 and 6 against the seed on the Docker stack and asserts every gate-bearing string named here, including the step 4 checks list and the `GA-01` general-advice warning on each of the five pages.

**Gates**
- **G04 Wholesale status valid:** done when three things hold. The Wholesale chip appears on every wholesale-only card. The aside note reads "Wholesale investors only" while A21 says so. A signed-in investor with an expired or missing classification sees `blocked`, naming G04 and the renewal screen, never a buy button.
- **G06 Residency and FIRB:** done when three things hold. The chips and the Details row derive from `residency_rule`. A non-resident on a residents-only listing sees `blocked`, naming G06. "Not permitted (established residential)" is never shown for a commercial listing.
- **G12 Disclosure acknowledged:** done when four things hold. The Documents tab shows the current IM version and date and marks superseded versions. A signed-in investor's open (and a token's open through the API) writes A35's open row. A visitor's open writes nothing. The page records no acknowledgement (INV-02 does).
- **G17 Marketing claims approved:** done when four things hold. Every visible string is on the allow-list, a published `site_copy` item, or formatter output. Yields carry their basis and date. No forecast bar is drawn. The audited strip shows only attested metrics. The `GA-01` general-advice warning renders on all five pages, and its absence fails closed. The lint fails on any new literal.
- **G19 Proof of property current:** done when A34's `due`, `overdue` and `failed` states render amber and red on the proof tab, the card chip and the sponsor profile, and an overdue item appears on PUB-05.
- **G20 Valuation current:** done when three things hold. Price, valuation, property value and "valued {date}" come from A34's current published valuation, and the discount from A36's field over it. A `null` valuation shows "Valuation pending" with no discount. A valuation past `valid_until` shows "Valuation review due".
- **G21 Licence perimeter:** done when three things hold. The eyebrow, phase pill and wholesale note come from A21. `car_active = false` or `G21_VISIBILITY_HIDDEN` fails closed to "Offers unavailable" with no prices. The `signed_in` and `verified_wholesale` modes hide prices as specified.

**Modules:** none owned. The pages consume the read contracts of A36 (M04), A34, A20 (M11), A35, A43, A21, A33 and A47. Done for those modules is unchanged by this node.

**State machines**
- **SM-03 Property and listing:** done when all twenty states render as the "Public states (SM-03 × SM-04)" table in PUB-04 says, with visibility and `cta_kind` taken only from A36's public read (A36 folds SM-03 in with the stated precedence). In short: the eight states before `spv_formed` are 404; `spv_formed` is public only with an `approved` or `scheduled` offering (Coming soon); `listed` follows the offering; `withdrawn` shows did-not-proceed; `funded` and `operating` show the paid-distributions chart; `distressed` and `legal_hold` show A34's public proof state in amber or red, never the internal cause; `exit_proposed`, `exit_approved`, `under_sale`, `sold` and `wound_down` show `unavailable` with the status label.
- **SM-04 Offering:** done when the fifteen states of A03's table after extension (b) map exactly to A36's label table and to the CTA kinds.
  - `approved` and `scheduled`: "Coming soon", notify.
  - `live` and `soft_cap_reached`: "Open offer", buy or sign up.
  - `suspended` (or an active hold): "Paused".
  - `reconfirming`: "Paused · investors re-confirming", with the "Terms changed" banner.
  - `closed` and `finalising`: "Closed" and "Closed · allotting", unavailable.
  - `minted`: "Fully funded", funded or sell.
  - `failed`, `refunding` and `refunded`: the did-not-proceed banner.
  - `draft`, `compliance_review` and `cancelled`: absent and 404.

  An upstream state change must appear on the page within one revalidation window.

**Founder requirement D04 and gate G22:** done when five things hold. Every fact, filter, sort, count and summary on the five pages is served by an API endpoint (and its A104 MCP tool) that an agent can call with the same parameters; none exists only in the web server. The watch, follow and document-open calls are token-reachable under A03 cells and audited with `on_behalf_of`. An agent's CTA comes from the same A19 `order_eligibility` evaluation with G22 at the head of the set. Web pages accept no API token. The parity map gives an API and MCP equivalent for every fact and action on these pages.

## Gauntlet loop for this node

**Specify.** The frozen spec is:
- this note;
- ANALYSIS §1-§3.1, §4, §6 and §7;
- the design regions above;
- A36's label table, public read schemas (with the IR-4 fields and query parameters) and its SM-03 folding precedence;
- A34's proof, valuation (with `property_value_cents`) and public property schemas;
- A20's public recon schema;
- A47's public distributions and transparency schemas;
- A33's public sponsor schema (with the IR-2 and IR-6 fields);
- A35's public document schema;
- A21's `perimeter/public` schema;
- A43's `site_copy` (including `GA-01`), announcement, fee-notice and watch contracts;
- A19's `order_eligibility` gate-set row and its investor-safe simulate response (IR-5), with A42's tipping-off rule;
- A03's matrix cells for the watch, follow and document-open calls (IR-8);
- A06's base seed and `coming-soon` scenario (IR-7);
- A05's component catalogue;
- A52's shell contract.

The node QA first confirms that the six graph edges named under "Graph change required" exist, then settles the eight interface requests. It checks every IR-4, IR-5 and IR-6 field and parameter by name against the OpenAPI document at its recorded hash. Each request is answered by the owning node, or that node is reopened; this node never substitutes a local parse, computation or filter. The node QA then records `specVersion`, the OpenAPI hash and the A05 catalogue hash in the evidence manifest. After the freeze, a change to a status label, a CTA kind, the filter list or the set of public states is a spec defect (reopen A36 or this note), not a fix.

**Challenge material (the node QA, before the build),** in `<workspace>\gauntlet\hidden\A54\`:
- (a) a reference implementation of the filters, sorts, counts and closest-match relaxation, run over 2,000 random offering sets loaded through A36's fixtures, against A36's `GET /v1/public/offerings` (including multi-page results, so whole-set counts and relaxation are checked), against the A104 MCP tool, and against the rendered PUB-03 page for the same parameters;
- (b) 5,000 random `(price_cents, valuation_cents)` pairs, including both sides of A36's "at valuation" band: A36's `discount_bps` and `discount_tone` must match the reference, and the rendered line and colour must match the API field exactly (the page computes nothing); plus Property value rendered from `property_value_cents` for trusts with and without debt;
- (c) the label and CTA matrix: every reachable SM-03 × SM-04 pair in the "Public states" table × {visitor, eligible investor, investor failing G01, G04, G06 or G07, investor under G02 sanctions or PEP review, investor under an AML restriction (G14)}, plus the API side for {token failing G22, token allowed}, asserting that the page CTA equals the API CTA for the same person apart from the G22 line;
- (d) the visibility matrix: each `listing.visibility` mode × `car_active`;
- (e) planted-literal patches ("guaranteed", "capital protected", a bare "8% yield");
- (f) the twelve reference crops at 1280 px (hero, featured card, how-it-works strip, open-offers card, filter panel, list card, open aside, tabs row, key facts, proof band, sponsor cards, fee rows) and a token-level comparison script;
- (g) a 375 px scroll-width and gutter check;
- (h) the seed-invariant table as an API-to-DOM contract.

**Build (Grok 4.7 xhigh, split by the planner).** Suggested shape: five parallel sub-agents on disjoint files, then one integration pass.
1. The `public-core` pure layer with unit tests: `query.ts` (URL parameters to A36 parameters, validation only), `labels.ts` (importing A36's `status_labels.ts`, not copying it), `proof.ts` (tone mapping), `trackRecord.ts` (A33's band to pill tone), the allow-list generator and lint, and the loaders with the revalidation policy. There is no filtering, sorting, discount or eligibility code.
2. PUB-01 and PUB-06.
3. PUB-03, with the URL state and the no-results block.
4. PUB-04, with the seven tabs, the session-keyed CTA segment and every state variant.
5. PUB-05 (the tab and the standalone page), plus the watch, follow and document-open actions.

The integration pass runs the Playwright `J01` journey and the fidelity screenshots on A23's Docker stack against the A06 seed (and the `coming-soon` scenario), builds the parity map, runs `npm run verify` and writes `app/evidence/A54.json`.

**Challenge (node QA, Opus 5.5 high).** The node QA runs its hidden material, then:
- drives the five pages from the seed in four passkey sessions (a visitor, Alex Morgan, a seeded investor with an expired certificate, a seeded non-resident) and makes the same calls through the API and MCP tools with two tokens (a mandate restricted to Bay Road, a full mandate). She diffs the CTA block and every gate string between page and API.
- sends a valid bearer token to each web route and checks that the page renders as for a visitor.
- flips A36 states through the local test transitions (`live → suspended`, `live → reconfirming → live`, `soft_cap_reached → closed → failed → refunding`) and checks each page within one revalidation window.
- plants a reconciliation break, an overdue proof row and a `null` valuation through the A20 and A34 fixtures.
- changes `listing.visibility` and `car_active`.
- publishes a future `fee_notice` and a new fee-schedule version.
- withdraws the published attested metrics, a `site_copy` item and `GA-01`.
- runs the literal lint on the delivered tree and on patched copies.
- compares the crops at 1280 px and every page at 375 px.
- requests hidden codes and unapproved slugs.
- reads the audit rows for the token calls.
- greps the client bundles for gate ids, caps, fee rates and status strings.
- writes `app/evidence/A54.qa.md`.

**Gate, fix loop, escalation.** A pass unlocks A61 and publishes the node QA's cases into the repo. At most three fix loops are allowed, each re-running the whole hidden set and the failing challenges. Escalate to the user in any of these cases:
- the same assumption fails a third time;
- a fix needs a field, state, component or seed value from another node (a spec defect: reopen that node, and this node waits);
- a design region and a product-map rule cannot both be met, for example a filter row the design has no room for at 375 px;
- a copy question needs the founder or counsel. In that case the page renders the conservative option and the question is logged under Human gates.

## Node-specific challenges (node QA)

- **Compliance: the buy button that lies.**
  - Setup: Alex Morgan's wholesale certificate is aged past expiry in the A32 fixture. Separately, a seeded non-resident opens Kent Street Terrace (residents only), and a token whose mandate covers only Bay Road calls the `order_eligibility` simulate for Kent Street through the API and through the MCP tool.
  - Expected: each gets the disabled button (or the API's `blocked`) and one line naming exactly the failing gate: G04 with the renewal link, G06 with "this listing is residents only", or G22 with "outside the token's allowed trusts". Never "Buy tokens", and never a second gate's text. A signed-in investor with no disclosure acknowledgement and no quantity still sees "Buy tokens" when everything else passes (G11, G12 and G16 are deferred to the order).
  - Then the certificate is renewed, and the next request must show "Buy tokens" without a rebuild.
  - Then an AML restriction (A42 fixture) is planted on a fourth investor, and a pending sanctions or PEP review (G02) on a fifth. Both pages must show only the generic review line. Neither may show "AML", "G14", "G02", "sanctions", "PEP", "restriction" or "case", in the HTML, in the `aria-live` region or in the API response to that person's own token.
  - Fails if: the page caches the CTA across sessions, evaluates or filters a gate itself, accepts a bearer token on a web route, or leaks an AML, sanctions or PEP reason.
- **Compliance: copy that was never approved.**
  - Setup: a `site_copy` item is withdrawn in A43, and the literal "Returns of 8% p.a." is planted in a scratch copy of the Summary tab.
  - Expected: the lint fails the verify run, naming the file and the string. The delivered page renders the withdrawn slot empty, not the old text.
  - Then a transparency response reaches the page with one total missing its `attestation`. The strip must drop that row.
  - Then an open offer's Monthly distributions tab is opened. It must show the empty state, not forecast bars.
  - Then `GA-01` is withdrawn. Every price and yield block on the five pages must fail closed to "Offers unavailable right now".
- **Money and truth: the soft-cap marker and the discount.**
  - Ann Street's `soft_cap_bps` is changed to 6,000 through an A36 fixture material change. While the re-confirmation window is open, the page must show "Paused · investors re-confirming" and the "Terms changed" banner. When the window closes (`reconfirming → live`), the marker must sit at 60 percent and the line must read "soft cap 60%". A hard-coded 75 percent fails.
  - Marion Street must show "1.6% above valuation" in brick on the card and in the aside. Ann Street must show "At valuation".
  - A fixture pair on each side of A36's band edge must render exactly A36's `discount_tone`; a page that recomputes the discount fails even when its result agrees.
  - Bay Road's Property value must be "A$1,560,000" from A34's `property_value_cents`. "A$1,186,000" (NAV per unit × 10,000, which is net assets) fails.
- **Security: reading what is hidden.**
  - Marine Parade Apartment (`compliance_review`) is requested by code, by offering id, through the sitemap and through the Next.js data route. An applied sponsor's slug and a suspended sponsor's slug are requested. The client bundle is grepped for `G0`, `bps`, `0.5%` and any status string. The watch endpoint is called with no session and a forged person id.
  - Expected: a 404 with no existence leak for the hidden listing and the applied sponsor; the neutral "Not currently listing" page with no Follow button for the suspended sponsor; an empty grep; and a 401 with no watch row created.
- **Design fidelity: the drift.**
  - At 1280 px the twelve crops must match exactly:
    - the hero h1 in Newsreader 400 64 px at `-.025em`;
    - the featured card's shadow and radius;
    - the list card's white sponsor pill top-right;
    - the aside sticky at `top: 20px`;
    - the active tab's 2 px ink underline at `margin-bottom: -1px`;
    - the ProofBand ink on cream with its `#B9B4AA` subline;
    - the fee grid `1.2fr .6fr 1.4fr`.
  - At 375 px every page must have `scrollWidth === 375`, a 16 px gutter, stacked property columns and a sticky bottom CTA.
  - Fails if: a page renders the product map's coral mocks, changes a font role, scrolls horizontally, or uses a colour outside the TONE map in a state variant.
- **Reconciliation and proof: the band that stays green.**
  - Setup: Gympie Road Retail's fixture run is a break (9,998 vs 10,000, opened 23 Sep 2026 06:02), and Bay Road's insurance row is set to `overdue`.
  - Expected:
    - Gympie's ProofBand is red, shows the opened date, and its tab is dotted.
    - Bay Road's row is amber, and its cards on PUB-01 and PUB-03 carry "Proof due".
    - Harbour & Co's profile lists "Bay Road Apartment · Insurance certificate · overdue".
  - Then A20's fixture closes the break, and the band must return to MATCHED within one revalidation window.
  - Fails if: the band is derived from anything but A20's last run, or a proof chip appears on the property page but not on the sponsor profile.
- **Agent misuse: a token that follows everything.**
  - A token with no watch cell (public reads only) calls watch and follow through the API. It must get the A12 scope refusal, and no row is written.
  - A token with the watch cell of IR-8 follows the same sponsor twice with one idempotency key. That must produce one subscription and one audit row, reading "agent {prefix} on behalf of Alex Morgan".
  - The same token's `order_eligibility` call for a listing outside its mandate must name G22. The same token sent as a bearer header to `/investments/{code}` must get the visitor page, not Alex's CTA.

## Pass criteria

- **Challenge agreement:** 100 percent of the node QA's cases in `<workspace>\gauntlet\hidden\A54\` pass on three consecutive runs. That includes:
  - zero disagreements with the reference filter, sort, count and relaxation implementation over 2,000 random sets, on the API, the MCP tool and the page;
  - zero disagreements on the 5,000 discount pairs between A36's field, the reference and the rendered line;
  - the full SM-03 × SM-04 × principal CTA matrix, with page and API agreeing;
  - the visibility matrix.
- **State coverage:**
  - A generated test iterates every SM-03 × SM-04 pair (20 × 15). For each pair the "Public states" table marks reachable, it renders PUB-04 from A36's fixture and asserts the label, CTA kind, aside variant, banners and 404 behaviour. For each pair A03's guards make unreachable, it asserts that A36's fixture refuses to produce it.
  - Each of the following has a screenshot at 1280 px and at 375 px: break, halt, paused, re-confirming, proof due, proof failed, pending, did-not-proceed, null valuation, no documents, no distributions, no announcements, offers unavailable, `signed_in` mode and `verified_wholesale` mode.
- **Gate enforcement:**
  - For each of G04, G06, G12, G17, G19, G20 and G21, plus G22 for tokens, a planted-failure test proves the page's response and a planted-pass test proves the normal rendering.
  - The AML (G14) and sanctions or PEP (G02) cases show only the generic line.
- **Design contract and fidelity:**
  - The seed reproduces every invariant in the design references from the API: numbers match, and strings match byte for byte.
  - The token-level comparison of the twelve crops reports zero differences in font family, weight, size, letter-spacing, colour, radius, padding and gap.
  - At 375 px, `scrollWidth === 375` on all five pages and every variant.
- **Journey:** Playwright `J01` steps 1, 3, 4, 5 and 6 pass on the Docker stack against the seed for a visitor and for Alex Morgan.
- **Copy and security:**
  - The literal lint is part of `npm run verify` and passes on the delivered tree, with the allow-list hash in the evidence.
  - The five routes' client bundles contain no gate id, cap, fee rate, status string or secret.
  - Every public read is made without credentials, and every write carries an idempotency key.
  - Token-made watch, follow and document-open calls produce audit rows with `on_behalf_of`.
  - Web routes ignore bearer tokens.
  - The parity map has no row whose only source is a web URL.
- **Performance and accessibility:**
  - Pages render on the server with a 60-second revalidation, with no client-side fetch for visitor content. The CTA segment is the only dynamic fragment.
  - Lighthouse accessibility is at least 95, with no WCAG AA contrast failure.
  - Keyboard-only navigation reaches every tab, chip, card and button.
- **Repo hygiene:**
  - `npm run verify` is green on Node 24.21.0.
  - There is no hex, font or radius literal in the owned path.
  - Only the owned paths changed.
  - `app/evidence/A54.json` exists, with routes, endpoint map, allow-list hash, screenshot set, parity map and interface-request answers. `A54.qa.md` exists too, and both match what is on disk.

## Human gates touched

- **None crossed.** The node runs against the A06 seed, sandbox adapters, A11's software authenticator and the local Docker stack. There is no real property data, money, keys or mainnet.
- **HG-LEGAL.** These counsel questions are logged but block nothing. Each answer is a setting or an approved copy item, so it changes data, not code.
  - May wholesale-only offers and their prices be shown to the public at all (`listing.visibility`, default `public` per the design)?
  - May "Vetted by RealX" and "audited" be used as written, and what must "audited capital raised" cite?
  - May per-sponsor incident and complaint counts be published on PUB-05?
  - What wording do the wholesale-only and no-cooling-off notes need on public pages?
  - How must interest on client money be treated and disclosed on PUB-06 (A16's question)?
  - May the site collect an anonymous "notify me" e-mail before sign-up (marketing consent)?
- **Founder defaults to confirm (`set_by_founder`):**
  - the featured-offer rule (first open offer in publication order, or a curated flag on A36);
  - the six-card cap on the home page's open offers;
  - the track-record bands behind "on track", "slightly under" and "under" (applied by A33, not by this page);
  - whether sponsor-side fees are published on PUB-06;
  - the design's "RealX platform fee 0.25% p.a." against the product map's per-investor monthly platform fee (design analysis §6.2). This page renders either one only as an approved fee-schedule row.
  - that an open offer's distributions tab shows an empty state rather than the design's mock bars.
- **HG-REVIEW (user).** PUB-05's standalone page and the drawn-screen supplements of PUB-01, PUB-03, PUB-04 and PUB-06 are drawn by A50 and reviewed there. The remaining state variants that neither the prototype nor A50 draws are compositions of existing components. The user sees them when walking the A61 preview. If the founder wants them drawn first, they are added to A50 by reopening it. Two seeded values deliberately differ from the prototype: Bay Road's Property value reads "A$1,560,000" (property value, per A06's valuation rule) rather than the design's "A$1,186,000" (net assets), and Harbour & Co's "Trusts on RealX" reads 2 rather than 3 (the hidden Marine Parade is not counted).

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
