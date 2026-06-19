# Adopt Generic Tree-Walking Middleware Pattern for Component Data Transformation: Callback Functions Return

These rules are ALWAYS ACTIVE for all component data transformation operations that traverse hierarchical component structures with nested content, zones, and slots.

### Rules

- **R-TREE-001** SHOULD: Callback functions SHOULD return the transformed content or null/void to indicate no change, allowing the middleware to preserve original values when appropriate.

### Verify

```bash
# Verify walkTree function exists with generic type parameters
grep -r 'function walkTree' packages/core/lib/data/ --include='*.ts'

# Verify walkTree is used with generic type parameters in codebase
grep -r 'walkTree<' packages/ --include='*.ts' | head -20

# Verify mapFields utility integration
grep -r 'mapFields' packages/core/lib/data/walk-tree.ts
```

**Accept when:**
- The walkTree function is present in packages/core/lib/data/walk-tree.ts with generic type parameters extending ComponentData, RootData, or UserData
- The function accepts a callback with signature (data: Content, options: WalkTreeOptions) and delegates field traversal to mapFields utility
- Implementation handles all three structural variants: props-based ComponentData, root-level data, and UserData with zones
- Callback functions return transformed content or null/void rather than mutating input directly
- New transformation operations use walkTree middleware rather than implementing custom traversal logic

<enforcement>
Clause Code MUST NOT skip or defer verification of walkTree middleware adoption. All component tree transformations MUST use the standardized middleware pattern with callbacks returning transformed content or null/void.
</enforcement>