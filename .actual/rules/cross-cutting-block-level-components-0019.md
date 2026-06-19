# Standardize React Component Exports as Public API Contracts in Next.js Applications: Block Level Components

These rules are ALWAYS ACTIVE for React functional components exported from apps/docs and apps/demo, TypeScript Props interfaces for all public components, Next.js application-level exports (middleware, _app, page components), block-based UI components (Stats, Flex, Template, Hero, Card, Grid, Heading, Text), and shared component utilities imported from @/core.

### Rules

- **R-BLOCK-001** SHOULD: Block-level components SHOULD co-locate styles using CSS modules with the naming pattern `styles.module.css`.
- **R-BLOCK-002** SHOULD: All block-level components SHOULD export a named TypeScript Props interface matching the pattern `{ComponentName}Props`.
- **R-BLOCK-003** SHOULD: Components SHOULD use named exports for both Props interfaces and component functions to enable tree-shaking and explicit dependency tracking.
- **R-BLOCK-004** MUST: Default exports are MUST be limited to Next.js convention files (_app.tsx, middleware.ts, page.tsx) only.
- **R-BLOCK-005** SHOULD: Props interfaces SHOULD be exported alongside component implementations to serve as machine-readable API documentation.

### Verify

```bash
# Count Props interface exports
grep -r "export.*Props" apps/docs apps/demo --include="*.tsx" | wc -l

# Check for non-compliant default exports (excluding Next.js convention files)
grep -r "export default" apps/docs apps/demo --include="*.tsx" --exclude="_app.tsx" --exclude="middleware.ts" --exclude="page.tsx" | wc -l

# Verify TypeScript compilation succeeds
npx tsc --noEmit --project tsconfig.json

# Verify styles.module.css co-location in block components
find apps/docs apps/demo -name "styles.module.css" | wc -l
```

**Accept when:**
- All component files export at least one Props interface matching the pattern `{ComponentName}Props`
- Default exports are limited to Next.js convention files (_app.tsx, middleware.ts, page.tsx)
- TypeScript compilation succeeds without type errors related to component prop contracts
- Block-level components co-locate styles using the `styles.module.css` naming pattern
- Named exports are used for both Props interfaces and component functions

<enforcement>
Claude Code MUST NOT skip or defer verification. ESLint rules in CI pipeline MUST check for named exports and Props interface naming conventions. TypeScript strict mode compilation MUST succeed in pre-commit hooks. Automated API documentation generation MUST fail if Props interfaces are missing. CI build MUST fail if components lack Props interface exports or use default exports outside allowed files.
</enforcement>