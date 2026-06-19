# Standardize React Component Exports as Public API Contracts in Next.js Applications: Component Props Interfaces

These rules are ALWAYS ACTIVE for React functional components exported from apps/docs and apps/demo, TypeScript Props interfaces for all public components, Next.js application-level exports (middleware, _app, page components), block-based UI components (Stats, Flex, Template, Hero, Card, Grid, Heading, Text), and shared component utilities imported from @/core.

### Rules

- **R-COMP-001** MUST: Component Props interfaces MUST follow the naming convention {ComponentName}Props

### Verify

```bash
# Count exported Props interfaces
grep -r "export.*Props" apps/docs apps/demo --include="*.tsx" | wc -l

# Check for disallowed default exports (excluding Next.js convention files)
grep -r "export default" apps/docs apps/demo --include="*.tsx" --exclude="_app.tsx" --exclude="middleware.ts" --exclude="page.tsx" | wc -l

# Verify TypeScript compilation succeeds
npx tsc --noEmit --project tsconfig.json
```

**Accept when:**
- All component files export at least one Props interface matching the pattern {ComponentName}Props
- Default exports are limited to Next.js convention files (_app.tsx, middleware.ts, page.tsx)
- TypeScript compilation succeeds without type errors related to component prop contracts

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands must pass before accepting changes to component exports.
</enforcement>