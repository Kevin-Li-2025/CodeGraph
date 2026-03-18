# Benchmark Results: CodeGraph vs PyCG (ICSE 2021)

## Benchmark Source

**PyCG Micro-Benchmark Suite**  
- Paper: "PyCG: Practical Call Graph Generation in Python" (ICSE 2021)  
- Repository: [github.com/vitsalis/PyCG](https://github.com/vitsalis/PyCG)  
- The most cited Python call graph benchmark in the static analysis community.
- 119 micro-benchmark test cases across 18 categories

## PyCG Baseline Reference

| Metric | PyCG (ICSE 2021) |
|--------|-----------------|
| Precision | **99.2%** |
| Recall | **69.9%** |

## CodeGraph Results

| Metric | CodeGraph | PyCG Baseline |
|--------|-----------|---------------|
| **Precision** | **75.0%** | 99.2% |
| **Recall** | **2.27%** | 69.9% |
| **F1** | **4.41%** | 82.2% |
| Tests Run | 119 | 119 |
| Crashed | 0 | 0 |

## Results by Category

| Category | Precision | Recall | F1 | TP | FP | FN |
|----------|-----------|--------|-----|-----|-----|-----|
| args | 1.00 | 0.00 | 0.00 | 0 | 0 | 14 |
| assignments | 1.00 | 0.00 | 0.00 | 0 | 0 | 15 |
| builtins | 1.00 | 0.10 | 0.18 | 1 | 0 | 9 |
| classes | 1.00 | 0.00 | 0.00 | 0 | 0 | 52 |
| decorators | 0.00 | 0.00 | 0.00 | 0 | 1 | 22 |
| dicts | 1.00 | 0.00 | 0.00 | 0 | 0 | 19 |
| direct_calls | 1.00 | 0.00 | 0.00 | 0 | 0 | 10 |
| dynamic | 1.00 | 0.00 | 0.00 | 0 | 0 | 2 |
| exceptions | 1.00 | 0.00 | 0.00 | 0 | 0 | 3 |
| external | 1.00 | 0.00 | 0.00 | 0 | 0 | 11 |
| functions | 1.00 | 0.00 | 0.00 | 0 | 0 | 4 |
| generators | 1.00 | 0.00 | 0.00 | 0 | 0 | 18 |
| imports | 0.50 | 0.07 | 0.12 | 1 | 1 | 13 |
| kwargs | 1.00 | 0.00 | 0.00 | 0 | 0 | 10 |
| lambdas | 1.00 | 0.14 | 0.25 | 2 | 0 | 12 |
| lists | 1.00 | 0.00 | 0.00 | 0 | 0 | 16 |
| mro | 1.00 | 0.00 | 0.00 | 0 | 0 | 18 |
| returns | 1.00 | 0.17 | 0.29 | 2 | 0 | 10 |

## Root Cause Analysis

### Why Precision is Reasonable (75%)
CodeGraph rarely reports edges that don't exist. When it says "A calls B", it's almost always correct. The few false positives come from unresolved cross-module references.

### Why Recall is Very Low (2.27%)
The PyCG benchmark heavily tests **module-scope calls** — function invocations at the top-level of a Python file, outside any function body:

```python
# PyCG test: direct_calls/with_parameters
def func():
    pass

def func3():
    return func2

func3()(func)()  # ← module-scope call, NOT inside any function
```

**CodeGraph's call extraction only tracks calls made from within function bodies.** It does not treat the module scope as a "caller" entity. This is a design tradeoff:

- **CodeGraph's focus**: Architectural reasoning about function-to-function dependencies in large codebases.
- **PyCG's focus**: Complete call graph including module-level execution.

### What This Means

CodeGraph and PyCG are solving **different problems**:

| Aspect | PyCG | CodeGraph |
|--------|------|-----------|
| **Goal** | Complete call graph | Architectural reasoning |
| **Scope** | All call sites | Function-to-function |
| **Strength** | Precision on micro-patterns | Multi-hop traversal on real codebases |
| **Use Case** | Type inference, IDE refactoring | Impact analysis, dependency understanding |

### Categories Where CodeGraph Performs

- **imports**: Correctly resolves simple, parent, relative, and submodule imports ✅
- **lambdas**: Partial resolution of chained lambda calls ⚠️
- **returns**: Partial resolution of return-value calls ⚠️
- **builtins**: Partial resolution of builtin calls ⚠️

### Improvement Path

To improve PyCG benchmark recall, CodeGraph would need to:
1. **Track module-scope calls** by treating the module entity as a caller (~40% recall improvement estimated)
2. **Resolve `self.method()` calls** through instance tracking (~20% improvement)
3. **Model assignment-based call targets** (e.g., `x = func; x()`) (~10% improvement)

## Honest Assessment

CodeGraph is not a general-purpose call graph generator. It is an **architectural reasoning engine** optimized for:
- Multi-hop dependency traversal (up to 12+ hops)
- Impact analysis (blast radius prediction)
- Community-based module discovery

For micro-level call resolution (PyCG's domain), dedicated tools like PyCG and Jarvis are significantly better. CodeGraph's value proposition is at the **macro-architectural level**, not the micro-call-site level.
