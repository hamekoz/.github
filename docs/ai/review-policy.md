# Human Review Policy — AI-Generated Changes

Any code generated or modified by an AI agent **requires human review and approval** before being
merged to protected branches.

---

## Governing principle

> AI agents are assistants, not decision-makers. A human is always responsible for the code that
> enters the repository.

---

## Mandatory process

```text
Task requested from the agent
        │
        ▼
Agent generates changes on a working branch
with a semantic prefix (feature/, fix/, docs/, ...)
        │
        ▼
PR opened by the agent or the requester
        │
        ▼
CI runs all automated checks
        │
        ▼
Mandatory human review ──► observations ──► agent iterates
        │
        ▼ (approved)
Merge to base branch
        │
        ▼
Log the task in the repository AI task log
```

---

## Review criteria by category

### Business logic changes

- [ ] Is the implemented logic correct with respect to the requirement?
- [ ] Are the relevant edge cases handled?
- [ ] Are there tests covering the new behavior?
- [ ] Is the implementation consistent with the existing design?

### Architecture changes

- [ ] Are the Clean Architecture layers respected?
- [ ] Do dependencies point in the correct direction?
- [ ] Were no circular dependencies introduced?

### CI/CD or configuration changes

- [ ] Is the effect of each workflow change understood?
- [ ] Was the level of verification of the pipeline not reduced?
- [ ] Are no secrets or sensitive variables exposed?

### Dependency changes

- [ ] Is the new dependency justified?
- [ ] Was it verified it has no known CVEs?
- [ ] Is the version appropriately pinned?

### Security (mandatory for any change)

- [ ] No secrets, tokens, or credentials hardcoded?
- [ ] Are all external inputs validated?
- [ ] Were no known vulnerabilities introduced?

---

## Required approvals

| Change type | Minimum approvals |
|---|---|
| Documentation / comments | 1 |
| Tests, CI | 1 |
| Application code | 1 |
| Architecture or design changes | 2 |
| Security or authentication changes | 2 (one must be a maintainer) |
| Shared workflow changes (`.github`) | 2 maintainers |

---

## What the reviewer must add when approving

In the approval comment or merge description, the reviewer must confirm:

- That they reviewed the full diff.
- That the applicable criteria from the list above were verified.
- If there is technical debt or necessary follow-up, create a linked issue.

---

## When to reject an AI-generated PR

- The code does not follow organization conventions.
- Tests do not pass or do not exist for the new behavior.
- The change exceeds the requested scope (the agent modified more than asked).
- There are reasonable security doubts that were not resolved.
- The logic is correct but the design is not maintainable.

---

## Logging the review

After merging any AI-generated changes, update the repository's [AI task log](./ai-task-log-template.md)
with the review result.