# Adopt .find() Predicate Pattern for Test Data Access in Component Testing: Predicates Use Arrow

These rules are ALWAYS ACTIVE for all test files in `packages/core/lib/__tests__/` and `packages/core/lib/data/__tests__/` that use @testing-library/react with describe/it test frameworks to verify component behavior and data transformations.

### Rules

- **R-PRED-001** MUST: Predicates MUST use arrow function syntax with explicit property comparison (e.g., `(item) => item.props.id === "target-id"`).
- **R-PRED-002** MUST: Always assign `.find()` results to a variable and add `expect(variable).toBeDefined()` before accessing nested properties to prevent undefined reference errors.
- **R-PRED-003** SHOULD: For component lookup, use `result.find((item) => item.props.id === 'target-id')` pattern.
- **R-PRED-004** SHOULD: For mocked call verification, use `mockedCalls.find((call) => call[1].trigger === 'action-name')` to locate specific invocations.
- **R-PRED-005** SHOULD: Cache `.find()` results in variables when the same item is accessed multiple times; consider Map-based lookup for large collections.
- **R-PRED-006** MAY: Extract repeated `.find()` patterns into test helper functions if the same predicate logic appears in more than three test files.

### Verify

```bash
# Verify .find() with arrow function predicates for component lookup
grep -r '\.find((.*) => .*\.props\.id ===' packages/core/lib/__tests__/ packages/core/lib/data/__tests__/

# Verify .find() with arrow function predicates for trigger matching
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
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes to test files in the specified directories.
</enforcement>