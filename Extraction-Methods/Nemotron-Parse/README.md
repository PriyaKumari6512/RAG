# Nemotron-Parse

A lightweight document parsing pipeline for converting PDFs into structured JSON, Markdown, and hierarchical outputs using [NVIDIA Nemotron Parse](https://build.nvidia.com/nvidia/nemotron-parse).

---

## Table of Contents

- [Overview](#overview)
- [Repository Structure](#repository-structure)
- [Output Format](#output-format)
- [Requirements](#requirements)
- [Setup](#setup)
- [Usage](#usage)
- [Example Workflow](#example-workflow)
- [Notes](#notes)
- [License](#license)

---

## Overview

Nemotron-Parse provides a set of Python utilities for:

- Parsing PDFs into page-level structured outputs
- Exporting Markdown with and without bounding boxes
- Generating detection-only outputs
- Building hierarchical document trees from parsed content
- Extracting table-of-contents (TOC) structure from PDFs
- Renaming and copying generated files for downstream workflows

---

## Repository Structure

```
nemotron-parse/
├── nemotron_parse_pipeline.py      # Main PDF parsing pipeline
├── build_hierarchy.py              # Builds hierarchical JSON using a local Ollama model
├── build_hierarchy_llm.py          # Builds hierarchical JSON via Ollama or NVIDIA NIM
├── PyMuPdf_structre_extraction.py  # Extracts PDF TOC/structure using PyMuPDF
├── folder_copy.py                  # Utility to copy generated JSON files between folders
├── rename.py                       # Utility for batch renaming generated hierarchy files
├── .env.example                    # Example environment variable file
└── README.md
```

---

## Output Format

Running the main pipeline produces the following output structure:

```
output/<doc_name>/
├── pages/
│   ├── page_001_markdown_bbox.json
│   ├── page_001_markdown_no_bbox.json
│   ├── page_001_detection_only.json
│   └── ...
├── <doc_name>_markdown_bbox.json
├── <doc_name>_markdown_bbox.md
├── <doc_name>_markdown_no_bbox.json
├── <doc_name>_markdown_no_bbox.md
├── <doc_name>_detection_only.json
└── <doc_name>_detection_only.md
```

The hierarchy builders produce an additional structure file:

```
<doc_name>_structure.json
```

---

## Requirements

### Python Packages

```bash
pip install requests pdf2image Pillow python-dotenv pymupdf
```

### System Dependency

PDF image conversion requires [Poppler](https://poppler.freedesktop.org/):

```bash
# Ubuntu / Debian
sudo apt-get install poppler-utils

# macOS
brew install poppler
```

### Optional

| Tool | Purpose |
|------|---------|
| [Ollama](https://ollama.com) | Local LLM summarization during hierarchy generation |
| NVIDIA API key | NIM-based hierarchy generation via NVIDIA cloud |

---

## Setup

1. **Clone the repository**

   ```bash
   git clone https://github.com/your-username/nemotron-parse.git
   cd nemotron-parse
   ```

2. **Create and activate a virtual environment**

   ```bash
   python -m venv venv
   source venv/bin/activate   # On Windows: venv\Scripts\activate
   ```

3. **Install dependencies**

   ```bash
   pip install requests pdf2image Pillow python-dotenv pymupdf
   ```

4. **Configure environment variables** *(required only for NVIDIA NIM)*

   Copy the example file and add your key:

   ```bash
   cp .env.example .env
   ```

   `.env`:
   ```env
   NVIDIA_API_KEY=nvapi-xxxxxxxxxxxxxxxxxxxx
   ```

---

## Usage

### 1. Parse a single PDF

```bash
python nemotron_parse_pipeline.py --pdf path/to/doc.pdf
```

### 2. Parse a folder of PDFs

```bash
python nemotron_parse_pipeline.py --pdf_dir path/to/pdf_folder/
```

### 3. Parse with a custom output directory and DPI

```bash
python nemotron_parse_pipeline.py --pdf_dir ./pdfs --output_dir ./results --dpi 200
```

---

### 4. Build a hierarchy from parsed JSON

**Single file:**

```bash
python build_hierarchy.py --input output/MyDoc/MyDoc_markdown_bbox.json
```

**Entire results directory:**

```bash
python build_hierarchy.py --results_dir output/
```

**Skip summaries for faster testing:**

```bash
python build_hierarchy.py --results_dir output/ --no_summary
```

---

### 5. Build a hierarchy with LLM support

**Using Ollama (local):**

```bash
python build_hierarchy_llm.py --input output/MyDoc/MyDoc_markdown_bbox.json --provider ollama
```

**Using NVIDIA NIM (cloud):**

```bash
python build_hierarchy_llm.py --input output/MyDoc/MyDoc_markdown_bbox.json --provider nvidia
```

**Process a directory in chunks:**

```bash
python build_hierarchy_llm.py --results_dir output/ --chunk_size 30
```

---

### 6. Extract TOC / structure from PDFs

**Single file:**

```bash
python PyMuPdf_structre_extraction.py --pdf path/to/doc.pdf
```

**Folder of PDFs:**

```bash
python PyMuPdf_structre_extraction.py --pdf_dir path/to/folder/
```

---

### 7. Utility scripts

**Copy generated JSON files:**

```bash
python folder_copy.py
```

**Rename generated hierarchy files:**

```bash
python rename.py
```

---

## Example Workflow

```bash
# Step 1 — Parse PDFs
python nemotron_parse_pipeline.py --pdf_dir ./pdfs --output_dir ./output

# Step 2 — Generate hierarchical structure
python build_hierarchy_llm.py --results_dir ./output --provider ollama

# Step 3 (optional) — Extract TOC metadata
python PyMuPdf_structre_extraction.py --pdf_dir ./pdfs

# Step 4 (optional) — Rename or copy outputs for downstream use
python rename.py
python folder_copy.py
```

---

## Notes

- `build_hierarchy.py` expects the `*_markdown_bbox.json` files produced by the main pipeline.
- `build_hierarchy_llm.py` supports both local (Ollama) and hosted (NVIDIA NIM) LLM backends.
- `PyMuPdf_structre_extraction.py` uses several fallback strategies to infer document structure when an embedded TOC is not available.
- Some utility scripts (`folder_copy.py`, `rename.py`) contain hardcoded paths for local experimentation — update these before use in your own environment.

---

## License

This project is currently unlicensed. Add your preferred license here (e.g., [MIT](https://choosealicense.com/licenses/mit/), [Apache 2.0](https://choosealicense.com/licenses/apache-2.0/)).
