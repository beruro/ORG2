# Performance Verification and Delivery

### 6. Verify proportionally

Always run:

- targeted unit tests for cache bounds, coalescing, invalidation, visibility, and stale-result rejection
- TypeScript typecheck and lint for changed frontend files
- Rust unit tests/checks for changed backend modules; if the shared Cargo cache is corrupt or policy-blocked, report it and use the narrowest valid independent compilation without deleting broad caches
- `git diff --check`

For rendered/background changes, also run the real Tauri surface when available:

1. Observe primary and secondary instances separately.
2. Measure visible idle, hidden idle, active streaming, and post-close/post-delete behavior.
3. Exercise account switch, endpoint switch, and direct secondary launch when relevant.
4. Confirm request/subscription/timer counts stabilize rather than grow after repeated open/close cycles.
5. Confirm strict rendered E2E uses user-visible actions for the behavior under assertion.

Do not claim a performance improvement from code shape alone. State the evidence actually collected and any environment blocker.

## Review rejection rules

Reject or revise a change when any applicable answer is unknown or false:

- Who owns this background resource, and exactly when is it stopped?
- Can this timer overlap itself or continue while hidden?
- Why is polling necessary instead of invalidation?
- Can two mounted consumers issue the same request?
- Does the cache have a maximum size, freshness rule, identity key, and eviction event?
- Can an old async completion write after a newer request or identity switch?
- Does one session's update wake unrelated session views?
- Does a growing transcript/history/diff require full eager materialization?
- Does a direct secondary launch inherit primary external history or auth state?
- Can a missing rendered element be skipped while the E2E still passes?

## Required delivery output

Report findings and evidence in this compact form:

| Area | Verdict | Evidence | Change or reason kept | Verification |
| --- | --- | --- | --- | --- |
| Background work | fix / keep | timer/subscription owner and cadence | exact lifecycle decision | test or measurement |
| Memory | fix / keep | retained structure and growth bound | cap/TTL/eviction | bound/eviction test |
| Scope/isolation | fix / keep | cache/request key | identity/generation guard | switch/revocation test |
| Rendering/hot path | fix / keep | subscription/allocation trace | narrowing/coalescing | render or unit evidence |

End with:

- `Performance verdict: pass` only when every applicable invariant is evidenced.
- `Performance verdict: blocked` when required real measurement or compilation cannot run; name the blocker.
- `Performance verdict: fail` when an unbounded, duplicate, hidden-active, stale-write, or cross-identity path remains.

Never promise that a skill can make regressions impossible. Enforce the gates, expose unknowns, and refuse an unsupported green verdict.
