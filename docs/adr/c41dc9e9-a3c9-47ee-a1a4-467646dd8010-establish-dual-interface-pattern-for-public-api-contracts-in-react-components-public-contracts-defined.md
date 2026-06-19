# Establish Dual-Interface Pattern for Public API Contracts in React Components: Public Contracts Defined

Status: proposed
Date: 2025-01-20
Deciders: Detection Pipeline (automated)

## Context

- The codebase uses React with a component-based architecture requiring clear separation between internal implementation details and public-facing APIs
- Components in apps/demo/config/blocks/Template/client.tsx expose both internal (TemplateInternal) and external (Template) contracts, indicating a deliberate boundary between implementation and consumption
- The @/core library provides foundational types and utilities (detected via imports from @/core, @/core/types, @/core/lib/generate-id) that components build upon
- Local storage integration (JSON.parse(localStorage.getItem(templateKey))) demonstrates state persistence requirements that cross component boundaries
- The createComponent concurrency model suggests a factory or higher-order component pattern that necessitates well-defined interface contracts

## Problem Statement

Components need to maintain internal implementation flexibility while providing stable, versioned public interfaces to consumers. Without explicit dual-interface contracts, internal refactoring risks breaking downstream consumers, and the boundary between private implementation details and public API surface becomes unclear.

## Decision

1. MUST: Public API contracts MUST be defined as explicit TypeScript interfaces or types, not inferred from implementation

## Policy Block

- MUST Public API contracts MUST be defined as explicit TypeScript interfaces or types, not inferred from implementation

In scope:
- React components in apps/demo/config/blocks/ that are consumed by external modules
- Components using createComponent factory pattern from @/core
- Public API surfaces exposed through barrel exports (index.ts)
- Components that integrate with shared state management or localStorage

Out of scope:
- Internal utility functions without external consumers
- Private helper components used only within a single module
- Type definitions in @/core/types that serve as foundational primitives
- Test fixtures and mock implementations

Exceptions:
- EXC-001: Component is explicitly marked as experimental or alpha with clear documentation
- EXC-002: Rapid prototyping phase with no external consumers

## Rationale

- The evidence shows TemplateInternal and Template as distinct contracts in api.public.contracts, demonstrating an established pattern of separating internal and external interfaces
- React component architecture benefits from explicit contract boundaries to enable independent evolution of implementation while maintaining API stability
- The createComponent concurrency model requires well-typed interfaces to maintain type safety through component factory abstraction layers
- Integration with localStorage and cross-component state sharing necessitates clear contracts to prevent coupling between component internals and external state management

## Consequences

Positive:
- Internal refactoring becomes safer as implementation details are isolated from public API surface
- Type safety improves across component boundaries, catching integration errors at compile time
- API versioning and deprecation strategies become clearer with explicit public contracts
- Testing and mocking are simplified through well-defined interface boundaries

Negative:
- Increased boilerplate for components requiring dual interface definitions
- Potential confusion for developers unfamiliar with the Internal/Public naming convention
- Additional maintenance burden to keep internal and public interfaces synchronized when appropriate
- Risk of over-engineering simple components that don't require strict API boundaries

## Alternatives

- Single interface per component with all properties marked as public (rejected)
  Rejected because: Exposes implementation details to consumers, creating tight coupling and making internal refactoring risky
  When valid: For simple, internal-only components with no external consumers
- Use TypeScript private/protected modifiers on class components (rejected)
  Rejected because: Does not work with functional components and createComponent factory pattern; runtime enforcement is limited
  When valid: Legacy class-based components that haven't migrated to functional patterns
- Rely on documentation and naming conventions without explicit type separation (rejected)
  Rejected because: Lacks compile-time enforcement and allows accidental coupling to internal implementation details
  When valid: Prototyping phase or proof-of-concept code not intended for production

## Risks

- Developers may inconsistently apply the dual-interface pattern, leading to fragmented API design across the codebase
  Mitigation: Establish linting rules to detect public components without dual interfaces; provide component templates and generator scripts
  Owner: Engineering team
- Internal interfaces may drift from public interfaces, causing confusion about which interface to use in different contexts
  Mitigation: Document clear guidelines on when to use each interface; implement automated tests that verify public interface is subset of internal
  Owner: Engineering team
- Over-application of pattern to simple components increases cognitive load without proportional benefit
  Mitigation: Define clear criteria for when dual interfaces are required (e.g., external consumers, published packages); allow exceptions for internal-only components
  Owner: Architecture team

## Implementation Notes

- Use naming convention ComponentNameInternal for internal interfaces and ComponentName for public interfaces to maintain consistency
- Export only the public interface from barrel exports (index.ts) while keeping internal interfaces accessible via direct imports for testing
- When using createComponent factory, pass the internal interface as the generic type parameter and return the public interface
- Document the distinction between interfaces in JSDoc comments, explaining what each interface is intended for

## Continuation Context


Verify commands:
- grep -r "export.*Internal" apps/demo/config/blocks/ | wc -l
- find apps/demo/config/blocks -name '*.tsx' -exec grep -l 'api.public.contracts' {} \;
- npx tsc --noEmit --strict && echo 'Type checking passed'

Accept when:
- All public-facing React components in apps/demo/config/blocks/ export both Internal and public interfaces
- TypeScript compilation succeeds with strict mode enabled, confirming type safety across interface boundaries
- Code review confirms that internal implementation details are not exposed through public interfaces

## Enforcement

- Verified by: TypeScript compiler strict mode checks during CI build
- Verified by: ESLint custom rule detecting public components without dual interfaces
- Verified by: Code review checklist item for API boundary verification
- Verified by: Automated tests validating public interface is proper subset of internal interface
- Violation handling: CI build fails if TypeScript compilation errors occur due to interface mismatches
- Violation handling: Pull requests blocked until dual-interface pattern is applied to public components
- Violation handling: Architecture review required for components that need exception approval
- Violation handling: Technical debt ticket created for legacy components requiring migration
- Exception process: Developer documents rationale for exception in component JSDoc and pull request description
- Exception process: Tech lead reviews exception request against policy_exceptions criteria
- Exception process: If approved, add @adr-exception comment with ADR-AUTO reference and exception ID
- Exception process: Track approved exceptions in architecture decision log for periodic review