---
name: new-change
description: Run the full reusable engineering-change workflow for a scoped fix or feature in a convention-heavy repository—understand, isolate Git context, discover conventions, plan, implement the smallest defensible change, strengthen tests, validate guardrails, review the diff, and produce a multi-role handoff. Use when the user asks to fix, implement, enhance, refactor, or otherwise change repository code/behavior; pastes a GitHub issue URL or issue number; supplies explicit change params; or invokes /new-change.
---

# Engineering Change

## Architectural constraints

Project Rules define non-negotiable policy. This Skill defines the procedure for executing an engineering change. Do not duplicate rule content unless necessary to explain when it applies.

Treat the current repository as the source of truth for conventions. Do not hardcode FastAPI-specific implementation knowledge.

Do not begin implementation until repository discovery and the implementation plan are complete, and the user has explicitly approved the PLAN.

Prefer the smallest defensible change and explicitly report uncertainty rather than guessing.

## When this skill applies

Apply for scoped engineering changes (fix, feat, enhancement, refactor, docs, test, chore). Defer to Project Rules for policy; this skill only sequences the work and required outputs.

## Inputs

Accept **either** of these invocation forms (or both, with explicit params overriding the issue where they conflict):

### A) GitHub issue link or issue number

User provides a GitHub issue as either a full URL or an issue number, for example:

- `https://github.com/<owner>/<repo>/issues/<n>` (full link)
- `<n>` or `#<n>` (issue number; resolved against the current repository)

Then:

1. If the input is an issue number (`<n>` or `#<n>`), not a full URL:
   - Determine the repository from the current Git remote.
   - Resolve the number against that repository.
   - Record the resolved repository and issue URL in the PLAN.
   - If the repository cannot be determined reliably, report the problem instead of guessing.
2. If the input is a full issue URL, use that URL/repository directly and record it in the PLAN.
3. Retrieve the issue from GitHub using an approved available tool. Prefer the GitHub CLI when it is configured for the current repository.
   - Example: `gh issue view <n> --json number,title,body,labels,assignees,url`
   - Example: `gh issue view <url> --json number,title,body,labels,assignees,url`
   - Do not make the Skill dependent on `gh` being installed.
4. Map issue fields into request params:
   - **Goal** ← title (+ short summary of body)
   - **Constraints / acceptance** ← body checklists, “Acceptance criteria”, “Requirements”, or equivalent sections when present
   - **Out of scope** ← explicit “Out of scope” / “Non-goals” sections when present
   - **Change type hint** ← labels such as `bug`, `enhancement`, `documentation` when present (still re-classify per `01-change-isolation`)
   - **Source** ← issue URL/number (carry through PLAN and FINAL HANDOFF)
5. If the issue cannot be fetched (auth, wrong repo, missing issue), stop and report what failed; do not invent issue content.

### B) Explicit params

Normalize direct user input into:

- `goal` — required — what should be achieved
- `impact` — optional — user/business reason
- `change_type` — optional hint; still classify using repository conventions
- `constraints` — optional — must-haves and requirements
- `out_of_scope` — optional — explicit exclusions
- `acceptance` — optional — how success will be judged
- `source` — optional — issue URL, issue number, ticket ID, or other source reference

If neither a usable issue (link or number) nor a `goal` is available, ask for one of the two input forms before continuing.

## Workflow

Execute in order. Do not skip ahead.

### 1. Understand the request

- Resolve inputs using **Inputs** above (issue link, issue number, and/or explicit params).
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

After presenting the PLAN, **ask the user whether to proceed** with the proposed change. Do not begin implementation until they explicitly approve (e.g. proceed / yes / approved). If they decline or request changes, revise the PLAN or stop as they direct.

### 6. Implement the smallest defensible change

- Only start this step after explicit user approval of the PLAN.
- Follow Project Rule `00-operating-model` for implementation scope.
- Change only what the plan requires.
- Follow Project Rule `02-documentation-updates` when documented behavior/APIs/examples are impacted.



### 7. Strengthen tests

- Add or update tests that would have failed before the change.
- Prefer the repository’s existing test entrypoints and patterns.
- Run the relevant tests and **show the user the status** of what ran:
  - commands
  - passed (counts / suites)
  - failed (test names + short error summary)
  - use `Failed: none` when clean
- **If any test fails: stop.** Do not continue to PR creation or FINAL HANDOFF as success. Tell the user tests are not passing, include the failure details, and **suggest a fix**. Ask what they want to do next. Do not weaken, delete, or skip failing tests unless they explicitly direct that.



### 8. Validate against repository guardrails

- Run the lint/type/test/docs checks the repository already defines for this kind of change.
- Show the user pass/fail status the same way as tests (commands + results).
- **If any required guardrail fails: stop**, report the failure, suggest a fix, and ask what to do next.
- Do not weaken guards to land the change unless the user explicitly directs that.



### 9. Review the final diff

- Only proceed here after reported tests and required guardrails have passed (or the user explicitly waived a failure).
- Re-read the diff for correctness, scope creep, missing tests/docs, and secret leakage.
- Confirm it matches the plan and Project Rules.



### 10. Open PR only when tests passed

- **Create or update a PR only if all required tests and guardrails reported in steps 7–8 passed.**
- If anything failed and was not explicitly waived by the user, do not open/update a PR.

### 11. Produce FINAL HANDOFF (required checkpoint)

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

`Tests run` / `Passed` / `Failed` must reflect the actual commands and outcomes from steps 7–8 (use `Failed: none` when clean).



## Notes

- If uncertainty remains after discovery, state it in PLAN and handoff rather than guessing.
- Keep the skill procedural; put lasting policy in Project Rules, not here.
- Project Rules define non-negotiable policy. This Skill owns task ingestion, normalization, SDLC orchestration, and structured outputs. Do not duplicate Project Rule content unnecessarily. Do not add FastAPI-specific implementation knowledge.

