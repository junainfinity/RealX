---
title: "R07 App Build Plan Assembled"
created: 2026-09-24
updated: 2026-09-24
status: sent
tags: [realx, report, app-build, qa]
turn: 17
prompt: "(stage A of the Opus finish completed; follows 'Riskiest now, rest Saturday')"
---

# R07 App Build Plan Assembled

**Open [[App Build Plan]] first. The graph is in [[App Development Graph]], and the canvas is `09 App Build/RealX App Build.canvas`.** The plan is complete and verified. An independent Opus 5.5 review rated it **Pass with fixes, 7/10**, in App Plan QA Report.

## What exists now (`09 App Build/`)

- **107 build steps** (A00–A106), each with a spec in `Nodes/`, plus 2 follow-up items. They are grouped into 11 milestones, AM0–AM10.
- **Plan documents:**
  - [[App Build Plan]];
  - [[App Development Graph]] and the canvas;
  - [[App Gauntlet]];
  - [[Build Orchestration]], which includes a runnable per-milestone workflow;
  - [[Human Gates]];
  - [[Design Integration]];
  - [[Screen Coverage Matrix]];
  - App Plan QA Report.
- **Coverage:** all 91 screens (plus 5 undrawn admin screens), 22 journeys, 22 gates, 27 modules (lending excluded) and 14 state machines.
- **Engineering decisions:** D05–D18 are proposed in [[Decisions]] for the founder to confirm.
- **Checker script:** `check_plan.py` passes all 40 of its checks.

## What changed in the numbers

- **Size:** 1,092–1,591 agent-hours in total (1,225–1,990 with follow-up loops). P1 launch at about 566–860 agent-hours on 3 streams, one milestone at a time.
- **Why it grew:** the old figure of 183–284 hours came from the first graph. The written specs added 344 dependencies, higher estimates and a Docker lock, and the critical path is now 388–562 hours to P1.

## Not finished

- **Spec re-checks:** only A05 has passed a re-check. The build will not start an unconfirmed spec unless you name it.
- **Batch 1 is running:** the 12 specs whose fixes for critical money and authority findings are unconfirmed. More batches follow while weekly usage stays under ~92%. The rest run after the Sat 26 Sep 16:30 IST reset.
- **For you:** confirm D05–D18, five product questions, and the counsel questions CQ-01 to CQ-10 (App Plan QA Report §6).
- **Usage:** Fable QA at every loop of the build needs a weekly budget per milestone ([[App Build Plan]] §7).
