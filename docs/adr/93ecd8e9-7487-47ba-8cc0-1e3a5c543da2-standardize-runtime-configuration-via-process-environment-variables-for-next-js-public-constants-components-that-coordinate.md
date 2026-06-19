# Standardize Runtime Configuration via Process Environment Variables for Next.js Public Constants: Components That Coordinate

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The application requires runtime configuration values that differ across deployment environments (development, staging, production) without rebuilding the application bundle
- Next.js provides a mechanism for exposing server-side environment variables to the browser runtime through the NEXT_PUBLIC_ prefix convention
- The ReleaseSwitcher component needs to coordinate with external API endpoints and adapt behavior based on deployment context (base URL, canary status, latest version flag)
- Configuration sources must be accessible at component initialization time within React useEffect hooks to fetch release data from environment-specific endpoints

## Problem Statement

Internal API components require access to deployment-specific configuration at browser runtime to coordinate with backend services, but hardcoding these values prevents environment portability and violates separation of configuration from code. The system needs a standardized approach to inject runtime configuration that works within Next.js client-side rendering constraints while maintaining clear boundaries between build-time and runtime values.

## Decision

1. MUST: Components that coordinate with internal APIs MUST NOT hardcode environment-specific values such as base URLs, deployment flags, or version identifiers

## Policy Block

- MUST Components that coordinate with internal APIs MUST NOT hardcode environment-specific values such as base URLs, deployment flags, or version identifiers

In scope:
- Client-side React components that fetch data from internal APIs
- Next.js pages and components requiring deployment-specific configuration
- UI components that adapt behavior based on environment flags (canary, latest, staging)
- Fetch calls to internal API endpoints where base URL varies by environment

Out of scope:
- Server-side API routes that have direct access to all environment variables without NEXT_PUBLIC_ prefix
- Build-time configuration that does not need browser runtime access
- Secret values such as API keys or tokens (these must never use NEXT_PUBLIC_ prefix)
- Static configuration that is identical across all environments

Exceptions:
- EXC-001: Development and testing environments where hardcoded localhost URLs simplify local debugging

## Rationale

- The evidence shows process.env access for NEXT_PUBLIC_BASE_URL, NEXT_PUBLIC_IS_CANARY, and NEXT_PUBLIC_IS_LATEST in apps/docs/components/ReleaseSwitcher/index.tsx, demonstrating an established pattern for runtime configuration
- Next.js inlines NEXT_PUBLIC_ prefixed variables at build time, making them available in the browser while maintaining a clear security boundary that prevents accidental exposure of server-side secrets
- The ReleaseSwitcher component uses these configuration sources to construct fetch URLs (fetch(`${BASE_URL}/api/releases`)), showing direct coordination between runtime config and internal API calls
- This pattern separates deployment concerns from component logic, enabling the same build artifact to run in multiple environments with different configuration values

## Consequences

Positive:
- Single build artifact can be deployed across multiple environments (development, staging, production) by changing environment variables only
- Clear separation between configuration and code improves maintainability and reduces risk of environment-specific bugs
- Next.js build process automatically inlines NEXT_PUBLIC_ variables, eliminating runtime lookup overhead
- Configuration values are accessible in client-side code without additional API calls or state management complexity

Negative:
- NEXT_PUBLIC_ prefix requirement creates a naming convention that must be consistently followed, with silent failures if omitted
- Environment variables are inlined at build time, requiring rebuild and redeploy if configuration values change (not truly runtime-configurable)
- All NEXT_PUBLIC_ values are exposed in the browser bundle, creating risk if developers accidentally prefix sensitive values
- Debugging configuration issues requires understanding Next.js build-time inlining behavior, which is not immediately obvious

## Alternatives

- Fetch configuration from a dedicated /api/config endpoint at component mount time (rejected)
  Rejected because: Adds network latency and complexity for values that are static per deployment; requires additional API endpoint maintenance and error handling
  When valid: When configuration must change without redeployment, or when configuration values are user-specific rather than environment-specific
- Use a centralized configuration module that imports values from a config.ts file with environment-specific exports (rejected)
  Rejected because: Requires build-time branching logic or multiple config files; does not integrate with Next.js conventions and loses framework-provided security boundaries
  When valid: In non-Next.js React applications where custom configuration management is already established
- Inject configuration via window object from server-rendered HTML script tag (rejected)
  Rejected because: Bypasses Next.js type safety and build optimizations; creates global namespace pollution and complicates TypeScript integration
  When valid: When integrating with legacy systems that already use window-based configuration injection

## Risks

- Developers may accidentally prefix sensitive values (API keys, secrets) with NEXT_PUBLIC_, exposing them in the browser bundle
  Mitigation: Implement pre-commit hooks and CI checks that scan for common secret patterns in NEXT_PUBLIC_ variables; provide clear documentation and training on the security boundary
  Owner: Security team and engineering leads
- Missing or misconfigured NEXT_PUBLIC_ variables result in undefined values at runtime, causing silent failures or incorrect API endpoint construction
  Mitigation: Add runtime validation in components that checks for required configuration values and fails fast with clear error messages; include configuration validation in deployment smoke tests
  Owner: Engineering team
- Configuration changes require full rebuild and redeploy, preventing rapid hotfixes for configuration-only issues
  Mitigation: Document which values are truly environment-specific and consider hybrid approach where critical runtime-changeable values use API-based configuration
  Owner: DevOps and engineering team

## Implementation Notes

- Create a centralized constants file (e.g., @/core/lib/config.ts) that reads process.env.NEXT_PUBLIC_* values and exports typed constants, providing a single source of truth
- Add TypeScript type definitions for expected NEXT_PUBLIC_ variables in next-env.d.ts or a custom types file to enable IDE autocomplete and type checking
- Include runtime validation at application bootstrap that checks for required NEXT_PUBLIC_ variables and logs clear error messages if missing
- Document all NEXT_PUBLIC_ variables in a central README or .env.example file with descriptions of their purpose and expected values per environment

## Continuation Context


Verify commands:
- grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs/components/ | grep -v 'NEXT_PUBLIC_BASE_URL\|NEXT_PUBLIC_IS_CANARY\|NEXT_PUBLIC_IS_LATEST' && echo 'Found non-standard NEXT_PUBLIC_ usage' || echo 'OK'
- grep -r 'const.*=.*['"]http' apps/docs/components/ | grep -v 'process.env' && echo 'Found hardcoded URLs' || echo 'OK'
- test -f .env.example && grep -q 'NEXT_PUBLIC_BASE_URL' .env.example && echo 'Configuration documented' || echo 'Missing .env.example documentation'

Accept when:
- All client-side components that call internal APIs source base URLs and environment flags from process.env.NEXT_PUBLIC_* variables
- No hardcoded environment-specific URLs or flags exist in component code (excluding test files and documented exceptions)
- All NEXT_PUBLIC_ variables are documented in .env.example with descriptions and example values

## Enforcement

- Verified by: Automated grep-based checks in CI pipeline scanning for hardcoded URLs and non-standard process.env access patterns
- Verified by: Code review checklist item requiring verification that new components use centralized configuration constants
- Verified by: Pre-commit hooks that warn when NEXT_PUBLIC_ prefix is used with common secret-related variable names
- Violation handling: CI pipeline fails if hardcoded environment-specific URLs are detected in component code
- Violation handling: Code review blocks merge if configuration values are not sourced from process.env with proper NEXT_PUBLIC_ prefix
- Violation handling: Security scanning tools flag and require manual review for any NEXT_PUBLIC_ variables containing patterns matching secrets or keys
- Exception process: Developer documents exception rationale in code comments referencing this ADR and the specific exception ID (EXC-001)
- Exception process: Team lead or architect reviews and approves exception during code review, confirming it meets allowed_when criteria
- Exception process: Exception is logged in a central exceptions registry (wiki or documentation) with approval date and review timeline