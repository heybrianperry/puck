# Adopt Modular Library Architecture with Core Package Pattern: Data Manipulation Utilities

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The codebase exhibits a consistent pattern of organizing functionality into a core package structure with modular library components, as evidenced by 51 files following this pattern with 87.69% confidence
- The pattern is concentrated in packages/core/lib and packages/core/components, indicating a deliberate architectural decision to separate core functionality from application-specific code
- Multiple utility modules (flatten-data, populate-ids, use-preview-mode-hotkeys) and reusable components (Puck, LayerTree, OutlineList, ViewportControls) demonstrate a library-first approach to code organization
- The presence of specialized build configuration (packages/tsup-config/react-import.js) and server-side rendering components (SlotRender/server.tsx) indicates support for multiple deployment contexts
- The architecture supports both client-side and server-side usage patterns, with clear separation between presentation components and data manipulation utilities

## Problem Statement

As the application grows in complexity, there is a need to maintain code reusability, enforce clear boundaries between core functionality and application logic, and enable independent testing and versioning of library components. Without a standardized modular library architecture, code duplication increases, maintenance becomes difficult, and the ability to share functionality across different parts of the application or external projects is compromised.

## Decision

1. MUST: Data manipulation utilities MUST be placed in packages/core/lib with clear, single-purpose module names (e.g., flatten-data.ts, populate-ids.ts)

## Policy Block

- MUST Data manipulation utilities MUST be placed in packages/core/lib with clear, single-purpose module names (e.g., flatten-data.ts, populate-ids.ts)

In scope:
- All reusable utility functions and data transformation logic
- All reusable UI components intended for use across multiple features or applications
- Custom React hooks that encapsulate reusable stateful logic
- Build and bundling configuration for library packages
- Server-side rendering implementations of core components

Out of scope:
- Application-specific business logic and feature implementations
- Route handlers and API endpoints specific to individual applications
- Application-level configuration and environment-specific settings
- One-off components used in a single feature without reuse potential
- Test fixtures and mock data specific to individual test suites

Exceptions:
- EXC-001: A component is being prototyped in an application directory before being promoted to core
- EXC-002: A utility function has application-specific dependencies that cannot be abstracted

## Rationale

- The detection of 51 files following this pattern with 87.69% confidence indicates this is an established and successful architectural approach in the codebase
- Modular library architecture enables code reuse across multiple applications (evidenced by recipes/next-ai consuming packages/core), reducing duplication and maintenance burden
- Clear separation between core libraries and application code creates well-defined boundaries that improve testability and enable independent versioning
- The pattern supports multiple rendering contexts (client and server) which is essential for modern React applications using frameworks like Next.js

## Consequences

Positive:
- Improved code reusability across different parts of the application and potentially external projects
- Clear architectural boundaries make it easier for developers to understand where to place new code
- Independent testing and versioning of core library components becomes possible
- Reduced code duplication leads to lower maintenance costs and fewer bugs
- Better support for tree-shaking and optimized bundle sizes through modular imports

Negative:
- Additional overhead in determining whether code belongs in core or application directories
- Potential for premature abstraction if components are moved to core before their interfaces stabilize
- Increased complexity in build configuration and dependency management across packages
- Learning curve for developers unfamiliar with monorepo or multi-package architectures

## Alternatives

- Flat directory structure with all code in a single src directory without package separation (rejected)
  Rejected because: Does not provide clear boundaries between reusable libraries and application code, leading to tight coupling and reduced reusability
  When valid: Only appropriate for very small applications with no reuse requirements
- Feature-based directory structure where each feature contains its own utilities and components (rejected)
  Rejected because: Leads to code duplication when multiple features need similar utilities, and makes cross-feature reuse difficult
  When valid: Can be used within application directories for feature-specific code, but core libraries should still be separated
- External npm packages for all reusable code published to a registry (deferred)
  Rejected because: Adds significant overhead for versioning and publishing, slows down development iteration
  When valid: Should be considered when libraries mature and need to be shared across multiple independent projects or organizations

## Risks

- Developers may prematurely abstract code into core packages before interfaces are stable, leading to frequent breaking changes
  Mitigation: Establish guidelines requiring components to be used in at least 2-3 places before promotion to core, and maintain semantic versioning with clear deprecation policies
  Owner: Engineering team and tech leads
- Core package may accumulate too many dependencies, creating a heavy bundle that affects all consumers
  Mitigation: Regular dependency audits, use of peer dependencies where appropriate, and consideration of splitting core into multiple focused packages if it grows too large
  Owner: Engineering team
- Circular dependencies may develop between core packages and application code if boundaries are not respected
  Mitigation: Implement linting rules to detect circular dependencies, enforce one-way dependency flow (applications depend on core, never vice versa), and use dependency-cruiser or similar tools in CI
  Owner: DevOps and engineering team

## Implementation Notes

- Use a monorepo tool (e.g., Turborepo, Nx, or Lerna) to manage dependencies between packages and optimize build caching
- Establish clear naming conventions: packages/core/lib for utilities and hooks, packages/core/components for UI components
- Create a CONTRIBUTING.md document that explains when and how to add code to core packages versus application directories
- Set up automated testing that runs against core packages independently to ensure they remain decoupled from application logic
- Consider using TypeScript path aliases to make imports cleaner (e.g., @core/lib/flatten-data instead of ../../packages/core/lib/flatten-data)

## Continuation Context


Verify commands:
- find packages/core/lib -type f -name '*.ts' -o -name '*.tsx' | wc -l
- find packages/core/components -type f -name 'index.tsx' | wc -l
- grep -r 'from.*recipes/' packages/core/ && echo 'ERROR: Core package imports from application code' || echo 'OK: No application imports in core'
- test -f packages/tsup-config/react-import.js && echo 'OK: Build config exists' || echo 'ERROR: Missing build config'

Accept when:
- Core package contains multiple utility modules in lib directory and multiple components in components directory
- No imports from application-specific directories (e.g., recipes/) are found in core package files
- Build configuration for library packages exists in a centralized location
- Components that require server-side rendering have corresponding server.tsx implementations

## Enforcement

- Verified by: Automated linting rules checking import paths and preventing core packages from importing application code
- Verified by: Code review checklist items verifying new code is placed in appropriate directories
- Verified by: CI pipeline checks running dependency analysis tools to detect circular dependencies
- Verified by: Automated tests ensuring core packages can be built and tested independently
- Violation handling: CI build fails if core packages contain imports from application directories
- Violation handling: Pull requests are blocked if linting rules detect architectural violations
- Violation handling: Code review feedback requires refactoring before merge if code is placed in incorrect directory
- Violation handling: Quarterly architecture reviews identify and remediate accumulated violations
- Exception process: Developer documents the reason for exception in code comments and creates a GitHub issue
- Exception process: Tech lead or architect reviews the exception request and approves or suggests alternative approach
- Exception process: If approved, exception is documented in architecture decision log with timeline for remediation
- Exception process: Exceptions are reviewed quarterly to determine if they can be resolved or need to become permanent patterns