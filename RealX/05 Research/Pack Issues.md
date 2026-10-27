---
title: "Pack Issues"
created: 2026-09-23
updated: 2026-09-23
status: active
tags: [realx, research, qa]
---

# Pack Issues

**Inconsistencies Claude found in the research pack.** None has been fixed: the pack stays as delivered unless the user asks for fixes. The pack's own QA corrections are in qa-summary, and its cross-file contradictions are in [[conflicts-and-gaps]].

| # | Issue | Where | Severity | Found |
|---|---|---|---|---|
| I-01 | Brickken's `needKyc=false` flag. QA corrected the Brickken profile: the flag is sandbox-only, and production rejects it. Downstream files still carry the old claim | `04-synthesis/implications-for-realx.md` line 51 ("allowed in production"); `04-synthesis/compliance-matrix.md` line 104; `05-admin-panels/00-overview.md` line 183; `05-admin-panels/06-per-platform-admin-panels.md` lines 183 and 472 | Medium: a wrong claim about a named competitor in the builder playbook | S-32 |
| I-02 | `methodology.md` lists `00-method/verification-report.md` as an output, but the file does not exist. The `00-method/qa/` reports replaced it | [[methodology]] §3 | Low | S-11 |
| I-03 | Admin role count: the README says 40 roles; the admin overview says 41 (CEO added in QA) | README ("What the admin panels look like"); [[00-overview]] §10 | Low | Turn 1 reading |
| I-04 | Archetype codes collide: the admin archetypes A1–A6 are a different clustering from the platform archetypes A1–A7. The pack's QA already annotates this | [[00-overview]] §6 | Low (already annotated) | S-12 |
| I-05 | BrickX's price floor. The pack says the order book has a floor at 80% of the latest valuation, citing the 2019 PDS. On 23 Sep 2026 BrickX's live listings show lowest available prices up to 34.98% below valuation (e.g., KGT01, BAN01), several clustered just under 35%. That suggests the floor has changed, perhaps to about 65% of valuation. This is inferred from listings; check the current PDS | [[01-brickx]] §9; BrickX investments page | Medium: RealX's price-band design cites this benchmark | S-50 |

**Checked and not an issue:** the one apparent broken link is the literal text `[..](x.md)` inside a QA report (S-13). All 75 real relative links resolve.
