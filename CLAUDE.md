# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Lightning Whisper MLX is a high-performance implementation of OpenAI's Whisper speech-to-text model optimized for Apple Silicon, built on top of MLX (Apple's machine learning framework). The project claims 10x faster performance than Whisper CPP and 4x faster than existing MLX Whisper implementations.

## Development Commands

### Basic Usage
```bash
python test.py  # Simple test script using tiny model
```

### Installation
```bash
pip install lightning-whisper-mlx  # Install from PyPI
pip install -e .  # Install in development mode from source
```

### Building/Distribution
```bash
python setup.py sdist bdist_wheel  # Build distribution packages
```

### Testing
- The project uses a simple `test.py` script for basic functionality testing
- No formal test framework is currently configured
- Run: `python test.py` to test with the tiny model and a local audio file

## Architecture Overview

### Core Components

1. **LightningWhisperMLX** (`lightning.py`): Main user-facing class
   - Handles model downloading from HuggingFace Hub
   - Supports quantized models (4bit, 8bit) for memory efficiency
   - Manages batch processing configuration
   - Downloads and caches models locally in `./mlx_models/`

2. **Whisper Model** (`whisper.py`): Core neural network implementation
   - `AudioEncoder`: Processes mel spectrograms to audio features
   - `TextDecoder`: Generates text tokens from audio features using cross-attention
   - `MultiHeadAttention`: Attention mechanism with optional cross-attention
   - `ResidualAttentionBlock`: Transformer blocks with residual connections

3. **Audio Processing** (`audio.py`): Audio preprocessing pipeline
   - Handles audio loading via ffmpeg subprocess
   - Converts audio to mel spectrograms using STFT
   - Uses pre-computed mel filter banks (assets/mel_filters.npz)
   - Fixed hyperparameters: 16kHz sample rate, 400 FFT size

4. **Transcription Engine** (`transcribe.py`): Main transcription logic
   - Implements batched decoding for improved throughput
   - Handles language detection for multilingual models
   - Supports fallback decoding with different temperatures
   - Processes audio in 30-second segments with configurable overlap

5. **Model Loading** (`load_models.py`): MLX model initialization
   - Loads model weights and configuration from local/HF paths
   - Handles quantization application during model loading
   - Uses MLX's tree_unflatten for weight management

### Key Features
- **Batched Processing**: Configurable batch_size (default 12) for throughput optimization
- **Model Variants**: Supports tiny, small, base, medium, large variants including distilled versions
- **Quantization**: 4bit/8bit quantization support for reduced memory usage
- **Language Detection**: Automatic language detection for multilingual models

### Model Storage
- Models downloaded to `./mlx_models/{model_name}/` directory
- Each model contains `weights.npz` and `config.json` files
- Distilled models use different directory structure

### Dependencies
- **mlx**: Apple's ML framework for model execution
- **huggingface_hub**: Model downloading and caching
- **torch**: Some compatibility layers (minimal usage)
- **ffmpeg**: Required system dependency for audio processing

## File Structure
```
lightning_whisper_mlx/
├── __init__.py          # Package entry point
├── lightning.py         # Main user interface class
├── whisper.py          # Core Whisper model implementation
├── transcribe.py       # Transcription logic and batching
├── audio.py           # Audio preprocessing utilities
├── load_models.py     # Model loading and initialization
├── decoding.py        # Decoding strategies and options
├── tokenizer.py       # Text tokenization utilities
├── timing.py          # Word-level timestamp alignment
├── torch_whisper.py   # Torch compatibility layer
└── assets/
    ├── mel_filters.npz    # Pre-computed mel filter banks
    ├── *.tiktoken         # Tokenizer files
    └── test audio files
```

## Development Notes

- The codebase prioritizes performance on Apple Silicon through MLX optimization
- Batch processing is the key performance differentiator from other implementations
- Model quantization significantly reduces memory usage with minimal accuracy loss
- Audio processing uses ffmpeg subprocess calls rather than Python audio libraries
- The project focuses on inference only - no training capabilities included