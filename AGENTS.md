# AGENTS.md

AI coding agent instructions for **@nanocollective/nanocoder**

## Project Overview

A local-first CLI coding agent that brings the power of agentic coding tools like Claude Code and Gemini CLI to local models or controlled APIs like OpenRouter

**Project Type:** React Web Application
**Primary Language:** TypeScript (99% of codebase)

## Architecture

**Key Frameworks & Libraries:**
- React (^19.0.0) - web

**Project Structure:**
- `assets/` - Static assets
- `docs/` - Documentation
- `source/` - Source code
- `.github/assets/` - Static assets
- `docs/configuration/` - Project files
- `docs/features/` - Project files
- `docs/getting-started/` - Project files
- `source/ai-sdk-client/` - Project files
- `source/app/` - Application code
- `source/auth/` - Project files
- `source/commands/` - Project files
- `source/components/` - React/UI components
- `source/config/` - Configuration files
- `source/context/` - Project files
- `source/custom-commands/` - Project files
- `source/hooks/` - Project files
- `source/init/` - Project files
- `source/lsp/` - Project files
- `source/markdown-parser/` - Project files
- `source/mcp/` - Project files

## Key Files

**Configuration:**
- `package.json` - Node.js dependencies and scripts
- `plugins/vscode/package.json` - Node.js dependencies and scripts
- `pnpm-lock.yaml` - Configuration file

**Documentation:**
- `docs/**.md`
- `README.md`
- `.devcontainer/README.md`
- `.github/ISSUE_TEMPLATE/bug_report.md`
- `.github/ISSUE_TEMPLATE/feature_request.md`

## Development Commands

**Build:**
```bash
npm run build
```

**Development:**
```bash
npm run dev
```

**Start:**
```bash
npm run start
```

## Code Style Guidelines

- Use camelCase for variables and functions
- Use PascalCase for classes and components
- Prefer const/let over var
- Use async/await over callbacks when possible
- Use functional components with hooks
- Follow React naming conventions for components

## Testing

**Test Files:**
- `.nanocoder/commands/test.md`
- `scripts/test-copilot.sh`
- `scripts/test.sh`
- `source/ai-sdk-client/ai-sdk-client.spec.ts`
- `source/ai-sdk-client/chat/chat-handler.spec.ts`

## Existing Project Guidelines

*The following guidelines were found in existing AI configuration files:*

### AI Agent Guidelines
**From CLAUDE.md:**
# Build and run
pnpm run build          # Compile TypeScript to dist/ with executable permissions
pnpm run build:credits  # Regenerate contributors.json from git history (CI/release only)
pnpm run start          # Run the compiled application
pnpm run dev            # Watch mode compilation (tsc --watch)

# Testing (run before committing)
pnpm run test:all       # Full suite: format, lint, types, AVA tests, knip

## Project Overview
Nanocoder is a React-based CLI coding agent built with Ink.js that provides local-first AI assistance with multiple provider support (Ollama, OpenRouter, any OpenAI-compatible API).

**Entry point**: `source/cli.tsx` → Ink render of `App` from `source/app.tsx`

### State Management Pattern
All state lives in `useAppState.tsx`. Other hooks (`useChatHandler`, `useToolHandler`, `useModeHandlers`) receive state and setters from it. `App.tsx` orchestrates these hooks together. Global `message-queue.ts` allows deep components to add chat messages.

### LLM Client Architecture
`client-factory.ts` creates clients via `createLLMClient(provider?)`. Uses Vercel AI SDK with `createOpenAICompatible` for any OpenAI-compatible API. Supports streaming responses and tool calling.

## Code Style
- **TypeScript strict mode** with `@/*` path alias mapping to `source/*`
- **Biome** for formatting (tabs, single quotes, semicolons, trailing commas)
- **Key lint rules**: `useExhaustiveDependencies: error`, `noUnusedVariables: error`, `noUnusedImports: error`
- **React 19** with Ink.js for CLI rendering

## Testing
- **Framework**: AVA with tsx loader
- **Location**: `source/**/*.spec.ts` files alongside source
- **Serial execution**: Tests run one at a time
- **Run single test**: `pnpm run test:ava source/path/to/file.spec.ts`


## AI Coding Assistance Notes

**Important Considerations:**
- Check package.json for available scripts before running commands
- Be aware of Node.js version requirements
- Consider impact on bundle size when adding dependencies
- Follow React hooks best practices
- Consider component reusability when creating new components
- Project has 809 files across 93 directories
- Large codebase: Focus on specific areas when making changes
- Check build configuration files before making structural changes

## Repository

**Source:** https://github.com/Nano-Collective/nanocoder.git

## Cursor Cloud specific instructions

This is a single product: a local-first CLI coding agent (Ink/React TUI) that ships as `dist/cli.js`. There is no backend/database/web server to run. Dependencies are installed automatically by the startup update script (`pnpm install --frozen-lockfile`). Node 22 + pnpm are pinned via `packageManager`/Corepack. Standard build/test/run commands are in `CLAUDE.md`, `package.json` scripts, and `scripts/test.sh` — use those rather than re-deriving them.

Non-obvious caveats:

- **Tests need color/TTY env vars.** Run the AVA suite with `FORCE_COLOR=1` and `TERM=xterm-256color` (this is what CI's `test-coverage` job sets). Without them, the Ink rendering specs in `source/app/components/settings-tabs.spec.tsx` fail on ANSI/inverse-video assertions even though the code is fine.
- **Build before CLI/benchmark tests.** `pnpm run build:credits` then `pnpm run build` produce `dist/` and `source/commands/contributors.json`, required by the CLI integration and benchmark specs.
- **`file-watcher` specs are timing-flaky under load.** `source/events/sources/file-watcher.spec.ts` uses 2s `waitFor` deadlines against chokidar polling; they can time out when the full serial suite is running on a busy machine, but pass reliably in isolation (`pnpm test:ava source/events/sources/file-watcher.spec.ts`).
- **`vscode/chat-panel-*` specs need the sibling media scripts.** The harness (`source/vscode/chat-panel-harness.ts`) evals `plugins/vscode/media/chat-panel.js` in a VM sandbox and must first eval `plugins/vscode/media/mention-utils.js` (which `chat-panel.html` loads before it) so `globalThis.NanocoderMentionUtils` is defined. If you add another `<script>` that `chat-panel.js` depends on, load it in the harness sandbox too.
- **`pnpm test:all` extras:** it also runs `knip`, `pnpm audit` (needs network), and `semgrep` (`scripts/test.sh` skips semgrep gracefully if the `semgrep` binary is not installed).
- **Running the app without an LLM key.** It talks to any OpenAI-compatible endpoint, so it can be exercised end-to-end against a local mock. Point a provider at the mock in `agents.config.json` (`nanocoder.providers[].baseUrl`, e.g. `http://127.0.0.1:8080/v1`) or via the `NANOCODER_PROVIDERS` env var; local `localhost`/`127.0.0.1` base URLs need no API key. Then drive it non-interactively: `node dist/cli.js --plain [--json] --provider <name> --model <model> --trust-directory run "<prompt>"`. Non-interactive `run` requires `--trust-directory` (or `NANOCODER_TRUST_DIRECTORY=1` in plain mode) to bypass the first-run directory-trust prompt. The mock must speak OpenAI **streaming** chat completions (SSE `data:` chunks ending with `data: [DONE]`), since the client always uses `streamText`.
- The `setlocale: LC_ALL: cannot change locale (en-US.UTF-8)` warning is harmless (the AVA config requests a locale name glibc doesn't recognize); it does not affect results.

---

*This AGENTS.md file was generated by Nanocoder. Update it as your project evolves.*