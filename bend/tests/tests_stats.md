# Profiling and Runtime Statistics Subskill (`bend/tests/stats`)

## Overview
This subskill guides runtime profiling, memory usage analysis, and interaction net rewrite throughput monitoring.

## Key Concepts
- **Rewrites per Second (RPS)**: Measure of execution throughput across CPU cores or GPU warps.
- **Node Allocations**: Tracking peak active interaction net nodes to prevent memory exhaustion.
- **Parallel Speedup**: Comparing single-threaded CPU reduction vs CUDA GPU performance.

## Agent Checklist
- Check performance counters when optimizing compute-intensive parallel pipelines.
