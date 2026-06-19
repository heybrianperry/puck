# Adopt .find() Predicate Pattern for Test Data Access in Component Testing: Test Code Use

These rules are ALWAYS ACTIVE for test code in packages/core/lib/__tests__/ and packages/core/lib/data/__tests__/ directories, particularly in files using @testing-library/react with describe/it test frameworks that access flattened data structures and mocked call arrays.

### Rules

- **R-TEST-001** MUST: Test code MUST use Array.find() with predicate functions when locating items in collections by property matching (e.g., props.id, trigger field).
- **R-TEST-002** MUST: Always assign find() results to a variable and add expect(variable).toBeDefined() before accessing nested properties to prevent undefined reference errors.
- **R-TEST-003** SHOULD: Cache find() results in variables when the same item is accessed multiple times within a test.
- **R-TEST-004** SHOULD: Extract repeated find() patterns into test helper functions if the same predicate logic appears in more than three test files.

### Verify

```bash
# Verify .find() with arrow function predicates for component lookup
grep -r '\.find((.*) => .*\.props\.id ===' packages/core/lib/__tests__/ packages/core/lib/data/__tests__/

# Verify .find() with arrow function predicates for mocked call verification
grep -r '\.find((.*) => .*trigger ===' packages/core/lib/__tests__/

# Run test suite to verify no undefined reference errors
npm test -- --testPathPattern='(move-component|flatten-data)\.spec\.tsx' --passWithNoTests
```

**Accept when:**
- grep commands return matches showing .find() with arrow function predicates in test files
- Test suite passes with no undefined reference errors from find() results
- Code review confirms new test files follow the .find() predicate pattern for data access
- Explicit assertions (expect(variable).toBeDefined()) are present before accessing properties on find() results

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review of test files and CI test execution are mandatory before accepting changes that access test data collections.
</enforcement>