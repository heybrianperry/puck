<rule_activation id="0f07a6ca-a072-4aca-8c47-0bd4e4cd3aaa" title="Implement Structured Server-Side Logging with Cache Layer Integration in Remix Applications: Error Conditions During" applies_to="**/entry.server.tsx">
These rules are ALWAYS ACTIVE for all Remix server entry points (entry.server.tsx) that handle server-side rendering and request processing.
</rule_activation>

### Rules

- **R-SSR-LOG-001** SHOULD: Error conditions during server-side rendering SHOULD be logged with full stack traces and request context.

### Verify

```bash
# Check for logging statements in entry.server.tsx files
grep -r 'entry.server.tsx' --include='*.tsx' -A 20 | grep -E '(log|logger|console)' | head -10

# Check for cache layer logging integration
grep -r 'cache.*log\|log.*cache' --include='*.tsx' --include='*.ts' recipes/

# Find all entry.server.tsx files with request handling
find . -name 'entry.server.tsx' -exec grep -l 'handleRequest\|renderToString' {} \;
```

**Accept when:**
- entry.server.tsx files contain logging statements that capture request processing events
- Cache layer operations are instrumented with logging that tracks hits, misses, or cache state
- Log output includes structured data (request context, timing, cache metrics) rather than plain console.log statements
- Error conditions include full stack traces and request context in log output
- Logging uses a structured logging library (e.g., pino, winston) with JSON output support

<enforcement>
Claude Code MUST verify logging implementation in entry.server.tsx files and MUST NOT skip verification of error condition logging with stack traces and request context.
</enforcement>