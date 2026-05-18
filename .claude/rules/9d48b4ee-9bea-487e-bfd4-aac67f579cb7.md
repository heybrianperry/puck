<rule_activation id="9d48b4ee-9bea-487e-bfd4-aac67f579cb7" title="Adopt Remix Route-Based Configuration Pattern for Service Boundaries: Routes Define Additional" applies_to="**/*">
These rules are ALWAYS ACTIVE for all Remix route modules and service boundary definitions.
</rule_activation>

### Rules

- **R-RBC-001** MAY: Routes MAY define additional service boundaries for specialized operations (e.g., AI services, external APIs) as needed for specific application variants

### Scope

**In scope:**
- All Remix route modules (app/routes/**/*.tsx)
- Loader and action functions within route modules
- Service configuration objects defined at route level
- Environment variable access within route boundaries

**Out of scope:**
- Shared utility functions outside route modules
- Global application configuration (root.tsx)
- Build-time configuration and bundler settings
- Static asset configuration

**Exceptions:**
- EXC-001: Legacy routes being migrated to the new pattern
- EXC-002: Third-party library integration requires global configuration access

### Verify

```bash
# Verify all route modules with service dependencies define configuration objects
grep -r 'export.*loader' app/routes/ | xargs -I {} sh -c 'grep -L "config" {} || echo "Missing config in: {}"'

# Find direct process.env access outside loaders/actions
find app/routes -name '*.tsx' -exec grep -l 'process\.env\.' {} \; | grep -v 'loader\|action'

# Run TypeScript strict mode and linting checks
npm run type-check && npm run lint -- --rule 'no-process-env: error' app/routes/
```

**Accept when:**
- All route modules with service dependencies define configuration objects within loader or action functions
- No direct process.env access exists in route component code (only in loaders/actions)
- TypeScript compilation succeeds with strict mode enabled for all route modules
- Code review checklist confirms service boundaries are explicitly defined for new routes

### Enforcement

- **Verified by:** ESLint rules preventing direct environment variable access in components
- **Verified by:** TypeScript strict mode compilation checks
- **Verified by:** Code review checklist items for route module structure
- **Verified by:** Automated tests verifying configuration isolation between routes
- **Violation handling:** CI pipeline fails if ESLint rules are violated
- **Violation handling:** Pull requests blocked until code review checklist is completed
- **Violation handling:** Architecture review required for any exceptions to the pattern
- **Violation handling:** Quarterly audits of route modules to identify and remediate violations
- **Exception process:** Developer documents exception rationale in route module comments
- **Exception process:** Tech lead reviews and approves exception with documented justification
- **Exception process:** Exception is logged in architecture decision log with timeline for remediation
- **Exception process:** Exceptions are reviewed quarterly and must be re-approved or remediated

<enforcement>
Claude Code MUST NOT skip or defer verification of route-based service boundary definitions. All verify commands MUST pass before accepting changes to Remix route modules.
</enforcement>