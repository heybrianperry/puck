# Establish Dual-Interface Pattern for Public API Contracts in React Components: React Components That

These rules are ALWAYS ACTIVE for all React components in apps/demo/config/blocks/ that serve as public APIs and are consumed by external modules, use the createComponent factory pattern, or expose public API surfaces through barrel exports.

### Rules

- **R-DUAL-001** MUST: React components that serve as public APIs MUST export both an internal interface (suffixed with 'Internal') and a public interface for external consumption.
- **R-DUAL-002** MUST: Use naming convention ComponentNameInternal for internal interfaces and ComponentName for public interfaces to maintain consistency.
- **R-DUAL-003** MUST: Export only the public interface from barrel exports (index.ts) while keeping internal interfaces accessible via direct imports for testing.
- **R-DUAL-004** SHOULD: When using createComponent factory, pass the internal interface as the generic type parameter and return the public interface.
- **R-DUAL-005** SHOULD: Document the distinction between interfaces in JSDoc comments, explaining what each interface is intended for.
- **R-DUAL-006** MAY: Allow exceptions for components explicitly marked as experimental or alpha with clear documentation, or during rapid prototyping phase with no external consumers.

### Verify

```bash
# Count exported Internal interfaces in public component blocks
grep -r "export.*Internal" apps/demo/config/blocks/ | wc -l

# Find components using api.public.contracts pattern
find apps/demo/config/blocks -name '*.tsx' -exec grep -l 'api.public.contracts' {} \;

# Verify TypeScript compilation with strict mode
npx tsc --noEmit --strict && echo 'Type checking passed'
```

**Accept when:**
- All public-facing React components in apps/demo/config/blocks/ export both Internal and public interfaces
- TypeScript compilation succeeds with strict mode enabled, confirming type safety across interface boundaries
- Code review confirms that internal implementation details are not exposed through public interfaces
- Public interfaces are proper subsets of internal interfaces, verified through automated tests

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript strict mode compilation and interface boundary checks are mandatory before accepting changes to public API components.
</enforcement>