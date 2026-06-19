# Register Process Signal Handlers for Graceful Shutdown in Event-Driven Boundaries: Signal Handlers Execute

These rules are ALWAYS ACTIVE for Node.js CLI applications in packages/create-puck-app and similar long-running tools that perform file system operations, template compilation, or external process execution.

### Rules

- **R-SIGNAL-001** MUST: Signal handlers MUST execute cleanup logic before allowing process termination, including closing file handles, terminating child processes, and recording partial state.

### Verify

```bash
# Check for SIGINT handler registration
grep -r "process\.on.*SIGINT" packages/create-puck-app/ packages/core/

# Check for SIGTERM handler registration
grep -r "process\.on.*SIGTERM" packages/create-puck-app/ packages/core/

# Verify signal handlers are present in CLI entry points
node -e "const fs = require('fs'); const files = ['packages/create-puck-app/index.js']; files.forEach(f => { const content = fs.readFileSync(f, 'utf-8'); if (!content.includes('process.on') || (!content.includes('SIGINT') && !content.includes('SIGTERM'))) { process.exit(1); } });"
```

**Accept when:**
- All Node.js CLI applications that perform file system operations register handlers for both SIGINT and SIGTERM signals
- Signal handlers reference a cleanup function that executes before process termination
- Grep commands return matches in packages/create-puck-app/index.js showing process.on('SIGINT') and process.on('SIGTERM') patterns

<enforcement>
Claude Code MUST NOT skip or defer verification of signal handler registration in CLI applications. All long-running operations must have explicit cleanup coordination via process signal handlers.
</enforcement>