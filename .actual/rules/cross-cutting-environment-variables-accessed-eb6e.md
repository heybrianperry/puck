# Isolate Environment Variables for Public Client Configuration in React Components: Environment Variables Accessed

These rules are ALWAYS ACTIVE for all client-side React components in Next.js applications that access runtime configuration through environment variables.

### Rules

- **R-ENV-001** MUST: All environment variables accessed in client-side React components MUST use the NEXT_PUBLIC_ prefix to explicitly mark them as client-exposed.

### Verify

```bash
# Count NEXT_PUBLIC_ environment variable accesses in client components
grep -r "process\.env\.NEXT_PUBLIC_" apps/docs/components/ | wc -l

# Check for non-compliant process.env access (should return no violations)
grep -r "process\.env\." apps/docs/components/ | grep -v "NEXT_PUBLIC_" | grep -v "node_modules" || echo 'No violations found'

# Run linting checks for environment variable access patterns
npm run lint -- --rule 'no-process-env: error' || echo 'Linting check complete'
```

**Accept when:**
- All process.env accesses in client components use the NEXT_PUBLIC_ prefix
- No server-only environment variables (without NEXT_PUBLIC_ prefix) are accessed from client component code
- Grep for process.env in components directory returns only NEXT_PUBLIC_ prefixed variables or returns zero non-compliant matches

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via grep or ESLint scanning for process.env usage patterns in client component files is mandatory. Code review must verify environment variable access follows NEXT_PUBLIC_ convention. CI pipeline checks must scan for secret patterns combined with NEXT_PUBLIC_ prefix. Build fails if non-NEXT_PUBLIC_ process.env access is detected in client component files.
</enforcement>