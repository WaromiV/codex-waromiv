# Codex-rs Project Specification

## Overview

`codex-rs` is a Rust workspace that implements the Codex agent platform end-to-end. It houses protocol definitions, core runtime helpers, sandboxed execution policies, client/server components, CLI/TUI frontends, and integration tooling that let GPT-powered agents run, respond to signals, and surface user interactions.

## Architecture

- **Workspace organization** – A Cargo workspace where each crate is prefixed with `codex-`. Shared utilities live under `common`, `protocol`, and `core`, while application layers (services, clients, UIs) build on those foundations.
- **Sandbox/runtime orchestration** – Execution flows are governed by `core` helpers and platform-specific sandbox providers (`exec`, `linux-sandbox`, `windows-sandbox-rs`, `execpolicy*`). Environment flags like `CODEX_SANDBOX*` gate integrations when running in restricted contexts.
- **Services & clients** – Backend services (`app-server`, `exec-server`, `responses-api-proxy`) speak the `protocol` schema, client adapters (`backend-client`, `rmcp-client`) handle HTTP/gRPC, and tooling (`mcp-server`, `cloud-tasks`, `feedback`) add operational features.
- **Frontends** – `cli` exposes a scriptable command-line interface; `tui`/`tui2` provide terminal dashboards with stylized views and snapshot tests; `lmstudio`, `ollama`, `chatgpt`, and `keyring-store` integrate with external tooling/auth flows.

## Key Interfaces

- **Protocol layer** – Defined in `protocol/` and consumed by every service/client. It centralizes request/response types, streaming contracts, SSE helpers, and shared status enums.
- **Core runtime** – `core/` adds request orchestration, logging, telemetry instrumentation, response mocks (for tests), and sandbox-aware utilities that coordinate agents across threads/processes.
- **UI/CLI** – `tui` uses `ratatui::Stylize`; style conventions in `tui/styles.md`. CLI/TUI share helpers in `common` to keep telemetry and formatting consistent.

## Development Workflow

- **Clippy/formatting** – `rustfmt.toml` and `clippy.toml` enforce consistent layout. Rules emphasize inline `format!` args, collapsible `if`, and method references instead of redundant closures.
- **Formatter & fixers** – After Rust changes run `just fmt`, then `just fix -p <project>` to resolve lints (ask before workspace-wide fix). Tests run per crate: `cargo test -p codex-<crate>`, and if `common`, `core`, or `protocol` changed, `cargo test --all-features`.
- **Snapshot discipline** – `codex-tui` relies on `insta`. Run `cargo test -p codex-tui`, inspect `.snap.new` files, and accept via `cargo insta accept -p codex-tui` when updates are intentional.

## Style & Documentation Expectations

- **Crate naming** – Prefix crates with `codex-`; workspace members listed in `Cargo.toml`.
- **Rust code style** – Inline `format!` arguments, collapse nested `if`s, prefer method references, avoid touching sandbox-related env vars (`CODEX_SANDBOX*`). Tests should compare entire objects with `pretty_assertions::assert_eq!`.
- **TUI conventions** – Favor `ratatui::Stylize` helpers (e.g., `"status".dim()`). Use `Line::from(vec![…])` or `.into()` depending on clarity. Leverage wrapping helpers (`tui/src/wrapping.rs`) for text layout.
- **Documentation** – Update `docs/` whenever APIs change. Primary docs include `README.md`, `config.md`, `docs/protocol_v1.md`, `docs/codex_mcp_interface.md`, and TM-specific guides.

## Security & Sandbox Considerations

- Sandbox environment flags protect integration tests (`CODEX_SANDBOX_NETWORK_DISABLED`, `CODEX_SANDBOX=seatbelt`). Do not modify logic related to those env vars; tests may early-exit when they detect the flags.
- Execution policy crates ensure safe command runs; avoid bypassing restrictions and honor `execpolicy-legacy` behavior when necessary.

## Testing & QA

- Prefer tooling from `core_test_support::responses` for integration tests: SSE mocks, response request inspection, helper constructors like `sse`, `ev_response_created`, `ev_function_call`, and `wait_for_event`.
- When validating TUI output, use `cargo insta pending-snapshots -p codex-tui` and review diffs before accepting.
- Tests should lean on pretty assertions and compare complete structs. For multi-request flows use `ResponseMock::requests()` to assert each behavior.

## Operational Notes

- Use `just` helpers (`just fmt`, `just fix`, `just test`, etc.) defined per crate.
- Keep an eye on `rust-toolchain.toml` for toolchain pins, and `deny.toml` for banned code patterns.
- README and docs outline onboarding; refer to them when preparing new features or onboarding new contributors.
