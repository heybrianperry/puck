# Standardize Core Library Imports via Aliased Module Paths: Shared Configuration Packages

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript files within monorepo applications (apps/*), including imports of shared core libraries, utility modules, configuration files that reference shared packages, component files importing from centralized component libraries, and build tool configurations.

### Rules

- **R-CORE-001** MUST: Shared configuration packages (e.g., eslint-config-custom) MUST be referenced by package name, not by relative file paths.

### Verify

```bash
# Count files using @/core path aliases
grep -r "from ['\"]@/core" apps/demo apps/docs | wc -l

# Count relative path imports crossing package boundaries (excluding styles)
grep -r "from ['\"]\.\.\." apps/demo/config apps/docs/components | grep -v "styles.module.css" | wc -l

# Verify tsconfig.json files define @/core path aliases
find apps -name 'tsconfig.json' -exec grep -l '"@/core"' {} \;
```

**Accept when:**
- At least 80% of imports from core libraries use path aliases rather than relative paths
- All tsconfig.json files in the monorepo define consistent path alias mappings for @/core
- No relative path imports (../) are used to cross application or package boundaries

<enforcement>
Clause Code MUST NOT skip or defer verification. Violations block CI builds and pull requests until imports are corrected to use path aliases.
</enforcement>