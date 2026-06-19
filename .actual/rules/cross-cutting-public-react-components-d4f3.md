# Standardize React Component Exports as Public API Contracts in Next.js Applications: Public React Components

These rules are ALWAYS ACTIVE for all React components exported from apps/docs, apps/demo, and shared @/core libraries in Next.js applications.

### Rules

- **R-EXPORT-001** MUST: All public React components MUST export both a TypeScript Props interface and the component function as named exports.
- **R-EXPORT-002** MUST: Props interfaces MUST follow the naming convention `{ComponentName}Props`.
- **R-EXPORT-003** MUST: Default exports are only permitted for Next.js convention files (_app.tsx, middleware.ts, page.tsx files).
- **R-EXPORT-004** SHOULD: Component files SHOULD import Props interfaces from centralized style modules and shared utilities (@/core/lib, @/core/components).
- **R-EXPORT-005** MAY: Dynamic imports using next/dynamic MAY defer component loading and are exempt from direct Props interface exports (EXC-001).
- **R-EXPORT-006** MAY: Internal utility components used only within a single block MAY omit Props interface exports (EXC-002).

### Verify

```bash
# Count Props interface exports
grep -r "export.*Props" apps/docs apps/demo --include="*.tsx" | wc -l

# Check for disallowed default exports (excluding Next.js convention files)
grep -r "export default" apps/docs apps/demo --include="*.tsx" --exclude="_app.tsx" --exclude="middleware.ts" --exclude="page.tsx" | wc -l

# Verify TypeScript compilation succeeds
npx tsc --noEmit --project tsconfig.json
```

**Accept when:**
- All component files export at least one Props interface matching the pattern `{ComponentName}Props`
- Default exports are limited to Next.js convention files (_app.tsx, middleware.ts, page.tsx)
- TypeScript compilation succeeds without type errors related to component prop contracts
- ESLint rules in CI pipeline confirm named exports and Props interface naming conventions
- No component files use default exports outside of allowed Next.js convention files

<enforcement>
Claude Code MUST NOT skip or defer verification. All component exports MUST be validated against R-EXPORT-001 through R-EXPORT-006 before accepting changes to component files.
</enforcement>