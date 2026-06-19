# Register Process Signal Handlers for Graceful Shutdown in Event-Driven Boundaries: Applications Log Shutdown

These rules are ALWAYS ACTIVE for Node.js CLI applications in the packages/create-puck-app directory that perform long-running operations involving file system writes, template compilation, external process execution, or child process management.

### Rules

- **R-SHUTDOWN-001** MAY: Applications MAY log shutdown events to console.error or structured logging systems to aid debugging of interrupted operations.

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
- Shutdown events are logged to console.error or structured logging systems for debugging interrupted operations

<enforcement>
Claude Code MUST NOT skip or defer verification of signal handler registration and shutdown logging in CLI applications.
</enforcement>