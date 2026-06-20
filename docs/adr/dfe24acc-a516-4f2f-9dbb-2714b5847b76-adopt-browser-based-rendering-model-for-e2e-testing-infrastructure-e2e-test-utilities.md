# Adopt Browser-Based Rendering Model for E2E Testing Infrastructure: E2e Test Utilities

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- E2E testing utilities require programmatic access to browser rendering APIs to measure and validate UI component behavior
- Test infrastructure needs to interact with the DOM and compute visual properties like bounding boxes for element positioning validation
- The pattern appears in utility modules (setup.mjs, get-box.mjs) that provide foundational browser interaction capabilities for test scenarios
- Browser-based rendering enables accurate simulation of user interactions and visual regression testing in automated test suites

## Problem Statement

E2E testing infrastructure requires a consistent approach to interact with browser rendering engines to validate UI behavior, measure element properties, and ensure visual correctness. Without a standardized rendering model, test utilities may use inconsistent APIs, leading to flaky tests and unreliable visual validation.

## Decision

1. SHOULD: E2E test utilities SHOULD expose public contracts for common rendering operations to promote reusability across test suites

## Policy Block

- SHOULD E2E test utilities SHOULD expose public contracts for common rendering operations to promote reusability across test suites

In scope:
- E2E test utility modules that interact with browser rendering engines
- Test setup and teardown scripts that configure browser contexts
- Helper functions that measure or validate UI element properties
- Visual regression testing utilities

Out of scope:
- Unit tests that do not require browser rendering
- Backend API integration tests
- Performance benchmarking tools (unless they measure rendering performance)
- Static code analysis tools

## Rationale

- Pattern detected with 88.75% confidence across 2 E2E utility files (setup.mjs, get-box.mjs), indicating a deliberate architectural choice for test infrastructure
- Browser-based rendering provides the most accurate representation of user experience and enables reliable visual validation
- Centralized utility modules for rendering operations reduce code duplication and ensure consistent behavior across test suites
- Public API contracts in the 'api.public.contracts' facet suggest these utilities are designed for reuse across multiple test scenarios

## Consequences

Positive:
- Consistent and reliable E2E tests that accurately reflect real user interactions with rendered UI
- Reusable utility functions reduce test code duplication and maintenance burden
- Standardized rendering model enables visual regression testing and layout validation
- Clear public contracts make it easier for developers to write new E2E tests following established patterns

Negative:
- Browser-based rendering adds overhead to test execution time compared to non-rendering alternatives
- Tests become dependent on browser rendering engine behavior, which may vary across browser versions
- Requires maintaining browser automation infrastructure (e.g., Playwright, Puppeteer)
- Flakiness may occur if rendering timing and layout stabilization are not properly handled

## Alternatives

- Use JSDOM or similar virtual DOM implementations for E2E testing (rejected)
  Rejected because: Virtual DOM implementations do not provide accurate rendering metrics or layout calculations, making them unsuitable for visual validation and bounding box measurements
  When valid: Acceptable for unit tests that only need basic DOM manipulation without rendering validation
- Inline rendering logic directly in each test file without shared utilities (rejected)
  Rejected because: Leads to code duplication, inconsistent rendering approaches, and increased maintenance burden across test suites
  When valid: Only for one-off tests with unique rendering requirements that cannot be generalized
- Use screenshot-based visual regression testing exclusively (deferred)
  Rejected because: Screenshot comparison alone lacks programmatic access to element properties needed for precise validation
  When valid: Can complement browser-based rendering for comprehensive visual regression coverage

## Risks

- Browser rendering timing issues may cause flaky tests if elements are measured before layout stabilization
  Mitigation: Implement explicit wait mechanisms and layout stabilization checks in utility functions before measuring element properties
  Owner: QA Engineering Team
- Cross-browser rendering differences may cause tests to pass in one browser but fail in others
  Mitigation: Run E2E test suites across multiple browser engines (Chromium, Firefox, WebKit) in CI pipeline
  Owner: DevOps and QA Teams
- Utility module API changes may break existing tests that depend on public contracts
  Mitigation: Version utility modules and maintain backward compatibility; use semantic versioning for breaking changes
  Owner: Engineering Team

## Implementation Notes

- Create centralized utility modules (e.g., setup.mjs, get-box.mjs) in scripts/e2e/utils/ directory for shared rendering operations
- Use browser automation frameworks (Playwright, Puppeteer) that provide reliable APIs for element measurement and rendering interaction
- Implement helper functions that wait for network idle and layout stabilization before measuring element properties
- Document public API contracts for utility functions to ensure consistent usage across test suites
- Consider creating a test utilities package if rendering utilities need to be shared across multiple projects

## Continuation Context


Verify commands:
- grep -r 'getBoundingClientRect\|getComputedStyle' scripts/e2e/utils/ --include='*.mjs' --include='*.js'
- test -f scripts/e2e/utils/setup.mjs && test -f scripts/e2e/utils/get-box.mjs
- npm test -- --grep 'e2e' 2>&1 | grep -q 'passed' || echo 'E2E tests not found or failing'

Accept when:
- E2E utility modules exist in scripts/e2e/utils/ and use browser-native rendering APIs (getBoundingClientRect, getComputedStyle, etc.)
- Test setup modules initialize browser contexts with consistent configurations before running tests
- E2E test suite passes successfully with rendering-based validations for UI element properties

## Enforcement

- Verified by: Code review of E2E test utilities to ensure browser-native rendering APIs are used
- Verified by: CI pipeline runs E2E test suite and validates that rendering-based tests pass
- Verified by: Static analysis checks for presence of required utility modules (setup.mjs, get-box.mjs)
- Violation handling: Pull requests that introduce E2E tests without using standardized rendering utilities are flagged in code review
- Violation handling: CI pipeline fails if E2E tests do not follow the established rendering model patterns
- Violation handling: Violations are documented and teams are notified to refactor tests to use centralized utility modules
- Exception process: Exceptions may be granted for tests with unique rendering requirements that cannot be generalized
- Exception process: Exception requests must be submitted to the QA Engineering lead with justification
- Exception process: Approved exceptions are documented in test file comments with rationale and review date