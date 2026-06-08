---
name: code-gene
description: >
  Code generation discipline guided by the Eight Honors and Eight Shames and other principles (code guide).
  Forces verification over guessing, confirmation over assumption, and
  convention over invention before any code is written or modified.
  Enforces anti-over-engineering rules and cross-method log continuity so the
  main flow stays clean and failures can be localized from logs alone.
  Use when user invokes /code-gene, asks to generate or modify code under
  strict discipline, says "use code-gene", or wants the Eight Honors checklist
  applied to a coding task.
---

# Code Gene Skill

When active, every code change passes the Eight Honors and the checklist before output.
Always read the relevant current code before advising or editing.

## Activation

Active when the user says `/code-gene`, `use code-gene`, or asks to apply it.
Stays active until `stop code-gene` / `normal mode`. Skipping the pre-flight checklist is itself a violation.

## Eight Honors and Eight Shames

1. **Research, don't guess interfaces.** Read the real class/method/signature before calling; never infer an API from naming.
2. **Confirm, don't execute vaguely.** Ambiguous request → ask before coding, not a plausible guess.
3. **Validate, don't imagine business logic.** Rules come from the user or existing code; if absent, stop and ask.
4. **Reuse, don't invent.** Search the repo for an existing util/service/DTO/abstraction before writing a new one.
5. **Verify, don't skip.** After a change, run build/test/typecheck/lint (or the UI); no "done" without evidence.
6. **Follow conventions, don't break architecture.** Match existing layering, naming, packages, error handling, logging, style.
7. **Admit ignorance, don't pretend.** Unclear domain term / framework behavior / legacy module → say so and ask.
8. **Refactor cautiously, don't modify blindly.** Touch only what the task needs; flag unrelated dead code but don't delete; confirm before DB schema / public API changes.

## Anti-Over-Engineering

Identify the **main path** and **necessary branches**; write only those.
Prefer the shortest clear implementation that satisfies the current requirement; do not add layers, branches, helpers, or configuration unless they reduce real current complexity.
Forbidden:

1. **No defensive `if`.** Contract validation happens once at the public entry (Controller/RPC/API), not in every downstream method.
2. **No speculative abstraction.** Extract an interface/factory/strategy only when two real implementations exist now. Single-impl interfaces are an anti-pattern unless the project mandates them.
3. **No exception-swallowing try-catch.** Either propagate, or make it an explicit business fallback and log the reason. Never `catch(Exception e){ log; return null; }`.
4. **No `if`/`else` nesting > 2 levels.** Use guard clauses (early return) or split the method. The main path stays flat at the outermost level.
5. **No "just in case" fields/params/config.** Not used by the current requirement → don't add it.
6. **No trivial extraction.** Do not create a private method for a one-line predicate, one-field null/blank check, or single put/set side effect. Inline it in the caller unless the helper is reused, names a non-obvious business rule, or materially flattens a complex branch.

**Main-path-first:** write the success path as one straight line; push boundaries to guard clauses or thrown exceptions.

```java
// Bad: main logic buried 4 levels deep
public Order createOrder(CreateOrderCmd cmd) {
    if (cmd != null) {
        if (cmd.getUserId() != null) {
            if (cmd.getItems() != null && !cmd.getItems().isEmpty()) {
                User user = userService.getUser(cmd.getUserId());
                if (user != null) { /* main logic */ }
            }
        }
    }
    return null;
}

// Good: validation up front, main path flat
public Order createOrder(CreateOrderCmd cmd) {
    validateCreateOrderCmd(cmd);                       // entry validation, throws on failure
    User user = userService.getUser(cmd.getUserId());  // throws if not found, not checked here
    Order order = buildOrder(cmd, user);
    orderRepository.save(order);
    return order;
}
```

When any line fails, the log and stack trace point straight to it.

Also avoid dense null-check ternaries; prefer a guard `if`, `Optional`, or a named local/helper when it reads clearer.

## Edit Placement

New methods and fields go at the **end** of the class/file — do not insert them into the middle of existing code. This keeps the diff small and leaves the existing layout undisturbed (Honor 8). Only deviate when the file enforces a strict, obvious member ordering; then follow that ordering.

## Comment Style

Rigorous and informative: the reader understands each class/method/field from Javadoc, and follows the body from step comments — without reverse-engineering.

- **Strictly forbidden: HTML tags.** Never `<p>`, `<li>`, `<ul>`, `<ol>`, `<br>`, `<pre>`, `<code>` in `/** */` or `//`. Plain text and natural line breaks only. Absolute, no "Javadoc convention" exception.
- **Javadoc states responsibility, not the obvious.** Class/interface: its role. Method: what it does and (if non-obvious) why/when. Use `@param`/`@return`/`@throws` — each tag carries real info, never echoes the name (`@param config the config` is noise).
- **Step comments in the body** mark each phase of the flow; for a non-obvious decision, state the reason in parentheses.
- **Scope:** public types/methods/fields and core business methods require full Javadoc. Private methods with clear business decisions, state transitions, fallback paths, cross-system side effects, or non-obvious domain rules also require Javadoc or a concise responsibility comment. Trivial private helpers such as simple parameter construction, parsing, formatting, null/blank normalization, one-line predicates, or obvious getters/setters do not need Javadoc. Rigor = responsibility + rationale, never boilerplate that restates the signature.

Canonical example (clear `@param`/`@return`, per-step comments, rationale on the ordering, zero HTML):

```java
/**
 * 动态组装 Prompt
 * @param config 包含所有模板组件和变量的配置对象
 * @return 组装后的完整 Prompt
 */
public static String buildPrompt(QueryPromptConfigModel config) {
    // 处理系统消息
    String systemMsg = new StringSubstitutor(config.getVariables(), "{{", "}}")
            .replace(config.getSystemMessageTemplate());
    // 处理用户消息
    String userMsg = new StringSubstitutor(config.getVariables(), "{{", "}}")
            .replace(config.getUserMessageTemplate());
    // 处理格式要求
    String formatMsg = new StringSubstitutor(config.getVariables(), "{{", "}}")
            .replace(config.getFormatTemplate());
    // 组装最终 Prompt（指令前置，数据后置，利用 primacy effect 提升指令遵循率）
    return String.join("\n\n", systemMsg, userMsg, formatMsg);
}
```

## Log Continuity

Goal: when something breaks, anyone locates the failing method and step within 10 seconds, from logs alone. Logs across adjacent methods must connect — no gaps in one flow.

1. **One `traceId` per flow**, generated at the outermost entry (Controller/MQ/job), propagated via MDC. No new traceId mid-flow.
2. **Paired logs at each cross-method boundary**: caller logs before/after the call, callee logs entry/exit — four lines, same traceId. A missing line shows where it died.
3. **Key business IDs on every line** of the flow (orderId/userId/...), not just traceId.
4. **Same key, same value across layers** — `userId=123` stays `userId=123`, never `uid`/`user`.
5. **Exit log carries outcome + elapsed**: result (success/failure/fallback), costMs, and on failure the error type/reason.
6. **Business private methods need enough logs when they branch or call outward.** If a private method contains meaningful business logic, fallback, state mutation, cache/db/rpc access, or can explain a production symptom, add entry/step/exit or caller/callee logs proportional to the risk. Do not add noisy logs to simple builders/parsers/formatters with no business decision.

Minimum log set for a non-trivial business method (read top-to-bottom = which flow, which user, which step, which call, success/fail, how long, where it broke):

```
[entry]  method=createOrder, traceId=.., userId=123, itemCount=3
[step]   calling inventoryRpc.checkInventoryAvailable, orderId=.., skuIds=[..]
[step]   inventoryRpc.checkInventoryAvailable returned, available=true, costMs=42
[exit]   method=createOrder, traceId=.., userId=123, orderId=.., result=success, costMs=187
```

Log before and after external calls (RPC/HTTP/MQ/DB): target service+method, key IDs (not full bodies), result, costMs, traceId, failure reason. `warn` for recoverable/slow (>threshold), `error` with stack trace for failures. Never log tokens/credentials/PII or full large bodies at `info` (use size/count). No anonymous logs like `log.info("success")`.

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

## Naming

Use clear, consistent, business-specific names. Avoid vague names (`getData`, `handle`, `process`, `doQuery`). Keep names within ~5 words; longer usually means mixed responsibilities. Don't stuff params into the name — prefer a query object:

```java
getUserByIdAndStatusAndType(...)   // Avoid
getUser(UserQuery query)           // Prefer
```

### Business method prefixes

| Prefix | Meaning |
| --- | --- |
| `getXxx` | Query a single object by unique ID |
| `queryXxx` | Query by conditions, one or many |
| `listXxx` | Query a list |
| `countXxx` / `existsXxx` | Count / check existence |
| `saveXxx` / `updateXxx` / `deleteXxx`·`removeXxx` | Persist / update / delete |
| `buildXxx` / `convertXxx`·`toXxx` | Build object / convert types |
| `checkXxx`·`validateXxx` / `parseXxx` | Check·validate / parse input |
| `loadXxx` / `refreshXxx` / `syncXxx` | Load heavy resource / refresh / synchronize |
| `handleXxx` / `executeXxx` | Handle a business event / execute a task |

### Cache-aware naming

For methods that read through cache, use the `WithCache` suffix; keep TTL, null-caching, and penetration protection in a shared cache template, not per call site.

| Name | Meaning |
| --- | --- |
| `getXxxWithCache` | Cache first, fall back to RPC/DB, backfill on success |
| `getXxxFromCache` / `getXxxFromDb` / `getXxxFromRpc` | Read from one source only |
| `refreshXxxCache` / `evictXxxCache` / `warmUpXxxCache` | Force refresh / invalidate / preload |

```java
getUserByIdWithCache(...)   queryOrderDetailWithCache(...)   evictUserCache(...)
```

### RPC client, DTO, and method naming

- **Remote-call client:** the wrapper that actually calls a remote/external RPC is named `XxxClient` or `XxxServiceClient` (e.g. `OrderServiceClient`). The standard layering is **Service wraps Client**: a business `XxxService` wraps and calls the `XxxClient`, and business code depends on the `Service`, not the `Client` directly. Avoid vague names like `CommonService` / `RemoteService` / `DataService`; don't encode transport in the name (the impl may: `OrderServiceClientDubboImpl`).
- **Request/Response:** input `XxxRequest` (extends `BaseRequest`), output `XxxResponse`. All transport objects implement `Serializable`. New fields are appended at the **end** of the class (serialization / forward compatibility).

```java
OrderService -> OrderServiceClient (or OrderClient)   // business Service wraps the remote-call Client
CreateOrderRequest / CreateOrderResponse              // input / output DTOs
OrderServiceClientDubboImpl                           // impl may name transport
```

- **RPC method prefixes:** `getXxx` (by id), `queryXxx` (by condition), `listXxx`, `batchGetXxx` (`Collection<ID>` → `Map<ID,T>`), `createXxx` / `updateXxx` / `cancelXxx`, `submitXxx` (trigger action/workflow), `notifyXxx` (fire-and-forget), `syncXxxFromYyy` / `syncXxxToYyy`, `checkXxx` (remote lookup), `validateXxx` (local rule check).

```java
batchGetSkuPrice(...)   submitRefund(...)   notifyOrderPaid(...)   syncUserFromCrm(...)
```

### Constants

No interfaces as constant holders — use `final class XxxConstants` for technical/shared constants, `enum` for business states/types. Keep interfaces focused on method contracts.

For 7Fresh / DongBoot Java projects, `~/.claude/rules/` also enforces layering, JSF provider/consumer rules, and detailed logging (`architecture-and-api.mdc`, `logging-exception.mdc`, ...); defer to those for the full project rules.

## Pre-flight Checklist

Before any edit, output a brief `<thinking>` covering the applicable items:

- [ ] Interfaces I will call: read/grepped.
- [ ] Ambiguity resolved or asked; business rules sourced from user/code, not invented.
- [ ] Existing util/service/DTO searched before creating new.
- [ ] Verification plan; layering/naming/style match the project.
- [ ] Every changed line traces back to the request.
- [ ] Each `if`/`try-catch`/abstraction maps to a concrete current requirement — else remove.
- [ ] Comments: Javadoc responsibility + `@param`/`@return`, step comments, no HTML tags.
- [ ] New methods/fields appended at the end, not inserted mid-file.
- [ ] From logs alone (no source), a reader can tell: which flow, which method, which entity, success/fail, costMs, where it failed.

Write code only after the checklist is clean.

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
