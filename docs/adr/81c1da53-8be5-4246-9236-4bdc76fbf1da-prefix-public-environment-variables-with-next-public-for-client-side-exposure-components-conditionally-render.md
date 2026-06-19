# Prefix Public Environment Variables with NEXT_PUBLIC_ for Client-Side Exposure: Components Conditionally Render

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- Next.js applications require explicit configuration to expose environment variables to the browser runtime, as server-side variables are not automatically available to client-side code
- The codebase uses process.env to access configuration values including NEXT_PUBLIC_PLAUSIBLE_DATA_DOMAIN, NEXT_PUBLIC_BASE_URL, NEXT_PUBLIC_IS_CANARY, and NEXT_PUBLIC_IS_LATEST across multiple components
- Client-side components in apps/docs/pages/_document.tsx and apps/docs/components/ReleaseSwitcher/index.tsx directly reference these environment variables for analytics integration and API endpoint configuration
- The NEXT_PUBLIC_ prefix pattern serves as a security boundary, preventing accidental exposure of server-only secrets (database credentials, API keys) to the browser bundle

## Problem Statement

Public-facing Next.js applications must safely expose configuration to client-side code without leaking server-side secrets, while maintaining clear boundaries between public and private environment variables to prevent security vulnerabilities from accidental exposure of sensitive credentials in browser-accessible JavaScript bundles.

## Decision

1. MAY: Components MAY conditionally render based on the presence of public environment variables

## Policy Block

- MAY Components MAY conditionally render based on the presence of public environment variables

In scope:
- All Next.js client-side components and pages
- Browser-accessible configuration (analytics domains, public API endpoints, feature flags)
- React components that execute in the browser runtime
- Build-time inlining of public environment variables

Out of scope:
- Server-side API routes and middleware
- Database connection strings and credentials
- Private API keys and authentication secrets
- Server-only configuration (internal service URLs, admin tokens)

Exceptions:
- EXC-001: Development and local testing environments where secrets are non-production test values

## Rationale

- The evidence shows consistent use of NEXT_PUBLIC_ prefixed variables (NEXT_PUBLIC_PLAUSIBLE_DATA_DOMAIN, NEXT_PUBLIC_BASE_URL, NEXT_PUBLIC_IS_CANARY, NEXT_PUBLIC_IS_LATEST) across client-side components, indicating an established pattern for public configuration
- Next.js framework semantics require this prefix to inline environment variables at build time into the browser bundle, making this a framework-enforced security boundary
- The pattern appears in both document-level configuration (_document.tsx) and component-level logic (ReleaseSwitcher), demonstrating application-wide adoption
- This approach provides compile-time safety by making the public/private distinction explicit in variable naming, reducing risk of accidental secret exposure

## Consequences

Positive:
- Clear security boundary between public and private configuration prevents accidental secret leakage to browser bundles
- Framework-enforced naming convention provides compile-time guarantees about variable exposure
- Explicit prefix makes code review easier by immediately identifying client-accessible configuration
- Enables safe configuration of third-party integrations (analytics, CDNs) that require browser-side initialization

Negative:
- Requires duplication of variable names with NEXT_PUBLIC_ prefix, increasing verbosity
- Build-time inlining means environment variable changes require rebuild and redeployment rather than runtime configuration updates
- All public variables are visible in browser source code and network inspector, limiting flexibility for semi-sensitive configuration
- Developers must remember the prefix convention or risk variables being undefined in client-side code

## Alternatives

- Use runtime configuration API endpoint that serves public config from server to client (rejected)
  Rejected because: Adds network latency and complexity for configuration that is static at build time; increases server load for every page load; complicates offline and static export scenarios
  When valid: When configuration must change without redeployment or when values are user-specific
- Expose all environment variables to client by default with opt-out mechanism (rejected)
  Rejected because: Inverts security model to unsafe-by-default; high risk of accidental secret exposure; contradicts principle of least privilege
  When valid: Never appropriate for production applications with sensitive data
- Use separate configuration files for public vs private config (deferred)
  Rejected because: Not rejected but not chosen; would require custom build tooling to replace Next.js built-in env handling
  When valid: In non-Next.js environments or when migrating away from framework-specific patterns

## Risks

- Developers accidentally prefix sensitive credentials with NEXT_PUBLIC_, exposing secrets in browser bundles
  Mitigation: Implement pre-commit hooks and CI checks that scan for common secret patterns (API_KEY, SECRET, PASSWORD) with NEXT_PUBLIC_ prefix; conduct security training on the prefix convention
  Owner: Security team and engineering leads
- Build-time inlining causes stale configuration in long-lived browser sessions after redeployment
  Mitigation: Implement cache-busting strategies and version headers; document expected behavior for configuration changes; use runtime APIs for truly dynamic values
  Owner: Platform engineering team
- Missing NEXT_PUBLIC_ prefix causes undefined variables in production, leading to runtime errors
  Mitigation: Add TypeScript type definitions for expected environment variables; implement build-time validation that fails if required public variables are missing; use linting rules to detect process.env access without proper prefix
  Owner: Engineering team

## Implementation Notes

- Create .env.example file documenting all NEXT_PUBLIC_ variables with descriptions and example values
- Add TypeScript declarations (e.g., in next-env.d.ts or custom types file) for process.env.NEXT_PUBLIC_* to enable autocomplete and type checking
- Implement conditional rendering patterns (as seen in _document.tsx) to gracefully handle missing optional public variables
- Document the build-time inlining behavior in team wiki/README so developers understand that changes require rebuild

## Continuation Context


Verify commands:
- grep -r 'process\.env\.' apps/docs --include='*.tsx' --include='*.ts' | grep -v 'NEXT_PUBLIC_' | grep -v 'node_modules'
- grep -r 'NEXT_PUBLIC_.*\(PASSWORD\|SECRET\|KEY\|TOKEN\)' . --include='.env*' || echo 'No suspicious public secrets found'
- npm run build 2>&1 | grep -i 'environment variable' || echo 'Build completed without env warnings'

Accept when:
- All client-side environment variable references use NEXT_PUBLIC_ prefix
- No sensitive credential patterns (PASSWORD, SECRET, private API_KEY) appear with NEXT_PUBLIC_ prefix in environment files
- Build process completes successfully with all required public variables defined

## Enforcement

- Verified by: Pre-commit hooks scanning for process.env usage patterns in client-side code
- Verified by: CI pipeline checks validating NEXT_PUBLIC_ prefix usage and detecting secret patterns
- Verified by: Code review checklist items for environment variable additions
- Verified by: Build-time validation failing on missing required public variables
- Violation handling: CI build fails if client-side code references non-NEXT_PUBLIC_ environment variables
- Violation handling: Security scanning tools flag NEXT_PUBLIC_ variables containing secret-like patterns for manual review
- Violation handling: Pull requests blocked until environment variable usage follows prefix convention
- Violation handling: Runtime errors in development mode when accessing undefined environment variables
- Exception process: Document exception rationale in ADR or technical design document
- Exception process: Obtain security team approval for any NEXT_PUBLIC_ variable containing potentially sensitive data
- Exception process: Add inline code comments explaining why non-standard pattern is necessary
- Exception process: Create tracking ticket for future refactoring to standard pattern