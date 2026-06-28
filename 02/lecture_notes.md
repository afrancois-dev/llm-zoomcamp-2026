# Vector

## Vector search
- keyword search : drawback -> exact words
- vector search : text to meaningful things 
    - (e.g make  discover, course, join <-> found, program, enroll)

```
An embedding model produces these vectors. It's a neural network trained to capture meaning, so texts that mean similar things land on similar vectors. We measure how close two vectors are with a distance metric. The most common one is cosine similarity.
```

*Recap here : https://github.com/DataTalksClub/llm-zoomcamp/blob/main/02-vector-search/lessons/01-intro.md*

## Embeddings
- turn text into a vectors is called EMBEDDINGS
    - word embeddings
    - sentence embeddings

- question : 
    - how the embedding model knows that 2 words are similar -> BERT-style model based on context, statistics, etc...
    - how it deals with every language -> there is multi lingual version of BERT
    - what is the size on disk ? -> approximately 0.5 Go

Cosine similarity measures the angle between two vectors, ignoring their length:
 - 1.0 = same direction (similar)
 - 0.0 = perpendicular (unrelated)
 - -1.0 = opposite direction (opposite meaning)

 ## Embeddings dataset
 - we embedd question + response together
 - we encode the vectors by batch of 50 documents (question + response) for performance reasons

 ## Vector search
 - We embed the query, compute dot products against all documents, and return the highest-scoring ones.

 ## Vector Search with minsearch
cf. notebook

## RAG Vector
- same as the module 1 except we have to override search method in order to use RAGVector instead of the keyword search

## SQLitesearch vector search
- minsearch VectorSearch: in-memory (numpy), exact cosine similarity, must re-compute embeddings on startup, good for experiments and notebooks
- sqlitesearch VectorSearchIndex: persistent (SQLite .db file), ANN (LSH/IVF/HNSW) with exact rerank, can open an existing index, good for projects and persistence

## PGVector vector search
- pgvector is the PostgreSQL extension that makes this work. Install it and Postgres can do vector similarity search. On top of that you get the usual production features, like concurrent access, transactions, and large datasets.