# Adopt Remix Route-Based Configuration Pattern for Service Boundaries: Environment Specific Configuration

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains multiple Remix applications (remix-ai and remix) that share similar routing patterns and service boundary definitions
- Route-based configuration in Remix applications provides a natural boundary for service definitions and environment-specific behavior
- The pattern appears in edit.tsx routes across different Remix variants, suggesting a standardized approach to handling edit operations with specific configuration requirements
- Service boundaries need to be clearly defined at the route level to ensure proper isolation and configuration management across different application contexts

## Problem Statement

When building Remix applications with multiple variants or contexts (e.g., AI-enhanced vs standard), there is a need to establish consistent service boundary definitions at the route level while maintaining environment-specific configurations. Without a standardized approach, route handlers may inconsistently manage service dependencies, environment variables, and configuration boundaries, leading to maintenance challenges and potential runtime errors.

## Decision

1. MUST: Environment-specific configuration MUST be isolated within route boundaries and not leak across route modules

## Policy Block

- MUST Environment-specific configuration MUST be isolated within route boundaries and not leak across route modules

In scope:
- All Remix route modules (app/routes/**/*.tsx)
- Loader and action functions within route modules
- Service configuration objects defined at route level
- Environment variable access within route boundaries

Out of scope:
- Shared utility functions outside route modules
- Global application configuration (root.tsx)
- Build-time configuration and bundler settings
- Static asset configuration

Exceptions:
- EXC-001: Legacy routes being migrated to the new pattern
- EXC-002: Third-party library integration requires global configuration access

## Rationale

- The pattern was detected across 2 files with 91% confidence, indicating a deliberate architectural choice for managing service boundaries in Remix applications
- Route-based service boundaries align with Remix's file-system routing paradigm and provide natural isolation points for configuration management
- Consistent service boundary definitions across application variants (remix-ai and remix) enable code reuse while maintaining flexibility for variant-specific requirements
- Encapsulating configuration at the route level improves testability, reduces coupling, and makes environment-specific behavior more explicit and maintainable

## Consequences

Positive:
- Clear service boundaries at the route level improve code organization and make dependencies explicit
- Environment-specific configuration is isolated and easier to test in isolation
- Route modules become more portable and reusable across different application contexts
- Reduced risk of configuration leakage and unintended side effects between routes

Negative:
- May introduce some duplication of configuration logic across similar routes
- Requires discipline to maintain consistent patterns across all route modules
- Initial migration effort for existing routes that don't follow this pattern
- Potential performance overhead if configuration objects are recreated on every request without proper caching

## Alternatives

- Use global singleton configuration accessed by all routes (rejected)
  Rejected because: Global state makes testing difficult, creates tight coupling between routes, and makes environment-specific behavior implicit and harder to reason about
  When valid: Only appropriate for truly global, immutable configuration that never varies by route or context
- Use React Context providers at the root level for all configuration (rejected)
  Rejected because: Doesn't leverage Remix's loader pattern for server-side configuration, and makes it harder to optimize data loading and caching
  When valid: Acceptable for client-side only configuration that doesn't affect server-side rendering or data loading
- Use environment variables directly in route components (rejected)
  Rejected because: Exposes server-side environment variables to the client, creates security risks, and makes configuration boundaries unclear
  When valid: Never valid for server-side configuration; only acceptable for public client-side configuration with VITE_PUBLIC_ prefix

## Risks

- Inconsistent implementation across routes as team grows or new developers join
  Mitigation: Create route templates, linting rules, and code review checklists to enforce the pattern. Document examples in developer onboarding materials.
  Owner: Engineering team lead
- Performance degradation if configuration objects are recreated unnecessarily on every request
  Mitigation: Implement caching strategies for configuration objects, use memoization in loaders, and monitor route performance metrics
  Owner: Performance engineering team
- Configuration drift between application variants (remix-ai vs remix) leading to subtle bugs
  Mitigation: Establish shared configuration schemas, use TypeScript types to enforce consistency, and implement integration tests that verify configuration behavior across variants
  Owner: QA and architecture team

## Implementation Notes

- Create a shared configuration utility module that provides type-safe configuration builders for common service boundaries
- Use TypeScript interfaces to define configuration shapes and ensure consistency across routes
- Implement loader functions that return configuration objects alongside data, making dependencies explicit in the route's data contract
- Consider using Remix's context API for passing configuration from loaders to components when needed, but keep the source of truth in the loader
- Document the pattern with examples in the project's architecture documentation and create route templates for common scenarios

## Continuation Context


Verify commands:
- grep -r 'export.*loader' app/routes/ | xargs -I {} sh -c 'grep -L "config" {} || echo "Missing config in: {}"'
- find app/routes -name '*.tsx' -exec grep -l 'process\.env\.' {} \; | grep -v 'loader\|action'
- npm run type-check && npm run lint -- --rule 'no-process-env: error' app/routes/

Accept when:
- All route modules with service dependencies define configuration objects within loader or action functions
- No direct process.env access exists in route component code (only in loaders/actions)
- TypeScript compilation succeeds with strict mode enabled for all route modules
- Code review checklist confirms service boundaries are explicitly defined for new routes

## Enforcement

- Verified by: ESLint rules preventing direct environment variable access in components
- Verified by: TypeScript strict mode compilation checks
- Verified by: Code review checklist items for route module structure
- Verified by: Automated tests verifying configuration isolation between routes
- Violation handling: CI pipeline fails if ESLint rules are violated
- Violation handling: Pull requests blocked until code review checklist is completed
- Violation handling: Architecture review required for any exceptions to the pattern
- Violation handling: Quarterly audits of route modules to identify and remediate violations
- Exception process: Developer documents exception rationale in route module comments
- Exception process: Tech lead reviews and approves exception with documented justification
- Exception process: Exception is logged in architecture decision log with timeline for remediation
- Exception process: Exceptions are reviewed quarterly and must be re-approved or remediated