# Common Errors and Anti-Patterns in Bend

This guide compiles, summarizes, and generalizes the most common errors encountered during **Bend** development (compilation, affine verification, type checking, and formal proofs), providing corresponding anti-patterns and idiomatic solutions.

---

## 1. Type and Constructor Declarations

### Error 1.1: Omission of Kind in Type Declaration
- **Symptom / Error**: `expected : 'is', observed : ':'`
- **Cause**: Declaring an algebraic data type without specifying its kind (`is Data` or `is Type`).
- **Anti-Pattern**:
  ```python
  type Expr:
    Val{val: F32}
  ```
- **Idiomatic Solution**:
  ```python
  type Expr is Data:
    Val{val: F32}
  ```

### Error 1.2: Spacing or Omission of `{}` in Constructors
- **Symptom / Error**: `expected : '{', observed : 'A'`
- **Cause**: Space between constructor name and `{}` or omitting `{}` on empty constructors.
- **Anti-Pattern**:
  ```python
  type Expr is Data:
    Var              # Missing brackets {}
    Val { val }      # Invalid space before {}
  ```
- **Idiomatic Solution**:
  ```python
  type Expr is Data:
    Var{}
    Val{val: F32}
  ```

### Error 1.3: Named Fields in Instantiations and Match Patterns
- **Symptom / Error**: `expected : a term, observed : ':'`
- **Cause**: Attempting to use `field: value` syntax when instantiating or pattern matching constructors. In Bend, field names are declared only in `type`. Constructor instantiations and pattern match extractions within `{}` are purely positional.
- **Anti-Pattern**:
  ```python
  node = Types.Val{val: 42.0}
  match expr:
    case Types.Add{left: l, right: r}: ...
  ```
- **Idiomatic Solution**:
  ```python
  node = Types.Val{42.0}
  match expr:
    case Types.Add{l, r}: ...
  ```

### Error 1.4: Duplicate Constructor Names in Module (Constructor Collision)
- **Symptom / Error**: `expected : a fresh constructor name (duplicate declaration: Module.ConstructorName), observed : '{'`
- **Cause**: Reusing the same constructor identifier (such as `Item`, `Val`, `Nil`) across distinct algebraic data types within the same module/file. In Bend, all constructors declared in a module share a unified namespace and must have unique names.
- **Anti-Pattern**:
  ```python
  type Client is Data:
    Item{id: U32, phone: U32}

  type Procedure is Data:
    Item{id: U32, price: F32} # ERROR: Duplicate 'Item' constructor
  ```
- **Idiomatic Solution**:
  Use unique constructor names or prefix them with the entity name:
  ```python
  type Client is Data:
    ClientItem{id: U32, phone: U32}

  type Procedure is Data:
    ProcedureItem{id: U32, price: F32}
  ```

### Error 1.5: Generic Type Parameters without Kind Annotation (`-T: Data`) vs Built-in `List`
- **Symptom / Error**: `expected : Data, observed : Quant (Context: - T : Quant)`
- **Cause**: When declaring generic algebraic data types such as `type MyList<T> is Data:`, identifiers inside `<...>` are parsed by default as quantity annotations (`Quant`), rather than kinds `Data`/`Type`. Moreover, the `Base` library already provides a built-in `List<T>` type with constructors `Con{head, tail}` and `Nil{}`.
- **Anti-Pattern**:
  ```python
  type List<T> is Data:
    Cons{head: T, tail: List<T>}
    Nil{}
  ```
- **Idiomatic Solution**:
  Use the built-in `List<T>` type and `Con{head, tail}` / `Nil{}` constructors from `Base`:
  ```python
  import Base
  # Type: List<Client>, List<Procedure>
  # Constructors: Con{h, t} and Nil{}
  ```
  If declaring a custom generic type, annotate the kind explicitly with `-T: Data`:
  ```python
  type Container<-T: Data> is Data:
    Box{content: T}
  ```

---

## 2. Type System and Signatures

### Error 2.1: Tuples vs. Pair Types (`&`) in Signatures
- **Symptom / Error**: `expected : Type, observed : Sigma`
- **Cause**: Using tuple syntax `(A, B)` in type signatures. In Bend, `(a, b)` is a value-level expression, while the Cartesian product type is denoted by `&`.
- **Anti-Pattern**:
  ```python
  def process_pairs(pts: List<(F32, F32)>) -> F32:
  ```
- **Idiomatic Solution**:
  ```python
  def process_pairs(pts: List<(F32 & F32)>) -> F32:
  ```

### Error 2.2: Missing Parentheses around Generic Compound Arguments
- **Symptom / Error**: `expected : a name, observed : '>'`
- **Cause**: The `&` symbol inside generic delimiters `< ... >` is parsed as a quantity annotation or binder unless wrapped in parentheses.
- **Anti-Pattern**:
  ```python
  def items() -> List<F32 & F32>:
  ```
- **Idiomatic Solution**:
  ```python
  def items() -> List<(F32 & F32)>:
  ```

### Error 2.3: Invalid Numeric Suffixes (e.g., `u`)
- **Symptom / Error**: `expected : a numeric literal (NUMBER is U32, NUMBER n is Nat), observed : 'u'`
- **Cause**: Using C/Rust-style suffixes (`10u`). In Bend, unsuffixed integer literals (`10`) are of type `U32` by default, while suffix `n` denotes `Nat` (`10n`). Suffix `u` is invalid.
- **Anti-Pattern**: `x = 10u`
- **Idiomatic Solution**: `x = 10`

---

## 3. Linearity, Quantities, and Variable Binding

### Error 3.1: Reusing Variables without Affine Marker (`+`)
- **Symptom / Error**: `expected : x, observed : x (consumed more than once)`
- **Cause**: Variables in Bend are affine by default (can be read at most once). Using the same variable more than once requires the `+` prefix.
- **Anti-Pattern**:
  ```python
  def safe_div(num: F32, den: F32) -> F32:
    if F32.is_eq(den, 0.0):
      0.0
    else:
      F32.div(num, den) # 'den' consumed twice
  ```
- **Idiomatic Solution**:
  ```python
  def safe_div(num: F32, +den: F32) -> F32:
    if F32.is_eq(den, 0.0):
      0.0
    else:
      F32.div(num, den)
  ```

### Error 3.2: Using the `let` Keyword in Statements
- **Symptom / Error**: `expected : a term (the keyword 'def' cannot head one), observed : ' '`
- **Cause**: In Bend 2, local assignments and variable declarations are written directly as `name = value` (or `+name = value`, `-name = value`), without the keyword `let`. Using `let` is treated by the parser as an open expression, preventing the function block from closing before the next `def`.
- **Anti-Pattern**:
  ```python
  def process(x: U32) -> U32:
    let temp = U32.add(x, 1)
    U32.mul(temp, 2)
  ```
- **Idiomatic Solution**:
  ```python
  def process(x: U32) -> U32:
    temp = U32.add(x, 1)
    U32.mul(temp, 2)
  ```

---

## 4. Scrutinees and `match` Restrictions

### Error 4.1: `match` on Expressions, Computed Values, or Local Binders
- **Symptom / Error**: `a parameter or field scrutinee (a match cannot scrutinize a computed value: give it its own def)` or `(a match cannot scrutinize a local binder: give it its own def)`
- **Cause**: The `match` statement in Bend inspects **exclusively** formal parameters of the current function or fields obtained by immediate constructor destructuring. Matching on inline function calls (`match F32.is_eq(...)`), computed values, or local variables is disallowed.
- **Anti-Pattern**:
  ```python
  def eval(a: F32, b: F32) -> F32:
    match F32.is_eq(b, 0.0): # ERROR: scrutinee is a computed value
      case 1: 0.0
      case _: F32.div(a, b)
  ```
- **Idiomatic Solution**:
  Pass the computed result to a helper function (`_step`) where it becomes a formal parameter:
  ```python
  def eval(a: F32, b: F32) -> F32:
    eval_step(F32.is_eq(b, 0.0), a, b)

  def eval_step(is_zero: U32, a: F32, b: F32) -> F32:
    match is_zero:
      case 1: 0.0
      case _: F32.div(a, b)
  ```

### Error 4.2: Nested Matches Inspecting Outer Binders
- **Symptom / Error**: `a match on a parameter or field (this name is a def or a consumed binder: give the value its own def)`
- **Cause**: A nested `match` inside another `match` branch attempts to inspect a secondary parameter without it being a clean branch parameter.
- **Idiomatic Solution**: Extract the nested `match` into an auxiliary step function (`_step`).

### Error 4.3: Assignment before Parameter `match`
- **Symptom / Error**: `a match on a parameter or field (this name is a def or a consumed binder...)` when placing assignment before `match`
- **Cause**: Placing an assignment statement at the start of a function before the `match` that inspects the main parameter invalidates the scrutinee status.
- **Anti-Pattern**:
  ```python
  def process(gen: U32, pop: List<Individual>) -> List<Individual>:
    evaluated = evaluate(pop) # ERROR: assignment before match gen
    match gen:
      case 0: pop
      case _: process(U32.sub(gen, 1), evaluated)
  ```
- **Idiomatic Solution**: Place the `match` at the top level and move the assignment inside the appropriate branch:
  ```python
  def process(gen: U32, pop: List<Individual>) -> List<Individual>:
    match gen:
      case 0: pop
      case _:
        evaluated = evaluate(pop)
        process(U32.sub(gen, 1), evaluated)
  ```

---

## 5. Operators, Comparisons, and `Base` Functions

### Error 5.1: Using `==` in Value Expressions
- **Symptom / Error**: `expected : ')', observed : '='`
- **Cause**: In Bend, `==` is reserved for propositional equality types in formal proofs. For runtime value equality, use functions from `Base` (`F32.is_eq`, `U32.is_eq`).
- **Anti-Pattern**: `if (den == 0.0):`
- **Idiomatic Solution**: `if F32.is_eq(den, 0.0):`

### Error 5.2: Bare/Unannotated Infix Operators
- **Symptom / Error**: `message : a type for this operator (write (a + b : Nat))`
- **Cause**: In Bend 2 (version 2.0.16+), bare infix operators (`+`, `-`, `*`, `/`, `%`) require explicit type annotation and do not implicitly default to `Nat`.
- **Anti-Pattern**:
  ```python
  def add(a: U32, b: U32) -> U32:
    a + b
  ```
- **Idiomatic Solution**:
  Use explicit module functions from the `Base` library (recommended) or wrap in a type annotation:
  ```python
  def add(a: U32, b: U32) -> U32:
    U32.add(a, b)
    # or alternatively: ((a + b) : U32)
  ```

### Error 5.3: Infix Type Annotation Conflict in Constructor Fields
- **Symptom / Error**: `expected : a term, observed : ':'` in record/constructor instantiations
- **Cause**: The parser encounters ambiguity when parsing infix annotations like `(a % b : U32)` inside constructor fields.
- **Idiomatic Solution**: Use explicit calls to `Base` functions (`U32.mod(a, b)`, `F32.add(a, b)`).

---

## 6. Laws and Proofs (`LAWS.bend` and `PROOF.bend`)

### Error 6.1: Parameter Syntax in Laws (`law`)
- **Symptom / Error**: `expected : a term, observed : ','`
- **Cause**: Attempting to declare multiple parameters in the same `for` line separated by commas.
- **Anti-Pattern**:
  ```python
  law safe_tree(expr: Types.Expr, x: F32):
    for expr: Types.Expr, x: F32:
      ...
  ```
- **Idiomatic Solution**: Each quantified variable must have its own `for` line without commas or trailing colons:
  ```python
  law safe_tree:
    for +expr: Types.Expr
    for +x: F32
    ...
  ```

### Error 6.2: Type Annotations in Proof Parameters (`def`)
- **Symptom / Error**: `expected : a name, observed : ':'`
- **Cause**: Attempting to annotate parameter types in the `def` function providing the proof implementation.
- **Anti-Pattern**: `def LAWS.safe_tree(expr: Types.Expr, x: F32):`
- **Idiomatic Solution**: Proof `def` functions accept plain parameter names without type annotations:
  ```python
  def LAWS.safe_tree(expr, x):
    {==}
  ```
