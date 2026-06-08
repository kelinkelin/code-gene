# Review Mode

When reviewing a PR, diff, or existing code, apply the same discipline as
editing: evidence before opinion, project convention before generic taste, and
concrete impact before broad advice.

## Workflow

1. Build a changed-file inventory before commenting. Cover behavior code, tests,
   public API/types, docs/release notes/config, and performance-sensitive paths
   when they appear in the diff.
2. Make a short internal issue ledger by category before choosing final comments:
   behavior/ownership, release-docs, tests, public API/types, performance, and
   project style. Prefer one strong comment per relevant category over several
   speculative comments on the same internal path.
3. Use the comment budget deliberately. In multi-file PRs, do not stop after the
   first one or two issues when the diff supports more; aim for 3-5 high-signal
   comments across distinct categories.
4. For each comment, name the exact file, cite diff evidence, state the user or
   maintainer impact, and mark uncertainty when project convention is not visible.
5. Do not stop at internally plausible bugs. Also inspect consumer paths,
   release/changelog wording, default/config/env semantics, stale cache paths,
   public type/docs coverage, and compatibility with existing behavior.
6. Prioritize comments that a maintainer can act on from the diff: user-visible
   contract or release-note mismatch, changed ownership between producer and
   consumer code, missing tests for the changed contract, then speculative
   internal risks. If a news/changelog/changeset file changed, reserve attention
   for whether its wording matches the exact behavior.
7. Apply code-gene style rules in review too: no trivial helper extraction,
   new private helpers belong at the end unless the file has strict ordering,
   comments must justify non-obvious behavior, and extra loops/walks/log fields
   must earn their cost.
8. For versioning or changesets, infer patch/minor/major only from the project's
   own convention or visible precedent; otherwise ask as a question instead of
   asserting a release level. For linters, type checkers, and static analysis
   tools, a false-negative fix can be minor because users may see new diagnostics;
   do not downgrade it to patch without project evidence.
9. For release/news wording, compare every noun to the behavior: command names,
   artifact types, full vs metadata-only download, wheels vs sdists, and whether
   the changed command paths are actually visible in the diff.

## Review Output

- Avoid generic praise or style-only comments unless the diff clearly supports
  them.
- Comment only on issues that still appear unresolved in the final diff; do not
  repeat suggestions that the visible diff has already implemented.
- Prefer concrete correctness, maintainability, API, typing, tests, release-note,
  compatibility, and performance issues.
- If the issue depends on project convention not visible in the diff, phrase it
  as a question or uncertainty instead of a hard assertion.
