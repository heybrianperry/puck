# Adopt React Component Creation Pattern with Client-Side State Persistence: Public Contracts Template

These rules are ALWAYS ACTIVE for all React components in `apps/demo/config/blocks` that require client-side state persistence, template components that expose public APIs, and components using localStorage for configuration or user preferences.

### Rules

- **R-PUBCON-001** MUST: Public API contracts (Template) MUST be separated from internal implementation contracts (TemplateInternal) to enforce encapsulation.
- **R-PUBCON-002** MUST: All localStorage.getItem calls in template components MUST be wrapped with JSON.parse and provide fallback values using the null coalescing operator (??).
- **R-PUBCON-003** MUST: Template components MUST use the createComponent pattern for instantiation.
- **R-PUBCON-004** MUST: Both TemplateInternal and Template contracts MUST be exported and properly separated in the public API.
- **R-PUBCON-005** SHOULD: Create a shared utility function in @/core/lib for safe localStorage reads that encapsulates the JSON.parse and null coalescing pattern.
- **R-PUBCON-006** SHOULD: Establish naming conventions for localStorage keys (e.g., templateKey) to prevent collisions and enable easier debugging.
- **R-PUBCON-007** SHOULD: Add TypeScript types for all Template and TemplateInternal contracts to catch type mismatches at compile time.

### Verify

```bash
# Detect unsafe localStorage access without JSON.parse and null coalescing
grep -r 'localStorage.getItem' apps/demo/config/blocks/ | grep -v 'JSON.parse' | grep -v '??'

# Verify createComponent pattern usage in Template components
grep -r 'createComponent' apps/demo/config/blocks/Template/ --include='*.tsx' --include='*.ts'

# Verify separation of TemplateInternal and Template contracts
grep -r 'export.*Template' apps/demo/config/blocks/Template/ | grep -E '(TemplateInternal|Template)'
```

**Accept when:**
- All localStorage.getItem calls in template components are wrapped with JSON.parse and provide fallback values using null coalescing operator
- Template components use createComponent pattern for instantiation as evidenced by grep results
- Both TemplateInternal and Template contracts are exported and properly separated in the public API
- No unsafe localStorage access patterns are detected by verification commands

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes to template components. ESLint custom rules and code review checklists MUST enforce these patterns. CI build MUST fail if unsafe localStorage patterns are detected.
</enforcement>