# Adopt describe/it Test Structure with @testing-library/react for Component Testing: Individual Test Cases

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- Component testing in the packages/core/components/Puck module requires a structured approach to organize test cases and assertions for UI behavior validation
- The @testing-library/react library provides React-specific testing utilities that enable component rendering and interaction testing without implementation details
- The describe/it pattern establishes a hierarchical test organization that groups related test cases under descriptive suite names
- Programmatic state updates via dispatch actions require verification that UI changes propagate correctly to preview components
- The richtext field type represents a complex input scenario requiring both standard and contentEditable interaction patterns

## Problem Statement

Component tests must verify that programmatic state changes correctly update the UI preview, particularly for complex field types like richtext that support both standard and contentEditable modes, while maintaining clear test organization and readability across the testing suite.

## Decision

1. MUST: Individual test cases in it blocks MUST describe the specific behavior being verified in plain language

## Policy Block

- MUST Individual test cases in it blocks MUST describe the specific behavior being verified in plain language

In scope:
- Component test files in packages/core/components/**/__tests__/
- Tests verifying UI behavior in response to state changes
- Tests for form field components and their preview rendering
- Integration tests that render components and verify DOM output

Out of scope:
- Unit tests for pure functions without UI rendering
- End-to-end tests using browser automation tools
- Backend API tests
- Performance or load tests

## Rationale

- The describe/it pattern provides hierarchical test organization that improves readability and maintenance by grouping related test cases under descriptive suite names
- @testing-library/react aligns with React testing best practices by focusing on user-facing behavior rather than implementation details, making tests more resilient to refactoring
- The evidence shows explicit testing of programmatic updates via dispatch, indicating a pattern of verifying that state management correctly propagates to UI components
- Testing both standard and contentEditable richtext modes demonstrates comprehensive coverage of field interaction patterns

## Consequences

Positive:
- Clear test organization through describe/it hierarchy improves test discoverability and maintenance
- Testing library approach focuses on user behavior rather than implementation details, reducing test brittleness
- Explicit verification of dispatch-to-preview flow ensures state management integrity
- Separate test cases for interaction modes provide comprehensive coverage of field behavior variants

Negative:
- @testing-library/react adds a dependency that must be maintained and updated
- Testing library's philosophy may require learning curve for developers familiar with enzyme or other testing approaches
- Comprehensive testing of multiple interaction modes increases test suite execution time
- Programmatic dispatch testing may not catch issues that only occur through actual user interactions

## Alternatives

- Use Enzyme for component testing with shallow rendering (rejected)
  Rejected because: Enzyme focuses on implementation details and shallow rendering, which creates brittle tests that break during refactoring and don't verify actual user-facing behavior
  When valid: When testing legacy React components that require access to component instance methods or internal state
- Use plain Jest without structured describe/it blocks (rejected)
  Rejected because: Flat test structure without describe blocks reduces organization and makes it difficult to group related test cases or understand test scope at a glance
  When valid: For very simple test files with only 1-2 test cases where hierarchy adds no value
- Use React Testing Library with test() instead of it() (deferred)
  Rejected because: Both test() and it() are aliases in Jest; it() was chosen but test() would be functionally equivalent
  When valid: When team prefers test() syntax for readability or consistency with other projects

## Risks

- Testing library queries may become complex or fragile when component structure changes significantly
  Mitigation: Use semantic queries (getByRole, getByLabelText) over structural queries (getByTestId) to maintain resilience; establish query patterns in testing guidelines
  Owner: engineering team
- Incomplete coverage of interaction modes may leave edge cases untested
  Mitigation: Document all supported interaction modes for each field type; require test cases for each mode in code review checklist
  Owner: engineering team
- Programmatic dispatch tests may pass while actual user interactions fail due to event handling issues
  Mitigation: Complement dispatch tests with user-event library interactions that simulate real user behavior; include integration tests for critical user flows
  Owner: engineering team

## Implementation Notes

- Place component test files in __tests__ directories adjacent to the components being tested, following the pattern packages/core/components/[Component]/__tests__/[feature].spec.tsx
- Import render and other utilities from @testing-library/react; import jest-dom matchers for enhanced assertions
- Structure each test file with a top-level describe block naming the component and feature area, with nested it blocks for individual test cases
- For state update tests, dispatch the action, then query the DOM to verify the expected changes appear in the preview

## Continuation Context


Verify commands:
- grep -r "describe(" packages/core/components/**/__tests__/*.spec.tsx | wc -l
- grep -r "@testing-library/react" packages/core/components/**/__tests__/*.spec.tsx | wc -l
- grep -r "it(\|test(" packages/core/components/**/__tests__/*.spec.tsx | wc -l

Accept when:
- All component test files in packages/core/components/**/__tests__/ use describe blocks to organize test suites
- All component test files import and use @testing-library/react for component rendering
- Test cases use it() or test() blocks with descriptive names that explain the behavior being verified

## Enforcement

- Verified by: Code review checklist verifying test structure and testing library usage
- Verified by: CI pipeline runs test suite and reports coverage metrics
- Verified by: Linting rules enforce test file naming conventions and location
- Violation handling: Pull requests with component changes lacking corresponding tests are blocked from merge
- Violation handling: Tests not following describe/it structure are flagged in code review for refactoring
- Violation handling: Tests using deprecated testing approaches (e.g., Enzyme) trigger migration tasks
- Exception process: Exceptions for alternative testing approaches require architecture team approval with documented justification
- Exception process: Legacy test files may temporarily use different patterns but must be tagged for migration
- Exception process: Experimental features may defer comprehensive testing until API stabilizes