# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OpenClaw is a multi-channel AI messaging gateway CLI that connects various messaging platforms (WhatsApp, Telegram, Discord, Slack, Signal, iMessage, LINE, etc.) to AI providers. It runs as a gateway server with native apps for macOS, iOS, and Android.

## Build, Test, and Development Commands

**Runtime baseline:** Node 22+ (Bun also supported for TypeScript execution)

```bash
# Install dependencies
pnpm install

# Type-check and build
pnpm build

# Lint and format
pnpm lint         # oxlint
pnpm format       # oxfmt (check)
pnpm format:fix   # oxfmt (write)

# Run tests
pnpm test                 # unit/integration tests (vitest)
pnpm test:coverage        # with coverage
pnpm test:e2e             # gateway e2e tests
pnpm test:live            # live tests (requires real API keys)

# Full gate (run before pushing)
pnpm lint && pnpm build && pnpm test

# Run CLI in dev mode
pnpm openclaw <command>   # or: pnpm dev

# Run gateway in dev mode
pnpm gateway:dev
```

**Single test file:** `pnpm test src/path/to/file.test.ts`

**Mobile apps:**
- iOS: `pnpm ios:open` (generates Xcode project), `pnpm ios:build`, `pnpm ios:run`
- Android: `pnpm android:run`, `pnpm android:test`
- macOS: `pnpm mac:package`, `pnpm mac:open`

## Project Structure

```
src/
├── cli/           # CLI wiring, command definitions, prompts
├── commands/      # Command implementations (agent, auth, etc.)
├── gateway/       # Gateway server (WebSocket/HTTP, protocol handling)
├── agents/        # Agent logic, tools, auth profiles
├── channels/      # Channel abstraction layer and plugin types
├── config/        # Configuration loading and schemas
├── infra/         # Low-level infrastructure (env, errors, ports, binaries)
├── providers/     # AI provider integrations
├── routing/       # Message routing logic
├── plugin-sdk/    # Plugin SDK exports for extensions
├── terminal/      # Terminal UI utilities (tables, colors, progress)
├── {telegram,discord,slack,signal,imessage,web,line}/  # Channel implementations
└── *.test.ts      # Colocated tests

apps/
├── macos/         # SwiftUI macOS menubar app
├── ios/           # SwiftUI iOS app
├── android/       # Kotlin Android app
└── shared/        # Shared Swift code (OpenClawKit)

extensions/        # Channel plugins (workspace packages)
docs/              # Mintlify documentation (docs.openclaw.ai)
scripts/           # Build, release, and utility scripts
skills/            # Bundled agent skills
```

## Architecture Overview

**Gateway:** The core is a WebSocket/HTTP gateway server (`src/gateway/`) that:
- Manages channel connections and message routing
- Handles AI agent sessions and tool execution
- Exposes OpenAI-compatible and OpenResponses APIs
- Supports multiple concurrent messaging channels

**Channels:** Each messaging platform has a channel implementation with adapters:
- Core channels: `src/{telegram,discord,slack,signal,imessage,web,line}/`
- Plugin channels: `extensions/{msteams,matrix,googlechat,...}/`
- Common interfaces: `src/channels/plugins/types.ts`

**Agents:** AI agents (`src/agents/`) handle:
- Tool execution (bash, file operations, etc.)
- Auth profile management for multiple AI providers
- Session persistence and history

**Plugin SDK:** Extensions use `openclaw/plugin-sdk` exports from `src/plugin-sdk/index.ts`

## Coding Conventions

- **Language:** TypeScript (ESM), strict typing, avoid `any`
- **Naming:** Product = "OpenClaw", CLI/package = "openclaw"
- **File size:** Aim for ~500-700 LOC max; split when it helps clarity
- **Tests:** Colocated `*.test.ts`; e2e tests use `*.e2e.test.ts`; live tests use `*.live.test.ts`
- **CLI progress:** Use `src/cli/progress.ts` (not hand-rolled spinners)
- **Terminal output:** Use `src/terminal/palette.ts` for colors, `src/terminal/table.ts` for tables
- **Dependency injection:** Follow existing patterns via `createDefaultDeps`

## Key Files to Know

- `src/index.ts` - Main entry point and public exports
- `src/cli/program.ts` - CLI command tree (Commander.js)
- `src/gateway/server.ts` - Gateway server entry
- `src/config/config.ts` - Configuration loading
- `src/plugin-sdk/index.ts` - Plugin SDK exports
- `src/channels/plugins/types.ts` - Channel adapter interfaces

## Commit and PR Guidelines

- Create commits with `scripts/committer "<msg>" <file...>` (keeps staging scoped)
- Use concise, action-oriented messages (e.g., `CLI: add verbose flag to send`)
- Add changelog entries for user-facing changes (PR # + contributor thanks)
- Full gate before committing: `pnpm lint && pnpm build && pnpm test`

## Documentation (Mintlify)

- Docs at `docs/` are hosted at docs.openclaw.ai
- Internal links: root-relative, no `.md` extension (e.g., `[Config](/configuration)`)
- Avoid em dashes and apostrophes in headings (breaks anchor links)
- Use placeholders for device names/paths in examples

## Important Constraints

- Never update the Carbon dependency
- Patched dependencies (`pnpm.patchedDependencies`) must use exact versions (no `^`/`~`)
- Patching dependencies requires explicit approval
- Tool schemas: avoid `Type.Union`, `anyOf`/`oneOf`/`allOf`; use `stringEnum`/`optionalStringEnum` instead
- SwiftUI (iOS/macOS): prefer `@Observable`/`@Bindable` over `ObservableObject`/`@StateObject`

## Version Locations

- CLI: `package.json`
- macOS: `apps/macos/Sources/OpenClaw/Resources/Info.plist`
- iOS: `apps/ios/Sources/Info.plist`
- Android: `apps/android/app/build.gradle.kts`
