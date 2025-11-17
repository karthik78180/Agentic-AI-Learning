# Day 11: RAG (Retrieval Augmented Generation) Architecture

## RAG Complete Pipeline

```mermaid
graph TB
    subgraph "Ingestion Pipeline"
        DOCS[Documents] --> LOAD[Document Loader]
        LOAD --> CHUNK[Text Chunking]
        CHUNK --> EMBED[Create Embeddings]
        EMBED --> STORE[Vector Database]
    end

    subgraph "Query Pipeline"
        QUERY[User Query] --> QEMBED[Query Embedding]
        QEMBED --> SEARCH[Similarity Search]
        STORE --> SEARCH
        SEARCH --> RETRIEVE[Top-K Documents]
        RETRIEVE --> RERANK[Re-ranking]
        RERANK --> CONTEXT[Relevant Context]
    end

    subgraph "Generation Pipeline"
        CONTEXT --> PROMPT[Augmented Prompt]
        QUERY --> PROMPT
        PROMPT --> LLM[LLM Generation]
        LLM --> RESPONSE[Final Response]
    end

    style EMBED fill:#4ecdc4
    style SEARCH fill:#45b7d1
    style LLM fill:#f9ca24
    style RESPONSE fill:#7ed321
```

## RAG Workflow Detailed

```mermaid
sequenceDiagram
    participant User
    participant Agent
    participant Embedder
    participant VectorDB
    participant LLM

    Note over VectorDB: Pre-populated with<br/>embedded documents

    User->>Agent: "What is RAG?"
    Agent->>Embedder: Embed query
    Embedder->>Agent: Query vector
    Agent->>VectorDB: Similarity search
    VectorDB->>Agent: Top 3 relevant chunks

    Note over Agent: Retrieved:<br/>1. RAG definition<br/>2. RAG benefits<br/>3. RAG example

    Agent->>Agent: Build augmented prompt
    Agent->>LLM: Prompt + Context
    LLM->>Agent: Generated response
    Agent->>User: Final answer with citations
```

## Document Chunking Strategies

```mermaid
graph TB
    DOC[Long Document] --> STRATEGY{Chunking<br/>Strategy}

    STRATEGY -->|Fixed Size| FIXED[Split every<br/>500 tokens]
    STRATEGY -->|Semantic| SEMANTIC[Split by<br/>paragraphs/sections]
    STRATEGY -->|Recursive| RECURSIVE[Split recursively<br/>until size met]
    STRATEGY -->|Overlap| OVERLAP[Sliding window<br/>with overlap]

    FIXED --> CHUNKS1[Chunk 1<br/>Chunk 2<br/>Chunk 3]
    SEMANTIC --> CHUNKS2[Intro<br/>Method<br/>Results]
    RECURSIVE --> CHUNKS3[Section → Para<br/>→ Sentence]
    OVERLAP --> CHUNKS4[Chunk 1: [0-500]<br/>Chunk 2: [400-900]<br/>Chunk 3: [800-1300]]

    style STRATEGY fill:#4a90e2
    style CHUNKS1 fill:#4ecdc4
    style CHUNKS2 fill:#45b7d1
    style CHUNKS3 fill:#f9ca24
    style CHUNKS4 fill:#6c5ce7
```

**Chunk Metadata Structure:**

```mermaid
classDiagram
    class DocumentChunk {
        +String id
        +String content
        +float[] embedding
        +ChunkMetadata metadata
    }

    class ChunkMetadata {
        +String source_document
        +int chunk_index
        +int start_char
        +int end_char
        +dict custom_fields
    }

    class Document {
        +String filename
        +String content
        +datetime created_at
        +List~DocumentChunk~ chunks
    }

    Document --> DocumentChunk
    DocumentChunk --> ChunkMetadata
```

## Vector Search Architecture

```mermaid
graph LR
    subgraph "Search Process"
        Q[Query:<br/>"Python web frameworks"] --> E[Embedding<br/>Model]
        E --> V[Query Vector<br/>[0.12, -0.34, ...]]

        V --> INDEX[Vector Index<br/>HNSW/IVF]

        INDEX --> S1[Similarity:<br/>Django chunk<br/>0.92]
        INDEX --> S2[Similarity:<br/>Flask chunk<br/>0.89]
        INDEX --> S3[Similarity:<br/>FastAPI chunk<br/>0.87]

        S1 --> RESULTS[Top-K<br/>Results]
        S2 --> RESULTS
        S3 --> RESULTS
    end

    style E fill:#4ecdc4
    style INDEX fill:#45b7d1
    style RESULTS fill:#7ed321
```

## Embedding Models Comparison

```mermaid
graph TB
    subgraph "Embedding Options"
        OPT1[OpenAI<br/>text-embedding-3-small<br/>1536 dims]
        OPT2[Sentence Transformers<br/>all-MiniLM-L6-v2<br/>384 dims]
        OPT3[Cohere<br/>embed-english-v3.0<br/>1024 dims]
        OPT4[Custom Fine-tuned<br/>Domain-specific<br/>Variable dims]
    end

    OPT1 -.->|Quality: ⭐⭐⭐⭐⭐| Q1
    OPT1 -.->|Cost: $$$| C1
    OPT1 -.->|Speed: ⚡⚡| S1

    OPT2 -.->|Quality: ⭐⭐⭐⭐| Q2
    OPT2 -.->|Cost: Free| C2
    OPT2 -.->|Speed: ⚡⚡⚡| S2

    OPT3 -.->|Quality: ⭐⭐⭐⭐⭐| Q3
    OPT3 -.->|Cost: $$| C3
    OPT3 -.->|Speed: ⚡⚡⚡| S3

    OPT4 -.->|Quality: ⭐⭐⭐⭐⭐| Q4
    OPT4 -.->|Cost: $$$$| C4
    OPT4 -.->|Speed: ⚡| S4
```

## Hybrid Search Architecture

```mermaid
graph TB
    QUERY[User Query] --> SPLIT{Search<br/>Type}

    SPLIT --> VECTOR[Vector Search<br/>Semantic similarity]
    SPLIT --> KEYWORD[Keyword Search<br/>BM25/TF-IDF]

    VECTOR --> VRESULTS[Semantic<br/>Results]
    KEYWORD --> KRESULTS[Keyword<br/>Results]

    VRESULTS --> FUSION[Result Fusion<br/>Reciprocal Rank]
    KRESULTS --> FUSION

    FUSION --> FINAL[Combined<br/>Top-K Results]

    style VECTOR fill:#4ecdc4
    style KEYWORD fill:#45b7d1
    style FUSION fill:#f9ca24
    style FINAL fill:#7ed321
```

## Re-ranking Pipeline

```mermaid
sequenceDiagram
    participant Search
    participant Retriever
    participant Reranker
    participant LLM

    Search->>Retriever: Top 20 chunks
    Retriever->>Reranker: Coarse results
    Note over Reranker: Cross-encoder<br/>or LLM-based reranking
    Reranker->>Reranker: Score each chunk
    Reranker->>Retriever: Top 3 refined
    Retriever->>LLM: Best chunks only
```

## RAG with Agent Integration

```mermaid
graph TB
    USER[User Query] --> AGENT[Agent Controller]

    AGENT --> DECIDE{Need<br/>Knowledge?}

    DECIDE -->|Yes| RAG[RAG System]
    DECIDE -->|No| DIRECT[Direct LLM]

    RAG --> R1[Retrieve Documents]
    R1 --> R2[Build Context]
    R2 --> LLM1[LLM + Context]

    DIRECT --> LLM2[LLM Only]

    LLM1 --> RESPONSE[Response]
    LLM2 --> RESPONSE

    RESPONSE --> CITE[Add Citations]
    CITE --> USER

    style AGENT fill:#4a90e2
    style RAG fill:#4ecdc4
    style CITE fill:#f9ca24
```

## Advanced RAG: Query Transformation

```mermaid
graph LR
    Q[Original Query:<br/>"What are the latest AI trends?"] --> T{Transform}

    T --> T1[Decompose:<br/>"AI trends in 2024"<br/>"Recent AI breakthroughs"<br/>"Emerging AI tech"]

    T --> T2[Expand:<br/>Add synonyms<br/>Add related terms]

    T --> T3[Rewrite:<br/>"Recent developments<br/>in artificial intelligence"]

    T1 --> SEARCH[Multiple Searches]
    T2 --> SEARCH
    T3 --> SEARCH

    SEARCH --> COMBINE[Combine Results]

    style T fill:#4a90e2
    style COMBINE fill:#7ed321
```

## RAG Performance Optimization

```mermaid
graph TB
    PERF[Performance Optimization]

    PERF --> P1[Caching<br/>Cache embeddings<br/>Cache results]
    PERF --> P2[Indexing<br/>HNSW, IVF<br/>Approximate NN]
    PERF --> P3[Batching<br/>Batch queries<br/>Batch embeddings]
    PERF --> P4[Pruning<br/>Filter metadata<br/>Pre-filter results]
    PERF --> P5[Async<br/>Parallel retrieval<br/>Non-blocking]

    P1 --> FASTER[Faster<br/>Retrieval]
    P2 --> FASTER
    P3 --> FASTER
    P4 --> FASTER
    P5 --> FASTER

    style PERF fill:#4a90e2
    style FASTER fill:#7ed321
```

## RAG Evaluation Metrics

```mermaid
graph LR
    subgraph "Retrieval Metrics"
        R1[Recall@K:<br/>Relevant docs<br/>in top K]
        R2[Precision@K:<br/>% relevant<br/>in top K]
        R3[MRR:<br/>Mean reciprocal<br/>rank]
    end

    subgraph "Generation Metrics"
        G1[Faithfulness:<br/>Response grounded<br/>in context?]
        G2[Relevance:<br/>Answers the<br/>question?]
        G3[Citation Accuracy:<br/>Correct source<br/>attribution?]
    end

    R1 --> EVAL[Overall<br/>RAG Quality]
    R2 --> EVAL
    R3 --> EVAL
    G1 --> EVAL
    G2 --> EVAL
    G3 --> EVAL

    style EVAL fill:#7ed321
```

## Multi-Document RAG

```mermaid
graph TB
    Q[Query: Compare<br/>React vs Vue] --> RAG1[RAG System 1:<br/>React Docs]
    Q --> RAG2[RAG System 2:<br/>Vue Docs]

    RAG1 --> C1[React Context]
    RAG2 --> C2[Vue Context]

    C1 --> LLM[LLM Synthesis]
    C2 --> LLM

    LLM --> COMPARE[Comparative<br/>Analysis]

    style RAG1 fill:#4ecdc4
    style RAG2 fill:#45b7d1
    style LLM fill:#f9ca24
    style COMPARE fill:#7ed321
```

## RAG Error Handling

```mermaid
stateDiagram-v2
    [*] --> Query
    Query --> Embed: Create embedding
    Embed --> Search: Vector search
    Search --> CheckResults: Evaluate results

    CheckResults --> Generate: Good results
    CheckResults --> Fallback: No/poor results

    Fallback --> Broaden: Expand query
    Fallback --> Rephrase: Rewrite query
    Fallback --> Default: Use base knowledge

    Broaden --> Search
    Rephrase --> Search
    Default --> Generate

    Generate --> Validate: Check quality
    Validate --> Return: Valid
    Validate --> Retry: Invalid
    Retry --> Query

    Return --> [*]
```

## Related Topics

- **Day 9**: Memory systems (semantic memory overlap)
- **Day 12**: Vector databases (storage backend)
- **Day 14**: Research assistant project (RAG application)
- **Day 18**: LangChain (RAG implementation)

## RAG Architecture Patterns

```mermaid
graph TB
    subgraph "Basic RAG"
        B1[Query] --> B2[Retrieve] --> B3[Generate]
    end

    subgraph "Iterative RAG"
        I1[Query] --> I2[Retrieve]
        I2 --> I3[Generate]
        I3 --> I4{Need more?}
        I4 -->|Yes| I2
        I4 -->|No| I5[Final]
    end

    subgraph "Agentic RAG"
        A1[Query] --> A2[Agent Decides]
        A2 --> A3[Retrieve 1]
        A2 --> A4[Retrieve 2]
        A2 --> A5[Process]
        A3 --> A6[Synthesize]
        A4 --> A6
        A5 --> A6
    end

    style B2 fill:#4ecdc4
    style I2 fill:#45b7d1
    style A2 fill:#f9ca24
```

---

**Related Days**: [Day 9](./day-09-memory-systems.md) | [Day 12](./day-12-vector-databases.md) | [Day 18](./day-18-langchain.md)
