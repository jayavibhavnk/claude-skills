---
name: embeddings-rag
description: Build Retrieval-Augmented Generation (RAG) systems using embeddings for semantic search and knowledge bases.
metadata:
  priority: 9
  docs:
    - "https://python.langchain.com/docs/get_started/introduction"
    - "https://sdk.vercel.ai/docs/ai-sdk-core/embeddings"
  pathPatterns:
    - "**/embeddings/**"
    - "**/rag/**"
    - "**/vector/**"
  bashPatterns:
    - '\bembeddings\b'
    - '\brag\b'
    - '\bvector.?search\b'
  promptSignals:
    phrases:
      - "embeddings"
      - "rag"
      - "vector search"
    anyOf:
      - "semantic search"
      - "knowledge base"
      - "retrieval"
---

## Embeddings & RAG

### What are Embeddings?

Embeddings represent text/numbers/images as vectors in high-dimensional space. Similar items cluster together.

```typescript
import { embed } from 'ai';
import { openai } from '@ai-sdk/openai';

const { embedding } = await embed({
  model: 'openai/text-embedding-3-small',
  value: 'Hello, world!',
});

// embedding is a float32 array, e.g.:
// [0.123, -0.456, 0.789, ...]  (1536 dimensions for text-embedding-3-small)
```

### Cosine Similarity

```typescript
function cosineSimilarity(a: number[], b: number[]): number {
  const dot = a.reduce((sum, val, i) => sum + val * b[i], 0);
  const normA = Math.sqrt(a.reduce((sum, val) => sum + val * val, 0));
  const normB = Math.sqrt(b.reduce((sum, val) => sum + val * val, 0));
  return dot / (normA * normB);
}
```

### Vector Databases

| Provider | Best For | Dimensions |
|----------|----------|------------|
| Pinecone | Production scale | Up to 40k |
| Weaviate | Hybrid search | Up to 40k |
| Qdrant | High performance | 1536+ |
| Chroma | Local/simple | 512-2560 |
| Supabase | Built-in pgvector | 1536+ |

### Building a RAG System

```typescript
// 1. Index documents
async function indexDocuments(documents: string[]) {
  const embeddings = await Promise.all(
    documents.map(async (doc) => {
      const { embedding } = await embed({
        model: 'openai/text-embedding-3-small',
        value: doc,
      });
      return embedding;
    })
  );

  // Store in vector DB
  await vectorDB.upsert({
    ids: documents.map((_, i) => `doc-${i}`),
    embeddings,
    documents,
  });
}

// 2. Retrieve relevant context
async function retrieveContext(query: string, topK: number = 5) {
  const { embedding } = await embed({
    model: 'openai/text-embedding-3-small',
    value: query,
  });

  const results = await vectorDB.search({
    vector: embedding,
    topK,
  });

  return results.matches.map(m => m.document);
}

// 3. Generate with context
async function ragQuery(question: string) {
  const context = await retrieveContext(question);

  const { text } = await generateText({
    model: 'openai/gpt-5.4',
    prompt: `Context: ${context.join('\n\n')}

Question: ${question}

Answer based on the context provided.`,
  });

  return text;
}
```

### Chunking Strategies

```typescript
// Simple fixed-size chunks
function chunkText(text: string, chunkSize: number = 1000): string[] {
  const words = text.split(' ');
  const chunks: string[] = [];
  let currentChunk: string[] = [];

  for (const word of words) {
    currentChunk.push(word);
    if (currentChunk.join(' ').length >= chunkSize) {
      chunks.push(currentChunk.join(' '));
      currentChunk = [];
    }
  }

  if (currentChunk.length) chunks.push(currentChunk.join(' '));
  return chunks;
}

// Semantic chunking (by sentences)
function semanticChunk(text: string, maxChunkSize: number = 1000): string[] {
  const sentences = text.match(/[^.!?]+[.!?]+/g) || [text];
  const chunks: string[] = [];
  let currentChunk = '';

  for (const sentence of sentences) {
    if ((currentChunk + sentence).length > maxChunkSize) {
      if (currentChunk) chunks.push(currentChunk.trim());
      currentChunk = sentence;
    } else {
      currentChunk += sentence;
    }
  }

  if (currentChunk) chunks.push(currentChunk.trim());
  return chunks;
}
```

### Metadata Filtering

```typescript
// Add metadata during indexing
await vectorDB.upsert({
  embeddings,
  documents,
  metadata: [
    { source: 'docs', page: '1', date: '2024-01-01' },
    { source: 'docs', page: '2', date: '2024-01-02' },
    // ...
  ],
});

// Filter during search
const results = await vectorDB.search({
  vector: embedding,
  filter: { source: 'docs/page-1' },  // Only docs from page-1
  topK: 10,
});
```

### HyDE (Hypothetical Document Embeddings)

```typescript
// Instead of embedding the query directly,
// first generate a hypothetical answer
const { text: hypotheticalAnswer } = await generateText({
  model: 'openai/gpt-5.4',
  prompt: `Question: ${query}

Give a brief hypothetical answer to this question:`,
});

// Use the hypothetical answer for embedding
const { embedding } = await embed({
  model: 'openai/text-embedding-3-small',
  value: hypotheticalAnswer,
});

const results = await vectorDB.search({ vector: embedding, topK: 5 });
```

### Re-ranking

```typescript
// After initial retrieval, re-rank with a cross-encoder
constreranker = new CrossEncoder('cross-encoder/ms-marco');

const reranked = await reranker.rank(query, documents, {
  topK: 10,
});
```

### Evaluation

| Metric | What it measures |
|--------|------------------|
| Precision@K | Relevant docs in top K |
| Recall@K | Relevant docs retrieved in top K |
| MRR | Mean reciprocal rank |
| NDCG | Normalized discounted gain |

### Best Practices

1. **Chunk size** - 500-1000 tokens usually works well
2. **Overlap** - 10-20% overlap helps capture context
3. **Metadata** - Store source, date, category for filtering
4. **Hybrid search** - Combine vector + keyword search
5. **Re-ranking** - Use cross-encoders for better results
6. **Freshness** - Update embeddings when content changes
