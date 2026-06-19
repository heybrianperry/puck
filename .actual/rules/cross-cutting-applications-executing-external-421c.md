# Register Process Signal Handlers for Graceful Shutdown in Event-Driven Boundaries: Applications Executing External

These rules are ALWAYS ACTIVE for CLI applications and long-running Node.js processes that perform multi-step file system operations, external process execution, or stateful operations requiring coordinated cleanup on termination.

### Rules

- **R-EX-001** SHOULD: Applications executing external processes via execSync or spawn SHOULD track child process handles and terminate them in signal handlers.

### Verify

```bash
# Check for SIGINT signal handler registration
grep -r "process\.on.*SIGINT" --include="*.js" --include="*.ts" packages/

# Check for SIGTERM signal handler registration
grep -r "process\.on.*SIGTERM" --include="*.js" --include="*.ts" packages/

# Check for common signal handler function names
grep -r "handleSigTerm\|handleSignal" --include="*.js" --include="*.ts" packages/
```

**Accept when:**
- All CLI applications and long-running Node.js processes register handlers for both SIGINT and SIGTERM signals
- Signal handlers complete cleanup of file system operations, child processes, and resources before calling process.exit()
- Grep commands identify signal handler registration in application entry points
- Signal handlers are registered early in application initialization before performing stateful operations
- A shared handler function is used for both SIGINT and SIGTERM to ensure consistent cleanup behavior

<enforcement>
Clause Code MUST NOT skip or defer verification of signal handler registration in CLI tools and stateful applications. Violations MUST be flagged in code review and CI pipeline checks.
</enforcement>