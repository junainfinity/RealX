---
title: "Australia Notes"
created: 2026-09-23
updated: 2026-09-23
status: active
tags: [realx, research, australia, regulation]
---

# Australia Notes

**Australian rules that shape RealX, extracted from the research pack.**
- Labels are the pack's: [C] confirmed, [I] inferred, [S] speculative.
- Section numbers refer to [[australia]] unless another file is named.

Research, not legal advice.

## 1. What the token is
- A fractional property token is "likely to be an interest in a managed investment scheme" (ASIC INFO 225, Example 9, "83Smith") [C] (§1).
- Tokenising changes the register, not the wrapper. The trustee's or RE's register (s169) is the legal record unless the constitution designates the token ledger [I] (§5; [[03-tokeniser-liquidise]] §2.2).
- The RBA's Project Acacia report names enforceable on-chain ownership records and legal settlement finality as open issues [C] (§10).

## 2. Licence routes (§2)

| Route | What it allows | Time |
|---|---|---|
| Corporate authorised representative (CAR) of an AFSL holder | Distribute or arrange under the partner's licence | Weeks [I] |
| RE-for-hire | A professional RE issues; the platform acts as manager or promoter | Months [I] |
| Wholesale unregistered MIS | Wholesale investors only. The trustee needs an AFSL (INFO 251) [C] or acts as an authorised representative [I] | Lighter [I] |
| Own RE (registered MIS) | Retail. Requires a public company, compliance plan, PDS, TMD and an RG 133 custodian. NTA: A$10M with custody, A$150k if custody is incidental | 6–12+ months [S] |
| Australian market licence | A secondary venue | 12+ months [S] |
| Crowd-sourced funding | Likely unavailable: companies whose main business is investing are excluded [I] | — |

**Wholesale tests (§3):** the investor puts in at least A$500k, or holds an accountant's certificate for net assets of at least A$2.5M or gross income of at least A$250k in each of the last two years. Certificates last 2 years [I]. No change to the thresholds has been enacted [I][S].

## 3. Digital Assets Framework Act 2026 (§2, §10)
- Act No. 38 of 2026. Royal Assent 8 Apr 2026; commences **8 Apr 2027**. Existing operators get 6 months' transition, to **8 Oct 2027** (s1732) [C].
- Low-value exemption: at most A$10m of transactions in 12 months and at most A$5,000 entry value per client (s911A(2)(ja)) [C].
- A tokenised custody platform (TCP) creates one token per underlying asset (s761GD), so fractional tokens probably fall outside that definition [I]. Holding tokens for clients may make a platform a digital asset platform (DAP) [I].
- The Act carves out "public digital token infrastructure". Tokeniser relies on this carve-out for its trading pools, an untested position [C][I] (§6).

## 4. Trading, custody and money
- A facility where offers to buy and sell are regularly made or accepted is a financial market. It needs a market licence or an exemption (s791A), and settlement may need a CS facility licence [I] (§6).
- A bulletin board or bilateral deals may fall outside that definition, depending on design [I] (§6).
- Scheme property sits with the RE or an RG 133 custodian. Client money sits in an ADI trust account [I] (§5).
- ASIC relief exists for omnibus custody (Instrument 2025/871) and for stablecoin distributors (Instrument 2025/867) [C] (§5).

## 5. Property rules: residential vs commercial

| Rule | Residential | Commercial | Source |
|---|---|---|---|
| FIRB | Foreign persons are banned from buying established dwellings from 1 Apr 2025 to 30 Jun 2029. Exceptions: redevelopments adding at least 20 dwellings; commercial-scale housing (build-to-rent, student, aged care); PALM employers. Residential acquisitions must be notified regardless of value | Indirect interests in land-rich trusts can need FIRB approval above thresholds [I] | §9 [C]/[I] |
| FIRB and fractional units | Guidance Note 6 v5 does not address fractional or tokenised interests | Same | §9 [C] |
| Vacancy fee | Foreign owners pay it if the dwelling isn't occupied or available for rent 183 days a year | — | §9 [C] |
| Landholder duty | Triggered above thresholds in land-rich trusts (e.g., 20% for private unit trusts in NSW), with associates aggregated | Same | §8 [I] |
| Foreign surcharges | Surcharge duty and land tax when a trust is "foreign" (e.g., at least 50% foreign-held in NSW) | The pack does not split this by property type | §8 [I] |
| GST on rent | Input-taxed | Taxable | §8 [I] |

## 6. AML, distribution and tax
- **AML:** issuing, dealing in or arranging MIS interests are AUSTRAC designated services. Tranche 2 (real-estate professionals) applies from 1 Jul 2026 [C] (§4).
- **DDO:** each product needs a TMD. Distributors report significant dealings outside the TMD within 10 business days [I] (§3).
- **Advertising:** RG 234 was updated on 9 Jun 2026 [C] (§7).
- **Tax** [I] (§8):
  - AMIT/AMMA statements;
  - MIT withholding of 15% or 30% for foreign residents;
  - TFN withholding of 47%;
  - CRS/FATCA reporting;
  - foreign-resident capital-gains withholding of 15% (per a QA note).

## 7. The market
- Australia has no live retail per-property on-chain tokeniser [C] (AU summary §0).
- **BrickX:** mainly residential (Sydney, Melbourne), one trust per property, an 80% sell floor and a 5% holder cap [C] ([[01-brickx]]).
- **Assetora:** one registered scheme with 130+ sub-funds. Its market was frozen from Jul 2024 to Aug 2025 after NDIS vacancies [C] ([[05-assetora-domacom-bricklet]]).
- **Tokeniser:** B2B SaaS for 160+ issuers, wholesale by default, no property-specific product [C] ([[03-tokeniser-liquidise]]).
- **Entry scorecard:** Australia scores 13/25, the lowest of five regions, but has the most open field [I] ([[regional-comparison]] §6).
- **The pack's plan for an Australia-based founder:** start wholesale now, and time the retail launch to 8 Apr 2027 [C] ([[implications-for-realx]] §2).

## 8. Questions for Australian counsel (collected so far)
1. Does RealX need a TCP licence, a DAP licence, or neither under the DAF Act?
2. Does the low-value exemption (A$10m a year, A$5k per client) cover an early launch?
3. Would launching wholesale before 8 Apr 2027 make RealX an existing operator, with transition to 8 Oct 2027?
4. What authorisations does RealX need to arrange sponsors' offers, and what is the CAR's scope?
5. Can a resale board for units across many sponsors' trusts run without a market licence, or with ASIC relief?
6. How does the FIRB established-dwelling ban apply to fractional trust units?
