# Log External API Failures to Console in Client-Side Components: Error Log Messages

These rules are ALWAYS ACTIVE for all client-side React components in Next.js applications that make fetch calls to external APIs at runtime.

### Rules

- **R-APIERR-001** MUST: Error log messages MUST include contextual information identifying the failed operation and error details.

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

<enforcement>
Claude Code MUST NOT skip or defer verification. All fetch calls in client-side components must include error handling with contextual console.error logging.
</enforcement>