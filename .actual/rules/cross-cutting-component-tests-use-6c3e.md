# Standardize on describe/it Test Structure for Component Testing: Component Tests Use

These rules are ALWAYS ACTIVE for all component test files in `packages/core/components/**/__tests__/` directories that test React components using @testing-library/react.

### Rules

- **R-COMP-001** MUST: Component tests MUST use @testing-library/react for rendering and interaction testing.
- **R-COMP-002** MUST: All component test files MUST use describe blocks to organize tests hierarchically by component and feature area.
- **R-COMP-003** MUST: Each individual test case MUST be defined with an it block containing a descriptive test name.
- **R-COMP-004** MUST: Test files MUST import @testing-library/react (render, screen, fireEvent) and @testing-library/jest-dom matchers at the top.
- **R-COMP-005** SHOULD: Place test files in `__tests__` directories adjacent to the components they test, following the pattern `packages/core/components/[Component]/__tests__/[test-file].spec.tsx`.
- **R-COMP-006** SHOULD: Structure tests with an outer describe block named after the component, inner describe blocks for feature areas if needed, and it blocks for individual test cases.
- **R-COMP-007** MAY: For server-side rendering tests, import from react-dom/server.node and verify HTML output rather than using client-side rendering utilities.

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
- All component test files in `packages/core/components/**/__tests__/` use describe blocks to organize tests.
- Each test case is defined with an it block containing a descriptive test name.
- @testing-library/react and @testing-library/jest-dom are imported in all component test files.
- Test files follow the naming convention `[test-file].spec.tsx` and are located in `__tests__` directories adjacent to components.

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules during code review and test file creation.
</enforcement>