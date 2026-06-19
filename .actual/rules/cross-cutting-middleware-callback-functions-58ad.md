# Adopt Generic Tree-Walking Middleware Pattern for Component Data Transformation: Middleware Callback Functions

These rules are ALWAYS ACTIVE for all component data transformation operations that traverse hierarchical component structures with nested content, zones, and slots.

### Rules

- **R-MIDDLEWARE-001** MUST: Middleware callback functions MUST receive both the content node and contextual options (parentId, propName) to enable context-aware transformations.

### Verify

```bash
# Verify walkTree function exists with generic type parameters
grep -r 'function walkTree' packages/core/lib/data/ --include='*.ts'

# Verify walkTree is used with proper generic type parameters
grep -r 'walkTree<' packages/ --include='*.ts' | head -20

# Verify mapFields utility integration
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
Clause Code MUST NOT skip or defer verification of these rules. All component tree transformations MUST use the walkTree middleware pattern with proper callback signatures including contextual options. Code review verification is mandatory before merge.
</enforcement>