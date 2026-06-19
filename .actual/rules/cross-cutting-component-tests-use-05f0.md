# Adopt describe/it Test Structure with @testing-library/react for Component Testing: Component Tests Use

These rules are ALWAYS ACTIVE for all component test files in `packages/core/components/**/__tests__/` that verify UI behavior in response to state changes, form field components, and their preview rendering.

### Rules

- **R-COMP-001** MUST: Component tests MUST use @testing-library/react for rendering and interacting with React components.
- **R-COMP-002** MUST: Component test files MUST use describe blocks to organize test suites hierarchically by component and feature area.
- **R-COMP-003** MUST: Test cases MUST use it() or test() blocks with descriptive names that explain the behavior being verified.
- **R-COMP-004** SHOULD: Prefer semantic queries (getByRole, getByLabelText) over structural queries (getByTestId) to maintain test resilience during refactoring.
- **R-COMP-005** SHOULD: Complement programmatic dispatch tests with user-event library interactions that simulate real user behavior for critical user flows.
- **R-COMP-006** SHOULD: Document all supported interaction modes for each field type and require test cases for each mode in code review.

### Verify

```bash
# Count describe blocks in component test files
grep -r "describe(" packages/core/components/**/__tests__/*.spec.tsx | wc -l

# Verify @testing-library/react imports in test files
grep -r "@testing-library/react" packages/core/components/**/__tests__/*.spec.tsx | wc -l

# Count it() and test() blocks in component test files
grep -r "it(\|test(" packages/core/components/**/__tests__/*.spec.tsx | wc -l
```

**Accept when:**
- All component test files in `packages/core/components/**/__tests__/` use describe blocks to organize test suites
- All component test files import and use @testing-library/react for component rendering
- Test cases use it() or test() blocks with descriptive names that explain the behavior being verified
- Tests verify state management propagation to UI components through dispatch-to-preview flows
- Multiple interaction modes (standard and contentEditable) are tested for complex field types

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules during component test review and implementation.
</enforcement>