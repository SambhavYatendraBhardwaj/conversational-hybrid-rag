🏛️ Conversational Hybrid RAG

A conversational Retrieval-Augmented Generation (RAG) assistant over a Medieval Indian History document (the Gurjara-Pratihara Empire). It combines dense semantic search, BM25 keyword search, and cross-encoder reranking, and it resolves follow-up questions using chat history so multi-turn conversations stay grounded in the source document.

 Why hybrid retrieval

Pure vector search often misses exact proper nouns and dynastic titles, while pure keyword search misses paraphrased questions. This project retrieves with both, merges the candidates, and lets a cross-encoder pick the most relevant passages.

 Key Features

- **Hybrid retrieval:** dense search (ChromaDB + `text-embedding-004`) and sparse BM25 keyword search, k=20 each.
- **Cross-encoder reranking:** merged and deduplicated candidates are re-scored with `ms-marco-MiniLM-L-6-v2`; the top 5 passages go to the LLM.
- **Conversational memory:** follow-up questions with pronouns ("his father", "their capital") are rewritten into standalone queries before retrieval.
- **Grounded answers:** the prompt instructs the model to answer only from the retrieved context.
- **Inspection UI:** Streamlit interface that shows reranker scores and the retrieved context blocks.

 Architecture

```
User Query + Chat History
          │
          ▼
[ Query Reformulation (Gemini Flash) ]  ← resolves pronouns / coreferences
          │
          ├──► [ Dense Retriever: ChromaDB + text-embedding-004 ]  (k=20)
          │
          └──► [ Sparse Retriever: BM25 keyword search ]           (k=20)
          │
          ▼
[ Deduplication & Candidate Merging ]
          │
          ▼
[ Cross-Encoder Reranker: ms-marco-MiniLM-L-6-v2 ]
          │
          ▼  (top 5 passages)
[ Prompt with Context ]
          │
          ▼
[ Gemini Flash: Answer Generation ]
          │
          ▼
[ Streamlit UI: answer + relevance scores + retrieved chunks ]
```

 Benchmarks

Measured locally; numbers vary with hardware and API latency.

| Component | Measurement |
|---|---|
| BM25 keyword lookup | ~0.3 ms |
| Cross-encoder reranking (`ms-marco-MiniLM-L-6-v2`) | ~879 ms average inference |
| Prompt size after reranking | 47.8% fewer downstream tokens sent to the LLM |
| End-to-end latency (query to answer) | ~7.7 s average |

 Tech Stack

| Layer | Tools |
|---|---|
| Orchestration | LangChain (LCEL) |
| LLM & embeddings | Google Gemini Flash, `text-embedding-004` |
| Dense store | ChromaDB |
| Sparse retrieval | Rank-BM25 |
| Reranker | Sentence-Transformers `cross-encoder/ms-marco-MiniLM-L-6-v2` |
| Frontend | Streamlit |

 Project Structure

```
├── ingestion.py                  # Document ingestion and indexing
├── frontend.py                   # Streamlit app (chat, retrieval, reranking, generation)
├── Gurjara_Pratihara_Empire.pdf  # Source document
├── requirements.txt
├── .gitignore
└── README.md
```

 Setup

```bash
git clone https://github.com/SambhavYatendraBhardwaj/conversational-hybrid-rag.git
cd conversational-hybrid-rag

python -m venv myenv
# Windows: myenv\Scripts\activate
# Linux / macOS: source myenv/bin/activate

pip install -r requirements.txt
```

Create a `.env` file in the project root (never commit it):

```
GOOGLE_API_KEY=your-gemini-api-key
```

 Run

```bash
streamlit run frontend.py
```

 Limitations

- Built around a single source document; multi-document support is not implemented.
- Answers are only as good as the retrieved passages; no automated retrieval evaluation yet.
- Requires a Google API key for generation and embeddings.

 Roadmap

- Retrieval evaluation (hit-rate / MRR) comparing dense-only, BM25-only, and hybrid
- Multi-document upload
- Source citations with page numbers in answers
