# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **Ava**, a WhatsApp AI agent built with LangGraph that can engage in conversations, process voice messages, recognize images, and maintain long-term memory. It's inspired by the film Ex Machina and demonstrates production-ready AI agent development.

## Core Architecture

### LangGraph Workflow Structure
The application is built around a LangGraph workflow defined in `src/ai_companion/graph/graph.py`. The main workflow follows this sequence:

1. **memory_extraction_node** - Extracts memories from user messages
2. **router_node** - Determines response type (conversation/image/audio)
3. **context_injection_node** - Injects contextual information
4. **memory_injection_node** - Injects relevant memories
5. **Response nodes** - conversation_node, image_node, or audio_node
6. **summarize_conversation_node** - Summarizes when conversation gets too long

### State Management
The workflow uses `AICompanionState` (extends MessagesState) to track:
- Conversation history and workflow type
- Audio buffers for speech processing
- Image paths for visual processing
- Current activity and memory context
- Conversation summaries

### Memory System
- **Short-term memory**: DuckDB/SQLite for conversation persistence
- **Long-term memory**: Qdrant vector database for semantic memory retrieval
- Memory extraction happens before routing to inject relevant context

### Multi-modal Capabilities
- **Speech-to-Text**: Whisper via Groq API
- **Text-to-Speech**: ElevenLabs API
- **Image Processing**: Llama 3.2 Vision via Groq
- **Image Generation**: FLUX models via Together AI

## Development Commands

### Environment Setup
```bash
# Install uv package manager first
uv venv .venv
source .venv/bin/activate  # macOS/Linux
uv pip install -e .
```

### Local Development
```bash
# Start full application (Chainlit + WhatsApp API + Qdrant)
make ava-run

# Stop services
make ava-stop

# Clean up (removes memory and generated files)
make ava-delete

# Build containers
make ava-build
```

### Code Quality
```bash
# Format code
make format-fix

# Fix linting issues
make lint-fix

# Check formatting
make format-check

# Check linting
make lint-check
```

### Testing LangGraph Workflow
The workflow is defined in `langgraph.json` and can be tested with LangGraph Studio:
```bash
# The main graph is exported at: ./src/ai_companion/graph/graph.py:graph
```

## Environment Configuration

Required environment variables (see `.env.example`):
- `GROQ_API_KEY` - For LLM and Whisper models
- `ELEVENLABS_API_KEY` + `ELEVENLABS_VOICE_ID` - For text-to-speech
- `TOGETHER_API_KEY` - For image generation
- `QDRANT_URL` + `QDRANT_API_KEY` - For vector database (cloud)
- WhatsApp variables for production deployment

## Service Architecture

### Local Development Stack
- **Chainlit Interface**: http://localhost:8000 (main chat interface)
- **FastAPI Backend**: http://localhost:8080 (WhatsApp webhook)
- **Qdrant Database**: http://localhost:6333 (vector database)

### Key Directories
- `src/ai_companion/graph/` - LangGraph workflow definition
- `src/ai_companion/modules/` - Speech, image, and memory modules
- `src/ai_companion/interfaces/` - Chainlit and WhatsApp interfaces
- `notebooks/` - Development notebooks for testing components

## Model Configuration

Models are configured in `settings.py`:
- Text: `llama-3.3-70b-versatile` (main), `gemma2-9b-it` (small tasks)
- Speech: `whisper-large-v3-turbo` (STT), `eleven_flash_v2_5` (TTS)
- Vision: `llama-3.2-90b-vision-preview` (image understanding)
- Image Generation: `black-forest-labs/FLUX.1-schnell-Free`

## Deployment

The application is containerized and can be deployed to Google Cloud Run. See `cloudbuild.yaml` and Docker files for production deployment configuration.