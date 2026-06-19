# Validate User Input Before Logging Error Messages: Logging Utilities Implement

These rules are ALWAYS ACTIVE for all console.error, console.log, and console.warn calls that include user-provided input across CLI applications, browser-side code, and error handlers.

### Rules

- **R-LOG-001** MAY: Logging utilities MAY implement automatic sanitization functions that are applied to all interpolated values in log messages.
- **R-LOG-002** MUST: All console.error, console.log, and console.warn calls that interpolate user-provided input (app names, recipe names, file paths, localStorage values, URL parameters) MUST use a sanitization wrapper function before logging.
- **R-LOG-003** MUST: Sanitization utility functions MUST remove control characters (\x00-\x1F, \x7F), ANSI escape sequences (\x1B[...m), and newlines (\n, \r, \t) from logged values.
- **R-LOG-004** MUST: Sanitization utility functions MUST truncate strings longer than 100 characters with ellipsis to prevent log flooding.
- **R-LOG-005** SHOULD: A centralized sanitizeForLog(value: unknown, maxLength = 100) utility function SHOULD be created and exported from a shared utilities module (e.g., packages/core/lib/sanitize-for-log.ts).
- **R-LOG-006** SHOULD: ESLint rules or custom linters SHOULD be configured to detect console.error/log/warn with template literals and flag variables not wrapped in sanitization functions for review.
- **R-LOG-007** SHOULD: Code review checklists SHOULD include validation of user input in log messages as a mandatory check item.

### Verify

```bash
# Verify no unsanitized console.error with template literals
grep -r "console\.error.*\${" packages/ | grep -v "sanitizeForLog" | wc -l | grep -q "^0$"

# Verify no unsanitized console.log with template literals
grep -r "console\.log.*\${" packages/ | grep -v "sanitizeForLog" | wc -l | grep -q "^0$"

# Verify sanitization utility exists and is exported
test -f packages/core/lib/sanitize-for-log.ts && grep -q "export.*sanitizeForLog" packages/core/lib/sanitize-for-log.ts
```

**Accept when:**
- All console.error and console.log statements that interpolate variables use sanitizeForLog wrapper function
- No grep matches found for console logging with template literals that bypass sanitization
- Sanitization utility function exists and is exported from shared utilities module (packages/core/lib/sanitize-for-log.ts)
- Sanitization function removes control characters, ANSI codes, and truncates to 100 characters
- All user-provided input sources (CLI arguments, prompts, localStorage, URL parameters) are sanitized before logging

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline MUST fail if verification commands detect unsanitized logging of user input. Code review MUST block merge if new logging statements interpolate variables without sanitization. Security review is required for any exceptions to sanitization requirements.
</enforcement>