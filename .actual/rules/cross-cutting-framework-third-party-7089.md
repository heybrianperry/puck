# Standardize Core Library Imports via Aliased Module Paths: Framework Third Party

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript files within monorepo applications (apps/*), including component files, configuration files, and build tool configurations that reference shared packages and core libraries.

### Rules

- **R-CORE-001** SHOULD: Framework and third-party library imports SHOULD use bare module specifiers without path aliases.

### Verify

```bash
# Count files using @/core path aliases
grep -r "from ['\"]@/core" apps/demo apps/docs | wc -l

# Count relative path imports crossing boundaries
grep -r "from ['\"]\.\.\." apps/demo/config apps/docs/components | grep -v "styles.module.css" | wc -l

# Verify tsconfig.json files define consistent path alias mappings
find apps -name 'tsconfig.json' -exec grep -l '"@/core"' {} \;
```

**Accept when:**
- At least 80% of imports from core libraries use path aliases rather than relative paths
- All tsconfig.json files in the monorepo define consistent path alias mappings for @/core
- No relative path imports (../) are used to cross application or package boundaries

<enforcement>
Claude Code MUST NOT skip or defer verification. ESLint rules checking for relative imports crossing package boundaries, code review checklists, and CI pipeline checks using grep or custom scripts MUST be applied to detect violations. Pull requests with violations are blocked until imports are corrected to use path aliases.
</enforcement>