# localRAG - Self-hosted RAG demo using n8n

## Executive Summary & Business Impact

`localRAG` is a proof-of-concept Retrieval-Augmented Generation (RAG) workflow
for asking natural-language questions over uploaded PDF documents. It combines
n8n workflow orchestration, Ollama local models, Qdrant vector search, and
PostgreSQL application storage so teams can evaluate private document
intelligence on a local machine before choosing a production architecture.

The business value is a repeatable pattern for secure document Q&A: sensitive
files can stay in a local environment during evaluation, answers are grounded in
retrieved document chunks, and the same workflow can later be adapted for cloud
models, managed infrastructure, and production monitoring.

This repository is adapted from the original
[n8n self-hosted AI starter kit](https://github.com/n8n-io/self-hosted-ai-starter-kit).

## Tech Stack

| Component | Usage in this POC |
| --- | --- |
| [n8n](https://n8n.io/) | Orchestrates the RAG workflow, including chat input, PDF loading, text splitting, embedding generation, vector insertion, retrieval, and answer generation. |
| [Ollama](https://ollama.com/) | Runs the local POC models. `nomic-embed-text` generates embeddings, and `llama3.2` generates natural-language answers. |
| [Qdrant](https://qdrant.tech/) | Stores vector embeddings and retrieves the most relevant chunks for each user question. |
| [PostgreSQL](https://www.postgresql.org/) | Stores n8n application data, workflow state, credential metadata, and configuration. |

For production, replace local Ollama models with managed cloud models for better
reliability, scalability, monitoring, and operational support.

| Local POC model | Production cloud model option | Purpose |
| --- | --- | --- |
| `nomic-embed-text` | `text-embedding-3-small` or `text-embedding-3-large` | Generate embeddings for documents and user queries. |
| `llama3.2` | `gpt-4o-mini` or `gpt-4o` | Generate final answers from retrieved context. |

## What you can build

This POC demonstrates the foundation for private document-intelligence use
cases:

- **Document Q&A over internal PDFs** - Upload policy documents, technical
  papers, product guides, or operational manuals and ask questions grounded in
  the retrieved document context.
- **Private research and knowledge assistants** - Help teams search dense
  reference material without sending sensitive files to third-party AI platforms
  during evaluation.
- **Customer support and field enablement copilots** - Turn product
  documentation, release notes, and troubleshooting guides into searchable
  assistants for support teams, sales engineers, and implementation specialists.
- **Automated document intake and summarization workflows** - Extend the n8n
  workflow to classify uploaded documents, summarize key points, extract fields,
  route exceptions, and trigger downstream business processes.
- **Production-ready RAG prototypes** - Validate chunking, embeddings, retrieval
  quality, and prompt design before moving to managed models and infrastructure.

## Screenshots

### localRAG chat execution

![localRAG n8n chat execution](n8n/demo-data/screenshot/%E2%96%B6%EF%B8%8F_localRAG_-_n8n-chat.png)

### localRAG workflow canvas

![localRAG n8n workflow canvas](n8n/demo-data/screenshot/%E2%96%B6%EF%B8%8F_localRAG_-_n8n.png)

## Prerequisites

- Docker Desktop or Docker Engine with Docker Compose
- Git
- Optional: a local Ollama installation if you want to run Ollama directly on
  your host machine instead of inside Docker

## Installation

```bash
git clone https://github.com/otyeung/localRAG.git
cd localRAG
cp .env.example .env
```

Update `.env` before starting the stack. The example values are placeholders and
should not be used for anything beyond local evaluation.

## Running the stack

### CPU / most local machines

```bash
docker compose --profile cpu up
```

### Nvidia GPU

```bash
docker compose --profile gpu-nvidia up
```

If you have not used an Nvidia GPU with Docker before, follow the
[Ollama Docker instructions](https://github.com/ollama/ollama/blob/main/docs/docker.md).

### AMD GPU on Linux

```bash
docker compose --profile gpu-amd up
```

### Mac / Apple Silicon with host Ollama

Docker cannot expose Apple Silicon GPU acceleration to the Ollama container. For
better local inference performance on Mac, install Ollama on the host and run:

```bash
ollama pull llama3.2
ollama pull nomic-embed-text
docker compose up
```

Set this value in `.env` before running Docker Compose:

```bash
OLLAMA_HOST=host.docker.internal:11434
```

When configuring the n8n Ollama credential, use
`http://host.docker.internal:11434` as the base URL. Qdrant still runs in Docker,
so the Qdrant credential URL should remain `http://qdrant:6333`.

## n8n setup

1. Open <http://localhost:5678/> and complete the first-time n8n setup.
2. The `localRAG` workflow is imported automatically on first startup when the
   n8n database is empty.
3. Open the workflow at <http://localhost:5678/workflow/B69QT7wHIUKI48g1>.
4. Create or select these n8n credentials if the imported workflow shows missing
   credential warnings:

| Credential | URL |
| --- | --- |
| Ollama account, Docker Ollama | `http://ollama:11434` |
| Ollama account, host Ollama on Mac | `http://host.docker.internal:11434` |
| Qdrant account | `http://qdrant:6333` |

The Docker Ollama profiles pull both required local models on startup:
`llama3.2` and `nomic-embed-text`.

## Quick start and usage

1. Open the `localRAG` workflow in n8n.
2. Click **Chat** at the bottom of the canvas.
3. Attach one of the sample PDFs.
4. Ask the matching test question.

### Sample PDF corpus and test questions

| Corpus | Test question |
| --- | --- |
| `1706.03762v7.pdf` | What specific hardware setup and optimizer were used to train the base and big Transformer models? Additionally, how long did the training take for each model, and what were their final BLEU scores on the WMT 2014 English-to-German dataset? |
| `cymbal-starlight-2024.pdf` | What is the cargo capacity of Cymbal Starlight? |

## Upgrading

Pull newer container images and recreate the stack with the same profile you use
to run it:

```bash
docker compose --profile cpu pull
docker compose --profile cpu up
```

Use `gpu-nvidia` or `gpu-amd` instead of `cpu` for GPU profiles.

## License

This project is licensed under the Apache License 2.0. See [LICENSE](LICENSE)
for details.
