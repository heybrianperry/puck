# Standardize Public API Contract Testing with Jest Mocking and E2E Validation: Unit Tests Validating

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase exposes public API contracts including UserConfig, UserData, dzZoneCompound, defaultData, defaultUi, setup, and getBox that require validation across unit and integration boundaries
- Unit tests in packages/core/reducer/actions/__helpers__/index.tsx use jest.mock to isolate dependencies like generate-id while validating state transformations and data structure contracts
- End-to-end tests in scripts/e2e/utils/ use puppeteer to validate browser-based API contracts through page.evaluate, page.goto, and browser automation workflows
- The testing strategy separates concerns: unit tests verify internal state consistency and data transformations, while E2E tests validate runtime behavior in browser contexts including iframe coordinate transformations

## Problem Statement

Public API contracts must be validated at multiple architectural boundaries to ensure correctness, but without a standardized testing strategy, teams may inconsistently test contract behavior, leading to integration failures, runtime errors in browser contexts, and broken assumptions about state transformations and data structures.

## Decision

1. MUST: Unit tests validating state transformations and data structures MUST use jest.mock to isolate external dependencies and ensure deterministic test execution

## Policy Block

- MUST Unit tests validating state transformations and data structures MUST use jest.mock to isolate external dependencies and ensure deterministic test execution

In scope:
- All exported public API contracts including configuration objects, data structures, and utility functions
- State management reducers and actions that transform public data structures
- Browser-based APIs that execute in page contexts or handle DOM interactions
- E2E utility functions that coordinate browser automation workflows

Out of scope:
- Internal implementation details not exposed through public APIs
- Private helper functions not exported from modules
- Development-only utilities not included in production builds
- Third-party library internals beyond integration points

## Rationale

- The evidence shows a dual-layer testing strategy with 3 files implementing contract validation: unit tests use jest.mock for isolation (significance 0.92) and E2E tests use puppeteer for browser validation (significance 0.90, 0.88)
- Public API contracts like UserConfig, UserData, setup, and getBox are explicitly tested across architectural boundaries, demonstrating the need for multi-level validation to catch both logic errors and runtime integration issues
- The pattern of using expect().toEqual() for state validation and page.evaluate() for browser context validation reflects mature testing practices that verify contracts at appropriate abstraction levels
- The 89.73% confidence score across 3 files indicates a consistent, intentional testing strategy rather than ad-hoc test coverage

## Consequences

Positive:
- Public API contracts are validated at multiple architectural boundaries, catching both unit-level logic errors and integration-level runtime failures
- Jest mocking enables fast, deterministic unit tests with isolated dependencies, improving test reliability and execution speed
- Puppeteer-based E2E tests validate actual browser behavior including complex scenarios like iframe coordinate transformations
- Separation of unit and E2E testing concerns allows teams to run fast feedback loops during development while maintaining comprehensive integration coverage

Negative:
- Maintaining two distinct testing frameworks (Jest and Puppeteer) increases tooling complexity and requires expertise in both ecosystems
- E2E tests with browser automation are slower and more resource-intensive than unit tests, potentially impacting CI pipeline duration
- Complex page.evaluate() logic for iframe transformations creates maintenance burden and may be fragile to DOM structure changes
- Mocking strategies in unit tests may not catch integration issues that only manifest when real dependencies interact

## Alternatives

- Use only unit tests with comprehensive mocking to validate all API contracts without E2E browser testing (rejected)
  Rejected because: Unit tests alone cannot validate browser-specific runtime behavior like iframe coordinate transformations, DOM interactions, and actual page navigation flows that are critical to the getBox and setup APIs
  When valid: For pure logic APIs with no browser dependencies or DOM interactions
- Use Playwright instead of Puppeteer for E2E testing with improved cross-browser support and modern API design (deferred)
  Rejected because: Not rejected; Playwright is a viable alternative but the existing Puppeteer implementation is functional and migration would require evidence of specific limitations
  When valid: When cross-browser testing becomes a requirement or Puppeteer limitations are encountered
- Consolidate all testing into Cypress or similar framework that combines unit and E2E capabilities (rejected)
  Rejected because: Cypress is primarily designed for E2E testing and would not provide the same level of unit test isolation and speed that Jest with mocking provides for state transformation validation
  When valid: For projects where E2E testing is the primary concern and unit test performance is less critical

## Risks

- E2E tests may become flaky due to timing issues, network conditions, or browser version changes, reducing confidence in test results
  Mitigation: Implement retry logic, explicit waits, and stable selectors; run E2E tests in controlled CI environments with pinned browser versions
  Owner: engineering team
- Over-mocking in unit tests may create false confidence where tests pass but real integrations fail due to incorrect mock assumptions
  Mitigation: Maintain integration tests that use real dependencies for critical paths; regularly review mock implementations against actual dependency behavior
  Owner: engineering team
- Complex page.evaluate() logic for iframe transformations may break when DOM structure or styling changes, requiring frequent maintenance
  Mitigation: Encapsulate iframe transformation logic in well-documented utility functions; add unit tests for transformation calculations where possible
  Owner: engineering team

## Implementation Notes

- Use jest.mock() at the top of test files to isolate dependencies like generate-id, ensuring mocks are hoisted before module imports
- Structure E2E tests with clear setup/teardown: launch browser with appropriate options, create new pages, navigate to URLs, and clean up resources
- When testing APIs that interact with iframes, use page.evaluate() to execute transformation logic in the browser context where DOM APIs are available
- Validate both the shape and content of data structures using expect().toEqual() for state objects, indexes, and transformed data representations
- Export E2E utility functions like setup and getBox to enable reuse across test suites and maintain consistent browser automation patterns

## Continuation Context


Verify commands:
- grep -r "jest.mock" packages/core/reducer/actions/__helpers__/ | grep -q "generate-id" && echo "Jest mocking verified"
- grep -r "puppeteer.launch" scripts/e2e/utils/ | grep -q "headless" && echo "Puppeteer E2E setup verified"
- grep -r "expect.*toEqual" packages/core/reducer/actions/__helpers__/ | grep -q "state.indexes" && echo "State contract validation verified"
- grep -r "page.evaluate" scripts/e2e/utils/ | grep -q "querySelector" && echo "Browser context evaluation verified"

Accept when:
- All public API contracts have corresponding unit tests using jest.mock for dependency isolation and expect().toEqual() for validation
- Browser-based APIs have E2E tests using puppeteer with page.evaluate() to validate runtime behavior in actual browser contexts
- Verification commands successfully detect jest.mock usage, puppeteer setup, state validation patterns, and page.evaluate() implementations in the codebase

## Enforcement

- Verified by: CI pipeline executes both Jest unit tests and Puppeteer E2E tests on every pull request
- Verified by: Code review checklist requires test coverage for new public API contracts at appropriate boundaries
- Verified by: Automated grep-based verification commands run in pre-commit hooks to detect missing test patterns
- Violation handling: Pull requests without tests for new public API contracts are blocked from merging
- Violation handling: CI failures in unit or E2E test suites prevent deployment to staging and production environments
- Violation handling: Code review feedback requests addition of missing test coverage before approval
- Exception process: Exceptions for untested APIs require architectural review and documented justification in ADR format
- Exception process: Temporary test exemptions must include a tracking ticket and timeline for adding coverage
- Exception process: Legacy APIs without tests must be documented in a technical debt register with prioritization for remediation