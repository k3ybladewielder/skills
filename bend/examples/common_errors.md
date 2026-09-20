# Common Errors and Anti-Patterns in Bend

Este guia compila, resume e generaliza os erros mais comuns cometidos durante o desenvolvimento em **Bend** (compilação, verificação afim, checagem de tipos e proofs), fornecendo os anti-padrões e as soluções idiomáticas correspondentes.

---

## 1. Declaração de Tipos e Construtores

### Erro 1.1: Omissão da Kind na Declaração de Tipo
- **Sintoma / Erro**: `expected : 'is', observed : ':'`
- **Causa**: Declaração de tipo algébrico sem a especificação da kind `is Data`.
- **Anti-Padrão**:
  ```python
  type Expr:
    Val{val: F32}
  ```
- **Solução Idiomática**:
  ```python
  type Expr is Data:
    Val{val}
  ```

### Erro 1.2: Espaçamento ou Omissão de `{}` em Construtores
- **Sintoma / Erro**: `expected : '{', observed : 'A'`
- **Causa**: Espaço entre o nome do construtor e `{}` ou omissão de `{}` em construtores sem campos.
- **Anti-Padrão**:
  ```python
  type Expr is Data:
    Var              # Faltam as chaves {}
    Val { val }      # Espaço inválido antes de {}
  ```
- **Solução Idiomática**:
  ```python
  type Expr is Data:
    Var{}
    Val{val}
  ```

### Erro 1.3: Campos Nomeados em Instanciações e Patterns
- **Sintoma / Erro**: `expected : a term, observed : ':'`
- **Causa**: Tentar usar sintaxe `campo: valor` ao instanciar ou desestruturar construtores. Em Bend, os nomes dos campos são declarados apenas no `type`. Na construção e no `match`, os argumentos dentro de `{}` são puramente posicionais.
- **Anti-Padrão**:
  ```python
  let node = Types.Val{val: 42.0}
  match expr:
    case Types.Add{left: l, right: r}: ...
  ```
- **Solução Idiomática**:
  ```python
  let node = Types.Val{42.0}
  match expr:
    case Types.Add{l, r}: ...
  ```

---

## 2. Sistema de Tipos e Assinaturas

### Erro 2.1: Tuplas vs. Tipo Par (`&`) em Assinaturas
- **Sintoma / Erro**: `expected : Type, observed : Sigma`
- **Causa**: Uso da sintaxe de tupla `(A, B)` em assinaturas de tipo. Em Bend, `(a, b)` representa uma expressão de valor, enquanto o tipo de produto cartesiano é denotado por `&`.
- **Anti-Padrão**:
  ```python
  def process_pairs(pts: List<(F32, F32)>) -> F32:
  ```
- **Solução Idiomática**:
  ```python
  def process_pairs(pts: List<(F32 & F32)>) -> F32:
  ```

### Erro 2.2: Ausência de Parênteses em Argumentos de Tipos Compostos Genéricos
- **Sintoma / Erro**: `expected : a name, observed : '>'`
- **Causa**: O caractere `&` dentro dos delimitadores genéricos `< ... >` é interpretado pelo parser como anotação de quantidade/binder se não estiver envolvido por parênteses.
- **Anti-Padrão**:
  ```python
  def items() -> List<F32 & F32>:
  ```
- **Solução Idiomática**:
  ```python
  def items() -> List<(F32 & F32)>:
  ```

### Erro 2.3: Sufixos Numéricos Inválidos (ex: `u`)
- **Sintoma / Erro**: `expected : a numeric literal (NUMBER is U32, NUMBER n is Nat), observed : 'u'`
- **Causa**: Uso de sufixos numéricos estilo C/Rust (`10u`). Em Bend, literais inteiros sem sufixo (`10`) já pertencem ao tipo `U32` por padrão, enquanto o sufixo `n` denota `Nat` (`10n`). O sufixo `u` é inválido.
- **Anti-Padrão**: `let x = 10u`
- **Solução Idiomática**: `let x = 10`

---

## 3. Linearidade e Quantidades (Affine Variables)

### Erro 3.1: Reuso de Variável Sem Marcação Afim (`+`)
- **Sintoma / Erro**: `expected : x, observed : x (consumed more than once)`
- **Causa**: Variáveis em Bend são afins por padrão (podem ser lidas no máximo uma vez). Usar a mesma variável mais de uma vez exige o prefixo `+`.
- **Anti-Padrão**:
  ```python
  def safe_div(num: F32, den: F32) -> F32:
    if F32.is_eq(den, 0.0):
      0.0
    else:
      F32.div(num, den) # 'den' consumido 2 vezes
  ```
- **Solução Idiomática**:
  ```python
  def safe_div(num: F32, +den: F32) -> F32:
    if F32.is_eq(den, 0.0):
      0.0
    else:
      F32.div(num, den)
  ```

---

## 4. Scrutinees e Restrições da Instrução `match`

### Erro 4.1: `match` em Binders Locais ou Expressões
- **Sintoma / Erro**: `a parameter or field scrutinee (a match cannot scrutinize a local binder: give it its own def)`
- **Causa**: A instrução `match` em Bend inspeciona **exclusivamente** parâmetros formais da função atual ou campos obtidos por desestruturação imediata. Não é permitido dar `match` em uma variável local (`let x = ...`) ou em uma expressão inline.
- **Anti-Padrão**:
  ```python
  def eval(a: F32, b: F32) -> F32:
    let is_zero = F32.is_eq(b, 0.0)
    match is_zero: # ERRO: binder local
      case True: 0.0
      case False: a / b
  ```
- **Solução Idiomática**: Crie uma função auxiliar que receba a condição ou o dado a ser inspecionado como parâmetro.
  ```python
  def eval(a: F32, b: F32) -> F32:
    eval_help(F32.is_eq(b, 0.0), a, b)

  def eval_help(is_zero: U32, a: F32, b: F32) -> F32:
    match is_zero:
      case 1: 0.0
      case _: F32.div(a, b)
  ```

### Erro 4.2: Matches Aninhados Inspecionando Binders Externos
- **Sintoma / Erro**: `a match on a parameter or field (this name is a def or a consumed binder: give the value its own def)`
- **Causa**: Um `match` aninhado dentro da ramificação de outro `match` tenta inspecionar um parâmetro secundário da função sem que este seja um parâmetro limpo da ramificação.
- **Solução Idiomática**: Extraia o `match` aninhado para uma função auxiliar de passo (`_step`).

### Erro 4.3: Atribuição `let` Antes do `match` de Parâmetro
- **Sintoma / Erro**: `a match on a parameter or field (this name is a def or a consumed binder...)` ao declarar `let` antes do `match`
- **Causa**: Uma instrução `let` colocada no início da função antes do `match` que avalia o parâmetro principal invalida o status do parâmetro para o scrutinee.
- **Anti-Padrão**:
  ```python
  def process(gen: U32, pop: List<Individual>) -> List<Individual>:
    let evaluated = evaluate(pop) # ERRO: let antes de match gen
    match gen:
      case 0: pop
      case _: process(gen - 1, evaluated)
  ```
- **Solução Idiomática**: Posicione o `match` no topo e mova o `let` para dentro do case apropriado.
  ```python
  def process(gen: U32, pop: List<Individual>) -> List<Individual>:
    match gen:
      case 0: pop
      case _:
        let evaluated = evaluate(pop)
        process(gen - 1, evaluated)
  ```

---

## 5. Operadores, Comparações e Funções da Biblioteca `Base`

### Erro 5.1: Uso de `==` em Expressões de Valor
- **Sintoma / Erro**: `expected : ')', observed : '='`
- **Causa**: Em Bend, `==` é reservado para igualdade de tipos/formal proofs. Para igualdade de valores numéricos em runtime, deve-se usar as funções da `Base` (`F32.is_eq`, `U32.is_eq`).
- **Anti-Padrão**: `if (den == 0.0):`
- **Solução Idiomática**: `if F32.is_eq(den, 0.0):`

### Erro 5.2: Inferencia Padrão de Operadores Infixo para `Nat`
- **Sintoma / Erro**: `expected : Nat, observed : F32`
- **Causa**: Operadores infixo (`/`, `*`, `%`, `+`, `-`) sem anotação explícita pertencem por padrão a `Nat`.
- **Anti-Padrão**: `let res = num / den` (onde num e den são `F32`)
- **Solução Idiomática**:
  Use anotações explícitas `(num / den : F32)` ou prefira as funções do módulo do tipo (`F32.div(num, den)`, `U32.mod(a, b)`).

### Erro 5.3: Conflito de Anotações Infixo em Inicializadores de Campos
- **Sintoma / Erro**: `expected : a term, observed : ':'` em construções de registros/construtores
- **Causa**: O parser se confunde ao parsear anotações de tipo infixo `(a % b : U32)` dentro de construtores de tipos.
- **Solução Idiomática**: Use chamadas explícitas da biblioteca `Base` (`U32.mod(a, b)`, `F32.add(a, b)`).

---

## 6. Leis e Provas (`LAWS.bend` e `PROOF.bend`)

### Erro 6.1: Sintaxe de Parâmetros em Leis (`law`)
- **Sintoma / Erro**: `expected : a term, observed : ','`
- **Causa**: Tentar declarar múltiplos parâmetros na mesma linha `for` separados por vírgulas.
- **Anti-Padrão**:
  ```python
  law safe_tree(expr: Types.Expr, x: F32):
    for expr: Types.Expr, x: F32:
      ...
  ```
- **Solução Idiomática**: Cada variável quantificada deve ter seu próprio bloco `for` sem vírgulas ou dois-pontos no final da linha.
  ```python
  law safe_tree(expr: Types.Expr, x: F32):
    for expr: Types.Expr
    for x: F32:
      ...
  ```

### Erro 6.2: Anotações de Tipo em Parâmetros de Provas (`def`)
- **Sintoma / Erro**: `expected : a name, observed : ':'`
- **Causa**: Tentar anotar os tipos dos parâmetros na função `def` que provê a implementação da prova em `PROOF.bend`.
- **Anti-Padrão**: `def LAWS.safe_tree(expr: Types.Expr, x: F32):`
- **Solução Idiomática**: Funções `def` de prova aceitam apenas nomes de parâmetros simples.
  ```python
  def LAWS.safe_tree(expr, x):
    ...
  ```
