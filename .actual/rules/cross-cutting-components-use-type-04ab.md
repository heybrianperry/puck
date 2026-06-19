# Adopt @/core as Standard React Component Library with Type-Safe Contracts: Components Use Type

These rules are ALWAYS ACTIVE for all React components in apps/demo/config/blocks, client-side components that interact with browser storage APIs, components that define public API contracts for external consumption, and shared utility functions and type definitions in @/core.

### Rules

- **R-CORE-001** SHOULD: Components SHOULD use type-safe imports from @/core/types for shared type definitions to maintain consistency.
- **R-CORE-002** MUST: All localStorage.getItem calls MUST be wrapped in try-catch blocks or use a helper function from @/core/lib that handles errors consistently.
- **R-CORE-003** SHOULD: Components SHOULD define explicit internal and public API contracts using naming convention: suffix Internal for implementation types, use plain name for public contracts.
- **R-CORE-004** MUST: TypeScript path mapping in tsconfig.json MUST resolve @/core alias to the core library root directory.
- **R-CORE-005** SHOULD: Shared utilities and types SHOULD be imported from @/core rather than using relative paths.

### Verify

```bash
# Check for @/core imports in component files
grep -r "from ['\"]@/core" apps/demo/config/blocks --include="*.tsx" --include="*.ts"

# Check for localStorage access without validation
grep -r "localStorage.getItem" apps/demo --include="*.tsx" | grep -v "JSON.parse"

# Verify TypeScript strict mode compilation
npx tsc --noEmit --strict && echo "Type checking passed"
```

**Accept when:**
- All components in apps/demo/config/blocks import from @/core for shared utilities and types
- No localStorage access occurs without JSON.parse or equivalent validation wrapper
- TypeScript compilation succeeds with strict mode enabled and no type errors in contract boundaries
- ESLint rules pass for import pattern enforcement in pre-commit hooks
- Security scanning finds no localStorage usage without validation

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline enforcement.
</enforcement>