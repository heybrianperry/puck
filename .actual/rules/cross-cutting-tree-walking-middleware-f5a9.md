# Adopt Generic Tree-Walking Middleware Pattern for Component Data Transformation: Tree Walking Middleware

These rules are ALWAYS ACTIVE for all component data transformation operations, tree traversal logic, and middleware implementations that work with hierarchical component structures (ComponentData, RootData, UserData).

### Rules

- **R-TREE-001** MUST: Tree-walking middleware functions MUST accept generic type parameters that extend ComponentData, RootData, or UserData to preserve type information through transformations.

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
- New transformation operations use walkTree middleware rather than implementing custom traversal logic
- Generic type parameters are correctly specified at call sites and pass TypeScript compilation
- Unit tests validate that walkTree handles all data shape variants correctly

<enforcement>
Clause Code MUST NOT skip or defer verification of R-TREE-001. All component tree transformation operations must be reviewed to ensure they use the walkTree middleware pattern with appropriate generic type parameters. Violations require code review feedback requesting refactoring to use walkTree middleware. Exceptions require documentation of performance requirements or structural constraints and approval from core team maintainers.
</enforcement>