# Log Continuity

Goal: when something breaks, anyone locates the failing method and step within 10 seconds, from logs alone. Logs across adjacent methods must connect; no gaps in one flow.

1. **One `traceId` per flow**, generated at the outermost entry (Controller/MQ/job), propagated via MDC. No new traceId mid-flow.
2. **Paired logs at each cross-method boundary**: caller logs before/after the call, callee logs entry/exit; four lines, same traceId. A missing line shows where it died.
3. **Key business IDs on every line** of the flow (orderId/userId/...), not just traceId.
4. **Same key, same value across layers**: `userId=123` stays `userId=123`, never `uid`/`user`.
5. **Exit log carries outcome + elapsed**: result (success/failure/fallback), costMs, and on failure the error type/reason.
6. **Business private methods need enough logs when they branch or call outward.** If a private method contains meaningful business logic, fallback, state mutation, cache/db/rpc access, or can explain a production symptom, add entry/step/exit or caller/callee logs proportional to the risk. Do not add noisy logs to simple builders/parsers/formatters with no business decision.
7. **Log fields must earn their place.** Do not add low-diagnostic fields just because they are easy to compute. Counts such as `detailCount`, list sizes, and map sizes are useful only when the count itself distinguishes a failure mode; otherwise prefer identifiers, routing keys, selected enum/code values, result status, and field-key sets that directly explain where the flow diverged.
8. **Do not manufacture log-only temporaries.** If a result object is non-null in the current branch, log `result.getCode()` / `result.getMsg()` / `result.getTaskId()` directly instead of predeclaring `success/code/msg/taskId` variables only for logging. Handle `null` result with an early branch, then keep the success/failure path short.

Minimum log set for a non-trivial business method:

```text
[entry]  method=createOrder, traceId=.., userId=123, itemCount=3
[step]   calling inventoryRpc.checkInventoryAvailable, orderId=.., skuIds=[..]
[step]   inventoryRpc.checkInventoryAvailable returned, available=true, costMs=42
[exit]   method=createOrder, traceId=.., userId=123, orderId=.., result=success, costMs=187
```

Log before and after external calls (RPC/HTTP/MQ/DB): target service+method, key IDs (not full bodies), result, costMs, traceId, failure reason. `warn` for recoverable/slow (>threshold), `error` with stack trace for failures. Never log tokens/credentials/PII or full large bodies at `info` (use size/count). No anonymous logs like `log.info("success")`.
