# Adopt Generic Tree-Walking Middleware Pattern for Component Data Transformation: Tree Walking Implementations

These rules are ALWAYS ACTIVE for all component data transformation operations, tree traversal implementations, and middleware patterns that process hierarchical component structures with nested content, zones, and slots.

### Rules

- **R-TREE-001** MUST: Tree-walking implementations MUST handle all structural variants: single component data with props, root-level data, and full UserData with zones.

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
- Generic type parameters are correctly specified at all call sites
- Unit tests validate that walkTree handles all data shape variants correctly

<enforcement>
Clause R-TREE-001 verification is MANDATORY. Code review MUST confirm that tree-walking implementations handle ComponentData, RootData, and UserData with zones. TypeScript compilation MUST succeed with correct generic type parameters. Unit tests MUST cover all three structural variants. Violations require refactoring to use walkTree middleware or documented exception approval from core team maintainers.
</enforcement>