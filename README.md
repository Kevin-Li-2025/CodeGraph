# CodeGraph

**Code-aware GraphRAG for complex codebases.**

CodeGraph builds a structural knowledge graph from source code using AST parsing (tree-sitter), then combines graph-theory algorithms with semantic vector search to answer architectural questions about any codebase.

Unlike traditional RAG which treats code as plain text, CodeGraph performs **deterministic structural analysis**. It navigates the actual dependency graph to provide provable answers about code relationships.

---

## 🚀 Key Features

- **AST-Driven Graph Construction**: Uses tree-sitter to extract exact call graphs, inheritance chains, and import relationships.
- **Logical Module Discovery**: Uses the **Leiden Algorithm** to detect communities of code that belong together, even across different directories.
- **Deep Impact Analysis**: Predict the blast radius of a change with categorized direct and transitive dependencies.
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
