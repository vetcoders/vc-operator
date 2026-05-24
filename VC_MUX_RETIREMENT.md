# VC_MUX_RETIREMENT

## Current State

`vc-mux` in `/Users/polyversai/Libraxis/vc-runtime/vc_/src/mux/State.zig`
implements the same narrow mux state machine that `mux-agent` now owns in
Rust: client registration, pending request tracking, initialize replay/wait,
global request ID rewriting, and pending response lookup.

The cross-swarm synthesis flagged `vc-mux` as a strict subset of
`mux-agent` and recommended convergence rather than deletion inside an
implementation wave. The live checkout mostly supports that recommendation,
with one important correction: the request ID format is semantically the same
client/request tuple, but not literally byte-identical in this checkout.

## Evidence

- `vc_/src/mux/State.zig:94-103` registers and unregisters client channels.
- `vc_/src/mux/State.zig:120-142` handles initialize cache / waiter behavior.
- `vc_/src/mux/State.zig:158-180` rewrites local request IDs into a global
  pending-map key and stores the original client/local ID.
- `vc_/src/mux/State.zig:182-190` removes the pending request when the global
  response ID returns.
- `vc-console/mux-agent/src/runtime/client.rs:241-293` handles the same
  initialize cache / waiter path in Rust.
- `vc-console/mux-agent/src/runtime/client.rs:339-371` rewrites local request
  IDs into a global pending-map key, stores client/local ID, and forwards the
  request to the server.

Request ID format correction:

- Zig currently emits `c{d}:r{d}` at `vc_/src/mux/State.zig:160`.
- Rust currently emits `c{client_id}:{counter}` at
  `vc-console/mux-agent/src/runtime/client.rs:275` and
  `vc-console/mux-agent/src/runtime/client.rs:351`.

That means the retirement proof is a strict behavioral subset proof, not a
literal string-identity proof in the current checkout. The remaining
normalization is small and should be handled before deleting the Zig source.

## Proposal

Retire `vc-mux` as a runtime owner and converge all live mux behavior on
`mux-agent`.

Keep `vc-mux` source in place until the operator chooses deletion timing.
This note is not a delete authorization.

## Convergence Work

Estimated effort: 2-4 ED.

1. Normalize or explicitly document the request ID format boundary between
   `vc_` and `mux-agent`.
2. Replace any `vc_` caller that shells to or links `vc-mux` with the
   `rust-mux` IPC/proxy path.
3. Add a parity smoke that sends initialize plus one normal request through
   the replacement path and verifies response ID restoration.
4. After operator approval, delete the Zig mux source with `git rm` in the
   `vc_` repo and link this note from the deletion commit.

## Operator Rail

Operator decides delete timing.

The safe next move is convergence plus smoke parity, not source deletion in
this wave.
