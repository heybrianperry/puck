# Validate User Input Before Logging Error Messages: Error Messages Truncate

These rules are ALWAYS ACTIVE for all console.error, console.log, and console.warn calls that include user-provided input across CLI applications and browser-side code.

### Rules

- **R-LOG-001** SHOULD: Error messages SHOULD truncate long user input values to a maximum length (e.g., 100 characters) when logging to prevent log overflow.
- **R-LOG-002** MUST: All console.error, console.log, and console.warn statements that interpolate user-controlled variables MUST wrap those variables with a sanitizeForLog utility function.
- **R-LOG-003** SHOULD: Sanitization utility functions SHOULD remove control characters (\x00-\x1F, \x7F), ANSI escape sequences (\x1B[...m), and newlines (\n, \r, \t) from user input before logging.
- **R-LOG-004** SHOULD: User input from command-line arguments, interactive prompts, localStorage, sessionStorage, URL parameters, and environment variables SHOULD be treated as untrusted and sanitized before logging.

### Verify

```bash
# Detect console.error with unsanitized template literals
grep -r "console\.error.*\${" packages/ | grep -v "sanitizeForLog" | wc -l | grep -q "^0$"

# Detect console.log with unsanitized template literals
grep -r "console\.log.*\${" packages/ | grep -v "sanitizeForLog" | wc -l | grep -q "^0$"

# Verify sanitization utility function exists and is exported
test -f packages/core/lib/sanitize-for-log.ts && grep -q "export.*sanitizeForLog" packages/core/lib/sanitize-for-log.ts
```

**Accept when:**
- All console.error and console.log statements that interpolate variables use sanitizeForLog wrapper function
- No grep matches found for console logging with template literals that bypass sanitization
- Sanitization utility function exists and is exported from shared utilities module (packages/core/lib/sanitize-for-log.ts)
- sanitizeForLog function validates type, trims whitespace, removes control characters and ANSI codes, and truncates to maximum length

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes to logging statements that include user input.
</enforcement>