# Establish Dual-Interface Pattern for Public API Contracts in React Components: Public Contracts Defined

These rules are ALWAYS ACTIVE for all React components in `apps/demo/config/blocks/` that are consumed by external modules, use the `createComponent` factory pattern, expose public APIs through barrel exports, or integrate with shared state management.

### Rules

- **R-DUAL-001** MUST: Public API contracts MUST be defined as explicit TypeScript interfaces or types, not inferred from implementation.
- **R-DUAL-002** MUST: Use naming convention `ComponentNameInternal` for internal interfaces and `ComponentName` for public interfaces.
- **R-DUAL-003** MUST: Export only the public interface from barrel exports (`index.ts`) while keeping internal interfaces accessible via direct imports for testing.
- **R-DUAL-004** MUST: When using `createComponent` factory, pass the internal interface as the generic type parameter and return the public interface.
- **R-DUAL-005** SHOULD: Document the distinction between interfaces in JSDoc comments, explaining what each interface is intended for.
- **R-DUAL-006** SHOULD: Ensure internal implementation details are not exposed through public interfaces.

### Verify

```bash
# Count exported Internal interfaces in blocks directory
grep -r "export.*Internal" apps/demo/config/blocks/ | wc -l

# Find components using api.public.contracts pattern
find apps/demo/config/blocks -name '*.tsx' -exec grep -l 'api.public.contracts' {} \;

# Verify TypeScript strict mode compilation
npx tsc --noEmit --strict && echo 'Type checking passed'
```

**Accept when:**
- All public-facing React components in `apps/demo/config/blocks/` export both `Internal` and public interfaces
- TypeScript compilation succeeds with strict mode enabled, confirming type safety across interface boundaries
- Code review confirms that internal implementation details are not exposed through public interfaces
- Public interface is verified as a proper subset of internal interface through automated tests

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler strict mode checks MUST pass during CI build. ESLint custom rules and code review checklists MUST validate dual-interface pattern compliance. Pull requests MUST be blocked until pattern is applied to public components.
</enforcement>