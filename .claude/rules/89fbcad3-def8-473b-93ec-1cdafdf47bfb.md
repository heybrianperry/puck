<rule_activation id="89fbcad3-def8-473b-93ec-1cdafdf47bfb" title="Implement Middleware-Based Boundary Control for Public API Data Processing: Middleware Components Chained" applies_to="**/*">
These rules are ALWAYS ACTIVE for all public-facing REST API endpoints, external API integrations, GraphQL resolvers, webhook payload transformations, and API response serialization layers.
</rule_activation>

### Rules

- **R-BOUNDARY-001** MAY: Middleware components MAY be chained to create transformation pipelines for complex data processing requirements.

### Scope

**In scope:**
- All public-facing REST API endpoints
- External API integrations that expose internal data
- GraphQL resolvers that return data to external consumers
- Webhook payload transformations
- API response serialization layers

**Out of scope:**
- Internal service-to-service communication
- Database query result processing (unless directly feeding an API response)
- Background job data transformations
- Internal admin interfaces
- Development/debugging endpoints not exposed to external consumers

**Exceptions:**
- EXC-001: Simple pass-through endpoints that return data without transformation (e.g., health checks, static configuration)
- EXC-002: Performance-critical endpoints where middleware overhead is measured and documented as unacceptable

### Verify

```bash
# Verify middleware pattern presence in core packages
grep -r "export.*Reducer\|export.*Walker" packages/core --include="*.ts" | wc -l

# Find middleware, transformer modules with reducer or walk patterns
find packages -name "*middleware*" -o -name "*transformer*" | grep -E "(reducer|walk)" | wc -l

# Check for middleware decorators and transformation logic in API layer
grep -r "@Middleware\|middleware" packages/*/api --include="*.ts" | grep -E "(transform|reduce|walk)" | wc -l
```

**Accept when:**
- At least 2 middleware modules (reducer, walk-tree) are present in core packages and exported for reuse
- API controllers delegate transformation logic to middleware components rather than implementing transformations inline
- Grep commands return non-zero counts indicating presence of middleware pattern implementations

<enforcement>
Claude Code MUST verify middleware-based boundary control compliance before approving API endpoint implementations. Verification is mandatory and MUST NOT be skipped or deferred.
</enforcement>