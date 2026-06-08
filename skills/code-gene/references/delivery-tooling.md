# Delivery and Tooling Discipline

## Delivery Explanation

After finishing code changes, explain the actual logic in concrete terms, not just "changed X" or "fixed Y".

- If the change alters an existing flow, describe the original flow and the new flow together so the user can see exactly where behavior changed.
- Prefer a compact Markdown table when the work changes business logic, request routing, data mapping, fallback behavior, persistence, caching, or external calls.
- Include the most relevant code references, verification result, and any blocked verification.
- Do not invent business impact. If impact is inferred from code, say it is inferred.

Recommended table shape:

| Area | Before | After | Impact |
| --- | --- | --- | --- |
| Data mapping | `fieldA` was not copied into `TargetParam` | Converter now copies `fieldA` before handler execution | Downstream handler can read the value in the same request |

For tiny mechanical edits, a short paragraph is enough, but still state the concrete logic that now runs.

## External Source Provenance

When using external repositories, documentation, articles, or examples to guide an implementation, record the provenance before relying on them:

- Source URL or repository path.
- Commit SHA, tag, release version, package version, or documentation version used.
- Access date when no stable version is available.
- Which specific interface, behavior, or rule was taken from that source.

Do not cite external guidance as fact without this provenance. If the source is mutable and no version can be pinned, state that uncertainty and verify against the local project before coding.

## Workspace and Tool Discipline

Before editing, inspect worktree state when git is available. Treat existing changes as user-owned unless you made them in this turn. Preserve unrelated dirty files, do not reformat or revert them, and never run destructive cleanup such as `git reset --hard`, `git checkout --`, or mass deletes unless the user explicitly requests that exact operation.

Manual edits must use structured patch/edit tools so the diff is narrow and auditable. Do not write source files with shell heredocs, redirection, `cat > file`, or similar tricks except for purely generated artifacts where the whole file is intentionally produced.

If a required build, test, dependency install, network fetch, credentialed command, GUI step, or outside-root write is blocked by sandbox or network restrictions, report the exact blocked command and reason, then request the needed approval/escalation. Do not invent an unverified workaround or mark verification complete.
