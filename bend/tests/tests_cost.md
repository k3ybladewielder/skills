# Work Metrics and Rewrite Cost Subskill (`bend/tests/cost`)

## Overview
This subskill guides complexity analysis, interaction net rewrite accounting, and work estimation for parallel and recursive algorithms.

## Key Concepts
- **Rewrite Step Accounting**: Each interaction net beta-reduction or node annihilation counts as a discrete work unit.
- **Asymptotic Work vs Depth**: Work is the total number of reductions; Depth (span) is the longest sequential chain.
- **Tree-shaped vs List-shaped Reductions**: Binary tree recursions achieve $O(\log N)$ parallel depth, whereas linked lists remain $O(N)$.

## Example Pattern from `bend/tests/cost`
```python
import Base

# Binary tree fold: O(log N) depth on GPU / parallel runtime
type Tree is Data:
  Leaf{val: U32}
  Node{left: Tree, right: Tree}

def sum_tree(t: Tree) -> U32:
  match t:
    case Leaf{val}: val
    case Node{l, r}:
      left right = sum_tree(l) sum_tree(r)
      U32.add(left, right)
```

## Agent Checklist
- Structure parallel operations into balanced binary trees rather than linear lists.
- Avoid unnecessary cloning of heavy `Data` terms that inflate work counters.
