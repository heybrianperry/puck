# Use console.error for CLI Error Reporting in Node.js Applications: Cli Applications Use

These rules are ALWAYS ACTIVE for all files matching the configured scope: CLI command handlers in packages/create-puck-app, input validation routines, file system operation error handling, browser-side localStorage access in React hooks (packages/core), and git operation failures during repository initialization.

### Rules

- **R-CLI-001** MUST: CLI applications MUST use console.error for all user-facing error messages, validation failures, and exceptional conditions.

### Verify

```bash
# Count console.error usage across CLI and core packages
grep -r 'console\.error' packages/create-puck-app packages/core --include='*.js' --include='*.ts' | wc -l

# Verify no console.log or console.warn are used for error reporting
grep -r 'console\.log.*error\|console\.warn.*error' packages/create-puck-app packages/core --include='*.js' --include='*.ts' | wc -l

# Test CLI error scenarios to verify stderr output
node -e "const { execSync } = require('child_process'); try { execSync('cd test-app && node cli.js 2>&1 >/dev/null'); } catch(e) { process.exit(e.status === 1 ? 0 : 1); }"
```

**Accept when:**
- All user-facing error messages in CLI handlers use console.error rather than console.log or console.warn
- Error output can be redirected to stderr independently of stdout using standard shell redirection
- localStorage access failures in React hooks are caught and logged with console.error including operation context

<enforcement>
Claude Code MUST NOT skip or defer verification. All console.error usage must be confirmed through code review and manual testing of CLI error scenarios.
</enforcement>