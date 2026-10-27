---
title: "R06 Agent Access for Every User"
created: 2026-09-23
updated: 2026-09-23
status: sent
tags: [realx, report, agents, api, product-map, build-plan]
turn: 10
prompt: "make sure any user type has api token management to let their agents handle all their account actions"
---

# R06 Agent Access for Every User

**Open the updated product map: https://claude.ai/artifact/23baWULw9bDd4kCsbkeAfB (version 2).** Every user type can now give their own AI agent a scoped API token. Recorded as founder decision D04 in [[Decisions]].

## What now exists

| Who | Screen | What their agent can do |
|---|---|---|
| Every investor type | ACC-05 API tokens and agents | Browse, invest, withdraw to their verified bank account, vote, get statements |
| Sponsor users, valuers, property managers | SPN-16 API tokens and agents | Draft listings, file rent rolls, prepare distributions, post updates |
| Every staff role and admin-console partner user | ADM-35 Agent and API access | Read, draft and prepare. Security also gets oversight, policy and a kill switch |

- **New gate G22 "Agent authority":**
  - tokens are created with a passkey;
  - scoped no wider than the person;
  - limited, expiring and revocable;
  - every call is logged as "agent for <person>".
- **Guardrails:**
  - staff agents never approve, check or sign;
  - identity checks, the first wallet, token creation, bank changes and recovery stay with the person.
- **New journey J22 "Delegate to your own agent"** shows it end to end.
- Product map rebuilt: 91 screens, 22 journeys, 22 gates, 0 warnings.

## The build plan

- D04 is inside the running plan. The drafts were kept; the judges and synthesis re-ran with D04 in view, so the API, MCP server and token model are designed in from the start.
- Cost: about 30 minutes. New ETA: about 00:30–01:30 IST. The per-node spec gauntlet (~80 nodes × write → challenge → fix) is the long part.

## For counsel

- Which declarations must a person make personally rather than through their agent (wholesale status, target-market answers, disclosure acknowledgements)? Until counsel answers, the design requires the person's own passkey for these.

## Logged

- Research Log turns 8–10.
- Search Log S-87 to S-95.
- [[RealX Product Map]], [[00 Start Here]] and memory.
