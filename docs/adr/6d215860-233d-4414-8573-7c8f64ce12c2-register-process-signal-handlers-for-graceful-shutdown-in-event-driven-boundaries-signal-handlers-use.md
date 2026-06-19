# Register Process Signal Handlers for Graceful Shutdown in Event-Driven Boundaries: Signal Handlers Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase includes Node.js CLI applications that perform long-running operations such as file system manipulation, template compilation with handlebars, and external process execution via execSync
- Process termination signals (SIGINT, SIGTERM) can interrupt these operations mid-execution, potentially leaving the file system in an inconsistent state or orphaning child processes
- The create-puck-app package demonstrates explicit signal handling with process.on('SIGINT', handleSigTerm) and process.on('SIGTERM', handleSigTerm) to coordinate cleanup before exit
- Event-driven boundaries are established at the process level to intercept termination signals and execute cleanup logic, separating normal execution flow from shutdown coordination
- The pattern appears in CLI tooling that orchestrates multi-step workflows involving file I/O, dependency installation, and git operations where partial completion creates technical debt

## Problem Statement

CLI applications that perform multi-step file system operations, template generation, and external process execution require coordinated shutdown behavior to prevent partial state, orphaned processes, or corrupted artifacts when users interrupt execution or when the runtime environment sends termination signals.

## Decision

1. SHOULD: Signal handlers SHOULD use a common cleanup function (e.g., handleSigTerm) to ensure consistent shutdown behavior across different signal types

## Policy Block

- SHOULD Signal handlers SHOULD use a common cleanup function (e.g., handleSigTerm) to ensure consistent shutdown behavior across different signal types

In scope:
- Node.js CLI applications in the packages/create-puck-app directory
- Long-running operations involving file system writes, template compilation, or external process execution
- Interactive CLI tools that accept user input via inquirer or similar prompt libraries
- Applications that use execSync, spawn, or fork to manage child processes

Out of scope:
- Short-lived scripts that complete in under 1 second
- Pure computation tasks with no external side effects
- Web server processes where framework-level shutdown hooks are preferred
- Browser-based applications where process signals are not applicable

## Rationale

- The evidence shows explicit signal handling in packages/create-puck-app/index.js with process.on('SIGINT', handleSigTerm) and process.on('SIGTERM', handleSigTerm), demonstrating a deliberate pattern for coordinating shutdown in event-driven boundaries
- CLI applications that perform file system operations (fs.mkdirSync, fs.writeFileSync) and spawn child processes (execSync for package installation and git operations) require cleanup coordination to prevent partial state
- The pattern separates normal execution flow from shutdown coordination by establishing event-driven boundaries at the process level, allowing the application to intercept termination signals before the runtime forcibly exits
- With 2 files showing this pattern at 88.55% confidence, the approach represents a consistent architectural choice for managing process lifecycle in CLI tooling

## Consequences

Positive:
- Prevents partial file system state when users interrupt CLI operations with Ctrl+C or when container orchestrators send SIGTERM
- Enables cleanup of child processes spawned via execSync, preventing orphaned npm/yarn/pnpm install processes
- Provides a single coordination point (handleSigTerm) for shutdown logic, improving maintainability
- Allows logging of shutdown events for debugging interrupted operations

Negative:
- Adds complexity to CLI application initialization with additional event handler registration
- Signal handlers must complete quickly to avoid delaying process termination, limiting cleanup scope
- Requires careful testing of shutdown paths that are not exercised during normal execution
- May introduce race conditions if cleanup logic is not idempotent or if multiple signals arrive simultaneously

## Alternatives

- Rely on Node.js default signal handling without explicit process.on() registration (rejected)
  Rejected because: Default behavior terminates the process immediately without cleanup, leaving file system in inconsistent state and orphaning child processes spawned via execSync
  When valid: Acceptable for short-lived scripts with no external side effects or child processes
- Use process.once() instead of process.on() for signal handlers (rejected)
  Rejected because: process.once() only handles the first signal, preventing graceful handling of repeated interrupts or multiple signal types with shared cleanup logic
  When valid: Valid when cleanup logic is guaranteed to complete before a second signal could arrive
- Implement cleanup via try-finally blocks around each operation (deferred)
  Rejected because: Try-finally does not intercept process signals, only handles exceptions within the current execution context
  When valid: Complementary approach for exception handling within normal execution flow, but does not replace signal handling

## Risks

- Signal handlers that perform async operations may not complete before process termination if cleanup takes too long
  Mitigation: Keep signal handlers synchronous or use process.exit() only after awaiting critical cleanup; set timeout for cleanup operations
  Owner: engineering team
- Multiple signal handlers registered across different modules may execute in unpredictable order, causing cleanup race conditions
  Mitigation: Centralize signal handler registration in application entry point; document cleanup order dependencies; ensure cleanup operations are idempotent
  Owner: engineering team
- Child processes spawned with { stdio: 'inherit' } may not terminate cleanly if parent signal handler does not explicitly kill them
  Mitigation: Track child process PIDs and send SIGTERM to each during cleanup; use process groups where available
  Owner: engineering team

## Implementation Notes

- Register signal handlers immediately after importing dependencies and before executing any long-running operations to ensure coverage of the entire application lifecycle
- Create a shared cleanup function (e.g., handleSigTerm) that can be reused across SIGINT and SIGTERM handlers to ensure consistent behavior
- For applications using execSync, consider switching to spawn with explicit process handle tracking to enable child process termination during cleanup
- Test signal handling by sending SIGINT (Ctrl+C) and SIGTERM during various stages of CLI execution to verify cleanup behavior

## Continuation Context


Verify commands:
- grep -r "process\.on.*SIGINT" packages/create-puck-app/ packages/core/
- grep -r "process\.on.*SIGTERM" packages/create-puck-app/ packages/core/
- node -e "const fs = require('fs'); const files = ['packages/create-puck-app/index.js']; files.forEach(f => { const content = fs.readFileSync(f, 'utf-8'); if (!content.includes('process.on') || (!content.includes('SIGINT') && !content.includes('SIGTERM'))) { process.exit(1); } });"

Accept when:
- All Node.js CLI applications that perform file system operations register handlers for both SIGINT and SIGTERM signals
- Signal handlers reference a cleanup function that executes before process termination
- Grep commands return matches in packages/create-puck-app/index.js showing process.on('SIGINT') and process.on('SIGTERM') patterns

## Enforcement

- Verified by: Code review checklist requiring signal handler registration for new CLI applications
- Verified by: Automated grep-based verification in CI pipeline checking for process.on('SIGINT') and process.on('SIGTERM') patterns
- Verified by: Manual testing of CLI tools with simulated SIGINT and SIGTERM signals during QA
- Violation handling: PR comments requesting addition of signal handlers for CLI applications missing them
- Violation handling: CI pipeline warnings (non-blocking) when new CLI entry points lack signal handling patterns
- Violation handling: Post-incident review if production CLI tools leave orphaned processes or partial state
- Exception process: Document rationale in code comments if signal handling is intentionally omitted (e.g., script completes in under 1 second)
- Exception process: Obtain approval from tech lead for CLI tools that rely on framework-level shutdown hooks instead of explicit signal handling
- Exception process: Record exception in ADR updates section with justification and scope