# CLAUDE.md — B.R.A.N.D.I.

> Guidelines for AI assistants working in this repository.

## Project Overview

**B.R.A.N.D.I.** is an early-stage project owned by **Fluid-Kiss-Consultations**. The repository is currently in its initial skeleton phase — no application code, dependencies, or build tooling have been added yet.

- **License:** GNU Affero General Public License v3 (AGPL-3.0)
- **Primary branch:** `main`

## Repository Structure

```
B.R.A.N.D.I./
├── CLAUDE.md        # This file — AI assistant guidelines
├── README.md        # Project description (placeholder)
├── LICENSE          # AGPL-3.0 license
└── .gitignore       # Git ignore rules (to be populated)
```

## Development Workflow

### Branching

- The default branch is `main`.
- Feature branches should follow the pattern: `<author>/<short-description>`.

### Commits

- Write clear, descriptive commit messages.
- Keep commits focused — one logical change per commit.

### Code Style

No linters or formatters are configured yet. When they are added, update this section with:
- The linting/formatting tools in use
- How to run them (e.g., `npm run lint`, `make fmt`)
- Any pre-commit hooks

### Testing

No test framework is configured yet. When one is added, update this section with:
- The test framework and runner
- How to run tests (e.g., `npm test`, `pytest`)
- Test file naming conventions and locations

### Building & Running

No build system is configured yet. When one is added, update this section with:
- How to install dependencies
- How to build the project
- How to run the project locally

## Conventions for AI Assistants

1. **Read before writing.** Always read existing files before modifying them.
2. **Respect the license.** All contributions fall under AGPL-3.0. Do not introduce incompatibly-licensed dependencies without flagging it.
3. **Keep this file current.** When you add tooling, dependencies, or architectural patterns, update the relevant sections of this CLAUDE.md.
4. **Do not create unnecessary files.** Prefer editing existing files over creating new ones.
5. **Ask when uncertain.** If a task is ambiguous or could have significant architectural impact, ask the user before proceeding.
6. **No secrets in code.** Never commit API keys, tokens, passwords, or other credentials. Use environment variables or secret management tools.

## Environment Setup

No environment configuration exists yet. When added, document:
- Required environment variables
- `.env` file setup
- External service dependencies

## CI/CD

No CI/CD pipelines are configured yet. When added, document:
- Pipeline triggers and stages
- Required secrets/variables
- How to check build status
