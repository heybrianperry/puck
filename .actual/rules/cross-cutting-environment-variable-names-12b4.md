# Isolate Environment Variable Access to process.env for Secrets and Configuration: Environment Variable Names

These rules are ALWAYS ACTIVE for all files accessing runtime configuration and secrets through process.env, specifically in CLI tooling, core runtime components, and any code that requires environment-based configuration injection.

### Rules

- **R-ENV-001** SHOULD: Environment variable names SHOULD follow consistent naming conventions (e.g., npm_config_user_agent, NODE_ENV) across the codebase.

### Verify

```bash
# Identify all process.env accesses in the codebase
grep -r 'process\.env\.' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -v node_modules

# Verify specific environment variables are used consistently
grep -r 'npm_config_user_agent\|NODE_ENV' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -v node_modules

# Check for environment variable documentation
test -f .env.example && echo 'Environment variable documentation exists' || echo 'Missing .env.example'
```

**Accept when:**
- All process.env accesses are identified and documented in environment variable reference documentation
- No hardcoded secrets or credentials are found in source files
- Environment variable validation occurs at application startup with clear error messages for missing required variables
- Environment variable names follow consistent naming conventions across all packages

<enforcement>
Clause Code MUST NOT skip or defer verification. All process.env accesses must be reviewed for naming consistency and documentation completeness before merge.
</enforcement>