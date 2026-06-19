# Use console.error for CLI Error Reporting in Node.js Applications: Error Messages Provide

These rules are ALWAYS ACTIVE for all files in CLI command handlers (packages/create-puck-app), input validation routines, file system operation error handling, browser-side localStorage access in React hooks (packages/core), and git operation failure handling.

### Rules

- **R-CC48-001** SHOULD: Error messages SHOULD provide actionable guidance (e.g., 'Please provide a name for your app' rather than generic error codes).
- **R-CC48-002** MUST: Use console.error for all user-facing error messages in CLI handlers and validation failures, ensuring errors are written to stderr.
- **R-CC48-003** MUST: Wrap localStorage access in try-catch blocks and log failures with console.error, including the operation context (e.g., 'Failed to load left sidebar width from localStorage').
- **R-CC48-004** SHOULD: Ensure error messages are user-friendly and actionable, avoiding technical jargon or stack traces in production.
- **R-CC48-005** MUST NOT: Log sensitive information (file paths, environment variables, secrets, tokens, or sensitive user data) in error messages.

### Verify

```bash
# Count console.error usage across CLI and core packages
grep -r 'console\.error' packages/create-puck-app packages/core --include='*.js' --include='*.ts' | wc -l

# Verify no console.log or console.warn used for error reporting
grep -r 'console\.log.*error\|console\.warn.*error' packages/create-puck-app packages/core --include='*.js' --include='*.ts' | wc -l

# Test CLI error scenarios to verify stderr output
node -e "const { execSync } = require('child_process'); try { execSync('cd test-app && node cli.js 2>&1 >/dev/null'); } catch(e) { process.exit(e.status === 1 ? 0 : 1); }"
```

**Accept when:**
- All user-facing error messages in CLI handlers use console.error rather than console.log or console.warn
- Error output can be redirected to stderr independently of stdout using standard shell redirection (e.g., `2>/dev/null` or `2>&1`)
- localStorage access failures in React hooks are caught and logged with console.error including operation context
- Error messages provide clear, actionable guidance on how to resolve the issue
- No sensitive information (secrets, tokens, file paths, environment variables) is exposed in error messages

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules during code review and testing.
</enforcement>