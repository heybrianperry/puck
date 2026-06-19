# Use console.error for CLI Error Reporting in Node.js Applications: Error Logging Include

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- Node.js CLI applications require consistent error reporting mechanisms to communicate validation failures, missing inputs, and runtime errors to users via stderr
- The codebase uses console.error throughout CLI command handlers and React hooks to report user-facing errors, configuration issues, and localStorage failures
- Error messages are displayed for input validation failures (missing app names, invalid recipes, existing directories) and runtime exceptions (localStorage access, git operations)
- The pattern appears in both CLI tooling (packages/create-puck-app) and browser-side code (packages/core), indicating a cross-environment logging approach
- Process signal handlers (SIGINT, SIGTERM) and environment variable access (npm_config_user_agent) suggest the application manages lifecycle events and requires observable error states

## Problem Statement

CLI applications and interactive tools need a standardized approach to report errors, validation failures, and exceptional conditions to users in a way that distinguishes error output from normal program output, enables proper stream redirection, and maintains consistency across different execution contexts (CLI vs browser).

## Decision

1. SHOULD: Error logging SHOULD include sufficient context to identify the failure point (e.g., sidebar position, recipe name, directory path)

## Policy Block

- SHOULD Error logging SHOULD include sufficient context to identify the failure point (e.g., sidebar position, recipe name, directory path)

In scope:
- CLI command handlers in packages/create-puck-app
- Input validation routines for user-provided parameters (app names, recipe selection)
- File system operation error handling (directory existence checks, template copying)
- Browser-side localStorage access in React hooks (packages/core)
- Git operation failures during repository initialization

Out of scope:
- Debug logging or verbose output intended for developers
- Informational messages about successful operations
- Structured logging to external services or log aggregators
- Performance metrics or telemetry data
- Security-sensitive information that should not be exposed to users

## Rationale

- The evidence shows consistent use of console.error across 2 files with 89.40% confidence, indicating an established pattern for error reporting in both CLI and browser contexts
- Using console.error ensures errors are written to stderr, enabling proper Unix-style stream redirection and allowing users to separate error output from normal program output
- The pattern provides immediate user feedback for validation failures (missing app names, invalid recipes, existing directories) without requiring additional logging infrastructure
- Cross-environment consistency (Node.js CLI and browser React hooks) simplifies developer mental models and reduces the need for environment-specific error handling code

## Consequences

Positive:
- Errors are properly separated from normal output, enabling shell redirection patterns like '2>/dev/null' or '2>&1'
- Users receive immediate, actionable feedback about validation failures and runtime errors
- No external logging dependencies required, reducing bundle size and complexity
- Consistent error reporting pattern across CLI and browser environments simplifies code review and maintenance

Negative:
- console.error output is not structured, making automated parsing and log aggregation more difficult
- No built-in error severity levels or categorization beyond stderr vs stdout
- Browser console.error calls may be visible in production, potentially exposing implementation details
- Limited control over error formatting and presentation without additional wrapper functions

## Alternatives

- Use a structured logging library (winston, pino, bunyan) for all error reporting (rejected)
  Rejected because: Adds external dependencies and complexity for simple CLI error reporting; evidence shows console.error meets current needs for user-facing errors
  When valid: When log aggregation, structured querying, or multiple severity levels are required
- Throw exceptions and rely on top-level error handlers (rejected)
  Rejected because: Requires additional error handling infrastructure; console.error provides immediate user feedback without unwinding the call stack
  When valid: When errors should terminate execution or trigger centralized error recovery logic
- Write errors to a log file instead of stderr (rejected)
  Rejected because: Reduces visibility for CLI users who expect immediate terminal feedback; adds file I/O complexity
  When valid: When persistent error logs are required for debugging or audit trails

## Risks

- Sensitive information (file paths, environment variables) may be exposed in error messages
  Mitigation: Review all console.error calls to ensure no secrets, tokens, or sensitive user data are logged; sanitize paths and inputs before logging
  Owner: engineering team
- Browser console.error calls in production may expose implementation details to end users
  Mitigation: Implement environment-aware error handling that provides user-friendly messages in production while preserving detailed errors in development
  Owner: engineering team
- Unstructured error messages make automated monitoring and alerting difficult
  Mitigation: Document error message patterns; consider adding error codes or structured metadata if monitoring requirements emerge
  Owner: engineering team

## Implementation Notes

- Use console.error for all validation failures in CLI command handlers, providing clear guidance on how to resolve the issue
- Wrap localStorage access in try-catch blocks and log failures with console.error, including the operation context (e.g., 'Failed to load left sidebar width from localStorage')
- Ensure error messages are actionable and user-friendly, avoiding technical jargon or stack traces in production
- Consider creating a shared error reporting utility function if error formatting needs become more complex

## Continuation Context


Verify commands:
- grep -r 'console\.error' packages/create-puck-app packages/core --include='*.js' --include='*.ts' | wc -l
- grep -r 'console\.log.*error\|console\.warn.*error' packages/create-puck-app packages/core --include='*.js' --include='*.ts' | wc -l
- node -e "const { execSync } = require('child_process'); try { execSync('cd test-app && node cli.js 2>&1 >/dev/null'); } catch(e) { process.exit(e.status === 1 ? 0 : 1); }"

Accept when:
- All user-facing error messages in CLI handlers use console.error rather than console.log or console.warn
- Error output can be redirected to stderr independently of stdout using standard shell redirection
- localStorage access failures in React hooks are caught and logged with console.error including operation context

## Enforcement

- Verified by: Code review checklist requiring console.error for all error reporting
- Verified by: Linting rules to detect console.log usage in error handling blocks
- Verified by: Manual testing of CLI error scenarios to verify stderr output
- Violation handling: Code review feedback requesting migration from console.log to console.error for error cases
- Violation handling: Documentation updates to clarify error reporting standards
- Violation handling: Refactoring of non-compliant error handling during regular maintenance
- Exception process: Document the specific use case requiring alternative error handling
- Exception process: Obtain approval from tech lead for exceptions involving structured logging or external error services
- Exception process: Add inline comments explaining why console.error is not appropriate for the specific case