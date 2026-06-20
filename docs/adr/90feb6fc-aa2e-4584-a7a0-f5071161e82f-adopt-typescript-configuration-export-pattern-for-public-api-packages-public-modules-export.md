# Adopt TypeScript Configuration Export Pattern for Public API Packages: Public Modules Export

Status: proposed
Date: 2025-01-03
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains 57 files exhibiting a consistent pattern for exposing public APIs through TypeScript configuration and build tooling
- Multiple packages (core, plugins, field adapters) require standardized export mechanisms to ensure consistent consumer experience
- Build configuration files (tsup.config.ts, next.config.js) and API route handlers demonstrate a pattern of explicit public interface definition
- The pattern appears across different framework integrations (Next.js, Remix, React Router) suggesting a framework-agnostic API contract approach
- Configuration-driven exports enable better tree-shaking, type safety, and versioning control for public-facing packages

## Problem Statement

Without a standardized approach to defining and exporting public APIs across packages, the codebase risks inconsistent consumer interfaces, poor tree-shaking, unclear API boundaries, and difficulty maintaining backward compatibility. The detection of this pattern across 57 files with 88.45% confidence indicates an emergent architectural decision that needs formal documentation to ensure continued consistency.

## Decision

1. MUST: Public API modules MUST export TypeScript type definitions alongside runtime code to ensure type safety for consumers

## Policy Block

- MUST Public API modules MUST export TypeScript type definitions alongside runtime code to ensure type safety for consumers

In scope:
- All packages under /packages/* intended for external consumption
- API route handlers in recipe/example applications that demonstrate public API usage
- Shared configuration packages (tsup-config, jest.config) that standardize build outputs
- Plugin packages that extend core functionality
- Field adapter packages that integrate with external services

Out of scope:
- Internal utility modules not exposed through package exports
- Test files and test utilities
- Development-only scripts and tooling
- Private implementation details within package internals
- Example/recipe application code not intended as reusable packages

## Rationale

- Pattern detected across 57 files with 88.45% confidence indicates this is an established architectural practice that has proven effective
- Explicit configuration-driven exports provide clear API boundaries, making it easier to maintain backward compatibility and version packages independently
- TypeScript configuration patterns enable superior developer experience through type safety, IDE autocomplete, and compile-time error detection
- Consistent export patterns across packages reduce cognitive load for both maintainers and consumers, establishing predictable integration patterns

## Consequences

Positive:
- Clear API boundaries make it easier to identify breaking changes and manage semantic versioning
- Improved tree-shaking and bundle optimization for consumers through explicit exports
- Enhanced type safety and developer experience through co-located TypeScript definitions
- Consistent patterns across packages reduce learning curve for new contributors and consumers
- Framework-agnostic core APIs enable broader ecosystem adoption

Negative:
- Additional configuration overhead for each new package requiring tsup.config.ts and export definitions
- Potential for configuration drift if shared build configs are not consistently applied
- Learning curve for contributors unfamiliar with advanced TypeScript module resolution and build tooling
- Risk of over-engineering simple packages that might not need complex export configurations

## Alternatives

- Use simple index.ts barrel exports without build configuration (rejected)
  Rejected because: Barrel exports without build optimization lead to poor tree-shaking, larger bundle sizes, and lack of control over public API surface. The pattern detection shows the codebase has moved beyond this approach.
  When valid: Only appropriate for internal packages with no external consumers or very simple single-file utilities
- Adopt a monolithic API package approach with all exports from a single package (rejected)
  Rejected because: Contradicts the detected pattern of modular packages (core, plugins, field adapters). Monolithic approach reduces flexibility and increases coupling between unrelated features.
  When valid: Could be considered for very small projects with tightly coupled functionality, but not suitable for this extensible architecture
- Use runtime API registration instead of static exports (deferred)
  When valid: May be appropriate for plugin systems requiring dynamic loading, but should complement rather than replace static exports for core APIs

## Risks

- Configuration complexity may deter external contributors from creating new packages or plugins
  Mitigation: Provide package scaffolding tools, comprehensive documentation, and shared configuration presets that reduce boilerplate
  Owner: engineering team
- Build configuration changes could inadvertently break consumer applications through changed export paths or missing type definitions
  Mitigation: Implement automated testing of package exports, use publint or similar tools in CI, and follow strict semantic versioning for any export changes
  Owner: engineering team
- Framework-specific optimizations may be missed by maintaining framework-agnostic APIs
  Mitigation: Provide framework-specific adapter packages that can leverage platform features while keeping core APIs portable
  Owner: engineering team

## Implementation Notes

- Use the shared tsup-config package as the foundation for all new public packages to ensure consistency
- Define package.json exports field with explicit entry points for main, types, and any conditional exports (import/require)
- Include both ESM and CJS outputs when necessary for broader compatibility, but prefer ESM as the primary format
- Document public API surface in README files and consider using API Extractor or similar tools to generate API documentation
- For API route handlers, follow the established naming pattern (api.*.tsx) to make external interfaces immediately recognizable
- Leverage TypeScript's declaration maps for better debugging experience in consumer projects

## Continuation Context


Verify commands:
- grep -r "tsup.config.ts" packages/*/tsup.config.ts | wc -l
- find packages -name 'package.json' -exec jq -e '.exports' {} \; 2>/dev/null | wc -l
- grep -r "export.*from" packages/*/index.ts | grep -v "// internal" | wc -l

Accept when:
- All public packages under /packages/* contain either tsup.config.ts or equivalent build configuration
- Package.json files for public packages define explicit exports field with type definitions
- API route handlers follow consistent naming conventions (api.* or routes/api/*) across recipe applications
- Shared build configuration (tsup-config) is used by at least 80% of public packages

## Enforcement

- Verified by: Automated CI checks using publint or package-lint to verify export configurations
- Verified by: Code review checklist requiring verification of tsup.config.ts and package.json exports for new packages
- Verified by: Automated tests that import packages and verify expected exports are available
- Verified by: TypeScript compilation checks ensuring type definitions are properly generated and exported
- Violation handling: CI pipeline fails if public packages lack proper export configuration
- Violation handling: Pull requests adding new packages must include build configuration or justification for exemption
- Violation handling: Quarterly audits identify packages not following the pattern and create remediation tickets
- Violation handling: Breaking changes to exports trigger major version bumps per semantic versioning
- Exception process: Exceptions require written justification in package README explaining why standard pattern doesn't apply
- Exception process: Architecture review approval needed for packages that deviate from standard export patterns
- Exception process: Temporary exceptions must include timeline for migration to standard pattern
- Exception process: All exceptions documented in central registry with review date