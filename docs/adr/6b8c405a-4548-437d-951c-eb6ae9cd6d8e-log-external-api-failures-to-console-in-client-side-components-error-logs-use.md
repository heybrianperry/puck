# Log External API Failures to Console in Client-Side Components: Error Logs Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase uses React components that fetch data from external APIs at runtime using the fetch API
- Client-side components in Next.js applications execute in browser environments where console logging is the primary debugging mechanism
- External API calls to endpoints like /api/releases can fail due to network issues, server errors, or configuration problems
- Environment variables (NEXT_PUBLIC_BASE_URL, NEXT_PUBLIC_IS_CANARY, NEXT_PUBLIC_IS_LATEST) configure runtime behavior and API endpoints
- The ReleaseSwitcher component in apps/docs demonstrates a pattern of catching fetch errors and logging them for developer visibility

## Problem Statement

When client-side React components make external API calls that fail, developers need visibility into these failures during development and debugging without implementing complex error tracking infrastructure. The challenge is providing sufficient diagnostic information while maintaining simplicity in client-side error handling.

## Decision

1. SHOULD: Error logs SHOULD use template literals to interpolate error objects or messages for readability

## Policy Block

- SHOULD Error logs SHOULD use template literals to interpolate error objects or messages for readability

In scope:
- Client-side React components making fetch calls to external APIs
- Components using useEffect hooks for data fetching
- API calls constructed from process.env runtime configuration
- Error handling in browser-executed JavaScript code

Out of scope:
- Server-side API routes or backend services
- Build-time errors or compilation failures
- Structured logging systems with log aggregation
- Production error monitoring services (e.g., Sentry, DataDog)

Exceptions:
- EXC-001: Component implements a dedicated error tracking service (e.g., Sentry) that captures errors automatically

## Rationale

- The evidence shows console.error usage in ReleaseSwitcher component for fetch failures, establishing a lightweight pattern for client-side error visibility
- Browser console logging provides immediate feedback during development without requiring additional infrastructure or dependencies
- The pattern balances simplicity with observability by capturing error context (operation description and error object) in a single log statement
- Environment variable usage (NEXT_PUBLIC_BASE_URL) in API construction makes configuration errors likely, requiring diagnostic logging to identify misconfigurations

## Consequences

Positive:
- Developers gain immediate visibility into external API failures during local development and debugging
- No additional dependencies or infrastructure required for basic error observability
- Simple implementation pattern that is easy to apply consistently across components
- Error messages with context help diagnose configuration issues with environment variables

Negative:
- Console logs are not captured or aggregated in production environments without additional tooling
- No structured error tracking or alerting capabilities for production issues
- Error information may be lost if users do not have browser developer tools open
- Limited ability to correlate errors across multiple users or sessions

## Alternatives

- Implement structured error tracking service (e.g., Sentry) for all client-side errors (rejected)
  Rejected because: Adds external dependency and complexity for a simple documentation site; console logging is sufficient for current needs
  When valid: When production error monitoring and alerting are required, or when error volume justifies aggregation infrastructure
- Silent error handling with only user-facing error states (rejected)
  Rejected because: Eliminates developer visibility into failures during development and debugging; makes troubleshooting significantly harder
  When valid: Never recommended; some form of error logging should always be present
- Custom logging utility that wraps console methods with additional metadata (deferred)
  Rejected because: Adds abstraction layer without clear benefit given current simple logging needs
  When valid: When consistent log formatting, log levels, or conditional logging based on environment becomes necessary

## Risks

- Production errors may go unnoticed if only logged to browser console without monitoring
  Mitigation: Document that console logging is for development; implement production error tracking if production monitoring becomes a requirement
  Owner: Engineering team
- Inconsistent error logging patterns across components if not enforced
  Mitigation: Establish code review guidelines and provide example implementations; consider linting rules for fetch error handling
  Owner: Engineering team
- Sensitive information (API keys, tokens) could be accidentally logged in error messages
  Mitigation: Review error logging to ensure only safe contextual information is included; avoid logging request headers or full URLs with secrets
  Owner: Security review

## Implementation Notes

- Wrap fetch calls in try-catch blocks within useEffect hooks or async functions
- Use console.error (not console.log) to ensure errors are visually distinct in browser dev tools
- Include operation context in error message template (e.g., 'Could not load releases:') followed by error interpolation
- Test error handling by temporarily breaking API URLs or simulating network failures
- Consider adding user-facing error states in addition to console logging for better UX

## Continuation Context


Verify commands:
- grep -r "console\.error" apps/docs/components/ --include="*.tsx" --include="*.ts"
- grep -r "fetch(" apps/docs/components/ --include="*.tsx" --include="*.ts" | grep -v "catch"
- grep -r "process\.env\.NEXT_PUBLIC" apps/docs/ --include="*.tsx" --include="*.ts"

Accept when:
- All fetch calls in client-side components have corresponding catch blocks with console.error statements
- Error log messages include descriptive context identifying the failed operation
- No fetch calls to external APIs exist without error handling

## Enforcement

- Verified by: Code review checklist for new components with external API calls
- Verified by: Manual testing of error scenarios during component development
- Verified by: Grep-based verification commands in CI to detect unhandled fetch calls
- Violation handling: Code review feedback requesting addition of error logging
- Violation handling: Pull request comments with examples of proper error handling pattern
- Violation handling: Blocking review if external API calls lack any error handling
- Exception process: Document exception rationale in component comments if alternative error handling is used
- Exception process: Obtain team lead approval for components using structured error tracking instead of console logging
- Exception process: Update this ADR if a new standard error handling pattern is adopted