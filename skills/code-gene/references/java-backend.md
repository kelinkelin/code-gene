# Java Backend Rules

## Naming

Use clear, consistent, business-specific names. Avoid vague names (`getData`, `handle`, `process`, `doQuery`). Keep names within about five words; longer usually means mixed responsibilities. Do not stuff params into the name; prefer a query object.

```java
getUserByIdAndStatusAndType(...)   // Avoid
getUser(UserQuery query)           // Prefer
```

## Business Method Prefixes

| Prefix | Meaning |
| --- | --- |
| `getXxx` | Query a single object by unique ID |
| `queryXxx` | Query by conditions, one or many |
| `listXxx` | Query a list |
| `countXxx` / `existsXxx` | Count / check existence |
| `saveXxx` / `updateXxx` / `deleteXxx` / `removeXxx` | Persist / update / delete |
| `buildXxx` / `convertXxx` / `toXxx` | Build object / convert types |
| `checkXxx` / `validateXxx` / `parseXxx` | Check / validate / parse input |
| `loadXxx` / `refreshXxx` / `syncXxx` | Load heavy resource / refresh / synchronize |
| `handleXxx` / `executeXxx` | Handle a business event / execute a task |

## Cache-Aware Naming

For methods that read through cache, use the `WithCache` suffix. Keep TTL, null-caching, and penetration protection in a shared cache template, not per call site.

| Name | Meaning |
| --- | --- |
| `getXxxWithCache` | Cache first, fall back to RPC/DB, backfill on success |
| `getXxxFromCache` / `getXxxFromDb` / `getXxxFromRpc` | Read from one source only |
| `refreshXxxCache` / `evictXxxCache` / `warmUpXxxCache` | Force refresh / invalidate / preload |

```java
getUserByIdWithCache(...)   queryOrderDetailWithCache(...)   evictUserCache(...)
```

## RPC Client, DTO, and Method Naming

- **Remote-call client:** the wrapper that actually calls a remote/external RPC is named `XxxClient` or `XxxServiceClient` (for example, `OrderServiceClient`). The standard layering is **Service wraps Client**: a business `XxxService` wraps and calls the `XxxClient`, and business code depends on the `Service`, not the `Client` directly. Avoid vague names like `CommonService`, `RemoteService`, or `DataService`; do not encode transport in the name. The impl may name transport, for example `OrderServiceClientDubboImpl`.
- **Request/Response:** input `XxxRequest` (extends `BaseRequest`), output `XxxResponse`. All transport objects implement `Serializable`. New fields are appended at the **end** of the class for serialization / forward compatibility.

```java
OrderService -> OrderServiceClient (or OrderClient)   // business Service wraps the remote-call Client
CreateOrderRequest / CreateOrderResponse              // input / output DTOs
OrderServiceClientDubboImpl                           // impl may name transport
```

- **RPC method prefixes:** `getXxx` (by id), `queryXxx` (by condition), `listXxx`, `batchGetXxx` (`Collection<ID>` -> `Map<ID,T>`), `createXxx` / `updateXxx` / `cancelXxx`, `submitXxx` (trigger action/workflow), `notifyXxx` (fire-and-forget), `syncXxxFromYyy` / `syncXxxToYyy`, `checkXxx` (remote lookup), `validateXxx` (local rule check).

```java
batchGetSkuPrice(...)   submitRefund(...)   notifyOrderPaid(...)   syncUserFromCrm(...)
```

## Constants

No interfaces as constant holders; use `final class XxxConstants` for technical/shared constants, `enum` for business states/types. Keep interfaces focused on method contracts.

For 7Fresh / DongBoot Java projects, `~/.claude/rules/` also enforces layering, JSF provider/consumer rules, and detailed logging (`architecture-and-api.mdc`, `logging-exception.mdc`, ...); defer to those for the full project rules.
