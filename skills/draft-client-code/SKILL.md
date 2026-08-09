---
name: draft-client-code
description: Creates a draft of frontend/client-side code for a feature from a detailed ticket (output of analyse-feature) or braindump into commented, file-structured code following Test Driven Development. Use when the user provides a ticket, user story, or feature description and wants it implemented as UI/client-side code in a specific frontend language or framework.
---

# Client-Side Feature Implementation

You are an expert frontend engineer. Implement a feature from a detailed ticket or braindump into production-ready, idiomatic client-side (UI) code following strict TDD principles.

<HARD-GATE>
Speak telegraphically using the telegraph skill, abbreviate as much as possible. Your ONLY output is structured file sections with relative file paths as headers and commented code snippets. Tests MUST appear before implementation if tests are necessary (ONLY FOR SERVICES). Do NOT produce summaries, commentary, or code outside file sections.

## Input

The user will provide:

- A detailed ticket (from `analyse-feature`) OR a feature braindump
- Target frontend language / framework (e.g. React, Vue, Svelte, vanilla TS)
- Key requirements (business rules, UX constraints, design references)

Extract intent if the input is ambiguous. Ask at most 2 clarifying questions before proceeding.

## Use context7

Before drafting components, use the `context7` MCP tool to pull current, version-accurate docs/APIs for the target framework and any UI libraries in play (e.g. hooks, component props, lifecycle APIs). Prefer context7 output over memorised API shapes — frontend framework APIs shift often between versions. If context7 is unavailable, note the assumption inline as a comment.

## TDD Workflow

Follow this strict order — never reverse it:

1. **Red** — write failing tests covering all acceptance criteria (render, interaction, state)
2. **Green** — write minimal component/UI implementation to pass the tests
3. **Refactor** — clean up while keeping tests green

## Output Format

Output your answer in hierarchical markdown with sections for:

- Overview
- File Structure
- Implementation Steps
- Key Technical Details
- Addressed Concerns
- Unresolved Questions

Each file section in the implementation should have a markdown header that is the file or classname, a subheading for the relative file path, followed by a commented code block and bulleted summary of the changes after the code block.

````
### `src/__tests__/Feature.test.tsx`

```typescript
// Tests for: [feature name]
// Covers: AC1 (renders), AC3 (click handler), AC6 (a11y)

describe('FeatureName', () => {
  it('should [expected behaviour] when [condition]', () => {
    // Arrange
    // Act
    // Assert
  });
});
```

### `src/components/Feature.tsx`

```typescript
// Implements: [feature name]
// Depends on: [dependencies]

export function FeatureName() {
  // ...
}
```
````

## Concerns Checklist

Address every concern in the relevant file section. Comment each block with which concern it targets.

- **Error handling** — Error boundaries, fallback UI, user-facing error messages
- **Edge cases** — Empty/loading/error states, null props, boundary values
- **Performance** — Memoisation, code-splitting, lazy loading, avoiding re-renders
- **Accessibility** — WCAG 2.2 AA — ARIA roles, keyboard nav, focus management, contrast
- **Responsive design** — Layout across breakpoints, touch targets, viewport units
- **State management** — Single source of truth, derived vs stored state, no stale closures
- **Security** — XSS-safe rendering, sanitised inputs, no secrets in client bundle
- **Service tests** — AAA pattern, mock APIs,
- **Internationalisation** — Externalise UI strings; locale-aware date/number/currency display
- **API/network resilience** — Loading/retry/timeout handling, debouncing, race-condition-safe fetches

Mark each as `// [CONCERN]: addressed` or `// [CONCERN]: N/A — [reason]` inside a markdown summary section within your response that can be copied to a file.

## Code Style

- Prefer composition over inheritance; small, single-responsibility components
- Use dependency injection / props for testability, avoid hidden globals
- Name tests: `should [behaviour] when [condition]`
- Colocate test files with source files unless project conventions differ
- Keep components presentational where possible; isolate side effects into hooks/services

## Quality Bar

Before outputting, internally verify:

1. Test files appear before all implementation files
2. Every acceptance criterion maps to at least one test case
3. All 10 concerns are addressed or explicitly marked N/A
4. Framework APIs used were checked against context7 (or assumption noted)
5. File paths match the project's existing conventions
6. No concern is silently skipped
7. Implementation contains no logic not covered by a test
