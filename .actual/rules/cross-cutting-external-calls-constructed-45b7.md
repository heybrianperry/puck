# Log External API Failures to Console in Client-Side Components: External Calls Constructed

These rules are ALWAYS ACTIVE for client-side React components in Next.js applications that make external API calls using the fetch API, particularly those constructed from environment variables (NEXT_PUBLIC_BASE_URL, NEXT_PUBLIC_IS_CANARY, NEXT_PUBLIC_IS_LATEST).

### Rules

- **R-EX-001** SHOULD: External API calls constructed from environment variables SHOULD log the failure with sufficient context to identify misconfiguration.
- **R-EX-002** MUST: Wrap fetch calls in try-catch blocks within useEffect hooks or async functions.
- **R-EX-003** MUST: Use console.error (not console.log) for error logging to ensure errors are visually distinct in browser dev tools.
- **R-EX-004** MUST: Include operation context in error message template (e.g., 'Could not load releases:') followed by error interpolation.

### Verify

```bash
# Find all console.error statements in client-side components
grep -r "console\.error" apps/docs/components/ --include="*.tsx" --include="*.ts"

# Find fetch calls without catch blocks
grep -r "fetch(" apps/docs/components/ --include="*.tsx" --include="*.ts" | grep -v "catch"

# Find environment variable usage in API construction
grep -r "process\.env\.NEXT_PUBLIC" apps/docs/ --include="*.tsx" --include="*.ts"
```

**Accept when:**
- All fetch calls in client-side components have corresponding catch blocks with console.error statements
- Error log messages include descriptive context identifying the failed operation
- No fetch calls to external APIs exist without error handling
- Error logging does not include sensitive information (API keys, tokens, full URLs with secrets)

<enforcement>
Claude Code MUST NOT skip or defer verification. All fetch calls in client-side components must have error handling with console.error logging before code is accepted.
</enforcement>