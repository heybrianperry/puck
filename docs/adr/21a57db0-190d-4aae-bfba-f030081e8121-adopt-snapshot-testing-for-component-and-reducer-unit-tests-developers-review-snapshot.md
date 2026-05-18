# Adopt Snapshot Testing for Component and Reducer Unit Tests: Developers Review Snapshot

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains React components and state management reducers that require comprehensive testing to ensure behavioral consistency across changes
- Manual assertion writing for complex data structures and component outputs is time-consuming and error-prone, particularly for tree-walking and state transformation logic
- Snapshot testing provides an efficient mechanism to detect unintended changes in component rendering and data transformation outputs during CI/CD pipelines
- Three test files (walk-app-state.spec.tsx, remove.spec.ts, walk-tree.spec.tsx) demonstrate consistent adoption of snapshot testing patterns with 91.47% confidence
- The testing.snapshot facet indicates this is a deliberate architectural choice for regression detection in the build and delivery pipeline

## Problem Statement

Teams need a scalable, maintainable approach to verify that complex component rendering logic and state transformation functions produce consistent outputs across code changes, without manually writing exhaustive assertions for every data structure field or DOM element. Traditional assertion-based testing becomes unwieldy for deeply nested objects and component trees, leading to incomplete test coverage and missed regressions.

## Decision

1. SHOULD: Developers SHOULD review snapshot diffs carefully during code review to ensure changes are intentional

## Policy Block

- SHOULD Developers SHOULD review snapshot diffs carefully during code review to ensure changes are intentional

In scope:
- React component unit tests in packages/core/lib
- Reducer and action unit tests in packages/core/reducer
- Data transformation and tree-walking utility tests
- Any test file with .spec.ts or .spec.tsx extension in __tests__ directories

Out of scope:
- Integration tests that verify cross-component interactions
- End-to-end tests that validate full user workflows
- Performance benchmarking tests
- Tests for simple utility functions with primitive return values

Exceptions:
- EXC-001: Component output is highly dynamic or non-deterministic (e.g., timestamps, random IDs)
- EXC-002: Legacy test files written before this ADR adoption

## Rationale

- Pattern detected across 3 files with 91.47% confidence indicates established team practice and proven effectiveness
- Snapshot testing significantly reduces test maintenance burden for complex data structures, allowing developers to focus on business logic rather than assertion boilerplate
- Automated snapshot comparison in CI/CD pipelines provides immediate feedback on unintended behavioral changes, improving regression detection
- The testing.snapshot facet alignment with CI/CD category demonstrates this is a deliberate quality gate in the delivery pipeline

## Consequences

Positive:
- Reduced test authoring time: developers can generate comprehensive test coverage with minimal code
- Improved regression detection: any change to component output or data transformation is immediately visible in snapshot diffs
- Better code review quality: snapshot diffs provide clear visual evidence of behavioral changes for reviewers
- Consistent testing patterns across the codebase improve maintainability and developer onboarding

Negative:
- Snapshot files can become large and difficult to review if not properly scoped to specific test cases
- Risk of developers blindly accepting snapshot updates without understanding the underlying changes
- Snapshot tests may be less readable than explicit assertion-based tests for developers unfamiliar with the pattern
- Refactoring that changes output format (but not behavior) requires snapshot updates, potentially masking real issues

## Alternatives

- Use explicit assertion-based testing for all component and reducer outputs (rejected)
  Rejected because: Explicit assertions for complex nested objects and component trees are verbose, error-prone, and difficult to maintain. Evidence shows team has adopted snapshot testing for these scenarios.
  When valid: For simple functions with primitive return values or critical business logic requiring specific field validation
- Use visual regression testing tools (e.g., Percy, Chromatic) for component verification (rejected)
  Rejected because: Visual regression testing is complementary but addresses UI appearance rather than data structure and rendering logic. Adds infrastructure complexity and cost.
  When valid: For end-to-end tests validating visual design consistency across browsers
- Hybrid approach: snapshots for structure, assertions for critical values (accepted)
  When valid: This is the recommended approach per rule R-50-007, allowing teams to supplement snapshots with targeted assertions

## Risks

- Developers may approve snapshot updates without proper review, allowing bugs to pass through CI/CD
  Mitigation: Enforce mandatory code review for all snapshot changes; provide training on snapshot diff interpretation; use CI checks to flag large snapshot changes
  Owner: Engineering team leads
- Snapshot files may grow too large, impacting repository size and diff readability
  Mitigation: Scope snapshots to specific test cases; use inline snapshots for small outputs; periodically audit and refactor overly large snapshots
  Owner: Development team
- Non-deterministic outputs (timestamps, IDs) may cause snapshot test flakiness
  Mitigation: Use snapshot serializers to normalize dynamic values; mock time-dependent functions; document exceptions per EXC-001
  Owner: Test infrastructure team

## Implementation Notes

- Use Jest's toMatchSnapshot() or toMatchInlineSnapshot() methods for snapshot assertions in test files
- Organize snapshot files in __snapshots__ directories adjacent to test files, following Jest conventions
- Configure snapshot serializers for common data types (React elements, dates, etc.) to improve snapshot readability
- Include snapshot review guidelines in team documentation and code review checklists
- Set up CI pipeline to fail on uncommitted snapshot changes to prevent accidental omissions

## Continuation Context


Verify commands:
- grep -r 'toMatchSnapshot\|toMatchInlineSnapshot' packages/core/lib/data/__tests__/ packages/core/reducer/actions/__tests__/
- find packages/core -type d -name '__snapshots__' | wc -l
- npm test -- --coverage --testPathPattern='spec\.(ts|tsx)$'

Accept when:
- All .spec.ts and .spec.tsx files in packages/core/lib/data/__tests__/ and packages/core/reducer/actions/__tests__/ contain at least one snapshot assertion
- Snapshot directories exist alongside test files and contain .snap files for each test suite
- CI pipeline successfully runs snapshot tests and fails on snapshot mismatches

## Enforcement

- Verified by: Automated CI/CD pipeline runs snapshot tests on every pull request
- Verified by: Code review process includes mandatory review of snapshot diffs
- Verified by: Pre-commit hooks warn developers of uncommitted snapshot changes
- Violation handling: CI pipeline fails if snapshot tests do not pass
- Violation handling: Pull requests with unreviewed snapshot changes are blocked from merge
- Violation handling: Developers must update snapshots explicitly using npm test -- -u and commit changes
- Exception process: Request exception approval from tech lead via pull request comment
- Exception process: Document rationale in test file comments explaining why snapshot testing is unsuitable
- Exception process: Alternative testing approach must be implemented and reviewed before merge approval