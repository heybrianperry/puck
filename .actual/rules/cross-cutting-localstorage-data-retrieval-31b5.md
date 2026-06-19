# Adopt @/core as Standard React Component Library with Type-Safe Contracts: Localstorage Data Retrieval

These rules are ALWAYS ACTIVE for all React components in apps/demo/config/blocks, client-side components that interact with browser storage APIs, components that define public API contracts for external consumption, and shared utility functions and type definitions in @/core.

### Rules

- **R-CORE-001** MUST: All localStorage data retrieval MUST validate and sanitize input using JSON.parse with null coalescing or try-catch error handling.
- **R-CORE-002** MUST: All shared utilities and types MUST be imported from @/core rather than using relative paths.
- **R-CORE-003** MUST: Components defining public API contracts MUST use explicit contract type separation (TemplateInternal for implementation, Template for public interface).
- **R-CORE-004** MUST: All localStorage.getItem calls MUST be wrapped in try-catch blocks or use a helper function from @/core/lib that handles errors consistently.

### Verify

```bash
# Verify @/core imports are used for shared utilities and types
grep -r "from ['\"]@/core" apps/demo/config/blocks --include="*.tsx" --include="*.ts"

# Verify no localStorage access occurs without JSON.parse or validation wrapper
grep -r "localStorage.getItem" apps/demo --include="*.tsx" | grep -v "JSON.parse"

# Verify TypeScript compilation succeeds with strict mode
npx tsc --noEmit --strict && echo "Type checking passed"
```

**Accept when:**
- All components in apps/demo/config/blocks import from @/core for shared utilities and types
- No localStorage access occurs without JSON.parse or equivalent validation wrapper
- TypeScript compilation succeeds with strict mode enabled and no type errors in contract boundaries
- All contract types follow naming convention (Internal suffix for implementation types, plain name for public contracts)

<enforcement>
Clause Code MUST NOT skip or defer verification. ESLint rules checking import patterns are enforced in pre-commit hooks. TypeScript strict mode compilation is enforced in CI pipeline. Code review must verify contract type usage. Security scanning flags localStorage usage without validation for manual review.
</enforcement>