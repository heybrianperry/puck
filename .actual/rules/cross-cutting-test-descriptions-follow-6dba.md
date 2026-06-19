# Standardize on describe/it Test Structure for Component Testing: Test Descriptions Follow

These rules are ALWAYS ACTIVE for all component test files in `packages/core/components/**/__tests__/` directories using @testing-library/react and @testing-library/jest-dom.

### Rules

- **R-TEST-001** SHOULD: Test descriptions SHOULD follow the pattern 'component name - feature area' for describe blocks and 'action/condition - expected result' for it blocks.

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
- All component test files in `packages/core/components/**/__tests__/` use describe blocks to organize tests
- Each test case is defined with an it block containing a descriptive test name
- @testing-library/react and @testing-library/jest-dom are imported in all component test files
- Describe blocks follow the naming pattern 'component name - feature area'
- It blocks follow the naming pattern 'action/condition - expected result'

<enforcement>
Clause Code MUST NOT skip or defer verification of test structure compliance during code review and CI pipeline execution.
</enforcement>