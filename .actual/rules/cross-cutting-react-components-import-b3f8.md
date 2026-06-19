# Adopt @/core as Standard React Component Library with Type-Safe Contracts: React Components Import

These rules are ALWAYS ACTIVE for all React components in apps/demo/config/blocks, client-side components that interact with browser storage APIs, components that define public API contracts for external consumption, and shared utility functions and type definitions in @/core.

### Rules

- **R-CORE-001** MUST: All React components MUST import core functionality from @/core path alias rather than relative paths outside the local module.

### Verify

```bash
# Verify all components in scope use @/core imports
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
Claude Code MUST NOT skip or defer verification. ESLint rules checking import patterns are enforced in pre-commit hooks. TypeScript strict mode compilation is verified in CI pipeline. Code review checklist requires contract type verification for new components. Automated security scanning flags localStorage usage patterns. Violations block CI builds and pull requests until resolved.
</enforcement>