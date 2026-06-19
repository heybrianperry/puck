# Adopt .find() Predicate Pattern for Test Data Access in Component Testing: Test Code Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- Test suites in packages/core/lib/__tests__/move-component.spec.tsx and packages/core/lib/data/__tests__/flatten-data.spec.tsx use @testing-library/react with describe/it test frameworks
- Tests verify component behavior by accessing flattened data structures and mocked call arrays to locate specific items by property matching
- The codebase uses a data model where components have props.id fields and mocked calls have trigger properties that require predicate-based lookup
- Test assertions depend on finding specific items within collections returned by store operations, data transformations, and component interactions

## Problem Statement

Test code needs a consistent, readable pattern for locating specific items within collections (flattened component data, mocked call arrays) when verifying component behavior, data transformations, and store interactions. Without a standard approach, test data access becomes inconsistent and harder to maintain across the test suite.

## Decision

1. MUST: Test code MUST use Array.find() with predicate functions when locating items in collections by property matching (e.g., props.id, trigger field)

## Policy Block

- MUST Test code MUST use Array.find() with predicate functions when locating items in collections by property matching (e.g., props.id, trigger field)

## Rationale

- The evidence shows consistent use of .find() with arrow function predicates across move-component.spec.tsx and flatten-data.spec.tsx, indicating an established pattern for test data access
- Property-based matching (props.id, trigger field) provides precise item location in collections returned by store operations and data transformations
- This pattern aligns with the data model where components have identifiable props and mocked calls have distinguishable trigger properties
- The approach supports readable test assertions by making the search criteria explicit in the predicate function

## Consequences

Positive:
- Test code becomes more readable with explicit search criteria visible in predicate functions
- Consistent pattern across test files reduces cognitive load when writing and reviewing tests
- Property-based matching provides precise item location without index-based access fragility
- Pattern works naturally with TypeScript type inference for found items

Negative:
- find() returns undefined when no match is found, requiring null checks before property access
- Linear search performance may degrade with large test data collections
- Multiple find() calls on the same collection repeat traversal work
- Pattern assumes unique identifiers; duplicate matches return only the first item

## Alternatives

- Use index-based array access with hardcoded positions (e.g., result[0], mockedCalls[2]) (rejected)
  Rejected because: Index-based access is fragile when test data order changes and provides no semantic meaning about what item is being accessed
  When valid: Only valid for single-item collections where position is guaranteed
- Create helper functions that encapsulate find() logic (e.g., findComponentById(result, 'root')) (deferred)
  Rejected because: Not rejected; could be adopted if pattern usage grows significantly
  When valid: Valid when the same find patterns are repeated frequently across many test files
- Use filter() followed by array destructuring to extract items (rejected)
  Rejected because: filter() returns an array requiring additional destructuring; find() directly returns the item or undefined, which is more concise for single-item lookup
  When valid: Valid only when multiple matching items need to be extracted

## Risks

- Tests may fail with undefined errors if find() returns no match and code accesses properties without null checks
  Mitigation: Add explicit assertions that found items exist (e.g., expect(foundItem).toBeDefined()) before accessing nested properties
  Owner: engineering team
- Performance degradation if find() is called repeatedly on large collections within tight test loops
  Mitigation: Cache find() results in variables when the same item is accessed multiple times; consider Map-based lookup for large collections
  Owner: engineering team
- Predicate logic errors (e.g., wrong property name, incorrect comparison) may cause tests to silently fail by not finding expected items
  Mitigation: Use TypeScript strict mode to catch property name typos; add explicit assertions on find() results
  Owner: engineering team

## Implementation Notes

- When writing new tests in packages/core/lib/__tests__/ or packages/core/lib/data/__tests__/, use result.find((item) => item.props.id === 'target-id') pattern for component lookup
- For mocked call verification, use mockedCalls.find((call) => call[1].trigger === 'action-name') to locate specific invocations
- Always assign find() results to a variable and add expect(variable).toBeDefined() before accessing nested properties
- Consider extracting repeated find() patterns into test helper functions if the same predicate logic appears in more than three test files

## Continuation Context


Verify commands:
- grep -r '\.find((.*) => .*\.props\.id ===' packages/core/lib/__tests__/ packages/core/lib/data/__tests__/
- grep -r '\.find((.*) => .*trigger ===' packages/core/lib/__tests__/
- npm test -- --testPathPattern='(move-component|flatten-data)\.spec\.tsx' --passWithNoTests

Accept when:
- grep commands return matches showing .find() with arrow function predicates in test files
- Test suite passes with no undefined reference errors from find() results
- Code review confirms new test files follow the .find() predicate pattern for data access

## Enforcement

- Verified by: Code review of test files in packages/core/lib/__tests__/ and packages/core/lib/data/__tests__/
- Verified by: CI test execution verifying no undefined errors from data access
- Verified by: Grep-based pattern matching in pre-commit hooks or CI linting stage
- Violation handling: Code review feedback requesting conversion to .find() pattern
- Violation handling: Test failures from undefined errors trigger investigation of data access patterns
- Violation handling: Linting warnings for index-based access in test files (if tooling is configured)
- Exception process: Document rationale in test file comments when alternative patterns are necessary
- Exception process: Obtain approval from test infrastructure maintainers for systematic deviations
- Exception process: Create helper functions for repeated exceptional patterns rather than inline alternatives