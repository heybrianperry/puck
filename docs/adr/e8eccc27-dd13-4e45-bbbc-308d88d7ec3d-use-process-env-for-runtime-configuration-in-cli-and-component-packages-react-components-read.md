# Use process.env for Runtime Configuration in CLI and Component Packages: React Components Read

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The packages/create-puck-app CLI tool and packages/core/components/Puck component both require runtime configuration to adapt behavior based on execution environment
- The codebase uses process.env as the primary configuration source, accessing npm_config_user_agent for package manager detection and NODE_ENV for environment-specific behavior
- CLI tools need to detect the invoking package manager (npm, yarn, pnpm) to provide appropriate installation commands and maintain consistency with user preferences
- React components require environment awareness to enable development-specific features like warnings and debugging without impacting production builds
- The pattern emerged across 2 files with 88.60% confidence, indicating consistent adoption of process.env as the configuration integration point

## Problem Statement

Applications spanning CLI tools and UI components need a consistent mechanism to access runtime configuration without introducing complex configuration management systems or requiring explicit configuration file parsing, while maintaining compatibility with Node.js and bundler environments.

## Decision

1. MUST: React components MUST read process.env.NODE_ENV to determine development vs production mode

## Policy Block

- MUST React components MUST read process.env.NODE_ENV to determine development vs production mode

In scope:
- CLI tools in packages/create-puck-app
- React components in packages/core/components
- Any package requiring runtime environment detection
- Build-time and runtime configuration decisions

Out of scope:
- Static configuration files (package.json, tsconfig.json)
- Build-time constants that do not vary by environment
- Server-side configuration management systems
- Browser-only runtime configuration (localStorage, cookies)

Exceptions:
- EXC-001: Package runs exclusively in browser environments where process.env is not available

## Rationale

- The process.env API is universally available in Node.js environments and supported by modern bundlers (webpack, Vite, etc.) through compile-time replacement, providing zero-dependency configuration access
- Using npm_config_user_agent allows CLI tools to respect the user's package manager choice without requiring explicit flags, improving user experience and reducing configuration burden
- NODE_ENV is the de facto standard for environment detection in JavaScript ecosystems, with widespread tooling support and developer familiarity
- Direct environment variable access eliminates the need for configuration file parsing libraries, reducing bundle size and dependency complexity

## Consequences

Positive:
- Zero runtime dependencies for configuration access, reducing package size and installation time
- Seamless integration with existing Node.js and bundler ecosystems that already populate process.env
- CLI tools can automatically detect and use the appropriate package manager without user intervention
- Development-specific features (warnings, debugging) can be conditionally enabled without production overhead

Negative:
- Environment variables must be set correctly by the execution environment; misconfiguration can cause silent failures
- Bundlers require explicit configuration to replace process.env references at build time for browser targets
- Testing requires mocking or stubbing process.env, adding complexity to test setup
- No type safety or validation for environment variable values without additional runtime checks

## Alternatives

- Use configuration files (e.g., .puckrc.json) for runtime settings (rejected)
  Rejected because: Requires file system access and parsing libraries, increasing bundle size and complexity; less compatible with bundler environments
  When valid: When configuration is complex, hierarchical, or requires user customization beyond environment detection
- Pass configuration explicitly through function parameters or constructor arguments (rejected)
  Rejected because: Requires threading configuration through all call sites; impractical for cross-cutting concerns like environment detection
  When valid: When configuration is component-specific and varies per instance rather than per environment
- Use a configuration management library (e.g., dotenv, config) (rejected)
  Rejected because: Adds unnecessary dependencies for simple environment variable access; process.env is sufficient for current needs
  When valid: When configuration requires validation, schema enforcement, or complex merging logic

## Risks

- Bundlers may not correctly replace process.env references, causing runtime errors in browser environments
  Mitigation: Document bundler configuration requirements; provide example configurations for webpack, Vite, and other common bundlers; test in multiple bundler environments
  Owner: Engineering team
- Missing or incorrect environment variables cause unexpected behavior without clear error messages
  Mitigation: Implement fallback values for all environment variable reads; add validation and warnings when critical variables are missing
  Owner: Engineering team
- Environment variable values may be logged or exposed in error messages, potentially leaking sensitive information
  Mitigation: Avoid logging raw environment variable values; sanitize error messages; document which variables may contain sensitive data
  Owner: Security team

## Implementation Notes

- For CLI tools, use process.env.npm_config_user_agent to detect package manager; parse the user agent string to identify npm, yarn, or pnpm
- For React components, check process.env.NODE_ENV === 'development' to enable development-only warnings and debugging features
- Always provide fallback behavior when environment variables are undefined; use logical OR (||) or nullish coalescing (??) operators
- Document all environment variables used by each package in README.md, including expected values and default behavior
- For browser-targeted packages, ensure bundler configuration includes process.env replacement; test that process.env references are eliminated from production bundles

## Continuation Context


Verify commands:
- grep -r 'process\.env\.' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -E '(npm_config_user_agent|NODE_ENV)'
- node -e "const fs = require('fs'); const files = ['packages/create-puck-app/index.js', 'packages/core/components/Puck/index.tsx']; files.forEach(f => { if (fs.existsSync(f)) { const content = fs.readFileSync(f, 'utf8'); if (!content.includes('process.env')) { console.error('Missing process.env in ' + f); process.exit(1); } } });"
- grep -r 'process\.env\.' packages/ --include='*.js' --include='*.ts' --include='*.tsx' | grep -v -E '(npm_config_user_agent|NODE_ENV|NEXT_PUBLIC_|VITE_)' || true

Accept when:
- All CLI tools in packages/create-puck-app access process.env.npm_config_user_agent for package manager detection
- All React components in packages/core check process.env.NODE_ENV for environment-specific behavior
- No direct process.env access occurs without fallback values or validation
- Documentation exists for all environment variables used by each package

## Enforcement

- Verified by: Automated grep-based verification in CI pipeline checking for process.env usage patterns
- Verified by: Code review checklist requiring fallback values for all environment variable reads
- Verified by: Integration tests running in multiple environments (development, production, test) to verify behavior
- Violation handling: CI pipeline fails if process.env access lacks documented fallback behavior
- Violation handling: Code review blocks merge if new environment variables are introduced without documentation
- Violation handling: Runtime warnings logged in development mode when environment variables are missing
- Exception process: Request architecture review for packages that cannot use process.env (e.g., browser-only packages)
- Exception process: Document alternative configuration mechanism in ADR exception
- Exception process: Obtain approval from tech lead before merging exception