# Project Reference Selection

Use this reference for every code writing or code review task. Its job is to
force automatic project style/reference discovery before Codex edits or judges
code. User-mentioned references are priority signals, not the trigger.

## Default Behavior

When `code-gene` is active for implementation or review, do not wait for the
user to say "参考". After loading the core reference baseline
(`code-style.md`, `logging.md`, `java-backend.md`, and `delivery-tooling.md`),
discover the local project pattern:

- Implementation: target file to EOF, direct call sites/models, same package or
  layer, and 2-4 same-feature examples.
- Review: changed files, affected contracts/call sites, and same-layer examples
  needed to judge whether the diff matches project conventions.
- New feature family: find the nearest existing analogue before inventing a
  structure.

Only skip this when the task is purely informational and no code behavior,
review judgment, or implementation style is involved.

## Priority Signals

Treat these as highest-priority references on top of the default discovery:

- File paths, class names, method names, modules, features, screenshots, PRDs,
  spreadsheets, PDFs, prompt/skill keys, table names, or API names.
- Phrases such as "参考", "like", "类似", "照着", "按 X", "全量 reference",
  "看 X 的代码风格", "像 blueglass 那样", or "不要漏 reference".
- People or author/style names, for example `baolongjie` or `liujishuai`.

If a priority reference cannot be loaded, stop before editing and say exactly
which reference is missing and what was searched.

## Reference Coverage Note

Before substantial edits, report a compact coverage note:

- Requested references: what the user named or implied, if any.
- Loaded references: files, classes, docs, configs, or artifacts actually read.
- Project samples: same-module/same-layer/author-style examples read.
- Gaps: references not found, unreadable, or intentionally skipped.

Do not claim project consistency without this evidence.

## Local Project Sampling

For implementation work, sample in this order:

1. Target file, read to EOF.
2. Direct call sites and data models.
3. Same package or same layer examples.
4. Same feature-family examples, such as Blueglass, prompt config, skill config,
   chat handler, tool service, facade, repository, mapper, or DTO flow.
5. Author/style examples when the user names a person or when ownership is clear.

Prefer `rg`, `rg --files`, `git log --follow`, `git blame`, and `@author`
searches. Use samples to infer placement, naming, logging, null handling,
fallbacks, DTO conversion style, and whether new fields/methods belong at the
end.

## Author and Style Signals

In the 7Fresh MCP projects, use these as search hints, not absolute rules:

- `liujishuai`: conversation chain, handler orchestration, context flow,
  planning/tool execution, SSE/chat behavior, and user-facing dialogue flow.
- `baolongjie`: DUCC/config, DTO/Form/VO shape, facade/client wrappers,
  converter patterns, external service integration, and common utilities.

When matching author style, read 2-4 nearby examples owned by that author and
prefer the local layer's convention over a generic rule.

## "Like X" Requests

For "like Blueglass", "像配置平台那样", "类似商品推荐", or similar requests:

1. Search for `X` in both implementation and configuration.
2. Read the existing entry point, service/tool/facade path, DTOs, constants, and
   prompt/skill/config records involved.
3. Identify what must be copied as a pattern and what is feature-specific.
4. State the selected pattern before editing.

Do not implement from memory when an existing project analogue exists.

## "全量 Reference" Requests

When the user asks for "全量 reference", load all `code-gene` references needed
by the touched areas, then add project-local references from the sampling order
above. If context is too large, load the references most likely to affect the
edit first and state the deferred files explicitly.
