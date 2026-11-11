# Phase 3 — Corpus build, decontamination and the integrity audit

**Lane:** CREDIT · **Model:** 2 — Credit Risk · **Phase:** 3 of 6
**Revision:** 1 · **Date:** 12 November 2026 · **Author:** Sharmila Sirivaram
**Spec:** `credit-risk-token-spec/1.0` · **Pipeline:** [`phase3-risk/`](phase3-risk/) — `build.py` · `controls.py` · `verify.py`
**Receipts:** [`build_report.json`](phase3-risk/out/build_report.json) · [`controls.json`](phase3-risk/out/controls.json) · [`audit.json`](phase3-risk/out/audit.json) · [`verify.json`](phase3-risk/out/verify.json)
**Status:** corpus built and audited. **Released for training: NO** — single synthetic source, owner's tap pending (D-R12).

> **The one sentence.** A corpus is not a pile of rows; it is a pile of rows that survived a scan, a
> set of fixtures that prove the scan works, and an arithmetic line that ties — and this one is
> built from a single synthetic corpus, so it is a **candidate** that proves the machinery, not a
> training release.

---

## 0. What Phase 3 does, and what it cannot do on this lane

Phase 3 builds the training corpus, decontaminates **before** any split, plants controls that prove
the scanner, and audits the final files. All four are done here, on real bytes, with receipts.

What it cannot do is hide the Phase 2 finding: **there are zero evidence-grade consumer-PD rows
available for training.** The corpora that could measure a verdict are the benchmarks (eval-only), the
Polish corpus is corporate and narrows to 30 positives, and the one clean admission is synthetic. So
this phase produces:

- a **pipeline** that is proven, reusable, and carries over from the fraud lane;
- a **corpus** of 32,409 records from the only admissible consumer-PD source, stamped and audited;
- an honest **release verdict**: `NO`, pending D-R12.

That is the correct shape. A pipeline with no corpus would be untested; a corpus with no audit would
be a liability. What is *not* acceptable is presenting 32k synthetic rows as a training set for a
regulated product, and this packet does not.

---

## 1. The corpus, in numbers

| | |
|---|---|
| Source | **R-08** `credit_risk_laotse` — Kaggle, **CC0 1.0**, `DATASET_ROW`, `synthetic: true` |
| Rows in | 32,581 |
| Structural drops | **7** (age > 100, employment > 60, employment > age — values no lender could issue) |
| Content drops | **0** (no benchmark row found) |
| Duplicate drops | **165** exact duplicates, collapsed to one representative per family |
| **Rows out** | **32,409** |
| Positives | 7,088 (21.87 %) |
| Splits | train **25,921** · val **3,247** · test **3,241** |
| Spec | `credit-risk-token-spec/1.0` — `col=value`, ordered, `__NA__`, no rounding |

**Tie-out, per source and in total:** `32,581 = 0 provenance + 0 content + 7 structural + 165 duplicate
+ 32,409 kept` — **ties**.

**What is not in it, by rule.** German (1,000 rows) and Taiwan (30,000 rows) are **evaluation-only**
under `CREDIT-RISK-P0-PREREG-001`; any row of theirs appearing here would be a benchmark leak, which
is what the scan exists to catch. R-03 Credit Approval is excluded on target semantics — its label is
1987 lending policy, not default. R-04 Polish is excluded from the PD corpus: corporate bankruptcy is
a different question, and its usable slice is 30 positives.

---

## 2. Decontaminate before splitting — the order is the point

```
load → structural drops → tokenise → CONTENT SCAN → provenance drops → duplicate families
     → near-duplicate families → split by family → audit → verify at the destination
```

The scan runs **before** the split. A row that matches a benchmark is removed while it is still in one
undifferentiated corpus, so no split is ever built on contaminated rows — the failure mode the Recipe
names is a corpus where the leak was found after the split and "fixed" by moving rows.

### 2.1 The matcher had to change, and the reason is measured

The fraud matcher requires **eight consecutive tokens, in order** — correct for long transaction
strings, because a bag-of-words scanner once produced totals that had to be withdrawn in full.

On this lane that rule is necessary for benchmark leaks and **useless for near-duplicates**, and the
controls proved it: an 11-token risk record contains only **4** eight-grams, so changing one field
destroys all four and an 8-gram matcher reports **0 near-duplicates** on a corpus that has 1,003 of
them. The first control run failed on exactly this, and the failure is recorded rather than tuned away.

So the scan is two matchers, each for its own job:

| Matcher | Unit | Threshold | Catches |
|---|---|---|---|
| Benchmark leak | 8 consecutive tokens, in order | any shared run | a benchmark row, or a paraphrase long enough to share a run |
| Near-duplicate | `field=value` token set | overlap ≥ **0.90** | the same applicant with one field changed (10/11 = 0.909) |

Both are in the receipt with the threshold they were frozen at. The overlap coefficient, not Jaccard,
is the near-duplicate measure: Jaccard scores the same one-field edit 10/12 = 0.83 and would pass it.

### 2.2 Controls — a scan without fixtures is not evidence

| Control | Required | Observed | Verdict |
|---|---|---|---|
| Planted benchmark leaks (real German + Taiwan rows, serialised) | all dropped | **100 / 100** | pass |
| Known-clean records (corpus rows) | all kept | **400 / 400** | pass |
| Near-duplicate pairs (one numeric field nudged) | all caught | **10 / 10** | pass |
| **Negative control** — matcher disabled | leaks must survive | **100 / 100 survive** | pass (the harness fails when it should) |
| Arithmetic | scanned = dropped + kept | **500 = 100 + 400** | ties |

**The negative control is the one that matters.** A scan that drops everything, or nothing, can pass a
one-sided test. Running the same fixtures with the matcher disabled and confirming the 100 leaks come
through is what proves the drops came from the matcher and not from the plumbing.

---

## 3. Splitting by family — and the part that cannot be proven

Splits are built by **family root**, and a family is a duplicate component, not a row:

| Family construction | Observed |
|---|---:|
| Exact-duplicate groups | 165 (collapsed to one row each) |
| Near-duplicate **pairs** (overlap ≥ 0.90) | **1,003** |
| Near-duplicate **components** | 31,406 |
| Largest component | **3 rows** |
| Near-duplicate pairs crossing a split | **0** |

**The limit, stated plainly:** R-08 publishes **no entity identifier**. There is no customer, no
account, no application id — so an *entity* family cannot be constructed and the claim "no customer
appears in two splits" is **UNPROVEN**, not passed. What is proven is narrower and still valuable:
every exact and near-duplicate component is kept whole, and that was verified on the shipped files
(§4). This is the same defect the fraud lane found in ULB, in a different guise, and it is recorded
rather than papered over.

---

## 4. The integrity audit — on the final files, not in the builder's memory

The Recipe's lesson is a split-writer defect that collided on **793 duplicate record ids** while the
code "obviously" could not do that. `verify.py` therefore re-reads the three JSONL files from disk and
re-derives every claim.

| Check | Result |
|---|---|
| Re-derive each record's `token_sha256` from its own `prompt_fields` | **0 failures** of 32,409 |
| Row ids unique across the whole corpus | **0 duplicates** |
| Exact duplicate in two splits (train↔val, train↔test, val↔test) | **0 / 0 / 0** |
| Near-duplicate pair straddling two splits | **0** |
| File row counts match the builder's receipts | **yes** |
| Tie-out re-derived from the files | `32,581 = 172 + 32,409` — **ties** |
| **Positive control** — a one-field mutation of a val record | **5 / 5 caught** |

The positive control exists because "0 near-duplicates across splits" is a pass that a broken checker
also produces. Mutating five val records and requiring all five to be detected is what makes the zero
mean something.

---

## 5. The record format — verdict channel only, and why

```json
{"row_id":"cr-000004231","source_id":"credit_risk_laotse",
 "prompt_fields":{"person_age":"26","person_income":"48000", "…":"…","loan_int_rate":null},
 "risk":" default",
 "provenance":{"row_id":"cr-000004231","source_id":"credit_risk_laotse","source_ref":"https://www.kaggle.com/datasets/laotse/credit-risk-dataset",
               "source_row_id":"credit_risk_laotse-004231","source_row_index":4231,"retrieved_at":"2026-11-12T00:00:00Z",
               "licence":"CC0 1.0","licence_verdict":"ADMITTED","origin_class":"DATASET_ROW","synthetic":true,"benchmark_flag":false},
 "token_sha256":"…"}
```

**The verdict field is byte-identical to the Phase 0 scoring prefix** `{"risk": "` — the format the
model is trained on is the format the harness measures, which is the Phase 5 rule applied before
Phase 5.

**The reason channel is absent, deliberately.** No admitted corpus carries ground-truth adverse-action
reason codes. A reason derived from SHAP or from the model's own opinion is not verified supervision —
it is a model judgement, and the Recipe does not allow judgement as verification. Building reason
records from it would manufacture the one capability this lane most needs to measure (R4/W6) and make
it permanently unfalsifiable. So the corpus is verdict-only, and the explanation duty stays
**UNMEASURED** with its commission (`CM-R3`) rather than silently faked.

---

## 6. Provenance — the stamp is applied at ingest, not added later

Every record carries the frozen stamp from Phase 2 §8: `row_id`, `source_id`, `source_ref`,
`source_row_id`, `source_row_index`, `retrieved_at`, `licence`, `licence_verdict`, `origin_class`,
`synthetic`, `benchmark_flag`, `token_sha256`.

| Check | Result |
|---|---|
| Records whose `source_id` resolves to an `ADMITTED` register row | **32,409 / 32,409** |
| Records whose licence field matches the register row | **32,409 / 32,409** |
| `origin_class = DATASET_ROW` on every record | **32,409 / 32,409** |
| `synthetic = true` on every record | **32,409 / 32,409** |
| `benchmark_flag = true` | **0** (benchmarks are excluded entirely) |

A record whose `source_id` does not resolve to an admitted register row does not ship; that is a drop
(`P1_SOURCE_NOT_ADMITTED`), not a warning. It fired zero times here, which is correct — and the
controls prove it would fire if a benchmark row were slipped in.

---

## 7. What this corpus is, and what it is not

| It is | It is not |
|---|---|
| a proven pipeline, reusable for the fraud lane and any future corpus | evidence about real borrowers |
| 32,409 stamped, scanned, split, audited records | a training release |
| the W5 missingness substrate (9.56 % missing, two fields) | a calibration corpus — synthetic applicants cannot calibrate a real PD |
| a format that matches the Phase 0 scoring path exactly | a reason-code corpus (the channel is absent by design) |

**Release verdict: NO.** The owner's tap (D-R12) decides whether a synthetic-only corpus is ever used
to train. Until then this is a candidate, and the honest headline is that the lane's Phase 3 gate is
open — not passed.

---

## 8. What Phase 4 must verify, and on which files

The independent-verifier gate comes next, and this packet says exactly what it will be handed:

| Input | Path | State |
|---|---|---|
| Frozen spec | `phase3-risk/spec.py` | `credit-risk-token-spec/1.0`, hash in `build_report.json` |
| Three split files | `phase3-risk/out/corpus/{train,val,test}.jsonl` | hashes in `build_report.json` §files |
| Build receipt | `phase3-risk/out/build_report.json` | tie-out + audit + limits |
| Controls | `phase3-risk/out/controls.json` | 100/100, 400/400, 10/10, negative control |
| Destination check | `phase3-risk/out/verify.json` | re-derived hashes, 0 cross-split leakage, positive control |

**The verifier can refuse.** The three things it should refuse on: a hash that does not re-derive, a
cross-split duplicate, or a families claim presented as proven when R-08 has no entity id.

---

## 9. Decision rows and blockers

| ID | Decision | State |
|---|---|---|
| **D-R12** | Ingest/train on R-08 (synthetic)? | **Open** — corpus built, release withheld |
| **D-R15** | Accept "entity family UNPROVEN" for a synthetic corpus? | **Open** — relevant only if R-08 ever ships |
| **R-05** | Home Credit access | Open — the only route to a real training corpus |
| **R-06** | HELOC terms | Open — blocks the reason channel |
| **B-5** | GPU access | Open — blocks the base row, not this phase |
| **B-7** | Second-family verifier | Open — Phase 4 is unassigned |

---

## 10. Receipts and reproduction

```bash
cd credit
export PYTHONPATH=$PWD/.pylibs
python3 phase3-risk/build.py      # build, scan, split, audit  → out/build_report.json, out/audit.json
python3 phase3-risk/controls.py   # planted fixtures + negative control → out/controls.json
python3 phase3-risk/verify.py     # re-derive everything from the files → out/verify.json
```

The corpus under `phase3-risk/out/corpus/` and the candidate CSV are **git-ignored**: the receipts and
their hashes are the artefact. Anyone can rebuild the bytes from the URL in the register and check
the hash.

---

## 11. Recipe conformance

| Recipe clause | Where it is satisfied |
|---|---|
| Decontaminate **before** splitting | §2 — scan precedes split; order is in the script |
| Zero benchmark examples in training data | §2.2 — 100 planted leaks dropped; benchmarks excluded wholesale |
| Matcher requires words together, in order | §2.1 — 8-gram in-order for leaks; measured failure of that unit for near-dups, and the fix |
| Planted controls in every scan | §2.2 — plus a negative control that must fail |
| Silent exclusions are defects | §1 tie-out — every drop is counted and named, and the arithmetic ties |
| Integrity audit on final files | §4 — duplicates, near-duplicates, cross-split leakage, re-derived hashes |
| Splits by family root, verified on files | §3–§4 — components kept whole, verified; entity family recorded UNPROVEN |
| Arithmetic ties end to end | §1, §4 — `32,581 = 0 + 0 + 7 + 165 + 32,409` |
| Licence FINAL before training | §7 — release withheld; provenance stamp checked on every record |

---

## 12. Revision history

| Rev | Date | Change |
|---|---|---|
| 1 | 2026-11-12 | First Phase 3 packet for Model 2: corpus built from R-08, dual matcher, controls, destination audit, release withheld |

---

*No training was started. The corpus is a candidate built from a single synthetic source, with every
drop counted, every claim re-derived from the files, and the one thing that cannot be proven — entity
families — recorded as unproven rather than assumed.*
