# Day 11: RAG - Retrieval Augmented Generation

## Overview
RAG (Retrieval Augmented Generation) enables agents to access and reason over large knowledge bases. It's essential for building agents that work with documents, databases, and custom knowledge.

## Learning Objectives
- Understand RAG architecture and workflow
- Implement document chunking strategies
- Create retrieval pipelines
- Combine retrieval with generation
- Optimize retrieval quality
- Build RAG-powered agents

## Theoretical Concepts

### What is RAG?

RAG combines:
1. **Retrieval**: Find relevant information from a knowledge base
2. **Augmentation**: Add retrieved info to the prompt
3. **Generation**: LLM generates response using the context

```
User Query
    ↓
Embed Query → Search Vector DB → Retrieve Top-K Docs
    ↓
Augment Prompt with Retrieved Docs
    ↓
LLM Generation
    ↓
Response
```

### Why RAG?

- **Up-to-date Information**: Access current data beyond training cutoff
- **Domain Knowledge**: Incorporate specialized knowledge
- **Reduced Hallucinations**: Ground responses in facts
- **Citations**: Provide sources for claims
- **Cost-Effective**: Cheaper than fine-tuning

### RAG Pipeline Components

1. **Document Loading**: Import documents
2. **Chunking**: Split into manageable pieces
3. **Embedding**: Convert to vectors
4. **Indexing**: Store in vector database
5. **Retrieval**: Find relevant chunks
6. **Generation**: Create response

## Hands-On Exercises

### Exercise 1: Basic RAG Implementation

```python
# basic_rag.py
from openai import OpenAI
from typing import List, Dict
import numpy as np

client = OpenAI()

class SimpleRAG:
    """Basic RAG implementation"""

    def __init__(self):
        self.documents: List[str] = []
        self.embeddings: List[np.ndarray] = []

    def add_document(self, text: str):
        """Add a document to the knowledge base"""
        self.documents.append(text)

        # Create embedding
        response = client.embeddings.create(
            model="text-embedding-3-small",
            input=text
        )
        embedding = np.array(response.data[0].embedding)
        self.embeddings.append(embedding)

    def retrieve(self, query: str, top_k: int = 3) -> List[str]:
        """Retrieve relevant documents"""
        # Embed query
        response = client.embeddings.create(
            model="text-embedding-3-small",
            input=query
        )
        query_embedding = np.array(response.data[0].embedding)

        # Calculate similarities
        similarities = []
        for doc_emb in self.embeddings:
            similarity = np.dot(query_embedding, doc_emb) / (
                np.linalg.norm(query_embedding) * np.linalg.norm(doc_emb)
            )
            similarities.append(similarity)

        # Get top-k
        top_indices = np.argsort(similarities)[-top_k:][::-1]
        return [self.documents[i] for i in top_indices]

    def query(self, question: str) -> str:
        """Query with RAG"""
        # Retrieve relevant docs
        relevant_docs = self.retrieve(question, top_k=3)

        # Create augmented prompt
        context = "\n\n".join(relevant_docs)
        prompt = f"""Answer the question based on the following context:

Context:
{context}

Question: {question}

Answer based on the context above. If the answer is not in the context, say so.
"""

        # Generate response
        response = client.chat.completions.create(
            model="gpt-4-turbo-preview",
            messages=[{"role": "user", "content": prompt}],
            temperature=0.3
        )

        return response.choices[0].message.content

# Usage
rag = SimpleRAG()

# Add documents
rag.add_document("Python was created by Guido van Rossum and released in 1991.")
rag.add_document("Python is known for its simple syntax and readability.")
rag.add_document("FastAPI is a modern Python web framework released in 2018.")
rag.add_document("FastAPI is built on top of Starlette and Pydantic.")
rag.add_document("React is a JavaScript library for building user interfaces.")

# Query
answer = rag.query("When was Python created?")
print(f"Answer: {answer}")

answer2 = rag.query("What is FastAPI built on?")
print(f"\nAnswer: {answer2}")
```

See full Day 11 implementation guide with document chunking, vector databases, and advanced RAG patterns in the course materials.

## Key Concepts

### Document Chunking Strategies

1. **Fixed Size**: Split by character/token count
2. **Semantic**: Split by paragraphs/sections
3. **Recursive**: Split recursively until size met
4. **Overlapping**: Add overlap between chunks

### Retrieval Optimization

- Use hybrid search (keyword + semantic)
- Implement re-ranking
- Add metadata filtering
- Use query expansion
- Optimize chunk size

## Resources

- [LangChain RAG Guide](https://python.langchain.com/docs/use_cases/question_answering/)
- [Pinecone RAG Course](https://www.pinecone.io/learn/retrieval-augmented-generation/)
- [OpenAI RAG Best Practices](https://platform.openai.com/docs/guides/embeddings)

## Daily Challenge

Build a documentation RAG agent:
1. Load your project's documentation
2. Chunk and embed it
3. Create a Q&A interface
4. Add citation/source tracking
5. Implement conversation memory

---

**Progress**: 11/30 days completed

[← Previous: Day 10](./day-10.md) | [Back to Overview](../README.md) | [Next: Day 12 →](./day-12.md)
