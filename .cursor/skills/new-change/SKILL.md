---
name: new-change
description: Run the full reusable engineering-change workflow for a scoped fix or feature in a convention-heavy repository—understand, isolate Git context, discover conventions, plan, obtain explicit user approval, implement the smallest defensible change, strengthen tests, validate guardrails, show results in a table, obtain approval to draft the PR, publish only after validation passes, and produce a multi-role handoff. Use when the user asks to fix, implement, enhance, refactor, or otherwise change repository code/behavior; pastes a GitHub issue URL; supplies explicit change params; or invokes /new-change.
---

# Engineering Change

## Architectural constraints

Project Rules define non-negotiable policy. This Skill defines the procedure for executing an engineering change. Do not duplicate rule content unless necessary to explain when it applies.

Treat the current repository as the source of truth for conventions. Do not hardcode FastAPI-specific implementation knowledge.

Do not begin implementation until repository discovery and the implementation plan are complete, **and the user has explicitly approved proceeding**.

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

1. For `#N` references:
   - Determine the repository from the current Git remote.
   - Resolve `#N` against that repository.
   - Record the resolved repository and issue URL in the PLAN.
   - If the repository cannot be determined reliably, report the problem instead of guessing.
2. Retrieve the issue from GitHub using an approved available tool. Prefer the GitHub CLI when it is configured for the current repository.
   - Example: `gh issue view <n> --json number,title,body,labels,assignees,url`
   - Do not make the Skill dependent on `gh` being installed.
3. Map issue fields into request params:
   - **Goal** ← title (+ short summary of body)
   - **Constraints / acceptance** ← body checklists, “Acceptance criteria”, “Requirements”, or equivalent sections when present
   - **Out of scope** ← explicit “Out of scope” / “Non-goals” sections when present
   - **Change type hint** ← labels such as `bug`, `enhancement`, `documentation` when present (still re-classify per `01-change-isolation`)
   - **Source** ← issue URL/number (carry through PLAN and FINAL HANDOFF)
4. If the issue cannot be fetched (auth, wrong repo, missing issue), stop and report what failed; do not invent issue content.

### B) Explicit params

Normalize direct user input into:

- `goal` — required — what should be achieved
- `impact` — optional — user/business reason
- `change_type` — optional hint; still classify using repository conventions
- `constraints` — optional — must-haves and requirements
- `out_of_scope` — optional — explicit exclusions
- `acceptance` — optional — how success will be judged
- `source` — optional — issue URL, issue number, ticket ID, or other source reference

If neither a usable issue nor a `goal` is available, ask for one of the two input forms before continuing.

## Workflow

Execute in order. Do not skip ahead.

### 1. Understand the request

- Resolve inputs using **Inputs** above (issue link and/or explicit params).
- Restate goal, constraints, and out of scope (cite `source` when from an issue).
- Identify user/business impact and likely subsystems.
- Follow Project Rule `00-operating-model` for context boundaries and understanding requirements.



### 2. Establish safe Git context

- Follow Project Rule `01-change-isolation` before modifying files (inspect status/branch, classify type).
- Decide the intended dedicated branch name using repository conventions.
- Defer creating/checking out that branch until after the user approves the PLAN (still create it before any implementation edits).



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
- If context is incomplete, make safe repository-grounded assumptions when possible and record them under Risks / uncertainties. Ask only when the missing information would materially alter the implementation or create meaningful risk.



### 5. Produce PLAN (required checkpoint)

Emit the plan **before any implementation edits**, using this exact structure:

```markdown
## PLAN
- Source:
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

Do not begin implementation until this plan is complete **and the user has approved it**.

### 6. Confirm with the user (required checkpoint)

After emitting the PLAN, **stop and ask the user whether to proceed** with implementation.

- Ask clearly (for example: whether to proceed as planned, revise the plan, or abort).
- Do not create the implementation branch or edit files until the user explicitly approves proceeding.
- Approval unlocks implementation only; commit/push/PR remain gated by successful validation and a later publish confirmation (steps 8–11).
- If the user requests plan changes, update the PLAN, re-emit it, and ask again.
- If the user declines or aborts, stop without implementing and record that outcome briefly.

### 7. Implement the smallest defensible change

- Only after explicit user approval from step 6: create/checkout the dedicated branch (per step 2 / `01-change-isolation`), then implement.
- Follow Project Rule `00-operating-model` for implementation scope.
- Change only what the plan requires.
- Follow Project Rule `02-documentation-updates` when documented behavior/APIs/examples are impacted.



### 8. Strengthen tests

- Add or update tests that would have failed before the change.
- Prefer the repository’s existing test entrypoints and patterns.
- After running tests, **report results to the user in a markdown table** before continuing. Use one row per command/suite (and per failed test when useful):

```markdown
| Command | Scope | Result | Detail |
| --- | --- | --- | --- |
| `pytest …` | `tests/…` | pass / fail / skip | counts or failure summary |
```

- Include passed counts/suites, failed test names + short error summary, and skipped/xfailed only if relevant.
- **If any test fails: stop.** Do not continue to broader validation, review, handoff, commit, push, or PR. Ask the user what to do next (fix, extend scope, accept failure, abort, etc.).
- Do not weaken, delete, or skip failing tests to proceed unless the user explicitly directs that.



### 9. Validate against repository guardrails

- Run the lint/type/test/docs checks the repository already defines for this kind of change.
- Report pass/fail outcomes to the user in the **same tabular format** as tests (add rows or a second table for lint/type/docs).
- **If any required guardrail fails: stop** and ask the user what to do next. Do not continue the workflow until they decide.
- Do not weaken guards to land the change unless the user explicitly directs that.



### 10. Review the final diff

- Only proceed here after reported test and guardrail runs have passed (or the user explicitly waived a failure).
- Re-read the diff for correctness, scope creep, missing tests/docs, and secret leakage.
- Confirm it matches the plan and Project Rules.



### 11. Confirm publish / draft PR (required checkpoint)

- **Do not `git commit`, `git push`, or open/update a PR until steps 8–10 are complete and required tests/guardrails have passed** (or the user explicitly waived a failure).
- Local implementation edits may exist in the working tree before that; they must not be published early.
- Before publishing, show the user:
  1. The **full results table** from steps 8–9 (all commands/suites and pass/fail status)
  2. A short note that validation is clean (or which failures were explicitly waived)
- Then **stop and ask** whether to draft the PR (commit + push + open/update draft PR), commit/push without a PR, revise, or abort.
- Only after the user explicitly approves drafting/publishing: commit, push, and open/update the PR as they directed.



### 12. Produce FINAL HANDOFF (required checkpoint)

Emit the handoff using this exact structure:

```markdown
## FINAL HANDOFF
- Source:

### Developer
- What changed / where / why:
- Follow-ups / debt:

### QA
- Tests run:
- Passed:
- Failed:
- Results table:
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

`Tests run` / `Passed` / `Failed` / `Results table` must reflect the actual commands and outcomes from steps 8–9 (use `Failed: none` when clean; use `n/a` only when no repository test/guardrail applies to the change, and say why). Prefer pasting the same markdown table shown before the publish confirmation.



## Notes

- If uncertainty remains after discovery, state it in PLAN and handoff rather than guessing.
- Never skip the post-PLAN confirmation checkpoint; approval must be explicit before implementation.
- Never publish (commit/push/PR) before required tests and guardrails have passed, unless the user explicitly waives a failure.
- Never open or update a PR until the user has seen the results table and explicitly approved drafting/publishing.
- Keep the skill procedural; put lasting policy in Project Rules, not here.
- Project Rules define non-negotiable policy. This Skill owns task ingestion, normalization, SDLC orchestration, and structured outputs. Do not duplicate Project Rule content unnecessarily. Do not add FastAPI-specific implementation knowledge.

