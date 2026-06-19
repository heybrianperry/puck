# Adopt Fetch-Based External API Client Pattern for Internal API Boundaries: Components Use Additional

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The ReleaseSwitcher component in the documentation application requires dynamic release data that is not available at build time or through static imports
- The application uses Next.js with environment-based configuration (NEXT_PUBLIC_BASE_URL, NEXT_PUBLIC_IS_CANARY, NEXT_PUBLIC_IS_LATEST) to determine runtime API endpoints
- React useEffect hooks trigger client-side data fetching after component mount, requiring an HTTP client mechanism to retrieve data from internal API routes
- Error handling is implemented using console.error for logging fetch failures, indicating a need for observable failure modes in client-side API interactions
- The boundary between UI components and internal API routes is crossed using the native fetch API, establishing a pattern for client-server communication within the same application

## Problem Statement

Client-side React components in a Next.js documentation application need to dynamically retrieve data from internal API routes at runtime, requiring a consistent pattern for making HTTP requests, handling errors, and managing environment-specific endpoint configuration without introducing heavy external dependencies or complex client libraries.

## Decision

1. MAY: Components MAY use additional environment variables (NEXT_PUBLIC_IS_CANARY, NEXT_PUBLIC_IS_LATEST) to conditionally modify API behavior or endpoint selection

## Policy Block

- MAY Components MAY use additional environment variables (NEXT_PUBLIC_IS_CANARY, NEXT_PUBLIC_IS_LATEST) to conditionally modify API behavior or endpoint selection

## Rationale

- The evidence shows fetch API usage in apps/docs/components/ReleaseSwitcher/index.tsx with the pattern fetch(`${BASE_URL}/api/releases`), demonstrating a lightweight approach to internal API communication without external HTTP client dependencies
- Environment variable usage (process.env.NEXT_PUBLIC_BASE_URL, NEXT_PUBLIC_IS_CANARY, NEXT_PUBLIC_IS_LATEST) indicates a deliberate pattern for runtime configuration that supports multiple deployment environments
- The combination of React useEffect hooks with fetch calls establishes a client-side data fetching pattern that separates concerns between component rendering and data retrieval
- Console.error logging for fetch failures provides a minimal but observable error handling mechanism suitable for development and debugging of internal API boundaries

## Consequences

Positive:
- Zero additional dependencies required for HTTP client functionality, reducing bundle size and maintenance overhead
- Native fetch API provides modern Promise-based interface with broad browser support in Next.js client-side contexts
- Environment variable configuration enables flexible deployment across development, staging, and production environments without code changes
- Clear separation between UI components and API routes through HTTP boundaries enables independent testing and development

Negative:
- Native fetch API lacks built-in retry logic, request cancellation, and advanced features provided by specialized HTTP client libraries
- Console.error logging is insufficient for production error monitoring and requires additional observability tooling for comprehensive error tracking
- Each component implementing fetch must handle its own error states, loading states, and retry logic, leading to potential code duplication
- Environment variables must be prefixed with NEXT_PUBLIC_ to be available in client-side code, exposing configuration that may contain sensitive endpoint information

## Alternatives

- Use a dedicated HTTP client library (axios, ky, or fetch wrapper) for internal API communication (rejected)
  Rejected because: Adds external dependency overhead and bundle size for features not required by simple internal API calls; native fetch is sufficient for the documented use case
  When valid: When advanced features like request/response interceptors, automatic retries, request cancellation, or complex error handling are required across multiple components
- Use Next.js getServerSideProps or getStaticProps for server-side data fetching instead of client-side fetch (rejected)
  Rejected because: Release data appears to require runtime client-side fetching based on user interaction or dynamic conditions not available at build time or initial page load
  When valid: When data can be fetched at build time or server-side render time and does not depend on client-side state or user interactions
- Implement a centralized API client service layer with shared error handling and configuration (deferred)
  Rejected because: Current evidence shows only one fetch call; premature abstraction without multiple consumers demonstrating shared patterns
  When valid: When multiple components require API access with consistent error handling, retry logic, authentication, or request/response transformation

## Risks

- Inconsistent error handling across components as more internal API clients are added, leading to poor user experience and difficult debugging
  Mitigation: Establish error handling patterns and consider creating a shared fetch wrapper utility if more than 3 components require similar API access
  Owner: Frontend Engineering Team
- Environment variable misconfiguration or missing NEXT_PUBLIC_ prefix causing runtime failures in client-side code
  Mitigation: Implement startup validation checks for required environment variables and document configuration requirements in deployment guides
  Owner: DevOps and Frontend Engineering Teams
- Lack of request cancellation when components unmount may cause memory leaks or state updates on unmounted components
  Mitigation: Implement AbortController pattern in useEffect cleanup functions for all fetch calls to properly cancel in-flight requests
  Owner: Frontend Engineering Team

## Implementation Notes

- Wrap fetch calls in try-catch blocks within useEffect hooks and implement cleanup functions with AbortController to prevent memory leaks
- Create a shared constants file for API endpoint paths (e.g., /api/releases) to avoid string duplication and enable easier refactoring
- Document required NEXT_PUBLIC_ environment variables in .env.example and deployment documentation with clear descriptions of their purpose
- Consider implementing a simple loading state and error boundary pattern for components that fetch data to provide consistent user feedback

## Continuation Context


Verify commands:
- grep -r "fetch(" apps/docs/components --include="*.tsx" --include="*.ts" | grep -v "node_modules"
- grep -r "process.env.NEXT_PUBLIC" apps/docs --include="*.tsx" --include="*.ts" | grep -v "node_modules"
- grep -r "console.error" apps/docs/components --include="*.tsx" --include="*.ts" -A 2 | grep -i "fetch\|load\|api"

Accept when:
- All fetch calls to internal API routes use native fetch API without external HTTP client library dependencies
- API endpoint URLs are constructed using process.env.NEXT_PUBLIC_* environment variables for base URL configuration
- Error handling with console.error or equivalent logging is present for all fetch operations to capture request failures

## Enforcement

- Verified by: Code review checklist requiring verification of fetch API usage and environment variable configuration
- Verified by: ESLint rules detecting axios or other HTTP client imports in components that should use native fetch
- Verified by: Integration tests validating API client behavior with mocked fetch responses and error conditions
- Violation handling: Pull requests introducing external HTTP client libraries for internal API calls must provide justification for additional dependency
- Violation handling: Missing error handling on fetch calls triggers code review feedback requiring implementation before merge
- Violation handling: Hardcoded API URLs without environment variable configuration are flagged during code review and must be refactored
- Exception process: Request exception through architecture review if advanced HTTP client features (interceptors, automatic retries, complex authentication) are required
- Exception process: Document exception rationale in ADR or technical design document explaining why native fetch is insufficient
- Exception process: Obtain approval from frontend architecture lead before introducing new HTTP client dependencies