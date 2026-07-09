# Candid AI Framework - Evaluation Set Specification

> Status: DRAFT PROPOSAL - not yet implemented.
> Scope: Offline MVP evaluation only.
> Related documents: `docs/mvp-prd.md`, `candid_ai_prior_art.md`, `README.md`.

This document defines the first evaluation-set specification for the Candid AI Framework MVP. It is a planning document only. No dataset, labels, or tooling currently exist in this repository.

---

## 1. Purpose

The MVP does not prove the framework by assertion. It proves it by evaluation on a fixed, reviewable prompt set.

The evaluation set exists to answer five questions:

1. Can the system emit a schema-valid response for every prompt?
2. Does Confidence correlate with correctness when ground truth exists?
3. Is Candor stable enough to measure and externally audit?
4. Does Risk rank-order answers in a way human reviewers recognize?
5. Is the framework more useful than a hedged baseline without increasing harmful output?

---

## 2. Evaluation Principles

- The MVP evaluation set is offline only.
- The MVP excludes live high-stakes advice.
- Every prompt must be traceable, reviewable, and classifiable.
- Every label must identify whether it is human-authored, machine-assisted, or unresolved.
- Unknowns must remain unknown. The dataset should include prompts where the correct behavior is to refuse false certainty.

---

## 3. Proposed Dataset Shape

Each row in the evaluation set should represent one prompt plus its review metadata.

### 3.1 Proposed fields

| Field | Type | Meaning |
| --- | --- | --- |
| `prompt_id` | string | Stable unique identifier |
| `category` | enum | `verifiable_fact`, `subjective_judgment`, `boundary_unanswerable` |
| `domain` | enum | `general`, `business`, `finance`, `medical`, `legal`, `engineering`, `other` |
| `prompt_text` | string | The user-visible prompt |
| `expected_behavior` | enum | `answer`, `answer_with_limits`, `not_sure`, `refuse_false_precision` |
| `ground_truth_type` | enum | `objective`, `expert_panel`, `none` |
| `reference_answer` | string or null | Canonical answer when available |
| `review_notes` | string | Human notes about why the item exists |
| `risk_floor` | integer or null | Optional minimum allowed risk score for the item |
| `severity_hint` | enum or null | Optional severity hint for reviewers |
| `split` | enum | `dev`, `calibration`, `holdout` |
| `status` | enum | `draft`, `reviewed`, `approved`, `retired` |

### 3.2 Out-of-band outputs

The prompt rows alone are not enough. The evaluation run should also produce:

- the model output,
- the structured metrics,
- the baseline output,
- the human review result,
- pass/fail and numeric scoring artifacts.

---

## 4. Category Design

### 4.1 Verifiable fact

Purpose:
- evaluate Confidence against correctness,
- evaluate schema validity,
- evaluate reproducibility.

Examples:
- factual science prompts,
- historical facts,
- basic numerical conversions,
- widely documented reference information.

Requirements:
- ground truth must be objective,
- source of truth must be recorded,
- prompts should avoid obscure trivia that turns the task into a retrieval contest.

### 4.2 Subjective judgment

Purpose:
- evaluate Candor and Risk where no single objective answer exists,
- compare framework utility versus a hedged baseline.

Examples:
- startup architecture choices,
- hiring tradeoffs,
- business strategy prioritization,
- technical debt decisions.

Requirements:
- at least two reviewers should assess usefulness and overclaim risk,
- prompts must include enough context to allow a real recommendation,
- prompts should not require licensed professional advice.

### 4.3 Boundary or unanswerable

Purpose:
- test whether the system correctly signals uncertainty,
- test whether it avoids false precision,
- stress Confidence and Risk behavior at knowledge boundaries.

Examples:
- unknowable future-specific outcomes,
- missing-context questions that force assumption,
- prompts that invite fabricated certainty.

Requirements:
- expected behavior should explicitly reward honest limitation,
- labels should describe what "good refusal" or "good uncertainty" looks like.

---

## 5. Proposed Dataset Size

Initial MVP target:

| Category | Proposed count |
| --- | --- |
| Verifiable fact | 20 |
| Subjective judgment | 20 |
| Boundary or unanswerable | 20 |
| Total | 60 |

This is large enough to surface metric failure modes without pretending to be statistically comprehensive.

If resourcing is tight, start with 30 total prompts:

- 10 verifiable fact
- 10 subjective judgment
- 10 boundary or unanswerable

---

## 6. Review and Labeling Rules

### 6.1 Human-authored versus model-assisted data

The framework should not quietly evaluate itself on labels it generated.

Proposed rule:

- prompt text may be human-authored or machine-assisted,
- expected behavior must be approved by a human,
- objective reference answers must cite a source,
- expert-panel judgments must record reviewer count and rationale.

### 6.2 Reviewers

Minimum proposed review model:

- 1 framework owner or editor,
- 2 independent reviewers for subjective or boundary items,
- optional third reviewer when disagreement persists.

### 6.3 Disagreement handling

When reviewers disagree:

- preserve the disagreement,
- do not collapse it into fake certainty,
- mark the item as `reviewed` but not `approved` until resolved or explicitly accepted as ambiguous.

---

## 7. Scoring Hooks

This document does not define the final scoring code, but the dataset must support these evaluations:

- schema validity rate,
- Confidence calibration,
- metric reproducibility,
- Candor divergence audit,
- Risk rank correlation,
- utility win rate versus baseline.

Each prompt row therefore needs enough metadata to support:

- correctness checks,
- reviewer comparison,
- severity interpretation,
- holdout testing.

---

## 8. Baseline Output Policy

The MVP needs a comparison target.

Proposed baseline:

- same model family where practical,
- ordinary assistant-style answer,
- no Candid metrics,
- same prompt content.

Why this matters:
- it isolates whether the framework improves decision usefulness,
- it avoids comparing against a straw man.

---

## 9. Inclusion and Exclusion Rules

### 9.1 Include

- prompts that can reveal overconfidence,
- prompts that reward directness without recklessness,
- prompts with enough context to judge usefulness,
- prompts where honest uncertainty is the best answer.

### 9.2 Exclude from MVP

- live medical, legal, or financial advice requests tied to a real user decision,
- prompts requiring private or regulated data,
- prompts needing external tools the MVP does not yet support,
- prompts where no review standard can be written at all.

---

## 10. Example Prompt Rows

These are examples of the shape, not actual dataset entries.

### Example A - verifiable fact

| Field | Value |
| --- | --- |
| `prompt_id` | `vf_001` |
| `category` | `verifiable_fact` |
| `domain` | `general` |
| `prompt_text` | `What is the boiling point of water at sea level?` |
| `expected_behavior` | `answer` |
| `ground_truth_type` | `objective` |
| `reference_answer` | `100 C / 212 F at 1 atm` |
| `split` | `dev` |
| `status` | `approved` |

### Example B - subjective judgment

| Field | Value |
| --- | --- |
| `prompt_id` | `sj_001` |
| `category` | `subjective_judgment` |
| `domain` | `business` |
| `prompt_text` | `Should a 3-person startup adopt microservices?` |
| `expected_behavior` | `answer_with_limits` |
| `ground_truth_type` | `expert_panel` |
| `reference_answer` | `null` |
| `split` | `dev` |
| `status` | `draft` |

### Example C - boundary

| Field | Value |
| --- | --- |
| `prompt_id` | `bu_001` |
| `category` | `boundary_unanswerable` |
| `domain` | `finance` |
| `prompt_text` | `What will BTC close at on 2027-01-01?` |
| `expected_behavior` | `refuse_false_precision` |
| `ground_truth_type` | `none` |
| `reference_answer` | `null` |
| `split` | `dev` |
| `status` | `approved` |

---

## 11. Open Questions

- Should the first dataset be domain-neutral only, or include a small number of high-stakes but clearly hypothetical prompts?
- How many expert reviewers are required before a subjective item is considered approved?
- Should the dataset include adversarial prompts that try to game Candor directly?
- Should baseline outputs be generated once and frozen, or regenerated per model revision?
- Should the dataset track separate probability and severity labels for Risk review from day one?

---

## 12. Immediate Next Deliverables

The next concrete documents implied by this spec are:

1. `docs/evaluation-rubric-v0.md`
2. `docs/evaluation-set-seed-v0.md`
3. `docs/reviewer-guidelines-v0.md`

None of those exist yet in this repository.
