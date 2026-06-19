# Validate User Input Before Logging Error Messages: Json Parse Operations

These rules are ALWAYS ACTIVE for all console.error, console.log, and console.warn calls that include user-provided input across CLI applications, browser-side code, and error handlers throughout the codebase.

### Rules

- **R-JSON-001** SHOULD: JSON.parse operations on user-controlled data SHOULD wrap parsing in try-catch blocks and log sanitized error details rather than raw input.
- **R-JSON-002** MUST: All console.error, console.log, and console.warn calls that interpolate user-provided input (app names, recipe names, file paths, localStorage values, URL parameters) MUST sanitize values before logging.
- **R-JSON-003** MUST: User input interpolated into log messages MUST be wrapped with a sanitizeForLog() utility function that removes control characters (\x00-\x1F, \x7F), ANSI escape sequences, and truncates to a maximum length.
- **R-JSON-004** SHOULD: A centralized sanitizeForLog(value: unknown, maxLength = 100) utility function SHOULD be created and exported from a shared utilities module for consistent sanitization across all packages.
- **R-JSON-005** SHOULD: ESLint rules or custom linters SHOULD be configured to detect console.error/log/warn with template literals and flag variables not wrapped in sanitization functions for review.

### Verify

```bash
# Verify no unsanitized console.error with template literals
grep -r "console\.error.*\${" packages/ | grep -v "sanitizeForLog" | wc -l | grep -q "^0$"

# Verify no unsanitized console.log with template literals
grep -r "console\.log.*\${" packages/ | grep -v "sanitizeForLog" | wc -l | grep -q "^0$"

# Verify sanitization utility function exists and is exported
test -f packages/core/lib/sanitize-for-log.ts && grep -q "export.*sanitizeForLog" packages/core/lib/sanitize-for-log.ts
```

**Accept when:**
- All console.error and console.log statements that interpolate variables use sanitizeForLog wrapper function
- No grep matches found for console logging with template literals that bypass sanitization
- Sanitization utility function exists and is exported from shared utilities module (packages/core/lib/sanitize-for-log.ts)
- Sanitization removes control characters (\x00-\x1F, \x7F), ANSI escape sequences (\x1B[...m), and truncates strings longer than 100 characters

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes. Code review MUST block merge if new logging statements interpolate variables without sanitization. CI pipeline MUST fail if verification commands detect unsanitized logging of user input.
</enforcement>