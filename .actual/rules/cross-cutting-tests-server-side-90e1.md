# Standardize on describe/it Test Structure for Component Testing: Tests Server Side

These rules are ALWAYS ACTIVE for all component test files in `packages/core/components/**/__tests__/` directories that verify component rendering, state updates, user interactions, and server-side rendering scenarios.

### Rules

- **R-TEST-001** SHOULD: Tests for server-side rendering scenarios SHOULD use `react-dom/server.node` for RSC mode verification.
- **R-TEST-002** MUST: All component test files MUST use `describe` blocks to organize tests hierarchically by component and feature area.
- **R-TEST-003** MUST: Each individual test case MUST be defined with an `it` block containing a descriptive test name.
- **R-TEST-004** MUST: All component test files MUST import `@testing-library/react` for render/screen/fireEvent utilities.
- **R-TEST-005** MUST: All component test files MUST import `@testing-library/jest-dom` for DOM matchers.
- **R-TEST-006** SHOULD: Test files SHOULD be placed in `__tests__` directories adjacent to the components they test, following the pattern `packages/core/components/[Component]/__tests__/[test-file].spec.tsx`.

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
- All component test files in `packages/core/components/**/__tests__/` use `describe` blocks to organize tests
- Each test case is defined with an `it` block containing a descriptive test name
- `@testing-library/react` and `@testing-library/jest-dom` are imported in all component test files
- Server-side rendering tests use `react-dom/server.node` and verify HTML output

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules during code review and test file creation.
</enforcement>