---
name: dod
description: Data-Oriented Design for performance-sensitive software. Guides data-first modeling, cache-friendly memory layouts, contiguous storage, batch processing, ECS, vectorization, profiling, and pragmatic trade-offs across systems, simulations, games, and analytics.
keywords: ["dod", "data-oriented-design", "data-oriented", "cache", "data-locality", "memory-layout", "soa", "aos", "ecs", "batch-processing", "vectorization", "simd", "performance"]
---

# Data-Oriented Design (DOD)

## What It Is

Data-Oriented Design is a software engineering and optimization approach that starts with the data being transformed and the way that data is accessed. It chooses representations, memory layouts, and processing loops to improve locality, throughput, and predictable performance instead of beginning with abstract object hierarchies.

DOD is especially useful for hot paths that process many homogeneous items: simulations, game engines, rendering, telemetry, numerical workloads, media processing, and large data pipelines. It is not a universal replacement for object-oriented design. Use it where measurements show that memory traffic, allocation, indirection, branch behavior, or per-object overhead matters.

DOD is related to, but distinct from, Data-Oriented Programming (DOP):

- **DOD** focuses primarily on physical representation, access patterns, locality, allocation, and execution efficiency.
- **DOP** focuses primarily on separating data from behavior and composing transformations, often with immutable data and pure functions.
- They can be combined, but DOD may use carefully controlled mutation, pools, arenas, or in-place updates when that produces a measurable benefit.

---

## Core Principles

### 1. Start With the Workload

Before designing types or classes, answer:

1. What data is transformed?
2. Which fields are read together?
3. Which fields are written together?
4. How many items are processed per frame, request, or batch?
5. What is the dominant access pattern: sequential, random, lookup, grouping, or append?
6. What are the performance and memory budgets?

The data layout should follow the dominant workload, not the conceptual shape of one individual entity.

### 2. Optimize Data Movement Before Instruction Count

Modern CPUs can execute arithmetic quickly, but waiting for data from main memory is expensive. The cost of a loop often comes from cache misses, pointer chasing, allocations, synchronization, and unnecessary bytes moved rather than from the arithmetic itself.

Prioritize, in roughly this order:

- avoid loading fields that the current operation does not use;
- keep frequently co-accessed data close together;
- process data sequentially when possible;
- reduce pointer indirection and scattered heap allocations;
- reuse storage and avoid allocation in hot loops;
- measure before and after changes.

Cache awareness is a design constraint, not a guarantee that every contiguous layout is automatically faster. A layout that is sequential but loads many unused bytes can still be inferior to a compact layout tailored to the access pattern.

### 3. Separate Data From Processing Logic

Represent state as simple, explicit data and implement operations as systems or functions over ranges of data. Avoid requiring every item to carry behavior, a virtual table pointer, ownership metadata, and unrelated fields when a batch operation only needs two numeric columns.

This separation improves:

- reasoning about access patterns;
- testing transformations independently;
- batching and vectorization;
- parallel execution over independent partitions;
- instrumentation of input/output counts and timings.

Separation does not mean that all code must be functional or immutable. It means that data representation and the code that processes it should be designed independently and deliberately.

### 4. Prefer Dense, Contiguous Storage

Use contiguous arrays, packed buffers, columnar tables, pools, arenas, or chunks when the workload benefits from sequential traversal. Contiguous storage improves spatial locality and allows hardware prefetching to work effectively.

Avoid linked lists, scattered object graphs, and collections of individually allocated objects in performance-critical loops unless the access pattern truly requires them.

### 5. Process Homogeneous Batches

Operate on ranges, chunks, columns, or groups of similar records instead of dispatching one call per item. Batch processing amortizes loop setup, enables SIMD/vectorization, reduces call and allocation overhead, and makes parallel partitioning easier.

Keep the hot loop simple and predictable. Move validation, logging, formatting, and exceptional cases outside it whenever possible.

### 6. Design for Predictable Control Flow

Branch misprediction can be costly in tight loops. Reduce unpredictable branches by:

- partitioning data into homogeneous groups;
- using masks or branchless arithmetic when it is clearer and measured to help;
- separating common and exceptional paths;
- replacing deep polymorphic dispatch with explicit system passes where appropriate;
- sorting or grouping by frequently tested state.

Do not remove branches blindly. A predictable branch is often cheaper and clearer than complicated branchless code.

---

## Array of Structures (AoS) vs Structure of Arrays (SoA)

### Array of Structures (AoS)

Each item stores all of its fields together:

```text
[position, velocity, health, name] [position, velocity, health, name] ...
```

AoS is a good fit when most operations consume nearly every field of one item, when records are passed together across an API, or when the data set is small and simplicity dominates.

### Structure of Arrays (SoA)

Each field is stored in its own contiguous array:

```text
positions: [p0, p1, p2, ...]
velocities: [v0, v1, v2, ...]
health:    [h0, h1, h2, ...]
```

SoA is a good fit when systems process one or a few fields across many items. It avoids loading unrelated fields and is friendly to SIMD, vectorized libraries, columnar engines, and parallel range processing.

### Array of Structures of Arrays (AoSoA)

AoSoA divides data into fixed-size tiles, with SoA fields inside each tile. It can combine SIMD-friendly blocks with locality for operations that process small groups:

```text
chunk 0: positions[W], velocities[W], health[W]
chunk 1: positions[W], velocities[W], health[W]
```

Use AoSoA when measurements show that pure AoS or pure SoA does not fit the workload, especially with explicit SIMD widths, tiled algorithms, or cache-sized chunks.

### Decision Table

| Access pattern | Often suitable | Reason |
|---|---|---|
| Read nearly every field of each record | AoS | One record fetch supplies most required data |
| Update one numeric field for all records | SoA | Only the required column is loaded |
| Multiple systems use different field subsets | SoA or ECS archetypes | Each system gets a compact working set |
| SIMD over fixed-size groups | AoSoA | Tiles align data with vector width and cache behavior |
| Random lookup by stable key | Indexed SoA, dense map, or hybrid | Separates lookup structure from hot storage |
| Small configuration or infrequently accessed state | Separate cold storage | Keeps cold bytes out of hot cache lines |

There is no universally best layout. Choose based on measured access patterns and update frequency.

---

## Hot, Warm, and Cold Data

Classify fields by how often and where they are used:

- **Hot data:** read or written in every iteration of a hot system. Keep compact and contiguous.
- **Warm data:** used by several systems but not every iteration. Store separately when it reduces hot working-set size.
- **Cold data:** metadata, debug labels, descriptions, configuration, or rarely used state. Keep out of the hot path and access it through an ID or index.

A common layout is an index into dense hot arrays plus a separate map or table for cold metadata:

```python
# Hot path: compact numeric arrays
positions = np.zeros((entity_count, 3), dtype=np.float32)
velocities = np.zeros((entity_count, 3), dtype=np.float32)

# Cold path: accessed only when displaying or diagnosing an entity
labels_by_entity_id: dict[int, str] = {}
```

Do not add large strings, optional payloads, debug data, or rarely used flags to every hot record merely because they conceptually belong to the same entity.

---

## Entity-Component-System (ECS)

ECS is a practical DOD architecture for large collections of entities:

- **Entities** are stable identifiers or indices, not behavior-heavy objects.
- **Components** are compact data sets, such as `Position`, `Velocity`, or `Health`.
- **Systems** contain logic and iterate over entities that have the required components.

Conceptually:

```text
Entity 42 -> Position + Velocity + Health
Entity 43 -> Position + Renderable

MovementSystem -> Position + Velocity
DamageSystem  -> Health + DamageEvent
RenderSystem   -> Position + Renderable
```

A minimal Python representation for a batch-oriented system can be simple arrays:

```python
import numpy as np

positions = np.array([[0.0, 1.0], [2.0, 3.0]], dtype=np.float32)
velocities = np.array([[0.5, 0.0], [-1.0, 0.25]], dtype=np.float32)


def integrate_positions(
    positions: np.ndarray,
    velocities: np.ndarray,
    delta_time: float,
) -> None:
    """Update all positions in one contiguous, vectorized pass."""
    positions += velocities * np.float32(delta_time)
```

For larger systems, consider sparse component membership, dense-set plus sparse-set indexing, archetype chunks, or query plans. The implementation should match the required operations; do not introduce a complete ECS framework for a small problem.

### ECS Invariants

When implementing ECS-like storage, document and enforce:

- entity ID validity and generation/version rules;
- component membership and query semantics;
- dense-array index maintenance after removal or compaction;
- stable versus unstable iteration order;
- ownership and lifetime of component storage;
- behavior when an entity is destroyed during iteration.

---

## Practical Implementation Patterns

### Dense Sets and Swap-Remove

When order is not required, dense storage can remove an item in constant time by moving the last item into the removed slot and updating its index:

```python
def remove_unordered(
    values: list[float],
    index_by_id: dict[int, int],
    id_by_index: list[int],
    entity_id: int,
) -> None:
    """Remove an entity while keeping storage dense; order is not preserved."""
    removed_index = index_by_id.pop(entity_id)
    last_index = len(id_by_index) - 1
    last_entity_id = id_by_index[last_index]

    if removed_index != last_index:
        values[removed_index] = values[last_index]
        id_by_index[removed_index] = last_entity_id
        index_by_id[last_entity_id] = removed_index

    values.pop()
    id_by_index.pop()
```

Use this only when callers tolerate unstable order and when the index bookkeeping is tested. For stable order, use a different strategy and accept its cost.

### Batch and Chunk Processing

Process data in chunks when the complete working set does not fit in cache or memory:

```python
def process_chunks(values: np.ndarray, chunk_size: int) -> np.ndarray:
    """Apply a transformation without requiring a second full-size working set."""
    result = np.empty_like(values)
    for start in range(0, len(values), chunk_size):
        end = min(start + chunk_size, len(values))
        result[start:end] = np.sqrt(values[start:end])
    return result
```

Choose chunk sizes through measurement. A chunk that is too small increases loop overhead; one that is too large increases cache pressure.

### Masks and Partitioned Paths

Represent eligibility as compact masks or partition data before expensive processing:

```python
active = health > 0
positions[active] += velocities[active] * delta_time
```

If the same condition is evaluated repeatedly, consider partitioning active and inactive items or maintaining a dense active set. Weigh the cost of maintaining the partition against the savings in repeated filtering.

### Indexes for Indirect Access

Separate stable identity from physical location. Use an ID-to-index map when callers require stable IDs but systems require dense arrays. Treat indexes as derived state and update them atomically with storage mutations.

### Object Pools and Arenas

Reuse allocations for short-lived, homogeneous objects when allocation and garbage collection are measurable bottlenecks. Pools add lifecycle complexity and can retain memory longer than necessary, so apply them only to measured hot paths.

### Double Buffering

For simulations or parallel stages, keep separate read and write buffers when systems must observe a consistent snapshot:

```text
read_state  -> system updates -> write_state
swap(read_state, write_state)
```

Double buffering can simplify dependencies and avoid write-after-read hazards, but it increases memory traffic. Prefer in-place updates when the dependency graph allows them and profiling supports the choice.

---

## Language and Runtime Guidance

### Python and NumPy/Pandas

- Prefer NumPy arrays with explicit numeric dtypes over Python lists for numeric hot paths.
- Prefer vectorized operations, boolean masks, and columnar DataFrames over row-by-row loops.
- Avoid `object` columns containing heterogeneous Python objects in performance-sensitive tables.
- Keep large arrays contiguous where the library expects it; inspect `dtype`, shape, and strides when necessary.
- Use chunking, Arrow, Parquet, Polars, or a compiled extension when data volume exceeds practical Python-loop performance.
- Do not assume `apply`, `iterrows`, or a list comprehension is vectorized; verify the implementation and benchmark.

### C, C++, Rust, and Similar Systems Languages

- Prefer contiguous containers and explicit ownership over scattered allocations.
- Reserve capacity when the final size is known to avoid repeated reallocation.
- Keep hot structs small and move cold fields to separate storage.
- Be deliberate about alignment, padding, false sharing, and SIMD only after profiling.
- Avoid virtual dispatch and pointer chasing in hot loops when a data-driven dispatch or batch pass is appropriate.
- Preserve safety and clear invariants; undefined behavior is not a valid optimization.

### SQL and Columnar Analytics

- Select only required columns.
- Filter and aggregate close to the storage engine.
- Use partitioning and clustering keys that match common predicates.
- Prefer set-based transformations to row-by-row procedural work.
- Inspect query plans and bytes scanned; columnar storage is a DOD-friendly representation, but poor predicates can still force excessive I/O.

---

## Performance Measurement

Never claim a DOD optimization based only on intuition. Establish a baseline and measure the relevant resource:

- wall-clock latency and throughput;
- CPU cycles and instructions;
- cache references and cache misses;
- branch instructions and mispredictions;
- allocations and garbage-collection time;
- peak and resident memory;
- bytes read/written and query bytes scanned;
- energy or thermal behavior when relevant.

Use representative data distributions, realistic batch sizes, warm-up behavior, and production-like hardware. Report variance, not only one run.

A practical workflow:

1. Identify the hot operation from profiling.
2. Define correctness invariants and a benchmark fixture.
3. Record a baseline.
4. Change one layout, access pattern, or algorithm at a time.
5. Re-run correctness tests and benchmarks.
6. Inspect whether the bottleneck moved.
7. Keep the change only if the improvement justifies complexity and memory costs.

Tools may include language profilers, benchmark harnesses, compiler reports, allocation profilers, Linux `perf`, platform tracing tools, and database query plans. Choose tools appropriate to the runtime and environment.

---

## Common Anti-Patterns

### Modeling Every Item as a Rich Object

**Problem:** Per-object headers, pointers, virtual dispatch, and unrelated fields increase memory footprint and scatter data.

**Better:** Use IDs and dense component arrays for the hot path; retain rich objects only at boundaries or for cold metadata.

### Prematurely Choosing SoA

**Problem:** SoA can make APIs, ownership, serialization, and operations that need complete records more complicated.

**Better:** Start from measured access patterns. Use AoS for record-oriented workloads and hybrid layouts when appropriate.

### Optimizing Arithmetic While Ignoring Memory

**Problem:** Replacing a multiply with a cheaper instruction rarely matters if the loop is waiting on cache misses or I/O.

**Better:** Measure cache behavior, bytes moved, allocation, and locality first.

### Hidden Allocations in Hot Loops

**Problem:** Temporary objects, conversions, closures, string formatting, and growing collections create allocator and GC pressure.

**Better:** Reuse buffers, preallocate capacity, batch operations, and move diagnostics outside the hot loop.

### Random Access to a Dense Layout

**Problem:** A contiguous array does not help if every iteration follows a random index chain.

**Better:** Reorder work, sort/group indices, use tiled processing, or choose an index structure that matches the lookup pattern.

### Overusing Branchless Code

**Problem:** Branchless expressions can increase instructions, evaluate unnecessary work, or reduce readability.

**Better:** Compare predictable branching, masks, and partitioning with a benchmark.

### Ignoring Concurrency Effects

**Problem:** Multiple workers writing adjacent memory can cause false sharing, synchronization overhead, or nondeterministic behavior.

**Better:** Partition ownership by cache-friendly blocks, minimize shared writes, and use reductions or per-worker buffers.

### Treating Immutability as Free

**Problem:** Copying large arrays on every transformation can overwhelm memory bandwidth.

**Better:** Use views, copy-on-write, double buffering, or controlled in-place mutation where correctness and ownership are explicit.

### Introducing ECS Everywhere

**Problem:** A full ECS adds lifecycle, query, indexing, and debugging complexity to small applications.

**Better:** Apply the smallest data-oriented structure that solves the measured bottleneck.

---

## Design Checklist

Before implementing a performance-sensitive data path, verify:

- [ ] The dominant workload and hot loop are identified.
- [ ] Fields are grouped by access frequency and co-access pattern.
- [ ] Hot data is compact and cold data is separated.
- [ ] The chosen AoS, SoA, AoSoA, or hybrid layout is justified by access patterns.
- [ ] Iteration is sequential or has an intentional index strategy.
- [ ] Batch processing replaces unnecessary per-item dispatch.
- [ ] Allocations, copies, and conversions are absent from the hot path or justified.
- [ ] Branches and polymorphic calls are predictable or measured.
- [ ] Entity/component indexes and mutation invariants are explicit where applicable.
- [ ] Concurrency ownership avoids races and false sharing.
- [ ] Correctness tests cover compaction, removal, empty inputs, boundaries, and numerical edge cases.
- [ ] A representative benchmark compares the old and new implementations.
- [ ] The complexity and memory cost of the design are documented.

---

## Rules for the Agent

When designing or modifying performance-sensitive code:

1. Ask what data is transformed and how it is accessed before proposing classes or abstractions.
2. Inspect the existing workload, data volume, and bottleneck; do not impose DOD on cold or insignificant paths.
3. Prefer dense, contiguous, batch-friendly representations for homogeneous hot data.
4. Separate hot, warm, and cold fields when unrelated data would pollute cache lines.
5. Choose AoS, SoA, AoSoA, ECS, or a hybrid based on access patterns and measurements.
6. Keep behavior in explicit systems or transformations over well-defined data ranges.
7. Avoid allocations, pointer chasing, per-item virtual dispatch, and unpredictable branching in hot loops unless justified.
8. Preserve data ownership, identity, ordering, lifecycle, and concurrency invariants.
9. Profile before optimizing and benchmark after optimizing; never promise a speedup without evidence.
10. Prefer the simplest design that achieves the measured goal. Maintainability, correctness, and observability remain requirements.
