# Adopt @/core as Standard React Component Library with Type-Safe Contracts: Components That Expose

These rules are ALWAYS ACTIVE for all React components in apps/demo/config/blocks, client-side components that interact with browser storage APIs, components that define public API contracts for external consumption, and shared utility functions and type definitions in @/core.

### Rules

- **R-CORE-001** MUST: Components that expose public APIs MUST define separate internal and public contract types to enforce interface boundaries.

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

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verification commands must pass before accepting changes to components in scope.
</enforcement>