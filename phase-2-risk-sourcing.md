# Phase 2 — Sourcing, the licence register and per-record provenance

**Lane:** CREDIT · **Model:** 2 — Credit Risk · **Phase:** 2 of 6
**Revision:** 1 · **Date:** 11 November 2026 · **Author:** Sharmila Sirivaram
**Gate:** per-record provenance (Recipe v1, Phase 2) · **Register:** [`phase2-risk/register.json`](phase2-risk/register.json)
**Evidence:** [`phase2-risk/out/gate_report.json`](phase2-risk/out/gate_report.json) · [`slices_polish.json`](phase2-risk/out/slices_polish.json) · [`candidate_laotse.json`](phase2-risk/out/candidate_laotse.json)
**Status:** 11 sources in the register. **5 admitted, 2 pending terms, 3 rejected, 1 out of scope.** No corpus built.

> **The one sentence.** Phase 2 is where a corpus becomes admissible, and for this model the
> arithmetic comes out badly: the two corpora that can measure a verdict are the two that must never
> train, the 43,405-row training corpus narrows to 3,194 rows and 30 bankruptcies once sliced, and the
> one clean new admission is synthetic — so the honest output of this phase is a **training gap**,
> stated with numbers, not a corpus.

---

## 0. What this packet covers, and the owner's instruction applied to Model 2

The owner's Phase 2 instruction is written for the fraud model: *public fraud datasets only, each
with a licence entry in a register and per-record provenance; published datasets are a permanently
flagged class — labelled as dataset rows, never described as anything else.* The same rule applies here
with the domain substituted — **public credit-risk datasets only** — and this packet keeps every part
of it:

| Instruction | How it is applied here |
|---|---|
| public datasets only | every source is a publisher-hosted corpus; mirrors are a named rejection class (§5) |
| licence entry in a register | 11 rows, each with the evidence line and the date it was read (§3) |
| per-record provenance | a frozen stamp, applied at ingest, one row per record (§6) |
| published datasets are permanently flagged | `origin_class = DATASET_ROW` on every record, forever; synthetic ones additionally carry `synthetic = true` and are never described as real applications (§7) |

**What Phase 2 is not.** It is not the corpus. Nothing here has been ingested, split, decontaminated or
trained on. Those are Phase 3, and they are gated on this phase's verdicts — which is why the negative
result below matters more than the admissions.

---

## 1. The gate, in three predicates

> A record ships only when **its own per-record data names the upstream corpus whose content it is**,
> and the licence is that corpus's licence. Dataset-level licence, mixture-card licence and disclosed
> generation method do not count. Per-record subset/category labels **do** count as origin.

| Predicate | Question | Executed by |
|---|---|---|
| **P1 origin** | can every record name its corpus, with a stamp applied at ingest rather than added later? | `gate.py` — re-hashes the curated file, checks row counts, unique record ids and that a stamp is buildable |
| **P2 licence** | does that corpus permit commercial training, with duties recorded? | register evidence line, read at the publisher, dated |
| **P3 coverage** | does it cover at least one weakness row from Phase 1? | register `coverage` field, checked against W1–W9 |

Verdicts are fixed: `ADMITTED` · `REJECTED` · `PENDING TERMS` · `OUT OF SCOPE`. **Pending terms is
never admissible** — an auth-gated licence that has not been read is an unknown, not a maybe.

**One addition this model needs that the fraud model did not.** A risk corpus can be perfectly
licensed and still unusable for training, because it is the **benchmark**. So the register carries a
`role` column — `eval`, `train` or `none` — and `gate.py` computes *training-eligible rows* from it.
Admissible and trainable are different verdicts, and the difference is the finding of this phase.

---

## 2. The register, executed

`gate.py` does not re-decide the verdicts. It executes them against the bytes, so a verdict cannot
drift from the file it describes.

| ID | Corpus | Licence | Role | Rows on disk | Coverage | Verdict |
|---|---|---|---|---:|---|---|
| **R-01** | German Credit (UCI 144) | CC BY 4.0 | **eval** | 1,000 | W3, W7 | **ADMITTED** |
| **R-02** | Taiwan default (UCI 350) | CC BY 4.0 | **eval** | 30,000 | W1–W4, W7 | **ADMITTED** |
| **R-03** | Credit Approval (UCI 27) | CC BY 4.0 | train | 690 | — | **ADMITTED** (no coverage) |
| **R-04** | Polish bankruptcy (UCI 365) | CC BY 4.0 | train | 43,405 | W5 | **ADMITTED** (corporate) |
| **R-05** | Home Credit Default Risk | unknown — 401 | train | 0 | W1, W2, W8 | **PENDING TERMS** |
| **R-06** | Lending Club | unproven — mirrors disagree | train | 0 | W8 | **REJECTED** |
| **R-07** | FICO HELOC | unverified — bare `cc` | eval | 0 | W6 | **PENDING TERMS** |
| **R-08** | Credit Risk Dataset (Kaggle `laotse`) | **CC0 1.0** | train | 32,581 (inspected) | W5 | **ADMITTED** (synthetic) |
| **R-09** | Give Me Some Credit | contradictory | none | 0 | — | **REJECTED** |
| **R-10** | HMDA LAR | public | none | 0 | — | **OUT OF SCOPE** |
| **R-11** | mirrors / re-uploads | labels never granted | none | 0 | — | **REJECTED** |

Executed checks (summary of `out/gate_report.json`):

| Check | Result |
|---|---|
| Curated file hashes match receipts | **4 / 4** admitted in-pack corpora |
| Record ids unique | **4 / 4** |
| Per-record stamp buildable | **4 / 4** in pack, plus R-08 inspected at source |
| Rows available for evaluation | **31,000** (R-01 + R-02) |
| Rows eligible for training, in pack | **44,095** — 43,405 corporate (R-04) + 690 policy rows (R-03) |
| Rows eligible for training, pending ingest | **32,581** — R-08, synthetic |
| Admitted but covering no weakness row | **R-03** |

**Evidence re-checked at source today, 2026-11-11.** Three candidates changed or confirmed:

- **R-05 Home Credit:** `api/v1/competitions/data/list/home-credit-default-risk` returns **HTTP 401**
  anonymously. The licence has still not been read. `PENDING TERMS`.
- **R-06 Lending Club:** the two mirrors disagree on the same upstream file — `wordsforthewise`
  declares **CC0**, `adarshsng` declares **ODbL**. A re-uploader's label cannot establish upstream
  rights, so the verdict is unchanged: **REJECTED at P2**, not at P1 or P3.
- **R-07 HELOC:** the Hugging Face mirror still declares `license: cc` with no version and no link. A
  bare "cc" is not a licence. `PENDING TERMS`.

---

## 3. The finding: a benchmark corpus cannot be a training corpus

This is the phase's central result, and it is arithmetic rather than opinion.

The lane's Phase 0 chose **cross-validation** for Taiwan (5 × 2 stratified folds) and the **published
10-fold file** for German. Under either protocol, every row is an evaluation row in some fold. If a
model is fine-tuned on the corpus and then evaluated on those folds, it has already seen every row it
is scored on — the evaluation measures memorisation, not skill. The Recipe's decontamination rule
(*zero benchmark examples in training data*) therefore consumes the whole corpus, not a test split.

| Corpus | Role | Rows | Why |
|---|---|---:|---|
| R-01 German | **eval only** | 1,000 | official OpenML task 31, 10-fold |
| R-02 Taiwan | **eval only** | 30,000 | lane protocol, 5 × 2 CV + holdout |
| R-03 Credit Approval | train | 690 | admissible, but 1987 policy labels — not a PD |
| R-04 Polish | train (corporate) | 43,405 | corporate distress, not consumer PD |
| R-08 Kaggle CC0 | train (synthetic) | 32,581 | clean licence, synthetic applicant |

**Net evidence-grade consumer-PD training rows available today: zero.** Every number that could train
a consumer PD model is either synthetic, off-mission (corporate), or a 690-row table of 1987 policy
decisions; every number that could *measure* one is reserved for evaluation.

That is not a dead end — it is a decision with three priced options (D-R11, §9):

| Option | What it costs | What it buys |
|---|---|---|
| **A.** Keep both benchmarks as eval-only | needs a new corpus (R-05 or R-06) before any training | the strongest measurement (MDE 0.014 AUC) survives intact |
| **B.** Re-scope Taiwan to a single 80/20 holdout | MDE rises **0.014 → 0.030** AUC; the 5 × 2 protocol dies and its receipt must be re-registered | 24,000 rows become trainable immediately |
| **C.** Leave both eval-only and train nothing | no trained model this cycle | a zero-shot base row that is honest and measurable now |

**Recommendation:** run **C** first — the Phase 0 base row is zero-shot anyway and costs one GPU-hour —
and decide **A vs B** only if that row shows skill worth training on. Option B is always available and
its price is now written down, which is the point of pricing it before it is chosen.

---

## 4. The one clean new admission — R-08, and exactly what it is

R-08 is a Kaggle dataset published under **CC0 1.0**, verified at source today: the dataset view
endpoint returns `CC0: Public Domain`, and the file downloads anonymously (HTTP 200, zip 376,518
bytes). It passes P1 (per-record origin from row index and hash), P2 (CC0, no duties) and P3
(W5 — it is the only corpus on disk with real missingness).

**It is also synthetic, and the evidence is in the values.** No lender could have issued these rows:

| Check | Observed |
|---|---|
| Oldest applicant | age **144** (5 applicants over 100) |
| Longest employment | **123 years**, in an applicant aged 22 |
| Employment longer than the applicant's life | 2 rows |
| Exact duplicate rows | **165** |
| Rows / defaults | 32,581 / 7,108 (**21.82 %**) |
| Missing cells | 895 `person_emp_length`, 3,116 `loan_int_rate` (**9.56 %**) |

Under the owner's rule it is a **published dataset** and stays `origin_class = DATASET_ROW` forever,
with `synthetic = true` on every record. It is admissible for training and **not evidence-grade**: a
simulated applicant cannot certify a calibration claim about real borrowers, and a model trained on it
inherits an age/employment distribution that no real portfolio has. Its honest use is narrow and
useful — the missingness substrate for W5, and a robustness corpus, never a headline row.

**Track C applied to it:** 165 exact duplicates must be collapsed or family-grouped before any split,
and the impossible-value rows (age > 100, employment > 60) should be quarantined rather than silently
trained on. That is a slice decision, recorded here so Phase 3 inherits it.

---

## 5. Rejections, and the mirror class

| ID | Corpus | Rejection | Level | Could it change? |
|---|---|---|---|---|
| **R-06** | Lending Club | two mirrors, two licences (CC0 vs ODbL) for one upstream file | licence | only via LendingClub's own grant |
| **R-09** | Give Me Some Credit | five re-uploads, five licences (Unknown, CC0, Apache-2.0, MIT, World Bank) | licence | only by reading the upstream competition terms |
| **R-11** | mirrors / re-uploads | a label its publisher never granted | origin | never — the origin corpus governs |

**R-11 is the class, not a row.** Observed today and previously: MIT labels on ULB mirrors, CC0 on
Lending Club, a bare `cc` on HELOC. Under the per-record gate a mirror is not a source at all, because
the licence belongs to the corpus, not to the person who copied it. Ingesting one would attach a
licence the publisher never issued *and* put the same rows on both sides of a split.

**R-10 HMDA** is the one rejection that is not about licensing: it is public US government data,
freely usable, and still useless here — it records **application outcomes** (approved / denied), not
performance, so there is no default label to learn. It is the wrong target variable.

---

## 6. Track C — the pool, sliced

The Recipe's lesson: a rejected dataset is a set of slices, not a verdict. Polish bankruptcy is the
only large admitted training pool, and slicing it changes the number that matters.

| Horizon (years to bankruptcy) | Rows | Bankrupt | Rate | Missing cells | Complete-case rows |
|---|---:|---:|---:|---:|---:|
| 5 | 5,910 | 410 | 6.94 % | 4,666 | 3,031 |
| 4 | 9,792 | 515 | 5.26 % | 8,776 | 4,769 |
| 3 | 10,503 | 495 | 4.71 % | 9,888 | 4,885 |
| 2 | 10,173 | 400 | 3.93 % | 12,157 | 4,088 |
| **1** | **7,027** | **271** | **3.86 %** | 5,835 | **3,194** |

| Slice | Rows | Bankrupt |
|---|---:|---:|
| All rows | 43,405 | 2,091 |
| Complete-case (no imputation) | 19,967 | 432 |
| **Horizon 1, complete-case** | **3,194** | **30** |

**The 43,405-row corpus is a 3,194-row corpus with 30 bankruptcies** once it is narrowed to one
prediction target and rows a model can consume without imputation. Thirty positives is not a training
set; it is a smoke test. The headline row count overstates the usable sample by roughly an order of
magnitude, and the register now says so.

**The limit this slice cannot test:** there is no company identifier in the published file, so the same
firm may appear in several horizons. Rows are not guaranteed independent, and a family split cannot be
proven — the same defect the fraud lane found in ULB, in a different guise.

---

## 7. Track B — generated data, and why it cannot carry this model

The fraud lane can generate synthetic transactions and verify them by execution (a call executes, a
ledger balances). Credit default has no execution semantics: nothing about a fabricated applicant can
be *run* to prove it is a correct default. The recipe's Track B requires correctness by execution
wherever the domain allows — here it does not allow it, and model judgement is explicitly not
verification.

So the honest position is a narrow one:

| Use of generated data | Verdict | Reason |
|---|---|---|
| The PD **target** (fabricate defaulters) | **Refused** | no execution check exists; a synthetic probability cannot certify calibration |
| Explanation / reason-code **fixtures** | Admissible with receipts | supervision is the fixture's own construction, not the model's opinion |
| Missingness and serialisation **stress fixtures** | Admissible | the property asserted (an `NA` was injected) is provable by construction |
| Additional real-shaped training rows | **Refused for v1** | a generator's bias becomes the model's prior, and there is no way to audit it here |

Any Track B row that is ever built carries `origin_class = GENERATED`, `generator`, `seed`,
`source_material` and a `verification` receipt, and it can never be described as a published dataset —
the same permanent-flag rule, pointed the other way.

---

## 8. Per-record provenance — the stamp

Frozen for this model, applied at ingest by the Phase 3 pipeline (not added later). It mirrors the
fraud lane's stamp so one verifier can read both.

| Field | Value | Why |
|---|---|---|
| `row_id` | `cr-` + 9 digits, assigned in fixed source order then publisher row order | stable identity across rebuilds |
| `source_id` | `german_credit` · `taiwan_default` · `credit_approval` · `polish_bankruptcy` · `credit_risk_laotse` | names the upstream corpus — **P1** |
| `source_ref` | publisher URL | the register row it belongs to |
| `source_row_id` | pack `record_id` | traceable to `pack/curated/` |
| `source_row_index` | publisher row position | the per-record locator |
| `retrieved_at` | ingest timestamp (UTC) | reproducibility |
| `licence`, `licence_verdict` | copied from the register row | **P2**, and it travels with the record |
| `origin_class` | `DATASET_ROW` on every row; `GENERATED` only for Track B | the permanent flag |
| `synthetic` | `true` for R-08 and any generated row | never described as real applications |
| `benchmark_flag` | `true` where the row belongs to a frozen evaluation split | keeps eval rows out of training |
| `subset_label` | `forecast_horizon_years` for R-04 | a per-record category label, which counts as origin |
| `row_sha256` | SHA-256 of the canonical token string | the decontamination unit |

**Worked example (R-04, horizon 1):**

```json
{"row_id":"cr-000042317","source_id":"polish_bankruptcy","source_ref":"https://archive.ics.uci.edu/dataset/365/polish+companies+bankruptcy+data",
 "source_row_id":"polish_bankruptcy-042317","source_row_index":42317,"retrieved_at":"2026-11-11T18:20:00Z",
 "licence":"CC BY 4.0","licence_verdict":"ADMITTED","origin_class":"DATASET_ROW","synthetic":false,
 "benchmark_flag":false,"subset_label":{"forecast_horizon_years":1},"row_sha256":"…"}
```

**The rule that makes the stamp mean something:** a record without a `source_id` that resolves to a
register row with an `ADMITTED` verdict does not ship. `P1_SOURCE_NOT_ADMITTED` is a drop, not a
warning.

---

## 9. Coverage against the Phase 1 weakness map

| Weakness | Covered by | State |
|---|---|---|
| **W1** thin-file discrimination | R-05 (pending), R-08 (synthetic, weak) | **gap** — no real thin-file corpus admitted |
| **W2** recall at the operating point | R-02 (eval only) | covered for measurement, **not for training** |
| **W3** calibration in the upper band | R-01, R-02 | covered |
| **W4** repayment-status dependence | R-02 (measured, structural) | covered — a scoping fact, not a data need |
| **W5** missing-data robustness | **R-08** (9.56 % missing), R-04 (41,322 cells) | **covered** |
| **W6** explanation fidelity | R-07 (pending terms) | **gap** |
| **W7** fair lending | R-01, R-02 | covered (diagnostic) |
| **W8** out-of-time stability | R-06 (rejected), R-05 (pending) | **gap** |
| **W9** the base battery | R-01, R-02 | covered |

**Three gaps, one cause.** W1, W6 and W8 need corpora this lane does not have: a real thin-file
population, an explanation benchmark, and a dated portfolio. None of them is a training-mixture
problem; all three are acquisition problems, and every one has a named candidate with a named blocker.

---

## 10. Decision rows

| ID | Decision | Options | Recommendation |
|---|---|---|---|
| **D-R11** | Benchmark/training allocation | A: keep both eval-only, source new training data · B: re-scope Taiwan to a single 80/20 holdout (MDE 0.014 → 0.030) · C: zero-shot only this cycle | **C now**, then A vs B after the base row shows skill |
| **D-R12** | Ingest R-08 (CC0, synthetic) as a training corpus? | yes, flagged · no | **Yes**, narrow: W5 substrate and robustness only, `synthetic = true`, never a headline row |
| **D-R13** | Accept the Kaggle rules for Home Credit? | yes · no | **Yes** — it is the only candidate that closes W1, W2 and W8 at once |
| **D-R14** | Track B policy | PD refused · fixtures only · none | **Fixtures only** (§7) |

---

## 11. Blockers

| # | Blocker | Owner | Effect |
|---|---|---|---|
| **R-05** | Home Credit rules unaccepted (401 today) | Sandeep | No real large training corpus; W1, W2, W8 stay open |
| **R-06** | HELOC terms unread (bare `cc`) | Sandeep | W6 and A5 — the explanation duty — cannot be measured |
| **B-5** | No GPU access | Sandeep | The base row, and therefore D-R11's evidence, is unavailable |
| **B-7** | No second agent, different family | Ronin | Phase 4 verification unassigned |

---

## 12. Receipts and reproduction

| Artefact | What it proves |
|---|---|
| [`phase2-risk/register.json`](phase2-risk/register.json) | 11 sources, verdicts, licence evidence, roles and coverage |
| [`phase2-risk/out/gate_report.json`](phase2-risk/out/gate_report.json) | P1/P2/P3 executed against the bytes; training-eligible rows computed from roles |
| [`phase2-risk/out/candidate_laotse.json`](phase2-risk/out/candidate_laotse.json) | R-08 licence at source, hashes, missingness, impossible values |
| [`phase2-risk/out/slices_polish.json`](phase2-risk/out/slices_polish.json) | the per-horizon and complete-case slice arithmetic |

```bash
cd credit
export PYTHONPATH=$PWD/.pylibs
python3 phase2-risk/gate.py              # executes the gate over the register and the pack
python3 phase2-risk/slices.py            # Polish horizon / complete-case slices
python3 phase2-risk/inspect_candidate.py # re-downloads and re-inspects R-08 (network)
```

The CSV that `inspect_candidate.py` fetches is git-ignored: the receipt is the artefact, not the data.

---

## 13. Recipe conformance

| Recipe clause | Where it is satisfied |
|---|---|
| Public datasets only, predicate-tested first | §1–§3 — three predicates executed by `gate.py` |
| Licence entry per corpus, evidence quoted | §3, register `licence_evidence` + `licence_checked_at` |
| Per-record provenance, applied at ingest | §8 — the frozen stamp, with a worked example |
| Subset/category labels count as origin | §8 `subset_label` (Polish horizon) |
| Dataset-level licence does not count | §5 — the mirror class rejected on exactly this ground |
| Published datasets permanently flagged | §7 — `origin_class = DATASET_ROW` on every record, synthetic flagged separately |
| Generated data: provenance by construction, correctness by execution | §7 — and the honest statement that credit has no execution check, so the PD channel refuses synthetic supervision |
| Coverage judged against the weakness map | §9 — three named gaps |
| Rejected corpus is a set of slices | §6 — Polish sliced from 43,405 to 3,194 rows / 30 positives |

**One departure, named.** The fraud lane's Phase 2 could assume every admitted corpus was trainable.
This packet adds a `role` dimension and reserves two admitted corpora for evaluation. That is stricter,
not looser: it refuses to spend the only measuring instruments on training data.

---

## 14. Revision history

| Rev | Date | Change |
|---|---|---|
| 1 | 2026-11-11 | First Phase 2 packet for Model 2: 11-source register, executed gate, benchmark/training separation, R-08 admission with synthetic flags, Polish slice arithmetic |

---

*No corpus was built. No row was ingested. No training was started. The register's verdicts are about
licences and roles, not about quality; the one number this phase produces that a reader should carry
is zero — the count of evidence-grade consumer-PD rows available for training today.*
