# LangChain

LangChain is a Python (and JavaScript) framework for building applications with language models. It provides abstractions for model calls, prompt templates, output parsers, memory, and tool integrations, plus a composition layer (LCEL — LangChain Expression Language) for chaining these components together.

## What It Does

LangChain's core value is an ecosystem of integrations: wrappers for dozens of model providers, vector stores, document loaders, and tools — all with a common interface. If you need to switch model providers, change your vector store, or add a new retrieval strategy, LangChain makes these swaps relatively low-friction.

### Key Components

- **Models**: unified interface for chat models, embeddings, and completions across providers (OpenAI, Anthropic, Cohere, local models, etc.).
- **Prompts**: template system for constructing prompts with variable substitution, few-shot examples, and message formatting.
- **Retrievers / Vector Stores**: integrations with Pinecone, Chroma, FAISS, Weaviate, etc., with a common retrieval interface.
- **Document Loaders**: parsers for PDFs, websites, databases, and more.
- **Tools**: pre-built integrations for search, calculators, code execution, APIs, and more.
- **LCEL**: a declarative composition syntax (using the `|` pipe operator) for chaining components into pipelines.

## Relationship to LangGraph

LangGraph (see [LangGraph](langgraph.md)) is LangChain's framework for stateful, graph-based agent orchestration. LangChain provides the components (models, tools, retrievers); LangGraph provides the execution model for complex multi-step agents. They are designed to be used together but are separate libraries.

## Criticisms and Tradeoffs

LangChain has been criticized for:
- **Abstraction overhead**: multiple layers of abstraction can make simple things complex and debugging difficult.
- **Rapidly changing APIs**: the library has undergone several major redesigns, making documentation and tutorials go stale quickly.
- **Over-engineering**: for simple use cases, direct API calls are often clearer.

For greenfield projects with straightforward requirements, using the model provider's SDK directly (plus a retrieval library of your choice) is often simpler. LangChain's value scales with the number of integrations you need and the complexity of your retrieval and composition logic.

## See Also

- [LangGraph](langgraph.md)
- [Memory](../concepts/memory.md)
- [Tool Use](../concepts/tool-use.md)
