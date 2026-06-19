# Standardize Core Library Imports via Aliased Module Paths: Path Aliases Configured

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript files within monorepo applications (apps/*), including imports of shared core libraries, utility modules, component files, and build tool configurations.

### Rules

- **R-PATH-001** MUST: Path aliases MUST be configured consistently across all build tools (TypeScript, bundlers, test runners) within each application.

### Verify

```bash
# Count files using @/core path aliases
grep -r "from ['\"]@/core" apps/demo apps/docs | wc -l

# Count relative path imports crossing boundaries (excluding styles)
grep -r "from ['\"]\.\.\." apps/demo/config apps/docs/components | grep -v "styles.module.css" | wc -l

# Verify tsconfig.json files define @/core path aliases
find apps -name 'tsconfig.json' -exec grep -l '"@/core"' {} \;
```

**Accept when:**
- At least 80% of imports from core libraries use path aliases rather than relative paths
- All tsconfig.json files in the monorepo define consistent path alias mappings for @/core
- No relative path imports (../) are used to cross application or package boundaries

<enforcement>
Claude Code MUST NOT skip or defer verification. Path alias consistency is mandatory across all build tools and must be verified before accepting changes.
</enforcement>