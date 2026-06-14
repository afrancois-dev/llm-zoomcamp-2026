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