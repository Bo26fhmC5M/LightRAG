# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Installation and Setup
```bash
# Install from source (recommended for development)
pip install -e .

# Install with API dependencies
pip install -e ".[api]"

# Copy environment configuration
cp env.example .env
```

### Running the Application
```bash
# Start LightRAG server
lightrag-server

# Run with Docker Compose
docker compose up

# Run with Gunicorn (production)
lightrag-gunicorn
```

### Testing
```bash
# Run tests
python -m pytest tests/

# Run specific test
python -m pytest tests/test_graph_storage.py
```

## Architecture Overview

### Core Components

**LightRAG Core (`lightrag/lightrag.py`)**
- Main LightRAG class that orchestrates document indexing and querying
- Requires explicit initialization: `await rag.initialize_storages()` and `await initialize_pipeline_status()`
- Supports multiple query modes: local, global, hybrid, naive, mix, bypass

**Storage Layer**
- **KV_STORAGE**: LLM response cache, text chunks, document information
- **VECTOR_STORAGE**: Entity vectors, relation vectors, chunk vectors
- **GRAPH_STORAGE**: Entity-relationship graph
- **DOC_STATUS_STORAGE**: Document indexing status

Each storage type has multiple implementations (JSON, PostgreSQL, Neo4j, MongoDB, Redis, etc.)

**API Server (`lightrag/api/`)**
- FastAPI-based REST API with Web UI
- Ollama-compatible interface for AI chat integration
- Routes in `routers/`: document operations, queries, graph visualization
- Authentication and configuration management

### Key Concepts

**Knowledge Graph Construction**
- Documents are processed to extract entities and relationships using LLMs
- Entity-relationship extraction requires LLMs with ≥32B parameters and 32KB+ context
- Knowledge graph is built incrementally with entity merging and deduplication

**Query Processing**
- Multiple retrieval modes combine vector search with graph traversal
- Reranking can be enabled for improved retrieval quality
- Token usage tracking available for cost monitoring

**Storage Flexibility**
- Workspace-based data isolation between LightRAG instances
- Production deployments typically use PostgreSQL, Neo4j, or specialized vector databases
- Default local storage uses JSON files and NanoVectorDB

### Important Patterns

**Async/Await Usage**
- Core operations are async: `ainsert()`, `aquery()`, `initialize_storages()`
- Always call initialization methods after creating LightRAG instance

**Model Integration**
- LLM and embedding functions are injected during initialization
- Support for OpenAI, Ollama, HuggingFace, and other providers
- Different models can be used for indexing vs querying

**Error Handling**
- Common errors: missing storage initialization, pipeline status not initialized
- Model switching requires clearing data directory (preserve `kv_store_llm_response_cache.json` for LLM cache)

## Project Structure

### Core Modules
- `lightrag/lightrag.py` - Main LightRAG class
- `lightrag/llm/` - LLM integrations (OpenAI, Ollama, HuggingFace, etc.)
- `lightrag/kg/` - Knowledge graph operations and shared storage
- `lightrag/storage/` - Storage backend implementations
- `lightrag/api/` - FastAPI server and web UI

### Examples
- `examples/lightrag_openai_demo.py` - Basic usage with OpenAI
- `examples/lightrag_openai_compatible_demo.py` - Streaming responses
- Other examples demonstrate various LLM providers and storage backends

Note: Only `lightrag_openai_demo.py` and `lightrag_openai_compatible_demo.py` are officially supported; others are community contributions.