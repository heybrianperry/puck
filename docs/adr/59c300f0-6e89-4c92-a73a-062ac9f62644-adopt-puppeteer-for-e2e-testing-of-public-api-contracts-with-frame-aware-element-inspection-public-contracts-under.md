# Adopt Puppeteer for E2E Testing of Public API Contracts with Frame-Aware Element Inspection: Public Contracts Under

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- E2E testing infrastructure exists in scripts/e2e/utils/ for validating public API contracts through browser automation
- The codebase uses Puppeteer as the core library for browser control, page navigation, and element interaction
- Testing scenarios require inspecting elements within nested iframe contexts, specifically a #preview-frame element
- Public API contracts named 'setup' and 'getBox' are exposed as testable interfaces
- Frame transformation calculations (position, scale) are necessary to accurately locate elements across iframe boundaries

## Problem Statement

Public API contracts require end-to-end validation in realistic browser environments, including scenarios where UI components are rendered within nested iframes. Standard element inspection fails to account for frame boundaries, coordinate transformations, and scaling factors, leading to inaccurate positioning data and unreliable test assertions.

## Decision

1. SHOULD: Public API contracts under test SHOULD be named descriptively (e.g., 'setup', 'getBox')

## Policy Block

- SHOULD Public API contracts under test SHOULD be named descriptively (e.g., 'setup', 'getBox')

In scope:
- E2E tests validating public/external API contracts
- Browser automation for UI component testing
- Element inspection across iframe boundaries
- Coordinate and scale transformation calculations
- Tests in scripts/e2e/ directory

Out of scope:
- Unit tests of individual functions
- Integration tests without browser automation
- Server-side API testing
- Performance or load testing
- Tests not involving iframe contexts

## Rationale

- Puppeteer provides a stable, well-documented API for browser automation with strong support for page.evaluate() enabling complex DOM inspection logic
- Nested iframe scenarios require explicit frame transformation calculations that standard element queries do not provide
- The evidence shows 2 files implementing this pattern with 88.75% confidence, indicating established practice for public API contract validation
- Frame-aware element inspection enables accurate positioning data essential for visual regression testing and interaction simulation

## Consequences

Positive:
- Accurate element positioning data across iframe boundaries enables reliable E2E test assertions
- Puppeteer's mature ecosystem provides extensive documentation and community support
- Configurable headless mode allows tests to run in CI environments or with visible browser for debugging
- Reusable utility functions (setup, getBox) reduce duplication across test suites

Negative:
- Puppeteer adds a heavyweight dependency requiring Chromium download and maintenance
- Frame transformation logic increases test complexity and maintenance burden
- Browser automation tests are slower than unit or integration tests
- Tests are tightly coupled to DOM structure and iframe implementation details

## Alternatives

- Use Playwright for cross-browser E2E testing (rejected)
  Rejected because: Evidence shows established Puppeteer usage; migration would require rewriting existing test infrastructure without clear benefit for current requirements
  When valid: When cross-browser compatibility becomes a requirement or when Playwright-specific features are needed
- Use Cypress for E2E testing with built-in iframe support (rejected)
  Rejected because: Cypress has limitations with iframe handling and cross-origin scenarios; existing Puppeteer implementation provides more control over frame transformation logic
  When valid: When test scenarios do not involve complex iframe transformations or when Cypress's developer experience benefits outweigh control requirements
- Avoid iframe-based architecture to simplify testing (rejected)
  Rejected because: Iframe architecture appears to be a product requirement (preview-frame) that cannot be changed solely for testing convenience
  When valid: When redesigning the application architecture and iframe isolation is no longer required

## Risks

- Puppeteer version updates may introduce breaking changes to browser automation APIs
  Mitigation: Pin Puppeteer version in package.json and test upgrades in isolated environment before deployment
  Owner: engineering team
- Frame transformation logic may fail with deeply nested or dynamically created iframes
  Mitigation: Add test coverage for edge cases and implement boundary checks in getFrameTransform function
  Owner: engineering team
- E2E tests may become flaky due to timing issues or race conditions in page loading
  Mitigation: Implement explicit wait strategies and retry logic for element queries and page navigation
  Owner: engineering team

## Implementation Notes

- Initialize Puppeteer with puppeteer.launch({ headless, ...options }) allowing configuration override for debugging
- Implement getFrameTransform recursively to handle arbitrary iframe nesting depth, accumulating position and scale transformations
- Use page.evaluate() to execute frame inspection logic in browser context, returning serializable coordinate data
- Structure E2E utilities as separate modules (setup.mjs, get-box.mjs) for reusability across test files
- Handle null cases when elements are not found in either main document or frame contexts

## Continuation Context


Verify commands:
- grep -r 'puppeteer.launch' scripts/e2e/ | grep -q 'headless'
- grep -r 'page.evaluate' scripts/e2e/ | grep -q 'getFrameTransform'
- test -f scripts/e2e/utils/setup.mjs && test -f scripts/e2e/utils/get-box.mjs

Accept when:
- Puppeteer launch configuration is present in scripts/e2e/utils/setup.mjs with headless option support
- Frame transformation logic exists in page.evaluate calls calculating position and scale
- Both setup.mjs and get-box.mjs utility files exist in scripts/e2e/utils/ directory

## Enforcement

- Verified by: Automated grep checks in CI pipeline verifying Puppeteer usage patterns
- Verified by: Code review ensuring E2E tests follow established utility patterns
- Verified by: Test execution in CI validating frame-aware element inspection functions
- Violation handling: CI build fails if E2E tests do not use Puppeteer or bypass frame transformation logic
- Violation handling: Code review blocks merges that introduce alternative browser automation libraries without architectural discussion
- Violation handling: Test failures due to incorrect element positioning trigger investigation of frame transformation implementation
- Exception process: Document alternative approach in ADR if different browser automation tool is required for specific test scenarios
- Exception process: Obtain approval from architecture review board for deviations from Puppeteer standard
- Exception process: Update this ADR with new alternatives or policy exceptions if valid use cases emerge