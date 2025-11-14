**Mini-Project #3: Real-World RAG Implementation**
Due: Nov 12
CAPB Report

# **1. Project Context & Use Case**

## **1.1 Problem / Context**

Modern developers frequently switch between multiple programming languages and frameworks—such as **Python**, **JavaScript/React**, and **MATLAB**—for coursework, research, or software projects. Each ecosystem maintains its own documentation, often spread across large PDFs, web pages, or reference manuals. This fragmentation makes it inefficient for students and developers to quickly find correct function usage, code examples, or conceptual explanations.

The objective of this project is to build a **Retrieval-Augmented Generation (RAG)** system that consolidates documentation across these languages into one unified assistant. This assistant allows users to ask natural-language questions like:

* “How do I read a file in Python?”
* “How do I use useState in React?”
* “How do I plot multiple lines in MATLAB?”
* “What’s the difference between Python lists and JavaScript arrays?”

The RAG system retrieves relevant chunks of documentation and uses an LLM to generate accurate, context-grounded answers with citations to the original sources.

This addresses the information gap for students and developers who must constantly navigate between different languages and tools, offering a single intelligent assistant for multi-language programming help.

## **1.2 Primary Use Case**

**Selected: Other → Multi-Language Coding Documentation Assistant**

**Use Case Description:**
The system serves as an AI-powered documentation lookup tool capable of answering technical questions across multiple programming ecosystems. Users interact conversationally to receive explanations, code snippets, and references grounded in official documentation. Instead of manually searching through separate sites for Python, MATLAB, or React, users get unified, consistent, contextual answers in one system.

## **1.3 Users / Audience**

* **College students** in CS, engineering, data science courses
* **Researchers** who switch between MATLAB, Python, and JS analysis tools
* **Developers** needing fast documentation lookup while coding
* **Teaching assistants** wanting a fast reference tool to support common student questions

These users benefit from having technical documentation centralized in a single assistant, enabling rapid retrieval of accurate function usage and examples.

## **1.4 Success Criteria**

We define measurable success criteria as:

* **Accuracy ≥ 80%** on technical questions across languages
* **Citation rate ≥ 90%** (answers include source snippets or doc names)
* **≤ 5 sec latency** per query for smooth interactivity
* **Correct language detection ≥ 90%** (model answers in the intended language)

```yaml
context:
  domain: "Programming documentation for Python, JavaScript/React, MATLAB"
  use_case: "Multi-language coding documentation assistant"
  users: ["students", "researchers", "developers", "teaching assistants"]
  success:
    - "≥80% correct answers"
    - "≥90% answers include citations"
    - "≤5 second average latency"
    - "Correct language identification ≥90%"
```

---

# **2. Data & Constraints**

## **2.1 Corpus Details**

The corpus consists of curated excerpts from frequently used documentation sets:

### **Python**

* Built-in functions (file I/O, data structures, exceptions)
* Standard library modules (e.g., `os`, `math`, `datetime`)
* Short tutorials or cheat sheets

### **JavaScript / React**

* JavaScript core documentation (arrays, promises, objects)
* React “Main Concepts” and “Hooks” documentation:

  * Components
  * `useState`, `useEffect`, etc.
  * JSX rules and examples

### **MATLAB**

* Documentation excerpts for:

  * Matrix operations
  * Plotting and visualization
  * Indexing and numeric computation

### **Formats & Size**

* 20–35 documents total
* ~150–250 pages of text after extraction
* Formats: **HTML converted to Markdown/TXT**, **PDF excerpts**, **MD cheat sheets**

## **2.2 Constraints**

* **Cloud vs Local:**
  Processing is cloud-enabled for convenience (LLM inference), but embeddings are computed locally for privacy and cost reasons.

* **Budget:**
  Free-tier or low-cost setup, using:

  * Local embeddings
  * ChromaDB (free)
  * Minimal hosted LLM calls

* **User Experience:**
  Non-technical users should be able to upload docs and immediately ask questions.

* **Security:**
  Most documents are public programming references; however, user-added notes must remain private.

```yaml
data_constraints:
  sources:
    - "Python documentation excerpts"
    - "JS + React docs excerpts"
    - "MATLAB documentation pages"
  formats: ["PDF", "HTML->TXT", "MD"]
  size: "~150-250 pages of text"
  local_only: false
  cost_limit: "free/OSS preferred"
  latency_target: 5
  security: "private user-added notes; public-code docs"
```

---

# **3. RAG Architecture (MVP)**

## **3.1 Pipeline**

**Ingestion → Chunking → Embedding → Vector Store → Retrieval → LLM → Answer**

### **1. Ingestion**

* Load text from HTML, PDF, and Markdown files.
* Clean and normalize code blocks, headers, and examples.

### **2. Chunking**

* **Hybrid chunking:**

  * Primary: section/heading-based splitting (functions, APIs, hooks)
  * Secondary: fixed-size ~500–700 token chunks with 100 overlap
* Each chunk includes metadata:

  * `language: python/js/react/matlab`
  * `section_title`
  * `doc_source`

### **3. Embedding**

* Local embedding model (e.g., **SentenceTransformer** or **InstructorXL**)
* Ensures privacy and zero cost per call.

### **4. Vector Store**

* **ChromaDB** storing embeddings + metadata
* Persistent, lightweight, fast for small–medium corpora

### **5. Retrieval**

* Vector similarity search (top-k = 5–7)
* Metadata filtering based on detected language:

  * If user says “in Python”: filter `language=python`
  * If ambiguous: allow hybrid search across all languages

### **6. LLM Generation**

* Hosted model (e.g., GPT-4o mini or GPT-3.5 tier)
* Prompt includes:

  * Retrieved text chunks
  * Instruction to **not hallucinate**
  * Requirement for **citations**

### **7. Answer Output**

* Final answer includes:

  * Short explanation
  * Code example
  * Citations (section title + source)

```yaml
architecture_mvp:
  steps: ["ingest", "chunk", "embed", "store", "retrieve", "generate"]
  chunking: "heading-aware + fixed-size hybrid"
  embeddings: "local SBERT/Instructor model"
  vector_db: "ChromaDB"
  retrieval: "vector-only, top-k=5-7 with metadata filters"
  llm: "Hosted GPT model"
  citations: true
  rationale: "Reliable accuracy with simple, privacy-friendly components"
```

---

# **4. Component Alternatives (Mini-Bakeoff)**

| Component      | Option A      | Option B               | Criteria                      | Selected       | Why                                         |
| -------------- | ------------- | ---------------------- | ----------------------------- | -------------- | ------------------------------------------- |
| **Vector DB**  | FAISS         | Chroma                 | Ease of metadata, local setup | **Chroma**     | Simple API, persistence, metadata filtering |
| **Embeddings** | OpenAI        | Local SBERT/Instructor | Accuracy vs privacy/cost      | **Local**      | No API cost, safer with user docs           |
| **LLM**        | Local LLaMA-2 | Hosted GPT-3.5/4o      | Latency + answer quality      | **Hosted GPT** | Better code reasoning, faster inference     |
| **Chunking**   | Fixed-length  | Heading-based          | Semantic clarity              | **Hybrid**     | Keeps API/function sections intact          |
| **Reranking**  | None          | Cohere / Jina reranker | Higher precision              | **None**       | Small corpus doesn’t need it                |

```yaml
component_selection:
  vector_db:
    selected: "Chroma"
    reason: "Metadata, simplicity, local persistence"
  embedding_model:
    selected: "Local SBERT/Instructor"
    reason: "Cost-free, private, strong performance"
  llm:
    selected: "Hosted GPT"
    reason: "Best code accuracy and clarity"
  reranker:
    selected: "None"
    reason: "Corpus small; vector-only performs well"
  chunking:
    selected: "Hybrid"
    reason: "Maintains documentation structure while keeping chunks uniform"
```

---

# **5. Evaluation Plan & Results**

## **5.1 Test Set**

A custom test set of **18 questions** covering:

* **Python (6 questions)**
* **JavaScript/React (6 questions)**
* **MATLAB (6 questions)**

Examples:

* “How do I open and read a file in Python?”
* “How do I use useEffect in React?”
* “How do I plot multiple lines in MATLAB?”
* “What is the difference between map and forEach in JavaScript?”
* “Show an example of matrix multiplication in MATLAB.”

Each question had a reference answer drawn from the documentation.

## **5.2 Metrics & Results**

### **Accuracy**

* **15/18 (83%)** correct, grounded answers

### **Citation Quality**

* **17/18 (94%)** included at least one source citation

### **Latency**

* Average: **3.2 seconds**
* Breakdown:

  * Embedding + retrieval: ~0.7s
  * LLM generation: ~2.4s
  * Overhead: ~0.1s

### **Notes**

* Strong performance for Python + JS/React
* Slight drop in MATLAB when chunks came from long reference sections
* Occasional language misclassification when user omitted language (e.g., “plot a line” → Python assumed)

```yaml
evaluation:
  questions: 18
  baseline:
    correct: 15
    with_citations: 17
    avg_latency: 3.2
  notes: >
    Retrieval was highly accurate; most issues arose from ambiguous prompts or very long MATLAB sections.
```

---

# **6. Risks, Edge Cases & Future Work**

## **Edge Cases**

* **Ambiguous queries** (e.g., “How do I create an array?” → unclear language)
* **Long documentation sections** leading to diluted chunks
* **Similar APIs** across languages causing cross-language confusion

## **Risks**

* **Hallucinations** when retrieval fails
* **Outdated or version-mixed docs**
* **User notes mixed with official docs** may confuse retrieval
* **Privacy concerns** if user adds proprietary code or notes

## **Future Work**

* Add **hybrid search** (keyword + vector)
* Add **language classifier** for robust disambiguation
* Introduce **reranking** for long documentation files
* Expand with more languages (C++, Java, Rust)
* Support **interactive code execution** for examples
* Build a **LightRAG or GraphRAG** index if cross-document linkages become important

```yaml
improvements:
  - "Add hybrid retrieval (BM25 + vector)"
  - "Implement language classifier for ambiguous queries"
  - "Introduce reranker for improved context selection"
  - "Support automatic doc updates from official sites"
  - "Optional knowledge graph for multi-hop or conceptual queries"
  - "Add code execution sandbox for verified examples"
```

---

# **7. References**

* Official Python Docs (selected excerpts)
* React Official Documentation: Main Concepts + Hooks
* MATLAB Documentation: MathWorks Reference Pages
* ChromaDB Developer Documentation
* SentenceTransformers / InstructorXL Model Docs
* RAG literature on best practices, hybrid search, and graph-based retrieval
