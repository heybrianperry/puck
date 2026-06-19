# Isolate Environment Variables for Public Client Configuration in React Components: Credentials Private Keys

These rules are ALWAYS ACTIVE for all client-side React components in Next.js applications that access runtime configuration through environment variables.

### Rules

- **R-CREDS-001** MUST: API credentials, private keys, and sensitive secrets MUST NOT be prefixed with NEXT_PUBLIC_ or accessed from client components.
- **R-CREDS-002** MUST: All process.env accesses in client components MUST use the NEXT_PUBLIC_ prefix.
- **R-CREDS-003** MUST: Server-only environment variables (without NEXT_PUBLIC_ prefix) MUST NOT be accessed from client component code.
- **R-CREDS-004** SHOULD: Document all NEXT_PUBLIC_ variables in a central README or .env.example file with descriptions of their purpose and expected values.
- **R-CREDS-005** SHOULD: Implement runtime guards that check for undefined environment variables and provide meaningful error messages or fallback values.

### Verify

```bash
# Check for all NEXT_PUBLIC_ environment variable accesses in client components
grep -r "process\.env\.NEXT_PUBLIC_" apps/docs/components/ | wc -l

# Check for any non-NEXT_PUBLIC_ process.env accesses in client components (should return no violations)
grep -r "process\.env\." apps/docs/components/ | grep -v "NEXT_PUBLIC_" | grep -v "node_modules" || echo 'No violations found'

# Run linting checks for environment variable access patterns
npm run lint -- --rule 'no-process-env: error' || echo 'Linting check complete'
```

**Accept when:**
- All process.env accesses in client components use the NEXT_PUBLIC_ prefix
- No server-only environment variables (without NEXT_PUBLIC_ prefix) are accessed from client component code
- Grep for process.env in components directory returns only NEXT_PUBLIC_ prefixed variables or returns zero non-compliant matches
- No environment variables matching secret patterns (API_KEY, SECRET, PASSWORD, TOKEN) are prefixed with NEXT_PUBLIC_

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via grep or ESLint scanning for process.env usage patterns in client component files is mandatory. Code review MUST verify environment variable access follows NEXT_PUBLIC_ convention. CI pipeline MUST fail if non-NEXT_PUBLIC_ process.env access is detected in client component files.
</enforcement>