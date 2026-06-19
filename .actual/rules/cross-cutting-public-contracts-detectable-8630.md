# Standardize React Component Exports as Public API Contracts in Next.js Applications: Public Contracts Detectable

These rules are ALWAYS ACTIVE for React components and TypeScript interfaces exported from Next.js applications in a monorepo structure, specifically targeting component files in apps/docs, apps/demo, and shared @/core libraries.

### Rules

- **R-PUB-001** MUST: Public API contracts MUST be detectable through static analysis of export statements and import declarations.
- **R-PUB-002** MUST: All public React functional components MUST export a corresponding TypeScript Props interface matching the pattern `{ComponentName}Props`.
- **R-PUB-003** MUST: Named exports for both Props interfaces and component functions MUST be used to enable tree-shaking and explicit dependency tracking.
- **R-PUB-004** MUST: Default exports are only permitted for Next.js convention files (_app.tsx, middleware.ts, page.tsx) and dynamic imports using next/dynamic.
- **R-PUB-005** SHOULD: Component Props interfaces SHOULD be exported alongside their corresponding component implementations in the same module.
- **R-PUB-006** SHOULD: Block-based UI components (Stats, Flex, Template, Hero, Card, Grid, Heading, Text) SHOULD follow consistent export patterns for API discoverability.
- **R-PUB-007** MAY: Internal utility components used only within a single block MAY omit Props interface exports if they are not consumed across module boundaries (EXC-002).

### Verify

```bash
# Count Props interface exports
grep -r "export.*Props" apps/docs apps/demo --include="*.tsx" | wc -l

# Identify non-compliant default exports (excluding Next.js convention files)
grep -r "export default" apps/docs apps/demo --include="*.tsx" --exclude="_app.tsx" --exclude="middleware.ts" --exclude="page.tsx" | wc -l

# Verify TypeScript compilation with strict mode
npx tsc --noEmit --project tsconfig.json

# Check for named exports in component files
grep -r "export.*function\|export.*const" apps/docs apps/demo --include="*.tsx" | grep -v "export default"
```

**Accept when:**
- All component files export at least one Props interface matching the pattern `{ComponentName}Props`
- Default exports are limited to Next.js convention files (_app.tsx, middleware.ts, page.tsx) and next/dynamic imports
- TypeScript compilation succeeds without type errors related to component prop contracts
- Named exports are used for both Props interfaces and component functions in all public component modules
- Static analysis can identify and trace component APIs through export statements

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for component export patterns in the configured scope. Violations MUST be flagged in CI pipeline checks and pull request reviews.
</enforcement>