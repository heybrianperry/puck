# Adopt .find() Predicate Pattern for Test Data Access in Component Testing: Tests Verify Found

These rules are ALWAYS ACTIVE for test files in `packages/core/lib/__tests__/` and `packages/core/lib/data/__tests__/` that use @testing-library/react with describe/it test frameworks to verify component behavior and data transformations.

### Rules

- **R-TEST-FIND-001** SHOULD: Tests SHOULD verify the found item exists before accessing nested properties to avoid undefined reference errors.
- **R-TEST-FIND-002** SHOULD: Use `result.find((item) => item.props.id === 'target-id')` pattern for component lookup in test assertions.
- **R-TEST-FIND-003** SHOULD: Use `mockedCalls.find((call) => call[1].trigger === 'action-name')` pattern for mocked call verification.
- **R-TEST-FIND-004** SHOULD: Assign find() results to a variable and add `expect(variable).toBeDefined()` before accessing nested properties.
- **R-TEST-FIND-005** MAY: Extract repeated find() patterns into test helper functions if the same predicate logic appears in more than three test files.

### Verify

```bash
# Verify .find() with arrow function predicates for component lookup
grep -r '\.find((.*) => .*\.props\.id ===' packages/core/lib/__tests__/ packages/core/lib/data/__tests__/

# Verify .find() with arrow function predicates for mocked calls
grep -r '\.find((.*) => .*trigger ===' packages/core/lib/__tests__/

# Run test suite to verify no undefined reference errors
npm test -- --testPathPattern='(move-component|flatten-data)\.spec\.tsx' --passWithNoTests
```

**Accept when:**
- grep commands return matches showing .find() with arrow function predicates in test files
- Test suite passes with no undefined reference errors from find() results
- Code review confirms new test files follow the .find() predicate pattern for data access
- Tests include explicit assertions verifying found items exist before property access

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verification commands MUST pass before accepting changes to test files in the specified scope.
</enforcement>