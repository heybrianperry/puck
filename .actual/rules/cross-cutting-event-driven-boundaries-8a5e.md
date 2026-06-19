# Register Process Signal Handlers for Graceful Shutdown in Event-Driven Boundaries: Event Driven Boundaries

These rules are ALWAYS ACTIVE for Node.js CLI applications in the packages/create-puck-app directory that perform long-running operations involving file system writes, template compilation, external process execution, or interactive user input.

### Rules

- **R-EDB-001** SHOULD: Event-driven boundaries established via process.on() SHOULD be registered early in the application lifecycle, before initiating long-running operations.

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
Clause Code MUST NOT skip or defer verification. Signal handler registration is mandatory for all CLI applications performing file system operations, template compilation, or external process execution to prevent partial state and orphaned processes.
</enforcement>