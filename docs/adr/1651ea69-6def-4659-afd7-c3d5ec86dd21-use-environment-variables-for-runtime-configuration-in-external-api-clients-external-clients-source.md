# Use Environment Variables for Runtime Configuration in External API Clients: External Clients Source

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase uses React components that make external API calls requiring runtime configuration for base URLs and deployment environment flags
- Configuration values must be available at both build time and runtime in a Next.js environment using the NEXT_PUBLIC_ prefix convention
- The ReleaseSwitcher component fetches release data from an external API endpoint constructed from environment variables
- Error handling via console.error indicates a need for observable failures when configuration or external dependencies fail
- The pattern separates configuration sources (process.env) from usage sites (fetch calls) to enable environment-specific deployments

## Problem Statement

External API clients in React components require runtime configuration that varies across deployment environments (development, staging, production, canary) without hardcoding URLs or flags, while maintaining type safety and providing clear error feedback when configuration is missing or API calls fail.

## Decision

1. MUST: External API clients MUST source base URLs from environment variables prefixed with NEXT_PUBLIC_ to ensure availability in browser runtime

## Policy Block

- MUST External API clients MUST source base URLs from environment variables prefixed with NEXT_PUBLIC_ to ensure availability in browser runtime

In scope:
- React components making external HTTP requests via fetch
- Configuration affecting client-side API endpoint construction
- Environment variables with NEXT_PUBLIC_ prefix in Next.js applications
- Error logging for external API failures

Out of scope:
- Server-side API routes or backend service configuration
- Environment variables without NEXT_PUBLIC_ prefix (server-only)
- Internal module imports or component-to-component communication
- Build-time configuration that does not affect runtime behavior

## Rationale

- The evidence shows process.env.NEXT_PUBLIC_BASE_URL being used to construct fetch URLs, demonstrating the pattern of environment-driven configuration for external clients
- Using NEXT_PUBLIC_ prefixed variables ensures configuration is available in the browser runtime while maintaining Next.js build-time replacement semantics
- Console.error logging provides observable failure modes for debugging configuration and network issues in production environments
- Separating configuration sources from usage enables the same component code to work across multiple deployment environments without modification

## Consequences

Positive:
- Components can be deployed to multiple environments without code changes, only environment variable updates
- Configuration is explicit and traceable through environment variable inspection
- Error logging provides clear diagnostic information when external dependencies fail
- Next.js build-time replacement of NEXT_PUBLIC_ variables enables static optimization while maintaining runtime flexibility

Negative:
- Missing or misconfigured environment variables only surface at runtime when API calls are made
- NEXT_PUBLIC_ prefix requirement increases verbosity and creates a naming convention dependency
- Console.error logging may not integrate with structured logging or monitoring systems
- Environment variable changes require rebuild and redeployment in Next.js static export scenarios

## Alternatives

- Hardcode API base URLs directly in fetch calls with conditional logic based on window.location (rejected)
  Rejected because: Creates tight coupling between code and deployment environments, requires code changes for new environments, and makes configuration opaque
  When valid: Only appropriate for single-environment applications with no deployment variation
- Use a centralized configuration service or API to fetch runtime configuration (rejected)
  Rejected because: Introduces circular dependency (need configuration to fetch configuration) and adds latency to application initialization
  When valid: Appropriate when configuration changes frequently without redeployment or requires dynamic updates
- Use Next.js runtime configuration (publicRuntimeConfig) instead of environment variables (rejected)
  Rejected because: Incompatible with static export and requires server-side rendering, reducing deployment flexibility
  When valid: Appropriate for server-rendered Next.js applications that never use static export

## Risks

- Missing NEXT_PUBLIC_ environment variables cause runtime failures that are not caught at build time
  Mitigation: Implement build-time validation script that checks for required environment variables and fails the build if missing
  Owner: engineering team
- Console.error logging may be insufficient for production monitoring and alerting
  Mitigation: Integrate structured error tracking service (e.g., Sentry) to capture and alert on API failures
  Owner: engineering team
- Environment variable values may be exposed in client-side bundle and browser DevTools
  Mitigation: Never use NEXT_PUBLIC_ prefix for sensitive values; document that these variables are public and should only contain non-sensitive configuration
  Owner: security team

## Implementation Notes

- Create a centralized configuration module that reads and validates all NEXT_PUBLIC_ variables at application initialization
- Use TypeScript to define types for configuration values and provide compile-time checking where possible
- Document all required NEXT_PUBLIC_ environment variables in .env.example with descriptions and example values
- Consider creating a custom hook (e.g., useApiClient) that encapsulates environment variable access and fetch logic with consistent error handling

## Continuation Context


Verify commands:
- grep -r 'process\.env\.NEXT_PUBLIC_' apps/docs/ | grep -v node_modules
- grep -r 'fetch.*BASE_URL' apps/docs/components/ | grep -v node_modules
- grep -r 'console\.error' apps/docs/components/ | grep 'Could not load'

Accept when:
- All external API fetch calls use environment variables for base URL construction
- All NEXT_PUBLIC_ variables are documented in .env.example with descriptions
- All external API failures are logged with console.error including error details

## Enforcement

- Verified by: Code review checklist requiring environment variable usage for external API clients
- Verified by: Automated grep-based checks in CI pipeline verifying NEXT_PUBLIC_ prefix usage
- Verified by: Runtime monitoring of console.error logs for API failure patterns
- Violation handling: CI pipeline fails if hardcoded URLs are detected in fetch calls
- Violation handling: Code review blocks merge if external API clients do not use environment variables
- Violation handling: Production monitoring alerts on repeated API failures indicating configuration issues
- Exception process: Document exception rationale in code comments explaining why hardcoded values are necessary
- Exception process: Obtain approval from tech lead for any external API client that does not follow environment variable pattern
- Exception process: Create tracking issue for technical debt if exception is temporary