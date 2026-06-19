# Adopt React Component Creation Pattern with Client-Side State Persistence: Template Components Use

These rules are ALWAYS ACTIVE for all React template components in `apps/demo/config/blocks/` that require client-side state persistence using localStorage.

### Rules

- **R-TEMPLATE-001** MUST: Template components MUST use the createComponent pattern for instantiation to ensure consistent lifecycle management.
- **R-TEMPLATE-002** MUST: All localStorage.getItem calls in template components MUST be wrapped with JSON.parse and provide fallback values using the null coalescing operator (??).
- **R-TEMPLATE-003** MUST: Template components MUST export both TemplateInternal and Template contracts to enforce API boundaries and prevent direct access to implementation details.
- **R-TEMPLATE-004** MUST: Input validation on localStorage data MUST use JSON.parse with fallback to empty objects to prevent injection attacks and data corruption.
- **R-TEMPLATE-005** SHOULD: Template components SHOULD import shared functionality from @/core and @/core/types rather than duplicating validation logic.

### Verify

```bash
# Check for unsafe localStorage access without JSON.parse and null coalescing
grep -r 'localStorage.getItem' apps/demo/config/blocks/ | grep -v 'JSON.parse' | grep -v '??'

# Verify createComponent pattern usage in template components
grep -r 'createComponent' apps/demo/config/blocks/Template/ --include='*.tsx' --include='*.ts'

# Verify both TemplateInternal and Template contracts are exported
grep -r 'export.*Template' apps/demo/config/blocks/Template/ | grep -E '(TemplateInternal|Template)'
```

**Accept when:**
- All localStorage.getItem calls in template components are wrapped with JSON.parse and provide fallback values using null coalescing operator (??)
- Template components use createComponent pattern for instantiation as evidenced by grep results
- Both TemplateInternal and Template contracts are exported and properly separated in the public API
- No direct localStorage usage without error handling is detected in template component files

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes to template components.
</enforcement>