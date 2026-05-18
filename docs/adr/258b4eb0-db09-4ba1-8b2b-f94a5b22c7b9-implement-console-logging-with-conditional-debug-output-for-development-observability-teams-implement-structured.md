# Implement Console Logging with Conditional Debug Output for Development Observability: Teams Implement Structured

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires runtime observability during development to understand component lifecycle, state changes, and data flow without requiring external debugging tools
- Console logging provides immediate feedback during development and troubleshooting, enabling developers to trace execution paths and identify issues quickly
- Pattern detected across 2 files with 87.95% confidence indicates a consistent approach to logging that balances development needs with production concerns
- The facet 'security.secrets_management' suggests logging implementation must carefully avoid exposing sensitive information such as API keys, tokens, or user credentials
- Frontend components require lightweight observability mechanisms that don't impact production bundle size or runtime performance significantly

## Problem Statement

Development teams need visibility into application behavior during development and debugging without introducing security risks from leaked secrets or performance degradation in production environments. The challenge is implementing a logging strategy that provides sufficient observability during development while ensuring sensitive data is never logged and production builds remain clean and performant.

## Decision

1. MAY: Teams MAY implement structured logging wrappers that provide consistent formatting and automatic sanitization of sensitive data

## Policy Block

- MAY Teams MAY implement structured logging wrappers that provide consistent formatting and automatic sanitization of sensitive data

In scope:
- All frontend components and React components
- Development and debugging workflows
- Client-side JavaScript and TypeScript code
- Build-time logging configuration and optimization
- Local development environments and staging environments

Out of scope:
- Server-side logging (covered by separate backend logging ADRs)
- Production monitoring and observability platforms (e.g., Sentry, DataDog)
- Structured application logging for analytics or business intelligence
- Performance profiling and metrics collection
- Third-party library logging behavior

Exceptions:
- EXC-001: Critical production errors that require immediate console output for emergency debugging
- EXC-002: Feature flags enable verbose logging for specific users or sessions during production troubleshooting

## Rationale

- Pattern detected with 87.95% confidence across 2 files indicates this is an established practice in the codebase that provides value to development workflows
- Console logging is the most accessible and immediate debugging tool for frontend development, requiring no additional infrastructure or tooling setup
- The security.secrets_management facet association indicates the codebase has recognized the risk of logging sensitive information and requires careful implementation
- Conditional logging based on environment allows teams to maintain development observability while ensuring production builds remain clean, secure, and performant

## Consequences

Positive:
- Developers gain immediate visibility into component behavior, state changes, and data flow during development without requiring external debugging tools
- Faster debugging cycles and reduced time-to-resolution for issues during development and testing phases
- Consistent logging patterns across the codebase improve code readability and make it easier for team members to understand execution flow
- Build-time stripping of console statements reduces production bundle size and eliminates potential performance overhead

Negative:
- Risk of accidentally logging sensitive information if developers are not vigilant about data sanitization
- Console logging can create noise in development environments if not properly scoped or filtered by log level
- Maintenance burden of ensuring logging statements remain conditional and don't leak into production
- Over-reliance on console logging may discourage adoption of more sophisticated observability tools for complex debugging scenarios

## Alternatives

- Use a dedicated logging library (e.g., winston, pino, loglevel) with structured logging and automatic sanitization (rejected)
  Rejected because: Adds additional dependency weight to frontend bundles and introduces complexity for simple development logging needs. The pattern evidence shows direct console usage is preferred for lightweight observability.
  When valid: Consider for applications requiring structured logging in production, complex log routing, or integration with centralized logging platforms
- Disable all console logging and rely exclusively on browser DevTools debugger and breakpoints (rejected)
  Rejected because: Significantly slows development workflow by requiring manual breakpoint placement and step-through debugging for every investigation. Console logging provides passive observability without interrupting execution.
  When valid: Appropriate for investigating specific complex issues where step-through debugging provides more value than passive logging
- Implement a custom logging wrapper with automatic environment detection and data sanitization (deferred)
  Rejected because: Not rejected, but deferred pending team capacity. This would provide better safety guarantees and consistency but requires upfront investment to build and maintain.
  When valid: Should be reconsidered if the codebase grows significantly or if multiple incidents of sensitive data logging occur

## Risks

- Sensitive data (tokens, PII, credentials) accidentally logged to console and exposed in production or development environments
  Mitigation: Implement automated code scanning for common sensitive data patterns in logging statements. Provide developer training on data sanitization. Consider implementing a logging wrapper with automatic redaction.
  Owner: Security team and engineering leads
- Console logging statements not properly stripped from production builds, increasing bundle size and exposing internal logic
  Mitigation: Configure build tools (terser, babel) to automatically remove console statements in production builds. Add CI checks to verify production bundles don't contain console.log statements. Implement bundle size monitoring.
  Owner: DevOps and frontend infrastructure team
- Excessive logging in development environments impacts performance and makes it difficult to identify relevant log messages
  Mitigation: Establish conventions for log levels and encourage use of console.warn and console.error for important messages. Implement namespace-based filtering or feature flags to enable/disable specific logging categories.
  Owner: Engineering team

## Implementation Notes

- Use environment variable checks (e.g., process.env.NODE_ENV !== 'production') to conditionally execute logging statements
- Configure build tools to strip console statements: In webpack/terser config, set 'drop_console: true' for production builds
- Create a sanitization utility function that removes or redacts sensitive fields from objects before logging (e.g., sanitize({ token: 'abc123', name: 'John' }) => { token: '[REDACTED]', name: 'John' })
- Prefix log messages with component or module names for easier filtering: console.log('[ReleaseSwitcher]', 'Switching to version:', version)
- Consider using console.group/console.groupEnd for related log messages to improve readability in complex operations
- Document team conventions for when to use console.log vs console.warn vs console.error in the project's contributing guidelines

## Continuation Context


Verify commands:
- grep -r "console\.log.*token\|console\.log.*password\|console\.log.*secret\|console\.log.*apiKey" --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" .
- NODE_ENV=production npm run build && grep -r "console\.log\|console\.warn" dist/ || echo 'No console statements in production build'
- eslint . --ext .ts,.tsx,.js,.jsx --rule 'no-console: [error, { allow: [warn, error] }]' --env-info

Accept when:
- No console logging statements contain sensitive data patterns (token, password, secret, apiKey, credentials) as verified by grep scan
- Production builds have all console.log statements removed or disabled as verified by bundle inspection
- ESLint or similar linting tools are configured to warn or error on console usage in production code paths

## Enforcement

- Verified by: Pre-commit hooks running ESLint with no-console rules configured for production code
- Verified by: CI/CD pipeline checks that scan for sensitive data patterns in logging statements
- Verified by: Build process verification that production bundles have console statements stripped
- Verified by: Code review checklist includes verification that new logging statements are conditional and don't contain sensitive data
- Violation handling: CI build fails if sensitive data patterns are detected in console logging statements
- Violation handling: ESLint errors block commits if console statements are used without environment guards in production code paths
- Violation handling: Code review process flags any logging statements for manual inspection before merge approval
- Violation handling: Post-deployment monitoring alerts if console statements are detected in production bundle analysis
- Exception process: Developer submits exception request documenting the specific need for console logging and why alternatives are insufficient
- Exception process: Security team reviews the request to ensure no sensitive data will be logged and approves sanitization approach
- Exception process: Engineering lead approves the exception and documents it in the codebase with a comment explaining the rationale
- Exception process: Exception is time-bound (e.g., 90 days) and must be renewed or removed after the period expires