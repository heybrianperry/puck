# Standardize Message Queue Integration for Public API Event Distribution: Message Queue Integration

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The system requires asynchronous communication between public API endpoints and internal services to handle event-driven workflows
- Message queues provide decoupling between API request handling and downstream processing, improving response times and system resilience
- Multiple components (tsup-config, store slices for fields and permissions) exhibit consistent patterns of message queue integration at API boundaries
- The pattern appears in 3 files with 90% confidence, indicating an established architectural practice for external API event distribution

## Problem Statement

Public APIs need a consistent mechanism to publish events to internal systems without blocking request-response cycles, while maintaining loose coupling between external interfaces and internal domain logic. Without standardized message queue integration, teams may implement inconsistent event distribution patterns leading to reliability issues and maintenance overhead.

## Decision

1. MUST: Message queue integration MUST be implemented at the API boundary layer, not within core domain logic

## Policy Block

- MUST Message queue integration MUST be implemented at the API boundary layer, not within core domain logic

In scope:
- All public-facing REST APIs that trigger state changes
- External webhook receivers that initiate internal workflows
- GraphQL mutations that require asynchronous processing
- API gateway integration points with internal services

Out of scope:
- Internal service-to-service communication within the same bounded context
- Synchronous read operations and queries
- Administrative APIs used exclusively by internal tooling
- Real-time streaming APIs using WebSocket or SSE protocols

Exceptions:
- EXC-001: API operations require immediate synchronous validation from downstream services that cannot be deferred
- EXC-002: Legacy APIs undergoing gradual migration where immediate refactoring would disrupt existing integrations

## Rationale

- Pattern detected across 3 files (tsup-config, fields store slice, permissions store slice) with 90% confidence indicates this is an established architectural practice
- Message queues at API boundaries enable horizontal scaling of API servers independently from processing workers, improving system elasticity
- Decoupling public APIs from internal service implementations reduces cascading failures and improves overall system availability
- Asynchronous event distribution allows for multiple consumers to react to API events without modifying the API layer

## Consequences

Positive:
- Improved API response times as requests complete immediately after event publication
- Enhanced system resilience through loose coupling between API and internal services
- Better scalability as API servers and event processors can scale independently
- Simplified API implementation as complex business logic is delegated to event consumers

Negative:
- Increased system complexity with additional message queue infrastructure to operate and monitor
- Eventual consistency challenges as API responses may not reflect completed processing state
- Additional operational overhead for message queue monitoring, dead letter handling, and replay mechanisms
- Debugging distributed workflows becomes more complex with asynchronous event chains

## Alternatives

- Direct synchronous service invocation from API endpoints (rejected)
  Rejected because: Creates tight coupling between API and internal services, leading to cascading failures and poor scalability. API response times become dependent on downstream service performance.
  When valid: Only appropriate for simple CRUD operations with no complex business logic or when strong consistency guarantees are absolutely required
- Hybrid approach with synchronous calls for critical paths and async for non-critical operations (rejected)
  Rejected because: Inconsistent patterns across the API surface create confusion for developers and complicate operational procedures. Difficult to maintain clear boundaries between critical and non-critical paths.
  When valid: May be considered during migration phases with clear documentation and sunset timelines
- Event sourcing with event store instead of message queues (deferred)
  Rejected because: Not rejected, but deferred for future consideration. Event sourcing provides stronger guarantees but requires more significant architectural changes.
  When valid: Should be reconsidered if audit requirements or temporal query capabilities become critical system requirements

## Risks

- Message queue outages could prevent API operations from completing, impacting system availability
  Mitigation: Implement circuit breakers, fallback mechanisms, and comprehensive monitoring with alerting. Consider write-ahead log for critical events.
  Owner: Platform Engineering Team
- Event ordering guarantees may be lost in distributed queue systems, causing race conditions
  Mitigation: Use partition keys or message groups for events requiring ordering. Document ordering guarantees in event schemas.
  Owner: API Development Team
- Schema evolution may break existing consumers if not managed carefully
  Mitigation: Implement schema registry with compatibility checking. Enforce backward compatibility rules and version all event schemas.
  Owner: Architecture Review Board

## Implementation Notes

- Use a message queue abstraction layer to avoid vendor lock-in and facilitate testing with in-memory implementations
- Implement idempotency keys in event payloads to handle duplicate message delivery scenarios
- Establish naming conventions for queue names and event types (e.g., api.{domain}.{entity}.{action})
- Configure dead letter queues for failed message handling and implement monitoring dashboards for queue depth and processing latency

## Continuation Context


Verify commands:
- grep -r "publish.*Event\|sendMessage\|enqueue" packages/*/src/api --include="*.ts" | grep -v test
- grep -r "import.*MessageQueue\|import.*EventBus" packages/core packages/tsup-config --include="*.ts"
- find . -name "*.ts" -path "*/api/*" -exec grep -l "await.*service\." {} \; | wc -l

Accept when:
- All public API endpoints that trigger state changes publish events to message queues rather than making direct service calls
- Message queue integration code is located in API boundary layers (controllers, resolvers) and not in core domain services
- Event schemas are documented with version numbers and backward compatibility is maintained across versions

## Enforcement

- Verified by: Automated code review checks scanning for direct service invocations in API layer
- Verified by: Architecture review for new API endpoints during design phase
- Verified by: Integration tests verifying event publication for all state-changing API operations
- Violation handling: CI pipeline fails if API layer contains direct synchronous service calls without documented exception
- Violation handling: Pull requests flagged for architecture review if message queue patterns are not followed
- Violation handling: Quarterly architecture audits identify violations for remediation planning
- Exception process: Submit exception request to Architecture Review Board with justification and performance impact analysis
- Exception process: Document approved exceptions in ADR amendments with expiration dates
- Exception process: Include migration plan for temporary exceptions showing path to compliance