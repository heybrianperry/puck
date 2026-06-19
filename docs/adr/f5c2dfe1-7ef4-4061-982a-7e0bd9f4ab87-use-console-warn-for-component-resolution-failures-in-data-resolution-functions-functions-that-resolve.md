# Use console.warn for Component Resolution Failures in Data Resolution Functions: Functions That Resolve

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The packages/core library implements data resolution functions (resolveAndReplaceData, resolveDataById, resolveDataBySelector) that look up components by ID or selector from a store
- Component lookups can fail when components are removed, IDs are invalid, or selectors do not match any component in the store
- The codebase requires a consistent approach to surfacing component resolution failures to developers during development and debugging
- Three separate data resolution modules exhibit identical logging behavior using console.warn with structured warning messages that include context about the failed lookup

## Problem Statement

When component resolution fails in data resolution functions, developers need visibility into the failure without throwing exceptions that would break application flow. The system must balance between silent failures that hide bugs and disruptive errors that prevent graceful degradation.

## Decision

1. SHOULD: Functions that resolve component data (resolveAndReplaceData, resolveDataById, resolveDataBySelector) SHOULD follow this logging pattern consistently

## Policy Block

- SHOULD Functions that resolve component data (resolveAndReplaceData, resolveDataById, resolveDataBySelector) SHOULD follow this logging pattern consistently

In scope:
- All data resolution functions in packages/core/lib/data/ that perform component lookups by ID or selector
- Functions resolveAndReplaceData, resolveDataById, and resolveDataBySelector
- Any new data resolution utilities that query the component store

Out of scope:
- Error handling for network failures, parsing errors, or other non-lookup failures
- Logging in non-core packages or application-level code
- Production logging infrastructure or structured logging systems
- User-facing error messages or UI notifications

## Rationale

- The evidence shows three separate data resolution modules (resolve-and-replace-data.ts, resolve-data-by-id.ts, resolve-data-by-selector.ts) all use console.warn with consistent message formatting, indicating an established pattern
- Using console.warn rather than console.error or exceptions allows developers to see issues during development while permitting the application to continue operating when components are dynamically removed
- The consistent message structure with 'Warning: Could not find component' prefix enables developers to grep logs and filter console output for resolution failures
- Including the specific ID or serialized selector in each warning provides sufficient context for debugging without requiring additional tooling or log aggregation

## Consequences

Positive:
- Developers receive immediate feedback about component resolution failures in the browser console during development
- Applications can gracefully handle missing components without crashing, supporting dynamic component removal scenarios
- Consistent warning format across all data resolution functions simplifies log searching and debugging
- Low implementation overhead using built-in console.warn API without additional logging dependencies

Negative:
- Console warnings may be overlooked in production environments without proper log aggregation or monitoring
- Using console.warn provides no structured logging, making it difficult to track failure rates or patterns in production
- No mechanism to escalate repeated failures or distinguish between expected vs unexpected missing components
- Browser console output can become noisy in applications with frequent dynamic component updates

## Alternatives

- Throw exceptions on component resolution failures (rejected)
  Rejected because: Would break application flow and prevent graceful degradation when components are legitimately removed during runtime
  When valid: In strict validation modes or during testing where all component references must be valid
- Silent failure with no logging (rejected)
  Rejected because: Would hide bugs and make debugging component resolution issues extremely difficult for developers
  When valid: Never appropriate for a core library where visibility into failures is essential
- Implement structured logging with configurable log levels and handlers (deferred)
  Rejected because: Would add complexity and dependencies to the core package, though may be valuable for production observability
  When valid: When production monitoring requirements justify the additional infrastructure and the library has a pluggable logging abstraction

## Risks

- Production environments may not capture or monitor console.warn output, causing resolution failures to go unnoticed
  Mitigation: Document the logging behavior and recommend log aggregation tools for production deployments; consider adding optional structured logging hooks in future versions
  Owner: engineering team
- High-frequency component updates could generate excessive console warnings, degrading browser performance or obscuring other important logs
  Mitigation: Implement debouncing or rate-limiting for warnings if this becomes an issue; consider adding a configuration option to suppress warnings in production
  Owner: engineering team
- Inconsistent application of this pattern in future data resolution functions could fragment the logging approach
  Mitigation: Document this ADR and reference it in code reviews; consider creating a shared logging utility function to enforce consistency
  Owner: engineering team

## Implementation Notes

- Use the exact message format: 'Warning: Could not find component with id "${id}" to resolve its data. Component may have been removed or the id is invalid.'
- For selector-based lookups, serialize the selector with JSON.stringify to provide complete diagnostic information
- Place the console.warn call immediately after detecting the component lookup failure and before any fallback or return logic
- Ensure the warning message includes enough context (ID, selector, function name) for developers to trace the issue back to the calling code

## Continuation Context


Verify commands:
- grep -r 'console.warn.*Could not find component' packages/core/lib/data/
- grep -r 'resolveAndReplaceData\|resolveDataById\|resolveDataBySelector' packages/core/lib/data/ | wc -l
- test -f packages/core/lib/data/resolve-and-replace-data.ts && test -f packages/core/lib/data/resolve-data-by-id.ts && test -f packages/core/lib/data/resolve-data-by-selector.ts

Accept when:
- All three data resolution functions (resolveAndReplaceData, resolveDataById, resolveDataBySelector) contain console.warn calls with the standardized message format
- Warning messages include the specific identifier (ID or selector) that failed to resolve
- No exceptions are thrown for component resolution failures in these functions

## Enforcement

- Verified by: Code review verification that new data resolution functions follow the console.warn pattern
- Verified by: Grep-based verification commands in CI to ensure warning messages maintain consistent format
- Verified by: Manual testing to confirm warnings appear in browser console during component resolution failures
- Violation handling: Code review feedback requesting alignment with the established logging pattern
- Violation handling: Documentation updates if legitimate exceptions to the pattern are discovered
- Violation handling: Refactoring of inconsistent implementations to match the standard approach
- Exception process: Document the specific reason why console.warn is inappropriate for the use case
- Exception process: Propose an alternative logging approach and get approval from the core team
- Exception process: Update this ADR if a new pattern emerges that should supersede console.warn