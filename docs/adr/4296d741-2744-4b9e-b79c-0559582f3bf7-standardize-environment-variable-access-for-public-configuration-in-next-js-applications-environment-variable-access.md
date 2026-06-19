# Standardize Environment Variable Access for Public Configuration in Next.js Applications: Environment Variable Access

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The Next.js documentation application requires runtime configuration for feature flags, deployment environment detection, and third-party service integration
- Public environment variables prefixed with NEXT_PUBLIC_ are exposed to the browser runtime and used for client-side feature detection and API endpoint configuration
- The application uses process.env to access configuration values including NEXT_PUBLIC_PLAUSIBLE_DATA_DOMAIN, NEXT_PUBLIC_BASE_URL, NEXT_PUBLIC_IS_CANARY, NEXT_PUBLIC_IS_LATEST, and VERCEL_GIT_COMMIT_REF
- Configuration values are accessed directly in React components, Next.js document customization, and build-time configuration files

## Problem Statement

The application needs a consistent approach to accessing environment variables for runtime configuration while maintaining clear boundaries between server-side secrets and client-exposed public configuration, ensuring that sensitive values are not inadvertently exposed to the browser bundle.

## Decision

1. SHOULD: Environment variable access in client components SHOULD be limited to NEXT_PUBLIC_ prefixed variables

## Policy Block

- SHOULD Environment variable access in client components SHOULD be limited to NEXT_PUBLIC_ prefixed variables

In scope:
- Next.js applications using environment variables for runtime configuration
- React components requiring feature flags or deployment environment detection
- Build-time configuration files (next.config.mjs, next.config.js)
- Custom Document and App components (_document.tsx, _app.tsx)

Out of scope:
- Server-side API routes with sensitive credentials
- Database connection strings and authentication tokens
- Third-party API keys that should remain server-side only
- Non-Next.js applications or frameworks with different environment variable conventions

Exceptions:
- EXC-001: Server-side only environment variables (without NEXT_PUBLIC_ prefix) may be accessed in API routes, getServerSideProps, getStaticProps, and middleware
- EXC-002: Build-time environment variables may be accessed in next.config.js/mjs for conditional build configuration

## Rationale

- The pattern is observed across 3 files with 86.17% confidence, showing consistent use of process.env for accessing NEXT_PUBLIC_ prefixed variables in client-side contexts
- Next.js automatically inlines NEXT_PUBLIC_ prefixed environment variables at build time, making them available in the browser while protecting server-side secrets
- Direct property access on process.env (rather than destructuring or dynamic access) enables Next.js static analysis and tree-shaking optimizations
- The evidence shows conditional rendering based on environment variable presence (e.g., Plausible analytics script), demonstrating the need for graceful handling of undefined values

## Consequences

Positive:
- Clear separation between client-exposed and server-only configuration through naming convention
- Build-time optimization through static analysis of environment variable access
- Reduced risk of accidentally exposing sensitive credentials to the browser bundle
- Consistent pattern for feature flags and deployment environment detection across the application

Negative:
- NEXT_PUBLIC_ prefixed variables are permanently embedded in the client bundle and cannot be changed without rebuilding
- Verbose syntax with repeated process.env.NEXT_PUBLIC_ prefixes throughout the codebase
- Potential for undefined values requiring defensive checks in components
- Limited runtime flexibility for client-side configuration compared to runtime API-based configuration

## Alternatives

- Use a centralized configuration module that imports and re-exports environment variables with type safety (rejected)
  Rejected because: Next.js static analysis requires direct process.env property access for build-time inlining; a centralized module would break this optimization
  When valid: Valid for server-side only configuration or when using runtime configuration APIs
- Use Next.js runtime configuration (publicRuntimeConfig) for client-side values (rejected)
  Rejected because: Runtime configuration is only available in pages and requires server-side rendering, reducing static optimization opportunities
  When valid: Valid when configuration must change without rebuilding or for server-rendered pages only
- Fetch configuration from an API endpoint at runtime (rejected)
  Rejected because: Adds network latency and complexity for simple feature flags; observed pattern uses build-time values for performance
  When valid: Valid for user-specific or frequently changing configuration that cannot be determined at build time

## Risks

- Developers may accidentally expose sensitive values by adding NEXT_PUBLIC_ prefix to secrets
  Mitigation: Implement pre-commit hooks and CI checks to scan for common secret patterns in NEXT_PUBLIC_ variables; document naming conventions clearly
  Owner: Security team and engineering team
- Environment variables may be undefined at runtime causing component errors or incorrect behavior
  Mitigation: Implement defensive checks for optional environment variables; use TypeScript to type environment variables; add runtime validation
  Owner: Engineering team
- Build-time inlining means configuration cannot be changed without rebuilding, reducing deployment flexibility
  Mitigation: Document which values are build-time vs runtime; use runtime APIs for truly dynamic configuration; implement feature flag service for frequently changing flags
  Owner: DevOps and engineering team

## Implementation Notes

- Create a TypeScript declaration file (e.g., env.d.ts) to type process.env with all expected NEXT_PUBLIC_ variables for IDE autocomplete and type safety
- For optional environment variables, use conditional rendering patterns like {process.env.NEXT_PUBLIC_VAR && <Component />} to handle undefined gracefully
- Document all environment variables in a .env.example file with descriptions and whether they are required or optional
- Consider using a validation library like zod or joi to validate environment variables at build time in next.config.js

## Continuation Context


Verify commands:
- grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs --include='*.tsx' --include='*.ts' --include='*.jsx' --include='*.js' | grep -v 'node_modules'
- grep -r 'process\.env\.[A-Z_]*[^N][^E][^X][^T]' apps/docs --include='*.tsx' --include='*.jsx' | grep -v 'node_modules' | grep -v '_document\|_app\|next\.config'
- test -f .env.example && echo 'Environment variables documented' || echo 'Missing .env.example'

Accept when:
- All client-side environment variable access uses NEXT_PUBLIC_ prefix and direct process.env property access
- Server-side only variables are accessed only in API routes, getServerSideProps, middleware, or configuration files
- Components handle undefined environment variables gracefully without runtime errors

## Enforcement

- Verified by: Code review checklist verifying NEXT_PUBLIC_ prefix usage for client-side variables
- Verified by: ESLint rules or custom linting to detect process.env access patterns
- Verified by: CI pipeline checks scanning for potential secret exposure in NEXT_PUBLIC_ variables
- Violation handling: CI build fails if server-side only variables are accessed in client components without NEXT_PUBLIC_ prefix
- Violation handling: Code review blocks merge if environment variables are not documented in .env.example
- Violation handling: Security scan alerts if common secret patterns are detected in NEXT_PUBLIC_ variable names
- Exception process: Document the exception in ADR or technical design document with security review
- Exception process: Add inline comments explaining why the exception is necessary and safe
- Exception process: Add suppression comments for linting rules with ticket reference for future remediation