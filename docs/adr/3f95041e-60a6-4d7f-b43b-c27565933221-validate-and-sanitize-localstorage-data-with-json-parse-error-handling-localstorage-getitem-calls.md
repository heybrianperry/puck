# Validate and Sanitize localStorage Data with JSON.parse Error Handling: Localstorage Getitem Calls

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- React components in the demo application retrieve template configuration data from browser localStorage to maintain user preferences across sessions
- The localStorage API returns string values that require parsing into JavaScript objects, introducing potential injection and malformed data risks
- The Template component uses JSON.parse with a fallback empty object pattern to handle missing or invalid localStorage entries
- Client-side storage is vulnerable to XSS attacks and user manipulation, requiring defensive parsing strategies to prevent runtime errors and security vulnerabilities

## Problem Statement

Unvalidated parsing of localStorage data can lead to runtime exceptions from malformed JSON, injection attacks through crafted payloads, and application crashes when user-controlled storage is compromised or corrupted. Without proper input validation and error handling, the application becomes vulnerable to both accidental data corruption and deliberate security exploits.

## Decision

1. MUST: All localStorage.getItem() calls MUST be wrapped with JSON.parse() and provide a fallback value using the nullish coalescing operator (??) to handle null or undefined cases

## Policy Block

- MUST All localStorage.getItem() calls MUST be wrapped with JSON.parse() and provide a fallback value using the nullish coalescing operator (??) to handle null or undefined cases

In scope:
- All React components accessing localStorage for configuration or state persistence
- Client-side data retrieval from browser storage APIs (localStorage, sessionStorage)
- Template and configuration management components in the demo application
- Any code path where user-controlled storage data influences application behavior

Out of scope:
- Server-side storage and database access patterns
- In-memory state management that does not persist to browser storage
- Third-party library internal storage mechanisms
- Development-only debugging storage that is not present in production builds

Exceptions:
- EXC-001: localStorage is used only for non-critical UI preferences (theme, layout) where parsing failures can safely default to application defaults without security impact

## Rationale

- The evidence shows JSON.parse(localStorage.getItem(templateKey) ?? "{}") pattern in apps/demo/config/blocks/Template/client.tsx, demonstrating defensive parsing with fallback for missing data
- Browser localStorage is a common XSS attack vector where malicious scripts can inject crafted JSON payloads that exploit parsing vulnerabilities or cause denial-of-service through malformed data
- The nullish coalescing operator (??) combined with empty object fallback prevents null/undefined errors but requires additional try-catch protection for malformed JSON scenarios
- React components that fail during render due to parsing errors can crash the entire component tree, making input validation critical for application stability

## Consequences

Positive:
- Prevents application crashes from malformed or corrupted localStorage data by providing graceful fallback behavior
- Reduces attack surface for XSS-based injection attacks that attempt to exploit JSON parsing vulnerabilities
- Improves application resilience when users manually edit browser storage or when storage becomes corrupted
- Establishes a consistent pattern for safe localStorage access across the React component tree

Negative:
- Adds boilerplate code to every localStorage access point, increasing code verbosity
- Silent fallback to default values may mask underlying data corruption issues that should be surfaced to users or monitoring systems
- Performance overhead from try-catch blocks and validation logic, though typically negligible for localStorage operations
- Developers may become complacent about storage security if they rely solely on parsing validation without addressing root XSS vulnerabilities

## Alternatives

- Use a localStorage wrapper library that provides automatic validation and type safety (e.g., zod-storage, typed-local-store) (rejected)
  Rejected because: Introduces additional dependency weight and learning curve; the current pattern is lightweight and sufficient for the demo application's needs
  When valid: For larger applications with complex storage schemas where type safety and validation logic justify the dependency cost
- Implement a centralized storage service layer that encapsulates all localStorage access with validation (deferred)
  Rejected because: Requires significant refactoring of existing component architecture; should be considered for future architectural improvements
  When valid: When the application grows to have dozens of localStorage access points that would benefit from centralized error handling and monitoring
- Avoid localStorage entirely and use only in-memory state management with server-side persistence (rejected)
  Rejected because: Eliminates offline capability and increases server load; localStorage provides valuable client-side caching for template configurations
  When valid: For applications with strict security requirements where client-side storage risks outweigh the benefits of local caching

## Risks

- Developers may forget to add try-catch blocks around JSON.parse calls, leaving some code paths vulnerable to parsing exceptions
  Mitigation: Implement ESLint rules to detect unprotected JSON.parse usage and require try-catch or safe parsing utilities
  Owner: Engineering team
- Fallback values may not match the expected data structure, causing type errors downstream in component logic
  Mitigation: Define TypeScript interfaces for localStorage data structures and validate parsed objects against these types using type guards
  Owner: Engineering team
- Silent failures from fallback behavior may hide legitimate bugs or data corruption issues that should be reported
  Mitigation: Add logging or monitoring for localStorage parsing failures to track frequency and patterns of invalid data
  Owner: Engineering team

## Implementation Notes

- Create a utility function safeParseLocalStorage(key, fallback) that encapsulates the try-catch and nullish coalescing pattern for reuse across components
- Define TypeScript interfaces for all localStorage data structures and export them from a central types file for consistency
- Add unit tests that verify component behavior when localStorage contains null, undefined, malformed JSON, and unexpected object shapes
- Document the expected localStorage schema in component comments or README files to help developers understand the data contract

## Continuation Context


Verify commands:
- grep -r 'localStorage\.getItem' apps/demo --include='*.tsx' --include='*.ts' | grep -v 'JSON\.parse' | grep -v '//' | wc -l | grep -q '^0$'
- grep -r 'JSON\.parse.*localStorage' apps/demo --include='*.tsx' --include='*.ts' | grep -v '??' | wc -l | grep -q '^0$'
- npm run test -- --testPathPattern='localStorage.*validation' --passWithNoTests=false

Accept when:
- All localStorage.getItem() calls in the codebase are wrapped with JSON.parse() and include nullish coalescing fallback values
- No direct localStorage access exists without parsing and validation in production code paths
- Unit tests demonstrate graceful handling of null, undefined, and malformed JSON from localStorage

## Enforcement

- Verified by: ESLint custom rules detecting unprotected localStorage.getItem() and JSON.parse() patterns
- Verified by: Code review checklist requiring validation of all browser storage access
- Verified by: Automated testing in CI pipeline that injects malformed localStorage data and verifies graceful degradation
- Violation handling: CI pipeline fails on ESLint violations for unprotected localStorage access
- Violation handling: Pull requests with localStorage changes require security team review approval
- Violation handling: Runtime monitoring alerts on excessive JSON parsing errors from localStorage operations
- Exception process: Submit exception request to security team with justification for why validation is not required
- Exception process: Document the exception in code comments with ticket reference and expiration date
- Exception process: Security team reviews exceptions quarterly to determine if they can be removed