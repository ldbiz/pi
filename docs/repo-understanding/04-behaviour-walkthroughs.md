# Behaviour walkthroughs

## 1. Run a one-shot prompt

### Trigger

A caller runs:

`pi --print "Summarize this repository"`

or invokes Pi with redirected stdin/stdout such that print mode is selected.

### Main files

- `packages/coding-agent/src/main.ts`
- print-mode implementation under `packages/coding-agent/src/modes/`
- `packages/coding-agent/src/core/agent-session.ts`
- `packages/agent/src/agent.ts`
- `packages/ai/src/`

### Bird's-eye flow

Startup resolves the current working directory, settings/resources, model, credentials, and session behavior. The supplied prompt is passed into the common AgentSession/Agent loop.

The model can call the enabled tools and receive their results across multiple turns. When the run settles, print mode emits the final assistant text and exits.

### Inputs and outputs

**Inputs:** startup prompt and optional piped data, cwd, config/resources, provider credentials, model/tool selection.

**Outputs:** final assistant text on stdout, diagnostics on stderr, optional persisted session/tool side effects.

### Tests

Coding-agent CLI/mode tests and the deterministic faux-provider suite cover non-interactive behavior without requiring paid model calls.

---

## 2. Control a long-lived Pi process through RPC

### Trigger

A host starts:

`pi --mode rpc --no-session`

or starts RPC mode with persistent session/model/tool options.

### Main files

- RPC mode under `packages/coding-agent/src/modes/rpc/`
- `packages/coding-agent/docs/rpc.md`
- `packages/coding-agent/docs/rpc-commands.md`
- exported `RpcClient`
- `packages/coding-agent/examples/rpc-client.ts`

### Bird's-eye flow

Pi remains alive and reads one JSON command per line from stdin. Commands can prompt the agent, inspect/change state, manage sessions, invoke shell behavior, or respond to extension UI requests.

Command responses are correlated by optional IDs. Agent/session events are emitted independently as the run proceeds. The host must continuously read stdout and wait for settled lifecycle state when completion matters.

### Inputs and outputs

**Inputs:** JSONL commands; startup selection of cwd/model/tools/resources/session behavior.

**Outputs:** JSONL command responses and asynchronous lifecycle events on stdout; diagnostics on stderr.

### Tests

`packages/coding-agent/test/rpc.test.ts` and RpcClient-specific tests cover protocol/client behavior. The repository includes a typed example used by TypeScript checks.

---

## 3. Execute a model-requested shell command

### Trigger

The selected model returns a call to the enabled `bash` tool.

### Main files

- built-in shell-tool code under `packages/coding-agent/src/tools/`
- `packages/agent/src/agent.ts`
- AgentSession tool/extension orchestration
- shell settings and environment injection code

### Bird's-eye flow

The low-level Agent recognizes the tool call and invokes the coding-agent's registered bash tool.

The tool launches a command in the configured working directory/shell with current process environment plus Pi session metadata such as session ID, provider, model, and reasoning level. Output returns as a tool result and becomes available to the next model turn.

### Inputs and outputs

**Inputs:** model tool-call arguments, cwd, shell settings, current environment, Pi session metadata.

**Outputs:** command output/tool result and any filesystem/process/network side effects of the command.

### Tests

Coding-agent shell-tool tests use repository harnesses and controlled environments. Generic agent tests separately cover tool-call lifecycle without depending on the real shell implementation.

---

## 4. Resume, switch, or fork session work

### Trigger

The caller uses CLI/session commands, interactive session actions, SDK methods, or RPC commands to move to another session/branch.

### Main files

- `packages/coding-agent/src/core/session-manager.ts`
- `packages/coding-agent/src/core/agent-session-runtime.ts`
- `packages/coding-agent/src/core/agent-session-services.ts`
- `packages/coding-agent/docs/sessions.md`

### Bird's-eye flow

The session manager reads the JSONL tree and identifies the target active branch. Runtime replacement settles the outgoing session, emits lifecycle hooks, disposes it, and creates new cwd-bound services for the target session.

Forking can keep related alternatives inside one tree or create a distinct persisted session, depending on the operation. The model receives only the active branch plus current system/resources/tools, not every branch stored in the file.

### Inputs and outputs

**Inputs:** session ID/path/entry, target cwd where applicable, current trust/configuration state.

**Outputs:** new active AgentSession runtime and future persisted entries on the selected branch/session.

### Tests

Session manager, session runtime, CLI/session, and suite tests cover branching, loading, persistence, and replacement behavior.

---

## 5. Load project resources and extensions

### Trigger

Pi starts or reloads in a working directory containing trusted `.pi` configuration/resources, configured packages, or explicit resource paths.

### Main files

- settings/resource-loading code under `packages/coding-agent/src/core/`
- extension runtime under `src/core/extensions/`
- built-in extensions under `src/extensions/`
- project-trust code
- `docs/configuration.md`

### Bird's-eye flow

Pi resolves user-level resources and evaluates project trust before activating project-local resources that can affect execution. It then loads configured extensions, skills, prompts, themes, system-prompt material, MCP configuration, and tool changes.

Extensions can register tools and hooks around session/agent behavior. Skills and context become model-visible instructions rather than executable agent-loop replacements.

### Inputs and outputs

**Inputs:** agent-directory config, project `.pi`, packages, explicit CLI resources, trust choice.

**Outputs:** effective tools/resources/system instructions and extension hooks for the session.

### Tests

Configuration/resource/extension tests plus the higher-level suite cover loading and behavior with deterministic providers.
