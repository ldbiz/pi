# Caveats and unknowns

## Scope of this analysis

This pack describes the repository at the `repo-analysis` branch point. It focuses on the coding-agent execution path and the packages that explain it, not every package or experimental subsystem in the monorepo.

The repository is large and fast-moving. Before implementation work, verify task-critical paths against the current branch rather than treating this pack as a substitute for source inspection.

## Permission model

Pi explicitly does not provide a built-in permission system that restricts filesystem, process, network, or credential access. By default it runs with the permissions of the user/process that launched it.

Project trust controls whether project-local resources are loaded; it is **not** an operating-system sandbox for model-called tools.

A host that automatically routes natural-language shell input into Pi should therefore decide whether to:

- restrict the enabled tools;
- wrap tool execution;
- run Pi in an external sandbox/container;
- or accept normal user-process permissions.

## RPC is long-lived but not the parent shell

RPC mode is a strong integration surface for an external host because the Pi process remains alive and can maintain agent/session state.

It still runs as a child process. Ordinary child-process semantics mean Pi or a Pi-called shell cannot directly mutate the parent interactive shell's cwd, exported variables, aliases, functions, shell options, or other internal state.

If an integration needs commands to change the caller's shell state, the host must define an explicit protocol for returning/applying those changes. Do not confuse persistent **agent context** with persistent **parent-shell process state**.

## Current shell context versus launch-time context

The working directory controls project configuration, resource discovery, and session grouping. Shell tools inherit process environment plus Pi session metadata.

For a host that keeps one RPC Pi process alive while the user's outer shell changes directories or environment variables, verify how and when Pi can be rebound or supplied updated cwd/environment state. A single long-lived subprocess does not automatically track arbitrary mutations in its parent after launch.

This is particularly relevant to terminal integrations that want the agent to behave exactly as if it had been invoked afresh from the current prompt.

## RPC lifecycle details matter

A successful RPC `prompt` response only means the prompt was accepted, queued, or handled.

Hosts must consume asynchronous events and use the documented settled state when they need to know that all automatic work has completed. `agent_end` alone can be followed by recovery, compaction, steering, or queued follow-up work.

RPC also reserves stdout for strict JSONL framing. Host integrations should not mix display output with protocol parsing.

## SDK versus RPC

The repository recommends:

- SDK for in-process Node.js/Bun integrations needing direct APIs;
- RPC for language-independent or process-isolated integrations.

The better choice for a specific host depends on ownership of process lifecycle, language/runtime, desired crash isolation, and how much Pi state the host wants to access directly.

Do not introduce a custom integration layer before checking whether RPC or SDK already exposes the required operation.

## Project trust and context files

Project `.pi` resources are trust-gated because extensions/settings/MCP configuration can affect execution.

Context-file discovery is documented separately and does not require project trust in the same way. Security-sensitive changes should inspect the exact trust/resource loading rules rather than assuming every project-local file is gated identically.

## Session data sensitivity

Session files can contain:

- prompts and assistant messages;
- tool arguments;
- command output;
- file contents;
- extension messages;
- provider/model metadata.

Exports/shares should therefore be treated as potentially sensitive. This is especially important if a host wants to inspect or synchronize session JSONL automatically.

## Public API stability

Pi exposes several integration layers: CLI, JSON, RPC, SDK, extension APIs, and lower-level packages.

The repository documentation is the authoritative contract for the current version, but this analysis does not establish long-term compatibility guarantees for every TypeScript internal path. Prefer documented/public exports over importing deep internal modules from a host integration.

## Build/source-checkout behavior

Some examples, especially the RPC client from a source checkout, require the coding-agent package to be built so a runnable CLI exists.

Installed releases and source checkouts therefore have different startup details even when the runtime protocol is the same.

## Areas not exhaustively analyzed

This pack does not deeply document:

- `pi-durable`;
- Chord's internal replicated-state/plugin machinery;
- the separate protocol/client/server stack beyond its relationship to the root build;
- every built-in extension;
- release/installer internals;
- every provider adapter.

Those areas should get a focused source pass if a future task directly depends on them.

## Questions worth resolving before a shell-host integration

1. Should the host use long-lived RPC or embed the SDK?
2. Which Pi tools should be enabled for automatically detected natural-language input?
3. Which outer-shell state must be synchronized on every request: cwd, exported environment, last command/status, history, aliases/functions, or terminal output?
4. When Pi wants an action that must affect the parent shell, should it return a proposed shell operation for the host to apply rather than executing it internally?
5. Should one persistent Pi session follow an outer terminal session, or should requests be ephemeral/forked based on cwd/repository?
6. Which project resources should be trusted automatically, if any?

These are host-integration decisions rather than unresolved questions about Pi's basic architecture.
