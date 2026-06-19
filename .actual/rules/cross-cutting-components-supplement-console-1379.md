# Log External API Failures to Console in Client-Side Components: Components Supplement Console

These rules are ALWAYS ACTIVE for all client-side React components in Next.js applications that make external API calls using the fetch API.

### Rules

- **R-CONSOLE-001** MUST: Wrap all fetch calls in client-side components with try-catch blocks that include error handling.
- **R-CONSOLE-002** MUST: Use `console.error()` (not `console.log()`) to log caught fetch errors, ensuring errors are visually distinct in browser developer tools.
- **R-CONSOLE-003** MUST: Include descriptive operation context in error log messages (e.g., 'Could not load releases:') followed by the error object for diagnostic information.
- **R-CONSOLE-004** MAY: Components MAY supplement console logging with user-facing error states or fallback UI to improve user experience.
- **R-CONSOLE-005** MUST: Avoid logging sensitive information (API keys, tokens, full URLs with secrets) in error messages; log only safe contextual information.

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
- Sensitive information is not included in error log messages

<enforcement>
Claude Code MUST NOT skip or defer verification. All fetch calls must be audited for proper error handling before accepting changes.
</enforcement>