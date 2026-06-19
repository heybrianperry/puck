# Adopt describe/it Test Structure with @testing-library/react for Component Testing: Test Files Import

These rules are ALWAYS ACTIVE for all component test files in `packages/core/components/**/__tests__/` that verify UI behavior in response to state changes, form field components, and their preview rendering.

### Rules

- **R-TEST-001** MAY: Test files MAY import @testing-library/jest-dom for enhanced DOM assertion matchers.
- **R-TEST-002** MUST: Component test files MUST use describe blocks to organize test suites hierarchically by component and feature area.
- **R-TEST-003** MUST: Test files MUST import and use @testing-library/react for component rendering and interaction testing.
- **R-TEST-004** MUST: Test cases MUST use it() or test() blocks with descriptive names that explain the behavior being verified.
- **R-TEST-005** MUST: Test files MUST be placed in __tests__ directories adjacent to the components being tested, following the pattern `packages/core/components/[Component]/__tests__/[feature].spec.tsx`.
- **R-TEST-006** SHOULD: Tests SHOULD use semantic queries (getByRole, getByLabelText) over structural queries (getByTestId) to maintain resilience to component structure changes.
- **R-TEST-007** SHOULD: State update tests SHOULD dispatch the action, then query the DOM to verify the expected changes appear in the preview.

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
- Test files follow the naming convention `[feature].spec.tsx` and are located in `__tests__` directories adjacent to components

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules during code review and test file creation. Violations must be flagged and remediated before merge.
</enforcement>