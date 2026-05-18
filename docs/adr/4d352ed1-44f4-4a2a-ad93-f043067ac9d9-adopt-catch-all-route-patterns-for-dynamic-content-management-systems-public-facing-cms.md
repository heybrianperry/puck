# Adopt Catch-All Route Patterns for Dynamic Content Management Systems: Public Facing Cms

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all public-facing API routes and dynamic content management integrations in Next.js, Remix, and React Router applications.

## Context

- Modern content management systems require flexible routing mechanisms that can handle arbitrary path structures without predefined route definitions
- Framework-specific catch-all route patterns (Next.js [...slug], Remix $, React Router splat) have emerged as the standard approach for delegating path resolution to external CMS systems
- The pattern was detected across 4 files with 90% confidence in Next.js, Remix, and React Router implementations, all integrating with the Puck CMS
- Public-facing APIs need to support dynamic content hierarchies where the URL structure is determined by content editors rather than developers
- The concurrency model facet indicates these routes handle asynchronous content fetching and rendering patterns consistently across frameworks

## Problem Statement

When integrating external content management systems with modern JavaScript frameworks, applications need a routing strategy that can handle arbitrary URL paths without requiring developers to predefine every possible route. Traditional static routing approaches create maintenance overhead and limit content editor flexibility, while inconsistent catch-all implementations across frameworks lead to fragmented patterns and increased cognitive load.

## Decision

1. MUST: Public-facing CMS integration routes MUST use framework-native catch-all route syntax: [...slug] for Next.js App Router, $ parameter for Remix, and splat routes for React Router

## Policy Block

- MUST Public-facing CMS integration routes MUST use framework-native catch-all route syntax: [...slug] for Next.js App Router, $ parameter for Remix, and splat routes for React Router

In scope:
- All public-facing routes that delegate path resolution to external CMS systems
- API routes that serve dynamic content based on CMS-managed URL structures
- Framework-specific route files in Next.js (page.tsx), Remix (routes/*.tsx), and React Router (routes/*.tsx)
- Routes handling user-generated or editor-managed content hierarchies

Out of scope:
- Static application routes with predefined paths (e.g., /login, /dashboard)
- API routes with fixed endpoints and parameter schemas
- Internal service-to-service communication routes
- Routes handling file uploads or binary content delivery
- Authentication and authorization endpoints

Exceptions:
- EXC-001: Legacy CMS integrations require custom routing logic that cannot be expressed through catch-all patterns
- EXC-002: Performance profiling demonstrates that catch-all routes create unacceptable latency for specific high-traffic paths

## Rationale

- Pattern detected with 90% confidence across 4 files demonstrates consistent adoption across major React frameworks (Next.js, Remix, React Router)
- Framework-native catch-all syntax provides optimal performance and type safety compared to custom routing solutions
- Delegating path resolution to CMS systems enables content editors to manage URL structures without developer intervention, reducing deployment cycles
- Consistent catch-all patterns reduce cognitive load when working across multiple framework implementations and improve code maintainability

## Consequences

Positive:
- Content editors gain full control over URL structure and content hierarchy without requiring code changes or deployments
- Reduced maintenance overhead as new content paths are automatically handled without route configuration updates
- Framework-native implementations provide optimal performance, type safety, and developer experience
- Consistent pattern across frameworks enables easier knowledge transfer and code reuse in polyglot environments

Negative:
- Catch-all routes can mask routing errors and make debugging more difficult when paths are not resolved correctly
- Performance implications if CMS resolution layer is not properly optimized, as every request requires external lookup
- Potential security concerns if path validation is not implemented correctly, allowing unauthorized access to content
- Framework-specific syntax creates migration overhead when switching between frameworks

## Alternatives

- Static route generation at build time for all CMS content paths (rejected)
  Rejected because: Requires full site rebuild for every content change, eliminating the flexibility benefits of dynamic CMS systems and creating unacceptable deployment latency
  When valid: Suitable for static site generators with infrequent content updates and build-time rendering requirements
- Custom middleware-based routing with manual path parsing (rejected)
  Rejected because: Bypasses framework routing optimizations, increases maintenance burden, and loses type safety benefits of native route parameters
  When valid: May be necessary for complex multi-tenant scenarios where framework routing is insufficient
- Hybrid approach with static routes for common paths and catch-all for remaining content (deferred)
  Rejected because: Adds complexity and potential for routing conflicts, but may be necessary for performance optimization
  When valid: Consider when performance profiling identifies specific high-traffic paths that would benefit from static optimization

## Risks

- CMS resolution layer becomes a single point of failure, causing all dynamic routes to fail if CMS is unavailable
  Mitigation: Implement caching layer, fallback mechanisms, and health checks for CMS connectivity. Consider static fallback pages for critical content.
  Owner: Platform Engineering Team
- Inadequate path validation in catch-all routes could expose unauthorized content or enable path traversal attacks
  Mitigation: Implement strict path validation, sanitization, and authorization checks before CMS resolution. Use allowlists for valid path patterns where possible.
  Owner: Security Team
- Performance degradation if CMS resolution requires multiple database queries or external API calls per request
  Mitigation: Implement aggressive caching strategies, optimize CMS query patterns, and monitor response times. Consider edge caching for published content.
  Owner: Performance Engineering Team

## Implementation Notes

- Use framework-specific parameter extraction: params.puckPath (Next.js), params.$ (Remix), or params['*'] (React Router) to access the full captured path
- Implement consistent error handling for unresolved paths, returning appropriate 404 responses rather than exposing internal errors
- Consider implementing path normalization (trailing slashes, case sensitivity) before passing to CMS resolution layer to ensure consistent behavior
- Add monitoring and logging for catch-all route performance to identify slow CMS resolution patterns and optimize accordingly

## Continuation Context


Verify commands:
- grep -r "\[\.\.\..*\]" --include="page.tsx" --include="route.tsx" app/ || grep -r "params\.\$" --include="*.tsx" app/routes/ || grep -r "splat" --include="*.tsx" app/routes/
- find . -type f \( -name "*[...]*" -o -name "*\$*" -o -name "*splat*" \) -path "*/routes/*" -o -path "*/app/*"
- grep -r "puckPath\|cmsPath" --include="*.tsx" --include="*.ts" app/ | grep -E "params\.(puckPath|\$|\*)"

Accept when:
- All CMS integration routes use framework-native catch-all syntax and can be identified by grep patterns
- Route handlers extract and pass full paths to CMS resolution without modification or truncation
- No custom routing middleware is implemented for CMS path resolution in new code
- All catch-all routes include proper error handling for unresolved paths with appropriate HTTP status codes

## Enforcement

- Verified by: Automated code review checks scanning for catch-all route patterns in pull requests
- Verified by: CI pipeline verification using grep commands to ensure framework-native syntax compliance
- Verified by: Architecture review for new CMS integrations to validate routing approach
- Verified by: Regular security audits of catch-all route implementations for path validation and authorization
- Violation handling: Pull requests with non-compliant routing patterns are blocked until corrected
- Violation handling: Existing violations are tracked in technical debt backlog with priority based on traffic and security risk
- Violation handling: New CMS integrations must demonstrate compliance before production deployment
- Violation handling: Quarterly reviews identify and prioritize migration of legacy routing patterns
- Exception process: Submit exception request to architecture review board with detailed justification and performance/security analysis
- Exception process: Provide evidence that framework-native catch-all patterns cannot meet requirements
- Exception process: Document alternative approach with security review and performance benchmarks
- Exception process: Exceptions are time-limited (6-12 months) with required migration plan to standard patterns