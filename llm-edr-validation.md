# Automated Engineering Drift Detection via an LLM-as-a-Judge Pattern

---

# Context

As an engineering ecosystem expands across multiple teams, maintaining alignment between approved **Engineering Decision Records (EDRs)** and the continuously evolving codebase has become increasingly difficult.

Engineering compliance is currently validated primarily during pull request (PR) reviews. This approach relies heavily on human reviewers remembering historical Engineering decisions while simultaneously reviewing implementation details. As engineering teams scale, this introduces several challenges:

- Engineering drift gradually accumulates unnoticed.
- PR reviews become slower due to increased reviewer cognitive load.
- Cross-team Engineering violations are frequently missed.
- Change failure rates increase because inconsistencies reach production.
- Engineering knowledge becomes concentrated among senior reviewers.

We require an automated and scalable mechanism that continuously validates incoming code changes against approved Engineering decisions before code is merged.

---

# Decision

Implement an **LLM-as-a-Judge** architecture validation pipeline integrated directly into the Continuous Integration (CI) workflow.

During every Pull Request:

1. Identify modified source files.
2. Determine the relevant EDRs for the affected components.
3. Provide the code diff, Engineering context, and EDR documentation to an LLM.
4. Ask the model to evaluate whether the implementation complies with the approved architecture.
5. Produce a structured compliance report including:
    - Compliance verdict
    - Violated EDR sections (if any)
    - Explanation
    - Confidence score
    - Recommended remediation
6. Block PR merges when Engineering violations exceed an agreed threshold.

The LLM acts as an automated Engineering reviewer rather than replacing human engineering judgment.

---

# Options Considered

## Option 1 — LLM-as-a-Judge Automated CI Process (Chosen)

Integrate an LLM into the CI pipeline to evaluate code changes against EDRs during every Pull Request.

### Pros

- Detects Engineering drift before code reaches production.
- Significantly reduces reviewer cognitive load.
- Shortens Pull Request review cycles.
- Understands semantic Engineering intent instead of relying solely on syntax.
- Automatically adapts as EDR documentation evolves.
- Eliminates the need to continuously rewrite architecture-specific validation rules.

### Cons

- Introduces LLM API costs.
- Adds CI execution latency.
- Requires ongoing prompt engineering.
- May produce false positives or hallucinations if prompts are poorly designed.
- Depends on external AI service availability unless self-hosted.

---

## Option 2 — Static Analysis and Custom Rule Engines

Examples include:

- ArchUnit
- Open Policy Agent (OPA)
- Custom Linters
- Static Architecture Validation Rules

### Pros

- Fully deterministic.
- No hallucination risk.
- Executes locally.
- No external API costs.
- Easy to audit.

### Cons

- High maintenance burden.
- Every EDR update requires corresponding rule changes.
- Poor at interpreting Engineering intent expressed in natural language.
- Difficult to model abstract Engineering patterns.

---

## Option 3 — Manual Architecture Reviews

Continue relying on senior engineers and staff engineers to review all Pull Requests for Engineering compliance.

### Pros

- Deep contextual understanding.
- Engineering judgment.
- Can interpret exceptions and business context.

### Cons

- Poor scalability.
- Creates review bottlenecks.
- High reviewer fatigue.
- Engineering inconsistencies are easily overlooked.
- Review quality varies by reviewer experience.

---

# Decision Drivers

The selected solution prioritizes:

- Early detection of Engineering drift
- Reduced reviewer cognitive load
- Faster Pull Request turnaround
- Continuous Engineering governance
- Scalability across multiple engineering teams
- Semantic understanding of architecture rather than rigid rule matching

---

# DORA Metrics Impact

| DORA Metric | Impact | Justification |
|-------------|--------|---------------|
| **Change Failure Rate (CFR)** | ✅ Positive | Detects Engineering violations before merge, reducing production defects caused by structural inconsistencies. |
| **Lead Time for Changes** | ✅ Positive | Automates Engineering review, reducing PR waiting time and accelerating merge velocity. |
| **Deployment Frequency** | ⚪ Neutral / Indirect | Removes architecture review bottlenecks, enabling smoother delivery without directly affecting deployment automation. |
| **Mean Time to Restore (MTTR)** | ➖ No Direct Impact | Preventative control executed before deployment; does not affect operational incident recovery. |

---

# Consequences

## Positive

- Continuous Engineering governance.
- Reduced Engineering drift.
- Faster engineering feedback loops.
- Lower reviewer fatigue.
- More consistent architecture enforcement.
- Improved Developer Experience (DX).
- Better onboarding through machine-assisted Engineering explanations.
- Architecture becomes executable rather than passive documentation.

## Negative

- Increased CI execution time.
- Additional LLM operational costs.
- Requires prompt maintenance.
- Requires monitoring for AI quality and false positives.
- Potential dependency on third-party AI services.

---

# Implementation Overview

```text
Developer
     │
     ▼
Create Pull Request
     │
     ▼
CI Pipeline
     │
     ├── Detect modified files
     │
     ├── Locate relevant EDRs
     │
     ├── Retrieve architecture documentation
     │
     ├── Generate evaluation prompt
     │
     ├── Invoke LLM
     │
     ├── Produce compliance report
     │
     └── Pass / Fail PR
```

---

# Risks

| Risk | Mitigation                                                                        |
|------|-----------------------------------------------------------------------------------|
| Hallucinated violations | Structured prompts, few-shot examples, confidence thresholds, and human override. |
| False positives | Maintain evaluation prompts and continuously tune with real PR feedback.          |
| API availability | Retry logic, graceful degradation, or self-hosted LLM deployment.                 |
| Increased CI latency | Cache EDR embeddings, limit context size, and parallelize evaluation.             |
| Cost growth | Evaluate only impacted EDRs and optimize token usage.                             |

---

# Success Metrics

The solution will be considered successful if it achieves measurable improvements in:

- Reduced Pull Request review time
- Lower Change Failure Rate
- Reduced Engineering drift incidents
- Faster Engineering compliance feedback
- Improved developer satisfaction
- Increased consistency across engineering teams

---
