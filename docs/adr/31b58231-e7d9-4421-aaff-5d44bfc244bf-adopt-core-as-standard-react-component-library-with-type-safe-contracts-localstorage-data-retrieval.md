# Adopt @/core as Standard React Component Library with Type-Safe Contracts: Localstorage Data Retrieval

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The codebase uses React as the primary UI framework with a centralized core library accessed via @/core path alias
- Components import shared types from @/core/types and utilities from @/core/lib/generate-id, establishing a common dependency pattern
- Template components define explicit internal and public API contracts (TemplateInternal, Template) to separate implementation from interface
- Client-side state persistence uses localStorage with JSON parsing, requiring input validation to prevent injection attacks
- The component architecture uses createComponent pattern suggesting a factory-based approach to component instantiation

## Problem Statement

Without standardized secure coding practices for React components that interact with external storage and user input, the application faces risks from injection attacks, type inconsistencies, and unclear API boundaries between internal implementation and public contracts.

## Decision

1. MUST: All localStorage data retrieval MUST validate and sanitize input using JSON.parse with null coalescing or try-catch error handling

## Policy Block

- MUST All localStorage data retrieval MUST validate and sanitize input using JSON.parse with null coalescing or try-catch error handling

In scope:
- All React components in apps/demo/config/blocks
- Client-side components that interact with browser storage APIs
- Components that define public API contracts for external consumption
- Shared utility functions and type definitions in @/core

Out of scope:
- Server-side components without browser API access
- Third-party library code outside the @/core namespace
- Build-time configuration and tooling scripts
- Test fixtures and mock data

Exceptions:
- EXC-001: Legacy components in migration phase may use relative imports temporarily
- EXC-002: Performance-critical paths may bypass contract types if profiling demonstrates measurable impact

## Rationale

- The evidence shows consistent use of @/core imports across template components, indicating an established pattern for centralizing shared functionality and reducing coupling
- Explicit contract separation (TemplateInternal vs Template) demonstrates architectural intent to control API surface area and prevent implementation details from leaking
- The presence of JSON.parse with null coalescing on localStorage access shows awareness of input validation requirements, though this pattern needs standardization
- Using createComponent pattern and centralized utilities like generate-id reduces code duplication and creates consistent behavior across the component tree

## Consequences

Positive:
- Centralized @/core library provides single source of truth for types and utilities, reducing inconsistencies and maintenance burden
- Explicit contract types enable safer refactoring by preventing breaking changes to public APIs while allowing internal implementation changes
- Standardized input validation on localStorage access mitigates XSS and injection attack vectors from untrusted storage
- Path alias imports improve code portability and reduce brittleness from deep relative path dependencies

Negative:
- Additional abstraction layer through @/core may increase cognitive overhead for developers unfamiliar with the codebase structure
- Contract type duplication (internal vs public) adds boilerplate and requires discipline to maintain separation
- Centralized library creates potential bottleneck for changes affecting multiple consumers
- Path alias configuration requires build tool setup and may complicate debugging with source maps

## Alternatives

- Use relative imports throughout codebase without centralized @/core library (rejected)
  Rejected because: Relative imports create tight coupling, make refactoring difficult, and provide no centralized location for security-critical validation logic
  When valid: Only appropriate for small, single-module applications with no shared utilities
- Combine internal and public contracts into single type definition (rejected)
  Rejected because: Exposes implementation details in public API surface, making breaking changes more likely and reducing encapsulation
  When valid: Acceptable for internal-only components with no external consumers
- Use runtime validation library (Zod, Yup) for localStorage parsing instead of manual JSON.parse (deferred)
  Rejected because: Not rejected, but deferred pending evaluation of bundle size impact and performance characteristics
  When valid: Should be reconsidered when complex validation schemas are needed or when type safety at runtime boundaries becomes critical

## Risks

- Malicious data in localStorage could bypass JSON.parse validation if error handling is incomplete
  Mitigation: Implement comprehensive try-catch blocks around all localStorage access with fallback to safe defaults; add runtime type validation for critical fields
  Owner: Frontend security team
- Breaking changes to @/core types or utilities could cascade failures across many components
  Mitigation: Implement semantic versioning for @/core exports; require deprecation warnings before removal; maintain comprehensive test coverage
  Owner: Core library maintainers
- Contract type drift where internal implementation diverges from public contract expectations
  Mitigation: Add automated tests verifying contract compatibility; use TypeScript strict mode to catch type mismatches at compile time
  Owner: Engineering team

## Implementation Notes

- Configure TypeScript path mapping in tsconfig.json to resolve @/core alias to the core library root directory
- Create ESLint rule to enforce @/core imports over relative paths for shared utilities and types
- Establish naming convention for contract types: suffix Internal for implementation types, use plain name for public contracts
- Wrap all localStorage.getItem calls in try-catch blocks or use helper function from @/core/lib that handles errors consistently
- Document public contract types with JSDoc comments specifying stability guarantees and deprecation policy

## Continuation Context


Verify commands:
- grep -r "from ['\"]@/core" apps/demo/config/blocks --include="*.tsx" --include="*.ts"
- grep -r "localStorage.getItem" apps/demo --include="*.tsx" | grep -v "JSON.parse"
- npx tsc --noEmit --strict && echo "Type checking passed"

Accept when:
- All components in apps/demo/config/blocks import from @/core for shared utilities and types
- No localStorage access occurs without JSON.parse or equivalent validation wrapper
- TypeScript compilation succeeds with strict mode enabled and no type errors in contract boundaries

## Enforcement

- Verified by: ESLint rules checking import patterns in pre-commit hooks
- Verified by: TypeScript strict mode compilation in CI pipeline
- Verified by: Code review checklist requiring contract type verification for new components
- Verified by: Automated security scanning for localStorage usage patterns
- Violation handling: CI build fails on ESLint violations related to import patterns
- Violation handling: Pull requests blocked until TypeScript strict mode passes
- Violation handling: Security scanner flags localStorage usage without validation for manual review
- Violation handling: Quarterly audit of contract type usage with remediation tracking
- Exception process: Submit exception request via architecture decision log with technical justification
- Exception process: Provide evidence of performance impact or migration constraints requiring exception
- Exception process: Obtain approval from tech lead and document exception inline with ADR reference
- Exception process: Schedule review of exception after 6 months to evaluate if still necessary