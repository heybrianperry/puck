# Adopt React Component Creation Pattern with Client-Side State Persistence: Components Import Core

These rules are ALWAYS ACTIVE for all React components in apps/demo/config/blocks that require client-side state persistence, template components that expose public APIs, components using localStorage for configuration or user preferences, and UI components created through the createComponent factory pattern.

### Rules

- **R-COMP-001** SHOULD: Components SHOULD import core functionality from @/core and @/core/types rather than implementing custom solutions.
- **R-COMP-002** MUST: All localStorage.getItem calls in template components MUST be wrapped with JSON.parse and provide fallback values using null coalescing operator (??)
- **R-COMP-003** MUST: Template components MUST use createComponent pattern for instantiation.
- **R-COMP-004** MUST: Both TemplateInternal and Template contracts MUST be exported and properly separated in the public API.
- **R-COMP-005** SHOULD: Components SHOULD encapsulate JSON.parse and null coalescing patterns through shared utility functions in @/core/lib.
- **R-COMP-006** SHOULD: localStorage keys SHOULD follow naming conventions (e.g., templateKey) to prevent collisions and enable easier debugging.
- **R-COMP-007** SHOULD: All Template and TemplateInternal contracts SHOULD have TypeScript types to catch type mismatches at compile time.

### Verify

```bash
# Detect unsafe localStorage access without JSON.parse and null coalescing
grep -r 'localStorage.getItem' apps/demo/config/blocks/ | grep -v 'JSON.parse' | grep -v '??'

# Verify createComponent pattern usage in template components
grep -r 'createComponent' apps/demo/config/blocks/Template/ --include='*.tsx' --include='*.ts'

# Verify proper separation of TemplateInternal and Template contracts
grep -r 'export.*Template' apps/demo/config/blocks/Template/ | grep -E '(TemplateInternal|Template)'
```

**Accept when:**
- All localStorage.getItem calls in template components are wrapped with JSON.parse and provide fallback values using null coalescing operator
- Template components use createComponent pattern for instantiation as evidenced by grep results
- Both TemplateInternal and Template contracts are exported and properly separated in the public API
- No direct localStorage usage without error handling is detected in the codebase

<enforcement>
Claude Code MUST NOT skip or defer verification. All verify commands MUST pass before accepting changes to components in scope. ESLint custom rules and code review checklists MUST be enforced in CI pipeline. Violations MUST result in build failure and security team notification.
</enforcement>