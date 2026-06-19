# Register Process Signal Handlers for Graceful Shutdown in Event-Driven Boundaries: Cli Applications That

These rules are ALWAYS ACTIVE for Node.js CLI applications in the codebase that perform long-running operations such as file system manipulation, template compilation, external process execution, or spawn child processes.

### Rules

- **R-CLI-SIG-001** SHOULD: CLI applications that spawn child processes via execSync or similar APIs SHOULD track process handles and terminate them during signal handling.

### Verify

```bash
# Check for SIGINT signal handler registration
grep -r "process\.on.*SIGINT" packages/create-puck-app/ packages/core/

# Check for SIGTERM signal handler registration
grep -r "process\.on.*SIGTERM" packages/create-puck-app/ packages/core/

# Verify signal handlers are present in CLI entry points
node -e "const fs = require('fs'); const files = ['packages/create-puck-app/index.js']; files.forEach(f => { const content = fs.readFileSync(f, 'utf-8'); if (!content.includes('process.on') || (!content.includes('SIGINT') && !content.includes('SIGTERM'))) { process.exit(1); } });"
```

**Accept when:**
- All Node.js CLI applications that perform file system operations register handlers for both SIGINT and SIGTERM signals
- Signal handlers reference a cleanup function that executes before process termination
- Grep commands return matches in packages/create-puck-app/index.js showing process.on('SIGINT') and process.on('SIGTERM') patterns
- Child processes spawned via execSync are tracked and terminated during cleanup
- Signal handlers complete synchronously or use process.exit() only after awaiting critical cleanup

<enforcement>
Claude Code MUST NOT skip or defer verification of signal handler registration in CLI applications. All verification commands MUST pass before accepting changes to CLI entry points or applications that spawn child processes.
</enforcement>