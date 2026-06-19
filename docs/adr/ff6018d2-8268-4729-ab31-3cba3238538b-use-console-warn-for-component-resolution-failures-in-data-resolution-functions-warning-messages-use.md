# Use console.warn for Component Resolution Failures in Data Resolution Functions: Warning Messages Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The packages/core library provides data resolution functions (resolveAndReplaceData, resolveDataById, resolveDataBySelector) that look up components by ID or selector from a store
- Component resolution can fail when components are removed, IDs are invalid, or selectors do not match any component in the store
- Failed component lookups represent recoverable runtime conditions that should not halt execution but require developer awareness during development and debugging
- The codebase uses console.warn consistently across three data resolution modules to signal component lookup failures with descriptive context including the failed ID or selector

## Problem Statement

When component resolution fails in data resolution functions, developers need immediate feedback about the failure without disrupting application execution, requiring a consistent logging approach that distinguishes warnings from errors while providing sufficient diagnostic context.

## Decision

1. SHOULD: Warning messages SHOULD use the prefix 'Warning:' to clearly distinguish them from errors

## Policy Block

- SHOULD Warning messages SHOULD use the prefix 'Warning:' to clearly distinguish them from errors

In scope:
- Data resolution functions in packages/core/lib/data/ that look up components by ID or selector
- Functions resolveAndReplaceData, resolveDataById, and resolveDataBySelector
- Any component lookup operation where the component may legitimately not exist

Out of scope:
- Logging for unrecoverable errors or system failures
- Logging in non-core packages or application-level code
- Production logging infrastructure or structured logging systems
- User-facing error messages or notifications

## Rationale

- The pattern appears consistently across three independent data resolution modules (resolve-and-replace-data.ts, resolve-data-by-id.ts, resolve-data-by-selector.ts), indicating an intentional architectural choice
- Using console.warn instead of console.error correctly signals that component resolution failures are warnings rather than fatal errors, allowing execution to continue
- Including the failed identifier and potential causes in the message provides developers with actionable diagnostic information without requiring additional debugging tools
- The consistent message structure ('Warning: Could not find component...') establishes a recognizable pattern that developers can search for in logs

## Consequences

Positive:
- Developers receive immediate feedback about component resolution failures during development without application crashes
- Consistent warning format across all data resolution functions makes it easy to identify and diagnose component lookup issues
- Including the failed identifier in the warning message reduces debugging time by providing specific context
- Using console.warn allows browser developer tools to filter and categorize these messages appropriately

Negative:
- Console warnings may be overlooked in production environments without proper log aggregation and monitoring
- Direct console API usage bypasses structured logging systems that could provide better filtering, aggregation, and alerting
- No mechanism to suppress or configure warning verbosity for different environments (development vs production)
- Warning messages are not localized and assume English-speaking developers

## Alternatives

- Throw exceptions for component resolution failures (rejected)
  Rejected because: Component resolution failures are recoverable conditions that should not halt execution; throwing exceptions would require extensive try-catch handling throughout the codebase
  When valid: When component resolution failure represents a critical error that should prevent further execution
- Use a structured logging library with log levels and filtering (rejected)
  Rejected because: Adds dependency overhead and complexity for a core library; console.warn provides sufficient functionality for development-time diagnostics
  When valid: When production log aggregation, filtering, and monitoring are required; when supporting multiple log transports or formats
- Silent failure with no logging (rejected)
  Rejected because: Developers would have no visibility into component resolution failures, making debugging extremely difficult
  When valid: Never appropriate for a core library where component resolution is a critical operation

## Risks

- Console warnings may create noise in production logs if component resolution failures are frequent
  Mitigation: Implement environment-aware logging that reduces verbosity in production; consider adding a configuration option to control warning output
  Owner: Core library maintainers
- Direct console API usage makes it difficult to intercept, test, or mock logging behavior in unit tests
  Mitigation: Consider wrapping console calls in a thin logging abstraction that can be mocked in tests while maintaining the same API
  Owner: Engineering team
- Warning messages may expose internal implementation details (component IDs, selectors) that should not appear in production logs
  Mitigation: Review warning message content for sensitive information; implement environment-specific message formatting if needed
  Owner: Security and core library teams

## Implementation Notes

- Use the exact message format: 'Warning: Could not find component with id/for selector "<identifier>" to resolve its data. Component may have been removed or the id/selector is invalid.'
- For selector-based lookups, serialize the selector using JSON.stringify() to ensure complex objects are readable in the warning message
- Place the console.warn call immediately after detecting the component resolution failure and before any fallback or recovery logic
- Ensure warning messages are emitted before the function returns or continues execution, so developers see the warning in temporal context with other operations

## Continuation Context


Verify commands:
- grep -r "console.warn" packages/core/lib/data/ | grep -c "Could not find component"
- grep -r "console.error" packages/core/lib/data/resolve.*\.ts | wc -l
- npm test -- --grep "component resolution" 2>&1 | grep -i "warning"

Accept when:
- All three data resolution functions (resolveAndReplaceData, resolveDataById, resolveDataBySelector) emit console.warn for component resolution failures
- Warning messages include the failed identifier (ID or serialized selector) and explain potential causes
- No console.error calls are used for component resolution failures in data resolution modules

## Enforcement

- Verified by: Code review checklist requiring console.warn for component resolution failures
- Verified by: Automated grep-based verification in CI pipeline checking for console.warn usage pattern
- Verified by: Unit tests that verify warning messages are emitted with correct format and content
- Violation handling: Code review feedback requesting changes to use console.warn instead of other logging methods
- Violation handling: CI pipeline warnings when component resolution functions lack appropriate console.warn calls
- Violation handling: Documentation updates to clarify the logging pattern for new contributors
- Exception process: Exceptions require justification in code review explaining why console.warn is inappropriate for the specific case
- Exception process: Alternative logging approaches must provide equivalent or better diagnostic information
- Exception process: Exceptions must be documented in code comments explaining the rationale