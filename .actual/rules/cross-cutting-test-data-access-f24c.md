# Adopt .find() Predicate Pattern for Test Data Access in Component Testing: Test Data Access

These rules are ALWAYS ACTIVE for test files in `packages/core/lib/__tests__/` and `packages/core/lib/data/__tests__/` that use @testing-library/react with describe/it test frameworks.

### Rules

- **R-TEST-001** SHOULD: Test data access SHOULD use descriptive variable names that indicate the expected item being located (e.g., `result.find((item) => ...)` where result represents the collection).
- **R-TEST-002** MUST: Always assign `.find()` results to a variable and add `expect(variable).toBeDefined()` before accessing nested properties to prevent undefined reference errors.
- **R-TEST-003** SHOULD: For component lookup, use the pattern `result.find((item) => item.props.id === 'target-id')` to locate specific items by property matching.
- **R-TEST-004** SHOULD: For mocked call verification, use `mockedCalls.find((call) => call[1].trigger === 'action-name')` to locate specific invocations by trigger property.
- **R-TEST-005** MAY: Extract repeated `.find()` patterns into test helper functions if the same predicate logic appears in more than three test files.

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
- Grep commands return matches showing `.find()` with arrow function predicates in test files
- Test suite passes with no undefined reference errors from `.find()` results
- Code review confirms new test files follow the `.find()` predicate pattern for data access
- All `.find()` results are assigned to variables with preceding `expect(...).toBeDefined()` assertions

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review of test files and CI test execution are mandatory before accepting changes that modify test data access patterns.
</enforcement>