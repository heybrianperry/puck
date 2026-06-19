# Adopt Environment-Aware Configuration with Nextra and Vercel Integration: Configuration Files Read

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The documentation application uses Next.js with Nextra framework for static site generation and documentation rendering
- The application deploys to Vercel platform and requires environment-specific configuration to distinguish between production and canary builds
- Configuration sources include process.env variables, specifically VERCEL_GIT_COMMIT_REF and NEXT_PUBLIC_IS_CANARY for deployment context
- The next.config.mjs file serves as the central configuration point, coordinating runtime behavior based on Git branch and deployment environment

## Problem Statement

Documentation applications require environment-aware configuration to manage deployment variants (production vs. canary), integrate with hosting platform metadata (Vercel), and coordinate framework-specific settings (Nextra) while maintaining clear separation between build-time and runtime configuration sources.

## Decision

1. MUST: Configuration files MUST read environment variables from process.env for deployment-specific settings

## Policy Block

- MUST Configuration files MUST read environment variables from process.env for deployment-specific settings

## Rationale

- The evidence shows explicit usage of process.env.VERCEL_GIT_COMMIT_REF and process.env.NEXT_PUBLIC_IS_CANARY in next.config.mjs, indicating intentional environment-aware configuration
- Detection of nextra in package.json and next.config.mjs demonstrates framework integration at the configuration layer
- The pattern appears in apps/docs/next.config.mjs with 86.70% confidence, suggesting this is an established configuration approach for the documentation application
- Vercel-specific environment variables indicate tight coupling with the deployment platform for branch-based deployment strategies

## Consequences

Positive:
- Clear separation between production and canary deployments through environment variable detection
- Client-side code can access deployment context via NEXT_PUBLIC_IS_CANARY for conditional behavior
- Integration with Vercel platform provides automatic Git branch context without manual configuration
- Centralized configuration in next.config.mjs provides single point of control for environment-specific behavior

Negative:
- Tight coupling to Vercel platform through VERCEL_GIT_COMMIT_REF creates vendor lock-in
- Environment variable dependencies may complicate local development if not properly documented
- Client-side exposure of deployment metadata via NEXT_PUBLIC_ prefix increases bundle size and may leak deployment information
- Configuration complexity increases with multiple environment-specific variables requiring coordination

## Alternatives

- Use build-time configuration files (e.g., config.json) instead of environment variables (rejected)
  Rejected because: Build-time configuration files do not integrate with Vercel's automatic deployment metadata and require manual updates per deployment
  When valid: When deploying to platforms without automatic environment variable injection or when configuration needs version control
- Implement runtime configuration API endpoint that serves environment-specific settings (rejected)
  Rejected because: Adds unnecessary runtime overhead and latency for configuration that can be determined at build time
  When valid: When configuration must change without redeployment or when secrets cannot be embedded at build time
- Use feature flags service (e.g., LaunchDarkly) for canary detection (rejected)
  Rejected because: Introduces external dependency and complexity for simple branch-based deployment detection
  When valid: When fine-grained feature rollout control is required beyond simple canary/production split

## Risks

- Missing or misconfigured environment variables in non-Vercel environments cause runtime failures
  Mitigation: Implement fallback defaults and validation checks in next.config.mjs with clear error messages
  Owner: engineering team
- Client-side exposure of NEXT_PUBLIC_IS_CANARY may enable users to detect internal deployment strategies
  Mitigation: Document that this variable is intentionally public and does not expose sensitive information
  Owner: security team
- Vercel platform changes to VERCEL_GIT_COMMIT_REF variable naming or behavior break deployment detection
  Mitigation: Monitor Vercel changelog and implement defensive checks with logging for undefined variables
  Owner: platform team

## Implementation Notes

- Ensure next.config.mjs includes validation logic to check for required environment variables and provide meaningful error messages
- Document all environment variables in README or .env.example file with descriptions of their purpose and expected values
- Use TypeScript type definitions for process.env to catch missing variables at build time
- Implement local development setup that simulates Vercel environment variables for consistent testing

## Continuation Context


Verify commands:
- grep -r 'process\.env\.VERCEL_GIT_COMMIT_REF' apps/docs/next.config.mjs
- grep -r 'process\.env\.NEXT_PUBLIC_IS_CANARY' apps/docs/next.config.mjs
- grep -r '"nextra"' apps/docs/package.json

Accept when:
- All three grep commands return matches indicating the presence of required environment variables and Nextra dependency
- next.config.mjs file exists and contains configuration logic that reads from process.env
- package.json declares nextra as a dependency in the docs application

## Enforcement

- Verified by: Automated grep-based verification in CI pipeline checking for required environment variable usage
- Verified by: Build-time validation in next.config.mjs that fails if required variables are undefined
- Verified by: Code review checklist ensuring new configuration follows established patterns
- Violation handling: CI pipeline fails if verification commands do not find required patterns
- Violation handling: Build process terminates with clear error message if environment variables are missing
- Violation handling: Pull requests are blocked until configuration patterns are corrected
- Exception process: Document exception rationale in ADR amendment or configuration comments
- Exception process: Obtain approval from platform team for alternative configuration approaches
- Exception process: Update verification commands to accommodate approved exceptions