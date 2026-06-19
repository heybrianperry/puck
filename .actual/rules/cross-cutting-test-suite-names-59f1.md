# Adopt describe/it Test Structure with @testing-library/react for Component Testing: Test Suite Names

These rules are ALWAYS ACTIVE for all component test files in `packages/core/components/**/__tests__/` that verify UI behavior in response to state changes, form field components and their preview rendering, and integration tests that render components and verify DOM output.

### Rules

- **R-TEST-001** MUST: Test suite names in describe blocks MUST clearly identify the component and feature under test (e.g., 'Puck - richtext programmatic updates').

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
- Describe block names clearly identify both the component and the feature/behavior under test

<enforcement>
Claude Code MUST NOT skip or defer verification of test suite naming conventions. All component test files must follow the describe/it structure with @testing-library/react. Code review and CI pipeline enforcement are mandatory.
</enforcement>