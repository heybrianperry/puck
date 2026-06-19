# Expose Public Environment Variables via NEXT_PUBLIC_ Prefix for Client-Side Configuration: Client Side Accessible

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- Next.js applications require runtime configuration values to be accessible in both server-side and client-side contexts, with different security boundaries for each
- Environment variables prefixed with NEXT_PUBLIC_ are embedded into the client-side JavaScript bundle at build time, making them publicly accessible to browsers
- The codebase uses process.env.NEXT_PUBLIC_PLAUSIBLE_DATA_DOMAIN, process.env.NEXT_PUBLIC_BASE_URL, process.env.NEXT_PUBLIC_IS_CANARY, and process.env.NEXT_PUBLIC_IS_LATEST across Document and ReleaseSwitcher components
- These public environment variables configure external integrations (Plausible analytics) and API endpoints that must be accessible from the browser runtime

## Problem Statement

Client-side React components in Next.js applications need access to configuration values for external service integration and API routing, but exposing arbitrary environment variables to the browser creates security risks. A clear naming convention is required to distinguish public configuration from sensitive credentials while maintaining build-time optimization.

## Decision

1. MUST: Client-side accessible environment variables MUST use the NEXT_PUBLIC_ prefix

## Policy Block

- MUST Client-side accessible environment variables MUST use the NEXT_PUBLIC_ prefix

In scope:
- All Next.js client-side components requiring runtime configuration
- Browser-accessible API endpoints and external service URLs
- Feature flags and environment indicators (canary, latest) visible to users
- Public analytics and monitoring service configuration

Out of scope:
- Server-side only configuration values
- Database connection strings and credentials
- API keys for backend services
- Authentication tokens and secrets
- Internal service endpoints not exposed to browsers

Exceptions:
- EXC-001: Server-side rendering (SSR) or API routes require configuration that should not be exposed to the client

## Rationale

- The NEXT_PUBLIC_ prefix provides an explicit security boundary between public and private configuration, preventing accidental exposure of sensitive credentials in client bundles
- Next.js build-time inlining of NEXT_PUBLIC_ variables enables static optimization while maintaining clear visibility into what configuration is publicly accessible
- The pattern observed across Document and ReleaseSwitcher components demonstrates consistent usage for external service integration (Plausible) and API routing (BASE_URL), establishing a working convention
- Conditional rendering based on environment variable presence (process.env.NEXT_PUBLIC_PLAUSIBLE_DATA_DOMAIN check) shows defensive programming that prevents runtime errors when optional configuration is missing

## Consequences

Positive:
- Clear security boundary between public and private configuration reduces risk of credential exposure
- Build-time inlining of public variables enables static optimization and eliminates runtime configuration lookup overhead
- Explicit naming convention makes code review and security audits straightforward
- Supports multiple deployment environments (development, staging, production) with environment-specific public configuration

Negative:
- Public environment variables are immutable after build time, requiring rebuild for configuration changes
- All NEXT_PUBLIC_ values are visible in client-side JavaScript bundles and browser developer tools
- Developers must remember to use the prefix, creating potential for configuration errors
- Build-time inlining increases bundle size proportional to the number and length of public environment variables

## Alternatives

- Use runtime configuration API endpoint to fetch client-side configuration dynamically (rejected)
  Rejected because: Adds network latency and complexity for configuration that is static per deployment; increases server load and requires additional API endpoint maintenance
  When valid: When configuration must change without redeployment or when configuration values are user-specific
- Inject configuration via server-side rendering props in every page component (rejected)
  Rejected because: Creates repetitive boilerplate across all pages; couples configuration to page-level data fetching; does not support static site generation
  When valid: When configuration is request-specific or requires server-side computation per request
- Use window object injection in custom Document to provide runtime configuration (rejected)
  Rejected because: Bypasses Next.js optimization and type safety; creates global state pollution; harder to track configuration usage across components
  When valid: When integrating with legacy systems that expect global configuration objects

## Risks

- Developers accidentally prefix sensitive credentials with NEXT_PUBLIC_, exposing them in client bundles
  Mitigation: Implement automated scanning in CI pipeline to detect common secret patterns in NEXT_PUBLIC_ variables; conduct security training on the prefix convention; use code review checklists
  Owner: Security team and engineering team
- Missing or misconfigured public environment variables cause runtime failures in production
  Mitigation: Add build-time validation to verify required NEXT_PUBLIC_ variables are defined; implement conditional rendering with fallbacks for optional variables; document all public variables in environment templates
  Owner: Engineering team
- Public environment variables contain environment-specific URLs that leak internal infrastructure details
  Mitigation: Use relative URLs where possible; review all NEXT_PUBLIC_ values for information disclosure; implement URL validation to ensure only approved domains are configured
  Owner: Security team

## Implementation Notes

- Create .env.example and .env.local.example files documenting all NEXT_PUBLIC_ variables with descriptions and example values
- Add TypeScript type definitions for process.env to provide autocomplete and type safety for public environment variables
- Implement build-time validation script that checks for required NEXT_PUBLIC_ variables and warns about potential secrets in public variables
- Use conditional rendering pattern (process.env.NEXT_PUBLIC_VAR && <Component />) for optional integrations as demonstrated in Document component

## Continuation Context


Verify commands:
- grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs --include='*.tsx' --include='*.ts' | grep -v 'API_KEY\|SECRET\|PASSWORD\|TOKEN'
- grep -r 'process\.env\.[A-Z_]*' apps/docs --include='*.tsx' --include='*.ts' | grep -v 'NEXT_PUBLIC_' | grep -v 'NODE_ENV' | wc -l
- npm run build 2>&1 | grep -i 'NEXT_PUBLIC_' || echo 'No public env vars in build output'

Accept when:
- All client-side environment variable references use NEXT_PUBLIC_ prefix and no sensitive patterns (API_KEY, SECRET, PASSWORD, TOKEN) appear in NEXT_PUBLIC_ variable names
- Server-side only environment variables (without NEXT_PUBLIC_ prefix) are not referenced in client-side component files
- Build process completes successfully with all required NEXT_PUBLIC_ variables defined

## Enforcement

- Verified by: Automated grep-based scanning in CI pipeline checking for NEXT_PUBLIC_ usage patterns
- Verified by: Code review checklist verifying no sensitive credentials use NEXT_PUBLIC_ prefix
- Verified by: Build-time validation script that fails if required public environment variables are undefined
- Violation handling: CI pipeline fails if sensitive patterns are detected in NEXT_PUBLIC_ variable names
- Violation handling: Pull requests blocked until code review confirms proper environment variable classification
- Violation handling: Build failures reported with clear error messages indicating missing or misconfigured public variables
- Exception process: Document the exception rationale in ADR or technical design document
- Exception process: Obtain security team approval for any non-standard public environment variable usage
- Exception process: Add inline code comments explaining why the exception is necessary and safe