# Validate User Input Before Logging Error Messages: User Input Included

These rules are ALWAYS ACTIVE for all console.error, console.log, and console.warn calls that include user-provided input across CLI applications and browser-side code.

### Rules

- **R-INPUT-LOG-001** MUST: User input included in error logs MUST be sanitized to remove or escape control characters, newlines, and ANSI escape sequences.
- **R-INPUT-LOG-002** MUST: All console.error and console.log statements that interpolate variables MUST use a sanitizeForLog wrapper function.
- **R-INPUT-LOG-003** MUST: User input from command-line arguments, interactive prompts, localStorage, sessionStorage, and URL parameters MUST be validated before logging.
- **R-INPUT-LOG-004** SHOULD: Sanitization utility function SHOULD truncate strings longer than 100 characters with ellipsis.
- **R-INPUT-LOG-005** SHOULD: Sanitization rules SHOULD preserve alphanumeric characters, common punctuation, and path separators while removing only control characters and ANSI codes.

### Verify

```bash
# Verify no console.error with unsanitized template literals
grep -r "console\.error.*\${" packages/ | grep -v "sanitizeForLog" | wc -l | grep -q "^0$"

# Verify no console.log with unsanitized template literals
grep -r "console\.log.*\${" packages/ | grep -v "sanitizeForLog" | wc -l | grep -q "^0$"

# Verify sanitization utility function exists and is exported
test -f packages/core/lib/sanitize-for-log.ts && grep -q "export.*sanitizeForLog" packages/core/lib/sanitize-for-log.ts
```

**Accept when:**
- All console.error and console.log statements that interpolate variables use sanitizeForLog wrapper function
- No grep matches found for console logging with template literals that bypass sanitization
- Sanitization utility function exists and is exported from shared utilities module
- Sanitization removes control characters (\x00-\x1F, \x7F), newlines, carriage returns, tabs, and ANSI escape sequences
- String truncation to 100 characters is implemented with ellipsis indicator

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes. Code review MUST block merge if new logging statements interpolate variables without sanitization wrapper.
</enforcement>