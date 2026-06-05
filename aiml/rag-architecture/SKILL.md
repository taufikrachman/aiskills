# RAG — Retrieval-Augmented Generation

Build RAG systems: semantic search over documents, then generate answers.

## Rules

### 1. Document Processing Pipeline
```
Raw docs → Parse → Chunk → Embed → Store → Retrieve → Generate
```
- Parse: extract text from PDF, HTML, Markdown, JSON.
- Chunk size: 500-1000 tokens. Overlap: 10-20% for context continuity.
- Embed: use `text-embedding-3-small` (OpenAI, cheap) or `bge-large` (open-source).
- Store: vector DB (pgvector, Pinecone, Weaviate) + metadata store.

### 2. Chunking Strategy
```ts
// Semantic chunking — split at paragraph boundaries
function chunkDocument(text: string, maxTokens = 500): Chunk[] {
  const paragraphs = text.split('\n\n');
  const chunks: Chunk[] = [];
  let current = '';
  for (const p of paragraphs) {
    if (tokenCount(current + p) > maxTokens) {
      chunks.push({ text: current, metadata: {} });
      current = p;
    } else {
      current += '\n\n' + p;
    }
  }
  chunks.push({ text: current, metadata: {} });
  return chunks;
}
```
Preserve heading context: each chunk stores its section title as metadata.

### 3. Vector Database Choice
| DB | Best For | Notes |
|----|----------|-------|
| **pgvector** | PostgreSQL users, < 1M docs | Free, no extra infra |
| **Pinecone** | Production, > 1M docs | Managed, fast, $ |
| **Weaviate** | Self-hosted, hybrid search | Open source |
| **Qdrant** | Rust-based, performance | Open source |

### 4. Hybrid Search
```ts
// Combine vector similarity + keyword search for better results
async function search(query: string, topK = 5): Promise<Document[]> {
  const queryEmbedding = await embed(query);
  const vectorResults = await db.query(`
    SELECT *, 1 - (embedding <=> $1) as similarity
    FROM documents
    ORDER BY embedding <=> $1 LIMIT $2
  `, [queryEmbedding, topK * 2]);

  const keywordResults = await db.query(`
    SELECT *, ts_rank(search_vector, plainto_tsquery('english', $1)) as rank
    FROM documents
    WHERE search_vector @@ plainto_tsquery('english', $1)
    ORDER BY rank DESC LIMIT $2
  `, [query, topK]);

  return mergeAndRerank(vectorResults, keywordResults, topK);
}
```

### 5. Retrieval Patterns
- Simple: embed query → find top-k similar → inject into prompt.
- Multi-hop: retrieve → generate → extract new query → retrieve more.
- Self-query: LLM extracts filters from user question → structured filter + semantic search.
- Time-weighted: boost recent documents for news/updates.

### 6. Prompt Template
```ts
const prompt = `Answer based on the following context. If not in context, say "I don't know."

Context:
${chunks.map((c, i) => `[${i + 1}] ${c.text}`).join('\n\n')}

Question: ${userQuery}

Answer:`;
```
- Cap context at 3000-4000 tokens.
- Include source references: `[1]`, `[2]` in answer.
- Guardrail: if confidence < threshold, say "I could not find a reliable answer."

### 7. Evaluation
- Hit rate: percentage of queries where relevant doc is in top-k.
- MRR (Mean Reciprocal Rank): position of first relevant result.
- Faithfulness: generated answer matches retrieved docs (no hallucination).
- Test with 50-100 real user queries + human-annotated relevance.

## Anti-Patterns
- ❌ No overlap between chunks (loses context)
- ❌ Embedding the entire document as one chunk
- ❌ Using raw cosine similarity without hybrid search
- ❌ No source attribution in answers
