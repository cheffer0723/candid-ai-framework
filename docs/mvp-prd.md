# Candid AI Framework - MVP Product Requirements Document (PRD)

> Status: DRAFT PROPOSAL - not yet implemented.
> Author of underlying framework: Christopher P. Heffernan, Alliance, Ohio, United States.
> Framework first public disclosure: April 29, 2026 (see `../candid_ai_prior_art.md`).
> This document date: July 8, 2026.
>
> This PRD translates the prose-only Candid AI Framework into a buildable specification.
> Nothing described here as "architecture," "component," "endpoint," or "schema" currently exists in this repository. Every technical construct below is a proposal only. The repository today contains three Markdown documents (`README.md`, `candid_ai_prior_art.md`, `LICENSE.md`) and no source code, tests, packages, or deployment assets.
>
> Licensing: This PRD is part of a repository governed by a proprietary license (see `../LICENSE.md`). Publishing this design document does not grant any implementation, commercial, training, or sublicensing rights. All such rights remain reserved by the author. This PRD is a planning artifact, not a license grant.

---

## 0. Reading Guide

This document is intentionally explicit about the boundary between what the framework asserts and what a builder would have to decide, define, or assume to ship it. Three labels are used throughout:

| Label | Meaning |
| --- | --- |
| `[DISCLOSED]` | Stated in the existing prior-art documents. |
| `[PROPOSAL]` | A design decision introduced by this PRD to make the framework buildable. Not disclosed, not implemented. |
| `[ASSUMPTION]` | A claim the framework relies on that is currently unverified and may be false. Flagged for validation. |

---

## 1. Purpose and Background

### 1.1 What the framework is `[DISCLOSED]`

The Candid AI Framework is a reasoning-and-output standard for LLMs built on one primary directive: an AI system must never waste the human's time, and this takes precedence over engagement optimization, liability hedging, and comfort management. Every substantive response is accompanied by three explicit, per-response metrics:

- Risk Level (0-100%) - probability that acting on the response leads to a negative outcome, scaled by severity.
- Candor Level (0-100%) - how direct or unhedged the response is versus a softened version.
- Confidence / Honesty (0-100% + Honest / Not Sure flag) - the model's self-assessed factual certainty.

The framework also discloses a four-stage internal reasoning process:

1. Best Answer Isolation
2. Risk Scoring
3. Candor Decision
4. Confidence Scoring

### 1.2 Why a PRD is needed

The prior-art documents define the concept but specify no implementation: no scoring functions, no calibration method, no interface contract, no evaluation methodology, and no definition of a "substantive response." Without these, the framework cannot be built or tested. This PRD supplies a proposed, buildable interpretation and surfaces the assumptions and open questions that must be resolved before or during a build.

### 1.3 Non-goals of the MVP `[PROPOSAL]`

- Not building a consumer chatbot product or brand.
- Not fine-tuning or training a model.
- Not claiming the three metrics are calibrated or accurate. The MVP's job is to demonstrate they can be produced, displayed, and evaluated, and to measure how good they are.
- Not implementing commercial licensing, billing, or multi-tenant infrastructure.

---

## 2. Problem Statement `[DISCLOSED + PROPOSAL]`

Modern LLMs default to a "Safe Loop" pattern:

1. Acknowledge the question broadly.
2. Present multiple perspectives without committing.
3. Add disclaimers.
4. Defer to a professional.
5. Offer to help again.

This consumes user time without delivering actionable intelligence. `[DISCLOSED]`

The MVP must demonstrate a concrete alternative: for a given user prompt, produce a direct best answer plus three transparent metrics in a form a user or evaluator can inspect. `[PROPOSAL]`

The measurable question the MVP exists to answer is:

Can an LLM emit Risk %, Candor %, and Confidence % per response in a way that is:

- reproducible,
- meaningfully correlated with ground truth where ground truth exists, and
- more useful to a decision-maker than a hedged baseline?

---

## 3. Users and Use Cases `[PROPOSAL]`

| Persona | Need | MVP relevance |
| --- | --- | --- |
| Framework evaluator | Verify the three metrics can be produced and scored | Core MVP audience |
| Decision-maker end user | A direct answer with visible risk, candor, and confidence | Demonstrated via demo harness |
| Prospective commercial licensee | Evidence the framework is real and measurable | Consumes the evaluation report |

The disclosed intended application domains are high-stakes: legal, financial, medical, business strategy, hiring, and engineering risk. `[PROPOSAL]` The MVP should deliberately avoid live high-stakes advice and instead use a fixed evaluation set to prevent the demo itself from causing the harm the Risk metric is meant to warn about.

---

## 4. Scope

### 4.1 MVP scope (in) `[PROPOSAL]`

1. A prompt contract that instructs a current LLM to run the four-stage reasoning process and emit a structured response containing the answer plus the three metrics.
2. A structured output schema so metrics are machine-readable, not just prose.
3. A demonstration harness that sends a prompt, receives the structured output, and renders the answer plus three metrics.
4. A fixed evaluation set of prompts spanning verifiable-fact, subjective-judgment, and unanswerable categories.
5. An evaluation report scoring the metrics against the criteria in Section 8.

### 4.2 MVP scope (out) `[PROPOSAL]`

- Model training or fine-tuning.
- Real-time production API, auth, rate limiting, or billing.
- UI beyond a minimal demo renderer.
- Any claim of calibration guarantees.
- Multi-model ensembling or cross-model arbitration.

### 4.3 Explicitly deferred (post-MVP) `[PROPOSAL]`

- Calibration layer mapping raw model self-scores to empirically calibrated probabilities.
- Human-in-the-loop candor auditing at scale.
- Domain-specific risk rubrics.

---

## 5. Metric Definitions and Scoring Assumptions

Each metric is broken into the disclosed definition, what must be human-defined, what is model-scored, and the assumptions or risks.

### 5.1 Risk Level (0-100%)

- `[DISCLOSED]` Definition: probability that acting on the response yields a negative outcome, scaled by severity.
- Human-defined `[PROPOSAL]`:
  - rubric anchors pinning what roughly 10%, 50%, and 90% risk mean,
  - severity scale such as trivial, moderate, serious, catastrophic,
  - domain risk floors where needed.
- Model-scored `[PROPOSAL]`: for a specific answer, the model estimates probability of negative outcome and severity, then maps to 0-100 against the anchors.
- `[ASSUMPTION]`:
  - an LLM can estimate probability of a negative real-world outcome with useful reliability,
  - a single 0-100 number can encode probability times severity without losing key decision signal.

Recommendation `[PROPOSAL]`: internally capture probability and severity as separate fields and derive the single Risk % for display.

### 5.2 Candor Level (0-100%)

- `[DISCLOSED]` Definition: degree to which the response reflects the model's actual highest-probability conclusion versus a softened or hedged version.
- Human-defined `[PROPOSAL]`: operational definition of softening and how to detect it, such as false balance, unnecessary deferral, buried conclusions, and hedging qualifiers.
- Model-scored `[PROPOSAL]`: the model reports how much it moderated its Stage 1 best answer when producing the final answer.
- `[ASSUMPTION]`:
  - the model can reliably self-report its own suppression or softening,
  - there is a meaningful external way to validate this self-report.

Recommendation `[PROPOSAL]`: treat Candor as the least validated metric and require a dedicated external audit method.

### 5.3 Confidence / Honesty (0-100% + flag)

- `[DISCLOSED]` Definition: self-assessed confidence that the response is factually accurate and complete, plus a binary Honest / Not Sure flag near the model's knowledge boundary.
- Human-defined `[PROPOSAL]`:
  - threshold at which the flag flips to Not Sure,
  - possibly rename the label to Sure / Not Sure or Verified / Unverified to avoid implying the alternative is dishonest.
- Model-scored `[PROPOSAL]`: self-reported confidence percentage.
- `[ASSUMPTION]`:
  - raw self-confidence is not calibrated and may be overconfident,
  - this is still the most testable of the three metrics because it can be compared to correctness on verifiable prompts.

### 5.4 Cross-cutting assumptions `[ASSUMPTION]`

1. Self-report validity: all three metrics are self-scores, not independent measurements.
2. Determinism: metrics may vary across runs unless temperature and prompt contract are pinned.
3. Prompt sensitivity: the numbers may shift materially with prompt wording.
4. False precision: a number like 67% implies more rigor than the system may actually have.

---

## 6. Proposed Functional Requirements `[PROPOSAL]`

| ID | Requirement |
| --- | --- |
| FR-1 | The system shall accept a text prompt and produce a single structured response object. |
| FR-2 | The response object shall contain a direct best-answer field plus `risk`, `candor`, and `confidence` metric objects. |
| FR-3 | Each metric object shall contain a 0-100 integer, a one-line rationale, and for confidence a Sure / Not Sure-style flag. |
| FR-4 | The system shall follow the four-stage reasoning process and be able to emit each stage's intermediate output when a `debug` flag is set. |
| FR-5 | The system shall run at a pinned, low temperature and record the model ID and prompt-contract version in every response. |
| FR-6 | The system shall not provide live advice on the disclosed high-stakes domains during MVP evaluation; it operates on a fixed evaluation set only. |
| FR-7 | The display shall label all metrics as model self-estimates, not calibrated measurements. |

### 6.1 Proposed structured output schema `[PROPOSAL]`

```jsonc
{
  "answer": "string - the direct best answer",
  "risk": {
    "score": 0,
    "probability_of_harm": 0,
    "severity": "trivial|moderate|serious|catastrophic",
    "rationale": "string"
  },
  "candor": {
    "score": 0,
    "softening_applied": "none|mild|significant",
    "rationale": "string"
  },
  "confidence": {
    "score": 0,
    "flag": "Sure|NotSure",
    "rationale": "string"
  },
  "meta": {
    "model_id": "string",
    "prompt_contract_version": "string",
    "temperature": 0.0
  }
}
```

---

## 7. Proposed Architecture `[PROPOSAL - NOT IMPLEMENTED]`

This section describes one possible build. It does not describe anything currently in the repo.

```text
[ Prompt ]
    -> [ Prompt Contract v0 ]  (human-authored instructions + rubric anchors)
    -> [ LLM call ]            (current model, low temp, structured output)
    -> [ Structured response ] (JSON per Section 6.1)
    -> [ Demo renderer ]       (shows answer + 3 metrics + "estimate" disclaimer)
    -> [ Evaluation harness ]  (scores response against Section 8 on the fixed eval set)
    -> [ Evaluation report ]   (Markdown)
```

Minimal proposed components:

1. A version-pinned prompt contract.
2. A thin call wrapper enforcing the schema.
3. A renderer.
4. An offline evaluation harness.

No database, auth, or service layer is required for the MVP. Language and framework choice is deliberately left open and is not implied to exist already.

---

## 8. Evaluation Plan

The evaluation is the heart of the MVP. It is how "the framework works" moves from assertion to evidence.

### 8.1 Evaluation dataset `[PROPOSAL]`

A fixed set of prompts, proposed at roughly 30-60 for MVP, across three categories:

| Category | Example | Primary metric tested | Ground truth available? |
| --- | --- | --- | --- |
| Verifiable fact | "What is the boiling point of water at sea level?" | Confidence | Yes |
| Subjective / judgment | "Should a 3-person startup adopt microservices?" | Candor, Risk | Partial |
| Unanswerable / boundary | "What will BTC close at on 2027-01-01?" | Confidence flag, Risk | Yes, because it should be treated as unknowable in advance |

### 8.2 Confidence: calibration `[PROPOSAL, testable]`

On the verifiable-fact subset, compare stated confidence to actual correctness. Report Expected Calibration Error (ECE) and a reliability diagram. Success target for MVP: higher stated confidence should correspond to higher empirical accuracy, even if not perfectly calibrated.

### 8.3 Reproducibility `[PROPOSAL, testable]`

Run each prompt multiple times, proposed `N=5`. Report the spread or standard deviation of each metric. Success target: metric standard deviation stays within a stated band, for example 10 points or less, at low temperature.

### 8.4 Candor: external audit `[PROPOSAL, partially testable]`

Because candor has no self-evident ground truth, use an A/B elicitation probe:

1. Ask the model for its most direct, unfiltered assessment, ignoring tone.
2. Ask for its normal framework answer.
3. Have independent human raters, or a separately disclosed judge process, score the divergence.
4. Compare measured divergence to the self-reported Candor %.

If self-reported candor does not track measured divergence, the metric is not valid and the MVP should say so plainly.

### 8.5 Risk: outcome proxy `[PROPOSAL, weakly testable]`

True risk requires observing real outcomes, which the MVP does not do. Proxy approach: have domain reviewers rank a subset of answers by riskiness and check rank-correlation, such as Spearman correlation, with the Risk %. This tests ordering, not absolute probability.

### 8.6 Utility vs. baseline `[PROPOSAL]`

Blind comparison: reviewers see framework output versus a hedged baseline answer for the same prompt and judge which better respects the primary directive without increasing harm. Report win rate.

### 8.7 Success criteria summary

| Dimension | MVP pass condition (proposed) |
| --- | --- |
| Producibility | 100% of eval prompts yield schema-valid output |
| Confidence validity | Monotonic accuracy vs confidence; ECE reported |
| Reproducibility | Metric standard deviation <= 10 points at low temp |
| Candor validity | Self-reported candor rank-correlates with measured divergence; honest null result acceptable |
| Risk ordering | Rank correlation with expert ordering reported |
| Utility | Framework >= baseline on blind decision-usefulness, with no increase in harmful answers |

Negative results are valid MVP outcomes. If candor or risk self-scores prove invalid, the MVP's value is documenting that clearly.

---

## 9. Example Outputs `[PROPOSAL, illustrative]`

These are illustrative renderings only, not outputs from an implemented system.

### Example A - verifiable fact

Answer: Water boils at 100 C (212 F) at standard sea-level pressure (1 atm).  
Risk: 3% (trivial severity - acting on this has negligible downside)  
Candor: 98% (no softening applied)  
Confidence: 99% - Sure

### Example B - subjective or judgment

Answer: No. A 3-person startup should not adopt microservices yet; the operational overhead will outpace its delivery capacity. Start with a modular monolith.  
Risk: 40% (moderate - outcome depends on unstated growth plans)  
Candor: 90% (direct; minor softening on tone)  
Confidence: 72% - Sure

### Example C - unanswerable or boundary

Answer: This is not knowable. No method can predict a specific future asset price.  
Risk: 95% (catastrophic severity if acted on as if reliable)  
Candor: 97% (stating the limit directly rather than guessing)  
Confidence: 15% - Not Sure

---

## 10. Failure Cases `[PROPOSAL]`

| ID | Failure mode | Why it matters | Mitigation / test |
| --- | --- | --- | --- |
| F-1 | Confident-but-wrong - high Confidence % on an incorrect answer | Directly harms trust | Calibration test |
| F-2 | False candor - high Candor % while heavily hedging | Metric is self-referential and unverifiable | Candor audit |
| F-3 | Risk collapse - rare catastrophe scored the same as common nuisance | Loses the critical distinction | Separate probability and severity internally |
| F-4 | False precision - "67%" read as a real measurement | Overtrust in an estimate | Mandatory estimate labeling |
| F-5 | Prompt or temperature drift - metrics swing across runs | Undermines reproducibility | Reproducibility test, low temp |
| F-6 | Directive misuse - "don't waste time" used to justify overconfident bluntness | Turns candor into recklessness | Utility test requires no increase in harm |
| F-7 | Schema or format break - model returns prose, not valid structured output | Breaks machine-readability | Schema validation |
| F-8 | Gaming - model learns to emit pleasing metric values decoupled from behavior | Metrics become theater | Cross-check against external signals where available |

---

## 11. Unverifiable Assumptions, Open Questions, and Blockers

### 11.1 Unverifiable or currently unverified assumptions `[ASSUMPTION]`

- An LLM can meaningfully self-report candor.
- Risk % as probability of real-world harm is estimable by a model that never observes outcomes.
- Collapsing probability times severity into one Risk number preserves decision value.
- The primary directive can be operationalized without incentivizing harmful overconfidence.
- Self-reported metrics are stable enough across prompts and runs to be comparable.

### 11.2 Open questions

- Should the confidence flag wording change from "Honest / Not Sure" to "Sure / Not Sure" or "Verified / Unverified"?
- What counts as a substantive response requiring metrics versus small talk that does not?
- Who authors the rubric anchors and severity scale, and for which domains first?
- Which LLM or LLMs should be used for evaluation, and does using them interact with the repository's licensing stance?
- Should the MVP ship raw self-scores with disclaimers, or block on a calibration layer first?
- Is documenting that candor or risk self-scoring is invalid an acceptable MVP outcome?

### 11.3 Blockers

- B-1: No rubric anchors or severity scale exist yet.
- B-2: No agreed candor ground-truth method exists yet.
- B-3: Any move from docs to code requires confirming it is consistent with the proprietary license and the author's intent.
- B-4: The fixed evaluation set and expert reviewers are not yet sourced.

---

## 12. Licensing and Compliance Notes

This PRD is a planning document. It does not implement the framework and grants no rights.

Per `../LICENSE.md`, implementing the framework, incorporating it into any model, or offering any API or SaaS based on it requires a signed commercial license from the author. This PRD changes none of that.

Any future build following this PRD must first clear blocker B-3 and remain within the license.

All proposed schemas, architectures, and examples in this PRD remain subject to the existing repository license and do not place the material in the public domain.

---

## 13. Traceability to Source Documents

| PRD element | Source |
| --- | --- |
| Three metrics, primary directive | `candid_ai_prior_art.md`, section "The Framework: Three-Metric Honest Output"; `README.md` |
| Four-stage reasoning | `candid_ai_prior_art.md`, section "Reasoning Architecture" |
| Differentiation claims | `candid_ai_prior_art.md`, sections "Differentiation from Prior Art" and "Differentiation from Current Market Products" |
| Proprietary constraints | `LICENSE.md` |
| Everything in Sections 4 through 10 marked `[PROPOSAL]` or `[ASSUMPTION]` | Introduced by this PRD; not present as implementation in the source repo |

End of DRAFT PROPOSAL. No implementation, tests, or packages are created or implied by this document.
