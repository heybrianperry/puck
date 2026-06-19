# Adopt React Component Creation Pattern with Client-Side State Persistence: Localstorage Reads Wrapped

These rules are ALWAYS ACTIVE for all React components in apps/demo/config/blocks that require client-side state persistence, template components that expose public APIs, components using localStorage for configuration or user preferences, and UI components created through the createComponent factory pattern.

### Rules

- **R-STORAGE-001** MUST: All localStorage reads MUST be wrapped with JSON.parse and provide a fallback value to handle missing or corrupted data.

### Verify

```bash
# Check for unsafe localStorage.getItem calls without JSON.parse and null coalescing
grep -r 'localStorage.getItem' apps/demo/config/blocks/ | grep -v 'JSON.parse' | grep -v '??'

# Verify createComponent pattern usage in template components
grep -r 'createComponent' apps/demo/config/blocks/Template/ --include='*.tsx' --include='*.ts'

# Verify proper separation of TemplateInternal and Template contracts
grep -r 'export.*Template' apps/demo/config/blocks/Template/ | grep -E '(TemplateInternal|Template)'
```

**Accept when:**
- All localStorage.getItem calls in template components are wrapped with JSON.parse and provide fallback values using null coalescing operator (??)
- Template components use createComponent pattern for instantiation as evidenced by grep results
- Both TemplateInternal and Template contracts are exported and properly separated in the public API
- No unsafe localStorage access patterns are detected in the grep output

<enforcement>
Clause Code MUST NOT skip or defer verification. All localStorage reads must be validated before merge. ESLint custom rules and code review checklists enforce this requirement in CI/CD pipeline.
</enforcement>