# GenEd Gemini CLI Fork

**A customized fork of Google's Gemini CLI optimized for AI-powered grading.**

This repository is a **downstream fork** of [Google's official Gemini CLI](https://github.com/google-gemini/gemini-cli). It maintains compatibility with upstream while adding GenEd-specific customizations for educational grading workflows.

## Why a Fork?

GenEd uses a fork rather than consuming Gemini CLI as a dependency because:

### 1. A2A Server for Browser Integration

The key component for GenEd is the **A2A (Agent-to-Agent) server** in `packages/a2a-server`. This exposes Gemini CLI capabilities over HTTP, allowing the React frontend to invoke AI operations:

```
Browser (React) ──HTTP──▶ A2A Server ──▶ Gemini API
```

Without the A2A server, students would need to install and run the CLI locally.

### 2. Custom Model Defaults

GenEd defaults to `gemini-2.5-flash-lite` instead of upstream's `gemini-2.5-pro`:
- More cost-effective for high-volume grading
- Faster response times for immediate feedback
- Sufficient capability for code review and grading tasks

### 3. Educational Tool Customizations

The fork can include GenEd-specific:
- Custom tools for rubric-based grading
- Specialized prompts for CS education
- Integration with DynamoDB for persisting feedback

### 4. Version Stability

By forking, GenEd controls exactly which upstream version runs in production. This ensures grading behavior remains consistent even as Google releases updates.

## Repository Structure

```
GenEd-Gemini-Cli/
├── packages/
│   ├── cli/           # @google/gemini-cli - Terminal interface
│   ├── core/          # @google/gemini-cli-core - Shared library
│   ├── a2a-server/    # @google/gemini-cli-a2a-server - HTTP API (key for GenEd)
│   ├── test-utils/    # Shared testing utilities
│   └── vscode-ide-companion/  # VSCode extension
├── docs/              # Documentation
├── evals/             # Evaluation scripts
└── README.md          # Upstream README (preserved)
```

### Package Roles

| Package | npm Name | Purpose in GenEd |
|---------|----------|------------------|
| **a2a-server** | `@google/gemini-cli-a2a-server` | Primary interface for GenEd frontend |
| **core** | `@google/gemini-cli-core` | Model routing, Gemini API integration |
| **cli** | `@google/gemini-cli` | Direct terminal access (development) |

## Getting Started

### Prerequisites

- Node.js 20 or higher
- npm

### Installation

```bash
# Clean install
npm ci

# Build all packages
npm run build
```

### Running the A2A Server

The A2A server is the primary way GenEd uses this fork:

```bash
# Start the A2A server on port 41242
npm run start:a2a-server
```

Once running, the server exposes:
- `/.well-known/agent-card.json` - Agent discovery/health check
- POST endpoints for Gemini operations

### Running the CLI Directly

For development and testing:

```bash
# Interactive mode
npm run start

# Non-interactive query
npm run start -- -p "Explain this code"
```

### Full Validation

Run the complete preflight check:

```bash
npm run preflight
# Runs: clean → install → format → build → lint → typecheck → test
```

## Working with Upstream

### Remote Configuration

This fork maintains two Git remotes:

```bash
git remote -v
# origin    https://github.com/KeySmash-LLC/GenEd-gemini-cli.git (fork)
# upstream  https://github.com/google-gemini/gemini-cli.git (Google)
```

### Syncing Upstream Changes

To pull in improvements from Google's repository:

```bash
# Fetch upstream changes
git fetch upstream

# Ensure you're on main
git checkout main

# Merge upstream (may require conflict resolution)
git merge upstream/main

# Push to GenEd fork
git push origin main
```

### Resolving Merge Conflicts

When syncing upstream, conflicts typically occur in:
- **Model defaults**: GenEd uses different model configurations
- **Environment variables**: GenEd-specific environment handling
- **Package versions**: Keep aligned with upstream unless necessary

Resolve conflicts by preserving GenEd customizations while accepting upstream improvements.

### When to Sync

Consider syncing when:
- Google releases new Gemini models you want to support
- Important bug fixes are merged upstream
- New features would benefit grading workflows
- Security patches are released

## GenEd Customizations

### Model Configuration

GenEd Infrastructure sets the model via environment variable:

```yaml
# docker-compose.yml in GenEd-Infrastructure
environment:
  GEMINI_MODEL: gemini-2.5-flash-lite  # Cost-effective for grading
```

### A2A Server Environment

| Variable | Default | Description |
|----------|---------|-------------|
| `CODER_AGENT_PORT` | `41242` | A2A server port |
| `GEMINI_API_KEY` | Required | Google AI Studio API key |
| `GEMINI_MODEL` | `gemini-2.5-pro` | Model to use |
| `DEBUG` | `false` | Enable debug logging |

### Grading-Specific Features (Future)

Planned customizations:
- `/grade` command with rubric support
- Structured JSON output for grade storage
- Integration with assignment workspace volumes

## Testing

### Unit Tests

```bash
# Run all tests
npm test

# Run tests in a specific workspace
npm test -w @google/gemini-cli-core

# Run a specific test file
npm test -w @google/gemini-cli-core -- src/routing/modelRouterService.test.ts
```

### Integration Tests

```bash
# With Docker sandbox
npm run test:integration:sandbox:local

# Without sandbox
npm run test:integration:no-sandbox:local
```

### CI Pipeline

```bash
npm run test:ci
```

## Code Style Guidelines

From upstream `GEMINI.md`:

- **Prefer interfaces over classes**: Use plain objects with TypeScript interfaces
- **No `any` types**: Use `unknown` with type narrowing
- **Array methods over loops**: Use `.map()`, `.filter()`, `.reduce()`
- **Logging**: Use `debugLogger` for debug, `coreEvents.emitFeedback` for user-facing
- **ES modules**: Use `import`/`export` for encapsulation
- **Testing**: Vitest with mocks at file top before imports

## Relationship to GenEd Infrastructure

When running in Docker via GenEd-Infrastructure:

```
┌────────────────────────────────────────────────────────────┐
│                   Docker Compose Network                    │
├─────────────────┬────────────────────┬────────────────────┤
│  GenEd Frontend │   Gemini A2A       │  OpenVSCode        │
│  (React)        │   (This Fork)      │  Server            │
│  :5173          │   :41242           │  :3000             │
└────────┬────────┴─────────┬──────────┴─────────┬──────────┘
         │                  │                    │
         │      ┌───────────▼───────────┐       │
         │      │   Shared Workspace    │◀──────┘
         │      │   Volume              │
         │      └───────────────────────┘
         │
         └──────────────▶ Backend API (:8080)
```

The A2A server and OpenVSCode Server both mount the same `assignment-workspace` volume, allowing the AI agent to read/write files that students can edit in the browser.

## Troubleshooting

### A2A Server Won't Start

```bash
# Check if port is in use
lsof -i :41242

# Verify GEMINI_API_KEY is set
echo $GEMINI_API_KEY

# Check build output
npm run build
```

### API Key Issues

Get an API key from [Google AI Studio](https://aistudio.google.com/apikey):

```bash
export GEMINI_API_KEY="your-key-here"
npm run start:a2a-server
```

### Upstream Sync Fails

If merge conflicts are too complex:

```bash
# Create a fresh branch from upstream
git checkout -b upstream-sync upstream/main

# Cherry-pick GenEd customizations
git cherry-pick <commit-hashes>

# Or manually apply changes
```

## Contributing

### GenEd-Specific Changes

For changes specific to GenEd grading:
1. Create a branch from `main`
2. Make changes
3. Submit PR to `KeySmash-LLC/GenEd-gemini-cli`

### Upstream-Worthy Changes

For improvements that could benefit all Gemini CLI users:
1. Consider contributing directly to [google-gemini/gemini-cli](https://github.com/google-gemini/gemini-cli)
2. Follow Google's [Contributing Guide](https://github.com/google-gemini/gemini-cli/blob/main/CONTRIBUTING.md)

## Resources

- [Upstream Gemini CLI Repository](https://github.com/google-gemini/gemini-cli)
- [Gemini CLI Documentation](https://geminicli.com/docs/)
- [Google AI Studio](https://aistudio.google.com/) (for API keys)
- [Gemini API Documentation](https://ai.google.dev/docs)

## License

This fork inherits the [Apache License 2.0](./LICENSE) from the upstream Gemini CLI project.
