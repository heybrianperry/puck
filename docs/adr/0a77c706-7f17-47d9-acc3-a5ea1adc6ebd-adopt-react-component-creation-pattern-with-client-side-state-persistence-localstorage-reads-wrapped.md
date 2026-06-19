# Adopt React Component Creation Pattern with Client-Side State Persistence: Localstorage Reads Wrapped

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase uses React as the UI framework with a component-based architecture, importing from @/core and @/core/types for shared functionality
- Template components require client-side state persistence using localStorage to maintain user configurations across sessions
- The createComponent pattern is used to instantiate React components with specific lifecycle and rendering behaviors
- Input validation is performed on localStorage data using JSON.parse with fallback to empty objects to prevent injection attacks
- The architecture separates internal template contracts (TemplateInternal) from public-facing contracts (Template) to enforce API boundaries

## Problem Statement

Client-side template components need to persist user state across browser sessions while maintaining secure input validation practices. Direct localStorage access without validation creates security vulnerabilities including XSS attacks and data corruption. The system requires a standardized pattern for component creation that enforces input validation and proper error handling when reading persisted state.

## Decision

1. MUST: All localStorage reads MUST be wrapped with JSON.parse and provide a fallback value to handle missing or corrupted data

## Policy Block

- MUST All localStorage reads MUST be wrapped with JSON.parse and provide a fallback value to handle missing or corrupted data

In scope:
- All React components in apps/demo/config/blocks that require client-side state persistence
- Template components that expose public APIs to other modules
- Components using localStorage for configuration or user preferences
- UI components created through the createComponent factory pattern

Out of scope:
- Server-side components that do not access browser APIs
- Components that use session storage or other persistence mechanisms
- Third-party library components not under direct control
- Static components without state management requirements

Exceptions:
- EXC-001: Legacy components during migration period that have documented technical debt tickets

## Rationale

- The evidence shows consistent use of JSON.parse with null coalescing (localStorage.getItem(templateKey) ?? '{}') which demonstrates a defensive programming pattern against malformed or missing data
- The createComponent pattern provides a standardized approach to component instantiation that can enforce security policies and lifecycle hooks consistently across the codebase
- Separation of TemplateInternal and Template contracts indicates architectural intent to control API surface area and prevent direct access to implementation details
- Centralized imports from @/core establish a shared foundation that can be updated to address security vulnerabilities in a single location rather than scattered throughout components

## Consequences

Positive:
- Reduced risk of XSS attacks and data injection through consistent input validation on all localStorage reads
- Improved maintainability through standardized component creation patterns that can be audited and updated centrally
- Better encapsulation and API stability through separation of internal and public contracts
- Consistent error handling prevents application crashes from corrupted or missing localStorage data

Negative:
- Additional boilerplate code required for every localStorage access with JSON.parse and fallback logic
- Performance overhead from parsing JSON on every component mount, though typically negligible for small configuration objects
- Increased complexity in component creation requiring understanding of the createComponent factory pattern
- Potential for inconsistent fallback values if developers choose different defaults across components

## Alternatives

- Use direct localStorage.getItem() calls without validation or error handling (rejected)
  Rejected because: Creates security vulnerabilities to XSS attacks and causes application crashes when localStorage contains malformed data or is unavailable
  When valid: Never valid in production code; only acceptable in isolated prototypes or demos
- Implement a centralized storage service with built-in validation and type safety (deferred)
  Rejected because: Would provide better type safety and centralized validation but requires significant refactoring effort across existing components
  When valid: Should be considered for future architectural improvements when refactoring the state management layer
- Use React Context or state management library (Redux, Zustand) for all persistent state (rejected)
  Rejected because: Adds unnecessary complexity and bundle size for simple configuration persistence; localStorage is sufficient for template preferences
  When valid: Valid for complex application state that requires synchronization across multiple components or server-side persistence

## Risks

- Developers may bypass the validation pattern and use direct localStorage access in new components
  Mitigation: Implement ESLint rule to detect direct localStorage usage without JSON.parse and null coalescing; add code review checklist item
  Owner: Engineering team and security review board
- localStorage quota limits (typically 5-10MB) may be exceeded with extensive template configurations
  Mitigation: Implement size monitoring and warnings when approaching quota limits; document maximum recommended configuration size
  Owner: Frontend architecture team
- Inconsistent fallback values across components may lead to unexpected behavior when localStorage is cleared
  Mitigation: Document standard fallback patterns in component guidelines; create shared default configuration objects
  Owner: Engineering team

## Implementation Notes

- Create a shared utility function in @/core/lib for safe localStorage reads that encapsulates the JSON.parse and null coalescing pattern
- Document the createComponent pattern with examples showing proper integration with localStorage validation
- Establish naming conventions for localStorage keys (e.g., templateKey) to prevent collisions and enable easier debugging
- Add TypeScript types for all Template and TemplateInternal contracts to catch type mismatches at compile time
- Consider implementing a localStorage wrapper that provides automatic JSON serialization/deserialization with error boundaries

## Continuation Context


Verify commands:
- grep -r 'localStorage.getItem' apps/demo/config/blocks/ | grep -v 'JSON.parse' | grep -v '??'
- grep -r 'createComponent' apps/demo/config/blocks/Template/ --include='*.tsx' --include='*.ts'
- grep -r 'export.*Template' apps/demo/config/blocks/Template/ | grep -E '(TemplateInternal|Template)'

Accept when:
- All localStorage.getItem calls in template components are wrapped with JSON.parse and provide fallback values using null coalescing operator
- Template components use createComponent pattern for instantiation as evidenced by grep results
- Both TemplateInternal and Template contracts are exported and properly separated in the public API

## Enforcement

- Verified by: ESLint custom rule detecting unsafe localStorage access patterns
- Verified by: Code review checklist requiring validation of all browser storage operations
- Verified by: Automated security scanning in CI pipeline flagging direct localStorage usage without error handling
- Violation handling: CI build fails if ESLint rule detects unsafe localStorage patterns
- Violation handling: Pull requests blocked until code review checklist items are addressed
- Violation handling: Security team notified of violations in production code for immediate remediation
- Exception process: Developer submits exception request with technical justification and risk assessment
- Exception process: Tech lead reviews and approves with documented migration timeline if legacy code
- Exception process: Exception logged in architectural decision log with expiration date and owner assigned