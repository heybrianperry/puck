# Implement Message Queue Logging for State Management Operations: Logs Include Correlation

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The system processes state management operations through message queue boundaries, requiring visibility into message flow and processing
- Debugging distributed state changes across component boundaries is challenging without structured logging at queue interaction points
- Message queue operations represent critical integration points where failures can cascade across the system
- Operational teams need to trace message lifecycle from enqueueing through processing to completion or failure
- The pattern was detected in core state management modules (use-puck.ts) and UI components (Puck/index.tsx) with 90.20% confidence across 2 files

## Problem Statement

Without standardized logging at message queue boundaries in state management operations, teams lack visibility into message flow, cannot effectively debug distributed state changes, and struggle to diagnose failures that occur during asynchronous message processing between components.

## Decision

1. SHOULD: Logs SHOULD include correlation IDs that allow tracing a message through its entire lifecycle across components

## Policy Block

- SHOULD Logs SHOULD include correlation IDs that allow tracing a message through its entire lifecycle across components

In scope:
- State management operations that cross component boundaries via message queues
- Asynchronous message processing in core state management modules
- UI component interactions that trigger state changes through queued messages
- Message queue integration points in use-puck.ts and Puck component hierarchy

Out of scope:
- Synchronous function calls within a single component
- Direct state mutations that do not involve message queues
- Third-party library internal logging
- Browser console logging for development debugging

Exceptions:
- EXC-001: High-frequency message operations where logging would degrade performance below acceptable thresholds
- EXC-002: Messages containing highly sensitive data where even masked logging poses security risks

## Rationale

- Pattern detected with 90.20% confidence across 2 critical files in state management layer, indicating established architectural practice
- Message queue boundaries represent natural observability points where state transitions occur between loosely coupled components
- Structured logging at these boundaries enables distributed tracing and root cause analysis without requiring invasive instrumentation
- The pattern's presence in both core logic (use-puck.ts) and UI components (Puck/index.tsx) suggests a cross-cutting concern that benefits from standardization

## Consequences

Positive:
- Improved debuggability of distributed state changes through end-to-end message tracing
- Faster incident response with clear visibility into message queue operations and failures
- Better operational metrics for queue performance, backlog monitoring, and throughput analysis
- Reduced mean time to resolution (MTTR) for issues involving asynchronous state management

Negative:
- Increased log volume and storage costs, especially for high-throughput message queues
- Potential performance overhead from logging operations in critical message processing paths
- Risk of logging sensitive data if payload redaction is not properly implemented
- Additional development effort to implement and maintain consistent logging across all queue boundaries

## Alternatives

- Use distributed tracing framework (OpenTelemetry) instead of structured logging (rejected)
  Rejected because: Adds significant infrastructure complexity and third-party dependencies; logging provides sufficient visibility for current scale
  When valid: Consider when system scales beyond 10,000 messages/second or requires cross-service tracing
- Implement event sourcing with built-in audit trail instead of separate logging (rejected)
  Rejected because: Requires fundamental architectural change to event-sourced model; too invasive for incremental improvement
  When valid: Consider during major architecture refactoring if full event history is required for business logic
- Use sampling-based logging to reduce volume (log 1 in N messages) (deferred)
  Rejected because: May be needed for high-throughput scenarios but not required at current scale
  When valid: Implement when log volume exceeds storage capacity or performance impact becomes measurable

## Risks

- Performance degradation in high-throughput message processing due to synchronous logging operations
  Mitigation: Use asynchronous logging with buffering; implement sampling for high-frequency operations; monitor performance metrics
  Owner: Engineering team
- Accidental logging of sensitive user data or credentials in message payloads
  Mitigation: Implement mandatory payload sanitization; conduct security review of logging code; use allowlist approach for logged fields
  Owner: Security team
- Log storage costs escalating with system growth and message volume increases
  Mitigation: Implement log retention policies; use log aggregation with compression; monitor storage costs and set up alerts
  Owner: Operations team

## Implementation Notes

- Create a centralized logging utility for message queue operations to ensure consistent format and correlation ID propagation
- Implement structured logging with JSON format to enable efficient querying and analysis in log aggregation systems
- Add logging instrumentation to existing message queue boundaries in use-puck.ts and Puck/index.tsx as reference implementations
- Configure log levels appropriately: INFO for enqueue/dequeue, WARN for retries, ERROR for failures, DEBUG for detailed payloads
- Use correlation IDs that persist across the entire message lifecycle and can be traced back to originating user actions

## Continuation Context


Verify commands:
- grep -r "queue.*log\|enqueue.*log\|dequeue.*log" packages/core/lib packages/core/components --include="*.ts" --include="*.tsx"
- grep -r "correlationId\|correlation_id" packages/core/lib packages/core/components --include="*.ts" --include="*.tsx"
- npm test -- --grep "message queue logging" --reporter json | jq '.tests[] | select(.title | contains("logs"))'

Accept when:
- All message queue enqueue and dequeue operations in use-puck.ts and Puck/index.tsx include structured log statements
- Correlation IDs are present in logs and can be traced through message lifecycle from enqueue to completion
- Unit tests verify that logging occurs for success and failure scenarios in message processing

## Enforcement

- Verified by: Code review checklist requiring logging verification for all message queue operations
- Verified by: Automated static analysis scanning for queue operations without corresponding log statements
- Verified by: Integration tests validating presence of expected log entries for message processing scenarios
- Violation handling: CI pipeline fails if static analysis detects unlogged queue operations
- Violation handling: Code review blocks merge if logging requirements are not met
- Violation handling: Post-deployment monitoring alerts if expected log patterns are missing from production logs
- Exception process: Submit exception request to architecture team with performance data or security justification
- Exception process: Document alternative observability mechanism in ADR exception log
- Exception process: Obtain approval from relevant stakeholders (architecture, security, or performance team)
- Exception process: Add exception documentation to code comments with reference to approval