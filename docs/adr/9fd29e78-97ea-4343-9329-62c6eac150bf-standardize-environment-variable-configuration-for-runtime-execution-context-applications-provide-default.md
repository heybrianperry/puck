# Standardize Environment Variable Configuration for Runtime Execution Context: Applications Provide Default

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all runtime environment and configuration management implementations across the codebase.

## Context

- The codebase exhibits a consistent pattern of environment-based configuration management across multiple framework implementations (Remix, Next.js) and execution contexts
- Pattern detected in 7 files with 90.81% confidence, spanning both server-side routes, API handlers, and client-side hooks, indicating a cross-cutting architectural concern
- Multiple recipe implementations (remix-ai, next-ai, remix) demonstrate parallel configuration approaches, suggesting an established architectural standard
- The pattern appears in both data access layers (page.server.ts, get-page.ts) and UI interaction layers (edit.tsx, use-sidebar-resize.ts), indicating environment configuration affects multiple architectural tiers
- The signature 505f23cebfad6620a1f286ffed928b1c represents a specific configuration management approach that has been consistently applied across the detected files

## Problem Statement

Applications require consistent, predictable access to runtime configuration across different execution environments (development, staging, production) and framework contexts (server-side, client-side, API routes). Without standardized environment configuration management, applications face configuration drift, security vulnerabilities from hardcoded values, and deployment inconsistencies that lead to runtime failures.

## Decision

1. MAY: Applications MAY provide default values for non-critical configuration parameters

## Policy Block

- MAY Applications MAY provide default values for non-critical configuration parameters

In scope:
- All server-side route handlers and API endpoints
- Data access layers and model files (*.server.ts)
- Application initialization and bootstrap code
- Client-side hooks that require runtime configuration
- Build-time configuration that affects runtime behavior

Out of scope:
- Static configuration that never changes across environments (e.g., UI constants, color schemes)
- Type definitions and interface declarations
- Test fixtures and mock data
- Documentation and example code

Exceptions:
- EXC-001: Development-only debugging utilities that require hardcoded values for local testing
- EXC-002: Framework-specific configuration files that require literal values (e.g., next.config.js publicRuntimeConfig)

## Rationale

- The pattern's 90.81% confidence across 7 files indicates this is an established, intentional architectural decision rather than accidental convergence
- Consistent environment configuration management enables seamless deployment across multiple environments without code changes, reducing deployment risk and configuration errors
- Separation of server-side and client-side configuration (evidenced by *.server.ts files) prevents accidental exposure of sensitive credentials or API keys to browser clients
- Centralized configuration management improves maintainability by providing a single source of truth for environment-dependent behavior, making it easier to audit and update configuration requirements

## Consequences

Positive:
- Improved security posture by preventing hardcoded credentials and enabling environment-specific secret management
- Simplified deployment processes with consistent configuration interfaces across all environments
- Enhanced testability through ability to inject test-specific configuration without modifying application code
- Better developer experience with clear separation between server-side and client-side configuration concerns

Negative:
- Increased complexity in local development setup requiring proper .env file management and documentation
- Potential runtime failures if environment variables are not properly validated at startup
- Additional boilerplate code required for configuration modules and type definitions
- Learning curve for developers unfamiliar with environment-based configuration patterns

## Alternatives

- Use configuration files (JSON/YAML) checked into version control for all environments (rejected)
  Rejected because: Configuration files in version control expose sensitive credentials and require code changes for environment-specific values, violating twelve-factor app principles
  When valid: Only appropriate for truly static, non-sensitive configuration that never varies by environment
- Implement a centralized configuration service (e.g., Consul, etcd) for runtime configuration (rejected)
  Rejected because: Adds significant infrastructure complexity and external dependencies; overkill for the current scale indicated by the pattern detection
  When valid: Consider for large-scale distributed systems with hundreds of services requiring dynamic configuration updates
- Use framework-specific configuration mechanisms without standardization across frameworks (rejected)
  Rejected because: Creates inconsistent patterns across the codebase, making it harder for developers to work across different framework implementations
  When valid: Only when framework-specific features provide critical functionality not available through standard environment variables

## Risks

- Missing or misconfigured environment variables in production causing runtime failures
  Mitigation: Implement startup validation that checks all required environment variables and fails fast with clear error messages. Use CI/CD pipeline checks to validate environment configuration before deployment.
  Owner: Engineering team and DevOps
- Accidental exposure of server-side environment variables to client-side code
  Mitigation: Enforce naming conventions (e.g., NEXT_PUBLIC_ prefix) and use framework-specific mechanisms to prevent server variables from being bundled in client code. Implement automated scanning in CI to detect violations.
  Owner: Security team and engineering team
- Configuration drift between environments leading to inconsistent behavior
  Mitigation: Maintain environment variable templates (.env.example) in version control, use infrastructure-as-code for environment provisioning, and implement automated configuration auditing
  Owner: DevOps and platform team

## Implementation Notes

- Create a centralized configuration module (e.g., config.server.ts) that validates and exports all environment variables with proper TypeScript types
- Use .env.example files to document all required and optional environment variables with descriptions and example values
- Implement framework-specific patterns: use *.server.ts suffix in Remix for server-only code, use NEXT_PUBLIC_ prefix in Next.js for client-accessible variables
- Add startup validation logic that checks for required environment variables and provides helpful error messages indicating which variables are missing and where they should be defined

## Continuation Context


Verify commands:
- grep -r 'process\.env\.' --include='*.ts' --include='*.tsx' | grep -v '\.server\.' | grep -v 'NEXT_PUBLIC_' | grep -v 'config\.' || echo 'No direct process.env access found outside config modules'
- find . -name '*.server.ts' -o -name 'config.ts' | xargs grep -l 'process\.env' | wc -l
- grep -r 'const.*=.*['\"]http' --include='*.ts' --include='*.tsx' | grep -v 'test' | grep -v 'example' || echo 'No hardcoded URLs found'

Accept when:
- All environment variable access is centralized in configuration modules or uses framework-specific safe patterns (*.server.ts, NEXT_PUBLIC_ prefix)
- No hardcoded URLs, API keys, or credentials are found in application code outside of example/test files
- Application startup includes validation that fails fast with clear error messages when required environment variables are missing

## Enforcement

- Verified by: Automated CI pipeline checks using grep patterns to detect direct process.env access outside configuration modules
- Verified by: Code review checklist requiring verification that new configuration is added to centralized config modules
- Verified by: Static analysis tools configured to flag hardcoded credentials or URLs
- Verified by: Pre-commit hooks that validate environment variable naming conventions
- Violation handling: CI pipeline fails if hardcoded credentials or direct process.env access is detected outside approved patterns
- Violation handling: Pull requests are blocked until configuration is moved to appropriate centralized modules
- Violation handling: Security scanning tools flag violations for immediate remediation
- Violation handling: Violations in production code trigger incident response and immediate hotfix deployment
- Exception process: Developer submits exception request with justification to tech lead and security team
- Exception process: Exception must document why standard pattern cannot be used and what compensating controls are in place
- Exception process: Approved exceptions are documented in code with EXC-XXX reference and expiration date for review
- Exception process: All exceptions are reviewed quarterly and must be re-approved or remediated