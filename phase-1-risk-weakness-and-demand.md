# Phase 1 — Weakness map and demand map

**Lane:** CREDIT · **Model:** 2 — Credit Risk · **Phase:** 1 of 6
**Revision:** 1 · **Date:** 11 November 2026 · **Author:** Sharmila Sirivaram
**Registration:** `CREDIT-RISK-P0-PREREG-001` (frozen 2026-11-11) · **Evidence:** `phase1-risk/out/`
**Harness:** [`phase1-risk/`](phase1-risk/) — `weakness.py` · `intervals.py`
**Status:** base **UNMEASURED**. Every weakness row below is either measured on the **reference model**,
sourced from **public literature**, or marked **UNMEASURED** with a commission. None is a claim about a
model that has not been run.

> **The one sentence.** Phase 1 does not invent weaknesses; it names them, prices each one in the
> smallest difference that would count as real, and attaches the measurement that would prove it —
> and on this lane the base is unmeasured, so the honest map has an empty base column, a measured
> reference column, and a commission register where the claim would go.

---

## 0. Read this first

**What Phase 1 is for.** Phase 0 fixed what to measure. Phase 1 answers two questions: *where does the
model lose points that a buyer would pay to recover*, and *what would that buyer actually accept?* The
first is the weakness map; the second is the demand map. Together they decide what Phase 2 sources
data **for** — because a mixture built around what the model already does well buys a few points near
a ceiling and is named maintenance, not mission.

**What this packet cannot say.** It cannot say "the base is weak at calibration". The base has never
been scored. Any sentence of that form in this document carries an **UNMEASURED** mark and a
commission number. What it *can* do — and does — is measure the ground the base will stand on: the
reference model's errors, the segment where the task is hardest, the interval around every difference,
and the acceptance tests a paying customer would run.

**The two maps, in one line each.**

- **Weakness map:** nine rows, of which **two** are measured weaknesses on Taiwan, **three** are
  structural findings (they describe the data, not the model), **one** is a measured non-weakness, and
  **three** are unmeasurable today and commissioned.
- **Demand map:** five buyer archetypes, **eight** acceptance criteria, of which **six** can be
  measured on corpora we hold and **two cannot** — explanation and out-of-time stability. Those two
  are the difference between a model and a product a regulated buyer can sign for.

---

## 1. The rule this phase runs on

The Recipe's Phase 1 text is unambiguous:

> *Per benchmark: base score, why points are lost, the smallest improvement that counts as real
> (per-benchmark MDE), and the stretch target… Every claimed weakness needs a measured base number
> behind it, or an UNMEASURED mark and a measurement commissioned.*

So every row in §3 carries three things: the **evidence class**, the **measured number** it rests on,
and the **MDE** — the smallest improvement that is distinguishable from noise on that benchmark. A
weakness without all three is not a weakness; it is a hypothesis with a test attached, and it is
labelled as one.

| Evidence class | What it means | Where it comes from |
|---|---|---|
| **A — measured here** | produced on the frozen folds by a script in `phase1-risk/` | `out/weakness.json`, `out/intervals.json` |
| **B — public literature / benchmark** | someone else's measurement, cited and labelled | §7, §5 |
| **C — UNMEASURED, commissioned** | no measurement exists; a test is named and priced | §10 |

The base's own row is class **C** in every table. That is not a placeholder: it is the honest state,
and `CM-R1` is the commission that ends it.

---

## 2. What the reference model is, and why it is allowed to stand in

The reference is the gradient-boosted control from Phase 0: `HistGradientBoostingClassifier`, seed
20261111, 300 iterations, fitted inside the frozen folds, scored **out-of-fold** so no row is scored
by a model that saw it. It is not a contender and it is not the base. It is the **floor of what the
job can be done with**, and its error structure is the best available proxy for where any model —
including a 4 B decoder — will lose points.

Why that proxy is legitimate for Phase 1 and not for a verdict: Phase 1 is asking *where the task is
hard*, not *how good our model is*. The task's difficulty is a property of the data. When Phase 6
asks how good the base is, the reference becomes the bar and the proxy stops being a substitute.

**One caution, stated before the numbers are read.** The reference sees 23 clean numeric fields and
uses them well. A language model sees the same fields serialised as text. A weakness that is
"structural for the task" (thin-file, imbalance, missingness) transfers; a weakness that is
"structural for trees" (splits on thresholds) may not. §7 separates the two.

---

## 3. The weakness map

Nine rows. The **MDE** column is the smallest change on that metric that reaches 80 % power against
another model — not against a fixed floor. `CM-x` is the commission (§10) that would turn a class-C
row into class A.

| # | Weakness | Benchmark | Base | Reference (measured) | MDE (vs a model) | Smallest real improvement | Stretch | Class |
|---|---|---|---|---|---|---|---|---|
| **W1** | Discrimination in the **thin-file** (low-limit) segment | Taiwan, lane protocol | UNMEASURED | AUC **0.7514** [0.7386, 0.7634] vs **0.7730** [0.7648, 0.7810] in the rest | **0.025** AUC | +0.025 AUC in segment | +0.04 | **A** |
| **W2** | Defaults missed at a **staffable decline rate** | Taiwan | UNMEASURED | 10 % decline catches **2,093 of 6,636** (31.5 %), misses **4,543** | **0.016** recall | +0.02 recall @ 10 % | +0.05 | **A** |
| **W3** | Calibration **in the upper band** | German + Taiwan | UNMEASURED | Taiwan max decile gap **3.3 pts**; German decile 8 **+8.1 pts** and decile 10 **−8.7 pts** on 100-row bins | **0.005** Brier | no gap > 3 pts on ≥ 1,000 rows | slope 1.0 ± 0.1 | **A** |
| **W4** | The score **rests on repayment status** | Taiwan | UNMEASURED | ablating `PAY_*` drops AUC **0.786 → 0.740** (−0.046) and recall@10 % **0.319 → 0.259** | 0.014 AUC | not trainable — scope the product | — | **A** (structural) |
| **W5** | **Missing-data** robustness | Polish bankruptcy (4.8 %, 41,322 missing) | UNMEASURED | none of the two headline corpora has a single null — the test does not exist yet | TBD | AUC loss < MDE at 10 % MCAR | < MDE at 30 % MCAR | **C** (`CM-R2`) |
| **W6** | **Explanation** fidelity (adverse action) | FICO HELOC xML | UNMEASURED | no measurement possible: licence unproven, protocol not reproduced | — | 3 faithful codes per decline | — | **C** (`CM-R3`) |
| **W7** | **Fair-lending** evidence | German + Taiwan | UNMEASURED (diagnostic only) | sex AIR **0.818** (passes 4/5ths); age AIR **0.747** (fails) at the 10 % cut | — | AIR ≥ 0.80 with reasons | ≥ 0.90 | **A** (diagnostic) |
| **W8** | **Out-of-time stability** | none held | UNMEASURED | neither corpus has a usable time axis: Taiwan is one 2005 window, German 1973–75 | — | PSI ≤ 0.10 | ≤ 0.05 | **C** (`CM-R5`) |
| **W9** | The base's **whole battery** | both | UNMEASURED | — | per Phase 0 §6 | clear G1–G4 | beat reference by MDE | **C** (`CM-R1`) |

**Read the table as a whole, not row by row.** Two rows (W1, W2) are measured weaknesses of the
*task* — places where a model that is good overall still loses the cases a lender cares about. Two
(W4, W8) are structural: no amount of training fixes a model whose signal is repayment status, and no
model can demonstrate stability on a corpus that has no time. Three (W5, W6, W8) cannot be measured on
what we hold. And one (W3) is a measured **non-weakness**: on 30,000 rows the reference's calibration
holds to within 3.3 points, which is the honest version of "we thought calibration would be bad".

---

## 4. The measured evidence

All figures are out-of-fold on the frozen Phase 0 splits; bootstrap 1,000 resamples, seed 20261111.
Receipts: [`phase1-risk/out/weakness.json`](phase1-risk/out/weakness.json),
[`intervals.json`](phase1-risk/out/intervals.json).

### 4.1 Where the defaults are missed

| Decline rate | Rows declined | Defaults caught | Defaults missed | Recall | Precision | Lift |
|---|---:|---:|---:|---:|---:|---:|
| 5 % | 1,500 | 1,136 | 5,500 | 0.171 | 0.757 | 3.42× |
| **10 %** | **3,000** | **2,093** | **4,543** | **0.315** | **0.698** | **3.15×** |
| 20 % | 6,000 | 3,386 | 3,250 | 0.510 | 0.564 | 2.55× |
| 30 % | 9,000 | 4,187 | 2,449 | 0.631 | 0.465 | 2.10× |

Taiwan, n = 30,000, 6,636 defaults. German is the same shape at a tenth of the size: 10 % decline
catches 80 of 300 (0.267), misses 220.

**Why this is a weakness and not a score.** At a 10 % decline the model catches roughly one default in
three. The other two are approved and then default. That is the operating reality of consumer credit —
but it is also the row where a point of recall is worth more than a point of AUC, because the lender
pays the loss, not the statistician. The interval is tight on Taiwan: **0.3154 [0.3080, 0.3229]**, SE 0.0039.

### 4.2 The segment where it is hardest

| Segment (credit limit) | n | defaults | default rate | AUC | Recall @ 10 % decline | Mean predicted |
|---|---:|---:|---:|---:|---:|---:|
| **≤ 50k** | 7,676 | 2,440 | 31.8 % | **0.7514** | **0.227** | 0.311 |
| 50k–140k | 7,614 | 1,882 | 24.7 % | 0.7764 | 0.299 | 0.246 |
| 140k–300k | 10,147 | 1,709 | 16.8 % | 0.7515 | 0.335 | 0.176 |
| > 300k | 4,563 | 605 | 13.3 % | 0.7701 | **0.388** | 0.140 |

The low-limit segment holds **37 % of all defaults** and the model is weakest exactly there: AUC
0.751 against 0.773 elsewhere, and within-segment recall of 0.227 against 0.388 in the highest band.
The two intervals do not overlap (0.7386–0.7634 vs 0.7648–0.7810), so this is a real gap and not a
binning artefact.

German shows the same pattern by age rather than limit: the 19–30 band scores AUC **0.736** against
**0.783** (31–45) and **0.788** (46+) — the young, thin-file applicant is the hard case in both
corpora, which is a property of credit data rather than of one dataset.

### 4.3 What the score actually rests on

| Feature removed (permuted) | AUC drop | SD |
|---|---:|---:|
| `PAY_0` (September repayment status) | **0.0779** | 0.0029 |
| `LIMIT_BAL` | 0.0224 | 0.0027 |
| `BILL_AMT1` | 0.0181 | 0.0011 |
| `PAY_AMT1` | 0.0090 | 0.0017 |
| `AGE` | 0.0011 | 0.0006 |

And the ablation, which is the finding that matters for product scope:

| Model | Features | AUC | PR-AUC | Recall @ 10 % |
|---|---:|---:|---:|---:|
| Full | 23 | **0.786** | 0.567 | 0.319 |
| **Without `PAY_*` repayment status** | 17 | **0.740** | 0.463 | **0.259** |

Removing seven repayment-status fields costs **0.046 AUC** and **6 points of recall**. The model is a
repayment-status model. That is not a defect to train away — it is a fact about which lender can use
it: one with bureau data, or one whose applicants have a repayment history. A lender underwriting
first-time borrowers is a different product, and this lane has no data for it.

### 4.4 Calibration by band — the measured non-weakness

Taiwan, tenths of the score distribution:

| Decile | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Predicted | 0.044 | 0.074 | 0.098 | 0.121 | 0.146 | 0.174 | 0.212 | 0.278 | 0.413 | 0.667 |
| Observed | 0.048 | 0.072 | 0.090 | 0.119 | 0.139 | 0.171 | 0.179 | 0.267 | 0.431 | 0.698 |
| Gap (pts) | −0.4 | +0.2 | +0.8 | +0.2 | +0.7 | +0.4 | **+3.3** | +1.1 | −1.8 | −3.1 |

No gap exceeds 3.3 points on bins of 3,000. On German the same table shows ±8 points, but on bins of
100 rows that is sampling noise, not miscalibration — which is exactly why the German calibration row
is reported with its interval and not as a finding.

### 4.5 The fairness diagnostic, with its limits stated

At a global 10 % decline cut, by sex: male 11.24 % declined (default rate 24.2 %), female 9.19 %
(20.8 %) → **disparate-impact ratio 0.818**, which clears the conventional 4/5ths screen. By age band:
31–45 8.84 %, 21–30 10.73 %, 46+ 11.84 % → **AIR 0.747**, which **fails** the same screen.

This is a diagnostic, not a verdict, and the packet says why: the decline rates track the base rates
(46+ defaults at 25.2 %, 31–45 at 20.9 %), the corpus is one Taiwanese card portfolio from 2005, and
the ratio is computed on a *policy* (decline the riskiest 10 %) rather than on a legally defined
protected class. It is enough to put fair-lending evidence on the Phase 2 list; it is not enough to
claim the model is or is not discriminatory.

---

## 5. The demand map — what a paying customer's bar actually is

A buyer of a credit-risk score does not buy AUC. They buy a **decision they can defend** to three
audiences at once: their credit committee, their regulator, and a declined applicant.

| Buyer | What they are buying | What they will actually test | What that maps to here |
|---|---|---|---|
| **Digital lender / BNPL** | a decision in milliseconds at application | AUC on their own holdout; recall at the decline rate they can afford; a latency budget | W1, W2, R6 |
| **Bank model risk (IRB-style)** | a PD they can put in a capital calculation | calibration slope and Brier; use test; validation evidence; documentation | W3, W8, §5.1 A3/A6 |
| **Risk operations** | an alert queue that is staffable | precision at the operating point; stability week to week | W2, W8 |
| **Compliance / legal** | a decline that survives an appeal | reason codes on every decline; fair-lending evidence; licence provenance | W6, W7 |
| **Platform / reseller** | something they can version and audit | reproducible scores, pinned revisions, provenance receipts | §5.1 A8 |

### 5.1 The bar, as acceptance criteria

Numbers below are **proposed thresholds**, assembled from four sources: regulatory requirement, industry
convention, the public leaderboard, and this lane's own MDEs. They are the owner's to confirm (D-R7),
and the *source class* column says how much weight each one carries.

| # | Criterion | Proposed threshold | Source class | Our proxy | State today |
|---|---|---|---|---|---|
| **A1** | Discrimination | ROC-AUC ≥ 0.75 on the buyer's own holdout | convention + MDE | Taiwan holdout, n = 6,000 | reference **0.785**; base UNMEASURED |
| **A2** | Operating point | recall @ 10 % decline ≥ 0.30 | our measurement | Taiwan OOF | reference **0.315** |
| **A3** | Calibration | Brier skill ≥ 0.10; slope 0.80–1.25; ECE ≤ 0.05 | IRB-style validation practice | Taiwan OOF | reference **0.217 / 1.06 / 0.012** |
| **A4** | Cost | ≤ 0.60 per applicant at 5:1 | our measurement | German + Taiwan | reference **0.582 / 0.556** |
| **A5** | Explanation | every decline carries 3 faithful reason codes | adverse-action law (Reg B, 12 CFR 1002.9) | HELOC xML | **UNMEASURED — no licence** |
| **A6** | Stability | PSI ≤ 0.10 out-of-time | model-risk guidance (SR 11-7) | none held | **UNMEASURED — no time axis** |
| **A7** | Latency / unit cost | p95 ≤ 100 ms; cost per 1,000 decisions stated | operations | harness | **UNMEASURED — no GPU run** |
| **A8** | Provenance | every training record licensed and traceable | this lane's register | `pack/README.md` | **measured — R-01…R-04** |
| **A9** | Fair lending | AIR ≥ 0.80 at the operating cut, with reasons | 4/5ths convention | Taiwan diagnostic | **0.818 (sex) · 0.747 (age)** |

### 5.2 The finding the demand map produces

**Six of the nine criteria can be measured with the corpora on disk. Two cannot be measured at all —
A5 (explanation) and A6 (stability) — and one is diagnostic only (A9).** A buyer in the first three
rows of the table will notice: no regulated lender signs for a PD with no stability evidence, and no
compliance function signs for a decline with no reason codes.

That is not a training problem. It is a **data problem**, and it belongs in Phase 2:

| Gap | What would close it | Where it stands |
|---|---|---|
| No out-of-time corpus | a loan-level set with issue dates and performance, licence-clear | Home Credit has no application date; Lending Club has `issue_d` but an unproven licence; Freddie Mac needs an account |
| No explanation benchmark | HELOC xML challenge, terms read and licence proven | blocked (`R-06`) |
| No thin-file population | a first-time-borrower or BNPL corpus, licence-clear | none identified |

**The honest consequence.** On the corpora we hold, this lane can build and prove a *discrimination and
calibration* product for a lender with bureau data. It cannot yet prove an *explanation or stability*
product for a regulated one — and those are the two things the lane's own Phase 0 said were the only
defensible reason to use a language model at all. The demand map and the weakness map point at the
same gap from opposite directions, which is the strongest signal either could give.

---

## 6. Where small models fail — the hypotheses, and the test that would prove each

The owner's question is where small models fail today. On this lane, the honest answer has two halves:
what is **known** (literature, class B), and what is **untested here** (class C, with a named test).
Nothing below is presented as our measurement.

### 6.1 What is known from public work

- **Trees still beat neural networks on medium-sized tabular data.** Grinsztajn, Oyallon & Varoquaux
  (NeurIPS 2022, *Why do tree-based models still outperform deep learning on typical tabular data?*)
  is the reference result, and it is why the bar for a 4 B decoder is a GBM and not another decoder.
- **Serialisation matters for language models on tables.** TabLLM (Hegselmann et al., AISTATS 2023)
  showed few-shot tabular classification with LLMs is feasible, and that the *text template* — not
  just the features — moves the result. That makes column order and number formatting a protocol
  variable on this lane, and Phase 0 froze them for exactly that reason.
- **Small-data tabular models are strong without training.** TabPFN v2 (Hollmann et al., *Nature*,
  January 2025) is the current demonstration, and it is why TabPFN sits in the Phase 0 shortlist as a
  *reference*, not a contender.

These are class B: someone else's measurement. They set the burden of proof. They do not settle it.

### 6.2 What is untested here — seven hypotheses with their tests

| # | Failure mode | Why it would bite on this lane | The test that proves it | State |
|---|---|---|---|---|
| **H1** | **Serialisation sensitivity** — score moves when columns are reordered or units rescaled | the only interface to a tabular row is text | permute field order and scale `BILL_AMT*` by 1/1000; AUC must move < MDE (0.014) | UNMEASURED (`CM-R6`) |
| **H2** | **Numeracy** — comparisons and thresholds on numbers | `PAY_0` and `LIMIT_BAL` are the whole signal; both are numeric | monotonicity probe: raising `PAY_0` (worse repayment) must not lower PD; count violations | UNMEASURED (`CM-R6`) |
| **H3** | **Miscalibrated token probabilities** | `P(default)` is a softmax over two tokens, not a fitted probability | gate **G2**: Brier, slope, intercept | UNMEASURED (`CM-R1`) |
| **H4** | **Imbalance collapse** — everything predicted to the majority | 4.8 % (Polish) to 30 % (German) base rates | recall @ 10 % decline, Brier, and the capture table | UNMEASURED (`CM-R1`) |
| **H5** | **Missing values** — `NA` handling | neither headline corpus has a null, so a base can look fine and fail in production | MCAR injection at 10 % and 30 % on Taiwan; AUC loss < MDE | UNMEASURED (`CM-R2`) |
| **H6** | **Unfaithful explanations** — plausible reasons that do not describe the score | the reason channel is the product's whole justification | HELOC protocol, or attribution-agreement diagnostic (never a score) | UNMEASURED (`CM-R3`) |
| **H7** | **Unit economics** — GPU cost per decision | a GBM scores in microseconds on CPU | measure rows/GPU-hour and cost per 1,000 decisions against the API price | UNMEASURED (`CM-R7`) |

**H7 deserves its sentence, and it is measured on one side.** The reference scores **443,842 rows/s**
— 2.25 µs per decision on one CPU core (`intervals.json`). The Phase 0 estimate for a 4 B model is
200–400 rows/s batched, so **1,100–2,200× slower per decision**, and an order of magnitude worse
single-stream. If the lane sells decisions, the model must be doing something the GBM cannot — and per
§5 that something is A5, which is unmeasurable. That is the commercial form of the same gap.

---

## 7. What not to train — the strengths, named before Phase 2

The Recipe's warning is specific: a mixture dominated by what the base already does well buys a few
points near a ceiling and is **maintenance, not mission**. Applied here, before any mixture exists:

| Do not build a mixture around… | Why |
|---|---|
| German Credit | 1,000 rows; every model is within 0.06 AUC of every other; it cannot resolve an improvement (§Phase 0 power table) |
| The majority class | at 22 % prevalence the "always good" baseline is already 78 % accurate and worth nothing |
| Taiwan's high-limit segment | AUC 0.770 and recall 0.388 — already the easy end; it is a **maintenance** row |
| Aggregate AUC | the reference gains most of its remaining error in the thin-file segment; aggregate AUC hides exactly that |

And the corollary: a row is **mission** only if it has a measured weakness behind it (W1, W2) or a
commission that will produce one (W5–W9). Everything else is maintenance and must be labelled so in
the Phase 5 mixture, with its share capped.

---

## 8. What Phase 1 hands to Phase 2

The weakness map is a shopping list only if it names the shape of the missing data:

| Weakness | Data shape that would address it | Track |
|---|---|---|
| W1 thin-file discrimination | first-time-borrower / BNPL applications with outcomes; alternative-data fields | A (public) or B (generated) |
| W2 recall at the operating point | **more defaults at the same prevalence**, not more rows — the segment is data-poor, not row-poor | A |
| W5 missing-data robustness | a corpus with documented missingness (Polish, 41,322 `?`) plus injected-missingness fixtures | A + fixture |
| W6 explanation | HELOC or equivalent with reason-code ground truth | A or B, licence-gated |
| W8 stability | any loan-level set with issue dates spanning ≥ 12 months | A |
| W3, W4, W7 | no data required — W3 is a gate, W4 is a scoping decision, W7 is a diagnostic | — |

**Coverage is judged against this table, not against row counts.** A 300k-row corpus that adds no
thin-file cases covers none of W1 or W2 and must be flagged as such in Phase 2, however large it is.

---

## 9. Decision rows

| ID | Decision | Options | Recommendation |
|---|---|---|---|
| **D-R7** | Confirm the acceptance bar (A1–A9 thresholds) | adopt as proposed · amend | Adopt A1–A4 and A8 now; A5/A6/A9 stay UNMEASURED until the data exists |
| **D-R8** | Is thin-file a target segment or out of scope? | in · out | **Out** until a thin-file corpus exists — the ablation (§4.3) says we cannot serve it honestly |
| **D-R9** | Accept "no stability evidence" as a known limitation of v1? | accept · block the product | Accept for an internal/analytical buyer; block for a regulated one |
| **D-R10** | Commission CM-R2/R5/R6 now or after the base run? | now · after CM-R1 | **After CM-R1** — the base's numbers decide whether robustness is worth measuring |

---

## 10. Commissions — the measurements that would end the UNMEASURED rows

| ID | What it measures | Depends on | Cost | Turns class |
|---|---|---|---|---|
| **CM-R1** | The base's whole battery (Phase 0 §10) | GPU access (B-5) | ~1 GPU-hour | C → A for W9, H3, H4 |
| **CM-R2** | Missing-data robustness, MCAR 10 % / 30 % | CM-R1 (a model to test) | < 1 GPU-hour | W5, H5 |
| **CM-R3** | Explanation fidelity on HELOC | licence (R-06) | 1–2 days | W6, H6 |
| **CM-R4** | Fair-lending formalisation (AIR with reasons, per segment) | CM-R1 | CPU only | W7 diagnostic → evidence |
| **CM-R5** | Out-of-time stability (PSI) | a dated corpus (Home Credit / Lending Club) | 1 day + licence | W8, A6 |
| **CM-R6** | Serialisation and numeracy probes (H1, H2) | CM-R1 | < 1 GPU-hour | H1, H2 |
| **CM-R7** | Unit economics (rows/GPU-hour, cost per 1k decisions) | CM-R1 | minutes | H7, A7 |

Six of the seven are cheap and blocked on the same thing: a base run. That is the same blocker as
Phase 0, which is the point — measurement debt compounds, and this packet prices it.

---

## 11. Blockers

| # | Blocker | Owner | Effect on this phase |
|---|---|---|---|
| **B-5** | No GPU access | Sandeep | W9, H3, H4, H7 stay UNMEASURED; five commissions queue behind it |
| **R-05** | Home Credit unaccepted | Sandeep | No stability corpus, no large-scale discrimination test |
| **R-06** | HELOC licence unproven | Sandeep | W6 and A5 — the product's justification — stay unmeasurable |
| **B-7** | No second agent, different family | Ronin | Phase 4 verification still unassigned |

---

## 12. Receipts and reproduction

| Artefact | What it proves |
|---|---|
| [`phase1-risk/out/weakness.json`](phase1-risk/out/weakness.json) | decile calibration, capture/lift, segment tables, permutation importance, both corpora |
| [`phase1-risk/out/intervals.json`](phase1-risk/out/intervals.json) | bootstrap SE/CI/MDE for recall@10 %, cost, segment AUC, the `PAY_*` ablation, and the fair-lending ratios |

```bash
cd credit
export PYTHONPATH=$PWD/.pylibs:$PWD/phase0-risk
python3 phase1-risk/weakness.py     # segments, capture, importance
python3 phase1-risk/intervals.py    # intervals, ablation, fair-lending diagnostic
```

Both scripts read only the frozen OOF files from Phase 0 and the curated corpora; neither trains a
language model, and neither produces a model score.

---

## 13. Recipe conformance

| Recipe clause | Where it is satisfied |
|---|---|
| Per benchmark: base score, loss mechanism, MDE, stretch | §3 — every row carries all four columns |
| No claimed weakness without a measured number or UNMEASURED + commission | §1, §3 — class A/B/C on every row; §10 names the commission |
| Train the weaknesses, not the strengths | §7 — maintenance rows named, caps deferred to Phase 5 |
| Coverage judged against the weakness map | §8 — the shopping list is shapes, not row counts |
| Competitor pass standard | §5.1 — acceptance criteria, with source class per row |
| UNMEASURED is a standing state | §3 (W5, W6, W8, W9), §5.2, §11 |

---

## 14. Revision history

| Rev | Date | Change |
|---|---|---|
| 1 | 2026-11-11 | First Phase 1 packet for Model 2. Nine weakness rows, five demand archetypes, nine acceptance criteria, seven commissions |

---

*No training was started. No corpus was built. No base score exists. The measured numbers in this
packet are properties of the task (segments, capture, ablation) and of the reference control; each has
a receipt, and each is labelled with how far it may be pushed.*
