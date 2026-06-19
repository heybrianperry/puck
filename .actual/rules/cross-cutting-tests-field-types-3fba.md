# Adopt describe/it Test Structure with @testing-library/react for Component Testing: Tests Field Types

These rules are ALWAYS ACTIVE for all component test files in `packages/core/components/**/__tests__/` that verify UI behavior in response to state changes, form field components and their preview rendering, and integration tests that render components and verify DOM output.

### Rules

- **R-TEST-001** SHOULD: Tests for field types with multiple interaction modes SHOULD include separate test cases for each mode (e.g., standard vs contentEditable).
- **R-TEST-002** MUST: Component test files MUST use describe blocks to organize test suites hierarchically.
- **R-TEST-003** MUST: Component test files MUST import and use @testing-library/react for component rendering.
- **R-TEST-004** MUST: Test cases MUST use it() or test() blocks with descriptive names that explain the behavior being verified.
- **R-TEST-005** SHOULD: Semantic queries (getByRole, getByLabelText) SHOULD be preferred over structural queries (getByTestId) to maintain resilience to component structure changes.
- **R-TEST-006** SHOULD: Programmatic dispatch tests SHOULD be complemented with user-event library interactions that simulate real user behavior.

### Verify

```bash
# Count describe blocks in component test files
grep -r "describe(" packages/core/components/**/__tests__/*.spec.tsx | wc -l

# Count @testing-library/react imports
grep -r "@testing-library/react" packages/core/components/**/__tests__/*.spec.tsx | wc -l

# Count it() and test() blocks
grep -r "it(\|test(" packages/core/components/**/__tests__/*.spec.tsx | wc -l
```

**Accept when:**
- All component test files in `packages/core/components/**/__tests__/` use describe blocks to organize test suites
- All component test files import and use @testing-library/react for component rendering
- Test cases use it() or test() blocks with descriptive names that explain the behavior being verified
- Tests for field types with multiple interaction modes include separate test cases for each mode
- Semantic queries are used in preference to structural queries where applicable

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules during code review and test file creation. All component test files must conform to this structure before merge.
</enforcement>