# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**Note:** This repository also has a GEMINI.md file with detailed coding guidelines. Both files are read by Claude Code.

## Project Overview

Fork of Google's Gemini CLI - an open-source AI agent that brings Gemini into your terminal. This is a Node.js/TypeScript workspace monorepo.

## Commands

```bash
# Full validation (run before submitting changes)
npm run preflight    # clean, install, format, build, lint, typecheck, test

# Development
npm run start        # Run in development mode
npm run debug        # Run with debugger attached
npm run build        # Build all packages
npm run bundle       # Create production bundle

# Testing
npm run test         # Run all workspace tests
npm run test:ci      # CI test mode

# Run specific test in a workspace (path MUST be relative to workspace root):
npm test -w @google/gemini-cli-core -- src/routing/modelRouterService.test.ts
npm test -w @google/gemini-cli -- src/ui/chat.test.tsx

# Integration tests
npm run test:integration:sandbox:none    # Without sandbox
npm run test:integration:sandbox:docker  # With Docker sandbox

# Linting & Formatting
npm run lint         # ESLint check
npm run lint:fix     # Fix lint errors
npm run format       # Prettier format
npm run typecheck    # TypeScript check
```

## Workspace Packages

```
packages/
├── cli/           # @google/gemini-cli - Main CLI application
├── core/          # @google/gemini-cli-core - Core library
├── a2a-server/    # @google/gemini-cli-a2a-server - A2A server
├── test-utils/    # Shared test utilities
└── vscode-ide-companion/  # VSCode extension
```

## Code Style

### TypeScript

- **Prefer plain objects with TypeScript interfaces over classes**
- Use ES module `import`/`export` for encapsulation (not class private/public)
- Avoid `any`; prefer `unknown` with type narrowing
- Use `checkExhaustive` helper in switch default clauses (see `packages/cli/src/utils/checks.ts`)
- Use array methods (`.map()`, `.filter()`, `.reduce()`) over imperative loops
- Use hyphens in flag names (`my-flag` not `my_flag`)

### Logging

- **Never use `console.log` directly**
- Debug/dev logs: `debugLogger` from `@google/gemini-cli-core`
- User-facing messages: `coreEvents.emitFeedback` from `@google/gemini-cli-core`

### React (Ink CLI UI)

- Functional components with hooks only
- Keep components pure - no side effects during render
- Side effects go in `useEffect` or event handlers
- Don't call `setState` inside `useEffect`
- Test with `ink-testing-library` using `render()` and `lastFrame()`

### Testing (Vitest)

- Test files co-located with source (`*.test.ts`, `*.test.tsx`)
- Use `vi.mock()` with `importOriginal` for selective mocking
- Place critical mocks (`os`, `fs`) at top of file before other imports
- Use `vi.hoisted()` for mocks needed in factory functions
- Use `vi.useFakeTimers()` for time-dependent tests
- Call `vi.resetAllMocks()` in `beforeEach`, `vi.restoreAllMocks()` in `afterEach`

## Documentation

When working in `/docs`:
- Do not invent facts, commands, or API names
- Base technical info on actual code in this repository
- Follow style guide in `CONTRIBUTING.md`
- Consider information architecture before making changes

## Git

- Main branch: `main`
- Refer to product as "Gemini CLI" (not "the Gemini CLI")
