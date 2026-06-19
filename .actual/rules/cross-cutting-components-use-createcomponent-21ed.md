# Adopt @/core as Standard React Component Library with Type-Safe Contracts: Components Use Createcomponent

These rules are ALWAYS ACTIVE for all React components in apps/demo/config/blocks, client-side components that interact with browser storage APIs, components that define public API contracts for external consumption, and shared utility functions and type definitions in @/core.

### Rules

- **R-CORE-001** MAY: Components MAY use createComponent factory pattern for consistent component instantiation with dependency injection.
- **R-CORE-002** MUST: All components in scope import shared utilities and types from @/core rather than using relative paths.
- **R-CORE-003** MUST: Components that interact with localStorage MUST wrap all localStorage.getItem calls in try-catch blocks or use a helper function from @/core/lib that handles errors consistently.
- **R-CORE-004** MUST: Components defining public API contracts MUST use explicit contract type separation (suffix Internal for implementation types, use plain name for public contracts).
- **R-CORE-005** SHOULD: All localStorage access MUST include JSON.parse or equivalent validation wrapper to prevent injection attacks.
- **R-CORE-006** SHOULD: Contract types SHOULD be documented with JSDoc comments specifying stability guarantees and deprecation policy.

### Verify

```bash
# Verify @/core imports are used for shared utilities and types
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
- Contract types use consistent naming convention (Internal suffix for implementation, plain name for public)

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline enforcement.
</enforcement>