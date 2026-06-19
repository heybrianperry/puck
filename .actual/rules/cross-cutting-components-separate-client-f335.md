# Standardize React Component Exports as Public API Contracts in Next.js Applications: Components Separate Client

These rules are ALWAYS ACTIVE for React functional components, TypeScript Props interfaces, and Next.js application-level exports across apps/docs, apps/demo, and shared @/core libraries in monorepo architectures.

### Rules

- **R-COMP-001** MAY: Components MAY separate client and server implementations (client.tsx, server.tsx) while maintaining consistent public API exports.
- **R-COMP-002** MUST: All public React components MUST export a TypeScript Props interface matching the pattern `{ComponentName}Props`.
- **R-COMP-003** MUST: Components MUST use named exports for both Props interfaces and component functions to enable tree-shaking and explicit dependency tracking.
- **R-COMP-004** MUST: Default exports are permitted only in Next.js convention files (_app.tsx, middleware.ts, page.tsx); all other component files MUST use named exports.
- **R-COMP-005** SHOULD: Block-based UI components (Stats, Flex, Template, Hero, Card, Grid, Heading, Text) SHOULD maintain consistent Props interface exports across client and server implementations.
- **R-COMP-006** SHOULD: Component imports SHOULD originate from centralized style modules (styles.module.css) and shared utilities (@/core/lib, @/core/components) to establish consistent dependency structure.

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
- Components with client/server splits (client.tsx, server.tsx) export the same Props interface for API consistency

<enforcement>
Claude Code MUST NOT skip or defer verification. ESLint rules in CI pipeline MUST check for named exports and Props interface naming conventions. TypeScript strict mode compilation MUST succeed in pre-commit hooks. Automated API documentation generation MUST fail if Props interfaces are missing. CI build MUST fail if components lack Props interface exports or use default exports outside allowed files.
</enforcement>