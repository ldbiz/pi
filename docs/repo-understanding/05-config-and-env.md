# Configuration and environment

## Configuration locations

Pi has user-level and project-level configuration.

### Agent directory

The agent directory defaults to `~/.pi/agent` and can be relocated with `PI_CODING_AGENT_DIR`.

Important paths include:

| Path | Purpose |
| --- | --- |
| `settings.json` | User preferences, defaults, resources, and package declarations |
| `keybindings.json` | TUI/application keybindings |
| `mcp.json` | User-level MCP servers |
| `models.json` | Compatible endpoints and model overrides |
| `auth.json` | Saved provider credentials |
| `AGENTS*.md` / `CLAUDE*.md` | User-level context instructions |
| `SYSTEM.md` | Replace the default system prompt |
| `APPEND_SYSTEM.md` | Append to the system prompt |
| `extensions/`, `skills/`, `prompts/`, `themes/` | User resources |
| `sessions/` | Default root for persisted sessions |

### Project configuration

Project-level configuration lives in `.pi` under the working directory. It can contain:

- `settings.json`
- `mcp.json`
- `SYSTEM.md`
- `APPEND_SYSTEM.md`
- `extensions/`
- `skills/`
- `prompts/`
- `themes/`

Project configuration loads after project trust is granted, except for documented bootstrap behavior such as locating a configured session directory.

Context files such as `AGENTS.md` are discovered separately from `.pi` project configuration and have their own inheritance/override rules.

## Model and thinking settings

Important settings include:

| Setting | Default | Purpose |
| --- | --- | --- |
| `defaultProvider` | automatic | Startup provider |
| `defaultModel` | automatic | Startup model |
| `defaultThinkingLevel` | medium | Startup reasoning level |
| `enabledModels` | all available | Startup/model-cycle scope |
| `thinkingBudgets` | built in | Token budgets for reasoning levels |
| `cacheWarming` | streaming | Keep eligible provider caches warm when economically useful |

CLI `--provider`, `--model`, `--models`, `--thinking`, and `--api-key` options can override normal selection for one process.

## Tool settings

The default model-callable tool selection is:

- `read`
- `bash`
- `edit`
- `write`

Other built-ins include `powershell`, `grep`, `find`, and `ls`. Built-in extensions can expose `codemode` and `tool_search`.

`defaultTools` can replace the inherited tool set or modify it using `+name` / `-name` entries. CLI flags such as `--tools`, `--exclude-tools`, `--no-builtin-tools`, and `--no-tools` override startup selection.

This is an important security/integration control because enabled tools execute with the Pi process's operating-system permissions.

## Sessions and compaction

| Setting / control | Purpose |
| --- | --- |
| `sessionDir` | Configure persisted-session storage |
| `PI_CODING_AGENT_SESSION_DIR` | Environment override for session storage |
| `--session-dir` | Highest-precedence CLI session directory |
| `--no-session` | Use an ephemeral in-memory session |
| `compaction.enabled` | Enable automatic compaction |
| `compaction.reserveTokens` | Reserve context for model output |
| `compaction.keepRecentTokens` | Preserve recent history without summarization |
| `branchSummary.*` | Control summaries when moving between branches |

The default session root is under the agent directory and is grouped by working directory.

## Shell settings

| Setting | Purpose |
| --- | --- |
| `shellPath` | Override the shell executable |
| `shellCommandPrefix` | Prefix every model-callable shell command |
| `npmCommand` | Override npm command/arguments used for package operations |

Model-callable shell tools also receive session-specific environment metadata.

## Session environment injected into tools

The built-in `bash` and `powershell` tools receive:

| Variable | Meaning |
| --- | --- |
| `PI_SESSION_ID` | Current Pi session ID |
| `PI_SESSION_FILE` | Current session JSONL path when persisted |
| `PI_PROVIDER` | Selected provider |
| `PI_MODEL` | Selected model ID |
| `PI_REASONING_LEVEL` | Current effective reasoning level |

These values are resolved for each command invocation, so model/reasoning changes affect subsequent tool calls.

They are specifically injected into model-callable shell tools, not ordinary user-entered shell escapes.

## Process environment

Important Pi process variables include:

| Variable | Purpose |
| --- | --- |
| `PI_CODING_AGENT_DIR` | Move the agent/config directory |
| `PI_CODING_AGENT_SESSION_DIR` | Override session storage |
| `PI_PACKAGE_DIR` | Override package asset root |
| `PI_OFFLINE` | Disable automatic network activity such as model-catalog refresh |
| `PI_SKIP_VERSION_CHECK` | Disable latest-version check |
| `PI_TELEMETRY` | Override install/update telemetry and attribution behavior |
| `PI_CACHE_RETENTION` | Request extended provider prompt caching where supported |
| `HTTP_PROXY`, `HTTPS_PROXY` | Proxy outbound HTTP |

Pi also sets:

- `AI_AGENT=pi`
- `PI_CODING_AGENT=true`

for child processes launched from the CLI/RPC entry points.

Provider API-key variables are provider-specific and documented with provider configuration.

## Network and retry settings

Settings include:

- `transport` — automatic/SSE/WebSocket variants;
- `httpProxy`;
- HTTP/WebSocket timeouts;
- agent-level retry enable/count/backoff;
- provider-level retry limits.

Provider-level retries default to zero so Pi can handle quota/usage conditions at the agent layer rather than allowing a provider adapter to hide prolonged retry behavior.

## Resources

Resource settings can point to packages, extensions, skills, prompts, and themes. User and project resource lists are combined.

Project resources are powerful: extensions and MCP definitions can execute code or expose tools. Project trust is therefore part of configuration semantics, not merely a UI preference.
