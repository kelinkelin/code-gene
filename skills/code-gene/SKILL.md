---
name: code-gene
description: >
  Code generation discipline for coding, modification, bugfix, refactor, and
  PR/diff review tasks. Forces read-before-write, evidence before assumptions,
  convention before invention, minimal implementation, and verified delivery.
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
2. **Resolve References**: load every user-requested reference and enough local
   project samples to prove the intended style before planning edits.
   Use `references/project-reference-selection.md` for explicit references,
   "like X" requests, author/style requests, and project-pattern discovery.
3. **Plan**: resolve ambiguity, identify the main path, search for existing
   patterns, choose the smallest project-consistent change.
4. **Implement or Review**: edit only what the task needs, or review the diff
   with evidence and maintainer impact. Use references below when the task
   touches that area.
5. **Verify**: run the appropriate checks, or report why they cannot run.
6. **Deliver**: explain the old flow and new flow when behavior changes; include
   code references, verification result, and any uncertainty.

## Mandatory Reference Resolution

Before writing or changing code, resolve references explicitly:

- If the user names a file, module, feature, person, author, prior flow, or says
  "参考", "like", "类似", "照着", "全量 reference", or "全量 reference 改",
  treat that as a mandatory reference request.
- Load the named reference, or search the repo for the closest real
  implementation. If it cannot be found, stop and state the missing reference
  instead of silently substituting a guess.
- Read the target file to EOF before editing it, then read nearby files in the
  same package/layer and relevant call sites/tests/models.
- Before substantial edits, give a short reference coverage note: requested
  references, loaded references, local project samples, and any gaps.

## Reference Map

Load only the references needed for the task; never preload every reference just
because it exists:

- PR, diff, CR, or review task: read `references/review.md`.
- Implementation, refactor, over-design risk, helper placement, or comments:
  read `references/code-style.md`.
- Business logs, RPC/HTTP/MQ/DB calls, failure diagnosis, or traceability:
  read `references/logging.md`.
- Java backend naming, cache, DTO, RPC client/service, constants, or DongBoot /
  7Fresh conventions: read `references/java-backend.md`.
- User-requested references, "like X" implementation requests, author/style
  matching, project-pattern discovery, or complaints that references were
  missed: read `references/project-reference-selection.md`.
- Delivery explanation, external source provenance, workspace safety, or tool
  discipline: read `references/delivery-tooling.md`.

## Pre-flight Checklist

- Relevant code, signatures, call sites, data models, and tests were read.
- User-requested references were either loaded or reported as missing.
- Target files were read to EOF before edit placement decisions.
- Same-layer or author/style examples were sampled when project convention
  matters.
- Ambiguity is resolved or explicitly asked.
- Existing project patterns were searched before adding new code.
- The planned change is minimal and tied to the request.
- Required reference files above were loaded for touched areas.
- Verification command is known, or the blocker is known.

Write or change code only after this checklist is clean.
