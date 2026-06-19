# Register Process Signal Handlers for Graceful Shutdown in Event-Driven Boundaries: Signal Handlers Log

These rules are ALWAYS ACTIVE for CLI applications performing file system mutations, tools executing external package managers (npm, yarn, pnpm), applications performing git operations via execSync, interactive applications using inquirer or similar prompt libraries, and long-running Node.js processes with stateful operations.

### Rules

- **R-SH-001** SHOULD: Signal handlers SHOULD log termination events to console.error or equivalent logging mechanism to provide visibility into shutdown sequences.

### Verify

```bash
# Verify signal handler registration for SIGINT
grep -r "process\.on.*SIGINT" --include="*.js" --include="*.ts" packages/

# Verify signal handler registration for SIGTERM
grep -r "process\.on.*SIGTERM" --include="*.js" --include="*.ts" packages/

# Verify common signal handler function names
grep -r "handleSigTerm\|handleSignal" --include="*.js" --include="*.ts" packages/
```

**Accept when:**
- All CLI applications and long-running Node.js processes register handlers for both SIGINT and SIGTERM signals
- Signal handlers complete cleanup of file system operations, child processes, and resources before calling process.exit()
- Signal handlers log termination events to console.error or equivalent logging mechanism
- Grep commands identify signal handler registration in application entry points

<enforcement>
Claude Code MUST NOT skip or defer verification. All CLI tools and stateful applications MUST register signal handlers with logging before merging to main branches.
</enforcement>