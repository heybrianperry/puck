# Adopt describe/it Test Structure with @testing-library/react for Component Testing: Component Test Files

These rules are ALWAYS ACTIVE for all component test files in `packages/core/components/**/__tests__/` that verify UI behavior in response to state changes, form field components and their preview rendering, and integration tests that render components and verify DOM output.

### Rules

- **R-COMP-TEST-001** MUST: Component test files MUST use the describe/it structure to organize test suites and individual test cases.
- **R-COMP-TEST-002** MUST: Component test files MUST import and use @testing-library/react for component rendering and interaction testing.
- **R-COMP-TEST-003** MUST: Test cases MUST use it() or test() blocks with descriptive names that explain the behavior being verified.
- **R-COMP-TEST-004** SHOULD: Prefer semantic queries (getByRole, getByLabelText) over structural queries (getByTestId) to maintain test resilience during refactoring.
- **R-COMP-TEST-005** SHOULD: Structure each test file with a top-level describe block naming the component and feature area, with nested it blocks for individual test cases.
- **R-COMP-TEST-006** SHOULD: For state update tests, dispatch the action, then query the DOM to verify the expected changes appear in the preview.
- **R-COMP-TEST-007** SHOULD: Document all supported interaction modes for each field type and require test cases for each mode in code review.
- **R-COMP-TEST-008** SHOULD: Complement programmatic dispatch tests with user-event library interactions that simulate real user behavior for critical user flows.

### Verify

```bash
# Count describe blocks in component test files
grep -r "describe(" packages/core/components/**/__tests__/*.spec.tsx | wc -l

# Verify @testing-library/react imports
grep -r "@testing-library/react" packages/core/components/**/__tests__/*.spec.tsx | wc -l

# Count it/test blocks in component test files
grep -r "it(\|test(" packages/core/components/**/__tests__/*.spec.tsx | wc -l
```

**Accept when:**
- All component test files in `packages/core/components/**/__tests__/` use describe blocks to organize test suites
- All component test files import and use @testing-library/react for component rendering
- Test cases use it() or test() blocks with descriptive names that explain the behavior being verified
- Semantic queries are preferred over structural queries in DOM assertions
- Tests for complex field types (e.g., richtext) include separate test cases for each interaction mode

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules during component test file review. All violations MUST be flagged in code review for remediation before merge.
</enforcement>