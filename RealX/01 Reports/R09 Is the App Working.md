---
title: "R09 Is the App Working"
created: 2026-09-30
updated: 2026-09-30
status: sent
tags: [realx, report, app-build, docker]
turn: 22
prompt: "is the app working? can you use docker desktop etc if needed and see if it works? / look at obsidian vault called RealX as well"
---

# R09 Is the App Working

**The RealX app is not running. The vault says the build never started, and there is nothing for Docker to launch.**

What the vault (handoff dated 25 Sep 2026) and the disk say:

1. **Code:** not started. The app is supposed to live at `<workspace>\app`. That folder does not exist. Neither does `<workspace>\gauntlet`.
2. **Docker:** the stack is planned for later milestones (first full boot at the end of AM1, in A23). Docker Desktop is running (29.8.0), but there is no RealX container or compose file.
3. **What exists:** the research pack, the product map, and the 107-node build plan. Only spec A05 has passed its re-check. The plan does not start a node until its re-check passes, unless the founder names the node.

**What runs today:** the design prototype `design/RealX App.dc.html`. Served locally, its home page rendered in the visitor view, with "Own part of a building that pays rent." and the Kent Street Terrace offer card. The temporary preview server was stopped afterwards.

Evidence: S-136 to S-140 in Search Log.
