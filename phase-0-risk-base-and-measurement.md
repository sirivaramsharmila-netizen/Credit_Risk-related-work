# Phase 0 — Base shortlist and measurement plan

**Lane:** CREDIT · **Model:** 2 — Credit Risk (built next, after the fraud packet)
**Revision:** 1 · **Date:** 11 November 2026 · **Author:** Sharmila Sirivaram
**Registration:** `CREDIT-RISK-P0-PREREG-001` — FROZEN 2026-11-11T14:44Z, SHA-256 `a5c0a46e…b29379`
**Harness:** [`phase0-risk/`](phase0-risk/README.md) — `split.py` · `metrics.py` · `controls.py` · `power.py` · `protocol.py` · `run_base.py`
**Receipts:** [`phase0-risk/out/`](phase0-risk/out/) · **HTML edition:** [`phase-0-risk-base-and-measurement.html`](phase-0-risk-base-and-measurement.html)
**Status:** controls and reference rows measured. **Base row UNMEASURED. No corpus. No training.**

> **The one sentence.** Fraud is a decision taken at transaction time; risk is a *probability* that
> must survive calibration and a *decline* that must survive an explanation — so this packet measures
> a different model on different metrics, and Phase 0 does not close until the base has one number
> on a public harness, read beside a calibrated reference, with the explanation duty either measured
> on a public benchmark or published as UNMEASURED.

---

## 0. Read this first

**What this packet is.** Phase 0 for Model 2, in the same lane and on the same machinery as the
fraud packet: the capability map fixed before data work, the public benchmarks that exist and the
ones that do not, the metrics that count for an imbalanced default prediction, the base shortlist
under the 5 B rule, and the frozen protocol the base will be scored on.

**What it is not.** It is not a corpus, not a weakness map, and not a score. No language model has
been run. Everything below that is a number is either a property of the *data* (prevalence, power,
baselines) or a property of the *reference model* (a gradient-boosted control), and both are
labelled. The one cell that matters — the base's calibrated PD — is empty and marked UNMEASURED.

**Why risk is a separate model.** The owner's instruction is explicit and the measurement agrees:
fraud is scored per transaction against a 0.17 % base rate and lives or dies on recall at a
staffable alert volume; risk is scored per application against a 4.8–30 % base rate and lives or dies
on a **probability** that a pricing and provisioning system can consume, plus a **reason** a
regulator will accept. Same lane, same provenance machinery, different job, different measures,
different model. Neither packet borrows the other's numbers.

**What already exists.** The fraud lane built the licence register, the provenance gate, the
decontamination engine and the split-by-family discipline; those carry over unchanged (§1). The
risk corpora are already acquired, hashed and licence-verified (`pack/`, 75,095 rows across four
corpora). German Credit's split is a **published OpenML fold file**, and this packet proves the join
to it row for row (§3). What does not exist is the base measurement, and it is blocked on compute.

**Five rules this packet does not bend.**

| # | Rule | What it forbids |
|---|---|---|
| 1 | **One protocol, frozen before the run** | Choosing the metric or the split after seeing which flatters |
| 2 | **Official harness where one exists; a named lane protocol where none does** | Calling a self-invented split "official", or hiding that it is not |
| 3 | **Accuracy is never a metric** | A 77.9 %-accurate model that catches nothing being reported as working |
| 4 | **Calibration is part of the measurement, not post-processing** | A ranking sold as a probability, or a calibration fit on the evaluation rows |
| 5 | **A number names its benchmark, harness and evaluation set** | A bare "0.78" that cannot be traced to a split |

---

## 1. What carries over from fraud, and what does not

The owner's instruction is that the lane's machinery is shared. It is. The *measurement* is not.

| Item | Fraud (Model 1) | Risk (Model 2) | Carried over? |
|---|---|---|---|
| Licence register | F-01…F-03, evidence quoted | R-01…R-04, same register format | **Yes — same register, new rows** |
| Provenance gate | per-record corpus name or subset label | identical rule | **Yes — unchanged** |
| Decontamination | word-order matcher, planted controls, tie-out | identical engine, identical fixtures | **Yes — unchanged** |
| Split discipline | temporal (ULB), entity-family (Sparkov) | published folds (German), stratified (Taiwan) | **Yes — same principle, different mechanism** |
| Power analysis | bootstrap on the frozen test split | bootstrap on the frozen OOF/holdout | **Yes — same method** |
| Trivial baselines | always-not-fraud, random, base rate | always-good, always-bad, base rate, random | **Yes — same discipline** |
| Headline metric | PR-AUC, recall @ 1 % FPR | **ROC-AUC, Brier, calibration slope, recall @ decline** | **No — different job** |
| The unmeasurable duty | adverse-action explanation (C5) | adverse-action explanation (R4) + fair lending (R5) | **No benchmark either way; both UNMEASURED** |
| Scoring path | two-token log-prob at `{"verdict": "` | two-token log-prob at `{"risk": "` | **Yes — one protocol family** |

The last row is deliberate. The fraud lane's hardest-won decision was to abandon generation-and-parse
for a single forward pass over two logits, because a parse failure silently became a score of 0.5 at
a 0.13 % base rate. Risk inherits that path: `P(default)` is a renormalised softmax over two tokens,
and a model that puts mass outside them is flagged by *coverage*, not silently rescaled.

---

## 2. Capability map — fixed before any data work

| # | Capability | Public benchmark | Harness / protocol | Metrics that count | Base | MDE |
|---|---|---|---|---|---|---|
| **R1** | Rank default risk correctly | German Credit (R-01), Taiwan (R-02) | OpenML task 31 (official) · lane protocol (Taiwan) | ROC-AUC, PR-AUC, KS | **UNMEASURED** | 0.014 AUC (Taiwan) · 0.063 (German) |
| **R2** | Emit a **calibrated** PD | same two corpora | same splits; calibration fitted in-fold | Brier, Brier skill, slope, intercept, ECE | **UNMEASURED** | slope ±0.2 · Brier 0.01 |
| **R3** | Hold an operating point | same two corpora | fixed decline rates, 5:1 cost matrix | recall @ 5/10/20 % decline, cost/applicant | **UNMEASURED** | ±0.05 recall |
| **R4** | Explain a decline (adverse action) | FICO HELOC xML challenge | official challenge protocol — **licence unproven** | reason-code agreement | **UNMEASURED (standing)** | — |
| **R5** | Fair lending / disparate impact | *none public* | — | AIR, subgroup AUC — **diagnostic only** | **UNMEASURED (standing)** | — |
| **R6** | Decision latency / cost per score | *none public* | — | ms/row, rows/GPU-hour | **UNMEASURED (standing)** | — |

**R4 is the load-bearing row.** It is the one capability a gradient-boosted scorecard cannot have,
and therefore the only defensible reason for a language model in this lane. It also has no public
benchmark this lane may use: the HELOC corpus's licence is recorded as *unverified* in the pack
register (a Hugging Face mirror carries a bare `cc`, which is not a licence). Under the Recipe a
number from it is not a number, so R4 publishes as UNMEASURED — and §9 says exactly what that costs.

R5 and R6 are diagnostics. A disparate-impact ratio or a latency figure can be computed, but there is
no public benchmark and the Recipe forbids a home-grown number from being quoted as a model score.

---

## 3. Benchmarks and protocols — the five dials

A benchmark is not a dataset. It is a machine with five dials, and changing one produces a number
that belongs to a different benchmark. The fraud lane lost two cycles to protocol failures (GPQA as
exact-answer; BFCL on a home-grown checker). For risk, the dials are these.

| # | Dial | Setting for this lane | State |
|---|---|---|---|
| 1 | **Corpus and split** | German: OpenML task 31 published folds. Taiwan: frozen stratified holdout | **Pass** — hashes in `out/split_report.json` |
| 2 | **Weights** | `Qwen/Qwen3-4B-Instruct-2507`, bf16, **pinned Hub revision** | **Open** — revision not yet recorded |
| 3 | **Prompt** | `credit-risk-score-protocol/1.0`, template SHA-256 `b2e9ac59…d3d77` | **Pass** — frozen in `protocol.py` |
| 4 | **Scoring path** | logits at `{"risk": "` over `" default"` / `" no_default"` | **Pass** — one path, no generation |
| 5 | **Metric implementation** | `phase0-risk/metrics.py`, validated vs scikit-learn to 1e-9 | **Pass** — `out/metrics_validation.json` |

### 3.1 German Credit — the official harness, and the join that had to be proven

OpenML **task 31** publishes a fold-assignment file for the 1,000-row German corpus: 10 folds,
100 test rows each, and the file is the harness — not a re-implementation of 10-fold CV that might
differ in seed or stratification.

Two joins had to be proven before the folds could be used:

1. **Row order.** OpenML's own ARFF for dataset 31 was compared to the curated parquet: every one of
   the seven numeric columns and the class agree **row for row, 0 mismatches**. The curated file
   preserves publisher order, and the publisher's order is OpenML's order.
2. **Index base.** The split file's `rowid` is **0-based**. Joined as 0-based, the folds are
   class-stratified at exactly **30 bad in every 100-row fold** — the publisher's stratification
   reproduced exactly. Joined as 1-based, every row shifts by one and the positives scatter over
   22–36 per fold. The exact 30/30 is independent evidence that the join is right.

Both proofs are in `out/split_report.json`. A failed proof makes `split.py` refuse to write folds.

**What German is good for:** protocol conformance, a published cost metric, and a corpus small enough
to iterate on. **What it is not good for:** a model-comparison verdict (§6 — at n = 1,000 and 300
bad, a 0.02 AUC gap is detected with 14 % power).

### 3.2 Taiwan default — no official harness, so a named lane protocol

OpenML carries dataset 42477 but only a **clustering** task (254211), verified against the API. There
is no publisher split and no supervised task, so there is no official harness to run. Pretending
otherwise would be the exact defect the Recipe names.

The lane therefore pre-registers one: stratified **5-fold × 2 repeats**, seed 20261111, plus a frozen
**80/20 holdout** (n = 6,000, 1,327 defaults, prevalence 22.1167 %). Fold and holdout hashes are in
the registration. Every number from it is labelled **lane protocol — not an official harness**, and
no table quotes it without that label.

This is the same position the fraud lane holds on ruling R2 (is a reconstructed FDB split
"official"?), and it is Ronin's call, not this packet's. The difference is that here the answer is
already known to be *no official harness exists*, so the honest fallback is a frozen protocol, not a
reconstruction claim.

### 3.3 The benchmarks that exist and cannot be run

| Benchmark | What it would settle | Why it cannot be run today | Changeable? |
|---|---|---|---|
| **Home Credit Default Risk** (307k applications) | The one public risk benchmark at scale; R1–R3 with a 0.001-class MDE | Kaggle competition: data and test labels behind accepted rules; anonymous API returns **401** (verified 2026-11-04). Test labels are never released — a local score requires a submission | **Yes** — one accepted account |
| **FICO HELOC xML challenge** | R4, the explanation duty | Licence **unproven**: the HF mirror declares bare `cc`, which is not a licence; the FICO challenge terms were not read | **Yes** — read the terms |
| **Lending Club** | Large real consumer-loan performance set | Two mirrors disagree on the upstream licence (CC0 vs ODbL); a re-uploader cannot establish upstream rights | Only via LendingClub's own grant |
| **Polish bankruptcy** (43,405 rows) | Corporate distress, 4.8 % | Not consumer PD; five forecast horizons pooled, so rows are not independent companies. Usable only as a shift check after grouping | Yes, but off-mission |
| **Credit Approval** (690 rows) | A hard tabular benchmark | 55 % "rejected" is 1987 lending policy, not a default rate; semantically opaque | No — wrong target |

These five refusals matter more than they look. With German (1,000 rows) and Taiwan (30,000 rows),
this lane has **two** usable predictions of default and **zero** usable benchmarks of explanation.
That is the honest inventory, and it shapes every decision row in §11.

---

## 4. Metrics that count — and the accuracy trap, measured

The owner's instruction is that accuracy alone is not a metric. On this lane it is worse than
uninformative; it is *inverted*. Both corpora reward the model that predicts nothing.

| Corpus | Predict "no default" every time | Accuracy | ROC-AUC | Defaults caught (of all) | Brier |
|---|---|---:|---:|---:|---:|
| German (30.0 % bad) | all good | **70.0 %** | 0.500 | **0 of 300** | 0.300 |
| Taiwan (22.12 % default) | all good | **77.9 %** | 0.500 | **0 of 6,636** | 0.221 |

A 77.9 %-accurate model with zero skill is the single most quotable wrong number available on this
lane. So the battery is fixed to three kinds of number, none of which is accuracy:

1. **Rank metrics, threshold-free** — ROC-AUC (primary; it is the industry statistic and the metric
   the one large public risk benchmark is scored on) and PR-AUC (co-primary, because prevalence
   ranges from 4.8 % to 30 % across this lane's corpora and AUC hides that).
2. **One decision-relevant operating point** — recall at a fixed **decline rate** (5 / 10 / 20 %),
   because that is the decision a lender actually makes, plus expected cost per applicant under the
   publisher's **5:1** cost matrix (german.doc §8).
3. **One calibration number** — Brier against the base-rate Brier, plus the slope and intercept of
   `y ~ sigmoid(a + b·logit(p))`, plus ECE (equal-width and equal-count, because 20 % prevalence
   against 0.2 % fraud prevalence breaks equal-width bins).

### 4.1 Trivial baselines — computed, not asserted

Out-of-fold / holdout, same engine as every model row. "Cost" is expected cost per applicant at the
5:1 Bayes cut (approve a bad = 5, decline a good = 1).

**German Credit — official folds, n = 1,000, 300 bad**

| Row | ROC-AUC | PR-AUC | KS | Brier | Cost | Recall @ 10 % decline |
|---|---:|---:|---:|---:|---:|---:|
| Always good | 0.500 | 0.300 | 0.000 | 0.300 | 1.500 | 0.100 |
| Always bad | 0.500 | 0.300 | 0.000 | 0.700 | 0.700 | 0.100 |
| Base-rate constant | 0.500 | 0.300 | 0.000 | 0.210 | 0.700 | 0.100 |
| Random uniform | 0.458 | 0.277 | 0.006 | 0.348 | 0.891 | 0.093 |
| **Reference (HGB)** | **0.771** | **0.622** | **0.401** | **0.180** | **0.673** | **0.267** |

**Taiwan — frozen holdout / OOF, n = 30,000, 6,636 defaults**

| Row | ROC-AUC | PR-AUC | KS | Brier | Cost | Recall @ 10 % decline |
|---|---:|---:|---:|---:|---:|---:|
| Always good | 0.500 | 0.221 | 0.000 | 0.221 | 1.106 | 0.100 |
| Always bad | 0.500 | 0.221 | 0.000 | 0.779 | 0.779 | 0.100 |
| Base-rate constant | 0.500 | 0.221 | 0.000 | 0.172 | 0.779 | 0.100 |
| Random uniform | 0.503 | 0.222 | 0.014 | 0.335 | 0.829 | 0.101 |
| **Reference (HGB)** | **0.779** | **0.556** | **0.428** | **0.135** | **0.556** | **0.316** |

Two things to read off these tables. First, **AP equals prevalence for every constant scorer** — the
tie-grouping defect that once made a constant fraud scorer report 0.0016 against a 0.001317 base rate
is fixed and proven in `out/metrics_validation.json`. Second, the **cost metric is compressed**: on
German the reference beats the best trivial policy by 0.027 per applicant, which is inside the noise
of 1,000 rows. Cost is reported; it is not a headline.

### 4.2 Controls that prove the harness, not the model

| Control | Expected | Observed (German / Taiwan) | Verdict |
|---|---|---|---|
| Leak probe — label as a feature | AUC ≈ 1.0 | **1.000 / 1.000** | Harness can see leakage |
| Null probe — labels permuted | AUC ≈ 0.5 | **0.492 / 0.491** | Harness invents no signal |
| Column audit — max \|corr\| with label | < 0.99 | **0.351 / 0.325**, no flags | No duplicate label column |
| Constant scorer | KS = 0, AP = prevalence | **0.000 / 0.221** | Tie handling correct |

The column audit is not ceremony: **`default_next_month` in the Taiwan pack restates the label**. It
is excluded by name in `protocol.py` and `controls.py`, and the probe proves the exclusion would be
caught if it were ever dropped from the exclusion list.

---

## 5. Calibration — the duty that separates risk from fraud

A fraud score can be a ranking: the alert queue is sorted, and nobody prices a transaction from it.
A default probability is **used as a number** — it sets pricing, provisioning and capital. A model
that ranks well but reports 0.40 when the truth is 0.20 is not a risk model, it is a ranking with a
number attached.

The reference model demonstrates the failure, on the official harness, today:

| German, official OOF | ROC-AUC | Brier | Brier skill | ECE | Calibration slope | Intercept | Cost |
|---|---:|---:|---:|---:|---:|---:|---:|
| Reference, raw output | 0.771 | 0.180 | 0.143 | 0.118 | **0.409** | −0.232 | 0.673 |
| Reference, Platt-recalibrated | 0.769 | **0.166** | **0.209** | **0.033** | **1.037** | 0.024 | **0.582** |

A slope of **0.41** means the raw probabilities are roughly 2.4× too confident in their spread; ECE
0.118 means an 11.8-point average gap between predicted and observed risk. Recall is *unchanged*
(0.267 both ways) because Platt is monotone — the ranking was never the problem, and only a
calibration metric sees the defect. After recalibration the slope is 1.04 and the average cost falls
from 0.673 to 0.582 per applicant: **13.5 % of the operating cost was a calibration error, not a
discrimination error.**

Taiwan tells the milder version: raw slope 0.900, Brier 0.1348 against base-rate 0.1723, ECE 0.0143 —
already close to calibrated, and Platt moves it to slope 1.062 without improving Brier. The lesson is
that calibration quality is corpus-dependent and cannot be assumed in either direction.

**The protocol, frozen:**

1. Platt scaling is fitted on **inner out-of-fold** scores inside the training folds only.
2. It is applied to the held-out fold, never refitted on it (V4 voids the run).
3. Both raw and recalibrated numbers are published side by side. Reporting only the recalibrated
   number hides a model defect; reporting only the raw number hides a usable PD.
4. Gate **G2**: Brier ≤ base-rate Brier − 0.01 **and** slope ∈ [0.80, 1.25] **and** |intercept| ≤ 0.10.
   A model that fails G2 may still be reported for R1 (ranking) — but it is **not** a PD and the
   verdict says so.

---

## 6. Power — what each benchmark can decide

Bootstrap of the frozen out-of-fold reference predictions, 1,000 resamples, seed 20261111. Full
receipt in `out/power.json`.

| Evaluation set | n | defaults | SE(AUC) | 95 % CI half-width | MDE vs a model (80 %) | power of a 0.02 gap |
|---|---:|---:|---:|---:|---:|---:|
| Taiwan, pooled OOF | 30,000 | 6,636 | 0.0035 | ±0.007 | **0.014** | **0.981** |
| Taiwan, frozen holdout | 6,000 | 1,327 | 0.0075 | ±0.015 | **0.030** | 0.470 |
| German, pooled OOF | 1,000 | 300 | 0.0159 | ±0.030 | **0.063** | 0.142 |
| German, single fold | 100 | 30 | 0.0417 | ±0.082 | **0.165** | 0.053 |

**Read it as three sentences.**

1. **Taiwan can decide a 0.02 AUC difference** (power 0.98). That is the only evaluation set on this
   lane that can support "the base beats the reference" or "it does not".
2. **German cannot** (power 0.14). Its official harness is the better-established instrument and the
   weaker measuring one; a verdict read off German is a coin flip with a citation.
3. **A single fold is never quoted.** Fold 0 of German scores AUC **0.844** against **0.771** pooled —
   the most flattering cell in the battery, and an accident of 30 positives in 100 rows. Per-fold
   spread is 0.696–0.844 (SD 0.047) on German against 0.767–0.788 (SD 0.008) on Taiwan.

This is the fraud lane's lesson applied before the mistake is made: a gate read as a *model
comparison* at low power is not a test. The fraud lane discovered that a +0.10 PR-AUC gate had 36 %
power; here the number is written down **before** the run, so no future verdict can quietly read the
floor as a win.

---

## 7. The base shortlist — small models, under 5 B

The 5 B rule is the programme's. Every parameter count and licence for the Hub models below was read
from the Hugging Face API on **2026-11-11**, not from a paper or a blog.

| Role | Model | Params | Licence | Gated | Verdict |
|---|---|---:|---|---|---|
| **Base** | `Qwen/Qwen3-4B-Instruct-2507` | 4,022,468,096 | Apache-2.0 | No | **Adopted** |
| Second-family verifier | `microsoft/Phi-4-mini-instruct` | 3,836,021,760 | MIT | No | **Adopted for Phase 4** |
| Throughput fallback | `Qwen/Qwen3-1.7B` | 2,031,739,904 | Apache-2.0 | No | Shortlisted — a documented departure if used |
| Alternative base | `HuggingFaceTB/SmolLM3-3B` | 3,075,098,624 | Apache-2.0 | No | Shortlisted, not adopted |
| Text encoder | `answerdotai/ModernBERT-base` | 149,655,232 | Apache-2.0 | No | **Not applicable** — neither risk corpus has a text field |
| Tabular foundation model | `Prior-Labs/TabPFN-v2-clf` | not published in Hub metadata | Prior Labs 1.1 (Apache-2.0 + attribution) | No | **Reference control only** |
| Competitor-class reference | HistGradientBoosting (scikit-learn) | — | BSD-3 | — | **Reference row** (measured) |
| Excluded | `meta-llama/Llama-3.2-3B-Instruct` | 3,212,749,824 | Llama 3.2 community | **manual** | No measured upside; gated + non-OSI adds legal review |
| Excluded | `google/gemma-3-4b-it` | 4,300,079,472 | Gemma terms | **manual** | Same, plus custom use restrictions |

### 7.1 Why Qwen3-4B, and what would overturn it

Three reasons, in order of weight:

1. **One stack.** The definitive report §10.4 forbids re-deciding the base per training. The fraud
   lane already measured and adopted Qwen3-4B; a second base for the second model would create two
   stacks, two tokenizer behaviours, two sets of prompt hashes, and twice the harness surface.
2. **It fits the job's scoring path.** A decoder that gives a clean two-token distribution at a fixed
   prefix is what R1–R3 need; the fallback 1.7 B is the same architecture and protocol, so a swap
   would be a documented departure, not a redesign.
3. **Licence and size clear both gates** — Apache-2.0, ungated, 4.02 B.

**Falsification, written before the run.** If the base's calibrated Brier on the German official row
does not beat the base-rate Brier (0.210), it is not a probability model at all, and no amount of
post-hoc calibration rescues a model whose ranking is noise. If it beats the base-rate Brier but
fails G2's slope band, the base is a ranker and the *risk product* cannot be the language model — the
scorer stays the GBM and the model's job becomes explanation only. That is decision row **D-R2**, and
it is the owner's, not this packet's.

**On TabPFN.** It is a genuinely small tabular foundation model and would be a fair *reference* on
both corpora — small-data accuracy is its design point, and both corpora are small. It is not
adopted as the trained model: it is a predictor, not a language model, it cannot emit a reason code,
and its licence (Apache-2.0 **plus an attribution requirement**) has not been cleared for shipping.
It belongs in the reference row set beside the GBM once compute exists, and the packet says so rather
than quietly omitting it.

### 7.2 One base, two jobs — the shortlist across the lane

The owner's rule is that each model gets its own packet, not that each model gets its own stack.

| Job | Scored when | Base | Headline metric | Second metric | What the base must not do |
|---|---|---|---|---|---|
| **Credit Fraud** (Model 1) | per transaction, 0.17 % base rate | `Qwen3-4B-Instruct-2507` | PR-AUC, recall @ 1 % FPR | ROC-AUC | Miss fraud at a staffable alert volume |
| **Credit Risk** (Model 2) | per application, 4.8–30 % base rate | `Qwen3-4B-Instruct-2507` | ROC-AUC | Brier + calibration slope | Report a probability that is not one |

Same weights, same scoring path family, different decision rule and different measures. That is the
whole point of the split: the model is shared to keep one stack, the *measurement* is not shared
because the jobs are not the same job. If risk's calibration gate (G2) fails, the risk model changes
role — ranker and explainer, with the GBM scoring — and the stack does not change (D-R2).

The only size move that stays inside the protocol is `Qwen3-1.7B`, which keeps the architecture and
the prompt format and halves the inference cost. It is shortlisted rather than adopted because the
lane's compute constraint is one 4 B model, not two, and a 1.7 B base would need its own measured row
to justify the swap — which is exactly the kind of unmeasured change this packet exists to prevent.

---

## 8. What the base must beat

The reference is a gradient-boosted model on the same folds with the same engine the fraud lane used
(`HistGradientBoostingClassifier`, seed 20261111, 300 iterations). It is a **control**, not a
contender: it is the bar the 4 B model must clear to be worth its inference cost, and the row that
makes "the model learned something" a measurable claim rather than a feeling.

| Row | Protocol | ROC-AUC | PR-AUC | KS | Brier (calibrated) | Recall @ 10 % decline |
|---|---|---:|---:|---:|---:|---:|
| Reference, German | **official** OpenML task 31 | 0.771 | 0.622 | 0.401 | 0.166 | 0.267 |
| Reference, Taiwan | lane protocol, OOF | 0.779 | 0.556 | 0.428 | 0.135 | 0.316 |
| Reference, Taiwan | lane protocol, holdout | 0.785 | 0.567 | 0.434 | — | — |
| **Base, German** | official | **UNMEASURED** | **UNMEASURED** | **UNMEASURED** | **UNMEASURED** | **UNMEASURED** |
| **Base, Taiwan** | lane protocol | **UNMEASURED** | **UNMEASURED** | **UNMEASURED** | **UNMEASURED** | **UNMEASURED** |

**Author-reported competitor figures** — the Home Credit leaderboard's top solutions are around 0.80
ROC-AUC. This packet has **not verified** that figure: the competition's evaluation page did not
render for an anonymous fetch and the API returns 401. It is recorded as an unverified bar to
reproduce later, never as a number this lane quotes as ours.

---

## 9. The explanation duty — the capability that decides whether this model has a job

Adverse-action explanation is not a nice-to-have in credit; it is the legal condition of declining an
application. The Recipe's rule is brutal and correct: *home-grown controls belong to the DATA side
only — they never produce a model score.* A SHAP-agreement diagnostic is a model-side home-grown
check. So it is recorded as a **diagnostic**, excluded from every score table, and the capability
publishes as UNMEASURED.

What would measure it: the **FICO HELOC xML challenge**, the one public benchmark built for exactly
this question — reason codes on a credit-decline task. It cannot be used today, for two independent
reasons:

1. **Licence.** The corpus's licence is recorded as unverified in the pack register; the HF mirror's
   bare `cc` is not a licence, and the FICO challenge terms have not been read. The Recipe's Phase 4
   rule is absolute: a record whose licence cannot be proven is dropped before the packet, never
   discovered mid-run.
2. **Protocol.** Even with a licence, the challenge's official scoring path is the challenge's; an
   approximation of it is a variant protocol, and a variant protocol's number is discarded.

**The consequence, stated plainly:** if explanation is the product — and on this lane it is the only
capability a GBM cannot have — then the Recipe cannot currently certify the product. That is not a
reason to fake a number; it is a reason to put the licence question to the owner as a decision row
(**D-R2/D-R4**) and to keep measuring R1–R3 in the meantime.

One thing is measurable without the benchmark and is worth stating: **the reason channel and the
verdict channel share one record format** (`{"risk": " default", "reasons": […]}`), so a model trained
on reasons cannot drift from the format its verdict is scored in. That is the same format-match
discipline the fraud lane learned the hard way, applied before the training run rather than after.

---

## 10. How the base will be measured — the run

Fixed before any data work, in this order. **No step is optional and no step may be reordered.**

| # | Step | Gate | Cost |
|---|---|---|---|
| 0 | `validate_metrics.py` → PASS | metric engine parity < 1e-9 | seconds |
| 1 | `split.py` → join proof + fold hashes | refuses on a failed OpenML join | seconds |
| 2 | `protocol.py --check` → schema match | refuses if the pack gains or loses a column | seconds |
| 3 | **G0 tokenizer check** — both verdict tokens are one token | **blocks the run** | minutes |
| 4 | Pin the Hub revision; record it in the receipt | G5 identity | minutes |
| 5 | Score German, all 1,000 rows, one forward pass each | coverage ≥ 0.98 | ~1 GPU-minute |
| 6 | Score Taiwan holdout, 6,000 rows | coverage ≥ 0.98 | ~5–15 GPU-minutes |
| 7 | Determinism: 200 rows scored twice, identical | V3 | minutes |
| 8 | `evaluate.py` → R1–R3 numbers, raw and calibrated | G1–G4 | seconds |
| 9 | Publish with the loss mechanisms and UNMEASURED rows intact | — | — |

**GPU budget, written before the stage.** One 4 B model at bf16, batched: **20–40 rows/s** single
stream, **200–400 rows/s** batched, on one A100-class GPU. The whole battery is 7,000 scored rows —
**under 0.1 GPU-hour**, call it 1 GPU-hour with queueing and a re-run. This is an *estimate*, and the
Recipe requires the estimate before the stage: **stop condition — if measured batched throughput is
below 50 rows/s, stop and report the throughput rather than burning GPU-days.** A full OOF re-scoring
of Taiwan (30,000 rows × 2 repeats) would be ~1–2 GPU-hours and is not needed for Phase 0.

**What voids the run:** a split or prompt hash mismatch (V2); a determinism failure (V3); calibration
fitted on evaluation rows (V4); a coverage below 0.98 (G5); a base that fails G1 (no measurable
skill — the run is not void, the verdict is "no skill").

**What the run does not do:** it does not choose the base (D-R1), does not start training, does not
touch the corpus, and does not settle whether explanation is the product (D-R2).

---

## 11. Decision rows — the owner's, not this packet's

| ID | Decision | Options | What it changes | Recommendation |
|---|---|---|---|---|
| **D-R1** | Confirm the base | Qwen3-4B (one stack) · Qwen3-1.7B (cheaper) · SmolLM3-3B | Which model gets the GPU hours | **Qwen3-4B** — one stack, already the fraud base |
| **D-R2** | Is explanation the product? | PD scorer · PD + reasons · reasons only | Whether R4's UNMEASURED state is gating | Decide **before** training, not after |
| **D-R3** | Home Credit access | Accept Kaggle rules on one account | Unlocks the only large public risk benchmark | Do it — it is the difference between a 0.014 and a ~0.001 MDE |
| **D-R4** | HELOC licence | Read the FICO challenge terms | Unlocks R4, the only capability that justifies an LLM | Read them before any explanation claim is made |
| **D-R5** | Confirm G2/G4 thresholds | As proposed in the registration | Whether a ratio-only model can pass | Confirm or amend **before** the run |
| **D-R6** | Benchmark primacy | German (official) · Taiwan (decisive) · both, named | Which number a verdict quotes | Both, named — never one without the other |

---

## 12. Blockers

| # | Blocker | Owner | Effect |
|---|---|---|---|
| **B-5** | No GPU/compute access | Sandeep | The base row cannot exist; everything above it is staging |
| **B-6** | `FAILURE-CORPUS.md` (47 rules) not in this workspace | Ronin | Required pre-flight input still missing |
| **B-7** | No second agent from a different model family | Ronin | Phase 4 gate unmet — naming Phi-4-mini as the *model* does not supply the *agent* |
| **B-9** | Corpora from UCI/OpenML, not Hugging Face | Ronin | Framework download rule; model downloads comply |
| **R-05** | Home Credit licence/rules unaccepted | Sandeep | The 307k-row benchmark is UNMEASURED |
| **R-06** | HELOC licence unproven | Sandeep | R4, the explanation duty, is UNMEASURED |

None of these is a reason to build a corpus first. The Recipe's gate is explicit: measurement debt
blocks data work, it does not trail it.

---

## 13. Receipts and reproduction

| Artefact | What it proves |
|---|---|
| [`phase0-risk/out/metrics_validation.json`](phase0-risk/out/metrics_validation.json) | AUC/AP/KS/Brier parity vs scikit-learn **1.1e-16**; calibration vs an independent MLE **5.7e-7**; constant scorer KS 0, AP = prevalence |
| [`phase0-risk/out/split_report.json`](phase0-risk/out/split_report.json) | OpenML task-31 join (0 mismatches, 30/100 stratification), fold hashes, Taiwan holdout counts |
| [`phase0-risk/out/controls.json`](phase0-risk/out/controls.json) | Trivial rows, reference rows raw and calibrated, leak probe 1.000, null probe 0.492/0.491, column audit |
| [`phase0-risk/out/power.json`](phase0-risk/out/power.json) | SE, CI and MDE per evaluation set; power of a 0.02 AUC gap |
| [`phase0-risk/out/protocol.json`](phase0-risk/out/protocol.json) | Schema match, template/prefix hashes, rendered fixtures, tokenizer check PENDING |
| [`phase0-risk/out/oof_german.csv`](phase0-risk/out/oof_german.csv) · [`oof_taiwan.csv`](phase0-risk/out/oof_taiwan.csv) | Per-row reference scores for the paired comparison when the base exists |
| [`phase0-risk/preregistration.json`](phase0-risk/preregistration.json) | The frozen registration, machine-readable |

Reproduce the whole packet's numbers in about three minutes on a laptop, no GPU:

```bash
cd credit/phase0-risk
export PYTHONPATH=../.pylibs
python3 validate_metrics.py && python3 split.py && python3 controls.py && python3 power.py && python3 protocol.py --check
```

The mock base run exists to prove the plumbing (`run_base.py --corpus german --mock`); its output is
stamped **SHAPE TEST — NOT A SCORE** and no number from it appears in this packet.

---

## 14. Recipe conformance

| Recipe clause | Where it is satisfied |
|---|---|
| Capability map before data work | §2 — six rows, each with a named benchmark or an explicit "none public" |
| Public benchmarks on official harnesses only | §3 — OpenML task 31 is the official harness; Taiwan is *named* as non-official and never called a score |
| A variant protocol's number is discarded | §3.2, §9 — no approximate challenge scoring, no home-grown explanation metric quoted |
| Trivial baselines, computed | §4.1 — five rows per corpus, including the 77.9 %-accurate null |
| Power / MDE measured, not assumed | §6 — bootstrap, 1,000 resamples, per evaluation set |
| UNMEASURED is a standing state | §2 (R4–R6), §9, §12 |
| Licence and provenance per record | §1, pack register R-01…R-04; Home Credit and HELOC refused for licence reasons |
| Calibration is part of the measurement | §5 — in-fold fit, raw and recalibrated published side by side, gate G2 |
| Decontaminate before splitting | Inherited from the fraud engine; splits are folds of already-clean corpora, family/horizon rules from `pack/README` §7 apply at corpus build |
| Training start is the owner's tap | §10, §11 — no training is started, no pilot is pre-approved |
| Format match (train = measure) | §9, `protocol.py` — one record format, verdict field first, one hash |

**Departures, named in writing.** (1) The lane has no official harness for its decisive benchmark,
so it pre-registers one and labels every number from it — a departure from "official harness only",
forced by the absence of an official harness, recorded rather than hidden. (2) Calibration is
measured as a first-class capability rather than a diagnostic, because the job requires a probability
— this is stricter than the fraud lane, not looser.

---

## 15. Revision history

| Rev | Date | Change |
|---|---|---|
| 1 | 2026-11-11 | First risk packet. Base shortlist, capability map, metrics, calibration duty, power, frozen registration, harness with measured controls |

---

*No training was started. No corpus was built. No model score was produced. The only numbers in this
packet are data properties and reference-model controls, each with a receipt.*
