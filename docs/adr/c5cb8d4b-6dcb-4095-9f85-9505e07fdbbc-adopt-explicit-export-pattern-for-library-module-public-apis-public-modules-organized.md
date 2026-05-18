# Adopt Explicit Export Pattern for Library Module Public APIs: Public Modules Organized

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains a library/framework architecture (Puck) with core packages that expose public APIs to consumers through explicit module exports
- 47 files across the packages/core directory demonstrate a consistent pattern of organizing functionality into discrete, importable modules with clear boundaries
- The pattern includes utility functions (flatten-data, populate-ids, get-ids-for-parent), React components (LayerTree, Loader, OutlineList), and hooks (use-preview-mode-hotkeys, use-component-list, use-reset-auto-zoom)
- The library serves both internal consumption (within the framework) and external consumption (by framework users), requiring stable public contracts
- The architecture supports server-side rendering (SlotRender/server.tsx) and API routes (Next.js route handlers), indicating multi-environment module usage

## Problem Statement

Without explicit export patterns and public API contracts, library modules risk exposing internal implementation details, creating tight coupling between consumers and internals, and making it difficult to evolve the codebase without breaking changes. The library needs a standardized approach to defining and maintaining public API boundaries across its 47+ modules.

## Decision

1. MUST: Public API modules MUST be organized in a clear directory structure that distinguishes between public exports (lib/, components/) and internal implementation

## Policy Block

- MUST Public API modules MUST be organized in a clear directory structure that distinguishes between public exports (lib/, components/) and internal implementation

In scope:
- All modules under packages/core/lib/
- All modules under packages/core/components/
- Public-facing hooks and utilities
- API route handlers that serve as integration points
- Server and client rendering modules

Out of scope:
- Internal test utilities and fixtures
- Build configuration and tooling scripts
- Private implementation details not exposed through package.json exports
- Development-only modules and debugging tools

Exceptions:
- EXC-001: A module needs to expose internal APIs for advanced use cases or plugin development
- EXC-002: Server-side modules need to share types with client modules

## Rationale

- The detected pattern across 47 files with 87.57% confidence indicates a mature, intentional architectural approach to module organization
- Explicit export patterns enable better tree-shaking, smaller bundle sizes, and clearer dependency graphs for library consumers
- Consistent module structure reduces cognitive load for contributors and makes the codebase more maintainable as it scales
- Clear public API boundaries allow the library to evolve internals without breaking consumer code, supporting semantic versioning

## Consequences

Positive:
- Improved discoverability of public APIs through consistent file naming and export patterns
- Better IDE autocomplete and TypeScript inference for library consumers
- Reduced bundle sizes through effective tree-shaking of unused exports
- Clearer separation of concerns between public contracts and private implementation
- Easier to maintain backward compatibility and follow semantic versioning

Negative:
- Additional boilerplate required for index files and explicit export declarations
- Potential for export sprawl if not carefully managed through code review
- Learning curve for contributors unfamiliar with explicit export patterns
- May require refactoring existing modules that don't follow the pattern

## Alternatives

- Barrel exports with single package entry point (index.ts re-exporting everything) (rejected)
  Rejected because: Prevents effective tree-shaking, increases bundle size, and makes it harder to track what is actually public API versus internal
  When valid: Only appropriate for very small libraries with fewer than 10 exports
- No explicit export management, rely on file system conventions only (rejected)
  Rejected because: Lacks enforcement mechanism, makes it easy to accidentally expose internals, and provides no guidance for consumers on what is stable API
  When valid: Only suitable for internal-only codebases not distributed as libraries
- Use package.json 'exports' field to define public API surface (accepted)
  When valid: Complements the file-based pattern by providing package-level enforcement of public APIs

## Risks

- Inconsistent application of export patterns across the codebase leading to confusion
  Mitigation: Establish linting rules and code review checklist items to verify export patterns; document patterns in contributing guide
  Owner: Engineering team
- Breaking changes when refactoring internal modules that were accidentally consumed as public APIs
  Mitigation: Use package.json exports field to explicitly allowlist public modules; run deprecation warnings before removal
  Owner: Library maintainers
- Server/client module boundary violations causing runtime errors or bundle bloat
  Mitigation: Implement build-time checks for server-only imports in client bundles; use .server.tsx/.client.tsx naming conventions
  Owner: Build tooling team

## Implementation Notes

- Use TypeScript's 'export type' for type-only exports to prevent runtime inclusion in bundles
- Establish naming conventions: index.tsx for component exports, descriptive names for utilities (e.g., flatten-data.ts, populate-ids.ts)
- Document public APIs in a centralized API reference, generated from TSDoc comments on exported members
- Consider using API Extractor or similar tools to generate API reports and detect unintentional breaking changes
- For server-specific modules, use .server.tsx extension and configure bundler to exclude from client builds

## Continuation Context


Verify commands:
- grep -r 'export {' packages/core/lib packages/core/components | wc -l
- find packages/core -name 'index.tsx' -o -name 'index.ts' | wc -l
- npx api-extractor run --local || echo 'API surface check'
- grep -r '\.server\.' packages/core/components | grep -v node_modules

Accept when:
- All public-facing modules under packages/core/lib and packages/core/components have explicit export statements
- No internal helper functions or private utilities are exported from public API modules
- Server-specific modules use .server.tsx extension and are not imported in client-side code
- Package.json exports field restricts access to only intended public modules

## Enforcement

- Verified by: Automated linting rules checking for export patterns in public modules
- Verified by: CI pipeline running API Extractor to detect breaking changes
- Verified by: Code review checklist requiring verification of export declarations
- Verified by: Build-time validation of server/client module boundaries
- Violation handling: CI build fails if API Extractor detects unintended breaking changes
- Violation handling: Linter warnings for modules missing explicit exports in public directories
- Violation handling: Code review blocks merge if new public APIs lack documentation
- Violation handling: Runtime warnings in development mode for deprecated export usage
- Exception process: Submit architecture review request documenting the need for exception
- Exception process: Obtain approval from library maintainers with justification
- Exception process: Document exception in ADR amendments or inline code comments
- Exception process: Add exception to linter configuration with explanatory comment