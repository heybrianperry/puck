# Standardize on describe/it Test Structure for Component Testing: Individual Test Cases

These rules are ALWAYS ACTIVE for all component test files in `packages/core/components/**/__tests__/` directories.

### Rules

- **R-TEST-001** MUST: Individual test cases MUST be defined using `it` blocks with descriptive names that clearly state the expected behavior.

### Verify

```bash
# Count describe blocks in component test files
grep -r "describe(" packages/core/components/**/__tests__/*.tsx | wc -l

# Count it blocks in component test files
grep -r "it(" packages/core/components/**/__tests__/*.tsx | wc -l

# Verify @testing-library/react imports
grep -r "@testing-library/react" packages/core/components/**/__tests__/*.tsx | wc -l
```

**Accept when:**
- All component test files in `packages/core/components/**/__tests__/` use `describe` blocks to organize tests
- Each test case is defined with an `it` block containing a descriptive test name
- `@testing-library/react` and `@testing-library/jest-dom` are imported in all component test files

<enforcement>
Claude Code MUST NOT skip or defer verification of test structure compliance.
</enforcement>