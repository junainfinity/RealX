---
title: "R02 Australian Launch Path"
created: 2026-09-23
updated: 2026-09-23
status: sent
tags: [realx, report, australia]
turn: 2
prompt: "out of US" / "Out of Australia, sorry"
---

# R02 Australian Launch Path

> **Prompt (turn 2):** "out of US", corrected mid-turn to "Out of Australia, sorry".
> **Sent:** 23 Sep 2026, ~11:12 IST. Below is the text as sent in chat, word for word. Links now point to the vault copy of the pack. Details: [[Australia Notes]]. Decision: D01 in [[Decisions]].

---

**Being based in Australia changes the plan.** The pack's default was to launch in the US first and reach Australia in months 18–30. For an Australia-based founder it recommends something else ([[implications-for-realx#2. Phased go-to-market by region|implications-for-realx.md:118]]): **take wholesale investors now, and launch to the public around 8 Apr 2027.** That's when Australia's new Digital Assets Framework Act starts, about 6.5 months from now.

## The Australian path

Under ASIC's own guidance, a fractional property token is an interest in a managed investment scheme (MIS). So everything runs through scheme licensing.

1. **Now (weeks): wholesale-only property trusts.**
   - RealX operates as an authorised representative (CAR) under an existing licence holder's financial services licence (AFSL).
   - Investors must put in A$500k or more, or have an accountant's certificate showing A$2.5M in net assets or A$250k income.
2. **Before 8 Apr 2027: get a lawyer's opinion on the new Act.** Three questions to ask:
   - Does RealX need a tokenised custody platform (TCP) licence, a digital asset platform (DAP) licence, or neither?
   - Does the small-operator exemption (up to A$10M a year, A$5k per client) cover an early launch?
   - Would launching wholesale now count as "already operating", which gets 6 months' grace to 8 Oct 2027?
3. **In parallel (months): hire the retail setup.**
   - You need a responsible entity (RE) to run the scheme, a custodian to hold the property titles, and a unit registry.
   - Use one registered scheme with a sub-fund per property, as Assetora does.
   - Apply for RealX's own RE licence at the same time (6–12+ months) and use the hired RE until it comes through.
4. **Around 8 Apr 2027: public launch.**
   - The token (ERC-3643) copies the RE's unit register, which remains the legal record.
   - Money moves in AUD; AUD stablecoins (AUDM/AUDD) can pay out rent.
5. **Resale: a controlled listing board first.** An open trading market needs a market licence (12+ months). Start with a listing board or a regulator exemption, plus a price floor like BrickX's 80% of valuation.

## 5 Australian rules that shape the product

1. **Foreign buyers:** foreigners can't buy existing homes until 30 Jun 2029. Residential properties go residents-only, or get a cap on foreign holdings.
2. **Stamp duty:** extra duty applies when one holder owns too much of a property trust (e.g., 20% in NSW). The token must enforce holder caps; BrickX caps holders at 5%.
3. **Trading:** any regular buy/sell facility legally counts as a market. No open order book or automated liquidity pool without a licence or exemption.
4. **Target market:** each property needs a document defining who it's for. Significant sales outside that group must be reported to ASIC within 10 business days.
5. **Legal record:** the RE's register is the legal record, not the blockchain. The Reserve Bank says it's still unresolved whether an on-chain record counts legally.

The full checklist of 19 obligations is in [[australia#Compliance obligations checklist|australia.md:159]].

**Trade-off:** Australia ranks lowest of the 5 regions in the pack's scorecard (13/25; the EU is highest at 17), because the public launch and resale both need extra licences. But it's the only region where no one offers a per-property token product to everyday investors.

Founder questions answered: 1 of 15 (home base: Australia).

Next: reply "own", "marketplace" or "both". "Own" means RealX buys and runs the properties in its own trusts, like BrickX. "Marketplace" means RealX lists other sponsors' property trusts, like Tokeniser. That decides which licence RealX itself needs.
