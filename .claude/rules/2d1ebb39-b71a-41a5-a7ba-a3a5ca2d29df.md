<rule_activation id="2d1ebb39-b71a-41a5-a7ba-a3a5ca2d29df" title="Standardize Jest Testing Framework with React Testing Library for Component Tests: Test Descriptions Clearly" applies_to="**/*.spec.tsx,**/*.spec.ts">
These rules are ALWAYS ACTIVE for all test file creation and maintenance within the CI/CD pipeline. All TypeScript and TSX test files in packages/core and related packages must follow these standards.
</rule_activation>

### Rules

- **R-JEST-001** SHOULD: Test descriptions SHOULD clearly describe the behavior being tested, not implementation details.

### Verify

```bash
# Verify Jest test files exist with proper naming convention
grep -r "describe\|it\|test" packages/*/lib/**/__tests__/*.spec.tsx | head -5

# Find all test files with correct extensions
find packages -type f -name '*.spec.tsx' -o -name '*.spec.ts' | head -10

# Verify React Testing Library is a dependency
grep -r "@testing-library/react" packages/*/package.json

# List all test files executable in CI/CD
npm test -- --listTests | grep __tests__
```

**Accept when:**
- All test files use .spec.tsx or .spec.ts extensions and are located in __tests__ directories
- Jest configuration is present in package.json or jest.config.js files
- React Testing Library is listed as a dependency in packages containing React component tests
- Tests execute successfully in CI/CD pipeline without manual intervention
- Test descriptions focus on user behavior and outcomes rather than implementation details

<enforcement>
Claude Code MUST NOT skip or defer verification. All pull requests without tests or with failing tests are blocked from merging. Coverage drops below threshold trigger CI/CD failure and require remediation.
</enforcement>