# Core concepts

## Agent

**Meaning in this repo:** The reusable stateful model/tool loop implemented by `@earendil-works/pi-agent-core`.

It owns the current transcript, selected model and tools, lifecycle events, active run, steering/follow-up queues, cancellation, and continuation behavior.

**Related files:** `packages/agent/src/agent.ts` and the rest of `packages/agent/src/`.

## AgentSession

**Meaning in this repo:** The coding-agent layer around the generic Agent. It connects the model loop to coding tools, session persistence, compaction, resources, extension hooks, and coding-agent lifecycle events.

**Related files:** `packages/coding-agent/src/core/agent-session.ts`, `sdk.ts`.

## AgentSessionRuntime

**Meaning in this repo:** The owner of the current `AgentSession` plus services bound to its effective working directory.

It is responsible for replacing the active runtime on new/resume/fork/import while cleanly shutting down the prior session and recreating cwd-dependent services.

**Related files:** `packages/coding-agent/src/core/agent-session-runtime.ts`, `agent-session-services.ts`.

## Session tree

**Meaning in this repo:** Persisted conversation state is not just a linear transcript. Session entries form a tree, and the active branch determines what conversation history is supplied to the model.

Moving within a session can preserve abandoned branches; forking can materialize a branch as a separate session.

**Related files:** `packages/coding-agent/src/core/session-manager.ts`, `docs/sessions.md`, `docs/session-format.md`.

## Active branch

**Meaning in this repo:** The path through a session tree used to reconstruct the current conversation context.

Only the active branch is supplied as historical conversation context; unrelated branches remain persisted but are not all sent to the model.

**Related files:** session manager and context/projection helpers.

## Compaction

**Meaning in this repo:** Summarization of older active-branch history when context pressure grows, while retaining original persisted session entries.

Pi keeps recent messages directly and inserts summary context so long work can continue within model limits.

**Related files:** `packages/coding-agent/src/core/compaction/`, `docs/compaction.md`, settings.

## Steering and follow-up queues

**Meaning in this repo:** Messages that can be queued around an active agent run.

- **Steering** messages are injected after the current assistant turn.
- **Follow-up** messages run after the agent would otherwise stop.

Queue mode determines whether one or all queued messages are drained at a time.

**Related files:** `packages/agent/src/agent.ts`, `settings.md`.

## Application mode

**Meaning in this repo:** The external interface used to control and observe the same coding-agent/session engine.

The main modes are interactive, print, JSON, and RPC. The SDK embeds the engine directly and is therefore not itself a CLI mode.

**Related files:** `packages/coding-agent/src/main.ts`, `src/modes/`, `docs/cli-integration.md`.

## RPC mode

**Meaning in this repo:** A long-lived child-process protocol using newline-delimited JSON on stdin/stdout.

A client can submit prompts, inspect/change state, manage sessions, run shell commands, and consume lifecycle events without entering the TUI.

**Related files:** `packages/coding-agent/src/modes/rpc/`, `docs/rpc.md`, `docs/rpc-commands.md`, `examples/rpc-client.ts`.

## Resource

**Meaning in this repo:** User/project material loaded into or alongside the coding agent: extensions, skills, prompt templates, themes, system prompts, MCP configuration, and context files.

Resources can come from conventional directories, explicit CLI paths, settings, or packages.

**Related files:** coding-agent resource loading, `docs/configuration.md`, `docs/settings.md`.

## Extension

**Meaning in this repo:** Executable/custom behavior loaded into the coding agent. Extensions can register tools, hook agent/session behavior, add commands, and interact with supported user interfaces.

**Related files:** `packages/coding-agent/src/core/extensions/`, `src/extensions/`, `docs/extensions.md`.

## Tool

**Meaning in this repo:** A model-callable capability. Default coding-agent tools are read, bash, edit, and write; other built-ins and extension tools can be enabled or disabled.

The lower-level Agent treats tools generically; the coding-agent package supplies repository/shell-oriented implementations.

**Related files:** `packages/agent/src/`, `packages/coding-agent/src/tools/`, CLI/settings docs.

## Codemode

**Meaning in this repo:** A built-in extension/tool that executes model-written JavaScript in a QuickJS sandbox to call, combine, parallelize, search, and filter other tools.

The QuickJS sandbox is for the composition script; the tools it calls retain their own real-world capabilities.

**Related files:** `packages/codemode/`, coding-agent built-in codemode extension, `docs/codemode.md`.

## Project trust

**Meaning in this repo:** The decision controlling whether project-local configuration/resources that can alter behavior are loaded.

Project `.pi` configuration is trust-gated, with documented exceptions such as session-directory resolution. Context files have their own discovery semantics.

**Related files:** project-trust/trust-manager code, `docs/configuration.md`, `docs/security.md`.

## Session environment

**Meaning in this repo:** Metadata injected into model-callable shell tools, including current Pi session ID/file, provider, model, and reasoning level.

This lets subprocesses called by the model identify their enclosing Pi session without parsing the system prompt.

**Related files:** shell-tool code and `docs/environment-variables.md`.
