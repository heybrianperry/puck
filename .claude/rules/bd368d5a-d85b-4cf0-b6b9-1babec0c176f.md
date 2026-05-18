<rule_activation id="bd368d5a-d85b-4cf0-b6b9-1babec0c176f" title="Standardize External API Documentation with Embedded Analytics and Metadata: Applications Extend Base" applies_to="**/*">
These rules are ALWAYS ACTIVE for all public-facing API documentation, demo applications, developer portals, and marketing pages for APIs.
</rule_activation>

### Rules

- **R-API-DOC-001** MAY: Applications MAY extend base metadata configuration with application-specific properties as needed.
- **R-API-DOC-002** MUST: No hardcoded API keys, secrets, or tokens in source code; all credentials MUST use environment variables.
- **R-API-DOC-003** MUST: All .env files MUST be properly gitignored and not tracked in version control.
- **R-API-DOC-004** SHOULD: Public-facing applications SHOULD include standard metadata tags (Open Graph, Twitter Card, etc.).
- **R-API-DOC-005** SHOULD: External service integrations SHOULD be conditionally loaded based on environment configuration.
- **R-API-DOC-006** SHOULD: Use framework-specific environment variable patterns (e.g., NEXT_PUBLIC_ for Next.js, VITE_ for Vite) to control client-side exposure.
- **R-API-DOC-007** SHOULD: Implement configuration validation layer that checks for required metadata and warns about missing environment variables.
- **R-API-DOC-008** MAY: Non-sensitive public API keys explicitly designed for client-side use with domain restrictions MAY be used (EXC-001).

### Verify

```bash
# Check for hardcoded API keys, secrets, or tokens not using environment variables
grep -r "API_KEY\|SECRET\|TOKEN" --include="*.tsx" --include="*.ts" --include="*.jsx" --include="*.js" | grep -v "process.env" | grep -v "import.meta.env"

# Verify .env files are gitignored
find . -name "*.env" -not -path "*/node_modules/*" -not -path "*/.git/*" | xargs git check-ignore -v

# Count standard metadata tags in public-facing applications
grep -r "<meta.*og:" --include="*.tsx" --include="*.html" | wc -l
```

**Accept when:**
- No hardcoded API keys, secrets, or tokens are found in source code (all use environment variables)
- All .env files are properly gitignored and not tracked in version control
- Public-facing applications include standard metadata tags (Open Graph, Twitter Card, etc.)
- External service integrations are conditionally loaded based on environment configuration
- Framework-specific environment variable patterns are used to control client-side exposure
- Configuration validation is implemented for required metadata

<enforcement>
Claude Code MUST verify all rules before accepting changes to public-facing API documentation, demo applications, developer portals, and marketing pages. Verification is mandatory and MUST NOT be skipped or deferred. CI/CD pipeline MUST fail if secrets are detected in code. Pull requests MUST be blocked until hardcoded credentials are removed.
</enforcement>