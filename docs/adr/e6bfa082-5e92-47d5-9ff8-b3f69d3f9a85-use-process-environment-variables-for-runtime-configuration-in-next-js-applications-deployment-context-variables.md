# Use Process Environment Variables for Runtime Configuration in Next.js Applications: Deployment Context Variables

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The Next.js documentation application requires runtime configuration to adapt behavior based on deployment environment and build context
- Vercel deployment platform provides environment variables at build time (VERCEL_GIT_COMMIT_REF) that indicate branch and deployment context
- Next.js applications distinguish between server-side and client-side environment variables using the NEXT_PUBLIC_ prefix convention
- The application needs to determine whether it is running a canary build to conditionally enable features or modify behavior
- Process environment variables provide a standard Node.js mechanism for injecting configuration without hardcoding values in source code

## Problem Statement

The Next.js documentation application must adapt its runtime behavior based on deployment context (branch, environment) and feature flags without hardcoding configuration values in source code, while maintaining clear separation between server-side and client-side accessible configuration.

## Decision

1. MUST: Deployment context variables (VERCEL_GIT_COMMIT_REF) MUST be accessed through process.env

## Policy Block

- MUST Deployment context variables (VERCEL_GIT_COMMIT_REF) MUST be accessed through process.env

In scope:
- Next.js configuration files (next.config.mjs, next.config.js)
- Runtime configuration accessed via process.env
- Environment variables provided by Vercel deployment platform
- Feature flags controlling application behavior
- Build-time and runtime environment detection

Out of scope:
- Static configuration values that never change across environments
- Configuration managed by external configuration services or APIs
- Secrets management systems (Vault, AWS Secrets Manager)
- Database connection strings managed outside environment variables

Exceptions:
- EXC-001: Local development requires hardcoded defaults when environment variables are not set

## Rationale

- Evidence shows process.env access for VERCEL_GIT_COMMIT_REF and NEXT_PUBLIC_IS_CANARY in next.config.mjs, demonstrating established pattern of environment-based configuration
- The NEXT_PUBLIC_ prefix pattern aligns with Next.js framework conventions for client-side environment variable exposure
- Using process.env enables deployment-time configuration injection without rebuilding application code, supporting continuous deployment workflows
- Separation of server-side and client-side configuration through naming conventions prevents accidental exposure of sensitive values

## Consequences

Positive:
- Configuration can be modified at deployment time without code changes or rebuilds
- Clear separation between server-side and client-side configuration reduces security risks
- Integration with Vercel platform environment variables enables branch-specific and environment-specific behavior
- Standard Node.js process.env pattern ensures compatibility with tooling and deployment platforms

Negative:
- Environment variables must be documented separately from code, increasing maintenance burden
- Missing or misconfigured environment variables can cause runtime failures that are not caught at build time
- Client-side environment variables are embedded in JavaScript bundles, increasing bundle size slightly
- Debugging configuration issues requires access to deployment environment settings

## Alternatives

- Use a centralized configuration service (e.g., AWS AppConfig, Consul) for runtime configuration (rejected)
  Rejected because: Adds external dependency and complexity for simple environment-based configuration needs; process.env is sufficient for current requirements
  When valid: When configuration needs to change dynamically at runtime without redeployment, or when configuration must be shared across multiple services
- Hardcode configuration values with build-time replacement using webpack DefinePlugin (rejected)
  Rejected because: Requires separate builds for each environment; prevents deployment-time configuration changes; reduces deployment flexibility
  When valid: When configuration is truly static and never varies across deployments of the same build artifact
- Load configuration from JSON or YAML files committed to repository (rejected)
  Rejected because: Requires committing environment-specific values to source control; complicates secret management; reduces deployment flexibility
  When valid: When configuration is non-sensitive and benefits from version control tracking and code review

## Risks

- Missing environment variables cause runtime failures that are not detected until deployment
  Mitigation: Implement environment variable validation at application startup; document required variables in README and deployment guides; use TypeScript or validation libraries to enforce presence of required variables
  Owner: Engineering team
- Accidental exposure of sensitive values through NEXT_PUBLIC_ prefix
  Mitigation: Establish code review checklist for environment variable usage; implement automated scanning for sensitive patterns in NEXT_PUBLIC_ variables; provide team training on Next.js environment variable conventions
  Owner: Security team and engineering team
- Environment variable configuration drift across deployment environments
  Mitigation: Use infrastructure-as-code (Terraform, Pulumi) to manage environment variables; maintain centralized documentation of required variables per environment; implement automated testing of configuration in staging environments
  Owner: DevOps team

## Implementation Notes

- Access environment variables in next.config.mjs using process.env.VARIABLE_NAME syntax
- Prefix client-side variables with NEXT_PUBLIC_ to enable access in browser JavaScript
- Document all required environment variables in README.md with descriptions and example values
- Consider implementing a configuration validation module that runs at application startup to fail fast on missing required variables
- Use TypeScript type definitions to provide autocomplete and type safety for environment variable access

## Continuation Context


Verify commands:
- grep -r 'process\.env\.' apps/docs/next.config.mjs
- grep -r 'NEXT_PUBLIC_' apps/docs/next.config.mjs
- grep -r 'VERCEL_GIT_COMMIT_REF' apps/docs/

Accept when:
- next.config.mjs contains process.env references for runtime configuration
- Client-side environment variables use NEXT_PUBLIC_ prefix
- Vercel platform variables (VERCEL_GIT_COMMIT_REF) are accessed via process.env

## Enforcement

- Verified by: Code review checklist verification of environment variable usage patterns
- Verified by: Automated grep-based scanning in CI pipeline for process.env and NEXT_PUBLIC_ patterns
- Verified by: Manual review of next.config.mjs changes in pull requests
- Violation handling: Pull requests with hardcoded configuration values are rejected during code review
- Violation handling: CI pipeline warnings for environment variables without NEXT_PUBLIC_ prefix used in client code
- Violation handling: Post-deployment monitoring alerts for missing environment variable errors
- Exception process: Document exception rationale in ADR or technical design document
- Exception process: Obtain approval from team lead or architect
- Exception process: Add inline code comments explaining why exception is necessary
- Exception process: Track exceptions in technical debt backlog for future remediation