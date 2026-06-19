# Log External API Failures to Console in Client-Side Components: Client Side Components

These rules are ALWAYS ACTIVE for all client-side React components in Next.js applications that make fetch calls to external APIs at runtime.

### Rules

- **R-CSAPI-001** MUST: Client-side components that fetch data from external APIs MUST catch and log fetch errors using console.error

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
Claude Code MUST NOT skip or defer verification. All client-side fetch calls must be audited for proper error handling with console.error logging before accepting changes.
</enforcement>