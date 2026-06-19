# Adopt React Component Creation Pattern with Client-Side State Persistence: Components Not Directly

These rules are ALWAYS ACTIVE for all React components in apps/demo/config/blocks that require client-side state persistence, template components that expose public APIs, components using localStorage for configuration or user preferences, and UI components created through the createComponent factory pattern.

### Rules

- **R-COMP-001** MUST NOT: Components MUST NOT directly parse localStorage values without null coalescing or error handling.

### Verify

```bash
# Detect direct localStorage.getItem calls without JSON.parse and null coalescing
grep -r 'localStorage.getItem' apps/demo/config/blocks/ | grep -v 'JSON.parse' | grep -v '??'

# Verify createComponent pattern usage in template components
grep -r 'createComponent' apps/demo/config/blocks/Template/ --include='*.tsx' --include='*.ts'

# Verify separation of TemplateInternal and Template contracts
grep -r 'export.*Template' apps/demo/config/blocks/Template/ | grep -E '(TemplateInternal|Template)'
```

**Accept when:**
- All localStorage.getItem calls in template components are wrapped with JSON.parse and provide fallback values using null coalescing operator (??)
- Template components use createComponent pattern for instantiation as evidenced by grep results
- Both TemplateInternal and Template contracts are exported and properly separated in the public API

<enforcement>
Clause MUST NOT skip or defer verification. ESLint custom rules detecting unsafe localStorage access patterns, code review checklists, and automated security scanning in CI pipeline are mandatory enforcement mechanisms. CI build fails if violations are detected.
</enforcement>