# Development Guidelines

This document contains critical information about working with this codebase. Follow these guidelines precisely.

## Core Development Rules

1. Package Management
   - ONLY use npm, NEVER yarn or pnpm
   - Installation: `npm install package`
   - Dev dependencies: `npm install -D package`
   - Running scripts: `npm run <script>` (e.g. `npm run test`)
   - Running binaries: `npx tool`
   - Upgrading: `npm update package`
   - Commit `package-lock.json`
   - FORBIDDEN: `yarn add`, `pnpm add`, global installs (`-g`), pinning to `@latest` in `package.json`

2. Code Quality
   - `strict: true` in `tsconfig.json` — non-negotiable
   - Type annotations required for all public APIs; rely on inference internally
   - NEVER use `any` — use `unknown` plus narrowing, or a precise type
   - Explicit return types on all exported functions
   - Public APIs must have TSDoc comments
   - Functions must be focused and small
   - Follow existing patterns exactly
   - Line length: 100 chars maximum

3. Testing Requirements
   - Framework: `npx vitest`
   - Async testing: native `async`/`await`
   - Coverage: test edge cases and errors
   - New features require tests
   - Bug fixes require regression tests
   - Co-locate tests as `*.test.ts` beside source, or under `__tests__/`

4. Code Style
   - camelCase for functions/variables
   - PascalCase for classes, interfaces, type aliases, enums, and components
   - UPPER_SNAKE_CASE for constants
   - Document with TSDoc
   - Use template literals for string formatting
   - Prefer `const`; use `let` only when reassigned; never `var`

- For commits fixing bugs or adding features based on user reports add:

  ```bash
  git commit --trailer "Reported-by:<name>"
  ```

  Where `<name>` is the name of the user.

- For commits related to a Github issue, add
  ```bash
  git commit --trailer "Github-Issue:#<number>"
  ```
- NEVER ever mention a `co-authored-by` or similar aspects. In particular, never
  mention the tool used to create the commit message or PR.

## Development Philosophy

- **Simplicity**: Write simple, straightforward code
- **Readability**: Make code easy to understand
- **Performance**: Consider performance without sacrificing readability
- **Maintainability**: Write code that's easy to update
- **Testability**: Ensure code is testable
- **Reusability**: Create reusable components and functions
- **Type Safety**: Let the compiler catch errors; model invalid states as unrepresentable
- **Less Code = Less Debt**: Minimize code footprint

## Coding Best Practices

- **Early Returns**: Use to avoid nested conditions
- **Descriptive Names**: Use clear variable/function names (prefix handlers with "handle")
- **Constants Over Functions**: Use constants where possible
- **DRY Code**: Don't repeat yourself
- **Functional Style**: Prefer functional, immutable approaches when not verbose (`readonly`, `const`, `map`/`filter`/`reduce` over mutation)
- **Discriminated Unions**: Model variant data with tagged unions over optional-field grab-bags
- **Minimal Changes**: Only modify code related to the task at hand
- **Function Ordering**: Define composing functions before their components
- **TODO Comments**: Mark issues in existing code with "TODO:" prefix
- **Simplicity**: Prioritize simplicity and readability over clever solutions
- **Build Iteratively**: Start with minimal functionality and verify it works before adding complexity
- **Run Tests**: Test your code frequently with realistic inputs and validate outputs
- **Build Test Environments**: Create testing environments for components that are difficult to validate directly
- **Functional Code**: Use functional and stateless approaches where they improve clarity
- **Clean logic**: Keep core logic clean and push implementation details to the edges
- **File Organisation**: Balance file organization with simplicity — use an appropriate number of files for the project scale

## System Architecture

[fill in here]

## Core Components

- `config.ts`: Configuration management
- `index.ts`: Entry point
  [etc... fill in here]

## Pull Requests

- Create a detailed message of what changed. Focus on the high level description of
  the problem it tries to solve, and how it is solved. Don't go into the specifics of the
  code unless it adds clarity.

- Always add `ArthurClune` as reviewer.

- NEVER ever mention a `co-authored-by` or similar aspects. In particular, never
  mention the tool used to create the commit message or PR.

## TypeScript Tools

## Code Formatting

1. Prettier
   - Format: `npx prettier --write .`
   - Check: `npx prettier --check .`
   - Config: `.prettierrc`
   - Owns: line length, quotes, semicolons, trailing commas
   - Do NOT hand-format what Prettier owns

2. Linting
   - Tool: ESLint (flat config `eslint.config.js` + `typescript-eslint`)
   - Check: `npx eslint .`
   - Fix: `npx eslint . --fix`
   - Critical rules:
     - `@typescript-eslint/no-explicit-any`
     - `@typescript-eslint/no-unused-vars`
     - `@typescript-eslint/no-floating-promises`
     - import sorting / no unused imports

3. Type Checking
   - Tool: `npx tsc --noEmit`
   - Requirements:
     - Explicit null/undefined checks for optional values
     - Type narrowing before use (`typeof`, `in`, discriminated unions)
     - No `@ts-ignore` — use `@ts-expect-error` with a reason if unavoidable
     - Version warnings can be ignored if checks pass

4. Pre-commit
   - Tooling: husky + lint-staged
   - Runs: on git commit
   - Steps: Prettier (format) → ESLint (`--fix`) → `tsc --noEmit`
   - Tool updates:
     - Check npm versions
     - Update config / dependency
     - Commit config first

## Error Resolution

1. CI Failures
   - Fix order:
     1. Formatting (Prettier)
     2. Type errors (`tsc`)
     3. Linting (ESLint)
   - Type errors:
     - Get full line context
     - Check optional / nullable types
     - Add type narrowing or guards
     - Verify function signatures and generics

2. Common Issues
   - Line length:
     - Let Prettier wrap; don't fight it
   - Types:
     - Add null/undefined checks
     - Narrow before access
     - Match existing patterns
   - Floating promises:
     - `await` it, or `void` it explicitly

3. Best Practices
   - Check git status before commits
   - Run formatter before type checks
   - Keep changes minimal
   - Follow existing patterns
   - Document public APIs
   - Test thoroughly
