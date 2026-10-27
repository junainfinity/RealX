---
title: "R11 No View-As Switch"
created: 2026-09-30
updated: 2026-09-30
status: sent
tags: [realx, report, app-build, design, sign-in]
turn: 26
prompt: "the realx prototype view as should be removed for final project and each user type should experience their flow organically"
---

# R11 No View-As Switch

**The final product has no "view as" role switcher in any build, local and test included (decision D20). Each user type reaches its own flow by signing up or signing in as that user, and lands on its own home. `check_plan.py` passes 40 of 40 checks after the change.**

## What was there before

- Production never shipped the prototype's black "RealX prototype · view as" bar.
- The plan kept a dev-only copy, `DevRoleSwitcher`: exported by A05, mounted by A52 (web) and A53 (console) in local and test runs, used by QA probes in A61, A79 and A87, and named in the A89 local launch guide as the way to sign in as the seeded personas.

## How each user type gets in now

| User type | Front door | Lands on |
|---|---|---|
| Visitor | Opens the public site signed out | `/` |
| New investor | "Get started", email verification, first passkey | A56 onboarding |
| Returning investor | "Log in" (SGN-01) | `/portfolio` |
| Sponsor user | "Log in" (SGN-02), an invitation email, or `/s/apply` | `/s/[tenant]` or `/s/apply` |
| Partner | SGN-03 | `/s/[tenant]` or `/p` |
| Staff | ADM-36 on the console's own origin, hardware-attested passkey | The sidebar `GET /v1/iam/me` grants |

A person who holds both an investor account and a sponsor membership sees their own "Sponsor console" link. Nothing lets one person act as, or preview, another.

## How testing and the local walkthrough work without a switcher

- **Automated runs:** Playwright opens the real sign-in page and answers the passkey ceremony with a virtual authenticator attached to that person's browser context. The authenticator is loaded with the persona's sandbox passkey from A06's gitignored `credentials.json`. There is one browser context per person, and app code never imports the helper.
- **Founder walkthrough:**
  - A61's `preview:window <persona>` and A89's `launch:window <persona>` open a separate, signed-out browser window whose authenticator holds only that persona's passkey.
  - The founder clicks "Log in" and signs in themselves. The command never signs in, sets no cookie and calls no API.
  - A new investor can also sign up with the founder's own passkey (Windows Hello).
- **A52's harness pages** (for example `/dev/fresh-assertion`) stay, behind a narrower `REALX_TEST_HARNESS` flag. They never sign anyone in.

## What enforces it

- A52, A53, A83 and A87 build checks find no `DevRoleSwitcher`, role-switcher component or persona name in any build. They also confirm that `?dev=1`, `?as=` and stored `role` values never change who is signed in.
- [[App Gauntlet]] §4 criterion 5 and the node QA prompt in `milestone-workflow.js` treat any switcher or sign-in shortcut as a High finding.

## What is kept

- **The design files' bars:** the prototype and design extensions A and B keep their persona bars, because they are design-review devices for the founder, not product.
- **Staff "view as tenant"** (A11 §4, A53, A67): a support tool that narrows a staff session to one sponsor's data. It requires a reason, is audited, expires after 60 minutes, and never changes who is signed in. It is kept unless the founder wants it removed as well.

## Files changed

A05, A51, A52, A53, A61, A79, A83, A87, A89, [[Design Integration]], [[App Gauntlet]], [[Decisions]] (D20), `build_matrix.py`, `milestone-workflow.js`, and the regenerated [[Build Orchestration]], [[Screen Coverage Matrix]] and canvas. Logged as S-162 to S-169 in the Search Log and Turn 26 in the Research Log. Not committed yet.
