# Code Style and Implementation Discipline

## Eight Honors and Eight Shames

1. **Research, don't guess interfaces.** Read the real class/method/signature before calling; never infer an API from naming.
2. **Confirm, don't execute vaguely.** Ambiguous request -> ask before coding, not a plausible guess.
3. **Validate, don't imagine business logic.** Rules come from the user or existing code; if absent, stop and ask.
4. **Reuse, don't invent.** Search the repo for an existing util/service/DTO/abstraction before writing a new one.
5. **Verify, don't skip.** After a change, run build/test/typecheck/lint (or the UI); no "done" without evidence.
6. **Follow conventions, don't break architecture.** Match existing layering, naming, packages, error handling, logging, style.
7. **Admit ignorance, don't pretend.** Unclear domain term / framework behavior / legacy module -> say so and ask.
8. **Refactor cautiously, don't modify blindly.** Touch only what the task needs; flag unrelated dead code but don't delete; confirm before DB schema / public API changes.

## Anti-Over-Engineering

Identify the **main path** and **necessary branches**; write only those.
Prefer the shortest clear implementation that satisfies the current requirement; do not add layers, branches, helpers, or configuration unless they reduce real current complexity.

Forbidden:

1. **No defensive `if`.** Contract validation happens once at the public entry (Controller/RPC/API), not in every downstream method.
2. **No speculative abstraction.** Extract an interface/factory/strategy only when two real implementations exist now. Single-impl interfaces are an anti-pattern unless the project mandates them.
3. **No exception-swallowing try-catch.** Either propagate, or make it an explicit business fallback and log the reason. Never `catch(Exception e){ log; return null; }`.
4. **No `if`/`else` nesting > 2 levels.** Use guard clauses (early return) or split the method. The main path stays flat at the outermost level.
5. **No "just in case" fields/params/config.** Not used by the current requirement -> don't add it.
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

New methods and fields go at the **end** of the class/file; do not insert them into the middle of existing code. This keeps the diff small and leaves the existing layout undisturbed. Only deviate when the file enforces a strict, obvious member ordering; then follow that ordering.

## Comment Style

Rigorous and informative: the reader understands each class/method/field from Javadoc, and follows the body from step comments without reverse-engineering.

- **Strictly forbidden: HTML tags.** Never `<p>`, `<li>`, `<ul>`, `<ol>`, `<br>`, `<pre>`, `<code>` in `/** */` or `//`. Plain text and natural line breaks only. Absolute, no "Javadoc convention" exception.
- **Javadoc states responsibility, not the obvious.** Class/interface: its role. Method: what it does and (if non-obvious) why/when. Use `@param`/`@return`/`@throws`; each tag carries real info, never echoes the name (`@param config the config` is noise).
- **Step comments in the body** mark each phase of the flow; for a non-obvious decision, state the reason in parentheses.
- **Scope:** public types/methods/fields and core business methods require full Javadoc. Private methods with clear business decisions, state transitions, fallback paths, cross-system side effects, or non-obvious domain rules also require Javadoc or a concise responsibility comment. Trivial private helpers such as simple parameter construction, parsing, formatting, null/blank normalization, one-line predicates, or obvious getters/setters do not need Javadoc. Rigor = responsibility + rationale, never boilerplate that restates the signature.

Canonical example:

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
