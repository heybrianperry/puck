# Standardize on describe/it Test Structure for Component Testing: Test Files Organize

These rules are ALWAYS ACTIVE for all component test files in `packages/core/components/**/__tests__/` directories that use React component unit and integration tests with @testing-library/react.

### Rules

- **R-TEST-001** MUST: Organize test files using describe blocks to group related test cases by component or feature area.
- **R-TEST-002** MUST: Define each individual test case using an it block with a descriptive test name.
- **R-TEST-003** MUST: Import @testing-library/react and @testing-library/jest-dom at the top of each component test file.
- **R-TEST-004** MAY: Organize multiple describe blocks hierarchically to represent component structure or feature groupings.
- **R-TEST-005** SHOULD: Place test files in `__tests__` directories adjacent to the components they test, following the pattern `packages/core/components/[Component]/__tests__/[test-file].spec.tsx`.
- **R-TEST-006** SHOULD: Use render/screen/fireEvent from @testing-library/react for component rendering and user interaction simulation.
- **R-TEST-007** SHOULD: For server-side rendering tests, import from react-dom/server.node and verify HTML output rather than using client-side rendering utilities.

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

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules when reviewing or creating component test files.
</enforcement>