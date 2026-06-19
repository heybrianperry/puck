# Standardize Core Library Imports via Aliased Module Paths: Core Library Imports

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript files within monorepo applications (apps/*), including imports of shared core libraries, utility modules, configuration files, and component files importing from centralized component libraries.

### Rules

- **R-CORE-001** SHOULD: Core library imports SHOULD be organized by subdomain (types, lib, components) using subpath aliases (e.g., @/core/types, @/core/lib).

### Verify

```bash
# Count files using @/core path aliases
grep -r "from ['\"]@/core" apps/demo apps/docs | wc -l

# Count relative path imports crossing boundaries (should be minimal)
grep -r "from ['\"]\.\.\." apps/demo/config apps/docs/components | grep -v "styles.module.css" | wc -l

# Verify all tsconfig.json files define @/core path aliases
find apps -name 'tsconfig.json' -exec grep -l '"@/core"' {} \;
```

**Accept when:**
- At least 80% of imports from core libraries use path aliases rather than relative paths
- All tsconfig.json files in the monorepo define consistent path alias mappings for @/core
- No relative path imports (../) are used to cross application or package boundaries

<enforcement>
Claude Code MUST NOT skip or defer verification. Violations block CI builds and pull requests until imports are corrected to use path aliases.
</enforcement>