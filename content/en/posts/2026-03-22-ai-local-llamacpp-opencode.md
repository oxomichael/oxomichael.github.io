---
title: "🚀 Local AI Installation Guide: llama.cpp + Docker + OpenCode (AMD Edition)"
date: "2026-03-22T10:00:00+02:00"
draft: false
translationKey: "ai-local-llamacpp-opencode"
---

This guide allows you to set up a local AI development environment leveraging ROCm hardware acceleration on an AMD Ryzen AI 7 processor with a Radeon 860M iGPU.

💻 Target Machine Specifications

    CPU: AMD Ryzen AI 7 350
    GPU: Radeon 860M (RDNA 3.5 Architecture / gfx1150)
    RAM: 32 GB (Shared with iGPU)
    OS: Ubuntu 24.04+ (Recommended)

# 1. Folder Structure
```
/home/your-user/ai-lab/
├── models/             # Folder where you place your .gguf files
└── compose.yaml        # Docker configuration file
```

# 2. The compose.yaml file (Dual Mode: GPU & CPU)

This file defines two services. You can choose which one to launch.
```YAML
services:
  # --- GPU Version (Radeon 860M) ---
  # Uses the optimized image for ROCm (AMD)
  llamacpp-gpu:
    image: ghcr.io/ggml-org/llama.cpp:server-rocm
    container_name: llamacpp-gpu
    restart: unless-stopped
    ports:
      - "8080:8080"
    devices:
      - /dev/kfd:/dev/kfd
      - /dev/dri:/dev/dri
    group_add:
      - video
      - render
    volumes:
      - ./models:/models
    environment:
      # To force recognition of the Strix Point iGPU
      - HSA_OVERRIDE_GFX_VERSION=11.0.0
    command: >
      -m /models/qwen2.5-coder-7b-instruct-q4_k_m.gguf
      --host 0.0.0.0
      --port 8080
      --n-gpu-layers 100
      --ctx-size 32768
      --flash-attn
      --metrics

  # --- CPU Version (Standard) ---
  llamacpp-cpu:
    image: ghcr.io/ggml-org/llama.cpp:server
    container_name: llamacpp-cpu
    ports:
      - "8081:8080"
    volumes:
      - ./models:/models
    command: >
      -m /models/qwen2.5-coder-7b-instruct-q4_k_m.gguf
      --host 0.0.0.0
      --port 8080
      --ctx-size 32768
      --threads 8
```

# 3. Usage

To launch the GPU version (recommended):
```Bash
docker compose up -d llamacpp-gpu
```

To check if the iGPU is being used (logs):
```Bash
docker logs -f llamacpp-gpu
```

    Look for the line: ggml_roc_init: found 1 ROCm devices

4. OpenCode Configuration (opencode.json)

Point to port 8080 (GPU) or 8081 (CPU) depending on your needs.
```JSON
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "llamacpp-docker": {
      "name": "Llama.cpp Docker",
      "npm": "@ai-sdk/openai-compatible",
      "options": {
        "baseURL": "http://127.0.0.1:8080/v1"
      },
      "models": {
        "qwen2.5-coder-7b": {
          "name": "qwen2.5-coder-7b"
        }
      }
    }
  },
  "model": "llamacpp-docker/qwen2.5-coder-7b"
}
```
