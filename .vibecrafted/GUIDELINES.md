# Vibecrafted Operator Workspace — Vetcoders GUIDELINES

> Per-workspace, agent-agnostic instructions for `operator/`. Same rules for
> Claude, Codex, Gemini, Junie, and Qwen. Global doctrine still applies; this
> file only extends it for the consolidated operator workspace.

## Identity

- **Workspace:** standalone `vetcoders/vc-operator` checkout.
- **Role:** consolidated operator platform workspace for `mux-agent`,
  `tui-agent`, `tray-agent`, and `shell-agent`.
- **Crate names:** keep existing distribution names stable. `mux-agent/`
  publishes as `rust-mux`; `tui-agent/` publishes as
  `vibecrafted-operator`.
- **Current split:** `mux-agent` owns lifecycle and MCP process supervision;
  `tui-agent` owns the terminal cockpit; `tray-agent` owns the menu bar
  control surface; `shell-agent` owns the macOS `.app` wrapper and UniFFI
  bridge.

## Quality Gates

Use the top-level `Makefile` from this directory:

```bash
make gates
cargo check --workspace --all-features
cargo check --workspace --no-default-features
```

`make gates` means `fmt-check + clippy -D warnings + test --workspace`.
Do not add `#[allow(...)]`, `nosemgrep`, `// noqa`, `--no-verify`, or other
silencers to get through a gate. Fix the cause or report the blocker.

## Living Tree Convention

This workspace is a shared live tree. Concurrent edits are expected.

- Re-read files before editing if time has passed.
- Run Loctree mapping before changing hub files.
- Do not revert another agent's work unless the operator explicitly asks.
- If a concurrent edit conflicts with the T0 contract, preserve evidence,
  reconcile the file, and report exactly what happened.
- `.vibecrafted/{plans,reports}` are daily symlinks into
  `$VIBECRAFTED_HOME/artifacts/vetcoders/vibecrafted-operator/<YYYY_MMDD>/`.
  Date-rotation drift is not product code.

## Wizard / Config Doctrine

The wizard/config truth lives in `mux-agent`, inherited from `rust-mux`.
Client config files remain the source of truth; running processes can enrich
status but must not drive discovery by themselves.

Keep the strategy split intact:

- **Unified:** generate mux outputs without rewriting host configs.
- **Per-client:** generate client-shaped mux configs while preserving the
  merged daemon config.
- **Auto-rewire:** backup-first, preview-first, explicit-confirm rewrite path.

Never silently rewrite host AI-client configs from a non-danger strategy.
Never collapse `mux_gen.rs` and `danger.rs` into one writer; that split is part
of the security model.

## Shell-Agent Build Shape

`shell-agent/ffi` is the Rust/UniFFI bridge.
`shell-agent/uniffi-bindgen` is the binding generator wrapper.
`shell-agent/app/Vibecrafted` is the macOS app target. Build it from the root
with `make app`; create local or signed DMGs with `make dmg` and
`make dmg-signed`.

## Commit Convention

- Title prefix: `[<agent>/<track>] <description>`.
- For workspace extraction/stabilization: `[codex/vc-operator] <description>`.
- Multi-file commits need a body with bullet points.
- Trailer:

```text
Authored-By: codex <agents@vetcoders.io>
```

Forbidden: vendor footers, personal signatures, and
`Co-Authored-By: Claude ...`.

Use the canonical brand line only when a sigblock is needed:

```text
𝚅𝚒𝚋𝚎𝚌𝚛𝚊𝚏𝚝𝚎𝚍. with AI Agents by Vetcoders (c)2024-2026
```

## Anti-Patterns Repo-Specific

- Renaming `rust-mux` or `vibecrafted-operator` just because their paths moved.
- Reintroducing a root-level TUI crate after the extraction; `tui-agent/` is the
  single source of truth.
- Reintroducing deleted rust-mux monoliths such as `src/runtime.rs`.
- Treating green `cargo check` as shipping readiness without install,
  discoverability, and first-user proof.
- Deleting historical audit Markdown instead of preserving it under
  `tui-agent/audits/historical/`.

---

_𝚅𝚒𝚋𝚎𝚌𝚛𝚊𝚏𝚝𝚎𝚍. with AI Agents by Vetcoders (c)2024-2026_
