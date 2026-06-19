# Standardize React Component Exports as Public API Contracts in Next.js Applications: Components That Serve

These rules are ALWAYS ACTIVE for React functional components, TypeScript Props interfaces, and Next.js application-level exports across apps/docs, apps/demo, and shared @/core libraries in monorepo architectures.

### Rules

- **R-COMP-001** SHOULD: Components that serve as application entry points (middleware, _app) SHOULD export Next.js-specific configuration objects alongside the primary export.
- **R-COMP-002** SHOULD: All public React components SHOULD export a named TypeScript Props interface matching the pattern `{ComponentName}Props`.
- **R-COMP-003** SHOULD: Block-based UI components (Stats, Flex, Template, Hero, Card, Grid, Heading, Text) SHOULD use named exports for both Props interfaces and component functions to enable tree-shaking and explicit dependency tracking.
- **R-COMP-004** MUST: Default exports MUST be limited to Next.js convention files (_app.tsx, middleware.ts, page.tsx) and MUST NOT be used for reusable component modules.
- **R-COMP-005** SHOULD: Components importing from centralized style modules and shared utilities (@/core/lib, @/core/components) SHOULD maintain consistent dependency structure and export patterns across the monorepo.

### Verify

```bash
# Count Props interface exports
grep -r "export.*Props" apps/docs apps/demo --include="*.tsx" | wc -l

# Identify default exports outside allowed files
grep -r "export default" apps/docs apps/demo --include="*.tsx" --exclude="_app.tsx" --exclude="middleware.ts" --exclude="page.tsx" | wc -l

# Verify TypeScript compilation
npx tsc --noEmit --project tsconfig.json
```

**Accept when:**
- All component files export at least one Props interface matching the pattern `{ComponentName}Props`
- Default exports are limited to Next.js convention files (_app.tsx, middleware.ts, page.tsx)
- TypeScript compilation succeeds without type errors related to component prop contracts
- Named exports are used for both Props interfaces and component functions in reusable component modules

<enforcement>
Claude Code MUST NOT skip or defer verification. ESLint rules in CI pipeline MUST check for named exports and Props interface naming conventions. TypeScript strict mode compilation MUST succeed in pre-commit hooks. Automated API documentation generation MUST fail if Props interfaces are missing. CI build MUST fail if components lack Props interface exports or use default exports outside allowed files.
</enforcement>