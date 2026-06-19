# Separate Internal and Public API Contracts for Component Interfaces: Components Use Runtime

These rules are ALWAYS ACTIVE for React components in apps/demo/config/blocks that expose public APIs, factory functions and component wrappers that transform external data to internal state, modules importing from @/core that bridge application and framework boundaries, and any code path that parses untrusted data (localStorage, URL params, API responses).

### Rules

- **R-CONTRACT-001** MAY: Components MAY use runtime validation libraries (Zod, io-ts) to enforce contract boundaries at parse time.
- **R-CONTRACT-002** SHOULD: Define internal contracts with 'Internal' suffix (e.g., TemplateInternal) and public contracts without suffix (e.g., Template) in the same module for discoverability.
- **R-CONTRACT-003** SHOULD: Implement transformation functions (e.g., createComponent) that validate external data against the public contract schema before constructing internal contract instances.
- **R-CONTRACT-004** SHOULD: Use TypeScript's 'Omit', 'Pick', or 'Partial' utility types to derive public contracts from internal ones when appropriate, reducing duplication.
- **R-CONTRACT-005** SHOULD: Add ESLint rules to prevent direct imports of internal contracts from outside the module boundary.
- **R-CONTRACT-006** MUST: Ensure all JSON.parse calls processing untrusted data are followed by validation logic within 5 lines.

### Verify

```bash
# Count components exporting internal contracts
grep -r 'export.*Internal' apps/demo/config/blocks --include='*.tsx' --include='*.ts' | wc -l

# Check for validation following JSON.parse on untrusted data
grep -r 'JSON.parse.*localStorage' apps/demo --include='*.tsx' --include='*.ts' -A 5 | grep -c 'validate\|parse\|schema'

# Verify TypeScript strict mode compilation
npx tsc --noEmit --strict && echo 'Type checking passed'
```

**Accept when:**
- At least one component in apps/demo/config/blocks exports both an internal contract (suffixed with 'Internal') and a public contract
- All JSON.parse calls processing untrusted data are followed by validation logic within 5 lines
- TypeScript strict mode compilation passes without errors related to contract type mismatches

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler in strict mode during CI builds, code review checklist requiring contract separation for components with external APIs, and ESLint rules enforcing import restrictions on internal contracts are mandatory. CI build fails on TypeScript compilation errors related to contract violations. Pull requests are blocked until code review approves contract boundary design. ESLint violations are reported as errors in pre-commit hooks.
</enforcement>