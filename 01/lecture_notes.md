# LLM

- a LLM is a neural network trained on massive amounts of text. Given a prompt, it generates a continuation - a plausible next piece of text.
- in this course, we don't go into technical details for LLM. There are trated as a black box.
- llm limitations
    - knowledge scoped to their trained data
    - no access to my data
    - hallucinations
- more information in https://github.com/DataTalksClub/llm-zoomcamp/blob/main/01-agentic-rag/lessons/01-intro.md

# RAG
- Retriaval-augmented generation
- RAG retrieves data at inference and hand it to the LLM, and the model generates a grounded response.

# RAG - dataset
In the RAG pipeline, this dataset is our knowledge base:

1. We index all the documents (the search step)
2. When a student asks a question, we search the index
3. The search returns the most relevant FAQ entries
4. We give those entries to the LLM as context
5. The LLM generates an answer based on the context

# RAG - search
For the search engine, we use a similarity function : it takes a query, score every document for similarity, and returns the top results
NB: We don't send all the dataset because it will be slow and it confuses the model

# RAG - building prompt
When we build AI systems, we usually split the prompt into two parts:

Instructions (also called the system prompt): this tells the LLM how to behave. It never changes, so it's the same for every request.
User prompt: this changes with every request. It carries the actual question and the retrieved context.

# The LLM
- https://github.com/DataTalksClub/llm-zoomcamp/blob/main/01-agentic-rag/lessons/07-llm.md

# data ingestion
- we use sqlite to load data and avoid re-indexing each time we start the python process.

With minsearch (single process):
```
Startup: fetch data -> parse -> index -> ready
Every restart: repeat all steps
```

With sqlitesearch (two processes):
```
Ingestion (runs once): fetch data -> parse -> write to faq.db
Query (runs every time): open faq.db -> search -> ready
```

SQLite is our knowledge base

:info: Pick the right tool for your data:
- minsearch: single process, in-memory only, re-indexes on every startup. Use when data is small and indexing is fast.
- sqlitesearch: separate ingestion and query, file-based (SQLite), opens existing index. Use when data is large or ingestion is slow.
Use minsearch when you can load and index the data on startup without noticeable delay. Switch to a persistent backend when ingestion takes too long or when you need the index to survive restarts.
For larger production systems, use the same pattern with a different backend:
- Elasticsearch
- OpenSearch
- Qdrant (vector database)
- Weaviate (vector database)

# agent

Before we didn't deal with an user making typos, unsual ways to ask questions, need information from two different searches

Instead of routing the user question straight to search, we can hand control to the LLM and let it drive.
The LLM is in charge now, and it can:
- fix typos
- search again with different terms
- ask the user a clarifying question

# function calling
- Instead of running search ourselves, we give the LLM a search tool. It decides when to call it and what to search for.
If there is a typo (cf. https://github.com/DataTalksClub/llm-zoomcamp/blob/main/01-agentic-rag/lessons/13-function-calling.md)

The difference is about who makes the decisions:
- With RAG, the developer decides. We fix the steps up front, so search always runs once with the exact user query.
- With an agent, the LLM decides. It chooses which actions to take and when to stop.

:info: NB: LLMs are stateless between API calls. That's why we need to send the chat history as input

Also about the search_tool : 
- The model doesn't see our Python code, only a schema describing what the function does and what arguments it takes.
- LLMs are language agnostic. At the end we're just making an HTTP call, so we describe the tool in JSON rather than in Python. The same schema would work from TypeScript or Java.
```
search_tool = {
    "type": "function",
    "name": "search",
    "description": "Search the FAQ database for entries matching the given query.",
    "parameters": {
        "type": "object",
        "properties": {
            "query": {
                "type": "string",
                "description": "Search query text to look up in the course FAQ."
            }
        },
        "required": ["query"],
        "additionalProperties": False
    }
}
```

# agentic loop
- Manual, single-turn function calling breaks down when an LLM needs to perform multiple or sequential searches to find an answer.
- To solve this, an agentic loop uses a while loop that keeps executing tools and feeding the results back to the model until no more function calls are requested.
- An agent consists of three core components: developer instructions (the role/behavior), tools (the executable functions), and memory (the conversation and execution history).
- System behavior, scope, and guardrails—such as forcing multiple searches or blocking off-topic questions—are primarily controlled by refining the developer instructions.
- This basic code pattern forms the underlying foundation of all major agent frameworks, including LangChain, PydanticAI, and the OpenAI Agents SDK.

# frameworks
- ToyAIKit is a minimal, educational library that wraps repetitive handwritten agent loops into a clean framework (and simple to understand)
- Automated schema generation: instead of manually writing JSON schemas, the library automatically derives them from a Python function's docstrings and type hints, mirroring the behavior of production frameworks like LangChain or PydanticAI. e.g "function": {"arguments": '{"query": "Ollama locally"}', "name": 'search'}
- By setting up a runner with tools, prompts, and a chat interface, ToyAIKit manages the while True loop, executes function calls, and handles multi-turn conversation memory seamlessly.
- The framework automatically tracks message history, token usage, and overall costs, sparing ux from calculating these metrics by hand during complex, multi-turn agent interactions.

# other frameworks
- https://github.com/DataTalksClub/llm-zoomcamp/blob/main/01-agentic-rag/lessons/16-other-frameworks.md