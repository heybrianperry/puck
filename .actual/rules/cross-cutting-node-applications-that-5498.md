# Register Process Signal Handlers for Graceful Shutdown in Event-Driven Boundaries: Node Applications That

These rules are ALWAYS ACTIVE for Node.js CLI applications and library components that execute long-running operations including file system operations, external process execution, and interactive prompts.

### Rules

- **R-SIGNAL-001** MUST: Node.js applications that perform multi-step file system operations MUST register handlers for SIGINT and SIGTERM signals using process.on() to enable graceful shutdown.

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
- Signal handlers are registered early in application initialization before performing any stateful operations
- A shared handler function is used for both SIGINT and SIGTERM to ensure consistent cleanup behavior

<enforcement>
Clause Code MUST NOT skip or defer verification of signal handler registration in CLI applications and stateful Node.js processes. Violations must be flagged during code review and CI pipeline checks.
</enforcement>