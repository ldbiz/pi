# Architecture diagrams

## 1. Coding-agent runtime

```mermaid
flowchart TD
    CLI["packages/coding-agent/src/main.ts"] --> Resolve["CLI + config + trust + resources + auth/model"]
    Resolve --> Services["AgentSessionServices for cwd"]
    Services --> Runtime["AgentSessionRuntime"]
    Runtime --> Session["AgentSession"]
    Session --> Agent["pi-agent-core Agent"]

    CLI --> Mode{"Mode"}
    Mode --> TUI["Interactive"]
    Mode --> Print["Print"]
    Mode --> JSON["JSON events"]
    Mode --> RPC["RPC"]

    TUI --> Session
    Print --> Session
    JSON --> Session
    RPC --> Session

    Agent --> AI["pi-ai"]
    AI --> Providers["LLM providers"]
    Providers --> AI
    AI --> Agent

    Agent --> Tools["Coding / extension tools"]
    Tools --> Agent

    Session --> Store["SessionManager / JSONL tree"]
    Session --> Ext["Extensions + resources"]
```

## 2. Monorepo interaction map

```mermaid
flowchart LR
    Coding["pi-coding-agent"] --> Agent["pi-agent-core"]
    Coding --> AI["pi-ai"]
    Coding --> TUI["pi-tui"]
    Coding --> CodeMode["pi-codemode"]
    Coding --> MCP["pi-mcp"]
    Coding --> Chord["chord"]
    Coding --> Protocol["protocol / client / server"]
    Agent --> AI
    MCP --> Ext["Coding-agent extension/tool system"]
    CodeMode --> Ext
    Ext --> Coding
    AI --> Providers["External LLM providers"]
```

## 3. RPC prompt sequence

```mermaid
sequenceDiagram
    participant H as Host
    participant R as Pi RPC mode
    participant S as AgentSession
    participant A as Agent
    participant M as Model/provider
    participant T as Tool

    H->>R: JSONL prompt command (id)
    R->>S: Submit prompt
    R-->>H: response: accepted/queued/handled

    S->>A: Start/continue run
    A->>M: Stream request
    M-->>A: Assistant stream / tool call
    A-->>S: Lifecycle events
    S-->>R: Session/agent events
    R-->>H: JSONL events

    alt Tool requested
        A->>T: Execute tool
        T-->>A: Tool result
        A->>M: Continue with result
        M-->>A: Next response
    end

    A-->>S: Run completes
    S-->>R: agent_settled
    R-->>H: agent_settled event
```

## 4. Session/runtime replacement

```mermaid
flowchart TD
    Current["Current AgentSessionRuntime"] --> Request{"new / resume / fork / import"}
    Request --> Before["before-switch/fork extension hooks"]
    Before --> Settle["Abort/settle active work"]
    Settle --> Shutdown["session_shutdown + dispose old session"]
    Shutdown --> Manager["Create/open target SessionManager"]
    Manager --> Services["Recreate cwd-bound services"]
    Services --> NewSession["Create target AgentSession"]
    NewSession --> Rebind["Rebind host/UI to new session"]
    Rebind --> Active["New active runtime"]
```

## 5. Configuration and resource boundaries

```mermaid
flowchart TD
    User["~/.pi/agent"] --> Merge["Effective session resources/settings"]
    Project["project .pi/"] --> Trust{"Project trusted?"}
    Trust -->|yes| Merge
    Trust -->|no| Skip["Skip trust-gated project resources"]
    Context["AGENTS/CLAUDE context files"] --> Merge
    CLI["Explicit CLI resources/options"] --> Merge
    Packages["Pi packages"] --> Merge
    Merge --> Session["AgentSession"]
```

The diagrams intentionally separate the generic Agent, coding-agent session/runtime, interface modes, and persisted session tree. Those are distinct ownership boundaries even though the end-user experiences them as one `pi` process.
