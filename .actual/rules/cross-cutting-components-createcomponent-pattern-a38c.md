# Establish Dual-Interface Pattern for Public API Contracts in React Components: Components Createcomponent Pattern

These rules are ALWAYS ACTIVE for all React components in `apps/demo/config/blocks/` that use the `createComponent` factory pattern or are consumed by external modules.

### Rules

- **R-DUAL-001** MUST: Components using the createComponent pattern MUST define their contracts before implementation to ensure type safety across the factory boundary.
- **R-DUAL-002** MUST: All public-facing React components in `apps/demo/config/blocks/` MUST export both an Internal interface (named `ComponentNameInternal`) and a public interface (named `ComponentName`).
- **R-DUAL-003** MUST: Only the public interface MUST be exported from barrel exports (`index.ts`); internal interfaces MUST remain accessible via direct imports for testing purposes.
- **R-DUAL-004** MUST: When using the createComponent factory, the internal interface MUST be passed as the generic type parameter and the public interface MUST be returned.
- **R-DUAL-005** SHOULD: Internal implementation details MUST NOT be exposed through public interfaces; the public interface MUST be a proper subset of the internal interface.
- **R-DUAL-006** SHOULD: JSDoc comments MUST document the distinction between internal and public interfaces, explaining what each interface is intended for.
- **R-DUAL-007** MAY: Components explicitly marked as experimental or alpha with clear documentation MAY be exempted from this pattern (EXC-001).
- **R-DUAL-008** MAY: Components in rapid prototyping phase with no external consumers MAY be exempted from this pattern (EXC-002).

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
- All public-facing React components in `apps/demo/config/blocks/` export both Internal and public interfaces
- TypeScript compilation succeeds with strict mode enabled, confirming type safety across interface boundaries
- Code review confirms that internal implementation details are not exposed through public interfaces
- Public interfaces are verified as proper subsets of internal interfaces through automated tests

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler strict mode checks MUST pass during CI build. ESLint custom rules MUST detect public components without dual interfaces. Code review MUST verify API boundary separation. Pull requests MUST be blocked until dual-interface pattern is applied to public components.
</enforcement>