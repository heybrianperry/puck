# Standardize on describe/it Test Structure for Component Testing: Component Test Files

These rules are ALWAYS ACTIVE for all component test files in `packages/core/components/**/__tests__/` directories that test React components using @testing-library/react and @testing-library/jest-dom.

### Rules

- **R-COMP-001** MUST: All component test files MUST use describe blocks to group related test cases by component or feature area.
- **R-COMP-002** MUST: Each test case MUST be defined with an it block containing a descriptive test name.
- **R-COMP-003** MUST: All component test files MUST import @testing-library/react and @testing-library/jest-dom at the top of the file.
- **R-COMP-004** MUST: Test files MUST be placed in `__tests__` directories adjacent to the components they test, following the pattern `packages/core/components/[Component]/__tests__/[test-file].spec.tsx`.
- **R-COMP-005** SHOULD: Structure tests with an outer describe block named after the component, inner describe blocks for feature areas if needed, and it blocks for individual test cases.
- **R-COMP-006** SHOULD: For server-side rendering tests, import from react-dom/server.node and verify HTML output rather than using client-side rendering utilities.

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
- Test files follow the naming convention `[test-file].spec.tsx` and are located in `__tests__` directories

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All component test files must be inspected for compliance with the describe/it structure before approval.
</enforcement>