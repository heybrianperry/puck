# Register Process Signal Handlers for Graceful Shutdown in Event-Driven Boundaries: Node Processes That

These rules are ALWAYS ACTIVE for Node.js CLI applications that perform multi-step file system operations, template compilation, external process execution, or manage child processes.

### Rules

- **R-SIGNAL-001** MUST: Node.js processes that perform multi-step file system operations MUST register handlers for SIGINT and SIGTERM signals using process.on() to coordinate graceful shutdown.

### Verify

```bash
# Check for SIGINT handler registration
grep -r "process\.on.*SIGINT" packages/create-puck-app/ packages/core/

# Check for SIGTERM handler registration
grep -r "process\.on.*SIGTERM" packages/create-puck-app/ packages/core/

# Verify both signal handlers are present in CLI entry points
node -e "const fs = require('fs'); const files = ['packages/create-puck-app/index.js']; files.forEach(f => { const content = fs.readFileSync(f, 'utf-8'); if (!content.includes('process.on') || (!content.includes('SIGINT') && !content.includes('SIGTERM'))) { process.exit(1); } });"
```

**Accept when:**
- All Node.js CLI applications that perform file system operations register handlers for both SIGINT and SIGTERM signals
- Signal handlers reference a cleanup function that executes before process termination
- Grep commands return matches in packages/create-puck-app/index.js showing process.on('SIGINT') and process.on('SIGTERM') patterns

<enforcement>
Claude Code MUST NOT skip or defer verification of signal handler registration in Node.js CLI applications. All three verify commands must pass before accepting changes to CLI entry points.
</enforcement>