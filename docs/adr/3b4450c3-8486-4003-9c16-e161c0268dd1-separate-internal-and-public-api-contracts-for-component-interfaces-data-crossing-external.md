# Separate Internal and Public API Contracts for Component Interfaces: Data Crossing External

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- React component interfaces in apps/demo/config/blocks/Template/client.tsx expose both internal implementation contracts (TemplateInternal) and public API contracts (Template)
- The codebase uses @/core library imports for shared types and utilities, establishing a boundary between application code and core framework functionality
- Component initialization involves parsing untrusted localStorage data via JSON.parse, requiring validation at the contract boundary
- The createComponent pattern suggests a factory or wrapper approach that may transform internal contracts to public ones

## Problem Statement

Components that expose both internal implementation details and public APIs create security and maintainability risks when untrusted data flows through the system. Without clear contract separation, validation logic may be bypassed, internal state may leak to consumers, and refactoring becomes hazardous as the blast radius of changes is unclear.

## Decision

1. MUST: Data crossing from external sources (localStorage, network, user input) MUST be validated against the public contract before transformation to internal contracts

## Policy Block

- MUST Data crossing from external sources (localStorage, network, user input) MUST be validated against the public contract before transformation to internal contracts

In scope:
- React components in apps/demo/config/blocks that expose public APIs
- Factory functions and component wrappers that transform external data to internal state
- Modules importing from @/core that bridge application and framework boundaries
- Any code path that parses untrusted data (localStorage, URL params, API responses)

Out of scope:
- Pure internal components with no external consumers
- Test fixtures and mock implementations
- Build-time configuration that does not process runtime data

## Rationale

- The evidence shows explicit separation of TemplateInternal and Template contracts in the same module, indicating intentional API boundary design
- The presence of JSON.parse(localStorage.getItem(...)) demonstrates untrusted data ingestion that requires validation at the contract boundary to prevent injection or type confusion attacks
- Importing from @/core establishes a framework/application boundary where contract stability and backward compatibility matter for maintainability
- The createComponent pattern suggests a transformation layer exists to mediate between internal and public contracts, enabling validation and sanitization

## Consequences

Positive:
- Clear API boundaries reduce the risk of exposing sensitive internal state or implementation details to external consumers
- Validation at contract boundaries prevents malformed or malicious data from corrupting internal component state
- Refactoring internal implementations becomes safer as changes are isolated from public API consumers
- Type safety is enforced at compile time, catching contract violations before runtime

Negative:
- Additional boilerplate required to define and maintain dual contract interfaces
- Transformation logic between contracts adds runtime overhead and complexity
- Developers must understand and respect the boundary, increasing cognitive load
- Over-separation may lead to unnecessary abstraction for simple components

## Alternatives

- Use a single unified contract for both internal and public APIs (rejected)
  Rejected because: Single contracts expose internal implementation details and prevent independent evolution of public APIs, increasing security and maintenance risks when untrusted data is involved
  When valid: For pure internal components with no external consumers or untrusted data sources
- Rely on runtime validation libraries (Zod, io-ts) without explicit contract separation (deferred)
  Rejected because: Runtime validation alone does not provide compile-time type safety or clear architectural boundaries, though it complements contract separation
  When valid: As an additional layer of defense when combined with explicit contract separation
- Use module visibility (internal packages) instead of naming conventions (deferred)
  Rejected because: Module-level separation provides stronger enforcement but requires more complex build configuration and may not be feasible in monorepo structures
  When valid: In projects with strict module boundaries and build tooling support for internal packages

## Risks

- Developers may bypass contract boundaries by directly accessing internal contracts from external code
  Mitigation: Enforce contract boundaries through code review, linting rules (e.g., ESLint import restrictions), and module visibility controls
  Owner: engineering team
- Incomplete validation at contract boundaries may allow malformed data to reach internal contracts
  Mitigation: Implement comprehensive validation logic in transformation functions and add integration tests that exercise untrusted data paths
  Owner: engineering team
- Contract drift between internal and public interfaces may occur over time without synchronization
  Mitigation: Use TypeScript mapped types or code generation to derive one contract from the other where appropriate, and add tests that verify contract compatibility
  Owner: engineering team

## Implementation Notes

- Define internal contracts with 'Internal' suffix (e.g., TemplateInternal) and public contracts without suffix (e.g., Template) in the same module for discoverability
- Implement transformation functions (e.g., createComponent) that validate external data against the public contract schema before constructing internal contract instances
- Use TypeScript's 'Omit', 'Pick', or 'Partial' utility types to derive public contracts from internal ones when appropriate, reducing duplication
- Add ESLint rules to prevent direct imports of internal contracts from outside the module boundary

## Continuation Context


Verify commands:
- grep -r 'export.*Internal' apps/demo/config/blocks --include='*.tsx' --include='*.ts' | wc -l
- grep -r 'JSON.parse.*localStorage' apps/demo --include='*.tsx' --include='*.ts' -A 5 | grep -c 'validate\|parse\|schema'
- npx tsc --noEmit --strict && echo 'Type checking passed'

Accept when:
- At least one component in apps/demo/config/blocks exports both an internal contract (suffixed with 'Internal') and a public contract
- All JSON.parse calls processing untrusted data are followed by validation logic within 5 lines
- TypeScript strict mode compilation passes without errors related to contract type mismatches

## Enforcement

- Verified by: TypeScript compiler in strict mode during CI builds
- Verified by: Code review checklist requiring contract separation for components with external APIs
- Verified by: ESLint rules enforcing import restrictions on internal contracts
- Violation handling: CI build fails on TypeScript compilation errors related to contract violations
- Violation handling: Pull requests blocked until code review approves contract boundary design
- Violation handling: ESLint violations reported as errors in pre-commit hooks
- Exception process: Document exception rationale in code comments explaining why contract separation is not applicable
- Exception process: Obtain approval from tech lead or security reviewer for components handling untrusted data
- Exception process: Add ESLint disable comments with justification for specific import violations