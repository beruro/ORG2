# Performance Surface and Lifecycle

### 1. Establish the performance surface

Read the changed call chain from its production entry point. Inventory every resource the change can create or retain:

- `setInterval`, recursive `setTimeout`, `requestAnimationFrame`, debounce, retry, backoff
- DOM/Tauri/network listeners and Realtime channels
- workers, subprocesses, watchers, file scans, git operations, database reads
- module globals, atom maps, per-store maps, promises, abort controllers, buffers
- React subscriptions, selectors, derived arrays, render-time sorting/grouping
- eager list/history/diff/replay loading

Use targeted searches, adapting paths to the diff:

```powershell
rg -n "setInterval|setTimeout|requestAnimationFrame|addEventListener|listen\(|subscribe|channel\(" src src-tauri
rg -n "new Map|new Set|WeakMap|cache|inFlight|buffer|queue|history" src src-tauri
rg -n "poll|refresh|retry|scan|watch|stream|delta|dispose|cleanup|abort" src src-tauri
```

Do not treat grep hits as findings. Trace ownership, start conditions, steady-state behavior, and cleanup.

### 2. Build the lifecycle matrix

For each resource, record the required behavior in these states:

| Dimension | States to check |
| --- | --- |
| App | start, idle, active, shutdown |
| Document | visible, hidden, focus return |
| Network | online, offline, retry/backoff |
| Identity | signed out, signed in, refresh, account switch, endpoint switch |
| Scope | personal org, cloud org, removed org, revoked share |
| Session | unopened, active, inactive, deleted, forked |
| Instance | primary, direct-launched secondary, launcher-created secondary |

Flag any resource whose owner or terminal state is ambiguous.
