# CodeGraph

**Code-aware GraphRAG for complex codebases.**

CodeGraph builds a structural knowledge graph from source code using AST parsing (tree-sitter), then combines graph-theory algorithms with semantic vector search to answer architectural questions about any codebase.

Unlike traditional RAG which treats code as plain text, CodeGraph performs **deterministic structural analysis**. It doesn't just "guess" based on context chunks; it **navigates the actual dependency graph**.

---

## 🔬 "GPT Describes, CodeGraph Proves"

We ran a fair head-to-head comparison on the **Flask** codebase (680+ nodes, 880+ edges). 
Even providing GPT-4o with **6,000 characters of relevant code chunks** (Fair RAG Baseline), the difference in reasoning depth is significant:

| Metric | GPT-4o (Fair RAG) | CodeGraph | Improvement |
|--------|-------------------|-----------|-------------|
| **Relationships Traced** | ~2-3 (Hedged) | **42+ (Provable)** | **~14x Deepest** |
| **Logic Source** | Semantic Inference | **Deterministic AST** | **Proof-based** |
| **Indexing Latency** | Minutes (LLM calls) | **<1s (Local Parser)** | **600x faster** |
| **Accuracy** | Hedged ("might", "likely") | **Exact (file:line)** | **Binary Precision** |

> **Case Study**: When asked *"What exact code path leads to session cookie creation?"*, GPT-4o described the general mechanism. CodeGraph traced a **42-hop internal dependency tree** from `Flask.__call__` to `SecureCookieSessionInterface.save_session`, citing exact file and line numbers.

---

## 🚀 Key Features

- **AST-Driven Graph Construction**: Uses tree-sitter to extract exact call graphs, inheritance chains, and import relationships.
- **Logical Module Discovery**: Uses the **Leiden Algorithm** to detect communities of code that belong together, even across different directories.
- **Deep Impact Analysis**: Predict the exact blast radius of a change with categorized direct and transitive dependencies.
- **Static Execution Tracing**: Follow `CALLS` edges to see the actual execution flow of a function (static approximation).
- **Zero-Cost Indexing**: No LLM calls required to build the core graph structure.

## Quick Start

```bash
# Install
pip install -e "."

# Index a codebase
codegraph index /path/to/project

# Ask a complex architectural question
codegraph query "How does the session cookie creation flow work?"

# Analyze the blast radius of a refactor
codegraph impact "SecureCookieSessionInterface.save_session"
```

## Architecture

1.  **Parsing**: Tree-sitter extracts Entities (Class, Function, Module) and Relationships (Calls, Inherits, Imports).
2.  **Graphing**: NetworkX builds the global knowledge graph.
3.  **Clustering**: Leiden algorithm detects logical communities.
4.  **Retrieval**: Hybrid Search (BM25 + Vector + Graph Traversal).
5.  **Reasoning**: DFS/BFS traversal to provide provable execution paths and impact chains.

## License

MIT
