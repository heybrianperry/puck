# Use process.env for Runtime Configuration in Client Components: Configuration Dependent Calls

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The ReleaseSwitcher component requires runtime configuration values (BASE_URL, IS_CANARY, IS_LATEST) that vary between deployment environments
- Next.js applications expose environment variables to the browser bundle through the NEXT_PUBLIC_ prefix convention
- Configuration values are accessed directly via process.env at component initialization time rather than through a centralized configuration service
- The component performs external API calls using fetch() that depend on these environment-sourced configuration values
- Error handling for failed API requests uses console.error for logging, establishing a pattern of direct console API usage for observability

## Problem Statement

Client-side React components need access to deployment-specific configuration values (base URLs, feature flags) at runtime without hardcoding environment-specific values or requiring a separate configuration management system, while maintaining the ability to log configuration-related errors during development and production.

## Decision

1. MUST: Configuration-dependent API calls MUST use environment variables to construct endpoint URLs rather than hardcoded values

## Policy Block

- MUST Configuration-dependent API calls MUST use environment variables to construct endpoint URLs rather than hardcoded values

In scope:
- Client-side React components in Next.js applications
- Components that make external API calls dependent on environment configuration
- UI components that require deployment-specific base URLs or feature flags
- Components within the apps/docs directory that interact with release APIs

Out of scope:
- Server-side API routes or middleware that can access all environment variables
- Build-time configuration that does not need runtime access
- Static configuration that does not vary between environments
- Backend services outside the Next.js application boundary

Exceptions:
- EXC-001: Configuration values are provided through a centralized configuration provider or context

## Rationale

- The evidence shows direct usage of process.env.NEXT_PUBLIC_* variables in a React component (ReleaseSwitcher), indicating a pattern of runtime configuration sourcing that aligns with Next.js conventions for client-side environment variable access
- The combination of runtime.config.sources and security.secrets_handling facets in the IR evidence demonstrates that environment variables are the primary mechanism for injecting deployment-specific configuration
- Console.error logging is used for error handling in configuration-dependent operations (fetch failures), establishing a lightweight observability pattern without additional logging infrastructure
- This pattern enables environment-agnostic component code that can be deployed across multiple environments (development, staging, production) without code changes

## Consequences

Positive:
- Components remain environment-agnostic and can be deployed to multiple environments without code modifications
- Next.js build process automatically inlines NEXT_PUBLIC_ variables, eliminating runtime configuration lookup overhead
- Simple console-based logging provides immediate visibility into configuration-related errors during development
- No additional dependencies or configuration management libraries required

Negative:
- Environment variables are embedded in the client bundle, making them visible to end users and potentially exposing non-sensitive configuration details
- Console.error logging lacks structured logging capabilities (log levels, metadata, aggregation) needed for production observability at scale
- Direct process.env access creates tight coupling between components and the Next.js environment variable convention
- No centralized configuration validation or type safety for environment variables

## Alternatives

- Implement a centralized configuration service that fetches configuration from a remote endpoint at application startup (rejected)
  Rejected because: Adds complexity, introduces additional network dependency, and increases initial load time. The current pattern of build-time environment variable injection is simpler and more performant for static configuration values.
  When valid: When configuration needs to change without redeployment or when configuration is user-specific rather than environment-specific
- Use React Context to provide configuration values throughout the component tree (rejected)
  Rejected because: Adds boilerplate for context provider setup without addressing the underlying need to source values from environment variables. Still requires process.env access at the provider level.
  When valid: When configuration needs to be computed, transformed, or validated before consumption, or when testing requires easy configuration mocking
- Adopt a structured logging library (e.g., winston, pino) for error logging instead of console.error (deferred)
  Rejected because: Not rejected, but deferred. Current console.error usage is sufficient for the observed error logging needs. Structured logging adds value at scale but introduces dependency overhead.
  When valid: When log aggregation, structured querying, or log levels become necessary for production observability requirements

## Risks

- Sensitive configuration values could be accidentally exposed in the client bundle if developers omit the NEXT_PUBLIC_ prefix check
  Mitigation: Implement linting rules to detect process.env access without NEXT_PUBLIC_ prefix in client components. Document the security boundary clearly in developer guidelines.
  Owner: Engineering team
- Console.error logs may be lost or difficult to aggregate in production environments without structured logging infrastructure
  Mitigation: Evaluate log aggregation needs as application scales. Consider integrating browser error tracking (e.g., Sentry) for production error monitoring.
  Owner: Platform team
- Type safety is not enforced for environment variables, leading to potential runtime errors if expected variables are undefined
  Mitigation: Create a typed configuration module that validates and exports environment variables with TypeScript types. Fail fast at build time if required variables are missing.
  Owner: Engineering team

## Implementation Notes

- Extract environment variable access into a dedicated configuration module (e.g., @/core/config) that exports typed constants and validates required variables at module initialization
- Use TypeScript const assertions or enums to provide type safety for configuration values accessed throughout the application
- Establish naming conventions for environment variables (NEXT_PUBLIC_<FEATURE>_<PROPERTY>) to improve discoverability and prevent naming conflicts
- Document which environment variables are required vs. optional, and provide sensible defaults where appropriate to prevent runtime undefined errors

## Continuation Context


Verify commands:
- grep -r "process\.env\.NEXT_PUBLIC_" apps/docs/components/ | grep -v "node_modules"
- grep -r "console\.error" apps/docs/components/ | grep -v "node_modules"
- test -f apps/docs/core/config.ts || echo 'Warning: No centralized config module found'

Accept when:
- All client-side environment variable access uses the NEXT_PUBLIC_ prefix pattern
- Error logging for configuration-dependent operations uses console.error with descriptive messages
- Configuration values are extracted into named constants rather than accessing process.env inline throughout component logic

## Enforcement

- Verified by: Code review checklist verifying NEXT_PUBLIC_ prefix usage in client components
- Verified by: ESLint rules detecting direct process.env access patterns
- Verified by: TypeScript compilation ensuring configuration constants are properly typed
- Violation handling: Code review feedback requesting refactoring to use NEXT_PUBLIC_ prefixed variables
- Violation handling: Build warnings for untyped environment variable access
- Violation handling: Documentation updates to clarify the configuration pattern for new team members
- Exception process: Exceptions require architecture review and documentation of alternative configuration approach
- Exception process: Server-side components and API routes are exempt from NEXT_PUBLIC_ requirement
- Exception process: Temporary exceptions during migration from legacy configuration patterns must include migration timeline