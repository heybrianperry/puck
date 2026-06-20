# Standardize External API Documentation with Embedded Analytics and Metadata: Public Facing Documentation

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- Public-facing API documentation and demo applications require consistent metadata management for SEO, analytics, and user tracking
- External APIs need standardized approaches to embedding third-party services (analytics, monitoring) without exposing sensitive credentials
- Documentation sites and demo applications serve as the primary interface for external developers and must maintain professional standards
- The pattern was detected across documentation theme configuration, document structure, and demo application layouts, indicating a cross-cutting concern
- Security considerations around secrets management in public-facing applications necessitate architectural guidance

## Problem Statement

Public-facing API documentation and demo applications lack a standardized approach for managing external service integrations (analytics, monitoring, SEO metadata) while maintaining security best practices for secrets management. This creates inconsistency in how external APIs are configured, how credentials are handled, and how metadata is structured across different public-facing surfaces.

## Decision

1. MUST: All public-facing API documentation and demo applications MUST include standardized metadata configuration (title, description, favicon, Open Graph tags)

## Policy Block

- MUST All public-facing API documentation and demo applications MUST include standardized metadata configuration (title, description, favicon, Open Graph tags)

In scope:
- Public-facing documentation websites
- Demo applications accessible to external users
- API reference documentation
- Developer portal applications
- Marketing and landing pages for APIs

Out of scope:
- Internal documentation systems
- Private admin dashboards
- Backend API implementations
- Internal monitoring tools
- Development-only prototypes

Exceptions:
- EXC-001: Non-sensitive public API keys that are explicitly designed for client-side use (e.g., Google Maps API keys with domain restrictions)

## Rationale

- The pattern was detected with 87.50% confidence across 3 files in documentation and demo contexts, indicating a consistent architectural approach
- Security facet (secrets_management) suggests this pattern addresses credential handling in public-facing applications
- Standardizing metadata and external service integration improves SEO, analytics consistency, and reduces security risks from credential exposure
- Consistent configuration patterns reduce cognitive load for developers working across multiple public-facing surfaces

## Consequences

Positive:
- Improved security posture by preventing accidental credential exposure in public repositories
- Consistent user experience across all public-facing API documentation and demo applications
- Better SEO and social media sharing through standardized metadata
- Simplified onboarding for developers working on public-facing applications
- Easier audit trail for external service integrations and their configurations

Negative:
- Additional configuration overhead for setting up environment variables for each deployment
- Potential complexity in managing multiple environment-specific configurations
- May require additional documentation for developers on how to configure external services locally
- Could slow down rapid prototyping if strict credential management is enforced too early

## Alternatives

- Hardcode all configuration including API keys directly in source code (rejected)
  Rejected because: Creates severe security risks by exposing credentials in version control and public repositories
  When valid: Never valid for production public-facing applications
- Use a centralized configuration service (e.g., AWS Secrets Manager, HashiCorp Vault) for all external service credentials (deferred)
  Rejected because: Adds infrastructure complexity and may be overkill for simple documentation sites
  When valid: Valid for large-scale enterprise deployments with many external integrations
- Implement per-application custom metadata and integration patterns without standardization (rejected)
  Rejected because: Creates inconsistency, increases maintenance burden, and makes it harder to enforce security policies
  When valid: Only when applications have truly unique requirements that cannot fit standard patterns

## Risks

- Developers may accidentally commit environment files (.env) containing secrets to version control
  Mitigation: Add .env files to .gitignore, implement pre-commit hooks to scan for secrets, use git-secrets or similar tools
  Owner: Security team and DevOps
- Inconsistent implementation of the pattern across teams leading to configuration drift
  Mitigation: Create reusable configuration templates, implement automated compliance checks in CI/CD, provide clear documentation and examples
  Owner: Platform team
- External service API keys may be exposed through client-side code inspection or network traffic
  Mitigation: Use domain-restricted API keys where possible, implement backend proxies for sensitive operations, regularly rotate credentials
  Owner: Security team

## Implementation Notes

- Create a shared configuration template or package that can be imported across documentation and demo applications
- Use framework-specific environment variable patterns (e.g., NEXT_PUBLIC_ prefix for Next.js, VITE_ for Vite) to control client-side exposure
- Implement a configuration validation layer that checks for required metadata and warns about missing environment variables
- Document the standard metadata schema and provide examples for common external service integrations (Google Analytics, Sentry, etc.)
- Consider using a configuration management library (e.g., dotenv, env-var) to provide type safety and validation for environment variables

## Continuation Context


Verify commands:
- grep -r "API_KEY\|SECRET\|TOKEN" --include="*.tsx" --include="*.ts" --include="*.jsx" --include="*.js" | grep -v "process.env" | grep -v "import.meta.env"
- find . -name "*.env" -not -path "*/node_modules/*" -not -path "*/.git/*" | xargs git check-ignore -v
- grep -r "<meta.*og:" --include="*.tsx" --include="*.html" | wc -l

Accept when:
- No hardcoded API keys, secrets, or tokens are found in source code (all use environment variables)
- All .env files are properly gitignored and not tracked in version control
- Public-facing applications include standard metadata tags (Open Graph, Twitter Card, etc.)
- External service integrations are conditionally loaded based on environment configuration

## Enforcement

- Verified by: Automated CI/CD pipeline checks using secret scanning tools (git-secrets, truffleHog)
- Verified by: Code review checklist requiring verification of environment variable usage
- Verified by: Static analysis tools configured to flag hardcoded credentials
- Verified by: Periodic security audits of public-facing applications
- Violation handling: CI/CD pipeline fails if secrets are detected in code
- Violation handling: Pull requests blocked until hardcoded credentials are removed
- Violation handling: Security team notified for manual review of violations
- Violation handling: Immediate credential rotation if secrets are exposed in version control
- Exception process: Submit exception request to security team with justification
- Exception process: Document why the credential is safe for public exposure (e.g., domain-restricted API key)
- Exception process: Obtain approval from both security team and technical lead
- Exception process: Add exception to allowlist with expiration date and review schedule