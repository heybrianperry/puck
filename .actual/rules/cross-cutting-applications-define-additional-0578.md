# Standardize Core Library Imports via Aliased Module Paths: Applications Define Additional

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript files within monorepo applications (apps/*), including imports of shared core libraries, utility modules, configuration files that reference shared packages, and component files importing from centralized component libraries.

### Rules

- **R-ALIAS-001** MAY: Applications MAY define additional application-specific path aliases beyond the core library aliases.

### Verify

```bash
# Count files using @/core path aliases
grep -r "from ['\"]@/core" apps/demo apps/docs | wc -l

# Count relative path imports crossing boundaries (excluding styles)
grep -r "from ['\"]\.\.\." apps/demo/config apps/docs/components | grep -v "styles.module.css" | wc -l

# Find all tsconfig.json files defining @/core path aliases
find apps -name 'tsconfig.json' -exec grep -l '"@/core"' {} \;
```

**Accept when:**
- At least 80% of imports from core libraries use path aliases rather than relative paths
- All tsconfig.json files in the monorepo define consistent path alias mappings for @/core
- No relative path imports (../) are used to cross application or package boundaries

<enforcement>
Claude Code MUST NOT skip or defer verification. ESLint rules checking for relative imports crossing package boundaries, code review checklist items, and CI pipeline checks using grep or custom scripts MUST be applied to detect violations. Pull requests with violations are blocked until imports are corrected to use path aliases.
</enforcement>