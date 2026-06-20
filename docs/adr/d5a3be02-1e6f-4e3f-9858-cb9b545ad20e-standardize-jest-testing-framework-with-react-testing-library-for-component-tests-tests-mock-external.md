# Standardize Jest Testing Framework with React Testing Library for Component Tests: Tests Mock External

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all test file creation and maintenance within the CI/CD pipeline.

## Context

- The codebase contains multiple test files using Jest as the primary testing framework with .spec.tsx extensions, indicating a TypeScript React testing pattern
- Tests are located in __tests__ directories following Jest conventions, suggesting an established testing structure integrated into the CI/CD pipeline
- The pattern appears across core packages including data utilities and store slices, indicating organization-wide testing standards
- React Testing Library patterns are evident in the test files, suggesting a user-centric testing approach over implementation details
- The testing infrastructure supports both unit and integration testing for React components and utility functions

## Problem Statement

Without a standardized testing framework and structure, teams may adopt inconsistent testing approaches leading to fragmented test coverage, difficult maintenance, unreliable CI/CD pipelines, and reduced confidence in code quality across the monorepo.

## Decision

1. SHOULD: Tests SHOULD mock external dependencies and API calls to ensure test isolation and reliability

## Policy Block

- SHOULD Tests SHOULD mock external dependencies and API calls to ensure test isolation and reliability

In scope:
- All TypeScript and TSX files in packages/core and related packages
- Unit tests for utility functions and data manipulation logic
- Integration tests for React components and store slices
- Tests executed as part of the CI/CD pipeline
- Pre-commit and pre-push test hooks

Out of scope:
- End-to-end tests using Cypress, Playwright, or similar tools
- Performance and load testing frameworks
- Manual testing procedures and QA processes
- Third-party library tests
- Build-time type checking (handled by TypeScript compiler)

Exceptions:
- EXC-001: Legacy test files using alternative frameworks may remain until scheduled refactoring
- EXC-002: Specialized testing scenarios requiring framework-specific features not available in Jest

## Rationale

- Jest provides a comprehensive, zero-configuration testing framework with built-in mocking, coverage reporting, and snapshot testing capabilities that align with TypeScript and React ecosystems
- React Testing Library encourages testing user behavior rather than implementation details, leading to more maintainable and meaningful tests
- The __tests__ directory convention is widely recognized in the JavaScript ecosystem and provides clear separation between source and test code
- Standardizing on a single testing framework reduces cognitive overhead, simplifies CI/CD configuration, and enables shared testing utilities across the monorepo
- The pattern detected across 3 files with 92.33% confidence indicates this is an established practice worth formalizing

## Consequences

Positive:
- Consistent testing patterns across the codebase improve developer productivity and reduce onboarding time
- Automated test execution in CI/CD pipelines provides rapid feedback on code quality and prevents regressions
- React Testing Library's user-centric approach leads to more resilient tests that survive refactoring
- Jest's built-in coverage reporting enables data-driven decisions about test completeness
- Standardized mocking patterns improve test isolation and reliability

Negative:
- Teams must invest time learning Jest and React Testing Library if unfamiliar with these tools
- Existing tests using alternative frameworks require migration effort
- Jest's default configuration may need customization for specific use cases, adding complexity
- Snapshot tests can become brittle if overused, requiring careful maintenance
- Mocking complex dependencies can be time-consuming and may obscure integration issues

## Alternatives

- Use Vitest as the primary testing framework instead of Jest (rejected)
  Rejected because: While Vitest offers faster execution and better ESM support, the existing codebase has established Jest patterns and migration would require significant effort without proportional benefit
  When valid: Consider for new greenfield projects or when ESM compatibility becomes critical
- Use Enzyme for React component testing instead of React Testing Library (rejected)
  Rejected because: Enzyme focuses on implementation details and shallow rendering, which leads to brittle tests. React Testing Library's user-centric approach is more aligned with modern React testing best practices
  When valid: Not recommended; Enzyme is no longer actively maintained for React 18+
- Allow teams to choose their preferred testing framework per package (rejected)
  Rejected because: Framework fragmentation increases maintenance burden, complicates CI/CD configuration, and prevents sharing of testing utilities and patterns across packages
  When valid: Only for isolated packages with unique requirements that cannot be met by Jest

## Risks

- Test execution time may increase as test suite grows, slowing down CI/CD pipeline
  Mitigation: Implement parallel test execution, use Jest's --maxWorkers flag, and consider test sharding for large suites. Monitor test execution time and optimize slow tests.
  Owner: Engineering team / DevOps
- Developers may write tests that pass but don't provide meaningful coverage or catch real bugs
  Mitigation: Establish code review guidelines for test quality, require coverage thresholds, and provide training on effective testing patterns. Use mutation testing to validate test effectiveness.
  Owner: Engineering team / Tech leads
- Mocking strategies may become inconsistent across the codebase, leading to maintenance challenges
  Mitigation: Create shared mocking utilities and patterns, document common mocking scenarios, and establish conventions for mock data factories. Review mocking approaches during code review.
  Owner: Engineering team

## Implementation Notes

- Create a shared testing utilities package with common mocks, test helpers, and custom matchers to promote consistency
- Configure Jest with appropriate presets for TypeScript and React in the root jest.config.js to ensure consistent behavior across packages
- Establish coverage thresholds (e.g., 80% for statements, branches, functions, and lines) and enforce them in CI/CD
- Provide team training on React Testing Library best practices, focusing on querying by accessibility roles and user interactions
- Document common testing patterns and anti-patterns in the team wiki or testing guide
- Set up pre-commit hooks to run tests on changed files to catch issues early

## Continuation Context


Verify commands:
- grep -r "describe\|it\|test" packages/*/lib/**/__tests__/*.spec.tsx | head -5
- find packages -type f -name '*.spec.tsx' -o -name '*.spec.ts' | head -10
- grep -r "@testing-library/react" packages/*/package.json
- npm test -- --listTests | grep __tests__

Accept when:
- All test files use .spec.tsx or .spec.ts extensions and are located in __tests__ directories
- Jest configuration is present in package.json or jest.config.js files
- React Testing Library is listed as a dependency in packages containing React component tests
- Tests execute successfully in CI/CD pipeline without manual intervention

## Enforcement

- Verified by: CI/CD pipeline test execution on every pull request
- Verified by: Code review checklist requiring test coverage for new features
- Verified by: Automated coverage reporting with threshold enforcement
- Verified by: Pre-commit hooks running tests on changed files
- Verified by: Periodic audit of test file locations and naming conventions
- Violation handling: Pull requests without tests or with failing tests are blocked from merging
- Violation handling: Coverage drops below threshold trigger CI/CD failure and require remediation
- Violation handling: Non-compliant test file locations or naming flagged during code review
- Violation handling: Quarterly review of test quality metrics with team retrospectives
- Violation handling: Repeated violations escalated to tech lead for coaching and support
- Exception process: Developer submits exception request to tech lead with justification
- Exception process: Tech lead reviews request and consults with architecture team if needed
- Exception process: Approved exceptions documented in test file header or ADR addendum
- Exception process: Exceptions reviewed quarterly to determine if they can be resolved
- Exception process: All exceptions tracked in central registry for visibility