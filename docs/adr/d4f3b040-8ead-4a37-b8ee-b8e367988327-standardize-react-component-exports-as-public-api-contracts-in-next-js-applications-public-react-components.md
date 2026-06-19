# Standardize React Component Exports as Public API Contracts in Next.js Applications: Public React Components

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains 13 files across apps/docs and apps/demo that consistently export React components and their TypeScript props interfaces as public API contracts
- Components are organized in a modular block-based architecture where each block (Stats, Flex, Template, Hero, Card, Grid, Heading, Text) exports both a Props interface and a component function
- The pattern uses Next.js framework conventions (next/app, next/server, next/dynamic) alongside React, with imports from a shared @/core library indicating a monorepo structure
- Components import from centralized style modules (styles.module.css) and shared utilities (@/core/lib, @/core/components), establishing a consistent dependency structure
- The middleware.ts and _app.tsx files export Next.js-specific contracts (middleware, config, DocsApp) that serve as application-level public APIs

## Problem Statement

Applications built with component-based frameworks require clear boundaries between internal implementation details and public-facing component APIs. Without standardized export patterns, component consumers face inconsistent interfaces, unclear prop contracts, and difficulty understanding which exports are stable versus internal. This pattern addresses the need for predictable, type-safe component APIs that can be consumed across application boundaries in a monorepo architecture.

## Decision

1. MUST: All public React components MUST export both a TypeScript Props interface and the component function as named exports

## Policy Block

- MUST All public React components MUST export both a TypeScript Props interface and the component function as named exports

In scope:
- React functional components exported from apps/docs and apps/demo
- TypeScript Props interfaces for all public components
- Next.js application-level exports (middleware, _app, page components)
- Block-based UI components (Stats, Flex, Template, Hero, Card, Grid, Heading, Text)
- Shared component utilities imported from @/core

Out of scope:
- Internal helper functions not exported from component modules
- Private implementation details within component files
- Third-party library APIs (react, next, lucide-react)
- Build-time configuration files (tsconfig, webpack)
- Test files and test utilities

Exceptions:
- EXC-001: Dynamic imports using next/dynamic defer component loading and may not export Props interfaces directly
- EXC-002: Internal utility components used only within a single block may omit Props interface exports

## Rationale

- The evidence shows 13 files with 89.62% confidence consistently exporting component contracts, indicating an established architectural pattern rather than isolated instances
- TypeScript Props interfaces provide compile-time type safety and serve as machine-readable API documentation for component consumers
- Named exports for both Props and components enable tree-shaking, explicit dependency tracking, and clear API boundaries in a monorepo structure
- The pattern aligns with Next.js conventions (middleware.ts, _app.tsx) and React best practices, reducing cognitive load for developers familiar with these frameworks

## Consequences

Positive:
- Type-safe component APIs prevent runtime errors from prop mismatches and enable IDE autocomplete
- Consistent export patterns reduce onboarding time and make component APIs discoverable through static analysis
- Named exports enable efficient tree-shaking and explicit dependency graphs in bundled applications
- Separation of Props interfaces from implementation allows API documentation generation and contract testing

Negative:
- Requires maintaining TypeScript interface definitions alongside component implementations, increasing code volume
- Strict naming conventions (ComponentNameProps) may feel verbose for simple components with few props
- Export pattern enforcement requires tooling (linters, CI checks) to prevent drift over time
- Refactoring component names requires updating both the component and Props interface names

## Alternatives

- Use default exports for components without separate Props interface exports (rejected)
  Rejected because: Default exports obscure API contracts, prevent static analysis of prop types, and complicate tree-shaking in bundlers
  When valid: Acceptable for internal utility components not consumed across module boundaries
- Inline prop types using React.FC<{prop: type}> without named interfaces (rejected)
  Rejected because: Inline types cannot be imported by consumers for type composition or documentation generation, reducing API discoverability
  When valid: May be used for one-off components with trivial props that will never be extended
- Generate Props interfaces automatically from JSDoc comments using tooling (deferred)
  Rejected because: Requires additional build tooling and may not capture complex type relationships; deferred pending evaluation of code generation tools
  When valid: Could be adopted if TypeScript code generation from JSDoc proves reliable and maintainable

## Risks

- Inconsistent application of export patterns across teams leads to fragmented API conventions
  Mitigation: Implement ESLint rules to enforce named exports and Props interface naming conventions; add CI checks to verify compliance
  Owner: Engineering team
- Breaking changes to Props interfaces impact multiple consumers in the monorepo without clear versioning
  Mitigation: Adopt semantic versioning for @/core library; use TypeScript's strict mode to catch breaking changes at compile time; maintain CHANGELOG for API changes
  Owner: Platform team
- Over-exporting internal implementation details as public APIs increases maintenance burden
  Mitigation: Document public vs internal exports using JSDoc annotations; conduct quarterly API reviews to identify and deprecate unused exports
  Owner: Architecture team

## Implementation Notes

- Use ESLint plugin eslint-plugin-import to enforce named exports and detect default export violations
- Create a component template or code generator that scaffolds the Props interface and component export pattern automatically
- Document the export pattern in the project's CONTRIBUTING.md with examples from existing components (Hero, Card, Grid)
- For components with client/server splits (Template), ensure both implementations export the same Props interface for API consistency

## Continuation Context


Verify commands:
- grep -r "export.*Props" apps/docs apps/demo --include="*.tsx" | wc -l
- grep -r "export default" apps/docs apps/demo --include="*.tsx" --exclude="_app.tsx" --exclude="middleware.ts" | wc -l
- npx tsc --noEmit --project tsconfig.json

Accept when:
- All component files export at least one Props interface matching the pattern {ComponentName}Props
- Default exports are limited to Next.js convention files (_app.tsx, middleware.ts, page.tsx)
- TypeScript compilation succeeds without type errors related to component prop contracts

## Enforcement

- Verified by: ESLint rules in CI pipeline checking for named exports and Props interface naming conventions
- Verified by: TypeScript strict mode compilation in pre-commit hooks
- Verified by: Automated API documentation generation that fails if Props interfaces are missing
- Violation handling: CI build fails if components lack Props interface exports or use default exports outside allowed files
- Violation handling: Pull request reviews include automated comments identifying export pattern violations
- Violation handling: Quarterly architecture reviews identify and track technical debt from non-compliant components
- Exception process: Request exception through architecture review board with justification for deviation
- Exception process: Document approved exceptions in ADR amendments with expiration dates
- Exception process: Tag non-compliant files with @adr-exception-ADR-AUTO comment linking to approval