# Validate User Input Before Logging Error Messages: Logging Utilities Implement

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase uses console.error for logging validation failures and operational errors in CLI tooling (packages/create-puck-app) and UI components (packages/core)
- User-provided input (app names, recipe names, directory paths) is incorporated directly into error messages without sanitization or validation before logging
- The CLI accepts input from multiple sources including command-line arguments, interactive prompts via inquirer, and environment variables (npm_config_user_agent)
- Error messages are constructed using template literals that interpolate user-controlled values such as appName, recipeName, and file paths
- The pattern appears in both Node.js CLI contexts and browser-side React hooks where localStorage parsing failures are logged with position identifiers

## Problem Statement

When user-provided input is logged directly in error messages without validation or sanitization, it creates potential security risks including log injection attacks, information disclosure, and log parsing failures. The current implementation in packages/create-puck-app/index.js and packages/core/lib/use-sidebar-resize.ts logs user-controlled strings (app names, recipe names, localStorage values) directly via console.error without bounds checking, encoding, or validation, which could allow malicious input to corrupt logs, inject control characters, or expose sensitive data.

## Decision

1. MAY: Logging utilities MAY implement automatic sanitization functions that are applied to all interpolated values in log messages

## Policy Block

- MAY Logging utilities MAY implement automatic sanitization functions that are applied to all interpolated values in log messages

In scope:
- All console.error, console.log, console.warn calls that include user-provided input
- CLI applications accepting command-line arguments, flags, or interactive prompts
- Browser-side code logging values from localStorage, sessionStorage, or URL parameters
- Error handlers that log exception messages containing user data
- Template literals used to construct log messages with interpolated variables

Out of scope:
- Logging of system-generated identifiers, timestamps, or internal state that does not include user input
- Debug logging in development environments where logs are not persisted or exposed
- Structured logging frameworks that automatically sanitize field values

Exceptions:
- EXC-001: Logging occurs in isolated development or testing environments where logs are not persisted, aggregated, or accessible to other users

## Rationale

- The evidence shows 2 files with 89.40% confidence where user input (appName, recipeName, localStorage values) is directly interpolated into console.error messages without validation
- The CLI in packages/create-puck-app/index.js accepts input from inquirer prompts, command-line arguments, and environment variables, then logs these values in error conditions without sanitization
- The use-sidebar-resize.ts hook parses JSON from localStorage and logs errors with position identifiers that could be manipulated through localStorage injection
- Validating input before logging prevents log injection attacks where malicious users insert control characters or ANSI codes to corrupt log files or mislead operators

## Consequences

Positive:
- Prevents log injection attacks where malicious input corrupts log files or injects false log entries
- Reduces risk of information disclosure through crafted input that exposes system paths or internal state
- Improves log parsing reliability by ensuring log messages follow consistent format without embedded control characters
- Enhances security posture by treating user input as untrusted data throughout the logging pipeline

Negative:
- Adds validation and sanitization overhead to error handling paths, slightly increasing code complexity
- May truncate or modify user input in logs, potentially making debugging more difficult when the exact input is needed
- Requires developers to remember validation rules when adding new logging statements
- Could mask legitimate special characters in user input that are relevant for troubleshooting

## Alternatives

- Log user input without validation or sanitization (current approach) (rejected)
  Rejected because: Creates security vulnerabilities including log injection, information disclosure, and log corruption. Evidence shows direct interpolation of user-controlled strings into console.error without bounds checking.
  When valid: Never valid in production code; only acceptable in isolated development environments with no log persistence
- Use structured logging library (e.g., winston, pino) with automatic field sanitization (deferred)
  Rejected because: Would require significant refactoring of existing console.error calls and introduction of new dependencies. Valid long-term solution but not immediately actionable.
  When valid: Recommended for future architectural improvement when standardizing logging infrastructure across packages
- Implement centralized sanitization utility function for all log messages (accepted)
  When valid: Provides immediate security improvement with minimal refactoring. Can wrap existing console.error calls with sanitization layer.

## Risks

- Developers may bypass validation when adding new logging statements, creating inconsistent security posture
  Mitigation: Implement linting rules to detect console.error with template literals containing variables. Add code review checklist item for input validation in logs.
  Owner: Engineering team
- Overly aggressive sanitization may remove legitimate characters needed for debugging, reducing log utility
  Mitigation: Define clear sanitization rules that preserve alphanumeric, common punctuation, and path separators while removing only control characters and ANSI codes. Document sanitization behavior.
  Owner: Engineering team
- Existing code in 2+ files requires remediation, creating technical debt if not addressed systematically
  Mitigation: Create tracking issue for remediation of packages/create-puck-app/index.js and packages/core/lib/use-sidebar-resize.ts. Prioritize CLI code due to higher attack surface.
  Owner: Engineering team

## Implementation Notes

- Create a sanitizeForLog(value: unknown, maxLength = 100) utility function that validates type, trims whitespace, removes control characters (\x00-\x1F, \x7F), and truncates to maxLength
- In packages/create-puck-app/index.js, wrap appName, recipeName, and appPath variables with sanitization before logging: console.error(`No recipe named ${sanitizeForLog(recipeName)} exists.`)
- In packages/core/lib/use-sidebar-resize.ts, sanitize the position parameter before logging localStorage errors: console.error(`Failed to load ${sanitizeForLog(position)} sidebar width from localStorage`, error)
- Add ESLint rule or custom linter to detect console.error/log/warn with template literals and flag for review if variables are not wrapped in sanitization function
- Document sanitization rules in security guidelines: remove \n, \r, \t, ANSI escape sequences (\x1B[...m), and truncate strings longer than 100 characters with ellipsis

## Continuation Context


Verify commands:
- grep -r "console\.error.*\${" packages/ | grep -v "sanitizeForLog" | wc -l | grep -q "^0$"
- grep -r "console\.log.*\${" packages/ | grep -v "sanitizeForLog" | wc -l | grep -q "^0$"
- test -f packages/core/lib/sanitize-for-log.ts && grep -q "export.*sanitizeForLog" packages/core/lib/sanitize-for-log.ts

Accept when:
- All console.error and console.log statements that interpolate variables use sanitizeForLog wrapper function
- No grep matches found for console logging with template literals that bypass sanitization
- Sanitization utility function exists and is exported from shared utilities module

## Enforcement

- Verified by: ESLint custom rule detecting console.error/log with unsanitized template literals
- Verified by: Code review checklist requiring validation of user input in log messages
- Verified by: CI pipeline running grep-based verification commands to detect violations
- Violation handling: CI build fails if verification commands detect unsanitized logging of user input
- Violation handling: Code review blocks merge if new logging statements interpolate variables without sanitization
- Violation handling: Security review required for any exceptions to sanitization requirements
- Exception process: Developer documents specific justification for exception in code comments and pull request description
- Exception process: Tech lead reviews exception request and confirms isolation or alternative mitigation
- Exception process: Exception is tracked in security review log with approval date and expiration timeline