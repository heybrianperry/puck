# Adopt describe/it Test Structure with @testing-library/react for Component Testing: Tests Verifying State

These rules are ALWAYS ACTIVE for all component test files in `packages/core/components/**/__tests__/` that verify UI behavior in response to state changes, particularly for form field components and their preview rendering.

### Rules

- **R-TEST-001** SHOULD: Tests verifying state updates SHOULD test both the dispatch action and the resulting preview changes.

### Verify

```bash
# Count describe blocks in component test files
grep -r "describe(" packages/core/components/**/__tests__/*.spec.tsx | wc -l

# Verify @testing-library/react is imported
grep -r "@testing-library/react" packages/core/components/**/__tests__/*.spec.tsx | wc -l

# Count it/test blocks for individual test cases
grep -r "it(\|test(" packages/core/components/**/__tests__/*.spec.tsx | wc -l
```

**Accept when:**
- All component test files in `packages/core/components/**/__tests__/` use describe blocks to organize test suites
- All component test files import and use @testing-library/react for component rendering
- Test cases use `it()` or `test()` blocks with descriptive names that explain the behavior being verified
- State update tests explicitly verify both the dispatch action and the resulting DOM changes in the preview

<enforcement>
Clause Code MUST NOT skip or defer verification of these rules during component test file review and CI pipeline execution.
</enforcement>