---
title: "R10 D19 Model Roles Rework"
created: 2026-09-30
updated: 2026-09-30
status: sent
tags: [realx, report, app-build, gauntlet, orchestration]
turn: 23
prompt: "Create a graph engineering plan with gauntlet loops for every node. Opus 5.5 high plans, Grok 4.7 high executes, Opus 5.5 checks, a does final QA / node level qa - Opus 5.5 High, graph level qa - Fable 5.1 high"
---

# R10 D19 Model Roles Rework

**The 107-node graph plan now uses the D19 roles at every node, and it has a new graph-level QA gate at each milestone close. `check_plan.py` passes 40 of 40 checks, and the workflow dry-run passes 47 of 47 scenarios. The spec re-checks have not run yet.**

## The loop at every node

| Step | Role | Model |
|---|---|---|
| Specify, Plan, clerk (records, merges) | Planner | Opus 5.5 (high) |
| Oracle (hidden tests first), Challenge at every loop, re-verify after rebase | Node QA, fresh session each time | Opus 5.5 (high) |
| Build and Fix | Builder | Grok 4.7 (xhigh) |
| Graph gate once every node of the milestone has passed | Graph QA | Fable 5.1 (high) |

- At most 3 fix loops per node, then escalate.
- The graph gate checks cross-node contracts, journeys, every published oracle on `main`, coverage and evidence ([[App Gauntlet]] §9). A fail reopens each owner node (commit `<id>: graph QA fix <round>`, counted against its 3 loops). After 2 failed rounds, or if a finding's owner is outside the milestone, the milestone escalates to the user.
- The next milestone cannot start until the previous graph gate has passed.
- Each run first asks all four models their names and stops on any mismatch. No role falls back to another model.

## What changed

1. [[App Gauntlet]]: role table, flowchart and new §9, graph QA.
2. `milestone-workflow.js`: role probes, Plan step, graph gate, reopen logic, report fields `models` and `graphGate`. `test-workflow.mjs`: 47 scenarios.
3. New runner `run-milestone.mjs` (Cursor SDK). The Claude Code Workflow tool cannot call Grok, so runs go through it ([[Build Orchestration]] §8). It has not been run against real agents.
4. All 107 node specs: role lines, section headings, 823 "Fable" mentions and 223 QA pronouns rewritten by `d19_node_roles.py`. Persona pronouns (Mia Chen, Aisha Karim, the CCO …) were left alone and listed in `06 Evidence/Tools/app_plan/d19-pronoun-review.txt`.
5. [[App Build Plan]], [[Build Orchestration]], [[Human Gates]], [[Design Integration]], [[Screen Coverage Matrix]], the canvas, [[00 Start Here]] and Handoff - App Build Plan.

## Still open

- **Spec re-checks.** Opus 5.5 (high) per spec, then a Fable 5.1 (high) review of the whole plan. The handoff says to run the cross-node reconciliation pass first: 35 of the 37 open High findings are conflicts between specs (A16, A14, A28 and A38 are named most).
- **Usage.** Opus 5.5 is now the binding allowance: it plans, clerks and QAs every loop.
- **Estimates.** The hours in the plan do not yet include the graph gates.

This is research and planning, not legal advice. No human gate was crossed.
