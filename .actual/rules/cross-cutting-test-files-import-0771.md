# Standardize on describe/it Test Structure for Component Testing: Test Files Import

These rules are ALWAYS ACTIVE for all component test files in packages/core/components/**/__tests__/ directories.

### Rules

- **R-TEST-001** MUST: Test files MUST import @testing-library/jest-dom for DOM assertion matchers

### Verify

```bash
# Verify describe blocks are used
grep -r "describe(" packages/core/components/**/__tests__/*.tsx | wc -l

# Verify it blocks are used
grep -r "it(" packages/core/components/**/__tests__/*.tsx | wc -l

# Verify @testing-library/react is imported
grep -r "@testing-library/react" packages/core/components/**/__tests__/*.tsx | wc -l

# Verify @testing-library/jest-dom is imported
grep -r "@testing-library/jest-dom" packages/core/components/**/__tests__/*.tsx | wc -l
```

**Accept when:**
- All component test files in packages/core/components/**/__tests__/ use describe blocks to organize tests
- Each test case is defined with an it block containing a descriptive test name
- @testing-library/react and @testing-library/jest-dom are imported in all component test files
- Test files are located in __tests__ directories adjacent to the components they test
- Tests follow the pattern packages/core/components/[Component]/__tests__/[test-file].spec.tsx

<enforcement>
Claude Code MUST NOT skip or defer verification of @testing-library/jest-dom imports in component test files.
</enforcement>