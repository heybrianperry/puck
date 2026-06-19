# Standardize React Component Exports as Public API Contracts in Next.js Applications: Components Import Framework

These rules are ALWAYS ACTIVE for React functional components, TypeScript Props interfaces, and Next.js application-level exports across apps/docs and apps/demo in monorepo structures using @/core shared utilities.

### Rules

- **R-COMP-001** MUST: Components MUST import framework dependencies (react, next/*) and shared utilities from @/core before local imports.

### Verify

```bash
# Count Props interface exports
grep -r "export.*Props" apps/docs apps/demo --include="*.tsx" | wc -l

# Count default exports (excluding Next.js convention files)
grep -r "export default" apps/docs apps/demo --include="*.tsx" --exclude="_app.tsx" --exclude="middleware.ts" | wc -l

# Verify TypeScript compilation
npx tsc --noEmit --project tsconfig.json
```

**Accept when:**
- All component files export at least one Props interface matching the pattern {ComponentName}Props
- Default exports are limited to Next.js convention files (_app.tsx, middleware.ts, page.tsx)
- TypeScript compilation succeeds without type errors related to component prop contracts
- Framework dependencies (react, next/*) appear before @/core imports, which appear before local imports in all component files

<enforcement>
Claude Code MUST NOT skip or defer verification. All component files must be checked for import ordering compliance and Props interface exports before accepting changes.
</enforcement>