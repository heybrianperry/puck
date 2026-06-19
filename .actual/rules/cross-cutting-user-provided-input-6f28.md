# Validate User Input Before Logging Error Messages: User Provided Input

These rules are ALWAYS ACTIVE for all console.error, console.log, and console.warn calls that incorporate user-provided input from command-line arguments, interactive prompts, environment variables, localStorage, sessionStorage, or URL parameters.

### Rules

- **R-INPUT-LOG-001** MUST: All user-provided input MUST be validated for type, format, and length constraints before being incorporated into log messages.
- **R-INPUT-LOG-002** MUST: Create and use a `sanitizeForLog(value: unknown, maxLength = 100)` utility function that validates type, trims whitespace, removes control characters (\x00-\x1F, \x7F), and truncates to maxLength.
- **R-INPUT-LOG-003** MUST: Wrap all user-controlled variables (appName, recipeName, appPath, localStorage values, URL parameters) with the sanitization function before logging: `console.error(\`Message: ${sanitizeForLog(userVar)}\`)`.
- **R-INPUT-LOG-004** MUST: Remove or escape control characters (\n, \r, \t), ANSI escape sequences (\x1B[...m), and other non-printable characters from logged user input.
- **R-INPUT-LOG-005** MUST: Truncate user-provided strings longer than 100 characters with ellipsis (...) to prevent log flooding and injection attacks.
- **R-INPUT-LOG-006** SHOULD: Add ESLint rule or custom linter to detect console.error/log/warn with template literals and flag for review if variables are not wrapped in sanitization function.
- **R-INPUT-LOG-007** SHOULD: Document sanitization rules in security guidelines and require code review checklist item for input validation in all new logging statements.

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
- Sanitization utility function exists and is exported from shared utilities module (packages/core/lib/sanitize-for-log.ts)
- Sanitization function removes control characters, ANSI codes, and truncates strings to 100 characters
- Code review checklist includes validation of user input in log messages

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes. CI pipeline MUST run these commands and fail the build if any verification command fails. Code review MUST block merge if new logging statements interpolate variables without sanitization.
</enforcement>