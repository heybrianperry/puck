# Use console.error for CLI Error Reporting in Node.js Applications: Localstorage Access Failures

These rules are ALWAYS ACTIVE for all files in packages/create-puck-app and packages/core that handle CLI command execution, input validation, file system operations, browser-side localStorage access, and git operations.

### Rules

- **R-CONSOLE-001** SHOULD: localStorage access failures and browser-side errors SHOULD be logged using console.error with contextual information about the operation that failed.
- **R-CONSOLE-002** MUST: All user-facing error messages in CLI handlers use console.error rather than console.log or console.warn.
- **R-CONSOLE-003** SHOULD: Wrap localStorage access in try-catch blocks and log failures with console.error, including the operation context (e.g., 'Failed to load left sidebar width from localStorage').
- **R-CONSOLE-004** SHOULD: Ensure error messages are actionable and user-friendly, avoiding technical jargon or stack traces in production.
- **R-CONSOLE-005** SHOULD: Review all console.error calls to ensure no secrets, tokens, or sensitive user data are logged; sanitize paths and inputs before logging.

### Verify

```bash
# Count console.error usage across CLI and core packages
grep -r 'console\.error' packages/create-puck-app packages/core --include='*.js' --include='*.ts' | wc -l

# Check for console.log or console.warn used for error reporting (should be minimal)
grep -r 'console\.log.*error\|console\.warn.*error' packages/create-puck-app packages/core --include='*.js' --include='*.ts' | wc -l

# Test CLI error scenarios to verify stderr output
node -e "const { execSync } = require('child_process'); try { execSync('cd test-app && node cli.js 2>&1 >/dev/null'); } catch(e) { process.exit(e.status === 1 ? 0 : 1); }"
```

**Accept when:**
- All user-facing error messages in CLI handlers use console.error rather than console.log or console.warn
- Error output can be redirected to stderr independently of stdout using standard shell redirection
- localStorage access failures in React hooks are caught and logged with console.error including operation context
- No sensitive information (file paths, environment variables, tokens) is exposed in error messages

<enforcement>
Claude Code MUST NOT skip or defer verification. All console.error usage must be reviewed during code inspection to ensure compliance with R-CONSOLE-001 through R-CONSOLE-005. Violations must be flagged for remediation before merge.
</enforcement>