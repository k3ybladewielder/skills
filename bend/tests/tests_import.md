# Modules and Import Resolution Subskill (`bend/tests/import`)

## Overview
This subskill guides multi-file module organization, import aliasing, namespace resolution, and cross-file proof linking.

## Key Concepts
- **One Module per File**: Every `.bend` file defines a module whose symbols are accessed via its alias.
- **Import Syntax**:
  - Standard Prelude: `import Base`
  - Relative Path Import: `import ./MyModule.bend as MyModule`
  - Relative Directory Import: `import ../Types.bend as Types`
- **Diamond Deduplication**: When multiple files import the same module, definitions are unified cleanly without duplication.

## Example Pattern from `bend/tests/import`
```python
import Base
import ./Types.bend as Types
import ./LAWS.bend as LAWS

def compute_total(items: List<Types.Item>) -> U32:
  match items:
    case Nil{}: 0
    case Con{item, rest}:
      match item:
        case Types.ItemEntity{val}:
          U32.add(val, compute_total(rest))
```

## Agent Checklist
- Always use explicit aliased imports (`import ./File.bend as Alias`).
- Qualify all imported types and constructors using the module alias (`Types.Item`).
- Ensure `PROOF.bend` sits beside and explicitly imports `LAWS.bend`.
