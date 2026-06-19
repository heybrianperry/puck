# Isolate Environment Variables for Public Client Configuration in React Components: Components Not Access

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- React components in the docs application access runtime configuration through process.env variables prefixed with NEXT_PUBLIC_
- The ReleaseSwitcher component reads three environment variables (NEXT_PUBLIC_BASE_URL, NEXT_PUBLIC_IS_CANARY, NEXT_PUBLIC_IS_LATEST) directly within component scope
- Environment variables are consumed at component initialization time and used to construct API fetch URLs to external endpoints
- The pattern appears in a UI interaction context (useEffect hook) where configuration drives client-side HTTP requests
- Error handling via console.error indicates runtime failure scenarios when external API calls fail

## Problem Statement

React components require runtime configuration for API endpoints and feature flags, but direct access to process.env throughout component code creates coupling between environment configuration and UI logic, making it difficult to audit which secrets or configuration values are exposed to the client bundle and increasing the risk of accidentally exposing sensitive credentials.

## Decision

1. MUST_NOT: Components MUST NOT access process.env variables without the NEXT_PUBLIC_ prefix, as these are server-only and will be undefined in the browser

## Policy Block

- MUST_NOT Components MUST NOT access process.env variables without the NEXT_PUBLIC_ prefix, as these are server-only and will be undefined in the browser

In scope:
- Client-side React components in Next.js applications
- Environment variables used for public API endpoints, feature flags, and client configuration
- Runtime configuration accessed during component initialization or effect hooks
- Configuration values that drive fetch() calls to external services

Out of scope:
- Server-side API routes and getServerSideProps functions
- Backend service credentials and database connection strings
- Private API keys and authentication tokens
- Build-time configuration that does not need runtime access

Exceptions:
- EXC-001: Development and testing environments may use non-prefixed variables if they are explicitly mocked or stubbed in test setup

## Rationale

- The NEXT_PUBLIC_ prefix convention in Next.js provides an explicit security boundary between server-only and client-exposed configuration, preventing accidental credential leakage
- Direct process.env access in components creates a clear audit trail of which configuration values are bundled into the client JavaScript, supporting security reviews
- The pattern observed in ReleaseSwitcher demonstrates a common need for runtime configuration in UI components that make external API calls
- Centralizing environment variable access would improve maintainability but the current inline pattern is detectable and auditable through static analysis

## Consequences

Positive:
- Explicit NEXT_PUBLIC_ prefix makes it immediately clear which environment variables are exposed to the browser
- Static analysis tools can easily grep for process.env usage to audit client-side configuration access
- Next.js build process automatically inlines these values, eliminating runtime environment variable lookup overhead
- Pattern is self-documenting and follows framework conventions familiar to Next.js developers

Negative:
- Environment variables are scattered across component files rather than centralized, making it harder to get a complete inventory
- No runtime validation ensures required environment variables are present, leading to potential undefined behavior
- Direct process.env access in components creates tight coupling between configuration and UI logic
- Error messages using console.error may not be captured by production error monitoring systems

## Alternatives

- Centralize all environment variable access in a dedicated config module that exports typed configuration objects (rejected)
  Rejected because: Current pattern is already established across the codebase; migration would require refactoring all components without immediate security benefit
  When valid: Valid for new projects or during major refactoring efforts where centralized configuration improves type safety and testability
- Use Next.js runtime configuration (publicRuntimeConfig) instead of environment variables (rejected)
  Rejected because: publicRuntimeConfig is deprecated in Next.js 13+ in favor of NEXT_PUBLIC_ environment variables
  When valid: Only valid for legacy Next.js applications prior to version 13
- Pass configuration as props from server components or getServerSideProps (deferred)
  Rejected because: Would require significant architectural changes to prop-drill configuration through component trees
  When valid: Valid when migrating to Next.js App Router with React Server Components where configuration can be injected at the server boundary

## Risks

- Developers may accidentally prefix sensitive credentials with NEXT_PUBLIC_, exposing them to the client bundle
  Mitigation: Implement pre-commit hooks and CI checks that scan for common secret patterns (API_KEY, SECRET, PASSWORD) with NEXT_PUBLIC_ prefix and fail the build
  Owner: Security team and DevOps
- Missing environment variables at runtime cause undefined behavior without clear error messages
  Mitigation: Add startup validation that checks for required NEXT_PUBLIC_ variables and fails fast with descriptive errors during build or initialization
  Owner: Engineering team
- Scattered process.env access makes it difficult to audit the complete surface area of client-exposed configuration
  Mitigation: Maintain automated inventory of all process.env accesses using grep or AST analysis in CI, generating a configuration manifest for security review
  Owner: Engineering team and Security team

## Implementation Notes

- Use grep or AST-based linting to detect all process.env accesses in client components and verify NEXT_PUBLIC_ prefix usage
- Consider adding ESLint rules that enforce environment variable naming conventions and prevent access to non-prefixed variables in client code
- Document all NEXT_PUBLIC_ variables in a central README or .env.example file with descriptions of their purpose and expected values
- Implement runtime guards that check for undefined environment variables and provide meaningful error messages or fallback values

## Continuation Context


Verify commands:
- grep -r "process\.env\.NEXT_PUBLIC_" apps/docs/components/ | wc -l
- grep -r "process\.env\." apps/docs/components/ | grep -v "NEXT_PUBLIC_" | grep -v "node_modules" || echo 'No violations found'
- npm run lint -- --rule 'no-process-env: error' || echo 'Linting check complete'

Accept when:
- All process.env accesses in client components use the NEXT_PUBLIC_ prefix
- No server-only environment variables (without NEXT_PUBLIC_ prefix) are accessed from client component code
- Grep for process.env in components directory returns only NEXT_PUBLIC_ prefixed variables or returns zero non-compliant matches

## Enforcement

- Verified by: Static analysis via grep or ESLint scanning for process.env usage patterns in client component files
- Verified by: Code review checklist item to verify environment variable access follows NEXT_PUBLIC_ convention
- Verified by: CI pipeline checks that scan for secret patterns combined with NEXT_PUBLIC_ prefix
- Violation handling: CI build fails if non-NEXT_PUBLIC_ process.env access is detected in client component files
- Violation handling: Code review blocks merge if environment variables lack proper prefix or expose sensitive values
- Violation handling: Security audit flags any NEXT_PUBLIC_ variables that match secret patterns (KEY, SECRET, TOKEN, PASSWORD)
- Exception process: Document exception in component file with comment explaining why non-standard access is required
- Exception process: Obtain tech lead approval for any deviation from NEXT_PUBLIC_ convention
- Exception process: Add exception to linting ignore file with justification and expiration date for review