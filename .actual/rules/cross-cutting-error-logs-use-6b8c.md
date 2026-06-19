# Log External API Failures to Console in Client-Side Components: Error Logs Use

These rules are ALWAYS ACTIVE for client-side React components in Next.js applications that make fetch calls to external APIs, particularly those using environment variables for runtime configuration.

### Rules

- **R-ERRLOG-001** SHOULD: Error logs SHOULD use template literals to interpolate error objects or messages for readability.

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
- Error logs use template literals to interpolate error objects or messages
- No fetch calls to external APIs exist without error handling

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules during code review and pull request analysis.
</enforcement>