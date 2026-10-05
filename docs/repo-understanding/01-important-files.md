# Important files and directories

This is a high-priority map rather than a monorepo inventory.

## Entrypoints and integration surfaces

| File / directory | Role | Why it matters |
| --- | --- | --- |
| `packages/coding-agent/src/main.ts` | Coding-agent CLI entry | Resolves CLI arguments, trust, settings, model/session state, resources, and selects interactive/print/JSON/RPC mode. |
| `packages/coding-agent/src/modes/` | Runtime interfaces | Separates terminal UI, one-shot output, JSON events, and long-lived RPC control from shared agent behavior. |
| `packages/coding-agent/src/core/sdk.ts` | Embedding API | Creates coding-agent sessions directly inside Node.js/Bun applications. |
| `packages/coding-agent/src/client/` and exported `RpcClient` | RPC client support | Provides a typed subprocess integration over Pi's RPC mode. |

## Core agent and session logic

| File / directory | Role | Why it matters |
| --- | --- | --- |
| `packages/agent/src/agent.ts` | Generic stateful agent | Owns transcript state, prompt/continue lifecycle, tools, queues, cancellation, and lifecycle events. |
| `packages/agent/src/` | Low-level agent loop/contracts | Keeps provider streaming and generic tool-call behavior reusable outside the coding-agent application. |
| `packages/coding-agent/src/core/agent-session.ts` | Coding-agent session | Adds coding-agent resources, persistence, compaction, extensions, tools, model behavior, and session events around the generic agent. |
| `packages/coding-agent/src/core/agent-session-runtime.ts` | Runtime/session replacement | Owns the current `AgentSession` plus cwd-bound services and handles new/resume/fork/import transitions. |
| `packages/coding-agent/src/core/agent-session-services.ts` | Cwd-bound services | Builds the settings/model/resource/service set against which a session is created. |
| `packages/coding-agent/src/core/session-manager.ts` | Session persistence/tree | Stores JSONL session entries, active-branch state, branching, and session metadata. |

## Model/provider layer

| File / directory | Role | Why it matters |
| --- | --- | --- |
| `packages/ai/src/` | Unified LLM layer | Defines provider/model APIs, streaming types, authentication contracts, model metadata, and provider implementations. |
| `packages/coding-agent/src/core/model-runtime.ts` | Coding-agent model runtime | Connects configured/available models and provider credentials to a live coding-agent session. |
| `packages/coding-agent/src/core/model-resolver.ts` | Model selection | Resolves CLI/settings model scope and selections. |

## Tools, extensions, and resources

| File / directory | Role | Why it matters |
| --- | --- | --- |
| `packages/coding-agent/src/tools/` | Built-in coding tools | Implements read/bash/edit/write and related tool capabilities. |
| `packages/coding-agent/src/core/extensions/` | Extension contracts/runtime | Allows behavior, tools, UI requests, hooks, and session events to be extended. |
| `packages/coding-agent/src/extensions/` | Built-in extensions | Hosts built-in MCP, codemode, tool-search, and other packaged extension behavior. |
| `packages/coding-agent/src/core/resource-loader.ts` and related resource code | Resource discovery | Brings in skills, prompts, themes, extensions, context files, and project resources. |
| `packages/codemode/` | Tool-composition runtime | Lets model-written QuickJS call/filter/compose other tools. |
| `packages/mcp/` | MCP support | Connects external MCP tools/resources to Pi's extension/tool system. |

## Configuration, trust, and persistence

| File / directory | Role | Why it matters |
| --- | --- | --- |
| `packages/coding-agent/src/core/settings-manager.ts` | Settings resolution | Merges user/project settings and exposes effective configuration. |
| `packages/coding-agent/src/core/trust-manager.ts` and project-trust files | Project trust | Gates project-local resources that can execute or alter agent behavior. |
| `packages/coding-agent/src/config.ts` | Installation/config paths | Centralizes package/config asset resolution across source, npm, and standalone forms. |
| `packages/coding-agent/docs/configuration.md` | Configuration map | Documents user and project config/resource locations. |
| `packages/coding-agent/docs/settings.md` | Settings reference | Documents model, tool, compaction, terminal, network, shell, and resource settings. |

## External interfaces

| File / directory | Role | Why it matters |
| --- | --- | --- |
| `packages/coding-agent/docs/cli-integration.md` | Interface comparison | States the lifetime and contract of interactive, print, JSON, RPC, and SDK integration. |
| `packages/coding-agent/docs/rpc.md` and `rpc-commands.md` | RPC protocol | Canonical reference for long-lived subprocess control. |
| `packages/coding-agent/docs/json.md` | JSON event stream | Defines structured event output and completion semantics. |
| `packages/coding-agent/docs/sdk.md` | SDK | Canonical in-process integration reference. |

## Tests and fixtures

| File / directory | Role | Why it matters |
| --- | --- | --- |
| `test.sh` | Repository test entry | Runs the non-e2e test suite using repository conventions. |
| `packages/coding-agent/test/` | Coding-agent tests | Covers CLI, sessions, RPC, configuration, extensions, tools, and integration behavior. |
| `packages/coding-agent/test/suite/` | Higher-level deterministic suite | Uses a faux provider/harness for coding-agent behavior without real provider calls. |
| `packages/ai/test/` | Provider/model tests | Validates provider adapters, model metadata, auth/environment behavior, and stream handling. |
| `packages/agent/test/` | Generic agent tests | Exercises the reusable agent runtime independently of the coding-agent UI. |
| `packages/tui/test/` | Terminal library tests | Covers rendering, input, terminal behavior, width, and UI primitives. |
