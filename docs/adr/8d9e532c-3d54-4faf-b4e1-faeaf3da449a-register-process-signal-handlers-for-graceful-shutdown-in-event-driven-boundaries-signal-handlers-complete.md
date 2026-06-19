# Register Process Signal Handlers for Graceful Shutdown in Event-Driven Boundaries: Signal Handlers Complete

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains Node.js CLI applications and library components that execute long-running operations including file system operations, external process execution via execSync, and interactive prompts using inquirer
- Process termination signals (SIGINT, SIGTERM) can arrive during file system writes, git operations, or npm/yarn/pnpm package installations, potentially leaving the system in an inconsistent state
- The create-puck-app CLI tool performs multi-step operations including directory creation, template compilation with Handlebars, dependency installation, and git repository initialization that require coordinated cleanup
- Event-driven boundaries are established using process.on() handlers to intercept termination signals and coordinate graceful shutdown sequences before process exit

## Problem Statement

CLI tools and library components that perform multi-step file system operations, external process execution, and state mutations need a mechanism to handle unexpected termination signals without leaving partial writes, orphaned processes, or corrupted state. Without explicit signal handlers, SIGINT and SIGTERM events cause immediate process termination, preventing cleanup of temporary resources and potentially corrupting in-progress operations.

## Decision

1. MUST: Signal handlers MUST complete or abort in-progress operations before allowing process termination

## Policy Block

- MUST Signal handlers MUST complete or abort in-progress operations before allowing process termination

In scope:
- CLI applications performing file system mutations
- Tools executing external package managers (npm, yarn, pnpm)
- Applications performing git operations via execSync
- Interactive applications using inquirer or similar prompt libraries
- Long-running Node.js processes with stateful operations

Out of scope:
- Stateless HTTP request handlers with framework-managed lifecycle
- Short-lived scripts completing in under 1 second
- Read-only operations without side effects
- Applications where immediate termination is acceptable

Exceptions:
- EXC-001: The application runs in a containerized environment with external orchestration handling graceful shutdown

## Rationale

- Evidence shows process.on('SIGINT', handleSigTerm) and process.on('SIGTERM', handleSigTerm) patterns in packages/create-puck-app/index.js, establishing event-driven boundaries for termination handling
- The CLI tool performs non-atomic operations including fs.mkdirSync, fs.writeFileSync, execSync for package installation, and git operations that require coordinated cleanup to prevent partial state
- Signal handlers provide a standard Node.js mechanism to intercept termination events and execute cleanup logic before process exit, preventing resource leaks and data corruption
- The pattern appears in 2 files with 88.55% confidence, indicating consistent adoption in CLI tooling and potentially in editor components that register event handlers (editor.on('focus', handleFocus) pattern in use-synced-editor.ts)

## Consequences

Positive:
- Prevents partial file system writes and corrupted application state when users interrupt CLI operations with Ctrl+C
- Enables cleanup of temporary resources, child processes, and file handles before process termination
- Provides consistent user experience by allowing operations to complete or roll back cleanly
- Reduces risk of orphaned processes from execSync calls to package managers and git

Negative:
- Adds complexity to application lifecycle management requiring explicit handler registration and cleanup coordination
- Signal handlers can delay process termination if cleanup operations are slow or blocking
- Incorrect handler implementation can prevent process termination entirely, requiring SIGKILL
- Requires testing of signal handling paths which are difficult to reproduce in automated tests

## Alternatives

- Rely on operating system cleanup of file handles and child processes without explicit signal handlers (rejected)
  Rejected because: OS cleanup does not guarantee atomicity of multi-step operations like template generation followed by package installation, leading to partial application state that confuses users
  When valid: Acceptable for read-only operations or stateless scripts where partial execution has no side effects
- Use process.once() instead of process.on() for signal handlers to prevent multiple invocations (deferred)
  Rejected because: Not rejected; process.once() may be preferable to prevent re-entrant cleanup logic, but evidence shows process.on() usage
  When valid: When cleanup logic is not idempotent and multiple signal deliveries could cause errors
- Implement transaction-like semantics with rollback for file system operations instead of signal handlers (rejected)
  Rejected because: Requires complex state tracking and rollback logic for each operation type; signal handlers provide a simpler interception point for cleanup
  When valid: When operations are complex enough to warrant full transaction semantics with commit/rollback phases

## Risks

- Signal handlers that perform blocking I/O or long-running cleanup can make the application appear unresponsive to termination signals
  Mitigation: Implement timeout mechanisms in handlers and use asynchronous cleanup where possible; log progress to indicate handler is executing
  Owner: Engineering team
- Errors thrown within signal handlers can prevent cleanup completion and leave resources in inconsistent state
  Mitigation: Wrap handler logic in try-catch blocks and ensure process.exit() is called even on error paths
  Owner: Engineering team
- Signal handlers may not be invoked in all termination scenarios (e.g., SIGKILL, process crashes, out-of-memory)
  Mitigation: Design operations to be resumable or self-healing; use lock files or state markers to detect incomplete operations on restart
  Owner: Engineering team

## Implementation Notes

- Register signal handlers early in application initialization before performing any stateful operations
- Use a shared handler function (e.g., handleSigTerm) for both SIGINT and SIGTERM to ensure consistent cleanup behavior
- Track resources requiring cleanup (file handles, child processes, temporary directories) in module-level state accessible to handlers
- Call process.exit() with appropriate exit code at the end of signal handlers to ensure process termination after cleanup

## Continuation Context


Verify commands:
- grep -r "process\.on.*SIGINT" --include="*.js" --include="*.ts" packages/
- grep -r "process\.on.*SIGTERM" --include="*.js" --include="*.ts" packages/
- grep -r "handleSigTerm\|handleSignal" --include="*.js" --include="*.ts" packages/

Accept when:
- All CLI applications and long-running Node.js processes register handlers for both SIGINT and SIGTERM signals
- Signal handlers complete cleanup of file system operations, child processes, and resources before calling process.exit()
- Grep commands identify signal handler registration in application entry points

## Enforcement

- Verified by: Code review checklist requiring signal handler registration for CLI tools and stateful applications
- Verified by: Automated grep-based verification in CI pipeline checking for process.on('SIGINT') and process.on('SIGTERM') patterns
- Verified by: Manual testing of Ctrl+C interruption during long-running operations to verify cleanup behavior
- Violation handling: CI pipeline warnings for CLI applications lacking signal handler registration
- Violation handling: Code review rejection for new CLI tools without documented signal handling strategy
- Violation handling: Post-incident review for production issues related to incomplete cleanup or resource leaks
- Exception process: Document justification for omitting signal handlers in application README or architecture documentation
- Exception process: Obtain architecture review approval for stateful applications that rely on external shutdown mechanisms
- Exception process: Add inline comments explaining why signal handlers are not required for specific use cases