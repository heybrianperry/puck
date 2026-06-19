# Isolate Environment Variable Access to process.env for Secrets and Configuration: Environment Variable Access

These rules are ALWAYS ACTIVE for all Node.js application code, CLI tools, and runtime components that require access to runtime configuration, secrets, or deployment-context-dependent behavior.

### Rules

- **R-ENV-001** MUST: Environment variable access MUST occur at runtime boundaries (application startup, component initialization) rather than at module load time.

### Verify

```bash
# Identify all process.env accesses in the codebase
grep -r 'process\.env\.' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -v node_modules

# Check for specific environment variables referenced in the ADR
grep -r 'npm_config_user_agent\|NODE_ENV' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -v node_modules

# Verify environment variable documentation exists
test -f .env.example && echo 'Environment variable documentation exists' || echo 'Missing .env.example'
```

**Accept when:**
- All process.env accesses are identified and documented in environment variable reference documentation
- No hardcoded secrets or credentials are found in source files
- Environment variable validation occurs at application startup with clear error messages for missing required variables
- Environment variables are accessed at runtime boundaries (component initialization, application startup) not during module evaluation

<enforcement>
Clause Code MUST NOT skip or defer verification. All process.env accesses must be reviewed for compliance with R-ENV-001. Pull requests containing hardcoded secrets are rejected. Missing environment variable documentation results in pull request comments requesting updates.
</enforcement>