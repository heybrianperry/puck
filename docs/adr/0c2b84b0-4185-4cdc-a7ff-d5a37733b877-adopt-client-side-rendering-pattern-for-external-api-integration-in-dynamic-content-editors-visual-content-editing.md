# Adopt Client-Side Rendering Pattern for External API Integration in Dynamic Content Editors: Visual Content Editing

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all external API integrations within client-side dynamic content editing components, particularly those implementing visual page builders or interactive editing interfaces.

## Context

- Dynamic content editing systems require real-time interactivity and immediate visual feedback that is best served through client-side rendering patterns
- External API integrations in visual page builders need to maintain state synchronization between the editing interface and the underlying data model without full page reloads
- The pattern was detected in Next.js applications using the Puck visual editor framework, which requires client-side JavaScript execution for drag-and-drop functionality and live preview capabilities
- Client-side rendering enables rich interactive experiences for external users while maintaining separation between the editing interface and the production rendering pipeline
- The use of client.tsx files in Next.js app router architecture indicates an intentional architectural decision to isolate client-side interactive components from server-side rendering concerns

## Problem Statement

When building public-facing APIs that power dynamic content editing interfaces, there is a tension between server-side rendering for performance and SEO versus client-side rendering for interactivity. Visual page builders and content management interfaces require immediate user feedback, state management, and complex interactions that are difficult to achieve with server-side rendering alone. The challenge is to establish a consistent pattern for when and how to use client-side rendering for external API integrations while maintaining performance, security, and maintainability.

## Decision

1. SHOULD: Visual content editing interfaces that integrate with external APIs SHOULD implement client-side state management to handle real-time updates without server round-trips

## Policy Block

- SHOULD Visual content editing interfaces that integrate with external APIs SHOULD implement client-side state management to handle real-time updates without server round-trips

In scope:
- Visual page builders and content editors with drag-and-drop functionality
- Interactive components requiring real-time state synchronization with external APIs
- Dynamic content editing interfaces exposed to external users or clients
- Components implementing live preview or WYSIWYG editing capabilities
- API integrations within Next.js app router using dynamic route parameters

Out of scope:
- Static content rendering for SEO-critical pages
- Server-side data fetching for initial page loads
- Internal administrative APIs not exposed to external clients
- Batch processing or background job integrations with external APIs
- Read-only API integrations that do not require user interaction

Exceptions:
- EXC-001: Performance profiling demonstrates that server-side rendering with progressive enhancement can achieve equivalent user experience metrics
- EXC-002: Regulatory or compliance requirements mandate server-side rendering for audit trail or data residency purposes

## Rationale

- The pattern was detected with 86.70% confidence across 2 files implementing the Puck visual editor framework, indicating a deliberate architectural choice for client-side rendering in dynamic content editing scenarios
- Client-side rendering enables the rich interactive experiences required for visual page builders, including drag-and-drop, live preview, and immediate visual feedback that would be impractical with server-side rendering
- Isolating client-side components in dedicated files (client.tsx) provides clear architectural boundaries and enables better code splitting and optimization in modern frameworks like Next.js
- The use of dynamic route parameters ([...puckPath]) combined with client-side rendering suggests a pattern for creating flexible, route-based editing interfaces that can adapt to various content structures while maintaining interactivity

## Consequences

Positive:
- Enables rich, interactive user experiences for external API consumers with immediate feedback and real-time updates
- Clear separation between client-side and server-side concerns improves code maintainability and enables better optimization strategies
- Reduces server load by offloading interactive state management and UI updates to the client
- Facilitates better code splitting and lazy loading of interactive components, improving initial page load performance for non-interactive content

Negative:
- Increases JavaScript bundle size and client-side complexity, potentially impacting initial load time and performance on low-powered devices
- Requires careful security considerations to avoid exposing sensitive API credentials or enabling client-side manipulation of protected resources
- May complicate SEO and accessibility if not properly implemented with progressive enhancement strategies
- Creates dependency on client-side JavaScript execution, which may not be available in all user environments or may be blocked by corporate policies

## Alternatives

- Server-Side Rendering with Form Actions (rejected)
  Rejected because: Server-side rendering with form submissions would require full page reloads for each interaction, making visual editing workflows impractical and degrading user experience for drag-and-drop and live preview features
  When valid: Appropriate for simple CRUD operations or administrative interfaces where real-time interactivity is not required
- Hybrid Approach with Islands Architecture (deferred)
  Rejected because: While islands architecture could provide benefits by server-rendering static portions and client-rendering interactive islands, the current framework (Next.js) and use case (full-page visual editor) do not align well with partial hydration patterns
  When valid: Should be reconsidered for pages with mixed static and interactive content where only specific components require client-side interactivity
- WebSocket-Based Real-Time Server Rendering (rejected)
  Rejected because: Maintaining WebSocket connections for real-time server rendering would increase infrastructure complexity and costs while still requiring significant client-side JavaScript for state management and UI updates
  When valid: May be appropriate for collaborative editing scenarios where multiple users need synchronized views of the same content

## Risks

- Client-side API integrations may inadvertently expose sensitive authentication tokens or API credentials in browser-accessible code
  Mitigation: Implement server-side API proxy endpoints that handle authentication and authorization, passing only sanitized data to client components. Conduct security reviews of all client-side API integration code.
  Owner: Security team and frontend engineering team
- Large JavaScript bundles for client-side rendering may negatively impact performance on mobile devices or slow network connections
  Mitigation: Implement code splitting, lazy loading, and bundle size monitoring. Establish performance budgets and conduct regular performance testing on representative devices and network conditions.
  Owner: Frontend engineering team and performance engineering team
- Over-reliance on client-side rendering may create accessibility barriers for users with JavaScript disabled or using assistive technologies
  Mitigation: Implement progressive enhancement where feasible, provide fallback experiences for non-JavaScript environments, and conduct regular accessibility audits with assistive technology testing.
  Owner: Frontend engineering team and accessibility team

## Implementation Notes

- Use Next.js 'use client' directive at the top of files containing interactive components that integrate with external APIs, and name these files with .client.tsx extension for clarity
- Implement a server-side API proxy layer (e.g., Next.js API routes or route handlers) to handle authentication and sensitive operations, keeping credentials server-side
- Utilize React hooks (useState, useEffect, useContext) or state management libraries (Zustand, Redux) for managing client-side state in API integration components
- Implement error boundaries around client-side API integration components to gracefully handle failures and provide user-friendly error messages
- Consider implementing skeleton screens or loading states to improve perceived performance during initial data fetching from external APIs

## Continuation Context


Verify commands:
- grep -r "'use client'" --include="*.tsx" --include="*.ts" app/puck/ recipes/*/app/puck/
- find . -name "*client.tsx" -o -name "*client.ts" | xargs grep -l "puck"
- grep -r "API_KEY\|SECRET\|TOKEN" --include="*.client.tsx" --include="*.client.ts" . && echo "FAIL: Credentials found in client files" || echo "PASS: No credentials in client files"

Accept when:
- All interactive API integration components for visual editors contain 'use client' directive and are isolated in files with .client.tsx naming convention
- No sensitive API credentials, tokens, or secrets are present in client-side component files
- Client-side components implement proper error boundaries and loading states for external API calls
- Server-side API proxy endpoints are in place for authentication and sensitive operations

## Enforcement

- Verified by: Automated static analysis in CI pipeline scanning for 'use client' directive in interactive components
- Verified by: ESLint rules enforcing client/server component separation and naming conventions
- Verified by: Security scanning tools checking for exposed credentials in client-accessible code
- Verified by: Code review checklist items for API integration patterns and client-side rendering decisions
- Violation handling: CI pipeline fails if client-side components lack proper 'use client' directive or contain exposed credentials
- Violation handling: Pull requests blocked until code review confirms proper client/server separation and security practices
- Violation handling: Security violations trigger immediate incident response and credential rotation procedures
- Violation handling: Performance budget violations require optimization work or architectural review before merge
- Exception process: Request exception through architecture review board with documented justification and alternative approach
- Exception process: Provide evidence that alternative pattern meets or exceeds user experience and security requirements
- Exception process: Obtain approval from security team for any deviations from credential handling requirements
- Exception process: Document approved exceptions in ADR amendments with time-bound review periods