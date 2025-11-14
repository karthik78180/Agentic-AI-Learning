# Day 12: Vector Databases & Embeddings

## Overview
Vector databases are the backbone of semantic search and RAG systems. Today you'll master embeddings and learn to use production vector databases.

## Learning Objectives
- Understand embeddings and vector similarity
- Work with production vector databases
- Implement efficient indexing strategies
- Optimize search performance
- Build scalable retrieval systems
- Compare vector database options

## Key Concepts

### Embeddings
- Numerical representations of text
- Capture semantic meaning
- Enable similarity search
- Models: OpenAI, Cohere, open-source

### Vector Databases
- **Pinecone**: Managed, easy to use
- **Weaviate**: Open-source, GraphQL
- **Qdrant**: Rust-based, fast
- **Chroma**: Embedded, simple
- **Milvus**: Scalable, enterprise

## Hands-On: Chroma DB Example

```python
# vector_db_example.py
import chromadb
from chromadb.utils import embedding_functions

# Initialize
client = chromadb.Client()
openai_ef = embedding_functions.OpenAIEmbeddingFunction(
    api_key="your-key",
    model_name="text-embedding-3-small"
)

# Create collection
collection = client.create_collection(
    name="knowledge_base",
    embedding_function=openai_ef
)

# Add documents
collection.add(
    documents=["Python is great", "FastAPI is fast", "React is popular"],
    ids=["doc1", "doc2", "doc3"]
)

# Query
results = collection.query(
    query_texts=["programming languages"],
    n_results=2
)

print(results)
```

## Resources
- [Chroma Documentation](https://docs.trychroma.com/)
- [Pinecone Learning Center](https://www.pinecone.io/learn/)
- [Embeddings Guide](https://platform.openai.com/docs/guides/embeddings)

## Daily Challenge
Build a semantic code search tool for your codebase using vector embeddings.

---

**Progress**: 12/30 days completed

[← Previous: Day 11](./day-11.md) | [Back to Overview](../README.md) | [Next: Day 13 →](./day-13.md)
