# Separate Internal and Public API Contracts for Component Interfaces: Internal Contracts Suffixed

These rules are ALWAYS ACTIVE for React components in apps/demo/config/blocks that expose public APIs, factory functions and component wrappers that transform external data to internal state, modules importing from @/core that bridge application and framework boundaries, and any code path that parses untrusted data (localStorage, URL params, API responses).

### Rules

- **R-CONTRACT-001** SHOULD: Internal contracts SHOULD be suffixed with 'Internal' or placed in separate internal modules to signal visibility boundaries.
- **R-CONTRACT-002** MUST: Transformation functions (e.g., createComponent) MUST validate external data against the public contract schema before constructing internal contract instances.
- **R-CONTRACT-003** SHOULD: Use TypeScript's 'Omit', 'Pick', or 'Partial' utility types to derive public contracts from internal ones when appropriate, reducing duplication.
- **R-CONTRACT-004** MUST: All JSON.parse calls processing untrusted data MUST be followed by validation logic within 5 lines.
- **R-CONTRACT-005** MUST: ESLint rules MUST prevent direct imports of internal contracts from outside the module boundary.

### Verify

```bash
# Count internal contracts exported
grep -r 'export.*Internal' apps/demo/config/blocks --include='*.tsx' --include='*.ts' | wc -l

# Verify JSON.parse calls have validation nearby
grep -r 'JSON.parse.*localStorage' apps/demo --include='*.tsx' --include='*.ts' -A 5 | grep -c 'validate\|parse\|schema'

# Type checking in strict mode
npx tsc --noEmit --strict && echo 'Type checking passed'
```

**Accept when:**
- At least one component in apps/demo/config/blocks exports both an internal contract (suffixed with 'Internal') and a public contract
- All JSON.parse calls processing untrusted data are followed by validation logic within 5 lines
- TypeScript strict mode compilation passes without errors related to contract type mismatches

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes to components with external APIs or untrusted data handling.
</enforcement>