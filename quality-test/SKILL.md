---
name: quality-testing
description: Guia completo para criação de testes voltados à mensuração objetiva da qualidade de código. Cobre cobertura, mutação, regressão, E2E, dependências, acoplamento, abstração, instabilidade, distância da sequência principal, connascence, tamanho de módulos, complexidade ciclomática, e métricas complementares. Use quando o usuário precisar criar, configurar ou entender testes de qualidade, métricas de código, ou health checks arquiteturais.
keywords: ["testes", "qualidade", "coverage", "mutação", "mutation", "regressão", "e2e", "end-to-end", "acoplamento", "coupling", "abstração", "instabilidade", "connascence", "complexidade", "ciclomática", "cyclomatic", "métricas", "dependências", "módulos", "linhas", "funções", "arquitetura", "health-check", "code-quality"]
---

# Quality Testing — Mensuração Objetiva da Qualidade de Código

## Filosofia

Esta skill orienta a criação de testes e métricas que produzem **números objetivos** sobre a saúde de um codebase. O objetivo não é atingir 100% em tudo, mas criar visibilidade, estabelecer baselines, detectar degradação, e guiar decisões de refatoração com dados.

Cada métrica deve ser:
- **Automatizável** — coletada sem intervenção humana
- **Rastreável** — comparável ao longo do tempo (trending)
- **Acionável** — quando piora, a equipe sabe o que fazer
- **Contextual** — thresholds variam por tipo de projeto e camada

---

## 1. Cobertura de Código (Coverage)

### O que mede
Percentual do código-fonte exercitado pelos testes automatizados. Existem múltiplos tipos:

| Tipo | O que cobre | Quando usar |
|------|-------------|-------------|
| Line Coverage | Linhas executadas | Baseline mínima |
| Branch Coverage | Cada ramificação (if/else) | Lógica condicional |
| Function Coverage | Funções invocadas | APIs e interfaces |
| Statement Coverage | Instruções executadas | Padrão geral |
| Condition Coverage | Cada sub-expressão booleana | Condições complexas |
| Path Coverage | Cada caminho possível | Fluxos críticos (caro) |
| MC/DC (Modified Condition/Decision) | Cada condição afeta o resultado | Sistemas safety-critical |

### Thresholds recomendados

| Camada | Mínimo | Alvo |
|--------|--------|------|
| Domínio/Core | 90% branch | 95%+ |
| Services/Use Cases | 80% branch | 90% |
| Infrastructure/Adapters | 60% line | 75% |
| Controllers/Handlers | 70% line | 80% |
| Utilities/Helpers | 85% branch | 95% |

### Configuração

**Python (pytest-cov)**
```toml
# pyproject.toml
[tool.pytest.ini_options]
addopts = "--cov=src --cov-report=term-missing --cov-report=html --cov-branch"

[tool.coverage.run]
branch = true
source = ["src"]
omit = ["*/tests/*", "*/__pycache__/*", "*/migrations/*"]

[tool.coverage.report]
fail_under = 80
show_missing = true
exclude_lines = [
    "pragma: no cover",
    "if TYPE_CHECKING:",
    "raise NotImplementedError",
    "@overload",
]
```

**Java (JaCoCo)**
```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.12</version>
    <executions>
        <execution>
            <goals><goal>prepare-agent</goal></goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals><goal>report</goal></goals>
        </execution>
        <execution>
            <id>check</id>
            <phase>verify</phase>
            <goals><goal>check</goal></goals>
            <configuration>
                <rules>
                    <rule>
                        <element>BUNDLE</element>
                        <limits>
                            <limit>
                                <counter>BRANCH</counter>
                                <value>COVEREDRATIO</value>
                                <minimum>0.80</minimum>
                            </limit>
                        </limits>
                    </rule>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>
```

**TypeScript/JavaScript (c8/istanbul via vitest)**
```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html', 'lcov'],
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
      exclude: ['**/tests/**', '**/mocks/**', '**/*.d.ts'],
    },
  },
});
```

### Anti-patterns de cobertura
- **Coverage gaming**: testes sem assertions que apenas executam código
- **Cobrir getters/setters triviais**: infla números sem valor
- **Ignorar branch coverage**: 100% line com 40% branch = falsa segurança
- **Não rastrear delta**: importa mais a tendência que o número absoluto

---

## 2. Teste de Mutação (Mutation Testing)

### O que mede
A **efetividade** dos testes — não apenas se executam o código, mas se realmente **detectam defeitos**. Introduz pequenas mudanças (mutantes) no código e verifica se os testes falham. Se um mutante sobrevive, os testes têm um ponto cego.

### Operadores de mutação comuns

| Categoria | Mutação | Exemplo |
|-----------|---------|---------|
| Relacional | `>` → `>=`, `<` → `<=` | `if x > 0` → `if x >= 0` |
| Aritmético | `+` → `-`, `*` → `/` | `a + b` → `a - b` |
| Lógico | `and` → `or`, `not` removido | `a and b` → `a or b` |
| Boundary | off-by-one em loops/ranges | `range(n)` → `range(n-1)` |
| Negação | condição invertida | `if valid` → `if not valid` |
| Retorno | valor de retorno alterado | `return x` → `return None` |
| Remoção | statement deletado | `lista.append(x)` → (removido) |
| Constante | literal alterado | `timeout=30` → `timeout=0` |

### Métricas

- **Mutation Score** = mutantes mortos / (total - equivalentes) × 100
- **Alvo**: ≥ 70% para código crítico, ≥ 50% para código geral
- **Equivalentes**: mutantes que não alteram o comportamento observável (falsos positivos)

### Ferramentas e configuração

**Python (mutmut)**
```bash
# Executar mutation testing
mutmut run --paths-to-mutate=src/ --tests-dir=tests/

# Ver resultados
mutmut results
mutmut show <id>  # ver mutante específico

# Gerar relatório HTML
mutmut html
```

```toml
# pyproject.toml (setup.cfg ou mutmut_config)
[tool.mutmut]
paths_to_mutate = "src/"
tests_dir = "tests/"
runner = "python -m pytest -x --tb=short"
```

**Java (PITest)**
```xml
<plugin>
    <groupId>org.pitest</groupId>
    <artifactId>pitest-maven-plugin</artifactId>
    <version>1.16.1</version>
    <configuration>
        <targetClasses>
            <param>com.company.core.*</param>
        </targetClasses>
        <targetTests>
            <param>com.company.core.*Test</param>
        </targetTests>
        <mutationThreshold>70</mutationThreshold>
        <coverageThreshold>80</coverageThreshold>
        <mutators>
            <mutator>DEFAULTS</mutator>
            <mutator>STRONGER</mutator>
        </mutators>
    </configuration>
</plugin>
```

**JavaScript/TypeScript (Stryker)**
```json
// stryker.conf.json
{
  "mutator": { "excludedMutations": ["StringLiteral"] },
  "testRunner": "vitest",
  "reporters": ["html", "clear-text", "progress"],
  "coverageAnalysis": "perTest",
  "thresholds": { "high": 80, "low": 60, "break": 50 }
}
```

### Estratégia de execução
- Mutation testing é **caro** (O(mutantes × tempo_dos_testes))
- Execute apenas em módulos críticos (domínio, core, regras de negócio)
- Use em CI como gate para PRs que tocam código crítico, com escopo incremental
- Execute completo semanalmente, incremental em PRs

---

## 3. Teste de Regressão

### O que mede
Se funcionalidades existentes continuam funcionando após mudanças. Regressão não é um tipo de teste separado — é um **propósito** aplicado a qualquer nível.

### Estratégias

| Estratégia | Quando usar | Trade-off |
|-----------|-------------|-----------|
| Re-run all | Build time < 10 min | Seguro mas lento em projetos grandes |
| Test Impact Analysis | Builds longos | Roda apenas testes afetados pela mudança |
| Snapshot Testing | UIs, APIs, schemas | Detecta mudanças inesperadas |
| Contract Testing | Microserviços | Valida compatibilidade entre serviços |
| Golden File Testing | Output determinístico | Compara com saída previamente aprovada |
| Visual Regression | Frontend | Compara screenshots pixel a pixel |

### Snapshot Testing (APIs)

```python
# tests/snapshots/test_user_api.py
import json
from pathlib import Path

SNAPSHOT_DIR = Path(__file__).parent / "snapshots"

def test_user_response_schema(client, snapshot):
    """Garante que a estrutura de resposta não muda sem intenção."""
    response = client.get("/api/v1/users/1")
    assert response.status_code == 200
    
    # Comparar com snapshot salvo
    snapshot_file = SNAPSHOT_DIR / "user_response.json"
    if snapshot_file.exists():
        expected = json.loads(snapshot_file.read_text())
        assert response.json().keys() == expected.keys(), (
            "Schema da resposta mudou! Se intencional, atualize o snapshot."
        )
    else:
        # Primeiro run — salva o snapshot
        snapshot_file.write_text(json.dumps(response.json(), indent=2))
```

### Contract Testing (Pact)

```python
# tests/contract/test_user_service_contract.py
from pact import Consumer, Provider

pact = Consumer("Frontend").has_pact_with(Provider("UserAPI"))

def test_get_user_contract():
    expected = {"id": 1, "name": "John", "email": "john@example.com"}
    
    (pact
        .given("user 1 exists")
        .upon_receiving("a request for user 1")
        .with_request("GET", "/api/v1/users/1")
        .will_respond_with(200, body=expected))
    
    with pact:
        result = user_client.get_user(1)
        assert result["name"] == "John"
```

### Configuração CI para regressão

```yaml
# .github/workflows/regression.yml
regression-tests:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - name: Run regression suite
      run: |
        pytest tests/ -m "not slow" --tb=short -q
    - name: Run slow regression (nightly only)
      if: github.event.schedule
      run: |
        pytest tests/ -m "slow" --tb=short
```

---

## 4. Testes End-to-End (E2E)

### O que mede
Se o sistema como um todo funciona corretamente do ponto de vista do usuário. Testa fluxos completos atravessando todas as camadas (UI → API → Database → External Services).

### Pirâmide de testes (proporção recomendada)

```
         /\          E2E: 5-10% (poucos, lentos, alto valor)
        /  \
       /    \        Integration: 20-30% (APIs, database)
      /      \
     /        \      Unit: 60-70% (rápidos, isolados)
    /          \
   /____________\
```

### Padrões de E2E

| Padrão | Uso |
|--------|-----|
| Page Object Model | Abstrai interação com UI |
| API-first E2E | Testa fluxos via API (mais estável que UI) |
| Scenario-based | Organiza por jornada do usuário |
| Data-driven | Mesmo cenário, múltiplos dados |
| Smoke Tests | Subset mínimo para validar deploy |

### E2E via API (recomendado como base)

```python
# tests/e2e/test_order_flow.py
import pytest
from httpx import AsyncClient

@pytest.mark.e2e
class TestOrderFlow:
    """Fluxo completo de compra: criar conta → adicionar item → checkout."""
    
    async def test_complete_purchase_flow(self, client: AsyncClient):
        # 1. Criar conta
        signup = await client.post("/api/v1/auth/signup", json={
            "email": "e2e@test.com", "password": "SecurePass123!"
        })
        assert signup.status_code == 201
        token = signup.json()["token"]
        headers = {"Authorization": f"Bearer {token}"}
        
        # 2. Adicionar item ao carrinho
        add_item = await client.post("/api/v1/cart/items", 
            json={"product_id": "prod_001", "quantity": 2},
            headers=headers)
        assert add_item.status_code == 200
        
        # 3. Realizar checkout
        checkout = await client.post("/api/v1/orders/checkout",
            json={"payment_method": "credit_card"},
            headers=headers)
        assert checkout.status_code == 201
        order_id = checkout.json()["order_id"]
        
        # 4. Verificar status do pedido
        order = await client.get(f"/api/v1/orders/{order_id}", headers=headers)
        assert order.status_code == 200
        assert order.json()["status"] == "confirmed"
```

### E2E com UI (Playwright)

```python
# tests/e2e/test_login_ui.py
from playwright.sync_api import Page, expect

def test_user_can_login_and_see_dashboard(page: Page):
    page.goto("/login")
    page.fill("[data-testid=email]", "user@example.com")
    page.fill("[data-testid=password]", "password123")
    page.click("[data-testid=login-button]")
    
    expect(page).to_have_url("/dashboard")
    expect(page.locator("[data-testid=welcome-message]")).to_be_visible()
```

### Boas práticas E2E
- Manter poucos (10-30 cenários críticos), não centenas
- Usar ambientes dedicados com dados seedados
- Isolar testes — cada teste cria e limpa seus dados
- Retry flaky tests no máximo 2x antes de investigar
- Separar smoke tests (< 5 min) de full E2E (< 30 min)

---

## 5. Análise de Dependências

### O que mede
A estrutura de dependências entre módulos, pacotes e serviços. Identifica:
- Dependências circulares
- Dependências transitivas excessivas
- Vulnerabilidades em dependências externas
- Violações de camadas arquiteturais

### Dependências internas (entre módulos)

**Python (pydeps, import-linter)**

```ini
# .importlinter
[importlinter]
root_package = my_service

[importlinter:contract:layers]
name = Layered architecture
type = layers
layers =
    my_service.routers
    my_service.services
    my_service.repositories
    my_service.models
```

```bash
# Verificar violações de camadas
import-linter --config=.importlinter

# Gerar grafo de dependências
pydeps src/my_service --max-bacon=2 --no-show
```

**Java (ArchUnit)**
```java
@AnalyzeClasses(packages = "com.company")
class ArchitectureTest {
    @ArchTest
    static final ArchRule layer_dependencies = layeredArchitecture()
        .consideringAllDependencies()
        .layer("Controllers").definedBy("..controllers..")
        .layer("Services").definedBy("..services..")
        .layer("Repositories").definedBy("..repositories..")
        .whereLayer("Controllers").mayOnlyAccessLayers("Services")
        .whereLayer("Services").mayOnlyAccessLayers("Repositories")
        .whereLayer("Repositories").mayNotAccessAnyLayer();
    
    @ArchTest
    static final ArchRule no_cycles = slices()
        .matching("com.company.(*)..")
        .should().beFreeOfCycles();
}
```

### Dependências externas (segurança e licenças)

```bash
# Python — auditar vulnerabilidades
pip-audit --requirement=requirements.txt --output=json

# Python — verificar licenças
pip-licenses --format=json --with-license-file

# Node.js
npm audit --json
npx license-checker --json

# Java
mvn org.owasp:dependency-check-maven:check
```

### Métricas de dependência

| Métrica | Fórmula | Threshold |
|---------|---------|-----------|
| Fan-in | Nº de módulos que dependem de X | Alto = muita responsabilidade |
| Fan-out | Nº de módulos que X depende | Alto = muito acoplado |
| Dependency Depth | Maior cadeia transitiva | < 5 níveis |
| Circular Dependencies | Ciclos no grafo | 0 (zero tolerância) |

---

## 6. Acoplamento (Coupling)

### O que mede
O grau de interdependência entre módulos. Alto acoplamento significa que mudanças em um módulo forçam mudanças em outros.

### Tipos de acoplamento (do pior ao melhor)

| Tipo | Gravidade | Descrição |
|------|-----------|-----------|
| Content Coupling | 🔴 Crítico | Módulo acessa internals de outro diretamente |
| Common Coupling | 🔴 Alto | Módulos compartilham estado global mutável |
| Control Coupling | 🟡 Médio | Módulo controla fluxo de outro via flags |
| Stamp Coupling | 🟡 Médio | Módulo recebe struct inteira mas usa parte |
| Data Coupling | 🟢 Baixo | Módulos comunicam apenas via parâmetros simples |
| Message Coupling | 🟢 Mínimo | Comunicação apenas via mensagens/eventos |

### Métricas de acoplamento

**Afferent Coupling (Ca)** — quem depende de mim
- Alto Ca = módulo é muito usado (estável, mas mudanças são caras)
- Típico para interfaces, modelos de domínio, utilities

**Efferent Coupling (Ce)** — de quem eu dependo
- Alto Ce = módulo depende de muitos (frágil, instável)
- Típico para controllers, orchestrators

### Medição automatizada

**Python (radon + custom)**
```python
# scripts/measure_coupling.py
"""Mede acoplamento entre módulos via análise de imports."""
import ast
from pathlib import Path
from collections import defaultdict

def analyze_imports(source_dir: str) -> dict:
    """Analisa imports e retorna grafo de dependências."""
    graph = defaultdict(set)
    
    for py_file in Path(source_dir).rglob("*.py"):
        module = str(py_file.relative_to(source_dir)).replace("/", ".").replace(".py", "")
        tree = ast.parse(py_file.read_text())
        
        for node in ast.walk(tree):
            if isinstance(node, ast.Import):
                for alias in node.names:
                    graph[module].add(alias.name)
            elif isinstance(node, ast.ImportFrom) and node.module:
                graph[module].add(node.module)
    
    return graph

def compute_coupling(graph: dict) -> dict:
    """Calcula Ca e Ce para cada módulo."""
    metrics = {}
    all_modules = set(graph.keys())
    
    for module in all_modules:
        ce = len(graph.get(module, set()) & all_modules)  # efferent
        ca = sum(1 for deps in graph.values() if module in deps)  # afferent
        metrics[module] = {"Ca": ca, "Ce": ce, "coupling_ratio": ce / max(ca + ce, 1)}
    
    return metrics
```

**Java (JDepend / ArchUnit)**
```java
@ArchTest
static final ArchRule services_should_not_be_highly_coupled =
    classes().that().resideInAPackage("..services..")
        .should().accessClassesThat().resideInAnyPackage(
            "..repositories..", "..models..", "..events.."
        );
```

### Thresholds

| Métrica | Aceitável | Investigar | Refatorar |
|---------|-----------|------------|-----------|
| Ce por módulo | < 7 | 7-12 | > 12 |
| Ca por módulo | < 20 | 20-40 | > 40 (exceto interfaces) |
| Dependências circulares | 0 | 1-2 | > 2 |

---

## 7. Abstração, Instabilidade e Distância da Sequência Principal

### O que mede
Métricas de Robert C. Martin que avaliam o equilíbrio entre estabilidade e flexibilidade de pacotes/módulos.

### Definições

**Abstração (A)**
- A = abstrações / total_de_classes (no pacote)
- Abstrações = interfaces + classes abstratas
- A = 0 → completamente concreto
- A = 1 → completamente abstrato

**Instabilidade (I)**
- I = Ce / (Ca + Ce)
- I = 0 → completamente estável (muitos dependem de mim, eu não dependo de ninguém)
- I = 1 → completamente instável (ninguém depende de mim, eu dependo de todos)

**Distância da Sequência Principal (D)**
- D = |A + I - 1|
- D = 0 → equilíbrio perfeito (na "main sequence")
- D → 1 → zona problemática

### Zonas de Risco

```
A (Abstração)
1.0 ┌─────────────────────────┐
    │  ZONA DE              / │
    │  INUTILIDADE        /   │  
    │  (muito abstrato, /     │
    │   ninguém usa)  /       │
    │               /         │
    │             / Sequência │
    │           /  Principal  │
    │         /    (ideal)    │
    │       /                 │
    │     /   ZONA DE DOR    │
    │   /    (muito concreto, │
    │ /      muito estável)   │
0.0 └─────────────────────────┘
    0.0                     1.0
              I (Instabilidade)
```

- **Zona de Dor** (A≈0, I≈0): Concreto E estável. Mudanças são difíceis e afetam muitos.
- **Zona de Inutilidade** (A≈1, I≈1): Abstrato E instável. Ninguém usa.
- **Sequência Principal** (A + I ≈ 1): Equilíbrio saudável.

### Medição

```python
# scripts/measure_stability.py
"""Calcula métricas de Martin para cada pacote."""
from dataclasses import dataclass

@dataclass
class PackageMetrics:
    name: str
    num_classes: int
    num_abstract: int  # interfaces + abstract classes
    ca: int  # afferent coupling
    ce: int  # efferent coupling
    
    @property
    def abstraction(self) -> float:
        return self.num_abstract / max(self.num_classes, 1)
    
    @property
    def instability(self) -> float:
        return self.ce / max(self.ca + self.ce, 1)
    
    @property
    def distance(self) -> float:
        return abs(self.abstraction + self.instability - 1)
    
    @property
    def zone(self) -> str:
        if self.distance < 0.3:
            return "✅ Main Sequence"
        elif self.abstraction < 0.3 and self.instability < 0.3:
            return "🔴 Zone of Pain"
        elif self.abstraction > 0.7 and self.instability > 0.7:
            return "🟡 Zone of Uselessness"
        else:
            return "🟡 Off Main Sequence"
```

### Thresholds

| Métrica | Saudável | Atenção | Problema |
|---------|----------|---------|----------|
| D (Distância) | < 0.3 | 0.3 - 0.5 | > 0.5 |
| Pacotes na Zona de Dor | 0-1 | 2-3 | > 3 |
| Pacotes na Zona de Inutilidade | 0 | 1-2 | > 2 |

---

## 8. Connascence

### O que mede
A qualidade e força do acoplamento entre componentes. Connascence é mais granular que "coupling" — classifica **como** dois componentes estão conectados.

### Taxonomia (da mais fraca/aceitável à mais forte/problemática)

#### Connascence Estática (detectável no código-fonte)

| Tipo | Descrição | Exemplo | Ação |
|------|-----------|---------|------|
| **Name** (CoN) | Concordância no nome | Chamar método pelo nome correto | ✅ Aceitável |
| **Type** (CoT) | Concordância no tipo | Parâmetro deve ser `int` | ✅ Aceitável |
| **Meaning** (CoM) | Concordância no significado de valores | `status=1` means "active" | 🟡 Usar enum |
| **Position** (CoP) | Concordância na ordem de parâmetros | `fn(name, age)` vs `fn(age, name)` | 🟡 Usar kwargs/named |
| **Algorithm** (CoA) | Concordância no algoritmo | Hash deve usar mesmo salt/rounds | 🔴 Encapsular |

#### Connascence Dinâmica (detectável apenas em runtime)

| Tipo | Descrição | Exemplo | Ação |
|------|-----------|---------|------|
| **Execution** (CoE) | Ordem de execução importa | `init()` antes de `run()` | 🟡 Tornar explícito |
| **Timing** (CoTm) | Timing importa | Race conditions | 🔴 Eliminar |
| **Value** (CoV) | Valores devem ser coordenados | Soma deve bater com partes | 🔴 Encapsular invariante |
| **Identity** (CoI) | Devem referenciar mesma instância | Dois módulos com mesmo objeto | 🔴 Refatorar |

### Propriedades para avaliação

1. **Força** — connascence mais forte = mais problemática
2. **Localidade** — connascence entre módulos distantes = pior que entre próximos
3. **Grau** — quantos componentes participam (2 é ok, 10 é problema)

### Regras de refatoração

```
Princípio: Converter connascence forte em fraca, ou dinâmica em estática.

CoM (Meaning) → Extrair para Enum ou constante nomeada
CoP (Position) → Usar named parameters ou objetos de parâmetro
CoA (Algorithm) → Encapsular algoritmo em módulo compartilhado
CoE (Execution) → Builder pattern ou state machine explícita
CoV (Value) → Invariante encapsulado em classe (ex: Money, DateRange)
CoI (Identity) → Dependency injection, não referência direta
```

### Detecção automatizada

```python
# scripts/detect_connascence.py
"""Detecta padrões de connascence no código."""
import ast

class ConnascenceDetector(ast.NodeVisitor):
    def __init__(self):
        self.issues = []
    
    def visit_Call(self, node):
        # CoP: funções com muitos args posicionais (> 3)
        if len(node.args) > 3 and not node.keywords:
            self.issues.append({
                "type": "CoP (Position)",
                "line": node.lineno,
                "msg": f"Chamada com {len(node.args)} args posicionais — usar kwargs",
            })
        self.generic_visit(node)
    
    def visit_Compare(self, node):
        # CoM: comparação com magic numbers/strings
        for comparator in node.comparators:
            if isinstance(comparator, ast.Constant) and isinstance(comparator.value, int):
                if comparator.value not in (0, 1, -1):
                    self.issues.append({
                        "type": "CoM (Meaning)",
                        "line": node.lineno,
                        "msg": f"Magic number {comparator.value} — extrair para constante",
                    })
        self.generic_visit(node)
```

---

## 9. Tamanho de Módulos

### O que mede
A dimensão dos módulos em termos de linhas, funções, classes e parâmetros. Módulos grandes demais indicam violação do Single Responsibility Principle.

### Métricas e thresholds

| Métrica | Aceitável | Atenção | Refatorar |
|---------|-----------|---------|-----------|
| Linhas por arquivo | < 300 | 300-500 | > 500 |
| Linhas por função | < 30 | 30-50 | > 50 |
| Linhas por classe | < 200 | 200-400 | > 400 |
| Funções por arquivo | < 15 | 15-25 | > 25 |
| Parâmetros por função | < 4 | 4-6 | > 6 |
| Métodos por classe | < 10 | 10-20 | > 20 |
| Profundidade de herança | < 3 | 3-5 | > 5 |
| Classes por arquivo | 1-2 | 3-4 | > 4 |

### Medição automatizada

**Python (radon raw)**
```bash
# Métricas brutas por arquivo
radon raw src/ --json | python -c "
import json, sys
data = json.load(sys.stdin)
for path, metrics in data.items():
    loc = metrics['loc']
    funcs = metrics['multi'] + metrics['single_comments']  # approximation
    if loc > 300:
        print(f'⚠️  {path}: {loc} linhas (max recomendado: 300)')
"
```

**Script genérico de medição**
```python
# scripts/measure_module_size.py
"""Mede tamanho de módulos e reporta violações."""
import ast
from pathlib import Path
from dataclasses import dataclass

@dataclass
class ModuleMetrics:
    path: str
    total_lines: int
    functions: int
    classes: int
    max_function_lines: int
    max_function_params: int
    max_class_lines: int

def analyze_module(filepath: Path) -> ModuleMetrics:
    source = filepath.read_text()
    tree = ast.parse(source)
    lines = source.splitlines()
    
    functions = []
    classes = []
    
    for node in ast.walk(tree):
        if isinstance(node, (ast.FunctionDef, ast.AsyncFunctionDef)):
            func_lines = node.end_lineno - node.lineno + 1
            func_params = len(node.args.args) + len(node.args.kwonlyargs)
            functions.append((node.name, func_lines, func_params))
        elif isinstance(node, ast.ClassDef):
            class_lines = node.end_lineno - node.lineno + 1
            classes.append((node.name, class_lines))
    
    return ModuleMetrics(
        path=str(filepath),
        total_lines=len(lines),
        functions=len(functions),
        classes=len(classes),
        max_function_lines=max((f[1] for f in functions), default=0),
        max_function_params=max((f[2] for f in functions), default=0),
        max_class_lines=max((c[1] for c in classes), default=0),
    )
```

---

## 10. Complexidade Ciclomática

### O que mede
O número de caminhos linearmente independentes no código. Cada `if`, `elif`, `for`, `while`, `and`, `or`, `except`, `case` adiciona um caminho.

### Fórmula
```
CC = E - N + 2P

Onde:
  E = número de arestas no grafo de controle de fluxo
  N = número de nós
  P = número de componentes conectados (geralmente 1 por função)

Na prática: CC = 1 + (número de pontos de decisão)
```

### Thresholds

| CC | Risco | Ação |
|----|-------|------|
| 1-5 | 🟢 Baixo | Simples, fácil de testar |
| 6-10 | 🟡 Moderado | Aceitável, mas monitorar |
| 11-20 | 🟠 Alto | Difícil de testar, considerar refatorar |
| 21-50 | 🔴 Muito alto | Refatorar obrigatoriamente |
| > 50 | ⚫ Crítico | Bug provável, refatorar imediatamente |

### Relação CC × Testes
- **Mínimo de test cases** para cobertura de branch = CC
- Uma função com CC=15 precisa de no mínimo 15 test cases para cobertura completa

### Medição

**Python (radon)**
```bash
# Complexidade por função (grade A-F)
radon cc src/ --min B --show-complexity --json

# Média por módulo
radon cc src/ --average

# Apenas funções com CC > 10 (precisam refatoração)
radon cc src/ --min C --no-assert
```

**Configuração CI**
```toml
# pyproject.toml — usar com flake8 ou ruff
[tool.ruff.lint]
select = ["C901"]  # McCabe complexity

[tool.ruff.lint.mccabe]
max-complexity = 10
```

**Java (PMD)**
```xml
<rule ref="category/java/design.xml/CyclomaticComplexity">
    <properties>
        <property name="methodReportLevel" value="10"/>
        <property name="classReportLevel" value="80"/>
    </properties>
</rule>
```

### Padrões de refatoração para reduzir CC

| Padrão | Quando usar |
|--------|-------------|
| Extract Method | Função faz muitas coisas |
| Replace Conditional with Polymorphism | if/switch por tipo |
| Strategy Pattern | Algoritmo varia por contexto |
| Guard Clauses | Nested ifs profundos |
| Table-Driven Methods | Mapeamento condição → ação |
| State Machine | Transições complexas de estado |

```python
# ❌ CC = 12 (difícil de testar)
def calculate_price(product, user, coupon):
    price = product.base_price
    if user.is_premium:
        if user.years > 5:
            price *= 0.7
        else:
            price *= 0.85
    elif user.is_member:
        price *= 0.9
    if coupon:
        if coupon.is_percentage:
            price *= (1 - coupon.value / 100)
        else:
            price -= coupon.value
    if price < 0:
        price = 0
    return price

# ✅ CC = 3 por função (fácil de testar independentemente)
def apply_user_discount(price: float, user: User) -> float:
    discount = USER_DISCOUNTS.get(user.tier, 0)
    return price * (1 - discount)

def apply_coupon(price: float, coupon: Coupon | None) -> float:
    if not coupon:
        return price
    return coupon.apply(price)

def calculate_price(product: Product, user: User, coupon: Coupon | None) -> float:
    price = product.base_price
    price = apply_user_discount(price, user)
    price = apply_coupon(price, coupon)
    return max(price, 0)
```

---

## 11. Cognitive Complexity

### O que mede
A dificuldade de **entender** o código (diferente de CC que mede paths). Penaliza:
- Nesting (cada nível de indentação adiciona peso)
- Breaks no fluxo linear (goto, break, continue, recursão)
- Sequências de operadores lógicos

### Diferença entre CC e Cognitive Complexity

```python
# CC = 4, Cognitive = 9 (difícil de ler por causa do nesting)
def process(items):                    # +0
    for item in items:                 # +1 (CC) +1 (cog)
        if item.is_valid:              # +1 (CC) +2 (cog: +1 nesting)
            if item.has_discount:      # +1 (CC) +3 (cog: +2 nesting)
                apply_discount(item)
            else:                      # +1 (CC) +1 (cog)
                apply_regular(item)

# CC = 4, Cognitive = 4 (flat structure, fácil de ler)
def process(items):
    valid = [i for i in items if i.is_valid]           # +1
    discounted = [i for i in valid if i.has_discount]  # +1
    regular = [i for i in valid if not i.has_discount] # +1
    for item in discounted: apply_discount(item)       # +1
    for item in regular: apply_regular(item)
```

### Threshold
- **< 15** por função: saudável
- **> 15**: refatorar para reduzir nesting

### Ferramenta
```bash
# SonarQube mede nativamente
# Para Python, usar cognitive-complexity plugin do flake8
pip install flake8-cognitive-complexity
flake8 --max-cognitive-complexity=15 src/
```

---

## 12. Code Churn e Hotspots

### O que mede
A frequência de mudanças em cada arquivo ao longo do tempo. Arquivos com alta churn + alta complexidade são **hotspots** — maiores fontes de bugs.

### Métricas

| Métrica | O que indica |
|---------|-------------|
| Churn (mudanças/período) | Instabilidade do arquivo |
| Churn × Complexity | Risco de bugs (hotspot) |
| Authors per file | Ownership difuso |
| Age since last change | Código estável ou abandonado |

### Medição via git

```bash
# Top 20 arquivos mais alterados nos últimos 6 meses
git log --since="6 months ago" --name-only --pretty=format: | \
    sort | uniq -c | sort -rn | head -20

# Hotspots: churn × complexity
# Combinar com radon/PMD para score composto
```

```python
# scripts/detect_hotspots.py
"""Identifica hotspots: alta churn + alta complexidade."""
import subprocess
import json

def get_churn(months: int = 6) -> dict[str, int]:
    """Conta mudanças por arquivo nos últimos N meses."""
    result = subprocess.run(
        ["git", "log", f"--since={months} months ago", "--name-only", "--pretty=format:"],
        capture_output=True, text=True
    )
    churn = {}
    for line in result.stdout.strip().split("\n"):
        if line.strip():
            churn[line.strip()] = churn.get(line.strip(), 0) + 1
    return churn

def get_complexity(source_dir: str) -> dict[str, float]:
    """Obtém complexidade média por arquivo via radon."""
    result = subprocess.run(
        ["radon", "cc", source_dir, "--json", "--average"],
        capture_output=True, text=True
    )
    # Parse e retorna CC médio por arquivo
    data = json.loads(result.stdout)
    return {path: sum(f["complexity"] for f in funcs) / max(len(funcs), 1) 
            for path, funcs in data.items()}

def detect_hotspots(source_dir: str, months: int = 6):
    churn = get_churn(months)
    complexity = get_complexity(source_dir)
    
    hotspots = []
    for path, changes in churn.items():
        cc = complexity.get(path, 0)
        risk = changes * cc
        if risk > 50:  # threshold
            hotspots.append({"file": path, "churn": changes, "cc": cc, "risk": risk})
    
    return sorted(hotspots, key=lambda x: x["risk"], reverse=True)
```

---

## 13. Duplicação de Código (DRY Violations)

### O que mede
Percentual de código duplicado ou quase-duplicado. Duplicação é a raiz de inconsistências — quando um bug é corrigido em uma cópia mas não nas outras.

### Tipos

| Tipo | Descrição | Detecção |
|------|-----------|----------|
| Tipo 1 | Cópia exata (ignoring whitespace) | Trivial |
| Tipo 2 | Cópia com renaming de variáveis | AST-based |
| Tipo 3 | Cópia com alterações (linhas adicionadas/removidas) | Token-based |
| Tipo 4 | Funcionalidade equivalente, implementação diferente | Semântica |

### Thresholds

| Duplicação | Status |
|-----------|--------|
| < 3% | 🟢 Excelente |
| 3-5% | 🟡 Aceitável |
| 5-10% | 🟠 Investigar |
| > 10% | 🔴 Refatorar |

### Ferramentas

```bash
# Python (pylint)
pylint --disable=all --enable=duplicate-code src/

# Multi-linguagem (jscpd — JS Copy/Paste Detector)
npx jscpd src/ --min-lines=5 --min-tokens=50 --reporters=json,html

# Java (PMD CPD)
mvn pmd:cpd-check -Dminimumtokens=100
```

---

## 14. Maintainability Index

### O que mede
Score composto que combina múltiplas métricas em um único número de 0-100 indicando quão fácil é manter o código.

### Fórmula (SEI/Microsoft)
```
MI = max(0, (171 - 5.2 * ln(V) - 0.23 * CC - 16.2 * ln(LOC)) * 100 / 171)

Onde:
  V = Halstead Volume
  CC = Cyclomatic Complexity
  LOC = Lines of Code
```

### Thresholds

| MI | Status | Interpretação |
|----|--------|---------------|
| 85-100 | 🟢 Alta | Fácil de manter |
| 65-84 | 🟡 Moderada | Aceitável |
| < 65 | 🔴 Baixa | Difícil de manter |

### Medição

```bash
# Python (radon)
radon mi src/ --show --min B

# Apenas arquivos com MI ruim
radon mi src/ --max A  # mostra apenas MI < 10
```

---

## 15. Testes de Performance e Benchmarks

### O que mede
Se o código mantém características de performance aceitáveis ao longo do tempo. Previne regressões de performance.

### Tipos

| Tipo | O que valida |
|------|-------------|
| Micro-benchmark | Tempo de execução de função específica |
| Load Testing | Comportamento sob carga |
| Stress Testing | Ponto de quebra do sistema |
| Soak Testing | Estabilidade sob carga prolongada |
| Performance Regression | Comparação com baseline |

### Benchmark como teste

```python
# tests/benchmarks/test_performance.py
import pytest
import time

PERFORMANCE_BUDGET = {
    "process_batch_1000": 2.0,  # max 2 segundos
    "search_index_query": 0.1,  # max 100ms
    "serialize_response": 0.05,  # max 50ms
}

@pytest.mark.benchmark
def test_process_batch_within_budget():
    start = time.perf_counter()
    result = process_batch(generate_items(1000))
    elapsed = time.perf_counter() - start
    
    assert elapsed < PERFORMANCE_BUDGET["process_batch_1000"], (
        f"Performance regression: {elapsed:.2f}s > {PERFORMANCE_BUDGET['process_batch_1000']}s"
    )
    assert len(result) == 1000

# Com pytest-benchmark (mais preciso)
def test_serialize_performance(benchmark):
    data = generate_large_response()
    result = benchmark(serialize_response, data)
    assert benchmark.stats["mean"] < PERFORMANCE_BUDGET["serialize_response"]
```

---

## 16. Testes de Propriedade (Property-Based Testing)

### O que mede
Se o código satisfaz **invariantes** para qualquer input válido, não apenas para exemplos manuais. Gera centenas de inputs aleatórios e verifica propriedades universais.

### Quando usar
- Funções puras com domínio definível
- Serialization/deserialization (roundtrip)
- Operações idempotentes
- Invariantes matemáticas
- Parsers e transformadores

```python
# tests/properties/test_serialization.py
from hypothesis import given, strategies as st

@given(st.dictionaries(st.text(min_size=1), st.integers()))
def test_json_roundtrip(data):
    """Serializar e deserializar deve retornar o original."""
    serialized = serialize(data)
    deserialized = deserialize(serialized)
    assert deserialized == data

@given(st.lists(st.integers()))
def test_sort_preserves_length(items):
    """Ordenar nunca perde ou adiciona elementos."""
    sorted_items = custom_sort(items)
    assert len(sorted_items) == len(items)
    assert set(sorted_items) == set(items)

@given(st.integers(min_value=0), st.integers(min_value=0))
def test_discount_never_exceeds_original(price, percentage):
    """Desconto nunca resulta em preço negativo ou maior que original."""
    percentage = percentage % 101  # bound to 0-100
    result = apply_discount(price, percentage)
    assert 0 <= result <= price
```

---

## 17. Dead Code Detection

### O que mede
Código que nunca é executado — funções não chamadas, imports não usados, variáveis não lidas, branches inalcançáveis.

### Ferramentas

```bash
# Python (vulture)
vulture src/ --min-confidence=80

# Python (dead code — importações)
autoflake --check --remove-all-unused-imports src/

# JavaScript/TypeScript (ts-prune)
npx ts-prune --project tsconfig.json

# Java (IntelliJ/PMD)
# PMD rule: UnusedPrivateMethod, UnusedLocalVariable, UnusedFormalParameter
```

### Threshold
- **0 dead code** em código novo (CI gate)
- Projetos legados: reduzir 10% por sprint até zero

---

## 18. Testes de Segurança Estática (SAST)

### O que mede
Vulnerabilidades no código-fonte sem executá-lo. Complementa testes funcionais com análise de segurança.

### Ferramentas por linguagem

| Linguagem | Ferramenta | Foco |
|-----------|-----------|------|
| Python | Bandit, Semgrep | Injection, crypto, exec |
| Java | SpotBugs + FindSecBugs, Semgrep | SQL injection, XSS, deserialization |
| JavaScript | ESLint security plugin, Semgrep | XSS, prototype pollution |
| Multi | SonarQube, Snyk Code | Cross-language |

```bash
# Python
bandit -r src/ -f json -o bandit-report.json

# Semgrep (multi-language)
semgrep --config=auto src/ --json > semgrep-report.json
```

---

## Dashboard de Qualidade — Visão Consolidada

### Score composto recomendado

```python
# scripts/quality_score.py
"""Calcula score de qualidade composto do projeto."""

WEIGHTS = {
    "coverage_branch": 0.15,
    "mutation_score": 0.15,
    "cyclomatic_avg": 0.10,
    "cognitive_avg": 0.10,
    "maintainability_index": 0.10,
    "duplication": 0.10,
    "coupling_distance": 0.10,
    "dead_code_pct": 0.05,
    "dependency_vulnerabilities": 0.05,
    "module_size_violations": 0.05,
    "connascence_strong": 0.05,
}

def calculate_quality_score(metrics: dict) -> float:
    """Retorna score 0-100 ponderado."""
    score = 0
    for metric, weight in WEIGHTS.items():
        raw = metrics.get(metric, 0)
        normalized = normalize_metric(metric, raw)  # 0-100
        score += normalized * weight
    return round(score, 1)
```

### CI Quality Gate

```yaml
# .github/workflows/quality-gate.yml
quality-gate:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    
    - name: Coverage
      run: pytest --cov=src --cov-branch --cov-fail-under=80
    
    - name: Complexity
      run: |
        radon cc src/ --min C --no-assert
        if [ $? -ne 0 ]; then echo "❌ Funções com CC > 10"; exit 1; fi
    
    - name: Duplication
      run: |
        npx jscpd src/ --threshold=5 --exitCode=1
    
    - name: Dead Code
      run: vulture src/ --min-confidence=90
    
    - name: Security
      run: bandit -r src/ --severity-level medium
    
    - name: Architecture
      run: import-linter --config=.importlinter
    
    - name: Module Size
      run: python scripts/check_module_sizes.py --max-lines=500 --max-cc=15
```

---

## Regras para o Agente

### Ao criar testes de qualidade:

1. **Começar pelo coverage** — é a base. Sem cobertura, outras métricas são inúteis.
2. **Mutation testing apenas em código crítico** — domínio, regras de negócio, cálculos financeiros.
3. **E2E mínimo e estável** — poucos cenários de alto valor, nunca dezenas de testes frágeis.
4. **Medir acoplamento antes de refatorar** — dados guiam a priorização.
5. **Complexidade ciclomática como gate** — nenhuma função nova com CC > 10.
6. **Connascence** — converter forte em fraca antes de adicionar features.
7. **Não perseguir 100%** — o custo marginal de 95% → 100% raramente justifica.
8. **Automatizar tudo no CI** — métricas que não rodam automaticamente morrem.
9. **Trending > absoluto** — mais importante que o número atual é se está melhorando ou piorando.
10. **Property-based para código puro** — complementa testes example-based em funções sem side-effects.

### Priorização de implementação

| Fase | Métricas | Esforço |
|------|----------|---------|
| 1 (Quick Wins) | Coverage, CC, Module Size, Dead Code | Baixo |
| 2 (Arquitetura) | Acoplamento, Dependências, Distância | Médio |
| 3 (Profundidade) | Mutation, Connascence, Hotspots | Alto |
| 4 (Performance) | Benchmarks, Load Testing, Churn | Alto |

### Quando recomendar cada teste

| Situação | Testes recomendados |
|----------|---------------------|
| Projeto novo | Coverage + CC + Architecture rules |
| Antes de refatoração | Hotspots + Coupling + Module Size |
| Bug em produção | Regression + Mutation no módulo afetado |
| Nova feature crítica | Property-based + E2E + Mutation |
| Preparar para escalar | Performance + Load + Dependency analysis |
| Audit de qualidade | Dashboard completo (todas as métricas) |
| Code review | CC + Connascence + Coverage delta |
