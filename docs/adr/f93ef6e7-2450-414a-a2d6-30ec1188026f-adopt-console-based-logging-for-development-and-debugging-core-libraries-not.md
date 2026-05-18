# Adopt Console-Based Logging for Development and Debugging: Core Libraries Not

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The codebase demonstrates consistent use of console-based logging across 8 files spanning core libraries, UI components, documentation, and server-side recipes
- Pattern detected with 90.29% confidence across multiple architectural layers including data resolution, React hooks, component rendering, and server entry points
- The logging pattern appears in both client-side (React components, hooks) and server-side (Remix entry points) contexts, indicating a unified approach to observability
- The facet 'libs.core.detected' suggests this is a foundational pattern embedded in core library functionality rather than application-specific logging

## Problem Statement

Development teams need a consistent, lightweight, and universally available logging mechanism for debugging, error tracking, and operational visibility across both client-side and server-side code without introducing heavy dependencies or complex logging infrastructure.

## Decision

1. MUST_NOT: Core libraries MUST NOT introduce third-party logging frameworks as required dependencies

## Policy Block

- MUST_NOT Core libraries MUST NOT introduce third-party logging frameworks as required dependencies

In scope:
- Core library code (packages/core/*)
- React components and hooks
- Server-side entry points and middleware
- Data resolution and transformation utilities
- Development and debugging scenarios

Out of scope:
- Production telemetry and metrics collection
- Structured logging for log aggregation systems
- Audit logging with compliance requirements
- Performance profiling and tracing

Exceptions:
- EXC-001: Production environments require structured logging for centralized log aggregation (e.g., ELK, Datadog)
- EXC-002: Security-sensitive operations require audit logging with tamper-proof guarantees

## Rationale

- Console logging is universally available in all JavaScript environments (browsers, Node.js, Deno, Bun) without requiring additional dependencies or configuration
- The pattern's 90.29% confidence across 8 files spanning multiple architectural layers demonstrates organic adoption and proven effectiveness
- Console-based logging provides immediate feedback during development with zero setup overhead, accelerating debugging cycles
- Maintaining a lightweight logging approach in core libraries prevents dependency bloat and allows consuming applications to implement their own production logging strategies

## Consequences

Positive:
- Zero external dependencies for logging functionality reduces bundle size and eliminates supply chain risks
- Immediate availability in all JavaScript runtimes ensures consistent developer experience across environments
- Browser DevTools and Node.js console provide rich formatting, filtering, and inspection capabilities out of the box
- Low barrier to entry for contributors who can use familiar console APIs without learning custom logging frameworks

Negative:
- Console logs lack structured metadata (timestamps, log levels, context) that production logging systems typically provide
- No built-in log filtering or level control in production environments without additional tooling
- Console output may expose sensitive information if not carefully managed in production builds
- Difficult to aggregate and analyze logs across distributed systems without additional infrastructure

## Alternatives

- Adopt a lightweight logging library (e.g., pino, winston, debug) as a core dependency (rejected)
  Rejected because: Introduces unnecessary dependencies for core libraries; increases bundle size; creates coupling to specific logging implementations that may not suit all consuming applications
  When valid: Consider for application-level code where structured logging requirements justify the dependency cost
- Implement a custom logging abstraction layer with pluggable transports (rejected)
  Rejected because: Over-engineering for the current needs; adds maintenance burden; console logging already provides sufficient functionality for development and debugging
  When valid: Revisit if production observability requirements demand standardized structured logging across all components
- Remove all logging from core libraries and rely on consuming applications to add instrumentation (rejected)
  Rejected because: Significantly degrades developer experience; makes debugging core library issues extremely difficult; shifts burden to library consumers
  When valid: Only for highly performance-critical code paths where logging overhead is measurably problematic

## Risks

- Sensitive data (PII, credentials, tokens) may be inadvertently logged to console and exposed in production environments
  Mitigation: Implement code review guidelines to check for sensitive data in log statements; use environment-based log level filtering; consider build-time log stripping for production bundles
  Owner: Engineering team with security review
- Excessive console logging may impact performance in production, especially in tight loops or high-frequency operations
  Mitigation: Use conditional logging based on environment variables (e.g., NODE_ENV); implement log level controls; profile and remove performance-critical logs
  Owner: Engineering team
- Lack of structured logging makes production debugging and incident response more difficult compared to centralized logging solutions
  Mitigation: Document that console logging is for development; provide guidance for applications to wrap console with structured logging adapters for production; maintain separation between debug logs and operational telemetry
  Owner: Architecture team

## Implementation Notes

- Use descriptive log messages with sufficient context (function name, relevant variables, operation being performed) to aid debugging
- Prefer console.error for error conditions, console.warn for warnings, and console.log for informational messages to leverage browser DevTools filtering
- Consider using environment checks (e.g., if (process.env.NODE_ENV === 'development')) to conditionally enable verbose logging
- For libraries, document logging behavior and provide guidance on how consuming applications can suppress or redirect console output if needed

## Continuation Context


Verify commands:
- grep -r 'console\.(log|warn|error|info|debug)' packages/core/ --include='*.ts' --include='*.tsx' | wc -l
- ! grep -r 'import.*winston\|import.*pino\|import.*bunyan' packages/core/ --include='*.ts' --include='*.tsx'
- npm list --depth=0 | grep -E 'winston|pino|bunyan|log4js' && exit 1 || exit 0

Accept when:
- Core library code uses console methods for logging without third-party logging dependencies
- No logging frameworks (winston, pino, bunyan, log4js) are listed as dependencies in core package.json
- Console usage is detectable via grep across the identified pattern files with consistent usage patterns

## Enforcement

- Verified by: Automated dependency scanning in CI to detect introduction of logging framework dependencies
- Verified by: Code review checklist item to verify console usage patterns and check for sensitive data in log statements
- Verified by: Periodic grep-based audits to ensure consistency with the established pattern
- Violation handling: Pull requests introducing logging framework dependencies to core libraries are flagged for architecture review
- Violation handling: Logs containing potential sensitive data trigger security review before merge
- Violation handling: Violations are discussed in code review with guidance to align with console-based approach
- Exception process: Submit exception request to architecture team with justification for alternative logging approach
- Exception process: Document the specific use case and why console logging is insufficient
- Exception process: Obtain approval from both architecture and security teams if the exception involves production logging
- Exception process: Update this ADR with approved exceptions and their rationale