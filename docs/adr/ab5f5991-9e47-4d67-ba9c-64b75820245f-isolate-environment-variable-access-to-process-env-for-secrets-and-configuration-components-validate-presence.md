# Isolate Environment Variable Access to process.env for Secrets and Configuration: Components Validate Presence

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase accesses runtime configuration and secrets through process.env, specifically npm_config_user_agent and NODE_ENV variables across multiple packages
- Environment variables provide a standard mechanism for injecting configuration and secrets into Node.js applications without hardcoding sensitive values
- The pattern appears in both CLI tooling (create-puck-app) and core runtime components (Puck component), indicating system-wide reliance on environment-based configuration
- Direct process.env access creates coupling between application code and runtime environment, requiring consistent variable naming and availability across deployment contexts

## Problem Statement

Applications require access to runtime configuration and secrets without embedding sensitive values in source code or build artifacts. The codebase must coordinate configuration injection across CLI tools, core components, and runtime environments while maintaining separation between development and production contexts.

## Decision

1. SHOULD: Components SHOULD validate the presence and format of required environment variables and provide clear error messages when missing

## Policy Block

- SHOULD Components SHOULD validate the presence and format of required environment variables and provide clear error messages when missing

In scope:
- Configuration values that vary between deployment environments (development, staging, production)
- Secrets and credentials required for external service integration
- Runtime behavior flags such as NODE_ENV that control conditional logic
- User agent strings and package manager detection variables like npm_config_user_agent

Out of scope:
- Build-time constants that are identical across all environments
- Type definitions and interface contracts
- Static asset paths and resource identifiers
- Application logic that does not depend on deployment context

Exceptions:
- EXC-001: Test environments require mocking or stubbing process.env for deterministic behavior

## Rationale

- The evidence shows process.env.npm_config_user_agent and process.env.NODE_ENV accessed in packages/create-puck-app/index.js and packages/core/components/Puck/index.tsx, demonstrating a pattern of environment-based configuration
- Using process.env as the single source for runtime configuration enables deployment flexibility without requiring code changes or rebuilds
- This pattern aligns with twelve-factor app principles by separating configuration from code and supporting multiple deployment targets
- The 88.60% confidence across 2 files indicates consistent adoption of environment variable access for secrets handling

## Consequences

Positive:
- Secrets and configuration remain external to source code, reducing risk of accidental exposure in version control
- Applications can be deployed to different environments without code modification or recompilation
- Environment variables provide a standard interface recognized by container orchestration platforms and deployment tools
- Runtime configuration access enables conditional behavior based on NODE_ENV without build-time branching

Negative:
- Direct process.env access creates implicit dependencies that are not visible in type systems or module imports
- Missing or misconfigured environment variables may cause runtime failures that are not caught during development
- Environment variable naming collisions across different tools or frameworks can cause unexpected behavior
- Testing requires additional setup to mock or stub process.env values for deterministic test execution

## Alternatives

- Centralized configuration module that wraps process.env access and provides typed interfaces (rejected)
  Rejected because: Evidence shows direct process.env access pattern already established across packages without abstraction layer
  When valid: Valid for new projects or during major refactoring when introducing type-safe configuration management
- Build-time environment variable substitution using bundler plugins (rejected)
  Rejected because: Would require rebuilds for configuration changes and prevent runtime environment switching
  When valid: Valid for client-side code where process.env is not available and values must be embedded at build time
- Configuration files (JSON, YAML) loaded at runtime (rejected)
  Rejected because: Adds complexity of file management and does not align with container-native deployment patterns
  When valid: Valid for complex configuration schemas requiring validation or hierarchical structure

## Risks

- Unvalidated environment variable access may cause runtime failures with unclear error messages when variables are missing or malformed
  Mitigation: Implement validation at application startup that checks required environment variables and provides clear error messages
  Owner: Engineering team
- Accidental logging or exposure of secrets accessed from process.env in error handling or debugging code
  Mitigation: Implement code review checks and linting rules to detect console.log or error message patterns that may expose environment variables
  Owner: Security team
- Environment variable naming conflicts between application code and third-party dependencies
  Mitigation: Document all environment variables used by the application and use namespaced prefixes for application-specific variables
  Owner: Engineering team

## Implementation Notes

- Access process.env variables at component initialization or application startup rather than during module evaluation to support testing and mocking
- Document all required environment variables in README or deployment documentation with expected formats and example values
- Consider using dotenv or similar tools for local development to load environment variables from .env files
- Implement validation logic that checks for required environment variables and fails fast with clear error messages if missing

## Continuation Context


Verify commands:
- grep -r 'process\.env\.' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -v node_modules
- grep -r 'npm_config_user_agent\|NODE_ENV' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -v node_modules
- test -f .env.example && echo 'Environment variable documentation exists' || echo 'Missing .env.example'

Accept when:
- All process.env accesses are identified and documented in environment variable reference documentation
- No hardcoded secrets or credentials are found in source files
- Environment variable validation occurs at application startup with clear error messages for missing required variables

## Enforcement

- Verified by: Code review process checks for new process.env accesses and ensures documentation is updated
- Verified by: Static analysis tools scan for hardcoded secrets or credentials in source files
- Verified by: Integration tests verify application behavior with different environment variable configurations
- Violation handling: Pull requests containing hardcoded secrets are rejected and must be revised before merge
- Violation handling: Accidental secret commits trigger immediate credential rotation and security incident review
- Violation handling: Missing environment variable documentation results in pull request comments requesting updates
- Exception process: Exceptions for non-standard environment variable access patterns require architecture review approval
- Exception process: Document exception rationale in code comments and ADR amendments
- Exception process: Time-bound exceptions must include migration plan to standard pattern