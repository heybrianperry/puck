# Validate JSON.parse Input in React Hooks for localStorage Operations: Parse Failures Logged

These rules are ALWAYS ACTIVE for all React hooks and utility functions that parse JSON from localStorage, particularly in packages/core/lib/use-sidebar-resize.ts, apps/demo/lib/use-demo-data.ts, and similar custom hooks coordinating state hydration from persistent storage.

### Rules

- **R-STORAGE-001** SHOULD: Parse failures SHOULD be logged with context identifying the storage key and operation (load/save).
- **R-STORAGE-002** MUST: All JSON.parse operations on localStorage data MUST be wrapped in try-catch blocks or use a validated parsing utility.
- **R-STORAGE-003** MUST: Hooks MUST gracefully degrade to default/fallback values when localStorage contains malformed or corrupted JSON.
- **R-STORAGE-004** SHOULD: A shared utility function parseLocalStorageJSON(key, fallback) SHOULD be created to encapsulate try-catch and logging logic for reuse across hooks.

### Verify

```bash
# Count unvalidated JSON.parse operations on localStorage
grep -r 'JSON\.parse.*localStorage' --include='*.ts' --include='*.tsx' | grep -v 'try' | wc -l

# Count validated parsing utility usage
grep -r 'parseLocalStorageJSON\|safeParseJSON' --include='*.ts' --include='*.tsx' | wc -l

# Verify localStorage error handling tests exist
npm test -- --testPathPattern='use-.*\.test' --testNamePattern='localStorage.*invalid'
```

**Accept when:**
- All JSON.parse operations on localStorage are wrapped in try-catch or use a validated parsing utility function
- Unit tests exist verifying hooks handle malformed localStorage data (null, undefined, invalid JSON) without throwing exceptions
- Error logging captures localStorage parse failures with sufficient context (storage key and operation type) for debugging
- Hooks implement graceful fallback to default values when localStorage parsing fails

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse operations on localStorage data MUST be validated before use. Code review and CI pipeline checks are mandatory before merge.
</enforcement>