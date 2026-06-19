# Adopt @/core as Standard React Component Library with Type-Safe Contracts: Utility Functions Generation

These rules are ALWAYS ACTIVE for all React components in apps/demo/config/blocks, client-side components that interact with browser storage APIs, components that define public API contracts for external consumption, and shared utility functions and type definitions in @/core.

### Rules

- **R-CORE-001** SHOULD: Utility functions for ID generation and common operations SHOULD be imported from @/core/lib rather than duplicated.
- **R-CORE-002** MUST: All localStorage.getItem calls MUST be wrapped in try-catch blocks or use a helper function from @/core/lib that handles errors consistently.
- **R-CORE-003** MUST: Components MUST define explicit internal and public API contracts (TemplateInternal, Template) to separate implementation from interface.
- **R-CORE-004** SHOULD: Path alias imports (@/core) SHOULD be preferred over relative imports for shared utilities and types.
- **R-CORE-005** MUST: TypeScript strict mode MUST be enabled and compilation MUST succeed with no type errors in contract boundaries.

### Verify

```bash
# Verify @/core imports are used for shared utilities
grep -r "from ['\"]@/core" apps/demo/config/blocks --include="*.tsx" --include="*.ts"

# Verify localStorage access includes validation
grep -r "localStorage.getItem" apps/demo --include="*.tsx" | grep -v "JSON.parse"

# Verify TypeScript strict mode compilation
npx tsc --noEmit --strict && echo "Type checking passed"
```

**Accept when:**
- All components in apps/demo/config/blocks import from @/core for shared utilities and types
- No localStorage access occurs without JSON.parse or equivalent validation wrapper
- TypeScript compilation succeeds with strict mode enabled and no type errors in contract boundaries
- Contract types follow naming convention: suffix Internal for implementation types, plain name for public contracts
- All localStorage.getItem calls are wrapped in try-catch blocks with fallback to safe defaults

<enforcement>
Claude Code MUST NOT skip or defer verification. ESLint rules checking import patterns MUST be enforced in pre-commit hooks. TypeScript strict mode compilation MUST pass in CI pipeline. Code review MUST verify contract type separation for new components. Security scanning MUST flag localStorage usage without validation for manual review. CI build MUST fail on violations. Pull requests MUST be blocked until TypeScript strict mode passes.
</enforcement>