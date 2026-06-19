# Use NEXT_PUBLIC_ Prefixed Environment Variables for Client-Side Runtime Configuration: Sensitive Credentials Keys

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The Next.js documentation application requires runtime configuration values to be accessible in client-side code for features like analytics integration and release switching
- Environment variables prefixed with NEXT_PUBLIC_ are exposed to the browser bundle by Next.js, enabling client-side components to access configuration without server-side rendering
- The application uses process.env.NEXT_PUBLIC_PLAUSIBLE_DATA_DOMAIN for analytics tracking and process.env.NEXT_PUBLIC_BASE_URL, NEXT_PUBLIC_IS_CANARY, NEXT_PUBLIC_IS_LATEST for release management
- Client-side configuration is required in both Document components (_document.tsx) and interactive UI components (ReleaseSwitcher) that execute in the browser

## Problem Statement

Client-side React components in Next.js applications need access to runtime configuration values for external service integration and feature flags, but standard environment variables are not available in browser contexts. A consistent mechanism is needed to safely expose configuration to client-side code while maintaining clear boundaries between server-only and client-accessible values.

## Decision

1. MUST_NOT: Sensitive credentials, API keys, or server-only secrets MUST NOT use the NEXT_PUBLIC_ prefix

## Policy Block

- MUST_NOT Sensitive credentials, API keys, or server-only secrets MUST NOT use the NEXT_PUBLIC_ prefix

In scope:
- Client-side React components requiring runtime configuration
- Browser-executed code in Next.js applications
- Public feature flags and non-sensitive service endpoints
- Analytics and telemetry configuration
- UI behavior switches (canary flags, version indicators)

Out of scope:
- Server-side API routes and getServerSideProps functions
- Database connection strings and credentials
- Private API keys and authentication tokens
- Build-time configuration that does not change at runtime
- Backend service-to-service communication secrets

## Rationale

- The evidence shows consistent use of NEXT_PUBLIC_ prefixed variables across both document-level (_document.tsx) and component-level (ReleaseSwitcher) code, indicating an established pattern for client-side configuration
- Next.js framework convention requires the NEXT_PUBLIC_ prefix to expose environment variables to the browser bundle, making this pattern necessary for client-side runtime configuration
- The pattern separates public client-accessible configuration from server-only secrets, with evidence showing usage for non-sensitive values like analytics domains and base URLs
- Two distinct use cases are observed: conditional script injection based on configuration (Plausible analytics) and API endpoint construction for client-side fetching (releases API)

## Consequences

Positive:
- Clear naming convention makes it immediately obvious which environment variables are exposed to client-side code
- Enables dynamic runtime configuration without rebuilding the application for different environments
- Supports conditional feature enablement in client-side code (e.g., analytics only when configured)
- Maintains separation between public and private configuration through explicit naming

Negative:
- All NEXT_PUBLIC_ variables are embedded in the client bundle, increasing bundle size proportionally to configuration volume
- Values are visible in browser DevTools and page source, requiring careful review to prevent accidental secret exposure
- Changes to NEXT_PUBLIC_ variables require application rebuild, limiting true runtime reconfiguration
- Developers must remember the prefix convention to avoid accidentally exposing server-only values

## Alternatives

- Use server-side API routes to proxy configuration to client components (rejected)
  Rejected because: Adds unnecessary network overhead and latency for static configuration values that are known at build time and do not contain secrets
  When valid: When configuration values must be dynamically computed per-request or contain sensitive data that requires server-side filtering
- Inject configuration through custom _app.tsx context provider from getInitialProps (rejected)
  Rejected because: Increases complexity and disables automatic static optimization in Next.js, negating performance benefits
  When valid: When configuration must be fetched from external sources at request time rather than build time
- Use runtime configuration with publicRuntimeConfig in next.config.js (rejected)
  Rejected because: publicRuntimeConfig is deprecated in Next.js 12.1+ in favor of NEXT_PUBLIC_ environment variables
  When valid: Only for legacy Next.js applications prior to version 12.1

## Risks

- Accidental exposure of sensitive credentials through NEXT_PUBLIC_ prefix misuse
  Mitigation: Implement pre-commit hooks and CI checks to scan for common secret patterns in NEXT_PUBLIC_ variables; conduct code review training on the security implications of client-side configuration
  Owner: Security team and engineering leads
- Configuration drift between environments if NEXT_PUBLIC_ variables are not properly managed in deployment pipelines
  Mitigation: Use environment-specific .env files with version control for non-sensitive defaults; document all required NEXT_PUBLIC_ variables in deployment runbooks
  Owner: DevOps team
- Runtime errors in client components when expected NEXT_PUBLIC_ variables are undefined
  Mitigation: Implement defensive checks (conditional rendering) before accessing environment variables; add TypeScript declarations for expected environment variables
  Owner: Engineering team

## Implementation Notes

- Define all NEXT_PUBLIC_ variables in .env.local for local development and document them in .env.example with descriptions
- Use TypeScript module augmentation to declare process.env types for NEXT_PUBLIC_ variables, enabling IDE autocomplete and type checking
- Implement conditional logic (if checks) before using NEXT_PUBLIC_ variables to gracefully handle undefined values, as seen in the Plausible analytics integration
- For API endpoints constructed from NEXT_PUBLIC_BASE_URL, provide fallback to relative paths or document the variable as required for production builds

## Continuation Context


Verify commands:
- grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs --include='*.tsx' --include='*.ts' | grep -v 'NEXT_PUBLIC_'
- grep -r 'process\.env\.[A-Z_]*[^N][^E][^X][^T]' apps/docs --include='*.tsx' --include='*.jsx' | grep -v node_modules
- npm run build 2>&1 | grep -i 'environment variable'

Accept when:
- All client-side environment variable references use the NEXT_PUBLIC_ prefix pattern
- No server-only secrets or credentials are prefixed with NEXT_PUBLIC_
- Build process completes without warnings about missing or misconfigured environment variables
- Client-side code includes conditional checks before accessing NEXT_PUBLIC_ variables

## Enforcement

- Verified by: Code review checklist requiring verification of NEXT_PUBLIC_ prefix usage
- Verified by: CI pipeline grep checks scanning for environment variable patterns in client-side code
- Verified by: Security scanning tools checking for credential patterns in NEXT_PUBLIC_ variables
- Verified by: Build-time Next.js warnings for undefined NEXT_PUBLIC_ variables
- Violation handling: Pull requests with client-side process.env access without NEXT_PUBLIC_ prefix are blocked pending correction
- Violation handling: Security scans detecting secrets in NEXT_PUBLIC_ variables trigger immediate incident response
- Violation handling: Build failures from missing NEXT_PUBLIC_ variables block deployment until configuration is provided
- Violation handling: Code review findings are documented and require resolution before merge
- Exception process: Exceptions for alternative configuration patterns require architecture review approval
- Exception process: Server-side rendering exceptions where process.env is accessed in getServerSideProps or API routes are permitted
- Exception process: Legacy code migration plans may temporarily exempt files with documented refactoring tickets
- Exception process: Exception requests must document the specific technical constraint preventing NEXT_PUBLIC_ usage