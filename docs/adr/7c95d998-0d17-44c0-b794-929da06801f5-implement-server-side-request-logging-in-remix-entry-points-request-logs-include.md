# Implement Server-Side Request Logging in Remix Entry Points: Request Logs Include

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all Remix applications with server-side rendering capabilities. It governs logging implementation in entry.server.tsx files.

## Context

- Remix applications use entry.server.tsx as the primary server-side entry point for handling all incoming HTTP requests during server-side rendering
- Server-side rendering frameworks require comprehensive observability to diagnose performance issues, track request patterns, and debug production incidents
- The entry.server.tsx file is the ideal location for centralized logging as it intercepts all requests before they reach route handlers
- Pattern detected across 2 Remix application implementations (remix and remix-ai recipes) with 91% confidence, indicating a consistent architectural approach
- Service boundary definitions require clear logging at entry points to establish observability perimeters and trace request lifecycles

## Problem Statement

Without standardized logging at the server entry point, Remix applications lack visibility into request handling, performance characteristics, and error conditions during server-side rendering. This creates blind spots in production monitoring and makes debugging difficult when issues occur in the SSR pipeline.

## Decision

1. MUST: Request logs MUST include at minimum: HTTP method, URL path, timestamp, and response status code

## Policy Block

- MUST Request logs MUST include at minimum: HTTP method, URL path, timestamp, and response status code

In scope:
- All entry.server.tsx files in Remix applications
- Server-side rendering request handling pipelines
- HTTP request/response logging at application boundaries
- Error logging during SSR hydration and rendering

Out of scope:
- Client-side logging in browser contexts
- Route-specific logging within individual route handlers
- Third-party API request logging (unless initiated during SSR)
- Database query logging (handled by separate data layer concerns)

Exceptions:
- EXC-001: Static site generation (SSG) mode where no server-side rendering occurs
- EXC-002: Development environments where verbose console logging is preferred over structured logs

## Rationale

- Pattern detected with 91% confidence across 2 independent Remix implementations indicates this is an established best practice in the Remix ecosystem
- Centralizing logging at the entry point ensures consistent observability across all routes without requiring individual route instrumentation
- Server-side rendering introduces complexity that requires dedicated logging to diagnose issues that don't occur in client-only applications
- Service boundary logging aligns with observability best practices for microservices and distributed systems, treating the Remix SSR layer as a distinct service boundary

## Consequences

Positive:
- Comprehensive visibility into all server-side requests enables faster incident response and debugging
- Centralized logging implementation reduces code duplication and ensures consistency across routes
- Performance metrics collected at entry point provide baseline measurements for optimization efforts
- Structured logs enable integration with modern observability platforms (DataDog, New Relic, CloudWatch)

Negative:
- Additional logging overhead may introduce minor performance impact (typically <5ms per request)
- Log volume increases operational costs for log storage and analysis in high-traffic applications
- Requires careful implementation to avoid logging sensitive data and maintain compliance (GDPR, HIPAA)
- Teams must maintain logging infrastructure and ensure log retention policies are properly configured

## Alternatives

- Implement logging at the route handler level instead of entry point (rejected)
  Rejected because: Requires duplicating logging logic across all routes, increases maintenance burden, and creates inconsistent logging patterns. Entry point logging provides complete coverage automatically.
  When valid: May be appropriate for route-specific business logic logging that supplements entry point logs
- Use reverse proxy (nginx, CloudFlare) logging exclusively without application-level logging (rejected)
  Rejected because: Proxy logs lack application context (SSR errors, rendering duration, React hydration issues) that are critical for debugging Remix-specific problems
  When valid: Proxy logs should complement but not replace application logging
- Implement logging via Remix middleware or custom server setup (deferred)
  Rejected because: While viable, this approach is less portable across different Remix deployment targets (Vercel, Netlify, custom Node servers)
  When valid: Appropriate for applications with custom server implementations that require more sophisticated middleware chains

## Risks

- Logging sensitive user data (PII, credentials) could create compliance violations and security risks
  Mitigation: Implement log sanitization middleware that filters sensitive fields before emission. Conduct security review of logged data. Use allowlist approach for logged fields.
  Owner: Security team and engineering team
- High-volume logging in production could impact application performance or exhaust log storage quotas
  Mitigation: Implement sampling for high-frequency endpoints. Use asynchronous logging. Set up log volume monitoring and alerts. Configure appropriate log retention policies.
  Owner: DevOps team
- Inconsistent logging implementations across different Remix applications could reduce effectiveness of centralized monitoring
  Mitigation: Create shared logging utility library or package. Document standard logging schema. Implement linting rules to enforce logging patterns.
  Owner: Platform engineering team

## Implementation Notes

- Use Remix's handleRequest lifecycle hook in entry.server.tsx to intercept requests and responses for logging
- Consider using established logging libraries (pino, winston) that support structured logging and performance optimization
- Implement environment-aware logging levels (verbose in dev, structured JSON in production)
- Set up correlation IDs using request headers (X-Request-ID) to enable distributed tracing across services
- Configure log aggregation early (CloudWatch, DataDog, Splunk) to avoid losing historical data during incidents

## Continuation Context


Verify commands:
- grep -r 'entry.server.tsx' app/ && grep -E '(console\.log|logger\.|log\()' app/entry.server.tsx
- test -f app/entry.server.tsx && grep -E '(handleRequest|handleDataRequest)' app/entry.server.tsx
- npm test -- --grep 'entry.server.*logging' || echo 'Add tests for entry.server logging'

Accept when:
- The entry.server.tsx file contains logging implementation that captures request method, URL, and status code
- Verification commands successfully identify logging statements in entry.server.tsx
- Manual code review confirms that sensitive data is filtered from logs and performance impact is minimal

## Enforcement

- Verified by: Automated code review checks in CI/CD pipeline scanning for logging patterns in entry.server.tsx
- Verified by: Manual architecture review during pull request approval for new Remix applications
- Verified by: Periodic audit of production logs to verify completeness and proper formatting
- Violation handling: CI pipeline warnings for missing logging in entry.server.tsx (non-blocking initially)
- Violation handling: Architecture review required before production deployment if logging is absent
- Violation handling: Post-incident reviews will identify missing logging as contributing factor and require remediation
- Exception process: Submit exception request to architecture team with justification and alternative observability approach
- Exception process: Document approved exceptions in project README and architecture decision log
- Exception process: Review exceptions quarterly to determine if they should be made permanent or remediated