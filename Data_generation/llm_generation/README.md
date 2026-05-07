# 🧠 LLM Question Generation Pipeline

> An end-to-end pipeline for automatically generating high-quality Q&A datasets from PDF documents — built for RAG evaluation using RAGAS, local LLMs (Ollama/Qwen), and NVIDIA NIM endpoints.

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue?logo=python)](https://www.python.org/)
[![RAGAS](https://img.shields.io/badge/RAGAS-v0.2%2B-orange)](https://docs.ragas.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## Overview

This project automates the creation of evaluation datasets for Retrieval-Augmented Generation (RAG) systems. Given a directory of PDFs, it extracts document structure, generates diverse questions across difficulty levels and reasoning types, performs quality validation, and exports datasets ready for RAGAS evaluation.

It supports multiple LLM backends — NVIDIA NIM, Ollama (local), and Azure OpenAI — making it flexible for both cloud and on-premise setups.

---

## Project Structure

```
llm_generation/
├── generate_dataset.py                       # RAGAS v0.2+ dataset generator (NVIDIA / Ollama)
├── generate_dataset_v2.py                    # Improved generator with checkpointing & parallelism
├── LLM_Question_Generation.py                # Q&A generation via Azure OpenAI GPT-4o-mini
├── LLM_Question_Generation_Qwen.py           # Q&A generation via Ollama (Qwen2.5)
├── LLM_Question_Generation_Qwen_improved.py  # AUTOSAR-aware, difficulty-stratified generation
├── Q_generation_new.py                       # Latest pipeline with semantic deduplication
├── create_GT.py                              # Ground truth builder (PageIndex + RAGAS + NVIDIA NIM)
└── LLM_Question_Quality_check.py             # Quality verification and reporting tool
```

---

## Components

### `generate_dataset_v2.py` ⭐ Recommended
The most robust RAGAS-based pipeline. Key features:
- Per-leaf **checkpointing** with atomic writes — safe to restart mid-run
- `raise_exceptions=False` on KG transforms — single-node failures don't abort the run
- vLLM health check before each step
- Configurable LLM and embedding providers (NVIDIA / Ollama / sentence-transformers)
- Parallel workers for KG transforms and question generation

### `generate_dataset.py`
RAGAS v0.2+ pipeline using the default query distribution:
- `SingleHopSpecificQuerySynthesizer` (weight 0.5)
- `MultiHopAbstractQuerySynthesizer` (weight 0.25)
- `MultiHopSpecificQuerySynthesizer` (weight 0.25)

Supports NVIDIA NIM and Ollama backends, KG persistence, and custom question counts.

### `LLM_Question_Generation_Qwen_improved.py` / `Q_generation_new.py`
AUTOSAR-aware generation with difficulty stratification:

| Document Complexity | Question Type | Difficulty |
|---------------------|---------------|------------|
| Pure prose | Lookup | Easy |
| Prose + requirement IDs | Parameter extraction | Medium |
| Tables / parameter lists | Relationship questions | Hard |
| Cross-document references | Requirement tracing | Hardest |

Includes optional **semantic deduplication** via BGE-M3 embeddings.

### `create_GT.py`
Builds ground truth datasets from PageIndex tree outputs + PDFs using NVIDIA NIM (Llama 3.1 405B). Outputs a HuggingFace `Dataset` ready for `ragas.evaluate()`.

### `LLM_Question_Quality_check.py`
Validates generated question sets with automated checks: type/difficulty distribution analysis, multi-hop ratio verification (target: 30%+), warnings, and actionable recommendations.

---

## Installation

### Prerequisites

- Python 3.8+
- Ollama running locally (`ollama serve`) — for local LLM inference

### Setup

```bash
git clone https://github.com/your-username/llm-generation.git
cd llm-generation

python -m venv venv
source venv/bin/activate        # macOS/Linux
venv\Scripts\activate           # Windows
```

### Install Dependencies

**For NVIDIA NIM backend:**
```bash
pip install ragas langchain langchain-nvidia-ai-endpoints \
            langchain-community pypdf nest_asyncio rapidfuzz \
            pdfplumber PyPDF2 tqdm datasets openai
```

**For Ollama (local) backend, additionally:**
```bash
pip install langchain-ollama
```

---

## Usage

### Generate a dataset (recommended)

```bash
# NVIDIA for both LLM and embeddings (default)
python generate_dataset_v2.py --pdf-dir ./my_pdfs

# Ollama for both (fully local)
python generate_dataset_v2.py --pdf-dir ./my_pdfs \
    --llm-provider ollama --embed-provider ollama

# Custom question count and output directory
python generate_dataset_v2.py --pdf-dir ./my_pdfs -n 100 --output-dir ./results

# Reuse an existing Knowledge Graph (skip expensive rebuild)
python generate_dataset_v2.py --pdf-dir ./my_pdfs -n 50 --load-kg
```

### Run quality checks

```bash
python LLM_Question_Quality_check.py
```

### Build a ground truth dataset

```bash
python create_GT.py
```

---

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `NVIDIA_API_KEY` | When using NVIDIA provider | Your NVIDIA NIM API key |
| `OLLAMA_BASE_URL` | Optional | Ollama server URL (default: `http://localhost:11434`) |
| `OLLAMA_MODEL` | Optional | Model to use (default: `qwen2.5:72b`) |
| `ENABLE_SEMANTIC_DEDUP` | Optional | Enable BGE-M3 deduplication (`true`/`false`) |
| `SEMANTIC_SIMILARITY_THRESHOLD` | Optional | Dedup threshold (default: `0.85`) |

---

## Workflow

```
PDF Documents
      │
      ▼
PDF Extraction (pdfplumber + PyPDF2)
      │
      ▼
Knowledge Graph Construction (RAGAS)
      │
      ▼
Question Generation (NVIDIA / Ollama / Azure OpenAI)
      │
      ▼
Quality Verification & Deduplication
      │
      ▼
RAGAS EvaluationDataset / HuggingFace Dataset
      │
      ▼
RAG System Evaluation
```

---

## Output

| File | Description |
|------|-------------|
| `knowledge_graph.json` | Persisted RAGAS knowledge graph for reuse |
| `dataset.csv` | Generated Q&A pairs in RAGAS format |
| `gt_dataset.json` | Full ground truth dataset with metadata |
| `ragas_eval_ready/` | HuggingFace Dataset for `ragas.evaluate()` |

---

## Use Cases

- Building evaluation benchmarks for RAG pipelines
- Testing retrieval quality on technical / specification documents (e.g., AUTOSAR)
- Generating synthetic Q&A for LLM fine-tuning or few-shot prompting
- Automated knowledge base quality assessment

---

## Author

**Praveen Solanki**  
M.Tech CSE — IIT Mandi

---

## License

This project is licensed under the [MIT License](LICENSE).
