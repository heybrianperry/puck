# Register Process Signal Handlers for Graceful Shutdown in Event-Driven Boundaries: Signal Handlers Complete

These rules are ALWAYS ACTIVE for CLI applications performing file system mutations, tools executing external package managers (npm, yarn, pnpm), applications performing git operations via execSync, interactive applications using inquirer or similar prompt libraries, and long-running Node.js processes with stateful operations.

### Rules

- **R-SIG-001** MUST: Signal handlers MUST complete or abort in-progress operations before allowing process termination.

### Verify

```bash
# Check for SIGINT signal handler registration
grep -r "process\.on.*SIGINT" --include="*.js" --include="*.ts" packages/

# Check for SIGTERM signal handler registration
grep -r "process\.on.*SIGTERM" --include="*.js" --include="*.ts" packages/

# Check for signal handler function definitions
grep -r "handleSigTerm\|handleSignal" --include="*.js" --include="*.ts" packages/
```

**Accept when:**
- All CLI applications and long-running Node.js processes register handlers for both SIGINT and SIGTERM signals
- Signal handlers complete cleanup of file system operations, child processes, and resources before calling process.exit()
- Grep commands identify signal handler registration in application entry points

<enforcement>
Claude Code MUST NOT skip or defer verification. Signal handler registration is mandatory for all stateful CLI applications and long-running processes to prevent resource leaks and data corruption.
</enforcement>