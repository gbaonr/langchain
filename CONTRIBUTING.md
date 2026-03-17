# Contributing to LangChain

Welcome to the LangChain monorepo! This guide helps you navigate the codebase
in a structured way — whether you are a newcomer who wants to understand how
everything fits together, or an experienced developer who wants to contribute.

## Recommended reading order for newcomers

Work through the phases below in order. Each phase builds on the previous one,
so you will gain a solid mental model before looking at implementation details.

---

### Phase 1 — Understand what LangChain is (15 min)

| File | What to focus on |
|------|-----------------|
| [`README.md`](./README.md) | Project mission, ecosystem overview, and key use-cases |
| [`libs/README.md`](./libs/README.md) | How the monorepo is split into packages and why |

After this phase you should be able to answer: *"What problem does LangChain solve
and which packages live in this repo?"*

---

### Phase 2 — Understand the monorepo architecture (20 min)

| File | What to focus on |
|------|-----------------|
| [`CLAUDE.md`](./CLAUDE.md) | Layer diagram, development tools (`uv`, `make`, `ruff`, `mypy`, `pytest`), code quality and commit standards |
| [`Makefile`](./Makefile) | Available developer commands (`make test`, `make lint`, `make format`) |
| [`libs/core/README.md`](./libs/core/README.md) | Role of `langchain-core` as the provider-agnostic base layer |

The monorepo has four conceptual layers:

```
langchain-core          ← base abstractions & interfaces (no third-party deps)
langchain               ← main package; pre-built agents, re-exports of core
langchain-classic       ← legacy chains; no new features added here
partners/               ← provider integrations (openai, anthropic, ollama …)
```

`text-splitters/`, `standard-tests/`, and `model-profiles/` are supporting
packages used across the layers above.

---

### Phase 3 — Study the core abstractions (45 min)

All other packages depend on `langchain-core`.  Read the files below in order:

```
libs/core/langchain_core/
```

| File / directory | Key concept |
|-----------------|-------------|
| `__init__.py` | Module-level docstring explains every interface the package exposes |
| `runnables/` | **LangChain Expression Language (LCEL)** — the universal composition protocol. Start with `runnables/__init__.py`, then `runnables/base.py` for `Runnable.invoke`, `stream`, `batch` |
| `language_models/` | `BaseChatModel` and `BaseLLM` — every model integration inherits from one of these |
| `prompts/` | `PromptTemplate`, `ChatPromptTemplate` — how inputs are formatted before being sent to a model |
| `messages/` | `HumanMessage`, `AIMessage`, `SystemMessage`, `ToolMessage` — the typed message protocol used everywhere |
| `output_parsers/` | How raw model output is parsed into structured Python objects |
| `tools/` | `BaseTool` and `@tool` decorator — the interface every tool must implement |
| `vectorstores/` | `VectorStore` — abstract interface for vector databases |
| `retrievers.py` | `BaseRetriever` — abstract interface for retrieving documents |
| `callbacks/` | Observability hooks that run during model calls, tool calls, and chain steps |

> **Tip:** focus on the class docstrings and the abstract methods first.
> You do not need to read every implementation line.

After this phase you should be able to answer: *"What are Runnables, and how
do prompts, models, output parsers, and tools connect together?"*

---

### Phase 4 — Explore the main `langchain` package (20 min)

The `langchain` package (`libs/langchain_v1/`) is built on top of `langchain-core`
and provides pre-built, production-ready components.

```
libs/langchain_v1/
├── langchain/
│   ├── __init__.py       ← public API surface; see what is exported
│   ├── agents/           ← pre-built agent architectures (ReAct, tool-calling …)
│   ├── chat_models/      ← thin re-exports + convenience constructors
│   ├── embeddings/       ← embedding model wrappers
│   ├── tools/            ← built-in tools (search, calculator …)
│   └── messages/         ← message helpers
└── pyproject.toml        ← package metadata and dependencies
```

| File | What to focus on |
|------|-----------------|
| [`libs/langchain_v1/README.md`](./libs/langchain_v1/README.md) | When to use `langchain` vs `langgraph` |
| `libs/langchain_v1/langchain/__init__.py` | Which symbols are part of the public API |
| `libs/langchain_v1/langchain/agents/` | How pre-built agents are structured |

---

### Phase 5 — Understand a partner integration (20 min)

Partner packages show how a third-party provider implements the `langchain-core`
interfaces.  The OpenAI integration is a good first read:

```
libs/partners/openai/
├── langchain_openai/
│   ├── __init__.py          ← exported classes
│   ├── chat_models.py       ← ChatOpenAI extends BaseChatModel
│   └── embeddings/          ← OpenAIEmbeddings extends Embeddings
└── tests/
    ├── unit_tests/
    └── integration_tests/
```

| File | What to focus on |
|------|-----------------|
| [`libs/partners/openai/README.md`](./libs/partners/openai/README.md) | Quick-start usage |
| `libs/partners/openai/langchain_openai/chat_models.py` | How `BaseChatModel` is implemented concretely |
| `libs/partners/openai/tests/unit_tests/` | Unit-test patterns used across all partner packages |

After this phase you should be able to answer: *"How do I create my own provider
integration by implementing the core interfaces?"*

---

### Phase 6 — Study the testing infrastructure (15 min)

`langchain-tests` (`libs/standard-tests/`) provides shared test suites that
every partner integration should pass.

| File | What to focus on |
|------|-----------------|
| [`libs/standard-tests/README.md`](./libs/standard-tests/README.md) | How to inherit `ChatModelUnitTests` / `ChatModelIntegrationTests` |
| `libs/standard-tests/langchain_tests/` | The actual standard test implementations |

Run the tests for a single package with:

```bash
cd libs/partners/openai
uv sync --all-groups
make test          # unit tests only
make integration_tests  # requires OPENAI_API_KEY
```

---

### Phase 7 — Development workflow (10 min)

Once you understand the code, read these files before opening a pull request:

| File | What to focus on |
|------|-----------------|
| [`CLAUDE.md`](./CLAUDE.md) | Code quality rules, docstring format, type-hint requirements, PR guidelines |
| [`.github/PULL_REQUEST_TEMPLATE.md`](./.github/PULL_REQUEST_TEMPLATE.md) | Checklist every PR must satisfy |
| [`.github/workflows/pr_lint.yml`](./.github/workflows/pr_lint.yml) | Allowed commit types and scopes |

---

## Setting up your development environment

```bash
# 1. Install uv (fast Python package manager)
pip install uv

# 2. Navigate to the package you want to work on
cd libs/core          # or libs/langchain_v1, libs/partners/openai, etc.

# 3. Install all dependency groups (test, lint, typing, …)
uv sync --all-groups

# 4. Run the checks
make format   # auto-format with ruff
make lint     # lint + type-check with ruff + mypy
make test     # run unit tests with pytest
```

## Creating your own project using LangChain

After reading through the codebase you will know that a typical LangChain
application:

1. **Picks a model** — install a partner package (`pip install langchain-openai`)
   and instantiate `ChatOpenAI` (which extends `BaseChatModel`).
2. **Builds a prompt** — use `ChatPromptTemplate.from_messages(...)`.
3. **Chains components with LCEL** — connect them with the pipe operator: `prompt | model | output_parser`.
4. **Adds tools** — decorate a Python function with `@tool` and pass it to
   `model.bind_tools(tools)`.
5. **Uses a pre-built agent** — call `create_tool_calling_agent` from
   `langchain.agents` and wrap it in `AgentExecutor`.
6. **Observes** — enable LangSmith tracing by setting `LANGCHAIN_TRACING_V2=true`.

For a minimal working example see the [LangChain documentation](https://docs.langchain.com).

## Useful links

| Resource | URL |
|----------|-----|
| Documentation | https://docs.langchain.com |
| API Reference | https://reference.langchain.com/python |
| Contributing Guide | https://docs.langchain.com/oss/python/contributing/overview |
| LangGraph (advanced agents) | https://langchain-ai.github.io/langgraph/ |
| LangSmith (observability) | https://docs.smith.langchain.com |
