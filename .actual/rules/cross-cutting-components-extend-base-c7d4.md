# Adopt React Component Creation Pattern with Client-Side State Persistence: Components Extend Base

These rules are ALWAYS ACTIVE for all React components in apps/demo/config/blocks that require client-side state persistence, template components that expose public APIs, components using localStorage for configuration or user preferences, and UI components created through the createComponent factory pattern.

### Rules

- **R-COMP-001** MUST: Wrap all `localStorage.getItem()` calls with `JSON.parse()` and provide fallback values using the null coalescing operator (`??`) to prevent XSS attacks and data corruption.
- **R-COMP-002** MUST: Use the `createComponent` factory pattern for instantiation of React components requiring lifecycle and rendering behaviors.
- **R-COMP-003** MUST: Separate internal template contracts (TemplateInternal) from public-facing contracts (Template) to enforce API boundaries.
- **R-COMP-004** SHOULD: Create shared utility functions in `@/core/lib` for safe localStorage reads that encapsulate the JSON.parse and null coalescing pattern.
- **R-COMP-005** SHOULD: Establish naming conventions for localStorage keys (e.g., templateKey) to prevent collisions and enable easier debugging.
- **R-COMP-006** MAY: Components MAY extend the base Template contract with additional properties specific to their use case.

### Verify

```bash
# Detect unsafe localStorage access without JSON.parse and null coalescing
grep -r 'localStorage.getItem' apps/demo/config/blocks/ | grep -v 'JSON.parse' | grep -v '??'

# Verify createComponent pattern usage in template components
grep -r 'createComponent' apps/demo/config/blocks/Template/ --include='*.tsx' --include='*.ts'

# Verify separation of TemplateInternal and Template contracts
grep -r 'export.*Template' apps/demo/config/blocks/Template/ | grep -E '(TemplateInternal|Template)'
```

**Accept when:**
- All `localStorage.getItem` calls in template components are wrapped with `JSON.parse` and provide fallback values using null coalescing operator (`??`)
- Template components use `createComponent` pattern for instantiation as evidenced by grep results
- Both `TemplateInternal` and `Template` contracts are exported and properly separated in the public API
- No direct `localStorage.getItem()` calls exist without validation or error handling in production code

<enforcement>
Claude Code MUST NOT skip or defer verification. ESLint custom rules detecting unsafe localStorage access patterns, code review checklists, and automated security scanning in CI pipeline are mandatory enforcement mechanisms. Violations result in CI build failure and pull request blocking until remediation is complete.
</enforcement>