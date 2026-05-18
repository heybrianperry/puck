<rule_activation id="3dbfeee0-f36a-42c0-bd6a-d5d51bb8ac08" title="Standardize Environment Variable Access via process.env for Runtime Configuration: Environment Variable Names" applies_to="**/*">
These rules are ALWAYS ACTIVE for all runtime configuration and environment management implementations across the codebase.
</rule_activation>

### Rules

- **R-ENV-001** SHOULD: Environment variable names SHOULD follow a consistent naming convention (e.g., SCREAMING_SNAKE_CASE with application prefix)

### Scope

**In scope:**
- All server-side runtime configuration (API endpoints, database URLs, service credentials)
- Feature flags and environment-specific behavior toggles
- Build-time configuration that varies across deployment targets
- Third-party service integration parameters (API keys, webhook URLs)
- Application-level settings (port numbers, timeouts, resource limits)

**Out of scope:**
- Static configuration that never changes across environments (e.g., application constants, algorithm parameters)
- Client-side only configuration in browser environments where process.env is not available
- Configuration loaded from external configuration services at runtime (though initial service URLs may still use process.env)
- Hardcoded values in test fixtures and mock data for unit tests

**Exceptions:**
- EXC-001: Client-side code in Next.js may access NEXT_PUBLIC_* prefixed environment variables that are explicitly intended for browser exposure
- EXC-002: Test environments may use hardcoded configuration values when testing configuration validation logic itself

### Verify

```bash
# Count process.env usage across TypeScript/JavaScript files
grep -r 'process\.env\.' --include='*.ts' --include='*.tsx' --include='*.js' --include='*.jsx' . | wc -l

# Find hardcoded URLs not using process.env (excluding comments)
grep -r 'const.*=.*['"']http' --include='*.ts' --include='*.tsx' | grep -v process.env | grep -v '//.*http' | wc -l

# Verify .env.example documentation exists
test -f .env.example && echo 'Configuration documentation exists' || echo 'Missing .env.example'
```

**Accept when:**
- All runtime configuration values are accessed via process.env rather than hardcoded strings or imported constants
- No hardcoded URLs, API keys, or environment-specific values appear in source code outside of test fixtures
- A .env.example file exists documenting all required environment variables with descriptions

<enforcement>
Claude Code MUST verify all three acceptance criteria before approving configuration-related changes. Automated scanning MUST flag hardcoded configuration values and sensitive patterns. Violations require tech lead approval and documented exception rationale.
</enforcement>