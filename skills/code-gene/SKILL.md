---
name: code-gene
description: >
  Code generation discipline for coding, modification, bugfix, refactor, and
  PR/diff review tasks. Forces read-before-write, evidence before assumptions,
  automatic project reference/style discovery plus auditable reference
  application before edits, convention before invention, minimal
  implementation, and verified delivery.
  Use when the user invokes /code-gene, says "use code-gene", asks for strict
  code generation discipline, or wants the code-gene checklist applied.
---

# Code Gene Skill

## What It Is

`code-gene` turns personal engineering discipline into a reusable workflow.
It is not a giant style guide loaded every time. This file is the thin entry:
trigger rules, core principles, execution flow, and a reference map.

## Activation

Active when the user says `/code-gene`, `use code-gene`, asks to apply
`code-gene`, or requests code generation/modification/review under strict
discipline. Stay active until `stop code-gene` or `normal mode`.

## Core Principles

1. Read real code before advising or editing: implementation, signatures, call
   sites, data models, tests, and project conventions.
2. Confirm unclear business rules; do not invent domain behavior or API shape.
3. Reuse existing utilities, services, DTOs, and patterns before creating new
   abstractions.
4. Keep the main path short and current-requirement driven; no speculative
   branches, fields, configs, helpers, or architecture.
5. Match project layering, naming, logging, comments, error handling, and file
   layout.
6. Verify with build/test/typecheck/lint/UI where applicable; if blocked, state
   the exact command and blocker.
7. Preserve user-owned changes and avoid destructive git/file operations unless
   explicitly requested.
8. Deliver concrete before/after behavior, evidence, verification, and residual
   risk instead of generic "done" summaries.

## Workflow

1. **Analyze**: inspect the relevant current code and worktree state. If outside
   sources guide the work, record provenance before relying on them.
2. **Resolve References**: load the core coding/review references, then
   automatically discover the project style and enough local samples to prove
   the intended implementation/review pattern. User-requested references raise
   priority; they are not required to trigger this step. Build the reference
   application ledger described below before planning.
3. **Plan**: resolve ambiguity, identify the main path, search for existing
   patterns, choose the smallest project-consistent change.
4. **Implement or Review**: edit only what the task needs, or review the diff
   with evidence and maintainer impact. Use references below when the task
   touches that area.
5. **Verify**: run the appropriate checks, or report why they cannot run.
6. **Deliver**: explain the old flow and new flow when behavior changes; include
   code references, verification result, and any uncertainty.

## Mandatory Reference Resolution

`code-gene` exists to write and review code in the project's own style.
Reference/style discovery is mandatory for every implementation or review task,
even when the user does not explicitly name a reference.

- Always load the core reference baseline before coding or review:
  `references/code-style.md`, `references/logging.md`,
  `references/java-backend.md`, `references/project-reference-selection.md`,
  and `references/delivery-tooling.md`.
- For coding tasks, read the target file to EOF, direct call sites/models, and
  2-4 same-layer or same-feature examples before planning edits.
- For review tasks, read the changed files, relevant call sites/contracts, and
  project examples needed to judge whether the diff follows local conventions.
- If the user names a file, module, feature, person, author, prior flow, or says
  "参考", "like", "类似", "照着", "全量 reference", or "全量 reference 改",
  load that reference first. If it cannot be found, stop and state what was
  searched instead of silently substituting a guess.
- Read the target file to EOF before editing it, then read nearby files in the
  same package/layer and relevant call sites/tests/models.
- Before substantial edits, give a short reference coverage note: requested
  references if any, automatically loaded project samples, and any gaps.

## Reference Application Gate

Loading a reference is not enough. Convert every loaded reference into an
explicit decision before editing or reviewing code.

1. Read every mandatory reference to EOF in the current task. Do not rely on a
   prior task's memory or claim a reference was loaded after reading only its
   heading or an excerpt.
2. Build a compact reference application ledger with one row per mandatory and
   scene-specific reference. Record:
   - `reference`: the file or user-provided source;
   - `applicable rules`: the concrete rules that affect this task;
   - `evidence`: target/project files proving how the rule applies locally;
   - `effect`: the implementation or review decision caused by the rule.
3. A reference may be marked `N/A` only when its domain is genuinely untouched.
   State the reason. For example, `java-backend.md` can be `N/A` for a pure
   TypeScript change, while `logging.md` is not `N/A` when changing an external
   call, failure path, or business-flow boundary.
4. Before substantial edits, report a concise coverage note naming all loaded
   mandatory references, scene-specific references, project samples, and gaps.
   The full ledger may stay internal unless the user asks for it.
5. If a mandatory or requested reference cannot be read, or local evidence is
   insufficient to resolve a conflict, stop before editing and report the gap.

The gate passes only when every loaded reference has either an evidenced effect
or an explicit `N/A` reason. Do not use `N/A` to bypass a rule that applies.

## Post-change Conformance Gate

After implementation or review and before delivery:

1. Re-read the final diff or reviewed lines against the reference application
   ledger.
2. Check every applicable rule for concrete evidence in the result. Fix any
   mismatch instead of merely mentioning it.
3. Run project verification appropriate to the touched behavior.
4. Report which references materially affected the result, which were `N/A`
   and why, verification performed, and unresolved risk.

Do not declare completion when reference loading, project sampling, diff
conformance, or verification is missing. If blocked, name the exact missing
evidence or command.

## Reference Map

For every code writing or code review task, load the core reference baseline
first. These are not conditional:

- `references/code-style.md`: formatting, helper placement, comments, edit
  placement, and anti-over-engineering.
- `references/logging.md`: log level, business traceability, exception and
  external-call logging.
- `references/java-backend.md`: naming, DTO/RPC/client/service/constants, cache,
  DongBoot and 7Fresh Java conventions.
- `references/project-reference-selection.md`: automatic local project
  reference/style discovery.
- `references/delivery-tooling.md`: workspace safety, verification, delivery
  explanation, and source provenance.

Then load scene-specific references as needed:

- PR, diff, CR, or review task: read `references/review.md`.
- User-requested docs, PRDs, screenshots, files, author/style examples,
  "like X" requests, or feature analogues: load those local/external references
  before planning.

## Pre-flight Checklist

- Relevant code, signatures, call sites, data models, and tests were read.
- Core reference baseline was loaded for implementation or review work.
- Automatic project style/reference samples were loaded for implementation or
  review work.
- User-requested references were either loaded or reported as missing.
- Target files were read to EOF before edit placement decisions.
- Same-layer or author/style examples were sampled when project convention
  matters.
- Ambiguity is resolved or explicitly asked.
- Existing project patterns were searched before adding new code.
- The planned change is minimal and tied to the request.
- Required reference files above were loaded for touched areas.
- Every loaded reference has an evidenced effect or an explicit valid `N/A`
  reason in the reference application ledger.
- Verification command is known, or the blocker is known.

Write or change code only after this checklist is clean.
