---
name: new-change
description: Run the full reusable engineering-change workflow for a scoped fix or feature in a convention-heavy repository—understand, isolate Git context, discover conventions, plan, implement the smallest defensible change, strengthen tests, validate guardrails, review the diff, and produce a multi-role handoff. Use when the user asks to fix, implement, enhance, refactor, or otherwise change repository code/behavior; pastes a GitHub issue URL; supplies explicit change params; or invokes /new-change.
---

# Engineering Change

## Architectural constraints

Project Rules define non-negotiable policy. This Skill defines the procedure for executing an engineering change. Do not duplicate rule content unless necessary to explain when it applies.

Treat the current repository as the source of truth for conventions. Do not hardcode FastAPI-specific implementation knowledge.

Do not begin implementation until repository discovery and the implementation plan are complete.

Prefer the smallest defensible change and explicitly report uncertainty rather than guessing.

## When this skill applies

Apply for scoped engineering changes (fix, feat, enhancement, refactor, docs, test, chore). Defer to Project Rules for policy; this skill only sequences the work and required outputs.

## Inputs

Accept **either** of these invocation forms (or both, with explicit params overriding the issue where they conflict):

### A) GitHub issue link

User pastes an issue URL or `#N` reference, for example:

- `https://github.com/<owner>/<repo>/issues/<n>`
- `#<n>` (resolved against the current repository)

Then:

1. Fetch the issue with the GitHub CLI when available (`gh issue view <n> --json number,title,body,labels,assignees,url` or the full URL).
2. Map issue fields into request params:
   - **Goal** ← title (+ short summary of body)
   - **Constraints / acceptance** ← body checklists, “Acceptance criteria”, “Requirements”, or equivalent sections when present
   - **Out of scope** ← explicit “Out of scope” / “Non-goals” sections when present
   - **Change type hint** ← labels such as `bug`, `enhancement`, `documentation` when present (still re-classify per `01-change-isolation`)
   - **Source** ← issue URL/number (carry through PLAN and FINAL HANDOFF)
3. If the issue cannot be fetched (auth, wrong repo, missing issue), stop and report what failed; do not invent issue content.

### B) Explicit params

User defines the change directly (chat text or structured fields). Normalize into:

| Param | Required | Meaning |
| --- | --- | --- |
| `goal` | yes | What to achieve |
| `impact` | no | User/business why |
| `change_type` | no | Hint only; still classify per repo/`01-change-isolation` |
| `constraints` | no | Must-haves / acceptance criteria |
| `out_of_scope` | no | Explicit exclusions |
| `acceptance` | no | How success will be judged |
| `source` | no | Issue URL/`#N` if this supplements a link |

If neither a usable issue nor a `goal` is available, ask for one of the two input forms before continuing.

## Workflow

Execute in order. Do not skip ahead.

### 1. Understand the request

- Resolve inputs using **Inputs** above (issue link and/or explicit params).
- Restate goal, constraints, and out of scope (cite `source` when from an issue).
- Identify user/business impact and likely subsystems.
- Follow Project Rule `00-operating-model` for context boundaries and understanding requirements.



### 2. Establish safe Git context

- Follow Project Rule `01-change-isolation` before modifying files (inspect status/branch, classify type, create/checkout a dedicated branch).



### 3. Discover repository conventions

Inspect the current repo only (do not assume stack-specific knowledge):

- Contribution / development docs
- Lint, test, and CI entrypoints
- Nearby modules and tests that match the change surface
- Docs/examples tied to the affected behavior
- Existing branch/commit/PR conventions when present

Record what you found and what remains uncertain.

### 4. Investigate the change surface

- Locate relevant code, tests, docs, and CI.
- Note public surfaces, failure modes, and adjacent callers.
- Stop and ask only if missing context blocks a defensible plan.



### 5. Produce PLAN (required checkpoint)

Emit the plan **before any implementation edits**, using this exact structure:

```markdown
## PLAN
- Goal:
- User/business impact:
- Change type:
- Repository conventions discovered:
- Relevant code/tests/docs/CI:
- Proposed files:
- Implementation approach:
- Test strategy:
- Documentation impact:
- CI/deployment impact:
- Risks / uncertainties:
```

Do not begin implementation until this plan is complete.

### 6. Implement the smallest defensible change

- Follow Project Rule `00-operating-model` for implementation scope.
- Change only what the plan requires.
- Follow Project Rule `02-documentation-updates` when documented behavior/APIs/examples are impacted.



### 7. Strengthen tests

- Add or update tests that would have failed before the change.
- Prefer the repository’s existing test entrypoints and patterns.



### 8. Validate against repository guardrails

- Run the lint/type/test/docs checks the repository already defines for this kind of change.
- Fix failures you introduced; do not weaken guards to land the change.



### 9. Review the final diff

- Re-read the diff for correctness, scope creep, missing tests/docs, and secret leakage.
- Confirm it matches the plan and Project Rules.



### 10. Produce FINAL HANDOFF (required checkpoint)

Emit the handoff using this exact structure:

```markdown
## FINAL HANDOFF
### Developer
- What changed / where / why:
- Follow-ups / debt:

### QA
- How to verify:
- Cases worth exercising:
- Known gaps:

### DevOps
- Deploy / config / CI impact:
- Rollback notes:

### PM
- User-facing outcome:
- Status:
- Residual risk:
```



## Notes

- If uncertainty remains after discovery, state it in PLAN and handoff rather than guessing.
- Keep the skill procedural; put lasting policy in Project Rules, not here.

