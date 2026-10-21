# Day 20 - Vector RAG Knowledge Base & Clinical Guidelines Embeddings

## Date
17 July 2026

## Topics Covered
- Retrieval-Augmented Generation (RAG)
- Clinical guideline preparation
- ChromaDB vector database
- Dense vector embeddings
- Semantic similarity search
- BM25 keyword fallback
- Metadata-rich document retrieval
- Clinical evidence grounding

## Learning Summary

### 1. Clinical Guideline Knowledge Base

The main objective of Day 20 was to build a persistent knowledge base containing authoritative clinical guidelines that could later be used to support evidence-grounded clinical responses.

The initial evidence corpus included:

```text
KDIGO 2024 Clinical Practice Guideline
for Diabetes and Chronic Kidney Disease

ADA Standards of Medical Care

WHO Hypertension Protocols
```

These documents were stored under:

```text
backend/evidence/
```

A dedicated RAG utility was implemented in:

```text
services/rag.py
```

### 2. Document Chunking

Long medical guidelines cannot be directly inserted into a vector database as single documents. Therefore, the documents were divided into smaller overlapping chunks.

The implemented strategy used:

```text
Chunk size  : 500 characters
Stride      : 100 characters
```

The overlap helped preserve context when an important statement continued across two adjacent chunks.

A markdown-aware chunking approach was also implemented to avoid unnecessarily separating medical tables and important multi-condition information.

Each chunk was enriched with its guideline title and section heading before embedding.

### 3. ChromaDB Vector Store

ChromaDB was configured as the persistent local vector database.

```python id="l7f1u8"
class ChromaGuidelineProvider:
    def __init__(self, persist_path="./chroma_db"):
        self.client = chromadb.PersistentClient(
            path=persist_path
        )

        self.collection = self.client.get_or_create_collection(
            name="clinical_guidelines"
        )
```

The database used persistent disk storage through:

```text
chroma_db/
```

This allowed the indexed clinical knowledge to remain available between application runs.

### 4. Embedding Generation

Each guideline chunk was converted into a numerical vector using Google's embedding models.

The implementation used:

```text
gemini-embedding-001
text-embedding-004
```

The generated embeddings were 768-dimensional.

Additional metadata was stored with every vector:

```text
Source Document
Chapter
Page Number
Guideline Year
Target Conditions
```

This metadata made the retrieved evidence easier to trace back to its original clinical source.

### 5. Semantic Retrieval

When a clinical query was received, the same embedding process was applied to the query.

The vector database then searched for the most semantically relevant guideline sections.

```text
Clinical Query
      ↓
Query Embedding
      ↓
ChromaDB Similarity Search
      ↓
Top-K Relevant Chunks
      ↓
Documents + Metadata + Scores
```

The default retrieval configuration returned the top three relevant chunks.

Similarity scores were normalized into a range from `0.00` to `1.00`, making the relevance of retrieved evidence easier to interpret.

### 6. BM25 / Keyword Fallback

The embedding service could become unavailable because of network problems or API issues. To avoid completely losing search functionality, an offline keyword/BM25 retrieval fallback was implemented.

The overall approach was:

```text
                 Clinical Query
                       ↓
              Embedding Available?
                /             \
              Yes              No
               ↓                ↓
        Vector Search      BM25 / Keyword
               \                /
                \              /
                 Relevant Evidence
```

This provided continued knowledge retrieval even when the external embedding service was unavailable.

## Challenges Faced

- Large clinical guidelines were difficult to search efficiently as complete documents.
- Simple chunking could separate medical tables or important contraindication information.
- External embedding APIs could become temporarily unavailable.
- Retrieved clinical information needed to remain traceable to its original source.

## Engineering Solutions

- Divided guidelines into overlapping chunks.
- Implemented markdown-aware chunking to preserve tables and section context.
- Added source and page metadata to every vector.
- Used persistent ChromaDB storage for local access.
- Added BM25/keyword fallback retrieval for offline operation.

## Key Takeaways

- Learned the practical implementation of Retrieval-Augmented Generation.
- Understood how text is converted into embeddings for semantic search.
- Learned how vector databases such as ChromaDB store and retrieve embedded documents.
- Understood the importance of metadata for traceability in clinical applications.
- Learned why fallback retrieval is important for maintaining system availability.
- Gained practical experience building an evidence-grounded clinical knowledge base.

## Outcome

By the end of Day 20, a persistent ChromaDB knowledge base had been created and populated with selected clinical guidelines. The system could generate embeddings, perform semantic top-k retrieval, attach source metadata, and fall back to keyword-based retrieval when embedding services were unavailable.

This completed the foundation required for the grounded clinical chat assistant developed in the next phase.
