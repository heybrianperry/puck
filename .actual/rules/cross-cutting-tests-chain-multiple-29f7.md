# Adopt .find() Predicate Pattern for Test Data Access in Component Testing: Tests Chain Multiple

These rules are ALWAYS ACTIVE for test files in `packages/core/lib/__tests__/` and `packages/core/lib/data/__tests__/` that use @testing-library/react with describe/it test frameworks.

### Rules

- **R-TEST-FIND-001** MAY: Tests MAY chain multiple find() operations when navigating nested data structures or verifying multiple related items.
- **R-TEST-FIND-002** MUST: Always assign find() results to a variable and add `expect(variable).toBeDefined()` before accessing nested properties to prevent undefined reference errors.
- **R-TEST-FIND-003** SHOULD: Use the pattern `result.find((item) => item.props.id === 'target-id')` for component lookup in test assertions.
- **R-TEST-FIND-004** SHOULD: Use the pattern `mockedCalls.find((call) => call[1].trigger === 'action-name')` for mocked call verification.
- **R-TEST-FIND-005** SHOULD: Cache find() results in variables when the same item is accessed multiple times within a test to avoid repeated traversal.
- **R-TEST-FIND-006** SHOULD: Extract repeated find() patterns into test helper functions if the same predicate logic appears in more than three test files.

### Verify

```bash
# Verify .find() with arrow function predicates for component lookup
grep -r '\.find((.*) => .*\.props\.id ===' packages/core/lib/__tests__/ packages/core/lib/data/__tests__/

# Verify .find() with arrow function predicates for mocked call verification
grep -r '\.find((.*) => .*trigger ===' packages/core/lib/__tests__/

# Run test suite to verify no undefined errors from data access
npm test -- --testPathPattern='(move-component|flatten-data)\.spec\.tsx' --passWithNoTests
```

**Accept when:**
- grep commands return matches showing .find() with arrow function predicates in test files
- Test suite passes with no undefined reference errors from find() results
- Code review confirms new test files follow the .find() predicate pattern for data access
- All find() results have explicit `expect(...).toBeDefined()` assertions before property access

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verification commands must pass before accepting changes to test files in the specified directories.
</enforcement>