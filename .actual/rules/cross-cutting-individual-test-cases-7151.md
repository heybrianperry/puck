# Adopt describe/it Test Structure with @testing-library/react for Component Testing: Individual Test Cases

These rules are ALWAYS ACTIVE for all component test files in `packages/core/components/**/__tests__/` that verify UI behavior in response to state changes, form field components and their preview rendering, and integration tests that render components and verify DOM output.

### Rules

- **R-TEST-001** MUST: Individual test cases in `it` blocks MUST describe the specific behavior being verified in plain language.
- **R-TEST-002** MUST: All component test files MUST use `describe` blocks to organize test suites hierarchically.
- **R-TEST-003** MUST: All component test files MUST import and use `@testing-library/react` for component rendering.
- **R-TEST-004** MUST: Test cases MUST use `it()` or `test()` blocks with descriptive names that explain the behavior being verified.
- **R-TEST-005** SHOULD: Prefer semantic queries (`getByRole`, `getByLabelText`) over structural queries (`getByTestId`) to maintain test resilience.
- **R-TEST-006** SHOULD: Complement programmatic dispatch tests with user-event library interactions that simulate real user behavior.
- **R-TEST-007** SHOULD: Document all supported interaction modes for each field type and require test cases for each mode in code review.

### Verify

```bash
# Count describe blocks in component test files
grep -r "describe(" packages/core/components/**/__tests__/*.spec.tsx | wc -l

# Count @testing-library/react imports
grep -r "@testing-library/react" packages/core/components/**/__tests__/*.spec.tsx | wc -l

# Count it/test blocks
grep -r "it(\|test(" packages/core/components/**/__tests__/*.spec.tsx | wc -l
```

**Accept when:**
- All component test files in `packages/core/components/**/__tests__/` use `describe` blocks to organize test suites
- All component test files import and use `@testing-library/react` for component rendering
- Test cases use `it()` or `test()` blocks with descriptive names that explain the behavior being verified
- Individual test cases describe specific behavior in plain language rather than implementation details

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules during code review and pull request assessment. Violations MUST be flagged for refactoring or blocked from merge per the ADR's violation handling policy.
</enforcement>