# Tests and fixtures

## Test organization

Pi is a monorepo with package-specific tests plus root-level orchestration.

The repository guidance distinguishes between:

- ordinary non-e2e tests, normally run through `./test.sh`;
- focused package tests;
- coding-agent deterministic suite tests;
- package-specific Node test runners where applicable.

The root `npm run check` command is the main static-quality gate after code changes. It runs formatting/lint checks, dependency/install-lock validations, TypeScript checks, entry-graph checks, and browser smoke validation. It is not the normal test runner.

## Coding-agent tests

`packages/coding-agent/test/` covers the end-user agent layer.

Representative areas visible in the repository include:

- CLI argument/path behavior;
- session manager/runtime behavior;
- RPC protocol and `RpcClient`;
- model/resource/configuration handling;
- extensions;
- tools;
- higher-level regression scenarios.

RPC coverage includes `rpc.test.ts` plus client-specific tests such as clone, queue, and process-exit behavior.

## Deterministic suite

`packages/coding-agent/test/suite/` is a higher-level behavioral suite.

The repository's `AGENTS.md` explicitly requires the suite harness plus the **faux provider** for these tests. This prevents regressions from depending on real API keys, provider uptime, or paid tokens.

`packages/coding-agent/test/suite/harness.ts` centralizes this setup. Regressions can then exercise realistic agent/session/tool flows against scripted provider behavior.

## Generic agent tests

`packages/agent/test/` tests `pi-agent-core` independently of the coding-agent UI.

This is important because prompt lifecycle, tool-call execution, agent events, queues, cancellation, and provider-stream behavior are generic runtime responsibilities rather than terminal concerns.

## AI/provider tests

`packages/ai/test/` covers the multi-provider/model layer.

Visible test areas include:

- model types and catalogs;
- provider registration and compatibility;
- generated/model metadata;
- environment compatibility;
- faux provider behavior;
- images;
- provider response IDs and reasoning levels.

These tests establish provider translation and model semantics without requiring the coding-agent application around them.

## TUI tests

`packages/tui/test/` uses Node's built-in test runner and exercises terminal-library behavior such as:

- width and wrapping;
- terminal rendering;
- fuzzy matching;
- colors;
- input buffering;
- shrink/redraw behavior;
- settings list/UI primitives.

Interactive coding-agent behavior has additional test guidance under the repository's dedicated interactive-testing skill.

## Fixtures and faux providers

The faux provider is a major deterministic fixture across agent and coding-agent tests. It allows scripted model streams/tool calls without remote services.

Coding-agent suite helpers build realistic sessions around that provider so tests can focus on behavior rather than transport availability.

The repository also uses temporary directories, test-specific settings/resources, and isolated process invocations for CLI/RPC behavior.

## What the tests document well

The visible suite provides broad separation of concerns:

- `pi-ai` validates provider/model contracts;
- `pi-agent-core` validates generic agent behavior;
- `pi-coding-agent` validates sessions, tools, resources, modes, and integration;
- `pi-tui` validates terminal primitives.

This package layering makes it possible to test most behavior below a full interactive terminal session.

## Limits and gaps

These are constraints of the test model rather than necessarily missing work:

- Remote providers can change independently of the source tree; faux-provider tests cannot prove live API compatibility forever.
- RPC/SDK behavior can be tested deterministically, but host-process integration still has OS/process-boundary behavior outside the agent loop.
- The TUI has platform-specific native helpers, so complete assurance depends on relevant OS coverage.
- The default tool set can mutate the filesystem and run arbitrary shell commands; unit tests cannot substitute for a deliberate host security model.

## Validation guidance for this documentation branch

This branch changes documentation only.

Useful validation is therefore:

- compare `repo-analysis` against `main` and confirm only `docs/repo-understanding/` was added;
- verify every referenced path exists at this revision;
- preview Mermaid diagrams.

The repository explicitly reserves `npm run check` for code changes, and its guidance says not to run the full build or test suite unless requested. A full runtime test is not needed for a documentation-only branch.
