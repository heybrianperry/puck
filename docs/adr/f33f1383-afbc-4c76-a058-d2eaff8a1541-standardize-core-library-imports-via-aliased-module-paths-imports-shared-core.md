# Standardize Core Library Imports via Aliased Module Paths: Imports Shared Core

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains multiple applications (demo, docs) that share common libraries and configuration modules, requiring a consistent import strategy across application boundaries
- Components and blocks within the demo application import from a core library using the @/core path alias, indicating a centralized module architecture for shared functionality
- Configuration files (eslint.config.mjs) reference a shared eslint-config-custom package, demonstrating cross-application configuration standardization
- The monorepo structure necessitates clear module boundaries between framework dependencies (React, Next.js), shared core libraries, and application-specific code
- Import patterns show consistent use of path aliases (@/core, @/core/types, @/core/lib, @/core/components) across 18 files, suggesting an established architectural convention

## Problem Statement

Without standardized import paths and module aliasing conventions, monorepo applications face increased coupling, brittle relative path dependencies, inconsistent module resolution across build tools, and difficulty refactoring shared library locations. The pattern addresses the need for stable, maintainable import contracts between applications and shared core libraries.

## Decision

1. MUST: All imports from shared core libraries MUST use path aliases (e.g., @/core) rather than relative paths when crossing application or package boundaries

## Policy Block

- MUST All imports from shared core libraries MUST use path aliases (e.g., @/core) rather than relative paths when crossing application or package boundaries

In scope:
- All TypeScript and JavaScript files within monorepo applications (apps/*)
- Imports of shared core libraries and utility modules
- Configuration files that reference shared packages (ESLint, TypeScript configs)
- Component files importing from centralized component libraries
- Build tool configurations (tsconfig.json, webpack, vite, etc.)

Out of scope:
- Imports within a single package or library (internal relative imports are acceptable)
- Third-party npm package imports
- Node.js built-in module imports
- Dynamic imports or require() statements for runtime module loading
- Test fixture or mock file imports within test directories

Exceptions:
- EXC-001: A component file is co-located with its styles or test files in the same directory
- EXC-002: Importing sibling components within the same feature directory structure

## Rationale

- Evidence shows 18 files consistently using @/core path aliases across demo application blocks and components, indicating an established pattern with high adoption
- The pattern enables refactoring of core library locations without updating import statements across all consuming applications
- Path aliases provide clear architectural boundaries between framework code (React, Next.js), shared libraries (@/core), and application-specific code
- Shared configuration packages (eslint-config-custom) demonstrate the value of named package references for cross-application consistency

## Consequences

Positive:
- Reduced coupling between applications and core library physical locations, enabling easier refactoring and reorganization
- Improved code readability with clear, consistent import paths that communicate architectural intent
- Simplified IDE autocomplete and navigation through stable, predictable import paths
- Easier onboarding for new developers who can quickly identify shared vs. application-specific code

Negative:
- Additional configuration overhead in build tools (TypeScript, bundlers) to maintain path alias mappings
- Potential confusion when debugging if source maps don't correctly resolve aliased paths
- Risk of alias conflicts if multiple applications define overlapping path aliases
- Increased complexity in tooling setup for new applications joining the monorepo

## Alternatives

- Use relative paths for all imports within the monorepo (rejected)
  Rejected because: Relative paths create brittle dependencies that break when files are moved, and obscure architectural boundaries between shared and application code
  When valid: Only valid for imports within a single package or co-located files in the same directory
- Publish core libraries as separate npm packages and import via package names (rejected)
  Rejected because: Adds significant overhead for local development, versioning, and publishing; path aliases provide similar benefits with lower friction
  When valid: Valid when core libraries need to be consumed by external projects outside the monorepo
- Use a single global path alias (e.g., @/) for all shared code without subdomain organization (rejected)
  Rejected because: Lacks clear organization and makes it difficult to distinguish between different types of shared code (types, utilities, components)
  When valid: Valid for small monorepos with minimal shared code

## Risks

- Path alias configurations may drift between build tools (TypeScript, Jest, Webpack), causing runtime or test failures
  Mitigation: Maintain a single source of truth for path aliases (e.g., tsconfig.json) and derive other tool configurations from it; add CI checks to verify consistency
  Owner: Platform Engineering Team
- Developers may create circular dependencies between core libraries and applications if boundaries are not clearly documented
  Mitigation: Implement dependency-cruiser or similar tools to detect and prevent circular imports; document core library boundaries in architecture documentation
  Owner: Engineering Team
- IDE support for path aliases may be inconsistent across different editors or require manual configuration
  Mitigation: Provide IDE configuration templates (e.g., .vscode/settings.json) in the repository; document setup steps in developer onboarding guides
  Owner: Developer Experience Team

## Implementation Notes

- Configure path aliases in tsconfig.json using the 'paths' field, mapping @/core/* to the core library source directory
- Ensure build tools (Next.js, Vite, Webpack) are configured to resolve the same path aliases, typically through their respective resolve.alias configurations
- Use ESLint plugins like eslint-plugin-import to enforce import ordering and detect unresolved path aliases during development
- Document the path alias conventions in the repository's architecture documentation or developer guide, including examples of correct usage

## Continuation Context


Verify commands:
- grep -r "from ['\"]@/core" apps/demo apps/docs | wc -l
- grep -r "from ['\"]\.\." apps/demo/config apps/docs/components | grep -v "styles.module.css" | wc -l
- find apps -name 'tsconfig.json' -exec grep -l '"@/core"' {} \;

Accept when:
- At least 80% of imports from core libraries use path aliases rather than relative paths
- All tsconfig.json files in the monorepo define consistent path alias mappings for @/core
- No relative path imports (../) are used to cross application or package boundaries

## Enforcement

- Verified by: ESLint rules checking for relative imports crossing package boundaries
- Verified by: Code review checklist items verifying path alias usage
- Verified by: CI pipeline checks using grep or custom scripts to detect relative path violations
- Violation handling: CI build fails if relative imports are detected crossing package boundaries
- Violation handling: Pull requests with violations are blocked until imports are corrected to use path aliases
- Violation handling: Automated suggestions or fixes provided via ESLint autofix where possible
- Exception process: Developer documents the exception rationale in a code comment explaining why path alias cannot be used
- Exception process: Tech lead reviews and approves the exception during code review
- Exception process: Exception is tracked in the architecture decision log with a reference to the specific file and justification