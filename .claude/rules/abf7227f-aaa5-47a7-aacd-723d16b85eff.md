<rule_activation id="abf7227f-aaa5-47a7-aacd-723d16b85eff" title="Adopt Catch-All Route Patterns for Dynamic Content Management Systems: Route Implementations Handle" applies_to="**/*">
These rules are ALWAYS ACTIVE for all public-facing API routes and dynamic content management integrations in Next.js, Remix, and React Router applications.
</rule_activation>

### Rules

- **R-CMS-001** MUST: Route implementations MUST handle both root path access (empty catch-all) and nested paths consistently.

### Verify

```bash
# Verify catch-all route patterns in Next.js, Remix, and React Router
grep -r "\[\.\.\..*\]" --include="page.tsx" --include="route.tsx" app/ || grep -r "params\.\$" --include="*.tsx" app/routes/ || grep -r "splat" --include="*.tsx" app/routes/

# Find catch-all route files by naming convention
find . -type f \( -name "*[...]*" -o -name "*\$*" -o -name "*splat*" \) -path "*/routes/*" -o -path "*/app/*"

# Verify CMS path parameter extraction
grep -r "puckPath\|cmsPath" --include="*.tsx" --include="*.ts" app/ | grep -E "params\.(puckPath|\$|\*)"
```

**Accept when:**
- All CMS integration routes use framework-native catch-all syntax and can be identified by grep patterns
- Route handlers extract and pass full paths to CMS resolution without modification or truncation
- No custom routing middleware is implemented for CMS path resolution in new code
- All catch-all routes include proper error handling for unresolved paths with appropriate HTTP status codes

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated code review checks, CI pipeline verification, architecture review, and security audits are mandatory before accepting catch-all route implementations.
</enforcement>