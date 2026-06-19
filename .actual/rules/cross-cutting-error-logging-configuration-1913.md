# Isolate Environment Variables for Public Client Configuration in React Components: Error Logging Configuration

These rules are ALWAYS ACTIVE for all client-side React components in Next.js applications that access runtime configuration through environment variables.

### Rules

- **R-ENV-001** SHOULD: Error logging for configuration-related failures SHOULD use console.error with descriptive messages that aid debugging without exposing sensitive values.

### Verify

```bash
# Verify all process.env accesses in client components use NEXT_PUBLIC_ prefix
grep -r "process\.env\.NEXT_PUBLIC_" apps/docs/components/ | wc -l

# Verify no server-only environment variables are accessed from client components
grep -r "process\.env\." apps/docs/components/ | grep -v "NEXT_PUBLIC_" | grep -v "node_modules" || echo 'No violations found'

# Run linting checks for environment variable access patterns
npm run lint -- --rule 'no-process-env: error' || echo 'Linting check complete'
```

**Accept when:**
- All process.env accesses in client components use the NEXT_PUBLIC_ prefix
- No server-only environment variables (without NEXT_PUBLIC_ prefix) are accessed from client component code
- Grep for process.env in components directory returns only NEXT_PUBLIC_ prefixed variables or returns zero non-compliant matches
- Error logging uses console.error with descriptive messages that do not expose sensitive configuration values

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via grep or ESLint scanning for process.env usage patterns in client component files is mandatory. Code review must verify environment variable access follows NEXT_PUBLIC_ convention. CI pipeline checks must scan for secret patterns combined with NEXT_PUBLIC_ prefix.
</enforcement>