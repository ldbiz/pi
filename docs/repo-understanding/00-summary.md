# Pi repository summary

## Purpose

Pi is a minimal, extensible agent harness with a full coding-agent CLI and reusable lower-level packages. It can run as an interactive terminal application, a one-shot command, a JSON event producer, a long-lived RPC subprocess, or an in-process TypeScript SDK.

The repository is a monorepo: the coding-agent application is assembled from independent packages for model/provider access, agent state and tool calling, terminal UI, durable state, RPC/protocol support, MCP/codemode integration, and other runtime services.

## Technology stack

| Area | Implementation |
| --- | --- |
| Primary language | TypeScript |
| Runtime | Node.js 22.19+; Bun is also a supported embedding/build target in documented surfaces |
| Package/build | npm workspaces, TypeScript, esbuild/Bun for bundled/standalone outputs |
| Coding-agent tests | Vitest plus focused suite harnesses; TUI also uses Node's test runner |
| Terminal UI | `@earendil-works/pi-tui` with differential rendering and native platform helpers |
| LLM access | `@earendil-works/pi-ai` multi-provider model/streaming layer |
| Agent runtime | `@earendil-works/pi-agent-core` |
| Persistence | JSONL session files, branching session tree, compaction/summaries |
| External control | Print, JSONL event mode, long-lived JSONL RPC, TypeScript SDK |

## Monorepo packages

The root build includes these major packages:

- `packages/ai` — unified provider/model API and model metadata.
- `packages/agent` — general agent loop, transcript state, tools, queues, lifecycle events.
- `packages/coding-agent` — end-user CLI, sessions, configuration, extensions, resources, tools, and modes.
- `packages/tui` — terminal UI/rendering library.
- `packages/durable` — durable conversation/task/document runtime.
- `packages/chord` — application-composition runtime for services, replicated state, RPC, and plugins.
- `packages/codemode` and `packages/mcp` — tool-composition and MCP integration.
- `packages/protocol`, `packages/client`, `packages/server` — typed protocol/client/server surfaces used by broader Pi infrastructure.
- `packages/telemetry` — telemetry contracts and adapters.

## Main runtime model

`packages/coding-agent/src/main.ts` parses CLI state, resolves trust/configuration/model/session resources, creates an `AgentSession` runtime, then selects a presentation/control mode.

All CLI modes use the same underlying agent/session machinery:

- **Interactive** — terminal UI until the user exits.
- **Print** — one invocation, final assistant text on stdout.
- **JSON** — one invocation, structured JSONL lifecycle/events on stdout.
- **RPC** — long-lived subprocess accepting JSONL commands and emitting responses/events.
- **SDK** — in-process TypeScript integration rather than a CLI mode.

The lower-level `Agent` owns the active transcript, lifecycle events, tool execution, steering/follow-up queues, cancellation, and repeated model turns.

## Configuration and state

User configuration defaults to `~/.pi/agent`. Project configuration lives under `.pi` and is subject to project-trust rules. Sessions default under `~/.pi/agent/sessions/`, grouped by working directory.

Project resources can include settings, MCP servers, extensions, skills, prompt templates, themes, system-prompt files, and context files.

## Main external services and dependencies

Pi supports multiple LLM providers through `pi-ai`, with credentials in environment/configured auth storage. Extensions can add tools and behavior; the built-in MCP extension connects external MCP servers.

Pi itself does **not** impose a filesystem/process/network permission sandbox. Tools run with the permissions of the process unless the host or deployment applies external sandboxing/containerization.

## Main entry points

- `packages/coding-agent/src/main.ts` — coding-agent CLI startup and mode selection.
- `packages/coding-agent/src/core/sdk.ts` — in-process session creation API.
- `packages/coding-agent/src/core/agent-session.ts` — coding-agent orchestration around the low-level agent.
- `packages/coding-agent/src/core/agent-session-runtime.ts` — current session plus cwd-bound service lifecycle.
- `packages/coding-agent/src/modes/` — interactive, print/JSON, and RPC interfaces.
- `packages/agent/src/agent.ts` — stateful low-level agent wrapper and agent-loop API.
- `packages/coding-agent/src/core/session-manager.ts` — JSONL session tree and persistence.

## Read these first

1. `README.md` — product scope and monorepo package map.
2. `AGENTS.md` — repository engineering rules and testing guidance.
3. `packages/coding-agent/src/main.ts` — CLI startup and mode routing.
4. `packages/coding-agent/docs/cli-integration.md` — clearest comparison of interactive, print, JSON, RPC, and SDK integration.
5. `packages/coding-agent/src/core/agent-session.ts` — coding-agent behavior above the generic agent loop.
6. `packages/agent/src/agent.ts` — low-level agent state, prompt lifecycle, queues, tools, and cancellation.
7. `packages/coding-agent/src/core/agent-session-runtime.ts` — session replacement and cwd-bound services.
8. `packages/coding-agent/src/core/session-manager.ts` — persisted branching session model.
9. `packages/coding-agent/docs/configuration.md` and `settings.md` — resource and behavior configuration.
10. `packages/coding-agent/docs/rpc.md` and `sdk.md` — external control surfaces.
