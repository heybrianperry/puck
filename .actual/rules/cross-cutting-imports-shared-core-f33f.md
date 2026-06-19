# Standardize Core Library Imports via Aliased Module Paths: Imports Shared Core

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript files within monorepo applications (apps/*), including component files, configuration files, and build tool configurations that import from shared core libraries and utility modules.

### Rules

- **R-CORE-001** MUST: All imports from shared core libraries MUST use path aliases (e.g., @/core) rather than relative paths when crossing application or package boundaries.

### Verify

```bash
# Count files using @/core path aliases
grep -r "from ['\"]@/core" apps/demo apps/docs | wc -l

# Count relative path imports that cross boundaries (should be minimal)
grep -r "from ['\"]\.\.\." apps/demo/config apps/docs/components | grep -v "styles.module.css" | wc -l

# Verify tsconfig.json files define @/core path aliases
find apps -name 'tsconfig.json' -exec grep -l '"@/core"' {} \;
```

**Accept when:**
- At least 80% of imports from core libraries use path aliases rather than relative paths
- All tsconfig.json files in the monorepo define consistent path alias mappings for @/core
- No relative path imports (../) are used to cross application or package boundaries

<enforcement>
Claude Code MUST NOT skip or defer verification. Violations block CI builds and pull requests until imports are corrected to use path aliases.
</enforcement>