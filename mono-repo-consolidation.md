# Consolidate Multi-Domain Data Engineering Platform Repositories into a Single Mono-Repo

---

## 1. Context

A **common data-platform template** — pipelines, environment configs, deployment manifests, and extension points — that gets deployed once per business domain. Today, each domain team keeps its own full copy of this template in its own repository:

| Example repo | Example domain | Notes |
| :--- | :--- | :--- |
| `repo-DomainOne` | DomainOne | |
| `repo-DomainTwo` | DomainTwo | |
| `repo-DomainThree` | DomainThree & Compliance | |
| `repo-DomainFour` | DomainFour | Internally already hosts 5 separate sub-projects (`source-a` ... `source-e`) side by side in one repo |

Each team also owns other data-engineering codebases, for other products/sources, that live outside these four repos entirely.

All four repos start from the *same template*. This DR asks a simple question: **should they be consolidated into one repository, and if so, how?**

---

## 2. The Problem

Looking across the four existing repos, the same pattern shows up again and again: a capability gets copy-pasted per domain instead of shared once, and the copies quietly drift apart. A few representative examples:

None of this is about individual mistakes. It's the predictable result of having no single place for shared code or shared config to live, and no automated way to notice when copies diverge.

---

## 3. Decision Drivers

1. **DomainThree** — duplicated domain-model logic means a correctness fix has to land in up to 7–9 places, with no coordination between them.
2. **Silent drift** — "identical" capabilities already disagree across repos, and nothing catches it except a manual review like the one behind this DR.
3. **Delivery speed** — lead time, failure rate, and review time for any cross-cutting change multiply with the number of repos that must separately branch, review, and release it.
4. **Planned shared tooling needs this** — planned investments in shared libraries and platform-wide automation assume it's cheap to change all domains at once. Today, it isn't.
5. **Scaling** — each team has further codebases outside these four. Whatever we choose should generalize, or we'll re-litigate this every time a new repo shows up.
6. **Developer experience** — engineers can't discover or reuse a sibling team's solution to the same problem without separate repo access and a separate clone.

---

## 4. Options Considered

| Option | Description | Fixes platform/config duplication? | Fixes code duplication? | Enables platform-wide tooling? | Migration effort           |
| :--- | :--- | :---: | :---: | :---: |:---------------------------|
| **A — Status quo** | Keep every repo independent, no coordination | ❌ | ❌ | ❌ | None                       |
| **B — Shared packages only** | Keep separate repos; publish shared Python code as versioned internal packages | ❌ |  | ❌ | Low                        |
| **C — Submodules / subtrees** | Keep separate repos; link a shared-code repo via git submodule or subtree | ⚠️ Partial |  | ❌ | Medium (ongoing sync pain) |
| **D — Full mono-repo** | One repo: one folder per domain, one folder for shared code |  |  |  | High (one-time)            |
| **E — Phased mono-repo** <br>*✔️ chosen* | Same end state as D, migrated in waves, deployment behaviour unchanged |  |  |  | High, de-DomainThree       |

**Why not B or C alone:** both only fix *importable code*. Neither touches duplicated pipeline definitions, job templates, CI scripts, or the version/config drift.

---

## 5. Decision

Consolidate into a single mono-repo, migrated in phases, without changing how each domain is deployed.

### 5.1 Phasing

* **Wave 1** — migrate the four known repos. Start with whichever is already closest to the target shape (e.g., a repo that already hosts multiple sub-projects side by side) as the lowest-risk proof point.
* **Wave 2** — inventory and migrate each team's other, currently-separate codebases, once Wave 1 proves out the mechanics (CI, access control, history migration).

---

## 6. Consequences

### Good

* One PR fixes a shared bug once, for everyone — instead of 4+ separate PRs.
* Drift (version mismatches, config-shape differences) becomes an automated check instead of a manual audit.
* Whole-platform code search — any engineer can find and reuse a sibling domain's implementation.
* One canonical contributor guide — no more "copied from another repo and never corrected."

### Costs

* Real migration effort: history needs merging or careful importing, repo by repo.
* Access control shifts from "repo = permission boundary" to path-scoped branch policies/code owners — a process change, not just a technical one.
* **Co-location alone doesn't prevent duplication.** One of our repos already hosts multiple sub-projects side by side today, and one shared framework has still drifted to 46% similarity between two of its own internal copies. The mono-repo has to ship *together with* shared-package extraction and a CI drift check, or duplication just continues inside one repo instead of across four.
* CI must be path-scoped from day one, or every PR ends up running every domain's full pipeline.
* Wave 2 scope and timeline are unknown until those other repos are inventoried.

### Unchanged

* Each domain's deployment cadence, environment progression, and platform identity stay the same.
* Domain-specific business logic stays domain-owned; only genuinely shared capability kept in common place.

---

## 7. Risks & Mitigations

| Risk | Mitigation                                                                                        |
| :--- |:--------------------------------------------------------------------------------------------------|
| Migration loses git history/blame | Trial the merge on a throwaway clone first; measure the result before committing to a method      |
| Teams feel they've lost autonomy | Path-scoped code owners + branch policies preserve each team's existing approval gate             |
| Mono-repo becomes a bottleneck | Federated ownership: core team owns generic/common, domains contribute via PR + code-owner review |
| CI runtime balloons | Path-filtered pipeline triggers per domain; only generic/common changes trigger a full-platform run  |
| Big-bang migration freezes all teams at once | Migrate one domain first, validate the mechanics, then proceed with the rest                      |
| Late-discovered repos expand scope unexpectedly | Explicit inventory task before Wave 2 is scheduled                                                |

---

## 8. Open Questions

1. Inventory each team's other, currently-separate codebases to scope Wave 2.
2. Confirm our CI/deployment tooling supports path-scoped, multi-domain triggering from a single repo.
3. Decide the migration mechanism (history-preserving merge vs. squash-import) and pilot it on one domain.
4. Confirm the replacement access-control model with whoever owns repo permissions today.
5. Confirm funding/staffing for the migration itself, separate from the shared-tooling work it unblocks.
