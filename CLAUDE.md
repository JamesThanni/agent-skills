# CLAUDE.md

This file provides context and guidance for AI assistants working in this codebase.

## General Conventions (TypeScript and Python)

### Development Philosophy

- Simplicity, readability, maintainability, testability, reusability
- Type safety: let the compiler/type-checker catch errors; model invalid states as unrepresentative
- Less code = less debt; minimize footprint
- In ALL interactions and messages, be extremely concise and sacrifice grammar for the sake of concision
- Find all information on the system architecture in the docs/ folder and
- If system architecture documentation does not exist, suggest markdown notes to be documented in a docs/architecture.md file

### Coding Best Practices

- Early returns over nested conditions
- Descriptive names (prefix handlers with "handle")
- Constants over functions; DRY; functional/immutable style when not verbose
- Discriminated unions / tagged variants over optional-field grab-bags
- Minimal, targeted changes; define composing functions before their components
- Mark known issues with `TODO:`
- Build iteratively; verify before adding complexity; test with realistic inputs
- Build test environments for hard-to-validate components
- Keep core logic clean; push implementation details to the edges
- Balance file organization with simplicity for project scale

### CI / Error Resolution

- Fix order: formatting → type errors → linting
- For type errors: get full context, check optional/nullable types, narrow before use, verify signatures/generics
- Let the formatter own wrapping/line length — don't hand-fight it
- Check git status before commits; keep changes minimal; follow existing patterns; document public APIs; test thoroughly

## TypeScript Development Guidelines

### Package Management

- ONLY npm — never yarn/pnpm; `npm install`, `npm install -D`, `npm run <script>`, `npx tool`, `npm update`
- Commit `package-lock.json`
- FORBIDDEN: `yarn add`, `pnpm add`, global installs (`-g`), pinning `@latest` in `package.json`

### Code Quality

- `strict: true` in `tsconfig.json` — non-negotiable
- Type annotations on public APIs; rely on inference internally; NEVER `any` (use `unknown` + narrowing)
- Explicit return types on exported functions; TSDoc on public APIs
- Functions focused and small; follow existing patterns; 100 char line length
- All commonly reused colours should be stored in (mobile/web)/global/constants/theme.ts, extract and refactor any you come across to follow this pattern
- All commonly reused spacing should be stored in (mobile/web)/global/constants/spacing.ts, extract and refactor any you come across to follow this pattern
- All commonly reused font styles should be stored in (mobile/web)/global/constants/spacing.ts, extract and refactor any you come across to follow this pattern

### Testing

- `npx jest`; native async/await; co-locate as `*.test.ts`
- New features and bug fixes both require tests

### Code Style

- camelCase functions/variables; PascalCase classes/interfaces/types/enums/components; UPPER_SNAKE_CASE constants
- Template literals for string formatting; `const` by default, `let` only when reassigned, never `var`

### Tools

- Format: `npx prettier --write .` / check `--check .` (owns line length, quotes, semicolons, trailing commas)
- Lint: `npx eslint .` / fix `--fix` (flat config + typescript-eslint) — critical: no-explicit-any, no-unused-vars, no-floating-promises, import sorting
- Types: `npx tsc --noEmit` — explicit null/undefined checks, narrow before use, no `@ts-ignore` (use `@ts-expect-error` with reason)
- Pre-commit (husky + lint-staged): Prettier → ESLint --fix → tsc --noEmit

## Python Development Guidelines

### Package Management

- ONLY uv — never pip; `uv add package`, `uv run tool`, `uv sync`, `uv add --dev package --upgrade-package package`
- FORBIDDEN: `uv pip install`, `@latest` syntax

### Code Quality

- Type hints required everywhere; public APIs documented via doc-strings + a corresponding `.md` in `/docs`
- Functions focused and small; follow existing patterns; 88 char line length

### Testing

- `uv run pytest`; async tests use `anyio`, not `asyncio`
- New features and bug fixes both require tests (regression tests for fixes)

### Code Style

- snake_case functions/variables (PEP 8); PascalCase classes; UPPER_SNAKE_CASE constants
- f-strings for formatting; doc-strings mirrored into `/docs/*.md`

### Tools

- Format: `uv run ruff format .` / check `uv run ruff check .` / fix `--fix` — line length 88, import sorting (I001), unused imports
- Types: `uv run pyright` — explicit `None` checks for `Optional`, narrow strings before use
- Pre-commit (`.pre-commit-config.yaml`): Prettier (YAML/JSON) + Ruff (Python); update by bumping the config rev, commit config first
