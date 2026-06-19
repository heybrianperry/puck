# Validate JSON.parse Input in React Hooks for localStorage Operations: Parse Failures Logged

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- React hooks in packages/core/lib/use-sidebar-resize.ts and apps/demo/lib/use-demo-data.ts parse JSON from localStorage without validation guards
- localStorage data can be corrupted, manually edited, or contain malformed JSON that causes runtime exceptions
- Error handling exists via console.error logging but occurs after parse failures, indicating defensive programming awareness
- Custom hooks (useSidebarResize, useDemoData) expose public contracts that must handle untrusted localStorage input reliably
- useEffect and useCallback patterns coordinate state hydration from persistent storage during component lifecycle

## Problem Statement

React hooks that parse JSON from localStorage without input validation expose the application to runtime exceptions when localStorage contains malformed or corrupted data. This creates fragile user experiences where UI state restoration fails silently or crashes components, particularly in interaction patterns that depend on persisted user preferences or demo data.

## Decision

1. SHOULD: Parse failures SHOULD be logged with context identifying the storage key and operation (load/save)

## Policy Block

- SHOULD Parse failures SHOULD be logged with context identifying the storage key and operation (load/save)

## Rationale

- Evidence shows JSON.parse(savedWidths) and JSON.parse(dataStr) operations in React hooks without visible validation guards, creating vulnerability to malformed input
- Existing console.error logging for localStorage failures demonstrates awareness of error conditions but indicates reactive rather than preventive handling
- Public hook contracts (useSidebarResize, useDemoData) coordinate UI state hydration in useEffect/useCallback patterns where failures cascade to component rendering
- Pattern detected across 2 files with 86.50% confidence indicates systematic approach to localStorage interaction requiring standardized validation

## Consequences

Positive:
- Prevents runtime exceptions from corrupted localStorage data causing component crashes or white screens
- Improves user experience by gracefully degrading to default values when persisted state is invalid
- Enables debugging through structured error logging that identifies specific storage keys and operations
- Creates consistent error handling pattern across custom hooks that interact with browser storage APIs

Negative:
- Adds boilerplate try-catch blocks to every localStorage parse operation, increasing code verbosity
- Silent fallback to defaults may mask underlying data corruption issues that should be surfaced to users
- Additional validation logic increases hook complexity and maintenance burden
- Performance overhead from validation checks on every localStorage read, though typically negligible

## Alternatives

- Use a centralized localStorage wrapper library (e.g., localforage) with built-in validation (rejected)
  Rejected because: Introduces external dependency and migration cost for existing localStorage usage patterns; evidence shows inline parsing is already established
  When valid: When starting a new project or performing major refactoring of storage layer
- Implement schema validation using Zod or similar library for all parsed localStorage data (deferred)
  Rejected because: Adds significant complexity and bundle size; may be overkill for simple key-value storage patterns
  When valid: When localStorage stores complex nested objects requiring strict type safety guarantees
- Accept parse failures and let React error boundaries catch exceptions (rejected)
  Rejected because: Creates poor user experience with component-level failures; error boundaries are too coarse-grained for localStorage validation
  When valid: Never - localStorage failures should be handled locally in hooks

## Risks

- Inconsistent validation implementation across different hooks leads to gaps in error handling coverage
  Mitigation: Create shared utility function for validated localStorage parsing that all hooks must use
  Owner: frontend engineering team
- Fallback values may not match user expectations, causing confusion when persisted state is silently discarded
  Mitigation: Log validation failures to monitoring system and consider user-facing notifications for critical state loss
  Owner: product and engineering teams
- Validation logic may not cover all edge cases (null, undefined, wrong type) leading to runtime errors downstream
  Mitigation: Implement comprehensive unit tests for localStorage parsing utilities covering malformed input scenarios
  Owner: frontend engineering team

## Implementation Notes

- Create a shared utility function parseLocalStorageJSON(key, fallback) that encapsulates try-catch and logging logic
- Refactor existing useSidebarResize and useDemoData hooks to use the validated parsing utility
- Add unit tests verifying hooks handle malformed JSON, null values, and missing localStorage keys gracefully
- Document expected localStorage schema for each hook in JSDoc comments to aid future validation implementation

## Continuation Context


Verify commands:
- grep -r 'JSON\.parse.*localStorage' --include='*.ts' --include='*.tsx' | grep -v 'try' | wc -l
- grep -r 'parseLocalStorageJSON\|safeParseJSON' --include='*.ts' --include='*.tsx' | wc -l
- npm test -- --testPathPattern='use-.*\.test' --testNamePattern='localStorage.*invalid'

Accept when:
- All JSON.parse operations on localStorage are wrapped in try-catch or use validated parsing utility
- Unit tests exist verifying hooks handle malformed localStorage data without throwing exceptions
- Error logging captures localStorage parse failures with sufficient context for debugging

## Enforcement

- Verified by: Code review checklist requiring validation for all localStorage.getItem operations
- Verified by: ESLint custom rule flagging unwrapped JSON.parse on localStorage data
- Verified by: Unit test coverage requirements for localStorage error handling paths
- Violation handling: CI pipeline fails if grep verification detects unvalidated JSON.parse on localStorage
- Violation handling: Code review blocks merge if localStorage parsing lacks error handling
- Violation handling: Runtime monitoring alerts on repeated localStorage parse failures in production
- Exception process: Document justification in code comment explaining why validation is unnecessary for specific case
- Exception process: Obtain approval from frontend tech lead for exception
- Exception process: Add exception to ESLint ignore list with ticket reference for future review