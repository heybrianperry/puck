# Use console.error for CLI Error Reporting in Node.js Applications: Applications Suppress Handle

These rules are ALWAYS ACTIVE for all files in CLI command handlers (packages/create-puck-app), input validation routines, file system operation error handling, browser-side localStorage access in React hooks (packages/core), and git operation failure handling.

### Rules

- **R-CONSOLE-001** MUST: Use console.error for all user-facing error messages in CLI command handlers, validation failures, and runtime exceptions.
- **R-CONSOLE-002** MUST: Write errors to stderr to enable proper Unix-style stream redirection and separation from normal program output.
- **R-CONSOLE-003** MUST: Wrap localStorage access in try-catch blocks and log failures with console.error, including the operation context.
- **R-CONSOLE-004** SHOULD: Ensure error messages are actionable and user-friendly, avoiding technical jargon or stack traces in production.
- **R-CONSOLE-005** MAY: Applications MAY suppress or handle console.error output differently in test environments or when specific error handling strategies are required.
- **R-CONSOLE-006** MUST NOT: Log sensitive information (file paths, environment variables, secrets, tokens) in error messages.

### Verify

```bash
# Count console.error usage across CLI and core packages
grep -r 'console\.error' packages/create-puck-app packages/core --include='*.js' --include='*.ts' | wc -l

# Verify no console.log or console.warn used for error reporting
grep -r 'console\.log.*error\|console\.warn.*error' packages/create-puck-app packages/core --include='*.js' --include='*.ts' | wc -l

# Test CLI error scenarios output to stderr
node -e "const { execSync } = require('child_process'); try { execSync('cd test-app && node cli.js 2>&1 >/dev/null'); } catch(e) { process.exit(e.status === 1 ? 0 : 1); }"
```

**Accept when:**
- All user-facing error messages in CLI handlers use console.error rather than console.log or console.warn
- Error output can be redirected to stderr independently of stdout using standard shell redirection (e.g., `2>/dev/null` or `2>&1`)
- localStorage access failures in React hooks are caught and logged with console.error including operation context
- No sensitive information (secrets, tokens, environment variables) is exposed in error messages
- Error messages are actionable and provide clear guidance on how to resolve the issue

<enforcement>
Claude Code MUST NOT skip or defer verification. All console.error usage must be reviewed for compliance with R-CONSOLE-001 through R-CONSOLE-006. Code review checklist, linting rules, and manual testing of CLI error scenarios are mandatory before acceptance.
</enforcement>