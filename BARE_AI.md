# Gemini CLI Project Context

Gemini CLI is an open-source AI agent that brings the power of Gemini directly
into the terminal. It is designed to be a terminal-first, extensible, and
powerful tool for developers.

## Project Overview

- **Purpose:** Provide a seamless terminal interface for Gemini models,
  supporting code understanding, generation, automation, and integration via MCP
  (Model Context Protocol).
- **Main Technologies:**
  - **Runtime:** Node.js (>=20.0.0, recommended ~20.19.0 for development)
  - **Language:** TypeScript
  - **UI Framework:** React (using [Ink](https://github.com/vadimdemedes/ink)
    for CLI rendering)
  - **Testing:** Vitest
  - **Bundling:** esbuild
  - **Linting/Formatting:** ESLint, Prettier
- **Architecture:** Monorepo structure using npm workspaces.
  - `packages/cli`: User-facing terminal UI, input processing, and display
    rendering.
  - `packages/core`: Backend logic, Gemini API orchestration, prompt
    construction, and tool execution.
  - `packages/core/src/tools/`: Built-in tools for file system, shell, and web
    operations.
  - `packages/a2a-server`: Experimental Agent-to-Agent server.
  - `packages/vscode-ide-companion`: VS Code extension pairing with the CLI.

## Building and Running

- **Install Dependencies:** `npm install`
- **Build All:** `npm run build:all` (Builds packages, sandbox, and VS Code
  companion)
- **Build Packages:** `npm run build`
- **Run in Development:** `npm run start`
- **Run in Debug Mode:** `npm run debug` (Enables Node.js inspector)
- **Bundle Project:** `npm run bundle`
- **Clean Artifacts:** `npm run clean`

## Testing and Quality

- **Test Commands:**
  - **Unit (All):** `npm run test`
  - **Integration (E2E):** `npm run test:e2e`
  - **Workspace-Specific:** `npm test -w <pkg> -- <path>` (Note: `<path>` must
    be relative to the workspace root, e.g.,
    `-w @google/gemini-cli-core -- src/routing/modelRouterService.test.ts`)
- **Full Validation:** `npm run preflight` (Heaviest check; runs clean, install,
  build, lint, type check, and tests. Recommended before submitting PRs. Due to
  its long runtime, only run this at the very end of a code implementation task.
  If it fails, use faster, targeted commands (e.g., `npm run test`,
  `npm run lint`, or workspace-specific tests) to iterate on fixes before
  re-running `preflight`. For simple, non-code changes like documentation or
  prompting updates, skip `preflight` at the end of the task and wait for PR
  validation.)
- **Individual Checks:** `npm run lint` / `npm run format` / `npm run typecheck`

## Development Conventions

- **Legacy Snippets:** `packages/core/src/prompts/snippets.legacy.ts` is a
  snapshot of an older system prompt. Avoid changing the prompting verbiage to
  preserve its historical behavior; however, structural changes to ensure
  compilation or simplify the code are permitted.
- **Contributions:** Follow the process outlined in `CONTRIBUTING.md`. Requires
  signing the Google CLA.
- **Pull Requests:** Keep PRs small, focused, and linked to an existing issue.
  Always activate the `pr-creator` skill for PR generation, even when using the
  `gh` CLI.
- **Commit Messages:** Follow the
  [Conventional Commits](https://www.conventionalcommits.org/) standard.
- **Coding Style:** Adhere to existing patterns in `packages/cli` (React/Ink)
  and `packages/core` (Backend logic).
- **Imports:** Use specific imports and avoid restricted relative imports
  between packages (enforced by ESLint).
- **License Headers:** For all new source code files (`.ts`, `.tsx`, `.js`),
  include the Apache-2.0 license header with the current year. (e.g.,
  `Copyright 2026 Google LLC`). This is enforced by ESLint.

## Testing Conventions

- **Environment Variables:** When testing code that depends on environment
  variables, use `vi.stubEnv('NAME', 'value')` in `beforeEach` and
  `vi.unstubAllEnvs()` in `afterEach`. Avoid modifying `process.env` directly as
  it can lead to test leakage and is less reliable. To "unset" a variable, use
  an empty string `vi.stubEnv('NAME', '')`.

## Documentation

- Always use the `docs-writer` skill when you are asked to write, edit, or
  review any documentation.
- Documentation is located in the `docs/` directory.
- Suggest documentation updates when code changes render existing documentation
  obsolete or incomplete.

---

## Bare AI CLI — Sovereign Extensions

This fork adds the following environment variables and behaviours on top of the upstream Gemini CLI:

### Environment Variables

| Variable | Required | Description |
|---|---|---|
| `BARE_AI_ENDPOINT` | No | OpenAI-compatible chat completions endpoint. Defaults to `http://localhost:11434/v1/chat/completions` |
| `BARE_AI_API_KEY` | No | API key for the endpoint. Defaults to `none` for local Ollama |
| `BARE_AI_MODEL` | No | Model name to use. Defaults to `default` |
| `BARE_AI_CONSTITUTION` | No | Path to constitution markdown file. Defaults to `~/.bare-ai/constitution.md` |
| `BARE_AI_SEARCH_URL` | No | URL of local SearXNG instance for sovereign web search. If unset, falls back to Google via Gemini API |
| `BARE_AI_LEAN_TOOLS` | No | Set to `true` to force lean mode, `false` to disable it regardless of model name |
| `DEBUG_BARE_AI` | No | Set to `true` to enable verbose request/response logging to `bare-ai-trace.log` |

### Lean Mode

Models with names containing `tiny`, `small`, `mini`, `1b`, or `3b` automatically enter Lean Mode, which:
- Strips tool schemas to required parameters only
- Filters tools to essential set: `run_shell_command`, `read_file`, `write_file`, `list_directory`, `google_web_search`, `web_fetch`
- Injects `num_ctx: 8192` into Ollama options to expand the context window

### Sovereign Web Search

Set `BARE_AI_SEARCH_URL` to route web searches through a local SearXNG instance:
```bash
# Quick start with Docker
docker run -d --name searxng --restart unless-stopped \
  -p 8080:8080 \
  --security-opt seccomp=unconfined \
  --security-opt apparmor=unconfined \
  -v searxng-data:/etc/searxng \
  searxng/searxng

export BARE_AI_SEARCH_URL="http://localhost:8080"
```

Results are pruned to the top 5 (title, URL, snippet) before being passed to the model.
