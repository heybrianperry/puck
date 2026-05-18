# Standardize Environment Variable Access via process.env for Runtime Configuration: Applications Use Environment

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all runtime configuration and environment management implementations across the codebase.

## Context

- The codebase spans multiple runtime environments including Next.js, Remix, and shared core packages, each requiring consistent access to environment-specific configuration
- Configuration values such as API endpoints, feature flags, and runtime parameters need to be externalized from code to support different deployment environments (development, staging, production)
- The pattern was detected across 7 files with 90.81% confidence, indicating a well-established architectural practice for environment variable access
- Public API contracts (api.public.contracts facet) require stable, predictable configuration interfaces that can be relied upon by multiple consumers
- Server-side routes, data models, and utility functions all demonstrate consistent use of process.env for accessing runtime configuration

## Problem Statement

Applications need a standardized, type-safe, and predictable mechanism to access runtime configuration values that vary across deployment environments. Without a consistent approach, configuration access becomes fragmented, error-prone, and difficult to validate, leading to runtime failures, security vulnerabilities from hardcoded values, and maintenance challenges when configuration requirements change.

## Decision

1. MAY: Applications MAY use environment variable validation libraries (e.g., zod, envalid) to enforce configuration schemas

## Policy Block

- MAY Applications MAY use environment variable validation libraries (e.g., zod, envalid) to enforce configuration schemas

In scope:
- All server-side runtime configuration (API endpoints, database URLs, service credentials)
- Feature flags and environment-specific behavior toggles
- Build-time configuration that varies across deployment targets
- Third-party service integration parameters (API keys, webhook URLs)
- Application-level settings (port numbers, timeouts, resource limits)

Out of scope:
- Static configuration that never changes across environments (e.g., application constants, algorithm parameters)
- Client-side only configuration in browser environments where process.env is not available
- Configuration loaded from external configuration services at runtime (though initial service URLs may still use process.env)
- Hardcoded values in test fixtures and mock data for unit tests

Exceptions:
- EXC-001: Client-side code in Next.js may access NEXT_PUBLIC_* prefixed environment variables that are explicitly intended for browser exposure
- EXC-002: Test environments may use hardcoded configuration values when testing configuration validation logic itself

## Rationale

- The pattern appears consistently across 7 files spanning multiple frameworks (Next.js, Remix) and layers (routes, models, utilities), indicating this is a proven architectural practice
- Using process.env provides a standard Node.js interface that works across all JavaScript/TypeScript runtime environments and integrates with deployment platforms (Docker, Kubernetes, serverless)
- Externalizing configuration via environment variables follows the Twelve-Factor App methodology, enabling the same codebase to be deployed across multiple environments without code changes
- The 90.81% confidence score and presence in public API contracts suggests this pattern is intentional and well-established rather than accidental

## Consequences

Positive:
- Configuration can be changed without code modifications or redeployment, enabling rapid environment-specific adjustments
- Secrets and sensitive values remain external to the codebase, improving security posture and enabling secret rotation
- The same application build can be promoted through multiple environments (dev → staging → prod) with only configuration changes
- Integration with container orchestration and CI/CD pipelines is simplified through standard environment variable injection mechanisms
- Local development is streamlined through .env files while production uses platform-native secret management

Negative:
- Runtime configuration errors may not be detected until application startup or first use, potentially causing production incidents
- Environment variable management becomes more complex as the number of configuration values grows, requiring careful documentation
- Type safety is lost at the process.env boundary unless additional validation layers are implemented
- Debugging configuration issues requires access to deployment environment settings, which may be restricted in production

## Alternatives

- Use configuration files (JSON, YAML, TOML) loaded at runtime with environment-specific overrides (rejected)
  Rejected because: Configuration files require file system access and deployment of multiple configuration artifacts, complicating containerized deployments and violating Twelve-Factor principles. Environment variables are more universally supported across deployment platforms.
  When valid: May be appropriate for complex configuration schemas with nested structures that are difficult to express in flat environment variables
- Use a centralized configuration service (e.g., Consul, etcd, AWS Parameter Store) with runtime fetching (rejected)
  Rejected because: Adds infrastructure complexity, external dependencies, and potential points of failure. The pattern evidence shows direct process.env access is sufficient for current needs.
  When valid: Appropriate for microservices architectures requiring dynamic configuration updates without restarts or when configuration needs to be shared across many services
- Hardcode configuration with build-time replacement via webpack DefinePlugin or similar (rejected)
  Rejected because: Requires separate builds for each environment, preventing build-once-deploy-many patterns and making emergency configuration changes require full rebuild and redeploy cycles.
  When valid: May be acceptable for truly static values that never change or for optimizing client-side bundle size by eliminating unused code paths

## Risks

- Missing or misconfigured environment variables cause runtime failures that are not caught during build or deployment
  Mitigation: Implement startup validation that checks for required environment variables and fails fast with clear error messages. Use TypeScript types and validation libraries (zod, envalid) to enforce configuration schemas.
  Owner: Engineering team
- Sensitive environment variables may be accidentally logged or exposed in error messages
  Mitigation: Implement logging sanitization that redacts known sensitive variable names. Use structured logging with explicit field inclusion rather than dumping entire environment. Conduct security reviews of error handling code.
  Owner: Security team
- Environment variable sprawl makes configuration management difficult as the application grows
  Mitigation: Maintain comprehensive .env.example files documenting all variables. Use consistent naming conventions with prefixes. Consider grouping related configuration into namespaced variables (e.g., DATABASE_*, API_*).
  Owner: Engineering team

## Implementation Notes

- Create a centralized configuration module that reads from process.env and exports typed configuration objects, providing a single source of truth for configuration access
- Use .env.example files in version control to document all required and optional environment variables with example values and descriptions
- Implement environment variable validation at application startup using libraries like zod or envalid to define schemas and fail fast on misconfiguration
- For Next.js applications, use NEXT_PUBLIC_* prefix only for variables that must be available in the browser, and document the security implications
- Consider using dotenv or similar libraries for local development to load .env files, while relying on platform-native environment variable injection in production

## Continuation Context


Verify commands:
- grep -r 'process\.env\.' --include='*.ts' --include='*.tsx' --include='*.js' --include='*.jsx' . | wc -l
- grep -r 'const.*=.*['"]http' --include='*.ts' --include='*.tsx' | grep -v process.env | grep -v '//.*http' | wc -l
- test -f .env.example && echo 'Configuration documentation exists' || echo 'Missing .env.example'

Accept when:
- All runtime configuration values are accessed via process.env rather than hardcoded strings or imported constants
- No hardcoded URLs, API keys, or environment-specific values appear in source code outside of test fixtures
- A .env.example file exists documenting all required environment variables with descriptions

## Enforcement

- Verified by: Automated code review checks scanning for hardcoded configuration values (URLs, API keys, environment-specific strings)
- Verified by: CI pipeline validation ensuring .env.example is kept up to date with actual environment variable usage
- Verified by: Manual code review focusing on new configuration additions and their documentation
- Violation handling: CI build warnings for detected hardcoded configuration values, with failure on sensitive patterns (API keys, passwords)
- Violation handling: Pull request comments automatically generated for configuration-related changes missing .env.example updates
- Violation handling: Security scanning tools flag potential credential exposure in code
- Exception process: Document exception rationale in code comments explaining why hardcoded value is necessary
- Exception process: Obtain tech lead approval for exceptions involving any sensitive or environment-specific data
- Exception process: Add exception to allowlist in automated scanning tools with expiration date for review