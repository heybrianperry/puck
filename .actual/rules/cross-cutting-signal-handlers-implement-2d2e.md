# Register Process Signal Handlers for Graceful Shutdown in Event-Driven Boundaries: Signal Handlers Implement

These rules are ALWAYS ACTIVE for CLI applications performing file system mutations, tools executing external package managers (npm, yarn, pnpm), applications performing git operations via execSync, interactive applications using inquirer or similar prompt libraries, and long-running Node.js processes with stateful operations.

### Rules

- **R-SIG-001** MAY: Signal handlers MAY implement timeout mechanisms to force termination if graceful shutdown exceeds a defined threshold.
- **R-SIG-002** MUST: Register signal handlers early in application initialization before performing any stateful operations.
- **R-SIG-003** MUST: Use a shared handler function (e.g., handleSigTerm) for both SIGINT and SIGTERM to ensure consistent cleanup behavior.
- **R-SIG-004** MUST: Track resources requiring cleanup (file handles, child processes, temporary directories) in module-level state accessible to handlers.
- **R-SIG-005** MUST: Call process.exit() with appropriate exit code at the end of signal handlers to ensure process termination after cleanup.
- **R-SIG-006** MUST: Wrap signal handler logic in try-catch blocks to ensure process.exit() is called even on error paths.

### Verify

```bash
# Check for SIGINT handler registration
grep -r "process\.on.*SIGINT" --include="*.js" --include="*.ts" packages/

# Check for SIGTERM handler registration
grep -r "process\.on.*SIGTERM" --include="*.js" --include="*.ts" packages/

# Check for signal handler function definitions
grep -r "handleSigTerm\|handleSignal" --include="*.js" --include="*.ts" packages/
```

**Accept when:**
- All CLI applications and long-running Node.js processes register handlers for both SIGINT and SIGTERM signals
- Signal handlers complete cleanup of file system operations, child processes, and resources before calling process.exit()
- Grep commands identify signal handler registration in application entry points
- Signal handlers wrap cleanup logic in try-catch blocks
- process.exit() is called with appropriate exit code after cleanup completes

<enforcement>
Claude Code MUST NOT skip or defer verification of signal handler registration in CLI tools and stateful applications. All violations MUST be flagged during code review.
</enforcement>