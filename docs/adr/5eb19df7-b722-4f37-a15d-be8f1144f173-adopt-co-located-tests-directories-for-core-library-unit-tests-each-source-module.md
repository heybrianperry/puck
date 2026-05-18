# Adopt Co-located __tests__ Directories for Core Library Unit Tests: Each Source Module

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The packages/core library contains critical business logic for component management, data resolution, and state transformation that requires comprehensive unit test coverage
- Test files are consistently organized in __tests__ directories co-located with the source code they test, following a widely-adopted Node.js and Jest convention
- The pattern appears across 10 distinct test files covering components (Puck), library utilities (move-component, migrate, transform-props, resolve-component-data, resolve-all-data), data operations (resolve-and-replace-data, resolve-data-by-id, resolve-data-by-selector), and reducer helpers
- TypeScript React (.tsx) test files use the .spec.tsx naming convention, indicating a preference for explicit test file identification over alternative conventions like .test.tsx
- The co-location strategy enables developers to quickly locate tests related to specific modules, reducing cognitive overhead and improving maintainability

## Problem Statement

Without a standardized approach to organizing unit tests in the core library, test discoverability suffers, maintenance becomes fragmented, and developers lack clear guidance on where to place new tests. The absence of consistent naming and location conventions can lead to orphaned tests, duplicate test suites, and difficulty in understanding test coverage boundaries.

## Decision

1. SHOULD: Each source module SHOULD have a corresponding test file with matching base name (e.g., resolve-data.ts → resolve-data.spec.tsx)

## Policy Block

- SHOULD Each source module SHOULD have a corresponding test file with matching base name (e.g., resolve-data.ts → resolve-data.spec.tsx)

In scope:
- All unit tests within the packages/core library
- Component tests (e.g., Puck component tests)
- Library utility tests (e.g., move-component, migrate, transform-props)
- Data operation tests (e.g., resolve-and-replace-data, resolve-data-by-id)
- Reducer and state management tests
- Test helper utilities and fixtures

Out of scope:
- End-to-end tests that test the entire application
- Integration tests that require external services or databases
- Performance and load tests
- Visual regression tests
- Tests in other packages outside packages/core
- Documentation examples that are not formal tests

Exceptions:
- EX-001: Legacy test files exist in a different structure and refactoring would introduce significant risk
- EX-002: Cross-cutting integration tests require access to multiple package internals

## Rationale

- The pattern demonstrates 92.25% confidence across 10 files, indicating strong consistency and deliberate architectural choice rather than ad-hoc organization
- Co-located __tests__ directories follow Jest's default test discovery patterns and align with Node.js ecosystem conventions, reducing configuration overhead and improving tool compatibility
- The .spec.tsx naming convention provides clear semantic distinction between test files and source files, preventing accidental imports of test code in production bundles
- Co-location reduces the cognitive distance between implementation and tests, making test-driven development more natural and encouraging developers to maintain tests alongside code changes

## Consequences

Positive:
- Improved test discoverability: developers can immediately locate tests by looking in the __tests__ directory adjacent to source code
- Reduced maintenance burden: when refactoring or moving modules, tests move with them, preventing orphaned test files
- Better IDE support: modern editors can quickly navigate between source and test files using standard conventions
- Simplified test configuration: Jest and other test runners automatically discover tests in __tests__ directories without complex glob patterns
- Enhanced code review: reviewers can easily verify that code changes include corresponding test updates

Negative:
- Directory structure becomes deeper with additional __tests__ folders throughout the codebase
- Potential for confusion if developers are unfamiliar with co-located test conventions and expect a separate test directory
- Test helpers and shared fixtures may be duplicated across multiple __tests__ directories if not properly organized
- Migration effort required for any existing tests that don't follow this pattern

## Alternatives

- Centralized test directory mirroring source structure (e.g., tests/packages/core/lib/...) (rejected)
  Rejected because: Creates significant cognitive distance between source and tests, making maintenance harder and increasing likelihood of orphaned tests during refactoring. Requires maintaining parallel directory structures.
  When valid: May be appropriate for integration tests that span multiple modules or packages
- Use .test.tsx extension instead of .spec.tsx (rejected)
  Rejected because: The existing pattern shows consistent use of .spec.tsx across all 10 detected files. Changing would require migration and both conventions are equally valid in the ecosystem.
  When valid: Could be adopted for new projects, but consistency within this codebase is more valuable than alignment with .test.tsx convention
- Place tests directly alongside source files (e.g., resolve-data.ts and resolve-data.spec.tsx in same directory) (rejected)
  Rejected because: Clutters source directories with test files, makes it harder to exclude tests from production builds, and mixes concerns in file listings
  When valid: Acceptable for very small modules or when using build tools that can easily filter test files

## Risks

- Inconsistent adoption: developers may place tests in wrong locations if not properly onboarded
  Mitigation: Add linting rules to enforce test file locations, document pattern in CONTRIBUTING.md, and include test location checks in CI pipeline
  Owner: Engineering team
- Test helper duplication: shared test utilities may be duplicated across multiple __tests__ directories
  Mitigation: Establish clear convention for __helpers__ subdirectories and create shared test utilities at appropriate levels (e.g., packages/core/__tests__/__helpers__)
  Owner: Engineering team
- Build tool misconfiguration: production builds might accidentally include test files if glob patterns are incorrect
  Mitigation: Verify build configurations explicitly exclude **/__tests__/** and **.spec.tsx patterns, add verification step to CI
  Owner: DevOps team

## Implementation Notes

- Configure Jest or your test runner to automatically discover tests in __tests__ directories: testMatch: ['**/__tests__/**/*.spec.tsx']
- Update TypeScript configuration to exclude test files from production builds: exclude: ['**/__tests__/**', '**/*.spec.tsx']
- Create a test file template or snippet that developers can use to quickly scaffold new tests in the correct location
- For shared test helpers, create a __helpers__ directory within __tests__ and export utilities via an index file
- When moving or renaming source files, ensure corresponding test files are moved to maintain co-location
- Add ESLint rules to prevent importing from __tests__ directories in production code

## Continuation Context


Verify commands:
- find packages/core -name '*.spec.tsx' -not -path '*/__tests__/*' | wc -l | grep -q '^0$'
- grep -r "testMatch" jest.config.* | grep -q "__tests__"
- find packages/core/__tests__ -name '*.spec.tsx' | wc -l

Accept when:
- All .spec.tsx test files in packages/core are located within __tests__ directories
- Jest or test runner configuration includes __tests__ in test discovery patterns
- Build configuration explicitly excludes __tests__ directories and .spec.tsx files from production bundles
- At least 10 test files follow the co-located __tests__ pattern (current baseline)

## Enforcement

- Verified by: Automated CI checks using find commands to verify test file locations
- Verified by: ESLint rules preventing imports from __tests__ directories in production code
- Verified by: Code review checklist requiring tests to be co-located with source code
- Verified by: Pre-commit hooks validating test file naming and location conventions
- Violation handling: CI pipeline fails if test files are found outside __tests__ directories
- Violation handling: Pull requests are blocked until tests are moved to correct locations
- Violation handling: Automated comments on PRs guide developers to correct test placement
- Violation handling: Quarterly audits identify and remediate any non-compliant test files
- Exception process: Developer documents rationale for exception in test file header comment
- Exception process: Tech lead reviews and approves exception with documented justification
- Exception process: Exception is recorded in TESTING.md with review date and migration plan if applicable
- Exception process: Exceptions are reviewed quarterly to determine if they can be resolved