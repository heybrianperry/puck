# Validate User Input Before Logging Error Messages: String Values External

These rules are ALWAYS ACTIVE for all console.error, console.log, and console.warn calls that include user-provided input from external sources such as command-line arguments, environment variables, localStorage, sessionStorage, URL parameters, or interactive prompts.

### Rules

- **R-VALIDATE-001** MUST: String values from external sources (command-line arguments, environment variables, localStorage, user prompts) MUST be trimmed and validated as non-empty before logging.
- **R-VALIDATE-002** MUST: All console.error, console.log, and console.warn calls that interpolate user-provided input MUST use a sanitization utility function that removes control characters (\x00-\x1F, \x7F), ANSI escape sequences, and truncates to a maximum safe length.
- **R-VALIDATE-003** MUST: User input logged in error messages MUST be wrapped with a sanitizeForLog() utility function before interpolation into template literals.
- **R-VALIDATE-004** SHOULD: A centralized sanitizeForLog(value: unknown, maxLength = 100) utility function SHOULD be created and exported from a shared utilities module for consistent application across all packages.

### Verify

```bash
# Detect console.error with unsanitized template literals
grep -r "console\.error.*\${" packages/ | grep -v "sanitizeForLog" | wc -l | grep -q "^0$"

# Detect console.log with unsanitized template literals
grep -r "console\.log.*\${" packages/ | grep -v "sanitizeForLog" | wc -l | grep -q "^0$"

# Verify sanitization utility exists and is exported
test -f packages/core/lib/sanitize-for-log.ts && grep -q "export.*sanitizeForLog" packages/core/lib/sanitize-for-log.ts
```

**Accept when:**
- All console.error and console.log statements that interpolate variables use sanitizeForLog wrapper function
- No grep matches found for console logging with template literals that bypass sanitization
- Sanitization utility function exists and is exported from shared utilities module
- The sanitizeForLog function removes control characters (\x00-\x1F, \x7F), ANSI escape sequences, and enforces maximum length truncation

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes to logging statements that include user-provided input.
</enforcement>