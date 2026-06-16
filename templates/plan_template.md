# Plan: <feature / paper method name>

> Reference paper: <Title> (<authors>, <venue year>)
> Target framework: <your codebase / module>

## 1. Mechanism (one sentence — confirm before designing)

This method changes **<what entity/operation>** at **<which call site or data structure>**,
in order to **<what effect>**.

Disambiguation:

- Entity it operates on: <observation / landmark / residual / frame / …>
- Existing call site it intercepts / replaces / wraps: <function name>
- Same result faster, or a different result?: <acceleration | behavior change>
- Bounded quantity / property it enforces: <e.g. ≤ N per cell per frame>

## 2. Core invariant(s)

- INV: **<property that must always hold>**
  - Maintained by: <atomic binding on add> + <unified hook on remove>

## 3. Key design decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| <e.g. sort primary key> | <choice> | <why> |
| <sort tie-breaker> | <choice> | <why> |
| <eviction / dedup mechanism> | <choice> | <why> |

## 4. Known trade-offs (record up front)

| Problem | Impact | Mitigation |
|---------|--------|------------|
| <e.g. bucket full → new entry dropped> | <effect> | <param / partial fix> |

## 5. Implementation checklist (= definition of done)

| Item | File(s) |
|------|---------|
| Data structures (<types>) | <header> |
| Manager full impl (add/remove/update/query) | <source> |
| Hook into main call site | <file> |
| Lifecycle: register on add | <file> |
| Lifecycle: update on optimize (cross-boundary migrate) | <file> |
| Lifecycle: remove on normal deletion | <file> |
| Lifecycle: remove on outlier rejection | <file> |
| Lifecycle: remove on marginalization / eviction | <file> |
| Visualization overlay (current frame only) + stats | <file> |
| Config params wired into options struct | <file> |
| Config params written to config file | <config> |
| Build system updated (new source files) | <build file> |

## 6. Validation plan

- What to visualize: <structure overlay + live stats: counts / rejections / dups>
- Scope: CURRENT frame/state only (not all-history)
- Success looks like: <observable behavior on real input>
