---
title: "A87 - Design fidelity and accessibility pass"
created: 2026-09-24
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am8]
node: A87
milestone: AM8
lane: "Quality"
oracle: no
human_gate: blocks-own-gate
---

# A87 - Design fidelity and accessibility pass

**The last look at the whole P1 interface before regression. This node checks every screen that is drawn in the prototype and every screen in the two design extensions. Each one is opened in the built app and in its design reference, in every listed state, at 1280 px and at 375 px, and the two are compared by design token (font, colour, radius, spacing), not by pixel. The same pass measures WCAG AA on the rendered pages: contrast on the real tokens, keyboard focus order, aria-live on counters and toasts, no horizontal scroll at 375 px and 320 px with 16 px gutters, text resize and spacing, hover content and focus not hidden. It also checks that jargon is spelled out on first use, and it proves that prototype bugs 2, 14, 16, 17 and 19 are fixed in the build. Each finding becomes a fix item on the UI node that owns the screen, and the founder signs off the visual QA before A88 starts.**

**Build status:** not started. **Milestone:** AM8 (Journeys, hardening and launch). **Lane:** Quality. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 10-16 agent-hours (build, challenge and verify loops). This covers the two drivers, the harness, the accessibility and jargon suites, the five bug tests, the D04 checks, the gallery, one full run, routing the findings and one re-run after the owning nodes fix them. It stays at this size only because region maps and state recipes are generated from the charter's, A50's and A51's `screens.json` and A05's component tags, with hand-written anchors only where generation cannot find a unique one; if more than a quarter of the regions need hand-written anchors, the lead records the overrun in the evidence. Fixes made by the owning nodes count against their own estimates.

**Depends on:** [[A79 - End-to-end journeys- investor money path]], [[A80 - End-to-end journeys- sponsor and operations]], [[A81 - End-to-end journeys- recovery, AML and planted breaks]]
**Unlocks:** [[A88 - P1 full regression- 18 P1 journeys, three clean runs, every gate exercised]]

**Oracle rule:** this is not an oracle node (`oracle: no`). No cent, unit, holding or authority moves here. The node reads screens and writes findings, and every rule a screen displays is enforced and tested server-side by the node that owns it. Before the build starts, the node QA still writes its Challenge material to `<workspace>\gauntlet\hidden\A87\` ([[App Gauntlet]] §2). That material is:
- its own token extractor for the rendered DOM, written independently of the builder's comparator;
- its own contrast script;
- a scratch copy of the built web app with twenty planted drifts, used for the mutation challenges below (listed by id and kept hidden from the builder);
- its per-screen checklist of the design copy that carries compliance meaning;
- the reference crops of twelve prototype regions;
- the step-by-step scripts for the five prototype bugs.

The builder never reads that folder.

What this node is for, in one sentence: the design says how RealX looks, N11 and N18 say it must be usable and readable by everyone, and this node proves both on the built screens before anything is called finished. It fixes no screen itself. Every defect goes back as a fix item to the node that owns that screen's code (the owner is listed in the charter's `screens.json`). This node then re-runs until the owners close their items. This plan is not legal advice.

## Build brief

**1. Scope: which screens, which states, which widths**

- **Screen list.** The screen list is read from the charter's `<workspace>\app\docs\charter\screens.json` (96 rows from [[A00 - Charter and acceptance baseline]]: 39 `drawn`, 7 `partial`, 50 `undrawn`), never typed by hand.
  - **In scope:** every row whose owning node is in AM4 to AM8 (the P1 screens).
  - **Also in scope (extension frames that are not charter rows):** every frame and state in A50's and A51's `screens.json` that is not a charter row, each given an owning node in the harness's own scope table (`e2e\fidelity\states\extension-scope.json`, generated, never typed):
    - the four `SGN-` sign-in and sign-out shell screens (owner A52 for investor and sponsor, A53 for the operator console, A105 for partners);
    - the four partner frames A50 draws (Trustee and RE, Auditor, Regulator observer, Licensee), each with its placeholder page, the partner light sidebar and the "Read-only · time-boxed" pill, plus the partner home pages A105 builds inside them (owner A105; the frame chrome itself owner A52);
    - ADM-36 to ADM-40 from A51's `screens.json` and A51's other shell frames (the seven-group sidebar, the hardware-key confirmation modal; owner A53 or the console node A51 names).
    These cells get the same token comparison, axe, contrast, focus, reflow and jargon checks as charter rows, and they are counted in the Coverage pass criterion.
  - **Deferred, not dropped:** rows owned by a P2 or P3 node are written to `coverage.json` as `deferred` with that node's id. These are ONB-07 (A90); INV-05 to INV-07 and ADM-21 (A92); GOV-01, GOV-02, SPN-13 and ADM-23 (A93); and INV-04 (A95). The harness is left runnable on those screens with the same thresholds (item 8), but this node does not claim that those nodes will run it: none of them depends on A87 in the graph.
  - **Fail closed:** a `deferred` row whose owner sits in AM8 or earlier fails the run.
- **Design reference for each screen.** Drawn and partial rows are compared with the matching region of `<workspace>\design\RealX App.dc.html`, found through the prototype screen key in `screens.json`. Undrawn rows are compared with `design-ext:<code>`: `<workspace>\app\design-ext\investor-sponsor\RealX App Ext A.dc.html` (A50) or `<workspace>\app\design-ext\admin\RealX Admin Ext.dc.html` (A51). Partial rows use both. The drawn part is compared against the prototype, and the missing part is compared against the component pattern that design analysis §5 names. A screen with no reference at all is itself a finding, owned by the design-extension node.
- **States: "every listed state" is the union of four lists, generated into `e2e\fidelity\states\` and each resolved to a state recipe.**
  - (a) **The prototype's own interactive states for the screen**:
    - property tabs ×7;
    - Type and Sort filters on PUB-03;
    - onboarding steps 1-8;
    - buy steps 0, 1 and 2, with acknowledgement off and on;
    - approvals: item selected, then approved, rejected and emptied to "Inbox clear.";
    - KYC case open and decided;
    - token operations at one, two and three signatures;
    - distribution chain before and after "Approve as sponsor owner";
    - listing wizard steps 1-4.
  - (b) **The product-map `states` entries for the screen**, from the charter (for example POR-01 "No holdings: shows open offers").
  - (c) **The states listed for the screen in A50's or A51's `screens.json`**: default, empty, blocked, not-eligible, refused, preview, sandbox, pending owner approval and agent-voted, whichever apply.
  - (d) **The shell states** from A52 and A53: loading, error, session-expired and the collapsed-sidebar drawer.
  - **A state the built app cannot reach** through its real routes and seeded scenarios is a finding against the owning node, never a skip.
- **Widths.**
  - Public, investor, sponsor and partner surfaces: 1280 × 800 and 375 × 812 for every check, plus 320 × 640 for the accessibility checks only (reflow, gutters, text resize and text spacing in item 4; no token comparison at 320).
  - Operator console: 1280 × 800, 1024 × 768 (drawer closed and open) and 375 × 812, plus 320 × 640 for reflow only (item 4). At 375 the console only has to meet the reflow and gutter rules; its layout is desktop-first by the design analysis §6.3 decision.
  - At 375, drawn screens have no phone design to compare with, so only colour, font and radius tokens are compared against the desktop prototype region. Layout is checked against the phone rules frozen in A05 (header nav collapsed to a menu, two-column sections stacked, card grids single-column, `DataTable` scrolling inside its card). Extension screens are compared with A50's and A51's own 375 captures where they exist.

**2. The harness (`e2e\fidelity\`, Playwright, run as its own workspace)**

- **Workspace prerequisite.** A01 declares the workspace glob `e2e`, which makes only `e2e` itself a workspace, so `e2e/fidelity` is not one and `npm -w e2e/fidelity` would be refused. This node cannot edit the root `package.json`. At Specify the orchestrator opens an A01 follow-up loop to change the glob from `e2e` to `e2e/*` (A83's `e2e/security` workspace needs the same change). The build does not start until that loop has passed; this node never adds its scripts to another node's `package.json` as a workaround.
- **Commands.** `npm -w e2e/fidelity run fidelity` runs everything. `-- --screen PUB-04`, `-- --node A57` and `-- --state POR-01:empty` re-run a subset, which is how an owning node checks its fix during its own fix loop. `run gallery` rebuilds the founder's review page. The full run is not wired into `npm run verify`, because it is too slow for every commit. A 12-region smoke subset is exposed as `run fidelity:smoke` for A88 to include, and A88 runs the full pass in its regression.
- **Built-app driver.**
  - The driver runs against the compose stack that A79-A81 use. Data comes from A06's seed, which is the design's own dataset, and A09's injectable clock is fixed to A06's exported `AS_OF` constant (imported from the seed manifest, never typed), so names, numbers, dates and relative times ("4h", countdowns, "x min ago") match what the seed asserts and what the prototype shows.
  - **Test isolation.** Tests are split into two Playwright projects so no test can change what another captures:
    - `read-only` (four workers): every capture, comparison and accessibility check that changes no data. It runs first, on a freshly seeded baseline.
    - `mutating` (`workers: 1`, `fullyParallel: false`, runs after `read-only`): every test that changes data or platform state. These are the five bug tests, the live-region checks that approve APR-2291, add a signature or change a quantity, the approve and reject flows on ADM-30 and the approvals inbox down to "Inbox clear.", the September distribution approval and the SPN-10 attestation, the ACC-05 and SPN-16 create, revoke and copy-once captures, and every kill-switch tier up to the platform tier. Each mutating test starts from a freshly re-seeded database (A06's seed script re-run against the stack's database with the baseline manifest, checked by `seed:verify`) and leaves nothing behind for the next; where re-seeding takes too long, the mutating project runs on a second compose project of the same profile instead. Which option was used is recorded in the evidence.
    - A test that changes data but sits in the `read-only` project is a harness defect: the harness records the audit head hash and the approval, token and kill-switch tables before and after the `read-only` project and fails the run if any of them changed.
  - Personas sign in with a real passkey or hardware-key ceremony through A11's sandbox authenticator. The personas are Alex Morgan, Mia Chen, Jordan Blake, the A51 console personas and one partner of each type from A105.
  - Each state is reached through real routes plus A06 simulator scenarios. A state is never reached by patching the DOM or by stubbing a network response. There is no role switcher (D20): each persona's states are reached by that persona signing in through its own front door in its own browser context.
  - Before capture, the driver waits for `document.fonts.ready` and for network idle, and turns off caret blink.
- **Prototype driver.**
  - The design folder is opened read-only through a static harness page that serves `<workspace>\design\` and loads React from A01's installed dependencies. This follows A51's harness pattern, and A61's side-by-side runner in `e2e\preview\` is imported read-only if it already offers what is needed.
  - Recipes are click scripts through the prototype's own "view as" bar and buttons, because the prototype has no URLs. A direct `setState` patch is used only if the runtime exposes the component instance, and that choice is recorded in the evidence.
  - Before and after every run, the SHA-256 of `RealX App.dc.html` and `support.js` is checked against A00's `design-baseline.json` (`8ef1e73c…` and `8fe7df74…`). A mismatch stops the run.
  - The A50 and A51 files are opened through their own harnesses, and their hashes are recorded.
- **Region maps (`e2e\fidelity\regions\<code>.ts`).** Each region pairs one prototype locator with one built-app locator.
  - Both locators are anchored by ARIA role and visible text from the shared seed ("Kent Street Terrace", "APR-2291", "Needs you"), or by a stable `data-testid` where text is not unique.
  - Each region is tagged with its A05 component name (`StatusPill`, `KpiTile`, `GateRow`, `DataTable`, `DistributionChain` and so on) so the right property set is compared.
  - An anchor that matches zero or several elements fails closed as a harness error. It is never resolved to the first match.
  - A region the builder cannot anchor without a test id becomes a Minor finding on the owning node ("add `data-testid` X").

**3. The token comparison (layout tokens, not pixels)**

- **What is compared.** For each region and each of its descendants of the same component, the comparator reads computed styles:
  - `font-family` (the first family), `font-size`, `font-weight`, `line-height`, `letter-spacing` and `text-transform`;
  - `color`, `background-color`, `border-*-color`, `border-*-width` and `border-radius`;
  - `padding`, `gap` and `grid-template-columns` (as a pattern, for example `minmax(0,1.4fr) minmax(280px,1fr)`);
  - `box-shadow`;
  - `position: sticky` and `top`.
- **Mapping to tokens.** Each value is mapped to a token name through A05's `packages\ui\tokens.json`: colours by exact hex, radii by the twelve radius values, font shorthands by the type-scale entry and spacing by `space.ts`. The comparison is then made on token names.
  - Colours and radii must match exactly.
  - `font-size` and `line-height` may differ by ±1 px.
  - Padding and gap may differ by ±2 px at 1280.
  - Families and weights must match exactly.
- **Checking that fonts really loaded.** The comparator calls `document.fonts.check()` and checks that the `FontFace` status is `loaded` for Newsreader 400 and 500, Geist 400 and 500 and Geist Mono 400 and 500. The CSS saying "Newsreader" is not enough: a silent fallback to `serif` is a Major finding. The fonts must be served from the app's own origin (A05). A request to `fonts.googleapis.com` from a built page is a Blocker.
- **Interaction states.** Hover (`style-hover` in the prototype: ink border on cards and rows, `#174838` on the primary button) and focus are forced with Playwright's `hover()` and `focus()` and compared in the same way. The prototype has no focus style, so focus is compared against A05's green focus outline instead.
- **Untokenised-colour sweep.** On every captured page, every element's computed `color`, `background-color` and `border-color` must be in the `tokens.json` set, or be `transparent` or `currentColor`. Pixels inside `<img>` and `<video>` are excluded, and so are `box-shadow` colours (shadows are compared as whole values against A05's `shadow.ts` in the token comparison above). A semi-transparent value is accepted only when `tokens.json` lists that exact value; the approved example is A51's modal scrim `rgba(23,25,28,.25)`. If A05's `tokens.json` lists that value only inside `shadow.ts` and not as a colour token, that is reopened upstream on A05 at Specify, never allowed by a local exception here. A near-miss such as `#1F5E4C`, or any semi-transparent value that `tokens.json` does not list, is a Major finding that names the element, the value and the nearest token, and is never counted as a pass.
- **Design copy that carries compliance meaning.** Token parity alone cannot catch a missing sentence. `docs\fidelity\compliance-copy.json` lists, for each screen and state, the visible text the design or the product map uses to carry a rule, and each entry must be present and visible:
  - the footer's CAR/AFSL disclosure (dc line 516) on every public page;
  - "Money waits in escrow; refunded automatically if the soft cap isn't reached" and "Wholesale investors only" on PUB-04;
  - the gate ids and names on the INV-02 GateRows, plus the product map's "no cooling-off" line and the documents-opened precondition;
  - the G12 acknowledgement text;
  - "Withdrawals over A$50,000 need a second RealX approval" on POR-05 (configured threshold);
  - "The register is the legal record" on ADM-17;
  - the maker-checker note on ADM-30;
  - "Agents prepare. A person approves." wherever A51 draws it.

  An entry that is missing is a Blocker. A change of wording is allowed only where the product map's text wins over the design (design analysis §6), and it is recorded as such.
- **Pixels are for people, not for the pass.** A pixel screenshot of both sides is still captured for each screen, state and width and placed side by side in the founder's gallery. A perceptual diff percentage is shown for information only, and no pass or fail depends on it.

**4. Accessibility, measured on the rendered pages (N11)**

- **axe-core.** `@axe-core/playwright` (the axe-core version A05 already pins) runs on every screen, state and width. Critical and serious findings are Blockers, and moderate findings are Minor.
- **Rendered contrast.** Every visible text node is measured against its effective background. Text must reach at least 4.5:1, and large text (at least 24 px, or at least 18.66 px at weight 600 or more) at least 3:1. Focus indicators and required control boundaries of enabled controls must reach 3:1 (WCAG 1.4.11).
  - **How the effective colours are found.** The text colour and every background between the text and the page root are composited in paint order: each semi-transparent colour is alpha-blended over what lies beneath it, and each ancestor's `opacity` is multiplied into the colours it paints (so text inside an element at `opacity: .5` is measured at its blended colour). Overlays and scrims (for example A51's `rgba(23,25,28,.25)` scrim under the hardware-key modal) are blended into the background of whatever they cover. Where the text sits over an image, a gradient (such as A05's striped photo placeholder) or any background the walk cannot resolve to one colour, the harness samples the screenshot pixels behind the text box, with the text itself hidden for that sample, and uses the worst-case ratio.
  - **Inactive controls are exempt (WCAG 1.4.3).** Text and boundaries of a disabled or inactive control (native `disabled`, or `aria-disabled="true"`), such as the design's disabled buttons at opacity .4 and .45 (dc lines 356 and 446) and the disabled main actions in A50's and A51's blocked and not-eligible states, are measured and recorded in the report for information only. They are never a Blocker. The exemption is decided from the accessibility tree, not from the colour: an enabled control that shows the same faded colours is still measured and fails.
  - **The disabled reason is not exempt.** The sub-line or banner that tells the person why the action is disabled is ordinary text and must meet 4.5:1. The token table in A05 is re-checked within ±0.01 on the pairs it lists. Examples: ink on cream 15.76, green on cream 6.81, dark.muted on dark.1 4.85 (the thinnest margin). Any text pair that appears on a rendered page but is missing from A05's table is added to the report with its measured value.
  - The design's own border contrast (A05's DF-A05-01: input border 1.37:1, card border 1.34:1) is not re-raised as a defect. The check confirms that A05's mitigation (green focus outline, input background step, always-visible label) is present on every rendered input, and the founder's recorded decision on DF-A05-01 is carried into this node's evidence.
- **Keyboard focus order.** On each screen and state the harness tabs through the page and records the sequence.
  - Every interactive element must be reachable, and nothing unreachable by keyboard may be clickable.
  - The order must follow reading order (top to bottom, then left to right within a row, with an 8 px tolerance). The sticky property aside and the onboarding stepper are allowed their DOM position.
  - A skip link must lead to the main content on the public and investor shells.
  - Every focused element must show a visible focus ring: computed `outline` or `box-shadow` is not `none`, and it reaches 3:1 against its background.
  - Modals must trap focus and close on Escape, and focus must return to the control that opened them. This covers the hardware-key confirmation modal (A53), the passkey step-up sheet, the ACC-05 and SPN-16 create-token sheet and the kill-switch confirmation.
  - The prototype's clickable `span` and `div` elements must be real links or buttons in the build. Examples are the footer "Fees" and "Complaints" (dc line 517), whole-row `DocumentRow` and `TaskRow`, and the stepper labels. A clickable element with no role, or with no keyboard handler, is a Blocker.
- **aria-live on counters and toasts.**
  - The live regions checked are:
    - toasts, which use `role="status"` and `aria-live="polite"`;
    - the operator nav badges (A53);
    - the "Approvals waiting" KPI (A66);
    - the raise progress on PUB-04 and SPN-07;
    - the INV-01 order total as the quantity changes;
    - the INV-03 order timeline;
    - the SignatureMeter "n of 3" (A72);
    - the ADM-35 kill-switch state and anomaly count (A77);
    - the agent activity list on ACC-05 and SPN-16 when it refreshes.
  - The harness changes the underlying value through a real action (approve APR-2291, add a signature, change quantity) and checks that the live region's text changes exactly once for that change. Polling that rewrites the same value must not re-announce it.
  - Sev1 alerts on ADM-01 and ADM-02 may use `assertive`, and nothing else may.
  - A toast that reports an outcome must also leave that outcome visible on the page after the toast has gone. For example, "Order placed · funds in escrow" leaves the INV-03 timeline showing the order. This is because a 2.6-second toast alone does not satisfy WCAG 2.2.1 for users who miss it.
- **Reflow and gutters at 375 px and 320 px.** `document.documentElement.scrollWidth` must be at most the viewport width on every captured state, including open modals, toasts, long error banners and expanded tables. This runs at 375 × 812 and, for WCAG 1.4.10 (defined at 320 CSS px), at 320 × 640 on the public, investor, sponsor and partner surfaces (the operator console is desktop-first; it is checked at 320 for reflow only, and a failure there is a `DF-A87` finding for the founder, not a Blocker, because design analysis §6.3 decided the console's width). The left and right edges of the main content box must be at least 16 px from the viewport edge at both widths. A `DataTable` may scroll only inside its own card. The harness also injects the longest seeded strings: the SMSF entity name, the PayID `alex.morgan.841@pay.realx.au`, a full wallet address, a `TOK-` id and a long property name. Wrapping or truncation of these must not create page scroll. Wallet addresses must use the one truncation rule of first 6 plus last 4 (§6.4 item 3), checked here in passing.
- **Other WCAG AA checks.**
  - Status must not rely on colour alone (1.4.1): every `StatusPill` has text. The agent "on behalf of" marker has text or an accessible name, not only a tint.
  - The heading order must have one `h1` per page.
  - The page must have landmarks (`header`, `nav`, `main`, `footer`).
  - Every form field must have a programmatic label, and error text must be linked with `aria-describedby`.
  - Masked fields (TFN, account number) must not expose the raw value in the accessible name.
  - Images must have `alt`, and decorative photo placeholders `alt=""`.
  - Page language must be `en-AU`.
  - **Target size (WCAG 2.2 2.5.8)** at 375: every control is at least 24 × 24 px, with WCAG's own exceptions applied: a smaller target passes when a 24 px circle centred on it does not overlap another target or its circle (spacing), and links inside a sentence or a block of text (inline) are exempt, such as the footer's 12 px links (dc line 517) where they sit in running text. The harness records which exception each passing small target used.
  - **Resize text (1.4.4).** With the root font size doubled (200 percent text zoom, not page zoom) at 1280 and 375, no text is clipped or overlapped (every text box's `scrollHeight` and `scrollWidth` fit its container unless the container scrolls by design, such as a `DataTable` card), and no content or control disappears.
  - **Text spacing (1.4.12).** With WCAG's test styles injected (line height 1.5, paragraph spacing 2 em, letter spacing 0.12 em, word spacing 0.16 em) at 1280 and 320, no text is clipped or overlapped and no content is lost, checked the same way.
  - **Content on hover or focus (1.4.13).** Every tooltip and hover card, including A66's and A53's `AgentMarker` hover that shows the token id and mandate, must: appear on keyboard focus as well as hover; be dismissable with Escape without moving focus or pointer; stay visible while the pointer moves onto it; and stay until dismissed, focus or hover leaves, or it is no longer valid. The harness drives each with `hover()`, a pointer move onto the tooltip, `focus()` and Escape.
  - **Focus not obscured (2.4.11).** For every element reached in the keyboard walk, the focused element's bounding box must not be entirely covered by a sticky or fixed element (the sticky property aside, sticky headers, the console top bar, toasts), checked with `elementsFromPoint` over the element's box. Partly covered passes (that is AAA 2.4.12); fully covered is a Blocker.

**5. Plain English: jargon expanded on first use (N18)**

- **Glossary.** `docs\fidelity\glossary.json` holds N18's list with its long forms: AFSL, MIS, PDS, DVS, AML/CTF, PEP, KYB, DDO, FIRB, MIT, AMMA, AFCA, NPP, EDD, CAR, RE, TMD, IM, SMR, TTR, IFTI, LVR, TCP, DAP. The harness adds every other upper-case abbreviation of two to six letters it finds on rendered pages (for example SMSF, TFN, FATCA, CRS, KYC, NAV, API, MCP) for the builder to classify as "expand" or "common" (AUD, PDF). The founder approves the "common" list under HG-REVIEW.
- **Rule.** On each rendered page, the first visible occurrence of each "expand" term, in DOM reading order, must have its long form in the same sentence, table cell or list item, or immediately before it.
  - These do **not** count as an expansion: a `title` attribute alone, an `abbr` tooltip alone, and an expansion that only appears lower on the page.
  - Exempt: identifiers in Geist Mono (gate ids such as `G02`, screen codes, approval ids, token prefixes), and table headers whose column is expanded in a caption above.
  - The check covers visible text, toasts, empty states, modal text, `aria-label` values and the A99 reference docs page `/docs`, which is built in the design language.
- **Tipping-off guard (N08).** The jargon check must never lead to AML wording being added where it could tip someone off about their own status. The guard is scoped to places that show a viewer's own account or a named person's status, on investor-facing, partner-facing and sponsor-facing surfaces and in the API responses and MCP results tied to the viewer's account: status banners, pills, notices (ACC-03), activity rows (POR-04), error and refusal messages, disabled-with-reason sub-lines, withdrawal and order statuses, token states on ACC-05 and SPN-16, and sponsor views of a named investor (register rows, allocation lists).
  - In those places, the tokens "AML", "AML/CTF", "anti-money laundering", "SMR", "suspicious", "EDD" or "enhanced due diligence" are a Blocker routed to the owning node, whether or not the term is expanded.
  - The allowed wording in those places is the generic set that N08 and the owning nodes fix, listed per surface in `compliance-copy.json` under `genericReviewStrings`: for example A59's "Under review" (withdrawal rows and the Funds available tile) and A60's "Paused while we review your account". Any other review wording on those surfaces is a Major finding for the founder to rule on, not an automatic pass.
  - Generic legal references are not tipping-off and are allowed: public help, privacy and collection notices and the onboarding identity-check explanation may name the Anti-Money Laundering and Counter-Terrorism Financing Act 2006 as the law that requires collection, and N18's rule that "AML/CTF" is spelled out on first use applies there as usual. Where exactly the line falls between a generic notice and a status message is logged as a counsel question (HG-LEGAL, through the founder), and the harness's classification of each page is recorded in `compliance-copy.json` so counsel's answer changes data only.

**6. Prototype bugs 2, 14, 16, 17 and 19 confirmed fixed (named tests in `e2e\fidelity\bugs\`)**

- **Bug 2 (SPN-10 attestation reads the distribution flag; owner A64).**
  - As Mia Chen, approve the September distribution on SPN-09, then open SPN-10. The "Operator attestation" row must still read "Sign now" (amber).
  - Sign the attestation with a passkey. The row reads "Signed", and the SPN-09 chain is unchanged.
  - Reload both pages and check that each state persists on its own.
- **Bug 14 (recon row mislabels applications as register units; owner A73, and A55 for PUB-08).**
  - ADM-17 shows, for each trust, chain supply / trustee register / bank or escrow / Δ.
  - For Kent Street Terrace (offer open), the register column shows 0 units and the 6,400 applications appear in the escrow column. No cell reads "Register 6,400 (escrow)".
  - The Δ is computed from the three sources and the row is "Matched" only when Δ = 0.
  - PUB-08's public view uses the same labels.
- **Bug 16 (one-click login; owners A52, A53, A105).**
  - No surface turns a visitor into a signed-in user without a passkey or hardware-key ceremony. "Log in" opens the sign-in screen. The investor, sponsor, partner and operator sign-in screens exist and match `design-ext:sign-in`.
  - No build of `apps/web` or `apps/console` contains `DevRoleSwitcher`, a role-switcher component or a persona name, and the release bundle has no `/__test/` path. The check searches the built assets and loads `?dev=1` and `?as=` queries to confirm that neither changes who is signed in (D20).
- **Owners of bug-test checks.** Each assertion inside a bug test is routed to the owning node of the screen it looks at, read from `screens.json` (charter, A50 or A51), never from a typed list; the owners named below are what that lookup is expected to return and are checked against it at Specify.
- **Bug 17 (ONB-02 choice does not branch; owner A56 for the onboarding branch, A59 for ACC-01, A57 for the INV-02 gate descriptions, A64 for the sponsor register's Type column on SPN-08).**
  - Choosing "SMSF, company or trust" leads to ONB-04 after ONB-03, and the continue button stays disabled until every trustee, director and 25%+ owner is verified.
  - Choosing "Joint" requires the second person to be verified, and the either-or-both signing choice is stored.
  - Choosing "Individual" never shows ONB-04.
  - The chosen type is used by later screens: it appears in ACC-01, in the INV-02 gate descriptions and in the sponsor register's Type column.
- **Bug 19 (Marine Parade public before approval; owners A54 for PUB-01, PUB-03 and the sitemap, A62 for SPN-02, A71 for ADM-07, A66 for ADM-01).**
  - While APR-2290 is pending, Marine Parade Apartment does not appear on PUB-01, on PUB-03 (under any filter or sort) or in the sitemap, and its direct URL returns the not-found page for a visitor and for Alex Morgan.
  - At the same time it appears on SPN-02 as "In RealX review" and on ADM-07 and ADM-01 as waiting for review.
  - **The same check through the API and MCP, with a live token.** An agent must see exactly what the person sees, on the same data. A sandbox token cannot prove that: A12 refuses an `rx_sb_` token on every live `/v1` route with `TOKEN_WRONG_ENVIRONMENT`, and on `/sbx/v1` it reads A99's separate per-user sandbox dataset, which does not hold the pending APR-2290 state. So:
    - At the start of the test, Alex Morgan creates a live-environment `rx_inv_` token named `a87-bug19-read` through ACC-05 with a passkey step-up against A11's sandbox authenticator, with the read scope that covers listings only, on the local compose stack (live environment of the local stack, not production).
    - The test calls the live `GET /v1/listings` (every page, following the pagination cursor) and the A104 MCP tool generated for that operation (looked up in A104's `catalogue.json` by operation id, not by a typed name), against the same seeded database the UI is showing at that moment.
    - It compares three sets of listing ids: the ones PUB-03 renders for Alex (all filters cleared), the ones REST returns, and the ones MCP returns. The three sets must be equal and must not contain Marine Parade.
    - The token is revoked through ACC-05 at the end of the test, pass or fail (in a `finally` step), and A12's table must then show it `revoked`.
    - The token's secret is never written to a trace, HAR, log or artefact (tracing and HAR are off for this test, and the secret scan in item 7 covers it). The evidence records the token's id and prefix, `environment = live`, its revocation time, and the three id sets.
    - Optional second check: with a sandbox token on `/sbx/v1`, A99's sandbox dataset also hides a listing that is pending review there. It is reported for information and is not a pass criterion.
  - After a checker approves APR-2290 with a hardware key, it appears with the status the product map gives it, and the "Coming soon" pill is used only from that point.
- The other §6.4 items are owned by earlier nodes (3 and 12 by A05; 9 by A58; 10 by A57 and A92; 11 by A54; 13 by A73). The harness reports any it sees as findings, but only the five above are pass criteria here.

**7. D04: agent surfaces, agent markers, and parity of what is shown**

- **No new actions.** This node adds no user action, so it adds no REST route, MCP tool, SDK method or webhook. Proving that every UI action has an API and MCP equivalent is A100's job. What this node does for D04 is below.
- **Agent screens in every state.** ACC-05, SPN-16 (including the valuer and property-manager variants) and ADM-35 are checked in every state A50 and A51 draw:
  - create with a passkey step-up;
  - the copy-once secret card, shown once and masked on return;
  - a scope picker limited to the viewer's rights, with ceiling and non-delegable rows shown as disabled with a reason;
  - mandate limits and expiry in plain English;
  - SPN-16 pending owner approval;
  - activity that includes G22 refusals with their reason;
  - revoke;
  - the sandbox pill;
  - notice settings;
  - kill switches at the token, user, tenant and platform tiers (platform as `SGL+R` with the 2-hour countdown).
- **Status vocabulary.** The tone map A12 fixes (`active` green, `pending` amber, `suspended` amber, `revoked` red, `expired` grey, `sandbox` brick) is checked on the rendered pills. Any status string not in A05's `toneFor` table or A12's vocabulary is a Major finding.
- **The agent marker.** The "agent <token> on behalf of <person>" marker from A53's shared cell is checked everywhere actions are listed: POR-04 activity, ACC-03 notices, ADM-30, ADM-31, SPN-14 team approvals and the ADM-35 G22 log. It must have the same tokens on every surface, text that a screen reader can read, and no colour-only meaning.
- **No approve controls for an agent.** The banned-control list is generated, not typed: it is every action that A03's `delegable.ts` marks as not delegable to an agent (requires a human, or is a checker, approver, ratifier, veto, countersign or custody-signer act), joined to the controls that render those actions through their `data-action` attribute (the action code) and their accessible name. A control matches when either its `data-action` is on the list or its accessible name matches the label A05 and A51 use for a listed action (Approve, Reject, Sign, Broadcast, Ratify, Toggle, Publish, Allocate, Halt, Halt all, Freeze, Revoke where it revokes someone else's authority, Authorise, Approve as sponsor owner, and so on). A control whose accessible name matches but has no `data-action` is itself a Minor finding on the owner ("add `data-action`").
  - **Console agent persona.** Signed in as the A51 persona "agent of Priya N." through the dev harness, the console must render zero matching controls on any screen, including authorisation slots in A74's screens, and the grey "Agents prepare. A person approves." banner must be present where a person would see a button.
  - **Sponsor agent persona.** Signed in as a sponsor-member agent persona (the seeded sponsor agent acting for Leo Park, or the one A06 seeds for the sponsor tenant) through the dev harness, SPN-01 to SPN-16 must render zero matching controls where the sponsor's policy requires a second person: "Approve as sponsor owner" on SPN-09, the SPN-14 team approvals, and the SPN-16 pending-owner approval of a token. The agent may see its own draft and prepare controls, which are not on the list.
  - The count must be zero for both personas. This is a check of the rendered screen only; the server-side refusal is A12's and A14's.
- **No secrets in evidence.**
  - The copy-once states are captured only with a sandbox token (`rx_sb_`), created for the capture and revoked straight after it. This sandbox-only rule applies to the copy-once capture only; the bug-19 parity check (item 6) needs a live-environment read-only token on the local stack and never captures its secret on screen.
  - Playwright traces, HAR files and DOM snapshots are turned off for that state.
  - Every text artefact this node writes (JSON, HTML gallery, logs, fix items) is scanned for the pattern `rx_[a-z]+_[A-Z2-7]{12}_[A-Z2-7]{32}`, and any match fails the run.
  - The PNG of the card is kept because the founder must see the design, and the evidence records that the token it shows was revoked.
- **Machine-readable findings.** The findings are JSON (`docs\fidelity\findings.json`, schema in `findings.schema.json`), so the build's own agents and the orchestrator read them through the ordinary file path and need no new API.

**8. Findings, fix items and re-runs**

- **Finding fields.** Every finding has these fields:
  - `id` (`FID-nnn`, stable across runs, hashed from screen, state, width, region and property);
  - `severity`: **Blocker** means a missing state, missing compliance copy, a contrast failure, horizontal scroll, a keyboard trap or unreachable control, an unfixed prototype bug, a secret in an artefact or AML wording in a status-bearing place (item 5). **Major** means a wrong token on a primary element (family, weight, colour, radius, pill tone), a font fallback, an untokenised colour or an unknown status string. **Minor** means spacing outside tolerance, a missing test id or a moderate axe finding;
  - the screen code and the state;
  - the width;
  - the region and component;
  - `expected` (token name and value) and `actual`;
  - both screenshot paths;
  - `owningNode`, taken from the charter's `screens.json` for the screen, or from the component's owner (A05) when the same wrong value appears on three or more screens that use that component;
  - `status` (`open | fixed | accepted | design-decision`).
- **Fix items.** `docs\fidelity\fix-items\<node>.md` lists each owning node's items in plain English, with the exact re-run command. The orchestrator attaches the file to that node's next fix loop. This node never edits another node's paths.
- **Design findings.** `DF-A87-nn` is used when the design itself conflicts with N11 or N18. Examples: a design colour pair that fails AA once rendered; design copy that uses an abbreviation without expanding it; a phone layout the design analysis did not decide. These go to the founder under HG-REVIEW with the measured numbers and a proposed change. They are never fixed by recolouring tokens or rewording design copy on the builder's own authority.
- **Re-runs.** After the owners' fix loops, the harness re-runs the affected screens and then the full set once. Every run writes `docs\fidelity\coverage.json` (screen × state × width → `pass | findings | deferred | unreachable`).
- **The founder's gallery.** `docs\fidelity\gallery\index.html` is a static page in the design's own tokens. It shows, for each screen and state, the prototype and the built page side by side at each width, the token mismatches as a list under each pair, and a checklist the founder ticks.
- **The accepted baseline (for A88).** After the founder's HG-REVIEW sign-off, and only then, the harness writes `docs\fidelity\baseline\<code>--<state>--<width>.json` for every `pass` cell. Each file holds, per region, the region id, its A05 component name and the set of token names found (font, colour, radius, spacing, shadow, tone), plus the resolved values. `docs\fidelity\baseline\baseline-manifest.json` lists every file with its SHA-256 and a `baselineSetSha256` over the sorted list, the commit, the design hashes and the sign-off date. This is the "A87's accepted baseline" that A88 compares its checkpoints with and whose set hash A88 freezes. The baseline is never written before the sign-off; a run that finds a baseline dated before the recorded sign-off fails.
- **Re-use by later nodes.** The harness stays runnable on the deferred P2 and P3 screens with the same thresholds (`npm -w e2e/fidelity run fidelity -- --node A92`, and so on), and its README says how. This node does not require A90, A92, A93 or A95 to run it, because none of them depends on A87 in the graph and the scheduler does not enforce milestone order; A94 (which follows A87 through A89 and A88) is the natural place to run it on the P2 screens. Making that binding is a change in A94's spec, not here.

**9. Error and empty states of the harness itself**

- The stack is not up, the seed does not match the design dataset, fonts fail to load, or the design folder's hashes differ: each of these stops the run with a named error, never a partial pass.
- A screen with zero anchored regions is a harness failure, not a pass.
- An empty `findings.json` is written as `{ "findings": [], "run": {…} }` with counts, so "nothing found" and "nothing checked" can never be confused. `coverage.json` must have a result in every cell.

## Owned paths (other nodes do not edit these)

- <workspace>\app\e2e\fidelity\
- <workspace>\app\docs\fidelity\

Inside `e2e\fidelity\` by convention:
- `playwright.config.ts` and `package.json` (the workspace);
- `prototype\` (the read-only harness page and the prototype recipes);
- `app-driver\` (persona sign-in and scenario seeding through A06);
- `states\` (generated state recipes);
- `regions\` (per-screen region maps);
- `compare\` (computed-style extraction, token mapping, untokenised sweep, copy check);
- `a11y\` (axe, rendered contrast, focus order, live regions, reflow and gutters, jargon);
- `bugs\` (the five named tests);
- `d04\` (agent-surface checks and the secret scan);
- `report\` (finding ids, fix-item writer, gallery generator).

Inside `docs\fidelity\`: `README.md` (how to run it, how to read a finding, how an owning node re-runs its screens), `findings.json`, `findings.schema.json`, `coverage.json`, `compliance-copy.json`, `glossary.json`, `fix-items\`, `gallery\`, `baseline\` (per-cell token baselines and `baseline-manifest.json`, written only after HG-REVIEW sign-off).

Evidence goes to `<workspace>\app\evidence\A87.json` and `<workspace>\app\evidence\A87\` (screenshots named `<code>--<state>--<width>--{proto|built}.png`), following A00's evidence-manifest schema. The manifest also lists the baseline files and `baselineSetSha256`, the bug-19 parity token record (id, prefix, `environment = live`, revocation time and the three listing-id sets), the isolation option used for the `mutating` project, and the page classification used by the tipping-off guard.

## Design references

- **The prototype, whole file and read-only.** `<workspace>\design\RealX App.dc.html`:
  - helmet and fonts: lines 10-20;
  - prototype-only role bar, not built: lines 21-31;
  - public shell and header: 33-55;
  - `home` 56-115, `invest` 116-157, `prop` 158-290 (tabs 202-290), `fees` 291-305, `onb` 306-362;
  - app pages 363-521 (`port` 379, `buy` 399, `wallet` 461, `sell` 481, shared table 504, footer 515-517);
  - sponsor shell 522-619 (`spDash` 538, `spNew` 558, `spOffer` 581, `spDist` 589);
  - admin shell 620-733 (sidebar 621-627, `adApprovals` 655, empty state 665, `adKyc` 683, `adToken` 706);
  - Toast 734-736;
  - script 739-921 (`TONE` 740, properties 755, approvals 778-779, state 785, toast timer 786, `ONB` 814-844, distribution chain 844, `lsteps` 845-855, `PG` 867-894, `login` 896).
- **Design tokens.** Design analysis §1.1-§1.5, frozen in A05's `packages\ui\tokens.json` and `design-source.json`. These are the only values the comparator accepts: cream `#F5F2EC`, ink `#17191C`, green `#1F5E4B` (hover `#174838`), brick `#9A4524`, the five-tone `TONE` map, Newsreader / Geist / Geist Mono and the twelve radii.
- **Components.** Design analysis §2 (46 components) as built in A05. The region tags use these names. The twelve reference regions for the smoke subset are PublicHeader, PropertyCard, FeaturedCard, StatusPill (all five tones), KpiGrid, DataTable, Button variants, QuantityStepper, GateRow with AcknowledgeCard, Timeline, ApprovalListCard with ApprovalDetail, SignatureMeter and AdminSidebar.
- **The design extensions.** `design-ext:all`: [[A50 - Design extension A- investor and sponsor screens not in the prototype]] (`RealX App Ext A.dc.html` and its `screens.json`, including the ACC-05 and SPN-16 states, the sign-in screens and the partner sidebar) and [[A51 - Design extension B- operator console screens not in the prototype]] (`RealX Admin Ext.dc.html` and its `screens.json`, including ADM-35, ADM-36 to ADM-40, the seven-group sidebar and the hardware-key confirmation modal). Their HG-REVIEW approvals and change lists are the version compared against.
- **Design versus product map.** Design analysis §6 is the rule for "who wins": the design wins on visuals and the product map wins on content, states and compliance wording. §6.3 covers the phone and console width decisions. §6.4 lists the bugs; items 2, 14, 16, 17 and 19 are verified here.

## Requirements covered

- **Product-map screens — none owned.** Done means every in-scope row of the charter's `screens.json` (the P1 rows of the 96, including ACC-05, SPN-16, ADM-35 and ADM-36 to ADM-40) has a `pass` result in `coverage.json` in every listed state at every required width, and every P2 and P3 row is marked `deferred` with the node that will re-run the harness.
- **Journeys — none owned.** Done means the states that journeys J01-J06, J09-J15, J17 and J19-J21 pass through in A79-A81 appear in the state recipes. They are reached through the same scenario seeds, so the pass looks at the screens people actually see on those journeys. J22 (delegate to your own agent) is covered through its three screens.
- **Gates — none owned.** The screens display G01-G22 but never decide them. Done means every gate-bearing design element renders in design tokens with its compliance copy present:
  - GateRow on INV-02;
  - the G12 AcknowledgeCard;
  - the G13 maker-checker note and hardware-key modal;
  - the G09 PriceBand, where built;
  - the G10 halt banners;
  - the G19 and G20 badges;
  - G22 refusal rows and banners.

  The focus, contrast and reflow rules also hold on all of these.
- **Modules — none owned.** The console modules M01-M27 are covered through their screens.
- **State machines — none owned.** Done means every state label that SM-00 to SM-13 and the A12 token table produce, and that a P1 screen renders, maps to a known tone. An unknown label is a finding.
- **Non-functional requirements.** **N11 Accessibility** (QA checks 50-52) is done when the measured results in item 4 are clean on every in-scope state. **N18 Plain English** (QA check 55) is done when item 5 reports zero unexpanded "expand" terms and the founder has approved the "common" list. **N08** is checked from the visual side only (item 5 guard).
- **Product-map QA report open item 7** (coral contrast in the BrickX-style mocks) is closed here: contrast is measured on the design's actual tokens as rendered, and the coral look appears nowhere. A search of the built assets for "coral" and "k-chip" finds nothing.
- **Founder requirement D04.** Done when the agent screens, the agent marker and the console and sponsor agent-persona "no approve controls" sweeps in item 7 pass, and the bug-19 listing check, run with a live-environment read token on the same seeded data, returns equal listing-id sets from the UI, REST and MCP.

## Gauntlet loop for this node

- **Specify.** This note is frozen together with:
  - A00's `screens.json` and `design-baseline.json` hashes;
  - A05's `tokens.json` hash and contrast table;
  - A50's and A51's `screens.json` hashes and HG-REVIEW records;
  - A06's seed version and `AS_OF`;
  - A03's `delegable.ts` export hash (the source of the banned-control list) and A104's `catalogue.json` hash (the MCP tool for `GET /v1/listings`);
  - the evidence manifests of A79, A80 and A81, which show the stack is green.

  At Specify the orchestrator also opens two upstream follow-ups this node needs and cannot make itself: A01 changes the workspace glob `e2e` to `e2e/*` (item 2), and A05 lists the scrim `rgba(23,25,28,.25)` as a colour token in `tokens.json` if it is not already there (item 3). The build starts only when both have passed.

  The node QA records `specVersion`, those hashes and the hash of its hidden challenge material in the evidence manifest.
- **Build (Grok 4.7 xhigh, split by the planner).** Suggested shape:
  - **Lead session first (about 1.5 hours).** The lead builds:
    - the workspace;
    - the two drivers;
    - the design-hash check;
    - the state-list generator from the three manifests plus the product-map states;
    - the finding schema;
    - the twelve smoke regions.

    Nothing else starts until the smoke subset runs green against the prototype compared with itself and produces findings against a deliberately broken local build. The self-comparison covers only the token comparator and region matching (the prototype driver on both sides of the twelve smoke regions), and it must yield zero findings. The accessibility suite, the copy check, the untokenised-colour sweep and the clickable-`span` rule never run against the prototype, because the prototype has clickable spans and no focus style by design.
  - **Four parallel sub-agents, each owning one folder:**
    - (a) region maps and state recipes for the public, investor and partner surfaces;
    - (b) region maps and state recipes for the sponsor and operator surfaces, including ADM-35 and ADM-36 to ADM-40;
    - (c) the accessibility and jargon suite with the tipping-off guard;
    - (d) the five bug tests, the D04 checks and the secret scan.
  - **Integration.** One full run, finding routing and fix-item files, the gallery, `coverage.json`, `app\evidence\A87.json`, then a re-run after the owners' fix loops.

  The builder never reads `<workspace>\gauntlet\hidden\`.
- **Challenge (node QA, Opus 5.5 high).** In order:
  1. Confirm the design folder's hashes are unchanged.
  2. Run its own token extractor over the same screens and diff it against the builder's comparator output. They must agree on every Major and Blocker.
  3. Run the twenty planted drifts in its scratch copy and require every one to be caught with the right severity and owning node.
  4. Recompute rendered contrast with its own script, agreeing within ±0.01.
  5. On twelve screens at 375, confirm the live-region announcements with its own automated check: Playwright accessibility-tree snapshots before and after each change, plus a `MutationObserver` recording of every text change inside each `aria-live` and `role="status"` region, compared with the builder's live-region results. No screen reader is installed on the build machine (its toolchain is Node, Docker, Git and Python), and installing one is a download that needs the user's permission. If the user approves installing NVDA, a guidepup run that reads NVDA's speech-viewer log may be added and is recorded as a prerequisite in the evidence. A real screen-reader walk by a person belongs to the founder's HG-REVIEW or to the optional independent accessibility audit logged below.
  6. Walk its copy checklist.
  7. Run the node-specific challenges below.
  8. Scan every artefact for the token pattern, and every status-bearing place of item 5 for AML wording.
  9. Check that `coverage.json` has no empty cell and that its counts equal the manifests.
  10. Check that each fix item names a real owning node and a re-run command.

  It writes `app\evidence\A87.qa.md`.
- **Gate, fix loop and escalation.**
  - **What passes the gate.** Pass needs the node QA's evidence and zero open Blocker or Major findings. Every Minor finding is either fixed or accepted by the founder, and the founder's HG-REVIEW sign-off must be recorded in `humanApproval`. Pass unlocks A88.
  - **Fix loops.** Fix loops on this node's own harness are capped at three on the same spec. Fix items on other nodes follow those nodes' own loops.
  - **Escalate to the user when:**
    - an owning node cannot close a Blocker within its three loops;
    - a finding needs the design itself changed (a DF-A87 finding: never resolved by the builder);
    - a finding needs a product-map requirement or A05 token changed (reopened upstream, not patched here);
    - the prototype cannot be run outside the Claude Design host within two hours of trying (fall back to A05's catalogue plus A50's and A51's `evidence` captures as the reference, recorded as an escalation, not a silent switch);
    - a secret is found in any artefact, or AML wording in a status-bearing place of item 5 (Critical: it stops the milestone).

## Node-specific challenges (node QA)

- **A look-alike colour passes by eye (design fidelity).** In its scratch build, the node QA changes the primary button green to `#1F5E4C`. It also changes the amber pill background to `rgba(246,236,214,0.98)` and makes the hover colour `#17483A`. The comparator must report three Major untokenised-colour findings. Each finding must name the element, the actual value and the nearest token, and each must be routed to A05, because the change is in the shared `Button` and `StatusPill`. None of them may be reported as a near-match pass. It then blocks the Newsreader font file: every heading must raise a font-fallback Major finding, although the computed `font-family` still reads "Newsreader".
- **The page looks right but a sentence is missing (compliance).** On PUB-04 it removes "refunded automatically if the soft cap isn't reached" from the aside, keeping identical tokens. On one route it removes the CAR/AFSL footer paragraph. On INV-02 it hides the documents-opened precondition. Each of the three must be a Blocker from the copy check. A pixel diff of under 1 percent must not rescue any of them.
- **A prototype bug comes back through another path (bug 2 and bug 19).** It approves the September distribution through the SDK with Mia Chen's passkey rather than the SPN-09 button, then opens SPN-10. The attestation must still read "Sign now". For Marine Parade it tries four ways in while APR-2290 is pending:
  - the direct URL;
  - PUB-03 sorted by "Below valuation";
  - the sitemap;
  - a live-environment `rx_inv_` read token for Alex Morgan on the local stack, created with a passkey step-up, calling the live `GET /v1/listings` and the A104 MCP tool for that operation against the same seeded data.

  All four must hide it, and the listing ids from PUB-03, REST and MCP must be equal sets. It then swaps in a sandbox `rx_sb_` token: the harness must reject the run as a harness error (a refused `TOKEN_WRONG_ENVIRONMENT` call or a sandbox-dataset answer must never count as agreement), and the evidence must show the live token's `environment = live` and `revoked` state after the run.
- **A disabled control is not a contrast failure, but a faded enabled one is (contrast).** In its scratch build it adds two buttons with identical colours at opacity .4 on INV-02: one `disabled`, one enabled. The disabled one must be recorded for information only; the enabled one must be a Blocker. It also places a caption over the striped photo placeholder and inside an `opacity: .5` container: the harness's ratio must match its own blended or sampled ratio within ±0.05 and fail where it falls under 4.5:1. The A51 modal scrim must not raise an untokenised-colour finding; a scrim at `rgba(23,25,28,.30)` must.
- **Horizontal scroll only after an interaction (reflow).** At 375 it:
  - opens the hardware-key modal on ADM-30;
  - triggers the longest toast;
  - expands the SMSF entity name on SPN-08;
  - opens ACC-05's copy-once card holding a full `rx_sb_` secret;
  - shows a G22 refusal row with its full reason codes.

  In each state `scrollWidth` must stay at 375 or less and the gutters at 16 px or more. A table may scroll only inside its own card.
- **Keyboard and announcements (accessibility).** It approves APR-2291 by keyboard only:
  - Tab to the card and select it with Enter.
  - Open the hardware-key modal. Focus must be trapped, and Escape closes it with focus back on Approve.
  - Reopen it and confirm. The nav badge and the "Approvals waiting" KPI must each announce the new count once, not on every poll.
  - The outcome must stay visible after the 2.6-second toast has gone.

  It then checks that the footer "Complaints" and the whole-row document links are real links reachable by Tab. Where the prototype used a clickable `span`, a `span` in the build is a Blocker.
- **Jargon expanded in the wrong place, or where it must not appear (plain English and tipping-off).** On ADM-10, the first "PEP" appears in a table header and the long form only appears two sections lower. On POR-05, "NPP" is expanded only in a `title` attribute. Both must fail. It then plants "AML review" on an investor POR-06 frozen-funds banner, fully expanded as "anti-money laundering". That must fail as a tipping-off Blocker, not pass as correctly expanded jargon.
- **Evidence leaks a live secret (agent misuse, security).** It runs the ACC-05 capture with a live `rx_inv_` token instead of a sandbox one, with Playwright tracing left on. The run must refuse to capture the copy-once state with a non-sandbox token. The artefact scan must find no token pattern in any JSON, HTML, log, trace or fix-item file. The sandbox token used for the capture must be `revoked` in A12's table when the run ends.
- **An agent persona sees an approve control (D04, design fidelity of authority).** In its scratch build it un-hides the ADM-19 Approve button for AGNT principals, and gives the ADM-35 platform kill switch a plain toggle instead of the `SGL+R` ratify variant. The agent-persona sweep must count one control and fail, and the kill-switch region must fail the `design-ext:ADM-35` comparison. Both must be routed to the owning console nodes (A74 and A77), with a note that the server-side refusal is tested in A12 and A14 and is not claimed by this check. It then shows "Approve as sponsor owner" on SPN-09 to the sponsor agent persona, and an "Allocate" button labelled only by its text (no `data-action`) to the console agent persona: the sponsor sweep must count one control routed to SPN-09's owner, and the console sweep must catch "Allocate" by accessible name.
- **Test order must not change the result (stability).** It runs the full suite twice with the test list shuffled inside each project (the harness's own shuffle, with the seed recorded), and once with the `mutating` project run first and the database re-seeded before the `read-only` project. The ids in `findings.json` must be identical across the three runs, and the before-and-after tables check in item 2 must pass in each.
- **Ambiguous anchors and the "nothing checked" pass (harness honesty).** It seeds a second property named "Kent Street Terrace". The region anchors must fail closed as ambiguous and must not pick the first match. It empties one screen's region map: the run must report "zero regions checked" as a harness failure. `coverage.json` must never show `pass` for a cell that compared nothing.

## Pass criteria

- **Coverage.** `coverage.json` has a result for 100 percent of in-scope screen × state × width cells. The cell count equals the count derived from the charter's `screens.json`, A50's and A51's manifests (including the extension frames that are not charter rows: the `SGN-` screens, the four partner frames and A105's pages inside them, and A51's shell frames) and the product-map states. There are zero `unreachable` cells that do not have an open or fixed finding on the owning node. Every P2 and P3 row is `deferred` with its node id.
- **Token agreement.** Across all compared regions, zero open Major or Blocker findings. Font family and weight, colour and radius agree on token name exactly. Font size and line height agree within ±1 px, and padding and gap within ±2 px at 1280. The untokenised-colour sweep finds zero values outside `tokens.json` on any captured page. All six font faces report `loaded` on every page, and the built pages make no request to `fonts.googleapis.com`.
- **Harness honesty.** The prototype compared against itself through the token comparator and region matching (not the accessibility suite) yields zero findings. The node QA's twenty planted drifts are all caught with the expected severity and owning node. Its independent extractor agrees with the builder's comparator on every Blocker and Major finding.
- **Accessibility.**
  - axe reports zero critical or serious findings on every in-scope cell.
  - Rendered contrast, computed with blending of semi-transparent colours and parent opacity and with pixel sampling over images and gradients, is at least 4.5:1 for all text of enabled content, and at least 3:1 for large text, focus indicators and required boundaries of enabled controls. Disabled and `aria-disabled` controls are recorded for information only (WCAG 1.4.3 exemption). The token pairs match A05's table within ±0.01.
  - Every interactive element is reachable by keyboard in reading order, with a visible focus ring of at least 3:1, and no focused element is fully covered by a sticky or fixed element.
  - Every modal traps and restores focus.
  - Every listed counter and toast is a live region that announces exactly once per change.
  - `scrollWidth` is at most the viewport width, and both gutters are at least 16 px, in every 375 px cell and every 320 px cell of the public, investor, sponsor and partner surfaces, including the injected long-string and open-modal states.
  - No clipped, overlapped or lost text at 200 percent text size or under the WCAG 1.4.12 spacing styles.
  - Every hover or focus tooltip is dismissable, hoverable and persistent (1.4.13).
  - Every control meets WCAG 2.5.8 (at least 24 × 24 px at 375, or its spacing or inline exception, recorded).
- **Plain English.** Zero unexpanded "expand" terms on any page, including `/docs`. The founder has approved the glossary's "common" list. Zero occurrences of AML, AML/CTF, anti-money laundering, SMR, EDD, enhanced due diligence or "suspicious" in the status-bearing places item 5 lists (viewer's own banners, pills, notices, activity, errors, token states and API or MCP results, and sponsor views of a named investor). Review wording in those places is only from `genericReviewStrings`. The counsel question on where a generic legal reference ends is logged.
- **Prototype bugs.** The named tests for bugs 2, 14, 16, 17 and 19 pass on three consecutive runs with no retries, each routed to the owners that `screens.json` gives. The bug-19 check records the parity token's `environment = live`, its revocation, and the listing ids returned by PUB-03, `GET /v1/listings` and the A104 MCP tool as three equal sets without Marine Parade.
- **D04.** ACC-05, SPN-16 and ADM-35 pass in every drawn state. The agent marker passes on every listed surface. The console agent persona and the sponsor agent persona each see zero controls from the banned list generated from A03's `delegable.ts` (matched on `data-action` and accessible name). The artefact scan finds zero token-pattern matches, and every capture and parity token is `revoked`.
- **Stability.** Two consecutive full runs on the same build, and the shuffled-order and mutating-first runs of the challenge, produce identical finding ids. The `read-only` project leaves the audit head hash and the approval, token and kill-switch tables unchanged. The full run completes in under 60 minutes on the local machine (four workers for `read-only`, one for `mutating`).
- **Baseline.** After the HG-REVIEW sign-off, `docs\fidelity\baseline\` holds one file per `pass` cell and `baseline-manifest.json` with `baselineSetSha256`, dated after the sign-off, and both are listed in the evidence manifest.
- **Closure.** Every finding has an owning node and a final status. Every `DF-A87` finding has a founder decision. `app\evidence\A87.json` validates against A00's evidence-manifest schema and matches disk. The design folder's hashes still equal `design-baseline.json`. `npm run verify` is green repo-wide.
- **Sign-off.** The founder's HG-REVIEW sign-off of the visual QA is recorded with a date and the list of accepted Minor findings.

## Human gates touched

- **HG-REVIEW (blocks this node's gate and therefore A88).** The founder opens `docs\fidelity\gallery\index.html` and walks the prototype and the build side by side, in the README's journey order, at 1280 and 375. The founder then:
  - accepts or rejects each open Minor finding;
  - decides every `DF-A87` design finding (and confirms the earlier decision on A05's DF-A05-01 border contrast);
  - approves the glossary's "common" list;
  - signs off the visual QA.

  *Agents prepare:* the gallery, a one-page summary (counts by severity and owning node, fixed versus accepted), the list of DF findings with measured numbers and proposed changes, and the glossary. *Evidence:* `humanApproval` in `app\evidence\A87.json` with the date, the decisions and any change requests. Change requests go through the owning nodes' fix loops like any other finding. A node with a human gate cannot pass on an agent's claim (A00).
- **Not a listed gate, logged for the founder.** This node produces software evidence against WCAG 2.2 AA. It is not an accessibility conformance statement. Whether to commission an independent accessibility audit before the P2 retail launch is logged as a founder decision, not as a blocker here.
- **Not touched.** No licences, counsel opinions, vendor contracts, production keys, real money, mainnet, smart-contract audit, pentest or go-live. Every sign-in is a ceremony against A11's sandbox authenticator. Every token used is created for the run and revoked when it ends: sandbox `rx_sb_` tokens for the copy-once captures, and one read-only live-environment `rx_inv_` token on the local stack for the bug-19 parity check. The stack is the local compose profile.
- **HG-LEGAL (question only, does not block this node).** Where a generic reference to the Anti-Money Laundering and Counter-Terrorism Financing Act 2006 on help, privacy and onboarding pages ends and a tipping-off risk begins (item 5) is logged for counsel through the founder. Until answered, the harness's page classification stands and is recorded in `compliance-copy.json`.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
