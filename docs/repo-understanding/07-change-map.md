# Change map

Use this as an orientation map, not an exhaustive edit list. Verify the current source before making a change.

## If I need to change CLI startup or mode selection

Start with:

- `packages/coding-agent/src/main.ts`
- CLI argument parsing under `packages/coding-agent/src/cli/`
- `packages/coding-agent/src/modes/`
- `docs/cli.md`
- `docs/cli-integration.md`

Why: startup resolves invocation mode, configuration, trust, resources, model/session state, and routes into the selected interface.

Caveat: interactive, print, JSON, and RPC should stay interfaces over shared session behavior rather than diverging agent implementations.

## If I need to change the generic agent loop

Start with:

- `packages/agent/src/agent.ts`
- other loop/type files in `packages/agent/src/`
- `packages/agent/test/`

Why: this package owns transcript state, prompt/continue semantics, tool execution, queues, events, cancellation, and provider-stream orchestration.

Caveat: coding-agent-specific concepts such as project resources or JSONL session trees should not leak into the generic agent core without a strong reason.

## If I need to change coding-agent session behavior

Start with:

- `packages/coding-agent/src/core/agent-session.ts`
- `agent-session-runtime.ts`
- `agent-session-services.ts`
- relevant extension/session events

Why: these files connect the generic Agent to persistence, tools, resources, compaction, extensions, and current cwd.

Caveat: distinguish changing one live AgentSession from replacing the whole cwd-bound runtime.

## If I need to change sessions, resume, fork, or branching

Start with:

- `packages/coding-agent/src/core/session-manager.ts`
- `packages/coding-agent/src/core/agent-session-runtime.ts`
- `docs/sessions.md`
- `docs/session-format.md`
- session tests

Why: the session manager owns persisted JSONL/tree semantics; runtime replacement owns switching the live agent to another persisted state.

Caveat: session files are user data and can outlive a release. Format changes should explicitly consider compatibility and import/resume behavior.

## If I need to change RPC integration

Start with:

- RPC mode implementation under `packages/coding-agent/src/modes/rpc/`
- exported `RpcClient`
- `docs/rpc.md`
- `docs/rpc-commands.md`
- `docs/json.md`
- `packages/coding-agent/test/rpc*.test.ts`

Why: RPC has explicit framing, correlation, event, and lifecycle contracts.

Caveat: a successful prompt command is not completion. Preserve `agent_settled` semantics and stdout-as-protocol discipline.

## If I need to change SDK embedding

Start with:

- `packages/coding-agent/src/core/sdk.ts`
- public exports in `packages/coding-agent/src/index.ts`
- SDK examples/docs
- `AgentSession` and session services

Why: the SDK exposes direct in-process access to the same engine used by the CLI.

Caveat: SDK embedding does not automatically set every CLI process marker/environment behavior; distinguish process-entry behavior from session APIs.

## If I need to change model/provider support

Start with:

- `packages/ai/src/`
- provider-specific implementation under `packages/ai/src/providers/` and `api/`
- coding-agent `model-runtime.ts` and `model-resolver.ts`
- provider/model tests

Why: provider protocol/model facts belong in the AI package; coding-agent code should primarily select and consume those abstractions.

Caveat: generated model data has explicit repository rules. Do not hand-edit generated model files when the generator is authoritative.

## If I need to add or change a tool

Start with:

- generic tool contracts in `packages/agent`
- concrete built-ins under `packages/coding-agent/src/tools/`
- coding-agent tool-selection/settings code
- relevant extension code if the tool is not a core built-in
- tool tests

Why: the low-level Agent executes generic tools, while the coding-agent layer supplies filesystem/shell-aware implementations.

Caveat: tool selection is a security surface. Adding a tool to the default set has different consequences from merely making it available.

## If I need to change shell execution or environment

Start with:

- coding-agent bash/powershell tool implementation
- `shellPath` / `shellCommandPrefix` settings
- session-environment injection
- `docs/environment-variables.md`

Why: shell tools are where Pi crosses from model intent into operating-system side effects.

Caveat: child shell processes cannot directly mutate the environment/cwd of the external parent process that launched Pi.

## If I need to change project resources or extensions

Start with:

- resource loading under `packages/coding-agent/src/core/`
- `src/core/extensions/`
- built-in extensions under `src/extensions/`
- project-trust implementation
- configuration/security/resource docs

Why: resource loading combines user, project, package, and explicit CLI sources, and some resources execute code or expose tools.

Caveat: preserve the project-trust boundary. Context-file discovery and `.pi` resource trust do not have identical semantics.

## If I need to change compaction or context projection

Start with:

- `packages/coding-agent/src/core/compaction/`
- session/context projection in session manager/session code
- compaction settings/docs
- agent/session tests

Why: persisted history, active-branch history, and current model context are related but not identical.

Caveat: avoid deleting source session entries merely to reduce model context; current design preserves durable history and changes the projection sent to the model.

## If I need to change terminal UI behavior

Start with:

- interactive mode under `packages/coding-agent/src/modes/interactive/`
- `packages/tui/src/`
- TUI tests
- the repository's interactive-testing guidance

Why: coding-agent interactive behavior composes the reusable TUI library with session events and commands.

Caveat: do not encode core agent behavior solely in TUI code if JSON/RPC/SDK modes need the same semantics.
