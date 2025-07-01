# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Memvid is a Python library for QR code video-based AI memory that enables:
- Chunking and encoding text data into QR code videos  
- Fast semantic search and retrieval from QR videos
- Conversational AI interface with context-aware memory
- Storage of millions of text chunks in compact MP4 files with sub-second retrieval

## Key Architecture

### Core Components
- **MemvidEncoder** (memvid/encoder.py): Handles text chunking and QR video creation with Docker/FFmpeg support
- **MemvidRetriever** (memvid/retriever.py): Fast semantic search, QR frame extraction, context assembly
- **MemvidChat** (memvid/chat.py): Manages conversations, context retrieval, and LLM interface  
- **IndexManager** (memvid/index.py): Embedding generation, storage, and vector search using FAISS
- **LLMClient** (memvid/llm_client.py): Multi-provider LLM support (OpenAI, Google, Anthropic)
- **DockerManager** (memvid/docker_manager.py): Docker backend for advanced video codecs
- **Utils** (memvid/utils.py): QR code generation/decoding, video processing utilities
- **Config** (memvid/config.py): Configuration defaults and codec parameters

### Data Flow
1. Text chunks → Embeddings → QR codes → Video frames
2. Query → Semantic search → Frame extraction → QR decode → Context  
3. Context + History → LLM → Response

## Development Commands

### Environment Setup
```bash
# Create and activate virtual environment
python -m venv .memvid
source .memvid/bin/activate  # On macOS/Linux
# On Windows: .memvid\Scripts\activate

# Install core dependencies  
pip install -r requirements.txt

# Install package in development mode
pip install -e .

# Install with all optional dependencies
pip install -e ".[dev,llm,epub,web]"
```

### Testing
```bash
# Run all tests
pytest tests/

# Run with coverage
pytest --cov=memvid tests/

# Run specific test file
pytest tests/test_encoder.py

# Run specific test function
pytest tests/test_encoder.py::TestEncoder::test_specific_function
```

### Code Quality
```bash  
# Format code
black memvid/

# Lint code
flake8 memvid/
```

### Docker-based H.265 Encoding
```bash
# Check setup and build container
make setup
make build

# Test container functionality
make test
make test-ffmpeg
make test-workflow

# Clean up Docker resources
make clean
```

### Example Usage
```bash
# Process documents with file_chat.py
python examples/file_chat.py --input-dir /path/to/docs --provider google
python examples/file_chat.py --files doc1.txt doc2.pdf --provider openai
python examples/file_chat.py --input-dir docs/ --codec h265 --provider google

# Load existing memory
python examples/file_chat.py --load-existing output/my_memory --provider google
```

## Key Dependencies

### Core Dependencies
- **qrcode[pil]**: QR code generation with image support
- **opencv-python + opencv-contrib-python**: Video processing and QR decoding
- **sentence-transformers**: Semantic embeddings for search
- **faiss-cpu**: Fast vector similarity search
- **numpy**: Vector operations (compatible with <2.0.0)
- **Pillow**: Image processing
- **tqdm**: Progress bars
- **PyPDF2**: PDF text extraction

### Optional Dependencies
- **LLM providers**: openai, google-generativeai, anthropic
- **Web interface**: fastapi, gradio  
- **EPUB support**: beautifulsoup4, ebooklib
- **Development**: pytest, pytest-cov, black, flake8

## Performance Requirements
- Retrieval (search + QR decode) must be < 2 seconds for 1M chunks
- Use batching and parallel processing for frame extraction
- Implement caching for hot frames and common queries
- Default embedding model: sentence-transformers all-MiniLM-L6-v2
- Video compression: H.265 preferred for optimal storage efficiency

## Configuration Options

### QR Code Settings  
- Version 35 QR codes (177x177 modules)
- Error correction level M
- Configurable box size and border

### Video Codec Support
- **h264**: Standard codec, wide compatibility
- **h265**: Better compression, requires Docker backend  
- **av1**: Next-gen codec, experimental support
- Fallback chain: h265 → h264 → libx264

### Chunking Parameters
- Default chunk size: 1024 characters
- Default overlap: 32 characters  
- Configurable via MemvidEncoder parameters

## Implementation Notes
- Uses FAISS for vector similarity search (CPU-optimized)
- Multi-provider LLM support with unified interface
- Docker integration for advanced codec support
- Thread pools for parallel QR decoding
- Disk-based index storage for large-scale deployments
- Cross-platform Makefile for Docker workflows
- Comprehensive example scripts in /examples/ directory