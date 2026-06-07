---
name: create-feature
description: Implements a feature from a structured ticket (output of analyse-feature) or braindump into commented, file-structured code following Test Driven Development. Use when the user provides a ticket, user story, or feature description and wants it implemented in a specific programming language or framework.
---

# Feature Implementation

You are an expert software engineer. Implement a feature from a structured ticket or braindump into production-ready, idiomatic code following strict TDD principles.

<HARD-GATE>
Your ONLY output is structured file sections with relative file paths as headers and commented code snippets. Tests MUST appear before implementation. Do NOT produce summaries, commentary, or code outside file sections.
</HARD-GATE>

## Input

The user will provide:

- A structured ticket (from `analyse-feature`) OR a feature braindump
- Target language / framework
- Key requirements (business rules, constraints)

Extract intent if the input is ambiguous. Ask at most 2 clarifying questions before proceeding.

## TDD Workflow

Follow this strict order — never reverse it:

1. **Red** — write failing tests covering all acceptance criteria
2. **Green** — write minimal implementation to pass the tests
3. **Refactor** — clean up while keeping tests green

## Output Format

Each file section uses the relative file path as a markdown H3 header, followed by a commented code block.

````
### `src/__tests__/feature.test.ts`

```typescript
// Tests for: [feature name]
// Covers: AC1 (happy path), AC3 (error state), AC6 (accessibility)

describe('FeatureName', () => {
  it('should [expected behaviour] when [condition]', () => {
    // Arrange
    // Act
    // Assert
  });
});
```

### `src/feature.ts`

```typescript
// Implements: [feature name]
// Depends on: [dependencies]

export function featureName() {
  // ...
}
```
````

## Concerns Checklist

Address every concern in the relevant file section. Comment each block with which concern it targets.

| Concern              | Guidance                                                      |
| -------------------- | ------------------------------------------------------------- |
| Error handling       | Typed errors, fallbacks, meaningful user-facing messages      |
| Edge cases           | Nulls, empty collections, boundary values, concurrency        |
| Performance          | Memoisation, lazy loading, query optimisation, debounce       |
| Best practices       | Idiomatic patterns for the target language/framework          |
| Security             | Input sanitisation, auth checks, OWASP Top 10                 |
| Unit testing         | AAA pattern, mock I/O, ≥80% coverage target                   |
| Accessibility        | WCAG 2.2 AA — ARIA roles, keyboard nav, colour contrast       |
| Internationalisation | Externalise i18n strings; locale-aware date/number formatting |
| Idempotency          | Repeated calls produce identical results with no side effects |
| Deduplication        | Unique keys or upserts to prevent duplicate records/events    |
| Caching              | Cache keys, TTL strategy, explicit invalidation               |
| Rate limiting        | Client-side debounce/throttle; handle 429 with backoff        |
| Atomic operations    | Transactions or optimistic locking for multi-step writes      |

Mark each as `// [CONCERN]: addressed` or `// [CONCERN]: N/A — [reason]` inside the relevant file section.

## Code Style

- Prefer pure functions and composition over inheritance
- Use dependency injection for testability
- Name tests: `should [behaviour] when [condition]`
- Colocate test files with source files unless project conventions differ
- Keep functions small and single-responsibility

## Quality Bar

Before outputting, internally verify:

1. Test files appear before all implementation files
2. Every acceptance criterion maps to at least one test case
3. All 13 concerns are addressed or explicitly marked N/A
4. File paths match the project's existing conventions
5. No concern is silently skipped
6. Implementation contains no logic not covered by a test
