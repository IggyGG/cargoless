# Held drain for verdict daemons

`POST /admin/quiesce-hold` uses the same bearer authorization as
`POST /admin/quiesce`. It refuses new pushed/batched verdict work, lets accepted
work finish and keeps the process alive after the activity counters reach zero.
It returns the same JSON activity shape as `GET /admin/active`. An adapter that
does not support held drain returns HTTP 501, rather than reporting a successful
no-op. Serve daemons with a separate build lane enabled also return 501: that
lane's queue is not covered by the verdict activity counters.

Ordinary `POST /admin/quiesce` keeps its existing drain-and-exit behavior. It can
also release an existing hold into a clean restart, after accepted work finishes.
Held drain does not change readiness or move routes; existing clients can keep
reading results through the same endpoint while accepted work finishes.

To remove a verdict daemon from a pool:

1. Establish an external admission/routing fence for replacement processes.
   Record the pod UID and container identity. A replacement must not rejoin the
   pool automatically during this operation.
2. Send authenticated `POST /admin/quiesce-hold` directly to that daemon. Require
   HTTP 200 and `quiescing: true`; any unsupported, unauthorized, malformed or
   unavailable response stops the operation.
3. Poll authenticated `/admin/active` on the same process. Require all seven
   counters to exist and equal zero: `active_worktrees`, `pending_pushes`,
   `pending_batch_waiters`, `pending_batch_members`, `inflight_batch_runs`,
   `waiting_witness_compiles`, and `inflight_witness_compiles`. Require
   `quiescing: true` throughout. Do not replace an unknown count with zero.
4. Remove discovery/routing membership, verify that removal, then recheck the
   same identities and zero activity before scaling. Keep the replacement fence
   until the owning workload's desired state excludes the retired ordinals.

The hold is in memory, not a durable lease: process/node restarts clear it. An
identity change invalidates the drain proof and must stop scale-down. A held
process does not exit just because it is idle, so an operator can inspect a
stable drain instead of racing the orchestrator's automatic restart. Signals
still follow the daemon's existing shutdown path; held drain is not protection
against forced termination. It is not an automatic scaler or an image-builder
lease protocol.
