# Review: "Grep-AST: Replacing Vector Search with One Library"

**Source:** https://habr.com/ru/companies/ecom_tech/articles/1005610/
**Author:** ecom.tech

---

## Summary

The article describes how ecom.tech engineers replaced a RAG (Retrieval-Augmented Generation) pipeline with **grep-ast** — a Python library that combines grep-style search with tree-sitter's Abstract Syntax Tree (AST) parsing — for automated extraction of REST API endpoints and Kafka message producers/consumers from multi-language repositories.

---

## Problem

The team needed to reverse-engineer API and Kafka documentation from codebases written in **Golang, Kotlin, Ruby, Elixir, and PHP**. The challenge: framework wrappers and DSLs make endpoint/handler detection structurally non-trivial.

---

## Approaches Tried (and Why They Failed)

| Approach | Problem |
|---|---|
| **Direct tree-sitter** | Framework abstractions look like plain functions; high false-positive rate |
| **RAG with gte-multilingual-base embeddings** | Only 85% accuracy; slow; hard to debug |
| **Regex + grep** | Brittle; breaks on comments, string literals, formatting variations |

---

## Solution: grep-ast

[grep-ast](https://github.com/paul-gauthier/grep-ast) — created by Paul Gauthier (author of Aider) — searches code using regex patterns but returns results **within their full AST context**: the enclosing function, class, and structural scope, rather than isolated matching lines.

### Key properties

- **Structural context:** Results include the full containing code block (function/class/module)
- **Multi-language:** Supports 30+ languages via tree-sitter grammars
- **Simple interface:** CLI mirrors `grep` usage — low learning curve
- **Interpretable output:** Search patterns and navigation decisions are fully logged and traceable

### Usage in an LLM agent

The library is wired up as a **tool call** for an LLM agent, enabling iterative, multi-step code exploration. The agent issues grep-ast queries, inspects structural results, and narrows its search — mimicking how a developer manually navigates a codebase.

---

## Core Insight

> Not every code-search task needs a vector store.

When the target is a **precise structural element** (endpoints, event producers, specific decorators), AST-aware search outperforms semantic embeddings on:

- **Accuracy** (exact structural matches vs. approximate similarity)
- **Speed** (no embedding inference or vector index)
- **Debuggability** (patterns are readable; results are deterministic)
- **Maintainability** (no vector index to refresh as code changes)

---

## Assessment

### Strengths

- Practical, well-motivated engineering decision — not chasing hype
- Clear comparison of alternatives with concrete failure modes
- Useful framing: grep-ast as an LLM tool call rather than one-shot retrieval
- Highlights an underappreciated library from the Aider ecosystem

### Weaknesses / Open Questions

- 85% RAG accuracy is cited but no equivalent precision/recall figure is given for grep-ast — makes the win hard to quantify
- No discussion of **false negatives**: patterns that structural search would miss (e.g., dynamically registered routes, metaprogramming-heavy Ruby code)
- The article is light on implementation detail — how the agent prompt is structured and how tool results feed back into reasoning would strengthen it
- Multi-language pattern management (different decorator/annotation conventions per language) is mentioned but not elaborated

### Verdict

A concise, pragmatic article. The key message — "match the retrieval method to the structural nature of the target" — is sound and underrepresented in LLM tooling discussions. Worth reading for teams building code-intelligence agents.

---

## Takeaways for Practitioners

1. Before reaching for embeddings + vector search, ask: **is the target structurally well-defined?** If yes, AST-based search is simpler, faster, and more reliable.
2. grep-ast is a practical drop-in for code-navigation tool calls in LLM agents.
3. Accuracy benchmarks should be reported for both baseline and proposed approaches — not just the baseline.
