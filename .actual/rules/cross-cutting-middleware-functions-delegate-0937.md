# Adopt Generic Tree-Walking Middleware Pattern for Component Data Transformation: Middleware Functions Delegate

These rules are ALWAYS ACTIVE for all component data transformation code that traverses hierarchical structures (ComponentData, RootData, UserData) within the codebase.

### Rules

- **R-MIDDLEWARE-001** MUST: Middleware functions MUST delegate field-level traversal to specialized utilities (e.g., mapFields) to maintain separation of concerns between tree navigation and field transformation.

### Verify

```bash
# Verify walkTree function exists with generic type parameters
grep -r 'function walkTree' packages/core/lib/data/ --include='*.ts'

# Verify walkTree is used with generic type parameters in codebase
grep -r 'walkTree<' packages/ --include='*.ts' | head -20

# Verify mapFields utility is integrated with walkTree
grep -r 'mapFields' packages/core/lib/data/walk-tree.ts
```

**Accept when:**
- The walkTree function is present in packages/core/lib/data/walk-tree.ts with generic type parameters extending ComponentData, RootData, or UserData
- The function accepts a callback with signature (data: Content, options: WalkTreeOptions) and delegates field traversal to mapFields utility
- Implementation handles all three structural variants: props-based ComponentData, root-level data, and UserData with zones
- New transformation operations use walkTree middleware rather than implementing custom traversal logic
- Generic type parameters are correctly specified at call sites and pass TypeScript compilation
- Unit tests validate that walkTree handles all data shape variants correctly

<enforcement>
Clause Code MUST NOT skip or defer verification of middleware delegation patterns. All component tree transformations MUST use the walkTree middleware with mapFields delegation unless explicitly documented exceptions are approved by core team maintainers.
</enforcement>