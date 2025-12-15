# Codex-rs Wiki

## Purpose

`codex-rs` is the Rust backbone of the Codex agent platform. The workspace couples protocol models, runtime helpers, sandboxed execution layers, client/server adapters, and both CLI and TUI frontends so agents can execute work, handle streaming responses, and present rich terminal views.

## Workspace Layout

- `protocol/` (`codex-protocol`) defines the shared schema for requests, responses, SSE events, and telemetry data. Almost every crate depends on these types to stay interoperable.
- `core/` (`codex-core`) orchestrates deliveries, logging, telemetry, and process isolation. Sandbox-aware utilities guard against running unsafe integrations while the runtime coordinates user events.
- `common/` (`codex-common`) houses utilities (logging, errors, config helpers) reused across CLI, TUI, services, and tests.
- Services like `app-server`, `exec-server`, `responses-api-proxy`, and `cloud-tasks` expose HTTP/gRPC endpoints and integrate with workflow engines in production.
- Clients (`backend-client`, `rmcp-client`, `mcp-server`, `chatgpt`, `ollama`, `lmstudio`) adapt the protocol layer to external APIs, telepresence layers, or sandboxed runtime environments.
- CLI/TUI frontends (`cli`, `tui`, `tui2`) provide terminal experiences. `tui` leverages `ratatui` stylized spans and snapshot testing for deterministic rendering.
- Execution layers (`exec`, `execpolicy`, `linux-sandbox`, `windows-sandbox-rs`) govern how agents run shell commands across platforms.

## Development & Style Guidelines

- Follow the crate naming convention (`codex-` prefix) and ensure documentation under `docs/` reflects API changes.
- Use inline `format!` arguments, collapse conditionals when possible, and prefer method references over redundant closures.
- Keep sandbox-related logic untouched; environment flags (`CODEX_SANDBOX*`) gate important guardrails. Tests already account for restricted networking via `CODEX_SANDBOX_NETWORK_DISABLED`.
- TUI styling should use `ratatui::Stylize` helpers, avoid explicit `.white()`, and use `.into()`/`Line::from` forms that keep layout concise.
- Text wrapping should rely on `textwrap::wrap` or the helpers in `tui/src/wrapping.rs` when dealing with Lines.
- Tests should assert on entire objects using `pretty_assertions::assert_eq!` for clearer diffs.

## Testing Workflow

1. Run `just fmt` after Rust edits (always).
2. Run `just fix -p <project>` for linters/clippy fixes before finalizing. Ask before running workspace-wide `just fix`.
3. Run crate-specific tests via `cargo test -p codex-<project>`.
4. If `common`, `core`, or `protocol` change, follow up with `cargo test --all-features`.
5. `codex-tui` relies on snapshot testing; use `cargo test -p codex-tui`, `cargo insta pending-snapshots -p codex-tui`, review `.snap.new` files, and accept updates with `cargo insta accept -p codex-tui`.

## Integration Testing Notes

- Use `core_test_support::responses` helpers for SSE-backed tests. Capture `ResponseMock`, inspect requests with helpers (e.g., `function_call_output`, `call_output`), and prefer `wait_for_event` over timeouts.
- Build SSE sequences using constructors like `ev_response_created`, `ev_function_call`, and `ev_completed`, wrapped by `sse(...)`.
- When verifying requests, call `mock.single_request()` for single-POST flows or `mock.requests()` to inspect every captured submission.

## Suggested Next Topics for Wiki Expansion

1. **Protocol Deep Dive** – Document each message type, necessary headers, and extension points.
2. **Sandbox Flow** – Outline how commands flow through `exec`, platform sandboxes, and `execpolicy`.
3. **CLI/TUI Walkthroughs** – Provide usage examples, describe the stylization conventions, and explain how snapshots are structured.
4. **Service Deployments** – Map how `app-server`, `exec-server`, and `responses-api-proxy` fit into production (auth, telemetry, etc.).
